# docker-compose

### ports和expose的区别

#### ports

**ports暴露容器端口到主机的任意端口或指定端**口，用法：

```bash
ports:
  - "80:80"         # 绑定容器的80端口到主机的80端口
  - "9000:80"       # 绑定容器的80端口到主机的9000端口
  - "443"           # 绑定容器的443端口到主机的任意端口，容器启动时随机分配绑定的主机端口号
```

不管是否指定主机端口，使用ports都会将端口暴露给**主机和其他容器**。

#### expose

**expose暴露容器给link到当前容器的容器，或者暴露给同一个networks的容器**，用法：

```bash
expose:
  - "3000"
  - "8000"
```

以上指令将当前容器的端口3000和8000暴露给**其他容器**。

**和ports的区别是，expose不会将端口暴露给主机，主机无法访问expose的端口**。

#### 示例

以下的`docker-compose.yml`的作用是使用`keycloak-gatekeeper`给`tomcat`做一个代理认证。`tomcat`服务使用`expose`暴露了`8080`端口；而`tomcat-proxy`服务使用`ports`暴露了`3000`端口并映射到host的`8080`端口。 同时`tomcat-proxy`容器和`tomcat`容器是在同一个容器网络平面中的。
由于`expose`并没有映射容器端口到主机端口，因此**在host上直接访问`127.0.0.0:8080`并不会直接访问`tomcat`服务**。
在该示例中， 访问`127.0.0.1:8080`会走到`tomcat-proxy`服务，在`tomcat-proxy`上完成认证后，通过配置的`upstream-url`参数**走容器网络平面**跳转到`tomcat`服务。



```bash
version: "3"

services:
  tomcat:
    image: tomcat
    container_name: tomcat
    restart: always
    expose:
      - 8080

  tomcat-proxy:
    container_name: tomcat-proxy
    image: keycloak/keycloak-gatekeeper
    restart: unless-stopped
    command: >
      --discovery-url=https://keycloak.dev.com/auth/realms/demo
      --upstream-url=http://tomcat:8080
      --redirection-url=http://hello-world.example.org:8080
      --client-id=gatekeeper
      --client-secret=xxxxxxxxxx
      --encryption-key=xxxxxxxxxx
      --cookie-domain=example.org
      --listen=0.0.0.0:3000
      --enable-refresh-tokens=true
      --enable-encrypted-token=true
      --enable-logout-redirect=true
      --enable-token-header=false
      --enable-authorization-header=false
      --secure-cookie=false
      --forbidden-page=/forbidden.html.tmpl
      --resources="uri=/*"
    volumes:
      - ./forbidden.html:/forbidden.html.tmpl
    ports:
      - 8080:3000
```

### 启动容器立马退出

1. docker-compose启动容器后就退出，查看日志没有报错
   ![docker-compose启动容器后自动退出](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzM5OC9mN2ZlODhkOGMwYTBmYzA5YTYwNDgzNmU3ZjE4MTVmZS5wbmc=)![docker-compose启动容器后自动退出](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzUxNy9lZTcwNDcyMDFhM2IyMTM3Zjg5NmQ1OTUxODkyNThkNS5wbmc=)
2. 查看资料发现是因为缺失了控制终端的配置，所以容器启动后就会退出，解决方法在docker-compose.yml文件增加tty: true
3. 重启后容器不再退出。
   ![docker-compose启动容器后自动退出](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/img)

### [volume 挂载时文件或文件夹不存在](https://segmentfault.com/a/1190000015684472)

#### 背景介绍

docker volume 可以使我们在启动docker容器时，动态的挂载一些文件（如配置文件）, 以覆盖镜像中原有的文件，但是，挂载一个主机上尚不存在的文件夹或者文件到容器中会怎样呢？LZ在工作中就遇到了这样的问题，故自己实践了一下，记录实验结果如下：

#### 文件夹挂载

docker在文件夹挂载上的行为是统一的，具体表现为：

- **若文件夹不存在，则先创建出文件夹**（若为多层文件夹，则递归创建）
- **用host上的文件夹内容覆盖container中的文件夹内容**

```bash
docker run -v /path-to-folder/A:/path-to-folder/B test-image
```

详细说明如下：

##### host上文件夹存在，且非空

