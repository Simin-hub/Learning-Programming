# docker学习

[参考](https://yeasy.gitbook.io/docker_practice/)、[参考](https://pdai.tech/md/devops/docker/docker-01-docker-vm.html)

## 一、初识docker

### 1.1docker历史和发展

### 1.2 docker架构

> 理解如下的一些概念，你才知道安装什么

Docker 使用**客户端-服务器 (C/S) 架构模式**，使用远程API来管理和创建Docker容器。

- **Docker 客户端(Client)** : Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。
- **Docker 主机(Host)** ：一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。

Docker 包括三个基本概念:

- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- **仓库（Repository）**：仓库可看着一个代码控制中心，用来保存镜像。

![img](https://pdai.tech/_images/devops/docker/docker-x-1.png)

## Dockerfile

[参考](https://www.baeldung.com/ops/docker-dockerfile-docker-compose)、[参考](https://www.bookstack.cn/read/docker_practice-1.3.0/image-dockerfile-README.md)、[参考](https://www.bookstack.cn/read/docker_practice-1.3.0/image-multistage-builds-README.md)

Dockerfile 是一个纯文本文件，其中包含有关构建 Docker [image](https://www.baeldung.com/ops/docker-images-vs-containers)的说明。他们遵循一个 Dockerfile 标准，**Docker 守护程序最终负责执行 Dockerfile 并生成映像**。

典型的 [Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 通常从包含另一个image开始(从一个基础image开始)。

从那里，Dockerfile可以执行各种操作来构建映像：

- **将文件从主机系统复制到容器中**。例如，我们可能想要复制包含应用程序代码的 JAR 文件。
- **运行相对于图像的任意命令**。例如，我们可能希望运行典型的 Unix 命令来更改文件权限或使用包管理器安装新包。
- **定义创建容器时应执行的命令**。例如，一个 *java* 命令，它加载我们的 JAR 文件并启动所需的 main 方法。

### FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 `nginx` 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 `FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 `FROM scratch` 会让镜像体积更加小巧。使用 [Go 语言](https://golang.google.cn/) 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

### RUN 执行命令

`RUN` 指令是**用来执行命令行命令**的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：

- ***shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样**。刚才写的 Dockerfile 中的 `RUN` 指令就是这种格式。

```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- ***exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式**。

既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```
FROM debian:stretch
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

之前说过，**Dockerfile 中每一个指令都会建立一层**，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

*Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。*

上面的 `Dockerfile` 正确的写法应该是这样：

```
FROM debian:stretch
RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 `RUN` 一一对应不同的命令，而是仅仅使用一个 `RUN` 指令，并使用 `&&` 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

此外，还可以看到这一组命令的最后**添加了清理工作的命令，删除了为了编译构建所需要的软件**，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了**每一层构建的最后一定要清理掉无关文件**。

### 构建镜像

好了，让我们再回到之前定制的 nginx 镜像的 Dockerfile 来。现在我们明白了这个 Dockerfile 的内容，那么让我们来构建这个镜像吧。

在 `Dockerfile` 文件所在目录执行：

```
$ docker build -t nginx:v3 .Sending build context to Docker daemon 2.048 kBStep 1 : FROM nginx ---> e43d811ce2f4Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html ---> Running in 9cdc27646c7b ---> 44aa4490ce2cRemoving intermediate container 9cdc27646c7bSuccessfully built 44aa4490ce2c
```

从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在 `Step 2` 中，如同我们之前所说的那样，`RUN` 指令启动了一个容器 `9cdc27646c7b`，执行了所要求的命令，并最后提交了这一层 `44aa4490ce2c`，随后删除了所用到的这个容器 `9cdc27646c7b`。

这里我们使用了 `docker build` 命令进行镜像构建。其格式为：

```
docker build [选项] <上下文路径/URL/->
```

在这里我们指定了最终镜像的名称 `-t nginx:v3`，构建成功后，我们可以像之前运行 `nginx:v2` 那样来运行这个镜像，其结果会和 `nginx:v2` 一样。

### 镜像构建上下文（Context）

如果注意，会看到 `docker build` 命令最后有一个 `.`。**`.` 表示当前目录**，而 `Dockerfile` 就在当前目录，因此不少初学者以为这个路径是在指定 `Dockerfile` 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定 **上下文路径**。那么什么是上下文呢？

首先我们要理解 `docker build` 的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 **`docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能**。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

当我们进行镜像构建的时候，并非所有定制都会通过 `RUN` 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 `COPY` 指令、`ADD` 指令等。而 **`docker build` 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的**。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。**当构建的时候，用户会指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件**。（即`docker build .`  表示将当前目录下的所有文件和目录全部打包上传到 docker 引擎中，然后COPY等命令通过相对路径进行指定文件）

如果在 `Dockerfile` 中这么写：

```
COPY ./package.json /app/
```

这并不是要复制执行 `docker build` 命令所在的目录下的 `package.json`，也不是复制 `Dockerfile` 所在目录下的 `package.json`，而是复制 **上下文（context）** 目录下的 `package.json`。

因此，**`COPY` 这类指令中的源文件的路径都是*相对路径***。这也是初学者经常会问的为什么 `COPY ../package.json /app` 或者 `COPY /opt/xxxx /app` 无法工作的原因，**因为这些路径已经超出了上下文的范围**，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

现在就可以理解刚才的命令 `docker build -t nginx:v3 .` 中的这个 `.`，实际上是在指定上下文的目录，**`docker build` 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像**。

如果观察 `docker build` 输出，我们其实已经看到了这个发送上下文的过程：

```
$ docker build -t nginx:v3 .Sending build context to Docker daemon 2.048 kB...
```

理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现 `COPY /opt/xxxx /app` 不工作后，于是干脆将 `Dockerfile` 放到了硬盘根目录去构建，结果发现 `docker build` 执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让 `docker build` 打包整个硬盘，这显然是使用错误。

一般来说，**应该会将 `Dockerfile` 置于一个空目录下，或者项目根目录下**。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 `.gitignore` 一样的语法**写一个 `.dockerignore`，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的**。

那么为什么会有人误以为 `.` 是指定 `Dockerfile` 所在目录呢？这是因为在默认情况下，如果不额外指定 `Dockerfile` 的话，会将上下文目录下的名为 `Dockerfile` 的文件作为 Dockerfile。

这只是默认行为，实际上 **`Dockerfile` 的文件名并不要求必须为 `Dockerfile`，而且并不要求必须位于上下文目录中**，比如可以用 `-f ../Dockerfile.php` 参数指定某个文件作为 `Dockerfile`。

当然，一般大家习惯性的会使用默认的文件名 `Dockerfile`，以及会将其置于镜像构建上下文目录中。

### 其它 `docker build` 的用法

#### 直接用 Git repo 进行构建

或许你已经注意到了，`docker build` 还支持**从 URL 构建**，比如可以直接从 Git repo 中构建：

```
# $env:DOCKER_BUILDKIT=0
# export DOCKER_BUILDKIT=0
$ docker build -t hello-world https://github.com/docker-library/hello-world.git#master:amd64/hello-world
Step 1/3 : FROM scratch
 --->
Step 2/3 : COPY hello /
 ---> ac779757d46e
Step 3/3 : CMD ["/hello"]
 ---> Running in d2a513a760ed
Removing intermediate container d2a513a760ed
 ---> 038ad4142d2b
Successfully built 038ad4142d2b
```

这行命令指定了构建所需的 Git repo，并且指定分支为 `master`，构建目录为 `/amd64/hello-world/`，然后 Docker 就会自己去 `git clone` 这个项目、切换到指定分支、并进入到指定目录后开始构建。

#### 用给定的 tar 压缩包构建

```
$ docker build http://server/context.tar.gz
```

如果所给出的 URL 不是个 Git repo，而是个 `tar` 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

#### 从标准输入中读取 Dockerfile 进行构建

```
docker build - < Dockerfile
```

或

```
cat Dockerfile | docker build -
```

如果标准输入传入的是文本文件，则将其视为 `Dockerfile`，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 `COPY` 进镜像之类的事情。

#### 从标准输入中读取上下文压缩包进行构建

```
$ docker build - < context.tar.gz
```

如果发现标准输入的文件格式是 `gzip`、`bzip2` 以及 `xz` 的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建。



### Dockerfile 指令

#### COPY 复制文件

格式：

- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

和 `RUN` 指令一样，也有**两种格式，一种类似于命令行，一种类似于函数调用**。

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。比如：

```
COPY package.json /usr/src/app/
```

**`<源路径>` 可以是多个，甚至可以是通配符**，其通配符规则要满足 Go 的 [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) 规则，如：

```
COPY hom* /mydir/COPY hom?.txt /mydir/
```

**`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）**。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```
COPY --chown=55:mygroup files* /mydir/COPY --chown=bin files* /mydir/COPY --chown=1 files* /mydir/COPY --chown=10:11 files* /mydir/
```

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

#### ADD 更高级的复制文件

**`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能**。

ADD 支持文件复制、下载（如果下载的是压缩包，不会解压）、解压缩。

比如 `<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去。**下载后的文件权限自动设置为 `600`**，如果这并不是想要的权限，那么还需要增加额外的一层 `RUN` 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 `RUN` 指令进行解压缩。所以不如直接使用 `RUN` 指令，然后使用 `wget` 或者 `curl` 工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 `ubuntu` 中：

```
FROM scratchADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /...
```

但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 `ADD` 命令了。

在 Docker 官方的 [Dockerfile 最佳实践文档](https://www.bookstack.cn/read/docker_practice-1.3.0/appendix-best_practices.md) 中要求，**尽可能的使用 `COPY`，因为 `COPY` 的语义很明确，就是复制文件而已，而 `ADD` 则包含了更复杂的功能，其行为也不一定很清晰**。最适合使用 `ADD` 的场合，就是所提及的需要自动解压缩的场合。

另外需要注意的是，`ADD` 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，**所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`**。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

### CMD 容器启动命令

**`CMD` 指令的格式和 `RUN` 相似，也是两种格式**：

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。**`CMD` 指令就是用于指定默认的容器主进程的启动命令的**。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 `exec` 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 `"`，而不要使用单引号。

**如果使用 `shell` 格式的话，实际的命令会被包装为 `sh -c` 的参数的形式进行执行**。比如：

```
CMD echo $HOME
```

在实际执行中，会将其变更为：

```
CMD [ "sh", "-c", "echo $HOME" ]
```

这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

提到 `CMD` 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 `systemd` 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 `CMD` 写为：

```
CMD service nginx start
```

然后发现容器执行后就立即退出了。甚至在容器内去使用 `systemctl` 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

**对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出**，其它辅助进程不是它需要关心的东西。

而使用 `service nginx start` 命令，则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务。而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：

```
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT 入口点

**`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式**。

`ENTRYPOINT` 的目的和 `CMD` 一样，都是在**指定容器启动程序及参数**。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

**当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令**，换句话说实际执行时，将变为：

```
<ENTRYPOINT> "<CMD>"
```

那么有了 `CMD` 后，为什么还要有 `ENTRYPOINT` 呢？这种 `<ENTRYPOINT> "<CMD>"` 有什么好处么？让我们来看几个场景。

#### 场景一：让镜像变成像命令一样使用

假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 `CMD` 来实现：

```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```

假如我们使用 `docker build -t myip .` 来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：

```
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```

嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的 `CMD` 中可以看到实质的命令是 `curl`，那么如果我们希望显示 HTTP 头信息，就需要加上 `-i` 参数。那么我们可以直接加 `-i` 参数给 `docker run myip` 么？

```
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```

我们可以看到可执行文件找不到的报错，`executable file not found`。之前我们说过，跟在镜像名后面的是 `command`，运行时会替换 `CMD` 的默认值。因此这里的 `-i` 替换了原来的 `CMD`，而不是添加在原来的 `curl -s http://myip.ipip.net` 后面。而 `-i` 根本不是命令，所以自然找不到。

那么如果我们希望加入 `-i` 这参数，我们就必须重新完整的输入这个命令：

```
$ docker run myip curl -s http://myip.ipip.net -i
```

这显然不是很好的解决方案，而使用 `ENTRYPOINT` 就可以解决这个问题。现在我们重新用 `ENTRYPOINT` 来实现这个镜像：

```
FROM ubuntu:18.04RUN apt-get update \    && apt-get install -y curl \    && rm -rf /var/lib/apt/lists/*ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

这次我们再来尝试直接使用 `docker run myip -i`：

```
$ docker run myip当前 IP：61.148.226.66 来自：北京市 联通$ docker run myip -iHTTP/1.1 200 OKServer: nginx/1.8.0Date: Tue, 22 Nov 2016 05:12:40 GMTContent-Type: text/html; charset=UTF-8Vary: Accept-EncodingX-Powered-By: PHP/5.6.24-1~dotdeb+7.1X-Cache: MISS from cache-2X-Cache-Lookup: MISS from cache-2:80X-Cache: MISS from proxy-2_6Transfer-Encoding: chunkedVia: 1.1 cache-2:80, 1.1 proxy-2_6:8006Connection: keep-alive当前 IP：61.148.226.66 来自：北京市 联通
```

可以看到，这次成功了。这是因为当存在 `ENTRYPOINT` 后，`CMD` 的内容将会作为参数传给 `ENTRYPOINT`，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl`，从而达到了我们预期的效果。

#### 场景二：应用运行前的准备工作

启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

比如 `mysql` 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

此外，可能希望避免使用 `root` 用户去启动服务，从而提高安全性，而在启动服务前还需要以 `root` 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 `root` 身份执行，方便调试等。

这些准备工作是和容器 `CMD` 无关的，无论 `CMD` 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。比如官方镜像 `redis` 中就是这么做的：

```
FROM alpine:3.4...RUN addgroup -S redis && adduser -S -G redis redis...ENTRYPOINT ["docker-entrypoint.sh"]EXPOSE 6379CMD [ "redis-server" ]
```

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本。

```
#!/bin/sh...# allow the container to be started with `--user`if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then    find . \! -user redis -exec chown redis '{}' +    exec gosu redis "$0" "$@"fiexec "$@"
```

该脚本的内容就是根据 `CMD` 的内容来判断，如果是 `redis-server` 的话，则切换到 `redis` 用户身份启动服务器，否则依旧使用 `root` 身份执行。比如：

```
$ docker run -it redis iduid=0(root) gid=0(root) groups=0(root)
```

## docker compose

https://www.bookstack.cn/read/docker_practice-1.3.0/compose-README.md

https://www.cnblogs.com/liaokui/p/11380590.html

## docker命令

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/dockerView.png" alt="preview" style="zoom:200%;" />



## docker 网络与容器互联



### 容器公开的连接信息

Docker 通过 2 种方式为容器公开连接信息：

- 环境变量
- 更新 /etc/hosts 文件

#### 环境变量

使用 env 命令来查看 web 容器的环境变量

```bash
[root@pdai ~]# docker exec -it web /bin/bash
root@1cbc9aeba2a8:/opt/webapp# env
HOSTNAME=1cbc9aeba2a8
DB_NAME=/web/db
DB_PORT_5432_TCP_ADDR=172.17.0.2
DB_PORT=tcp://172.17.0.2:5432
DB_PORT_5432_TCP=tcp://172.17.0.2:5432
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/opt/webapp
DB_PORT_5432_TCP_PORT=5432
SHLVL=1
HOME=/root
DB_PORT_5432_TCP_PROTO=tcp
DB_ENV_PG_VERSION=9.3
_=/usr/bin/env
```

其中 DB_ 开头的环境变量是供 web 容器连接 db 容器使用，前缀采用大写的连接别名。

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/devops/docker/docker-03-basic-web-app.html

#### hosts 文件

除了环境变量，Docker 还添加 host 信息到父容器的 /etc/hosts 的文件。下面是父容器 web 的 hosts 文件

```bash
root@1cbc9aeba2a8:/opt/webapp# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      db d992e3c761e0
172.17.0.3      1cbc9aeba2a8
root@1cbc9aeba2a8:/opt/webapp#
```

这里有 2 个 hosts:

- 第一个, `172.17.0.2 db d992e3c761e0` 表示 db 容器的 ip, ID和Name
- 第二个，`172.17.0.3 1cbc9aeba2a8` 表示 web 容器的 ip, ID

可以在 web 容器中安装 ping 命令来测试跟db容器的连通。

```bash
root@1cbc9aeba2a8:/opt/webapp# apt-get install -yqq inetutils-ping
(Reading database ... 18233 files and directories currently installed.)
Removing ubuntu-minimal (1.325) ...
Removing iputils-ping (3:20121221-4ubuntu1.1) ...
Selecting previously unselected package inetutils-ping.
(Reading database ... 18221 files and directories currently installed.)
Preparing to unpack .../inetutils-ping_2%3a1.9.2-1_amd64.deb ...
Unpacking inetutils-ping (2:1.9.2-1) ...
Setting up inetutils-ping (2:1.9.2-1) ...
root@1cbc9aeba2a8:/opt/webapp# ping db
PING db (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.110 ms
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.094 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.104 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.111 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.093 ms
64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.095 ms
^C--- db ping statistics ---
7 packets transmitted, 7 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.092/0.100/0.111/0.000 ms
```

用 ping 来测试db容器，它会解析成 172.17.0.2。

当然，你还可以ping db容器的ID或者内部IP, 结果是一样的。

```bash
root@1cbc9aeba2a8:/opt/webapp# ping -t 4 d992e3c761e0
ping: unsupported packet type: 4
root@1cbc9aeba2a8:/opt/webapp# ping d992e3c761e0
PING db (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.089 ms
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.093 ms
^C--- db ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.089/0.091/0.093/0.000 ms
root@1cbc9aeba2a8:/opt/webapp# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.094 ms
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.103 ms
^C--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.094/0.099/0.103/0.000 ms
root@1cbc9aeba2a8:/opt/webapp#
```

用户可以链接多个父容器到子容器，比如可以链接多个 web 到 db 容器上。

### Docker网络使用和配置

上文已经向你介绍了，容器之间为什么可以直接通信？主机和容器之间为何可以通信？如何进行自定义的配置呢？所以这节就是我们要讲述的Docker网络。

#### 单机中docker网络

##### 理解Docker 默认网桥

> 为何上文的例子中，容器之间通过--link就可以相互通信，主机和容器之间也能通信？

**在你安装Docker 服务默认会创建一个 docker0 网桥**（其上有一个 docker0 内部接口），它在内核层连通了其他的物理或虚拟网卡，这就**将所有容器和本地主机都放到同一个物理网络**。

我们可用 docker network ls 命令查看：

```bash
[root@pdai ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b8c5abdb0bec        bridge              bridge              local
84e86bc93121        host                host                local
8e521527a897        none                null                local
```

Docker 安装时会自动在 host 上创建三个网络：none，host，和bridge。我们看下docker0 网桥：(brctl可以通过yum install bridge-utils安装)

```bash
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no              veth0004826
                                                        veth4ad3278
[root@pdai ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:08:c1:ea brd ff:ff:ff:ff:ff:ff
    inet 172.31.165.194/20 brd 172.31.175.255 scope global dynamic eth0
       valid_lft 310072401sec preferred_lft 310072401sec
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:70:3f:9d:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
45: veth4ad3278@if44: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether c2:77:e4:ea:f1:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
49: veth0004826@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 6e:c9:1a:7c:18:b1 brd ff:ff:ff:ff:ff:ff link-netnsid 1

[root@pdai ~]# ip route
default via 172.31.175.253 dev eth0
169.254.0.0/16 dev eth0 scope link metric 1002
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
172.31.160.0/20 dev eth0 proto kernel scope link src 172.31.165.194
```

再用docker network inspect指令查看bridge网络：其Gateway就是网卡/接口docker0的IP地址：172.17.0.1。

```bash
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1cbc9aeba2a8a826d460ecb49de17ddf8ac336e150c752a3c762fd38a3e15254": {
                "Name": "web",
                "EndpointID": "adb47ed0c60c6b80a442c71a5f35d63378cecca9598e0cef8409a6719552f4c2",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "d992e3c761e00649eb436b88c737adc54093b76119af0fb7878596b523f743ca": {
                "Name": "db",
                "EndpointID": "6a3dab5c545dd26e0ca6e36d928a32fd0a6197c8dbf4eeb718a4560e219575ed",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

从上面你可以看到bridge的配置信息和容器信息。

##### 理解容器创建时的IP分配

> 为了理解容器创建时的IP分配，这里需要清理所有已经启动的环境，然后再启动容器，看前后对比

- 我们**清理所有容器**实例，下面展示的就是docker安装之后的, 注意和上面的对比下：

```bash
[root@pdai ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@pdai ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b8c5abdb0bec        bridge              bridge              local
84e86bc93121        host                host                local
8e521527a897        none                null                local
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@pdai ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:08:c1:ea brd ff:ff:ff:ff:ff:ff
    inet 172.31.165.194/20 brd 172.31.175.255 scope global dynamic eth0
       valid_lft 310069965sec preferred_lft 310069965sec
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:70:3f:9d:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no
```

- **创建容器**

Docker 在**创建一个容器的时候**，会执行如下操作：

- **创建一对虚拟接口/网卡，也就是veth pair，分别放到本地主机和新容器中**；
- 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 vethxxxxx；
- 容器一端放到新容器中，并修改名字作为 eth0，这个网卡/接口只在容器的名字空间可见；
- 从网桥可用地址段中（也就是与该bridge对应的network）获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 vethxxxx。

让我们启动一个容器，看下变化：

```bash
[root@pdai ~]# docker run -d --name db training/postgres
0ffd1092cd962bdbe335ce042b93d0f2082559600cacc82bbef40b8b66395e57
[root@pdai ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242703f9d02       no              vethd93e2ad
[root@pdai ~]# ip a | grep vethd93e2ad
51: vethd93e2ad@if50: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
[root@pdai ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "b8c5abdb0becacfa1bfa1d72e2e663fb0157b62a9b8bee37e2607211722713cc",
        "Created": "2020-02-17T14:10:10.424119543+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0ffd1092cd962bdbe335ce042b93d0f2082559600cacc82bbef40b8b66395e57": {
                "Name": "db",
                "EndpointID": "a90cb50031effc99b9254fe4f1231bfbac8c4bb23d94c5a1425c1e116ac452dc",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

**如果不指定--network，创建的容器默认都会挂到 docker0 上**，使用本地主机上 docker0 接口的 IP 作为所有容器的默认网关

当有多个容器创建后，容器网络拓扑结构如下：

![img](https://pdai.tech/_images/devops/docker/docker-y-4.jpg)

##### 理解容器和docker0的虚拟网卡的配对

> 在上图中容器中eth0是怎么和host中虚拟网卡配对上的呢？

```bash
[root@pdai ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
0ffd1092cd96        training/postgres   "su postgres -c '/us…"   11 minutes ago      Up 11 minutes       5432/tcp            db
[root@pdai ~]# docker exec -it 0ffd1092cd96 /bin/bash
root@0ffd1092cd96:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

我们可以看到host上`51: vethd93e2ad@if50`对应着容器中`50: eth0@if51`; 即host中index=51的接口/网卡vethd93e2ad的peer inferface index是50, container中index=50的网卡eth0的peer interface index是51。

可以利用ethtool来确认这种对应关系：分别在host和container中运行指令`ethtool -S <interface>`:

```bash
[root@pdai ~]# ethtool -S vethd93e2ad
NIC statistics:
     peer_ifindex: 50
[root@pdai ~]# docker exec -it 0ffd1092cd96 /bin/bash
root@0ffd1092cd96:/# ip a | grep 50
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
root@0ffd1092cd96:/#
```

#### 多物理机之间互联

> 如果在企业内部应用，或者做多个物理主机的集群，可能需要将多个物理主机的容器组到一个物理网络中来，那么就需要将这个网桥桥接到我们指定的网卡上。

##### 拓扑图

主机 A 和主机 B 的网卡一都连着物理交换机的同一个 vlan 101,这样网桥一和网桥三就相当于在同一个物理网络中了，而容器一、容器三、容器四也在同一物理网络中了，他们之间可以相互通信，而且可以跟同一 vlan 中的其他物理机器互联。

![img](https://pdai.tech/_images/devops/docker/docker-y-6.png)

##### ubuntu 示例

下面以 ubuntu 为例创建多个主机的容器联网: 创建自己的网桥,编辑 /etc/network/interface 文件

```bash
auto br0
iface br0 inet static
address 192.168.7.31
netmask 255.255.240.0
gateway 192.168.7.254
bridge_ports em1
bridge_stp off
dns-nameservers 8.8.8.8 192.168.6.1
```

将 Docker 的默认网桥绑定到这个新建的 br0 上面，这样就将这台机器上容器绑定到 em1 这个网卡所对应的物理网络上了。

ubuntu 修改 /etc/default/docker 文件，添加最后一行内容

```bash
# Docker Upstart and SysVinit configuration file
# Customize location of Docker binary (especially for development testing).
#DOCKER="/usr/local/bin/docker"
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"

# If you need Docker to use an HTTP proxy, it can also be specified here.
#export http_proxy="http://127.0.0.1:3128/"

# This is also a handy place to tweak where Docker's temporary files go.
#export TMPDIR="/mnt/bigdrive/docker-tmp"

DOCKER_OPTS="-b=br0"
```

在启动 Docker 的时候 使用 -b 参数 将容器绑定到物理网络上。重启 Docker 服务后，再进入容器可以看到它已经绑定到你的物理网络上了。

```bash
root@pdai:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                        NAMES
58b043aa05eb        desk_hz:v1          "/startup.sh"       5 days ago          Up 2 seconds        5900/tcp, 6080/tcp, 22/tcp   yanlx
root@pdai:~# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.7e6e617c8d53       no              em1
```

这样就直接把容器暴露到物理网络上了，多台物理主机的容器也可以相互联网了。需要注意的是，这样就需要自己来保证容器的网络安全了。

具体步骤，还可以看下这篇文章：[CentOS Docker跨宿主机通讯Open vSwitch](https://blog.csdn.net/gavinkelland/article/details/77976815)

### 具体实现

[参考](https://latelee.blog.csdn.net/article/details/81396532)

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/devops/docker/docker-06-data.html

## Docker 数据管理简介

### 数据卷(Data Volume)

> 数据卷的使用，类似于 Linux 下对目录或文件进行 mount

数据卷(Data Volume)是一个**可供一个或多个容器使用的特殊目录**，它绕过 UFS，可以提供很多有用的特性：

- **数据卷可以在容器之间共享和重用**
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 卷会一直存在，直到没有容器使用

#### 建一个数据卷

> 在用 docker run 命令的时候，使用 -v 标记来创建一个数据卷并挂载到容器里。在一次 run 中多次使用可以挂载多个数据卷。

下面创建一个 web 容器，并加载一个数据卷到容器的 /webapp-data 目录。

```bash
[root@pdai ~]# docker run -d -P --name web -v /webapp-data training/webapp python app.py
e331e83e59486a131919cba8698b24eaee051a947838bb1c15c03df8b3464b97
```

我们看下容器内部是否生成/webapp-data目录

```bash
[root@pdai ~]# docker exec -it web /bin/bash
root@e331e83e5948:/opt/webapp# cd /webapp-data
root@e331e83e5948:/webapp-data# ll
total 8
drwxr-xr-x 2 root root 4096 Feb 20 01:24 ./
drwxr-xr-x 1 root root 4096 Feb 20 01:24 ../
root@e331e83e5948:/webapp-data#
```

*注意：也可以在 Dockerfile 中使用 VOLUME 来添加一个或者多个新的卷到由该镜像创建的任意容器。

#### 挂载一个主机目录作为数据卷

> 使用 -v 标记也可以指定挂载一个本地主机的目录到容器中去。

```bash
[root@pdai ~]# docker rm -f web
web
[root@pdai opt]# docker run -d --name web -v /opt/webapp-data5:/opt/webapp2 training/webapp
fce27f6ea9ce9699864644a48aed6db8b772c96be36f46bee6154d2e2c9915b9
```

我们验证下：

```bash
[root@pdai opt]# docker exec -it web /bin/bash
root@fce27f6ea9ce:/opt/webapp# cd ..
root@fce27f6ea9ce:/opt# ls
webapp  webapp2
root@fce27f6ea9ce:/opt# cd webapp2
root@fce27f6ea9ce:/opt/webapp2# mkdir test
root@fce27f6ea9ce:/opt/webapp2# exit
exit
[root@pdai opt]# cd webapp-data5
[root@pdai webapp-data5]# ll
total 4
drwxr-xr-x 2 root root 4096 Feb 20 10:12 test
```

上面的命令加载主机的 /opt/webapp-data5 目录到容器的 /opt/webapp2 目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。**本地目录的路径必须是绝对路径，如果本地目录不存在 Docker 会自动为你创建它**。

*注意：**Dockerfile 显然是不支持这种用法**，这是因为 Dockerfile 是为了移植和分享用的, 因为不同操作系统的路径格式不一样，所以目前还不能支持。*

我们删除容器，看主机上数据是否会被删除

```bash
[root@pdai opt]# docker rm -f web
web
[root@pdai opt]# cd /opt/webapp-data5
[root@pdai webapp-data5]# ll
total 4
drwxr-xr-x 2 root root 4096 Feb 20 10:12 test
```

很明显，没有被删除

#### 挂载一个本地主机文件作为数据卷

> -v 标记也可以从主机挂载单个文件到容器中

```bash
[root@pdai ~]# docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
root@79eca07938db:/# ll | grep .bash_history
-rw-------   1 root root 19549 Feb 19 10:28 .bash_history
root@79eca07938db:/# exit
exit
```

这样就可以记录在容器输入过的命令了。

*注意：如果直接挂载一个文件，很多文件编辑工具，包括 `vi` 或者 `sed --in-place`，可能会造成文件 inode 的改变，从 Docker 1.1 .0起，这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。*

### 数据卷容器(Data Volume Container)

> 上面讲述的是**主机和容器之间共享数据**，那么如何你有一些持续更新的数据需要在**容器之间共享**，最好的方法就是**创建数据卷容器**。

**数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的**。

```bash
[root@pdai ~]# docker run -d -v /dbdata --name dbdata training/postgres
70966085a85b05dd741a44a96725e2e44f146cc404b1b4e3aa3e519cd546c6b4
[root@pdai ~]# docker run -d --volumes-from dbdata --name db1 training/postgres
4c92240096d919724b233e1a5cfca94b5ceb0505e43262a7121cb83cfd8542f6
[root@pdai ~]# docker run -d --volumes-from dbdata --name db2 training/postgres
25246ebfae2f8437316b10d7eac3b34c1bd1522f50ba81651aec198bc79415a2
[root@pdai ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
70966085a85b        training/postgres   "su postgres -c '/us…"   46 seconds ago       Up 45 seconds       5432/tcp            dbdata
25246ebfae2f        training/postgres   "su postgres -c '/us…"   About a minute ago   Up About a minute   5432/tcp            db2
4c92240096d9        training/postgres   "su postgres -c '/us…"   2 minutes ago        Up 2 minutes        5432/tcp            db1
```

-volumes-from 可以多次使用来 mount 多个conatainer里的多个volumes。

这个操作是链式的， 我们在db1 中通过 --volumes-from mount进来的 volume可以继续被其他container使用

```bash
[root@pdai ~]# docker run -d --name db3 --volumes-from db1 training/postgres
44d0719377e86e3080b26d22adcb6055de93033dc9509ca2ecd8be2c93dc33b5
[root@pdai ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44d0719377e8        training/postgres   "su postgres -c '/us…"   3 seconds ago       Up 2 seconds        5432/tcp            db3
70966085a85b        training/postgres   "su postgres -c '/us…"   3 minutes ago       Up 3 minutes        5432/tcp            dbdata
25246ebfae2f        training/postgres   "su postgres -c '/us…"   4 minutes ago       Up 4 minutes        5432/tcp            db2
4c92240096d9        training/postgres   "su postgres -c '/us…"   4 minutes ago       Up 4 minutes        5432/tcp            db1
```

**使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态**。

**如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 `docker rm -v` 命令来指定同时删除关联的容器。 这可以让用户在容器之间升级和移动数据卷**。

### 数据备份、恢复、迁移数据卷

可以利用数据卷对其中的数据进行进行备份、恢复和迁移。

#### 备份

首先使用 --volumes-from 标记来创建一个加载 dbdata 容器卷的容器，并从本地主机挂载当前到容器的 /backup 目录。命令如下：

```bash
[root@pdai ~]# docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
tar: Removing leading `/' from member names
/dbdata/
[root@pdai ~]# ll | grep backup.tar
-rw-r--r-- 1 root root    10240 Feb 20 12:39 backup.tar
[root@pdai ~]#
```

容器启动后，使用了 tar 命令来将 dbdata 卷备份为本地的 /backup/backup.tar。

#### 恢复

如果要恢复数据到一个容器

首先创建一个带有数据卷的容器 dbdata2

```bash
[root@pdai ~]# docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

然后创建另一个容器，挂载 dbdata2 的容器，并使用 untar 解压备份文件到挂载的容器卷中。

```bash
[root@pdai ~]# docker run --volumes-from dbdata2 -v $(pwd):/backup ubuntu tar xvf /backup/backup.tar
dbdata/
```

### 注意

#### volume 挂载时文件或文件夹不存在

[参考](https://segmentfault.com/a/1190000015684472)

##### 文件夹挂载

docker在文件夹挂载上的行为是统一的，具体表现为：

- 若文件夹不存在，则先创建出文件夹（若为多层文件夹，则递归创建）
- 用host上的文件夹内容覆盖container中的文件夹内容

```bash
docker run -v /path-to-folder/A:/path-to-folder/B test-image
```

详细说明如下：

###### host上文件夹存在，且非空

| host              | container         | mount result                                                 |
| ----------------- | ----------------- | ------------------------------------------------------------ |
| 存在的非空文件夹A | 不存在的文件夹B   | 先在contanier中创建文件夹B，再将A文件夹中的所有文件copy到B中 |
| 存在的非空文件夹A | 存在的非空文件夹B | 先将container中文件夹B的原有内容清空，再将A中文件copy到B中   |

> 无论container中的文件夹B是否存在， A都会完全覆盖B的内容

###### host上文件夹存在，但为空

| host            | container         | mount result                   |
| --------------- | ----------------- | ------------------------------ |
| 存在的空文件夹A | 存在的非空文件夹B | container中文件夹B的内容被清空 |

> container中对应的文件夹内容被清空

###### host上文件夹不存在

| host                | container         | mount result                                            |
| ------------------- | ----------------- | ------------------------------------------------------- |
| 不存在的文件夹A     | 存在的非空文件夹B | 在host上创建文件夹A，container中文件夹B的内容被清空     |
| 不存在的文件夹A/B/C | 存在的非空文件夹B | 在host上创建文件夹A/B/C，container中文件夹B的内容被清空 |

> container中对应的文件夹内容被清空

###### 总结

**host上文件夹一定会覆盖container中文件夹**：

| host                          | container                            | mount result                                                 |
| ----------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| 文件夹不存在/文件夹存在但为空 | 文件夹不存在/存在但为空/存在且不为空 | container中文件被覆盖（清空）                                |
| 文件夹存在且不为空            | 文件夹不存在/存在但为空/存在且不为空 | container中文件夹内容被覆盖（原内容清空， 覆盖为host上文件夹内容） |

##### 文件挂载

文件挂载与文件夹挂载最大的不同点在于：

- **docker 禁止用主机上不存在的文件挂载到container中已经存在的文件**
- 文件挂载不会对同一文件夹下的其他文件产生任何影响

除此之外， 其覆盖行为与文件夹挂载一致，即：

- 用host上的文件的内容覆盖container中的文件的内容

```bash
docker run -v /path-to-folder/non-existent-config.js:/path-to-folder/config.js test-image # forbidden
```

详细说明如下：

###### host上文件不存在

| host                   | container                 | mount result                                                 |
| ---------------------- | ------------------------- | ------------------------------------------------------------ |
| 不存在的文件configA.js | 已经存在的文件congfigB.js | 报错，Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type. 同时会在host上生成两个空目录 configA.js 和 configB.js, 但是container无法启动 |

###### host上文件存在

| host                 | container               | mount result                                                 |
| -------------------- | ----------------------- | ------------------------------------------------------------ |
| 存在的文件configA.js | 存在的文件congfigB.js   | container中文件名configB.js保持不变,但是文件内容被congfigA.js的内容覆盖了 |
| 存在的文件configA.js | 不存在的文件congfigB.js | container中新建一个文件configB.js，其内容为configA.js的文件内容， configB.js所在文件下的所有其他文件维持不变 |

###### 总结

**host上文件（一定存在，否则报错）一定会覆盖container中文件夹**

| host         | container                   | mount result                           |
| ------------ | --------------------------- | -------------------------------------- |
| 不存在的文件 | 已经存在的文件              | 禁止行为                               |
| 存在的文件   | 不存在的文件/已经存在的文件 | 新增/覆盖 （若目录不存在则会创建目录） |

##### 结论

###### 文件夹挂载

- 允许不存在的文件夹或者存在的空文件夹挂载进container, container中对应的文件夹将被清空
- 非空文件夹挂载进container将会覆盖container中原有文件夹

###### 文件挂载

- 禁止将不存在的文件挂载进container中已经存在的文件上
- 存在的文件挂载进container中将会覆盖container中对应的文件， 若文件不存在则新建

###### 应用场景

1. 从上面的分析可知，文件夹挂载以整个文件夹为单位进行文件覆盖，故可在需要将大量文件挂载进container时使用，另外，如果挂载一个空文件夹或者不存在的文件夹，一般是做逆向使用： 即容器启动后，可能会在容器内挂载点的文件夹下生成一些文件（如日志），此时，在对应的host上的文件夹内就能直接看到。
2. 文件挂载由于只会覆盖单个文件而不会影响container中同一文件夹下的其他文件，常常被用来挂载配置文件，以在运行时，动态的修改默认配置。

## 面试

### 1. 什么是 Docker 容器？

Docker 是一种流行的开源软件平台，可简化创建、管理、运行和分发应用程序的过程。它使用容器来打包应用程序及其依赖项。我们也可以将容器视为 Docker 镜像的运行时实例。

### 2. Docker 和虚拟机有什么不同？

Docker 是轻量级的沙盒，在其中运行的只是应用，虚拟机里面还有额外的系统。

### 3. 什么是 DockerFile？

Dockerfile 是一个文本文件，其中包含我们需要运行以构建 Docker 镜像的所有命令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。Docker 使用 Dockerfile 中的指令自动构建镜像。我们可以 `docker build` 用来创建按顺序执行多个命令行指令的自动构建。

**一些最常用的指令如下：**

```
FROM ：使用 FROM 为后续的指令建立基础映像。在所有有效的 Dockerfile 中， FROM 是第一条指令。

LABEL： LABEL 指令用于组织项目映像，模块，许可等。在自动化布署方面 LABEL 也有很大用途。在 LABEL 中指定一组键值对，可用于程序化配置或布署 Docker 。

RUN： RUN 指令可在映像当前层执行任何命令并创建一个新层，用于在映像层中添加功能层，也许最来的层会依赖它。

CMD： 使用 CMD 指令为执行的容器提供默认值。在 Dockerfile 文件中，若添加多个 CMD 指令，只有最后的 CMD 指令运行。
```

### 4. 使用Docker Compose时如何保证容器A先于容器B运行？

> Docker Compose 是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要使用shell脚本来启动容器。Compose 通过一个配置文件来管理多个Docker容器。简单理解：**Docker Compose 是docker的管理工具**。

Docker Compose 在继续下一个容器之前不会等待容器准备就绪。为了控制我们的执行顺序，我们可以使用“**取决于**”条件，`depends_on` 。这是在 docker-compose.yml 文件中使用的示例

```
version: "2.4"

services:

 backend:

   build: .    # 构建自定义镜像

   depends_on:

     - db

 db:

   image: mysql
```

用 `docker-compose up` 命令将按照我们指定的依赖顺序启动和运行服务。

### 5. 一个完整的Docker由哪些部分组成?

- DockerClient 客户端
- Docker Daemon 守护进程
- Docker Image 镜像
- DockerContainer 容器

### 6. docker常用命令

> 命令建议在本地安装做一个实操，记忆会更深刻。
>
> 也可以克隆基于docker的俩万（springboot+vue）项目练手，提供视频+完善文档。地址：https://gitee.com/rodert/liawan-vue

1. 查看本地主机的所用镜像：`docker images``
2. 搜索镜像：`docker search mysql``
3. 下载镜像：`docker pull mysql`，没写 tag 就默认下载最新的 lastest
4. 下载指定版本的镜像：`docker pull mysql:5.7``
5. 删除镜像：`docker rmi -f 镜像id 镜像id 镜像id``

### 7. 描述 Docker 容器的生命周期。

Docker 容器经历以下阶段：

- 创建容器
- 运行容器
- 暂停容器（可选）
- 取消暂停容器（可选）
- 启动容器
- 停止容器
- 重启容器
- 杀死容器
- 销毁容器

### 8. docker容器之间怎么隔离?

> 这是一道涉猎很广泛的题目，理解性阅读。

Linux中的PID、IPC、网络等资源是全局的，而Linux的NameSpace机制是一种资源隔离方案，在该机制下这些资源就不再是全局的了，而是属于某个特定的NameSpace，各个NameSpace下的资源互不干扰。

**Namespace实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容。**对于宿主机来说，这些被“隔离”了的进程跟其他进程并没有区别。

虽然有了NameSpace技术可以实现资源隔离，但进程还是可以不受控的访问系统资源，比如CPU、内存、磁盘、网络等，为了控制容器中进程对资源的访问，**Docker采用control groups技术(也就是cgroup)，有了cgroup就可以控制容器中进程对系统资源的消耗了**，比如你可以限制某个容器使用内存的上限、可以在哪些CPU上运行等等。

有了这两项技术，容器看起来就真的像是独立的操作系统了。

### Docker与虚拟机的不同点在哪里？

[参考](https://pdai.tech/md/devops/docker/docker-01-docker-vm.html)

答：Docker不是虚拟化方法。它依赖于实际实现**基于容器的虚拟化或操作系统级虚拟化的其他工具**。为此，Docker最初使用LXC驱动程序，然后移动到libcontainer现在重命名为runc。Docker主要专注于在应用程序容器内自动部署应用程序。应用程序容器旨在打包和运行单个服务，而系统容器则设计为运行多个进程，如虚拟机。因此，Docker被视为容器化系统上的容器管理或应用程序部署工具。

![img](https://pdai.tech/_images/devops/docker/docker-y-0.jpg)



- 虚拟机
  - 基础设施（Infrastructure）。它可以是你的个人电脑，数据中心的服务器，或者是云主机。
  - 主操作系统（Host Operating System）。你的个人电脑之上，运行的可能是MacOS，Windows或者某个Linux发行版。
  - 虚拟机管理系统（Hypervisor）。利用Hypervisor，可以在主操作系统之上运行多个不同的从操作系统。类型1的Hypervisor有支持MacOS的HyperKit，支持Windows的Hyper-V以及支持Linux的KVM。类型2的Hypervisor有VirtualBox和VMWare。
  - 操作系统（Guest Operating System）。假设你需要运行3个相互隔离的应用，则需要使用Hypervisor启动3个从操作系统，也就是3个虚拟机。这些虚拟机都非常大，也许有700MB，这就意味着它们将占用2.1GB的磁盘空间。更糟糕的是，它们还会消耗很多CPU和内存。
  - 各种依赖。每一个从操作系统都需要安装许多依赖。如果你的的应用需要连接PostgreSQL的话，则需要安装libpq-dev；如果你使用Ruby的话，应该需要安装gems；如果使用其他编程语言，比如Python或者Node.js，都会需要安装对应的依赖库。
- Docker容器
  - 主操作系统（Host Operating System）。所有主流的Linux发行版都可以运行Docker。对于MacOS和Windows，也有一些办法"运行"Docker。
  - Docker守护进程（Docker Daemon）。Docker守护进程取代了Hypervisor，它是**运行在操作系统之上的后台进程，负责管理Docker容器**。
  - 各种依赖。对于Docker，应用的所有依赖都打包在Docker镜像中，Docker容器是基于Docker镜像创建的。
  - 应用。应用的源代码与它的依赖都打包在Docker镜像中，不同的应用需要不同的Docker镜像。不同的应用运行在不同的Docker容器中，它们是相互隔离的。

**虚拟机是在物理资源层面实现的隔离**，相对于虚拟机，**Docker是从APP层面实现的隔离**，并且省去了虚拟机操作系统（Guest OS）），从而节省了一部分的系统资源；**Docker守护进程可以直接与主操作系统进行通信，为各个Docker容器分配资源**；它还可以将容器与主操作系统隔离，并将各个容器互相隔离。虚拟机启动需要数分钟，而Docker容器可以在数毫秒内启动。由于没有臃肿的从操作系统，Docker可以节省大量的磁盘空间以及其他系统资源。

虚拟机与容器docker的区别，在于**vm多了一层guest OS，虚拟机的Hypervisor会对硬件资源也进行虚拟化，而容器Docker会直接使用宿主机的硬件资源**。

下面我们采用形象的比喻区分两者的**隔离级别**：

- **服务器**：比作一个大型的仓管基地，包含场地与零散的货物——相当于各种服务器资源。
- **虚拟机技术**：比作仓库，拥有独立的空间堆放各种货物或集装箱，仓库之间完全独立——仓库相当于各种系统，独立的应用系统和操作系统。
- **Docker**：比作集装箱，操作各种货物的打包——将各种应用程序和他们所依赖的运行环境打包成标准的容器，容器之间隔离。

## docker 遇到的问题

### [彻夜怒肝！Docker 常见疑难杂症解决方案已撸完，快要裂开了。。。](https://segmentfault.com/a/1190000039426040)

#### 1.Docker 迁移存储目录

#### 2.[Docker](https://link.segmentfault.com/?enc=bZfweF5ivDGGLCp6LyBK5g%3D%3D.v5Bo2YCFwc48pPgbs4DFPR0s8RD2AIcJrN0lbItcLKiuYe6z%2BRZUZiwD6KoGoenl7SsKwaIFfwzlRiuOLu%2Br0Z1oqohIwcedP3GvIL6Y7FCjQGfNXei5AuxtjFHE0GFXUuSWD2neHqA6Mu60TzhVvavswLl9HLdpbYrjjOUnPTZ8ZmpG0GjPWyTkjoRGcaZDHKfKmGNaK0SiAloXMKOQUUFvOXLY6noJh%2BRcK0G5szGKwaJPJ0ePrj8oRPO5ITQIgfa0UgG%2FwLUz4bljnX4UjZvNV39%2Br%2BjQnDwGpFUI%2BeC0QDhmDkvYXS%2BLEIAG4fQu) 设备空间不足

#### 3.Docker 缺共享链接库

#### 4.Docker 容器文件损坏

#### 5.Docker 容器优雅重启

#### 6.Docker 容器无法删除

#### 7.Docker 容器中文异常

#### 8.Docker 容器网络互通

#### 9.Docker 容器总线错误

#### 10.Docker NFS 挂载报错

#### 11.Docker 默认使用网段

#### 12.Docker 服务启动串台

#### 13.Docker 命令调用报错

#### 14.Docker 定时任务异常

#### 15.Docker 变量使用引号

#### 16.Docker 删除镜像报错

#### 17.Docker 普通用户切换

