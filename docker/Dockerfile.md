# Dockerfile

### [Alpine Docker 容器中无法找到自己添加的二进制可执行文件](https://naiv.fun/Ops/binary-not-found-in-Alpine.html)

#### 问题描述

问题出现场景上面已经提到过了，这里举个例子，`dockerfile` 如下：

```dockerfile
FROM alpine
ADD recruit /usr/local/bin
RUN chmod a+x /usr/local/bin/recruit
CMD "recruit"
```

经过检查 `/usr/local/bin` 确实在 `$PATH` 里面，看着应该是没什么问题的。

但是当我试图启动容器测试时：

```shell
$ docker run --rm backend
/bin/sh: recruit: not found
```

随后我试图进入容器手动启动程序：

```shell
$ docker run --rm -it backend /bin/sh      # 进入容器
/ # echo $PATH                               # 这个目录确实在 $PATH 里
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
/ # ls /usr/local/bin/ -lh | grep recruit  # 可以发现确实存在这个程序，也有执行权限
-rwxr-xr-x    1 root     root       19.7M Jun 18 07:58 recruit
/ # ./usr/local/bin/recruit                # 但就是运行不了
/bin/sh: ./usr/local/bin/recruit: not found
```

总之非常玄学，死活不给你运行，下面讲一下解决方案。

#### 出现原因

原因非常令人大跌眼镜。

这里报的 `not found` 并不是说找不到这个程序，而是说，程序在运行的时候无法找到需要动态链接的目标文件。

那么什么是动态链接呢？

#### 动态链接

我们都知道，程序编写完毕后需要编译、链接才能运行。链接是把目标文件、操作系统的启动代码和用到的库文件进行组织，最终形成可执行代码的过程。按理来讲，我们的代码变成可执行的程序的时候，需要调用的代码都已经被链接好了，但是这并不完全准确，因为在此时完成的只是静态链接。

所谓静态链接是指**把要调用的函数或者过程链接到可执行文件中，成为可执行文件的一部分。**静态链接的问题很明显，它会造成空间的浪费和软件更新的困难。举例来说，当有好几个程序调用了相同的系统函数时，每个程序都把这个函数加载到内存里，重复占用了很多空间，造成了资源的浪费。

动态链接解决了这一问题，其基本思想是**把程序按照模块拆分成各个相对独立部分，在程序运行时才将它们链接在一起形成一个完整的程序**，而不是像静态链接一样把所有程序模块都链接成一个单独的可执行文件。

那么问题就出现了，这个动态链接是和系统内置的库相关的。

#### glibc 和 musl libc

[The GNU C Library(glibc)](https://www.gnu.org/software/libc/) 是当代绝大多数 Linux 发行版内置的 C 标准库实现，而 [musl libc](https://musl.libc.org/) 则是一个速度快、轻量化的，严格遵循 POSIX 定义的 C 标准库实现。虽然后者试图保持与 glibc 的兼容性，但两者间的差距始终不小，在使用时难免会遇到兼容性问题。

Alpine 为了简单轻量，没有使用大多数 Linux 发行版使用的 glibc 而选择了 musl libc。笔者的程序是在 De­bian 上编译的，基于 glibc，而 Docker 容器选择了 Alpine 镜像，使用的是 musl libc，所以无法正常动态链接，也就无法运行。

#### 解决方案

参考网上给出的思路，方案大概有几种：

1. 不用 Alpine，选择和编译环境一致的发行版；或者干脆在 Alpine 下编译运行。

2. 给 Alpine 安装一个轻量的 glibc 兼容层 [libc6 compatibility package](https://pkgs.alpinelinux.org/package/edge/main/x86_64/libc6-compat): `apk add libc6-compat`。（对于简单的程序，兼容层足够解决问题，但是复杂的程序可能仍然无法正常运行）

3. 给 Alpine 安装真正的 glibc。(网上嫖来的 `dockerfile` 写法)

   ```dockerfile
   # Source: https://github.com/anapsix/docker-alpine-java
   
   ENV GLIBC_REPO=https://github.com/sgerrand/alpine-pkg-glibc
   ENV GLIBC_VERSION=2.30-r0
   
   RUN set -ex && \
       apk --update add libstdc++ curl ca-certificates && \
       for pkg in glibc-${GLIBC_VERSION} glibc-bin-${GLIBC_VERSION}; \
           do curl -sSL ${GLIBC_REPO}/releases/download/${GLIBC_VERSION}/${pkg}.apk -o /tmp/${pkg}.apk; done && \
       apk add --allow-untrusted /tmp/*.apk && \
       rm -v /tmp/*.apk && \
       /usr/glibc-compat/sbin/ldconfig /lib /usr/glibc-compat/lib
   ```

4. 静态链接程序，避免动态链接。

个人感觉还是 1/3 比较靠谱，如果是 2 测试之后没什么问题也很不错。

至于 4 ，刚刚 "动态链接" 部分也提到了，感觉不是很容易操作，故不推荐。

#### 参考文献

1. [linux - Docker Alpine executable binary not found even if in PATH - Stack Overflow](https://stackoverflow.com/questions/66963068/docker-alpine-executable-binary-not-found-even-if-in-path)
2. [深入浅出静态链接和动态链接_kang___xi的博客-CSDN博客_动态链接](https://blog.csdn.net/kang___xi/article/details/80210717)

另参见：[docker 容器上编译 go 程序无法运行，提示找不到文件 - 运维・速度 | 运维・速度 (sudops.com)](https://www.sudops.com/docker-)