| host              | container         | mount result                                                 |
| ----------------- | ----------------- | ------------------------------------------------------------ |
| 存在的非空文件夹A | 不存在的文件夹B   | 先在contanier中创建文件夹B，再将A文件夹中的所有文件copy到B中 |
| 存在的非空文件夹A | 存在的非空文件夹B | **先将container中文件夹B的原有内容清空**，再将A中文件copy到B中 |

> 无论container中的文件夹B是否存在， A都会完全覆盖B的内容

##### host上文件夹存在，但为空

| host            | container         | mount result                   |
| --------------- | ----------------- | ------------------------------ |
| 存在的空文件夹A | 存在的非空文件夹B | container中文件夹B的内容被清空 |

> container中对应的文件夹内容被清空

##### host上文件夹不存在

| host                | container         | mount result                                            |
| ------------------- | ----------------- | ------------------------------------------------------- |
| 不存在的文件夹A     | 存在的非空文件夹B | 在host上创建文件夹A，container中文件夹B的内容被清空     |
| 不存在的文件夹A/B/C | 存在的非空文件夹B | 在host上创建文件夹A/B/C，container中文件夹B的内容被清空 |

> container中对应的文件夹内容被清空

##### 总结

**host上文件夹一定会覆盖container中文件夹**：

| host                          | container                            | mount result                                                 |
| ----------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| 文件夹不存在/文件夹存在但为空 | 文件夹不存在/存在但为空/存在且不为空 | container中文件被覆盖（清空）                                |
| 文件夹存在且不为空            | 文件夹不存在/存在但为空/存在且不为空 | container中文件夹内容被覆盖（原内容清空， 覆盖为host上文件夹内容） |

#### 文件挂载

文件挂载与文件夹挂载最大的不同点在于：

- **docker 禁止用主机上不存在的文件挂载到container中已经存在的文件**
- 文件挂载不会对同一文件夹下的其他文件产生任何影响

除此之外， 其覆盖行为与文件夹挂载一致，即：

- 用host上的文件的内容覆盖container中的文件的内容

```bash
docker run -v /path-to-folder/non-existent-config.js:/path-to-folder/config.js test-image # forbidden
```

详细说明如下：

##### host上文件不存在

|          host          |         container         | mount result                                                 |
| :--------------------: | :-----------------------: | ------------------------------------------------------------ |
| 不存在的文件configA.js | 已经存在的文件congfigB.js | 报错，Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type. 同时会在host上生成两个空目录 configA.js 和 configB.js, 但是container无法启动 |

##### host上文件存在

| host                 | container               | mount result                                                 |
| -------------------- | ----------------------- | ------------------------------------------------------------ |
| 存在的文件configA.js | 存在的文件congfigB.js   | container中文件名configB.js保持不变,但是文件内容被congfigA.js的内容覆盖了 |
| 存在的文件configA.js | 不存在的文件congfigB.js | container中新建一个文件configB.js，其内容为configA.js的文件内容， configB.js所在文件下的所有其他文件维持不变 |

##### 总结

**host上文件一定会覆盖container中文件夹**

| host         | container                   | mount result                           |
| ------------ | --------------------------- | -------------------------------------- |
| 不存在的文件 | 已经存在的文件              | 禁止行为                               |
| 存在的文件   | 不存在的文件/已经存在的文件 | 新增/覆盖 （若目录不存在则会创建目录） |

##### 结论

#### 文件夹挂载

- 允许不存在的文件夹或者存在的空文件夹挂载进container, container中对应的文件夹将被清空
- **非空文件夹挂载进container将会覆盖container中原有文件夹**

#### 文件挂载

- 禁止将不存在的文件挂载进container中已经存在的文件上
- 存在的文件挂载进container中将会覆盖container中对应的文件， 若文件不存在则新建

#### 应用场景

1. 从上面的分析可知，文件夹挂载以整个文件夹为单位进行文件覆盖，故可在需要将大量文件挂载进container时使用，另外，如果挂载一个空文件夹或者不存在的文件夹，一般是做逆向使用： 即容器启动后，可能会在容器内挂载点的文件夹下生成一些文件（如日志），此时，在对应的host上的文件夹内就能直接看到。
2. 文件挂载由于只会覆盖单个文件而不会影响container中同一文件夹下的其他文件，常常被用来挂载配置文件，以在运行时，动态的修改默认配置。