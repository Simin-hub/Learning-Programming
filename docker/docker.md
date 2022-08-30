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

> apt-get update 和 apt-get install 被放在一个 RUN 指令中执行，这样能够保证每次安装的是最新的包。如果 apt-get install 在单独的 RUN 中执行，则会使用 apt-get update 创建的镜像层，而这一层可能是很久以前缓存的。

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

#### CMD 容器启动命令

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

#### ENTRYPOINT 入口点

**`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式**。

`ENTRYPOINT` 的目的和 `CMD` 一样，都是在**指定容器启动程序及参数**。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

**当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令**，换句话说实际执行时，将变为：

```
<ENTRYPOINT> "<CMD>"
```

那么有了 `CMD` 后，为什么还要有 `ENTRYPOINT` 呢？这种 `<ENTRYPOINT> "<CMD>"` 有什么好处么？让我们来看几个场景。

##### 场景一：让镜像变成像命令一样使用

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
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

这次我们再来尝试直接使用 `docker run myip -i`：

```
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
$ docker run myip -i
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Tue, 22 Nov 2016 05:12:40 GMT
Content-Type: text/html; charset=UTF-8
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
X-Cache: MISS from cache-2
X-Cache-Lookup: MISS from cache-2:80
X-Cache: MISS from proxy-2_6
Transfer-Encoding: chunked
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
Connection: keep-alive
当前 IP：61.148.226.66 来自：北京市 联通
```

可以看到，这次成功了。这是因为**当存在 `ENTRYPOINT` 后，`CMD` 的内容将会作为参数传给 `ENTRYPOINT`**，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl`，从而达到了我们预期的效果。

##### 场景二：应用运行前的准备工作

**启动容器就是启动主进程**，但有些时候，启动主进程前，需要一些准备工作。

比如 `mysql` 类的数据库，**可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决**。

此外，可能希望避免使用 `root` 用户去启动服务，从而提高安全性，而在启动服务前还需要以 `root` 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 `root` 身份执行，方便调试等。

这些准备工作是和容器 `CMD` 无关的，无论 `CMD` 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。比如官方镜像 `redis` 中就是这么做的：

```
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 6379
CMD [ "redis-server" ]
```

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本。

```
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    find . \! -user redis -exec chown redis '{}' +
    exec gosu redis "$0" "$@"
fi
exec "$@"
```

该脚本的内容就是根据 `CMD` 的内容来判断，如果是 `redis-server` 的话，则切换到 `redis` 用户身份启动服务器，否则依旧使用 `root` 身份执行。比如：

```
$ docker run -it redis iduid=0(root) gid=0(root) groups=0(root)
```

##### Dockerfile RUN，CMD，ENTRYPOINT命令区别

Dockerfile中RUN，CMD和ENTRYPOINT**都能够用于执行命令**，下面是三者的主要用途：

- RUN命令**执行命令并创建新的镜像层，通常用于安装软件包**
- CMD命令**设置容器启动后默认执行的命令及其参数**，但CMD设置的命令能够被`docker run`命令后面的命令行参数替换
- ENTRYPOINT**配置容器启动时的执行命令**（不会被忽略，一定会被执行，即使运行 `docker run`时指定了其他命令）

- 使用 RUN 指令安装应用和软件包，构建镜像。
- 如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数。
- 如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令。

#### ENV 设置环境变量

格式有两种：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

**这个指令很简单，就是设置环境变量而已，无论是后面的其它指令**，如 `RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。

```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 `node` 镜像 `Dockerfile` 中，就有类似这样的代码：

```
ENV NODE_VERSION 7.2.0
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

在这里先定义了环境变量 `NODE_VERSION`，其后的 `RUN` 这层里，多次使用 `$NODE_VERSION` 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新 `7.2.0` 即可，`Dockerfile` 构建维护变得更轻松了。

下列指令可以支持环境变量展开： `ADD`、`COPY`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`、`RUN`。

可以从这个指令列表里感觉到，**环境变量可以使用的地方很多**，很强大。通过环境变量，我们可以让一份 `Dockerfile` 制作更多的镜像，只需使用不同的环境变量即可。

#### ARG 构建参数

格式：`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，**`ARG` 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的**。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。

灵活的使用 `ARG` 指令，能够在不修改 Dockerfile 的情况下，构建出不同的镜像。

ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
RUN set -x ; echo ${DOCKER_USERNAME}
```

使用上述 Dockerfile 会发现无法输出 `${DOCKER_USERNAME}` 变量的值，要想正常输出，你必须在 `FROM` 之后再次指定 `ARG`

```
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library
RUN set -x ; echo ${DOCKER_USERNAME}
```

对于多阶段构建，尤其要注意这个问题

```
# 这个变量在每个 FROM 中都生效
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
RUN set -x ; echo 1
FROM ${DOCKER_USERNAME}/alpine
RUN set -x ; echo 2
```

对于上述 Dockerfile 两个 `FROM` 指令都可以使用 `${DOCKER_USERNAME}`，对于在各个阶段中使用的变量都必须在每个阶段分别指定：

```
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library
RUN set -x ; echo ${DOCKER_USERNAME}
FROM ${DOCKER_USERNAME}/alpine
# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library
RUN set -x ; echo ${DOCKER_USERNAME}
```

#### VOLUME 定义匿名卷

格式为：

- `VOLUME ["<路径1>", "<路径2>"...]`
- `VOLUME <路径>`

之前我们说过，**容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中**。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，**不会向容器存储层写入大量数据**。

```
VOLUME /data
```

这里的 `/data` 目录就会在容器运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂载设置。比如：

```
$ docker run -d -v mydata:/data xxxx
```

在这行命令中，就使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 `Dockerfile` 中定义的匿名卷的挂载配置。

#### EXPOSE 声明端口

格式为 `EXPOSE <端口1> [<端口2>...]`。

**`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务**。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。**`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射**。

#### WORKDIR 指定工作目录

格式为 `WORKDIR <工作目录路径>`。

**使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录**，如该目录不存在，`WORKDIR` 会帮你建立目录。

之前提到一些初学者常犯的错误是把 `Dockerfile` 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

```
RUN cd /appRUN echo "hello" > world.txt
```

如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而**在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器**。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。

之前说过**每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更**。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此**如果需要改变以后各层的工作目录的位置**，那么应该使用 `WORKDIR` 指令。

```
WORKDIR /appRUN echo "hello" > world.txt
```

如果你的 `WORKDIR` 指令使用的相对路径，那么所切换的路径与之前的 `WORKDIR` 有关：

```
WORKDIR /aWORKDIR bWORKDIR cRUN pwd
```

`RUN pwd` 的工作目录为 `/a/b/c`。

#### USER 指定当前用户

格式：`USER <用户名>[:<用户组>]`

`USER` 指令和 `WORKDIR` 相似，都是**改变环境状态并影响以后的层**。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。

注意，`USER` 只是帮助你**切换到指定用户而已，这个用户必须是事先建立好的**，否则无法切换。

```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su` 或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 [`gosu`](https://github.com/tianon/gosu)。

```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.12/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

#### HEALTHCHECK 健康检查

格式：

- `HEALTHCHECK [选项] CMD <命令>`：**设置检查容器健康状况的命令**
- `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK` 指令是**告诉 Docker 应该如何进行判断容器的状态是否正常**，这是 Docker 1.12 引入的新指令。

在没有 `HEALTHCHECK` 指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。

而自 1.12 之后，Docker 提供了 `HEALTHCHECK` 指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 `starting`，在 `HEALTHCHECK` 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`。

`HEALTHCHECK` 支持下列选项：

- `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
- `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 `unhealthy`，默认 3 次。

和 `CMD`, `ENTRYPOINT` 一样，`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 `ENTRYPOINT` 一样，分为 `shell` 格式，和 `exec` 格式。命令的返回值决定了该次健康检查的成功与否：`0`：成功；`1`：失败；`2`：保留，不要使用这个值。

假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 `curl` 来帮助判断，其 `Dockerfile` 的 `HEALTHCHECK` 可以这么写：

```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

使用 `docker build` 来构建这个镜像：

```
$ docker build -t myweb:v1 .
```

构建好了后，我们启动一个容器：

```
$ docker run -d --name web -p 80:80 myweb:v1
```

当运行该镜像后，可以通过 `docker container ls` 看到最初的状态为 `(health: starting)`：

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds (health: starting)   80/tcp, 443/tcp     web
```

在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   18 seconds ago      Up 16 seconds (healthy)   80/tcp, 443/tcp     web
```

如果健康检查连续失败超过了重试次数，状态就会变为 `(unhealthy)`。

为了帮助排障，健康检查命令的输出（包括 `stdout` 以及 `stderr`）都会被存储于健康状态里，可以用 `docker inspect` 来查看。

```
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
        {
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</title>\n<style>\n    body {\n        width: 35em;\n        margin: 0 auto;\n        font-family: Tahoma, Verdana, Arial, sans-serif;\n    }\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully installed and\nworking. Further configuration is required.</p>\n\n<p>For online documentation and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCommercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
            "Start": "2016-11-25T14:35:37.780192565Z"
        }
    ],
    "Status": "healthy"
}
```

#### ONBUILD 为他人做嫁衣裳

格式：`ONBUILD <其它指令>`。

**`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令**，比如 `RUN`, `COPY` 等，而这些指令，**在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行**。

`Dockerfile` 中的其它指令都是为了定制当前镜像而准备的，唯有 `ONBUILD` 是为了帮助别人定制自己而准备的。

假设我们要制作 Node.js 所写的应用的镜像。我们都知道 Node.js 使用 `npm` 进行包管理，所有依赖、配置、启动信息等会放到 `package.json` 文件里。在拿到程序代码后，需要先进行 `npm install` 才可以获得所有需要的依赖。然后就可以通过 `npm start` 来启动应用。因此，一般来说会这样写 `Dockerfile`：

```
FROM node:slim
RUN mkdir /app
WORKDIR /app
COPY ./package.json /app
RUN [ "npm", "install" ]
COPY . /app/
CMD [ "npm", "start" ]
```

把这个 `Dockerfile` 放到 Node.js 项目的根目录，构建好镜像后，就可以直接拿来启动容器运行。但是如果我们还有第二个 Node.js 项目也差不多呢？好吧，那就再把这个 `Dockerfile` 复制到第二个项目里。那如果有第三个项目呢？再复制么？文件的副本越多，版本控制就越困难，让我们继续看这样的场景维护的问题。

如果第一个 Node.js 项目在开发过程中，发现这个 `Dockerfile` 里存在问题，比如敲错字了、或者需要安装额外的包，然后开发人员修复了这个 `Dockerfile`，再次构建，问题解决。第一个项目没问题了，但是第二个项目呢？虽然最初 `Dockerfile` 是复制、粘贴自第一个项目的，但是并不会因为第一个项目修复了他们的 `Dockerfile`，而第二个项目的 `Dockerfile` 就会被自动修复。

那么我们可不可以做一个基础镜像，然后各个项目使用这个基础镜像呢？这样基础镜像更新，各个项目不用同步 `Dockerfile` 的变化，重新构建后就继承了基础镜像的更新？好吧，可以，让我们看看这样的结果。那么上面的这个 `Dockerfile` 就会变为：

```
FROM node:slim
RUN mkdir /app
WORKDIR /app
CMD [ "npm", "start" ]
```

这里我们把项目相关的构建指令拿出来，放到子项目里去。假设这个基础镜像的名字为 `my-node` 的话，各个项目内的自己的 `Dockerfile` 就变为：

```
FROM my-node
COPY ./package.json /app
RUN [ "npm", "install" ]
COPY . /app/
```

基础镜像变化后，各个项目都用这个 `Dockerfile` 重新构建镜像，会继承基础镜像的更新。

那么，问题解决了么？没有。准确说，只解决了一半。如果这个 `Dockerfile` 里面有些东西需要调整呢？比如 `npm install` 都需要加一些参数，那怎么办？这一行 `RUN` 是不可能放入基础镜像的，因为涉及到了当前项目的 `./package.json`，难道又要一个个修改么？所以说，这样制作基础镜像，只解决了原来的 `Dockerfile` 的前4条指令的变化问题，而后面三条指令的变化则完全没办法处理。

`ONBUILD` 可以解决这个问题。让我们用 `ONBUILD` 重新写一下基础镜像的 `Dockerfile`:

```
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

这次我们回到原始的 `Dockerfile`，但是这次将项目相关的指令加上 `ONBUILD`，这样在构建基础镜像的时候，这三行并不会被执行。然后各个项目的 `Dockerfile` 就变成了简单地：

```
FROM my-node
```

是的，只有这么一行。当在各个项目目录中，用这个只有一行的 `Dockerfile` 构建镜像时，之前基础镜像的那三行 `ONBUILD` 就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行 `npm install`，生成应用镜像。

#### LABEL 指令

`LABEL` 指令用来给镜像以键值对的形式添加一些元数据（metadata）。

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

我们还可以用一些标签来申明镜像的作者、文档地址等：

```
LABEL org.opencontainers.image.authors="yeasy"
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```

具体可以参考 https://github.com/opencontainers/image-spec/blob/master/annotations.md

#### SHELL 指令

格式：`SHELL ["executable", "parameters"]`

```
SHELL ["/bin/sh", "-c"]
RUN lll ; ls
SHELL ["/bin/sh", "-cex"]
RUN lll ; ls
```

两个 `RUN` 运行同一命令，第二个 `RUN` 运行的命令会打印出每条命令并当遇到错误时退出。

当 `ENTRYPOINT` `CMD` 以 shell 格式指定时，`SHELL` 指令所指定的 shell 也会成为这两个指令的 shell

```
SHELL ["/bin/sh", "-cSHELL ["/bin/sh", "-cex"]
# /bin/sh -cex "nginx"
SHELL ["/bin/sh", "-cex"]
# /bin/sh -cex "nginx"
CMD nginx
```

### 多阶段构建

#### 全部放入一个 Dockerfile

一种方式是将所有的构建过程编包含在一个 `Dockerfile` 中，包括项目及其依赖库的编译、测试、打包等流程，这里可能会带来的一些问题：

- 镜像层次多，镜像体积较大，部署时间变长
- 源代码存在泄露的风险

例如，编写 `app.go` 文件，该程序输出 `Hello World!`

```
package main

import "fmt"

func main(){
	fmt.Printf("Hello World!");
}
```

编写 `Dockerfile.one` 文件
```
FROM golang:alpine

RUN apk --no-cache add git ca-certificates

WORKDIR /go/src/github.com/go/helloworld/

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app . \
  && cp /go/src/github.com/go/helloworld/app /root

WORKDIR /root/

CMD ["./app"]
```

构建镜像

```
$ docker build -t go/helloworld:1 -f Dockerfile.one .
```

#### 分散到多个 Dockerfile

另一种方式，就是我们事先**在一个 `Dockerfile` 将项目及其依赖库编译测试打包好后，再将其拷贝到运行环境中，这种方式需要我们编写两个 `Dockerfile` 和一些编译脚本才能将其两个阶段自动整合起来**，这种方式虽然可以很好地规避第一种方式存在的风险，但明显部署过程较复杂。

例如，编写 `Dockerfile.build` 文件
```
FROM golang:alpine

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
```


编写 `Dockerfile.copy` 文件
```
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY app .

CMD ["./app"]
```


新建 `build.sh`
```
\#!/bin/sh
echo Building go/helloworld:build

docker build -t go/helloworld:build . -f Dockerfile.build

docker create --name extract go/helloworld:build

docker cp extract:/go/src/github.com/go/helloworld/app ./app

docker rm -f extract

echo Building go/helloworld:2

docker build --no-cache -t go/helloworld:2 . -f Dockerfile.copy

rm ./app
```


现在运行脚本即可构建镜像
```
$ chmod +x build.sh

$ ./build.sh
```


对比两种方式生成的镜像大小
```
$ docker image ls

REPOSITORY      TAG    IMAGE ID        CREATED         SIZE
go/helloworld   2      f7cf3465432c    22 seconds ago  6.47MB
go/helloworld   1      f55d3e16affc    2 minutes ago   295MB
```


#### 使用多阶段构建

为解决以上问题，Docker v17.05 开始支持多阶段构建 (`multistage builds`)。使用多阶段构建我们就可以很容易解决前面提到的问题，并且只需要编写一个 `Dockerfile`：

例如，编写 `Dockerfile` 文件
```
FROM golang:alpine as builder

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld/

RUN go get -d -v github.com/go-sql-driver/mysql

COPY app.go .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .


FROM alpine:latest as prod

RUN apk --no-cache add ca-certificates

WORKDIR /root/
# 从编译阶段的中拷贝编译结果到当前镜像中
COPY --from=0 /go/src/github.com/go/helloworld/app .

CMD ["./app"]
```


构建镜像
```
$ docker build -t go/helloworld:3 .
```


对比三个镜像大小
```
$ docker image ls

REPOSITORY        TAG   IMAGE ID         CREATED            SIZE
go/helloworld     3     d6911ed9c846     7 seconds ago      6.47MB
go/helloworld     2     f7cf3465432c     22 seconds ago     6.47MB
go/helloworld     1     f55d3e16affc     2 minutes ago      295MB
```


很明显使用多阶段构建的镜像体积小，同时也完美解决了上边提到的问题。

#### 只构建某一阶段的镜像

我们可以使用 `as` 来为某一阶段命名，例如
```
FROM golang:alpine as builder
```


例如当我们只想构建 `builder` 阶段的镜像时，增加 `--target=builder` 参数即可

```
$ docker build --target builder -t username/imagename:tag .
```

#### 构建时从其他镜像复制文件

上面例子中我们使用 `COPY --from=0 /go/src/github.com/go/helloworld/app .` 从上一阶段的镜像中复制文件，我们也可以复制任意镜像中的文件。

```
$ COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```



## docker compose

[参考](https://www.bookstack.cn/read/docker_practice-1.3.0/compose-README.md)、[参考](https://www.cnblogs.com/liaokui/p/11380590.html)

`Docker Compose` 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用

`Compose` 定位是 「**定义和运行多个 Docker 容器的应用**（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

我们知道使用一个 `Dockerfile` 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose` 恰好满足了这样的需求。**它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）**。

`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。

### Compose 命令



### Compose 模板文件

模板文件是使用 `Compose` 的核心，涉及到的指令关键字也比较多。但大家不用担心，这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

```
version: "3"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

注意每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中重复设置。

下面分别介绍各个指令的用法。

#### compose 指令

##### `build`

指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```
version: '3'
services:
  webapp:
    build: ./dir
```

你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。

使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

##### `arg`

使用 `arg` 指令指定构建镜像时的变量。

```
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

使用 `cache_from` 指定构建镜像的缓存

```
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

##### `cap_add, cap_drop`

指定容器的内核能力（capacity）分配。

例如，让容器拥有所有能力可以指定为：

```
cap_add:
  - ALL
```

去掉 NET_ADMIN 能力可以指定为：

```
cap_drop:
  - NET_ADMIN
```

##### `command`

覆盖容器启动后默认执行的命令。

```
command: echo "hello world"
```

##### `configs`

仅用于 `Swarm mode`，详细内容请查看 [`Swarm mode`](https://www.bookstack.cn/read/docker_practice-1.3.0/$swarm_mode) 一节。

##### `cgroup_parent`

指定父 `cgroup` 组，意味着将继承该组的资源限制。

例如，创建了一个 cgroup 组名称为 `cgroups_1`。

```
cgroup_parent: cgroups_1
```

##### `container_name`

指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。

```
container_name: docker-web-container
```

> 注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

##### `deploy`

仅用于 `Swarm mode`，详细内容请查看 [`Swarm mode`](https://www.bookstack.cn/read/docker_practice-1.3.0/$swarm_mode) 一节

##### `devices`

指定设备映射关系。

```
devices:
  - "/dev/ttyUSB1:/dev/ttyUSB0"
```

##### `depends_on`

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

```
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> 注意：`web` 服务不会等待 `redis` `db` 「完全启动」之后才启动。

##### `dns`

自定义 `DNS` 服务器。可以是一个值，也可以是一个列表。

```
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 114.114.114.114
```

##### `dns_search`

配置 `DNS` 搜索域。可以是一个值，也可以是一个列表。

```
dns_search: example.comdns_search:  - domain1.example.com  - domain2.example.com
```

##### `tmpfs`

挂载一个 tmpfs 文件系统到容器。

```
tmpfs: /runtmpfs:  - /run  - /tmp
```

##### `env_file`

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

```
dns_search: example.com
dns_search:
  - domain1.example.com
  - domain2.example.com
```

环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

```
# common.env: Set development environment
PROG_ENV=development
```

##### `environment`

设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

```
environment:
  RACK_ENV: development
  SESSION_SECRET:
environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](https://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括

```
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```

##### `expose`

暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数

```
expose:
 - "3000"
 - "8000"
```

##### `external_links`

> 注意：不建议使用该指令。

链接到 `docker-compose.yml` 外部的容器，甚至并非 `Compose` 管理的外部容器。

```
extra_hosts:
 - "googledns:8.8.8.8"
 - "dockerhub:52.1.157.61"
```

##### `extra_hosts`

类似 Docker 中的 `--add-host` 参数，指定额外的 host 名称映射信息。

```
8.8.8.8 googledns
52.1.157.61 dockerhub
```

会在启动后的服务容器中 `/etc/hosts` 文件中添加如下两条条目。

```
8.8.8.8 googledns52.1.157.61 dockerhub
```

##### `healthcheck`

通过命令检查容器是否健康运行。

```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

##### `image`

指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

```
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

##### `labels`

为容器添加 Docker 元数据（metadata）信息。例如可以为容器添加辅助说明信息。

```
labels:
  com.startupteam.description: "webapp for a startup team"
  com.startupteam.department: "devops department"
  com.startupteam.release: "rc3 for v1.0"
```

##### `links`

> 注意：不推荐使用该指令。

##### `logging`

配置日志选项。

```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

目前支持三种日志驱动类型。

```
driver: "json-file"
driver: "syslog"
driver: "none"
```

`options` 配置日志驱动的相关参数。

```
options:
  max-size: "200k"
  max-file: "10"
```

##### `network_mode`

设置网络模式。使用和 `docker run` 的 `--network` 参数一样的值。

```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

##### `networks`

配置容器连接的网络。

```
version: "3"
services:
  some-service:
    networks:
     - some-network
     - other-network
networks:
  some-network:
  other-network:
```

##### `pid`

跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 ID 来相互访问和操作。

```
pid: "host"
```

##### `ports`

暴露端口信息。

使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

*注意：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 `YAML` 会自动解析 `xx:yy` 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。*

##### `secrets`

存储敏感数据，例如 `mysql` 服务密码。

```
version: "3.1"
services:
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

##### `security_opt`

指定容器模板标签（label）机制的默认属性（用户、角色、类型、级别等）。例如配置标签的用户名和角色名。

```
security_opt:
    - label:user:USER
    - label:role:ROLE
```

##### `stop_signal`

设置另一个信号来停止容器。在默认情况下使用的是 SIGTERM 停止容器。

```
stop_signal: SIGUSR1
```

##### `sysctls`

配置容器内核参数。

```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

##### `ulimits`

指定容器的 ulimits 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。

```
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```

##### `volumes`

数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。

该指令中路径支持相对路径。

```
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

如果路径为数据卷名称，必须在文件中配置数据卷。

```
version: "3"
services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql
volumes:
  mysql_data:
```

##### 其它指令

此外，还有包括 `domainname, entrypoint, hostname, ipc, mac_address, privileged, read_only, shm_size, restart, stdin_open, tty, user, working_dir` 等指令，基本跟 `docker run` 中对应参数的功能一致。

指定服务容器启动后执行的入口文件。

```
entrypoint: /code/entrypoint.sh
```

指定容器中运行应用的用户名。

```
user: nginx
```

指定容器中工作目录。

```
working_dir: /code
```

指定容器中搜索域名、主机名、mac 地址等。

```
domainname: your_website.com
hostname: test
mac_address: 08-00-27-00-0C-0A
```

允许容器中运行一些特权命令。

```
privileged: true
```

指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 `always` 或者 `unless-stopped`。

```
restart: always
```

以只读模式挂载容器的 root 文件系统，意味着不能对容器内容进行修改。

```
read_only: true
```

打开标准输入，可以接受外部输入。

```
stdin_open: true
```

模拟一个伪终端。

```
tty: true
```

###### 读取变量

Compose 模板文件支持动态读取主机的系统环境变量和当前目录下的 `.env` 文件中的变量。

例如，下面的 Compose 文件将从运行它的环境中读取变量 `${MONGO_VERSION}` 的值，并写入执行的指令中。

```
version: "3"
services:
db:
  image: "mongo:${MONGO_VERSION}"
```

如果执行 `MONGO_VERSION=3.2 docker-compose up` 则会启动一个 `mongo:3.2` 镜像的容器；如果执行 `MONGO_VERSION=2.8 docker-compose up` 则会启动一个 `mongo:2.8` 镜像的容器。

若当前目录存在 `.env` 文件，执行 `docker-compose` 命令时将从该文件中读取变量。

在当前目录新建 `.env` 文件并写入以下内容。

```
# 支持 # 号注释
MONGO_VERSION=3.6
```

执行 `docker-compose up` 则会启动一个 `mongo:3.6` 镜像的容器。

### 实战

[参考](https://www.bookstack.cn/read/docker_practice-1.3.0/compose-django.md)



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

## 安全

### 内核命名空间

Docker 容器和 LXC 容器很相似，所提供的安全特性也差不多。当用 `docker run` 启动一个容器时，**在后台 Docker 为容器创建了一个独立的命名空间和控制组集合**。

命名空间提供了最基础也是最直接的隔离，在容器中运行的进程不会被运行在主机上的进程和其它容器发现和作用。

**每个容器都有自己独有的网络栈**，意味着它们不能访问其他容器的 sockets 或接口。不过，如果主机系统上做了相应的设置，容器可以像跟主机交互一样的和其他容器交互。当指定公共端口或使用 links 来连接 2 个容器时，容器就可以相互通信了（可以根据配置来限制通信的策略）。

从网络架构的角度来看，所有的容器通过本地主机的网桥接口相互通信，就像物理机器通过物理交换机通信一样。

那么，内核中实现命名空间和私有网络的代码是否足够成熟？

内核命名空间从 2.6.15 版本（2008 年 7 月发布）之后被引入，数年间，这些机制的可靠性在诸多大型生产系统中被实践验证。

实际上，命名空间的想法和设计提出的时间要更早，最初是为了在内核中引入一种机制来实现 [OpenVZ](https://en.wikipedia.org/wiki/OpenVZ) 的特性。 而 OpenVZ 项目早在 2005 年就发布了，其设计和实现都已经十分成熟。

### 控制组

控制组是 Linux 容器机制的另外一个关键组件，**负责实现资源的审计和限制**。

它提供了很多有用的特性；以及**确保各个容器可以公平地分享主机的内存、CPU、磁盘 IO 等资源；当然，更重要的是，控制组确保了当容器内的资源使用产生压力时不会连累主机系统**。

尽管控制组不负责隔离容器之间相互访问、处理数据和进程，它在防止拒绝服务（DDOS）攻击方面是必不可少的。尤其是在多用户的平台（比如公有或私有的 PaaS）上，控制组十分重要。例如，当某些应用程序表现异常的时候，可以保证一致地正常运行和性能。

控制组机制始于 2006 年，内核从 2.6.24 版本开始被引入。

### Docker服务端的防护

运行一个容器或应用程序的核心是通过 Docker 服务端。**Docker 服务的运行目前需要 root 权限，因此其安全性十分关键**。

首先，确保只有可信的用户才可以访问 Docker 服务。Docker 允许用户在主机和容器间共享文件夹，同时不需要限制容器的访问权限，这就容易让容器突破资源限制。例如，恶意用户启动容器的时候将主机的根目录`/`映射到容器的 `/host` 目录中，那么容器理论上就可以对主机的文件系统进行任意修改了。这听起来很疯狂？但是事实上几乎所有虚拟化系统都允许类似的资源共享，而没法禁止用户共享主机根文件系统到虚拟机系统。

这将会造成很严重的安全后果。因此，当提供容器创建服务时（例如通过一个 web 服务器），要更加注意进行参数的安全检查，防止恶意的用户用特定参数来创建一些破坏性的容器。

为了加强对服务端的保护，Docker 的 REST API（客户端用来跟服务端通信）在 0.5.2 之后使用本地的 Unix 套接字机制替代了原先绑定在 127.0.0.1 上的 TCP 套接字，因为后者容易遭受跨站脚本攻击。现在用户使用 Unix 权限检查来加强套接字的访问安全。

用户仍可以利用 HTTP 提供 REST API 访问。建议使用安全机制，确保只有可信的网络或 VPN，或证书保护机制（例如受保护的 stunnel 和 ssl 认证）下的访问可以进行。此外，还可以使用 [HTTPS 和证书](https://docs.docker.com/engine/security/https/) 来加强保护。

最近改进的 Linux 命名空间机制将可以实现使用非 root 用户来运行全功能的容器。这将从根本上解决了容器和主机之间共享文件系统而引起的安全问题。

终极目标是改进 2 个重要的安全特性：

- 将容器的 root 用户 [映射到本地主机上的非 root 用户](https://docs.docker.com/engine/security/userns-remap/)，减轻容器和主机之间因权限提升而引起的安全问题；
- 允许 Docker 服务端在 [非 root 权限(rootless 模式)](https://docs.docker.com/engine/security/rootless/) 下运行，利用安全可靠的子进程来代理执行需要特权权限的操作。这些子进程将只允许在限定范围内进行操作，例如仅仅负责虚拟网络设定或文件系统管理、配置操作等。

最后，建议采用专用的服务器来运行 Docker 和相关的管理服务（例如管理服务比如 ssh 监控和进程监控、管理工具 nrpe、collectd 等）。其它的业务服务都放到容器中去运行。

### 内核能力机制

[能力机制（Capability）](https://man7.org/linux/man-pages/man7/capabilities.7.html) 是 Linux 内核一个强大的特性，可以提供细粒度的权限访问控制。 Linux 内核自 2.2 版本起就支持能力机制，它将权限划分为更加细粒度的操作能力，既可以作用在进程上，也可以作用在文件上。

例如，一个 Web 服务进程只需要绑定一个低于 1024 的端口的权限，并不需要 root 权限。那么它只需要被授权 `net_bind_service` 能力即可。此外，还有很多其他的类似能力来避免进程获取 root 权限。

默认情况下，Docker 启动的容器被严格限制只允许使用内核的一部分能力。

使用能力机制对加强 Docker 容器的安全有很多好处。通常，在服务器上会运行一堆需要特权权限的进程，包括有 ssh、cron、syslogd、硬件管理工具模块（例如负载模块）、网络配置工具等等。容器跟这些进程是不同的，因为几乎所有的特权进程都由容器以外的支持系统来进行管理。

- ssh 访问被主机上ssh服务来管理；
- cron 通常应该作为用户进程执行，权限交给使用它服务的应用来处理；
- 日志系统可由 Docker 或第三方服务管理；
- 硬件管理无关紧要，容器中也就无需执行 udevd 以及类似服务；
- 网络管理也都在主机上设置，除非特殊需求，容器不需要对网络进行配置。

从上面的例子可以看出，大部分情况下，容器并不需要“真正的” root 权限，容器只需要少数的能力即可。为了加强安全，容器可以禁用一些没必要的权限。

- 完全禁止任何 mount 操作；
- 禁止直接访问本地主机的套接字；
- 禁止访问一些文件系统的操作，比如创建新的设备、修改文件属性等；
- 禁止模块加载。

这样，就算攻击者在容器中取得了 root 权限，也不能获得本地主机的较高权限，能进行的破坏也有限。

默认情况下，Docker采用 [白名单](https://github.com/moby/moby/blob/master/oci/caps/defaults.go) 机制，禁用必需功能之外的其它权限。 当然，用户也可以根据自身需求来为 Docker 容器启用额外的权限。

## 底层实现

Docker 底层的核心技术包括 Linux 上的**命名空间**（Namespaces）、**控制组**（Control groups）、**Union 文件系统**（Union file systems）和**容器格式**（Container format）。

我们知道，传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

我们知道，在操作系统中，包括内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU 等等，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。

随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配。

### 基本架构

Docker 采用了 `C/S` 架构，包括客户端和服务端。**Docker 守护进程 （`Daemon`）作为服务端接受来自客户端的请求，并处理这些请求（创建、运行、分发容器）**。

客户端和服务端既可以运行在一个机器上，也可通过 `socket` 或者 `RESTful API` 来进行通信。

![Docker 基本架构](https://static.sitestack.cn/projects/docker_practice-1.3.0/underly/_images/docker_arch.png)

Docker 守护进程一般在宿主主机后台运行，等待接收来自客户端的消息。

Docker 客户端则为用户提供一系列可执行命令，用户用这些命令实现跟 Docker 守护进程交互。

### 命名空间

命名空间是 Linux 内核一个强大的特性。**每个容器都有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统中运行一样。命名空间保证了容器之间彼此互不影响**。

#### pid 命名空间

不同用户的进程就是通过 pid 命名空间隔离开的，且不同命名空间中可以有相同 pid。所有的 LXC 进程在 Docker 中的父进程为 Docker 进程，每个 LXC 进程具有不同的命名空间。同时由于允许嵌套，因此可以很方便的实现嵌套的 Docker 容器。

#### net 命名空间

有了 pid 命名空间，每个命名空间中的 pid 能够相互隔离，但是网络端口还是共享 host 的端口。网络隔离是通过 net 命名空间实现的， 每个 net 命名空间有独立的 网络设备，IP 地址，路由表，/proc/net 目录。这样每个容器的网络就能隔离开来。Docker 默认采用 veth 的方式，将容器中的虚拟网卡同 host 上的一 个Docker 网桥 docker0 连接在一起。

#### ipc 命名空间

容器中进程交互还是采用了 Linux 常见的进程间交互方法(interprocess communication - IPC)， 包括信号量、消息队列和共享内存等。然而同 VM 不同的是，容器的进程间交互实际上还是 host 上具有相同 pid 命名空间中的进程间交互，因此需要在 IPC 资源申请时加入命名空间信息，每个 IPC 资源有一个唯一的 32 位 id。

#### mnt 命名空间

类似 chroot，将一个进程放到一个特定的目录执行。mnt 命名空间允许不同命名空间的进程看到的文件结构不同，这样每个命名空间 中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个命名空间中的容器在 /proc/mounts 的信息只包含所在命名空间的 mount point。

#### uts 命名空间

UTS(“UNIX Time-sharing System”) 命名空间允许每个容器拥有独立的 hostname 和 domain name， 使其在网络上可以被视作一个独立的节点而非 主机上的一个进程。

#### user 命名空间

每个容器可以有不同的用户和组 id， 也就是说可以在容器内用容器内部的用户执行程序而非主机上的用户。

*注：更多关于 Linux 上命名空间的信息，请阅读 [这篇文章](https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/)。

### 控制组

控制组（[cgroups](https://en.wikipedia.org/wiki/Cgroups)）是 Linux 内核的一个特性，**主要用来对共享资源进行隔离、限制、审计等**。只有能控制分配到容器的资源，才能避免当多个容器同时运行时的对系统资源的竞争。

控制组技术最早是由 Google 的程序员在 2006 年提出，Linux 内核自 2.6.24 开始支持。

控制组可以提供对容器的内存、CPU、磁盘 IO 等资源的限制和审计管理。

### 联合文件系统

联合文件系统（[UnionFS](https://en.wikipedia.org/wiki/UnionFS)）**是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下**(unite several directories into a single virtual filesystem)。

联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 中使用的 AUFS（Advanced Multi-Layered Unification Filesystem）就是一种联合文件系统。 `AUFS` 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 `AUFS` 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。

Docker 目前支持的联合文件系统包括 `OverlayFS`, `AUFS`, `Btrfs`, `VFS`, `ZFS` 和 `Device Mapper`。

各 Linux 发行版 Docker 推荐使用的存储驱动如下表。

| Linux 发行版     | Docker 推荐使用的存储驱动                           |
| :--------------- | :-------------------------------------------------- |
| Docker on Ubuntu | `overlay2` (16.04 +)                                |
| Docker on Debian | `overlay2` (Debian Stretch), `aufs`, `devicemapper` |
| Docker on CentOS | `overlay2`                                          |
| Docker on Fedora | `overlay2`                                          |

在可能的情况下，[推荐](https://docs.docker.com/storage/storagedriver/select-storage-driver/) 使用 `overlay2` 存储驱动，`overlay2` 是目前 Docker 默认的存储驱动，以前则是 `aufs`。你可以通过配置来使用以上提到的其他类型的存储驱动。

### 容器格式

最初，Docker 采用了 `LXC` 中的容器格式。从 0.7 版本以后开始去除 LXC，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，从 1.11 开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 和 [containerd](https://github.com/containerd/containerd)。

### Docker 网络实现

Docker 的网络实现其实就是**利用了 Linux 上的网络命名空间和虚拟网络设备**（特别是 veth pair）。建议先熟悉了解这两部分的基本概念再阅读本章。

#### 基本原理

首先，要实现网络通信，机器需要至少一个网络接口（物理接口或虚拟接口）来收发数据包；此外，如果不同子网之间要进行通信，需要路由机制。

Docker 中的网络接口默认都是虚拟的接口。虚拟接口的优势之一是转发效率较高。 Linux 通过在内核中进行数据复制来实现虚拟接口之间的数据转发，发送接口的发送缓存中的数据包被直接复制到接收接口的接收缓存中。对于本地系统和容器内系统看来就像是一个正常的以太网卡，只是它不需要真正同外部网络设备通信，速度要快很多。

Docker 容器网络就利用了这项技术。它在本地主机和容器内分别创建一个虚拟接口，并让它们彼此连通（这样的一对接口叫做 `veth pair`）。

#### 创建网络参数

Docker 创建一个容器的时候，会执行如下操作：

- 创建一对虚拟接口，分别放到本地主机和新容器中；
- 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 veth65f9；
- 容器一端放到新容器中，并修改名字作为 eth0，这个接口只在容器的命名空间可见；
- 从网桥可用地址段中获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 veth65f9。

完成这些之后，容器就可以使用 eth0 虚拟网卡来连接其他容器和其他网络。

可以在 `docker run` 的时候通过 `--net` 参数来指定容器的网络配置，有4个可选值：

- `--net=bridge` 这个是默认值，连接到默认的网桥。
- `--net=host` 告诉 Docker 不要将容器网络放到隔离的命名空间中，即不要容器化容器内的网络。此时容器使用本地主机的网络，它拥有完全的本地主机接口访问权限。容器进程可以跟主机其它 root 进程一样可以打开低范围的端口，可以访问本地网络服务比如 D-bus，还可以让容器做一些影响整个主机系统的事情，比如重启主机。因此使用这个选项的时候要非常小心。如果进一步的使用 `--privileged=true`，容器会被允许直接配置主机的网络堆栈。
- `--net=container:NAME_or_ID` 让 Docker 将新建容器的进程放到一个已存在容器的网络栈中，新容器进程有自己的文件系统、进程列表和资源限制，但会和已存在的容器共享 IP 地址和端口等网络资源，两者进程可以直接通过 `lo` 环回接口通信。
- `--net=none` 让 Docker 将新容器放到隔离的网络栈中，但是不进行网络配置。之后，用户可以自己进行配置。

#### 网络配置细节

用户使用 `--net=none` 后，可以自行配置网络，让容器达到跟平常一样具有访问网络的权限。通过这个过程，可以了解 Docker 配置网络的细节。

首先，启动一个 `/bin/bash` 容器，指定 `--net=none` 参数。

```
$ docker run -i -t --rm --net=none base /bin/bashroot@63f36fc01b5f:/#
```

在本地主机查找容器的进程 id，并为它创建网络命名空间。

```
$ docker inspect -f '{{.State.Pid}}' 63f36fc01b5f2778$ pid=2778$ sudo mkdir -p /var/run/netns$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid
```

检查桥接网卡的 IP 和子网掩码信息。

```
$ ip addr show docker021: docker0: ...inet 172.17.42.1/16 scope global docker0...
```

创建一对 “veth pair” 接口 A 和 B，绑定 A 到网桥 `docker0`，并启用它

```
$ sudo ip link add A type veth peer name B$ sudo brctl addif docker0 A$ sudo ip link set A up
```

将B放到容器的网络命名空间，命名为 eth0，启动它并配置一个可用 IP（桥接网段）和默认网关。

```
$ sudo ip link set B netns $pid$ sudo ip netns exec $pid ip link set dev B name eth0$ sudo ip netns exec $pid ip link set eth0 up$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0$ sudo ip netns exec $pid ip route add default via 172.17.42.1
```

以上，就是 Docker 配置网络的具体过程。

当容器结束后，Docker 会清空容器，容器内的 eth0 会随网络命名空间一起被清除，A 接口也被自动从 `docker0` 卸载。

此外，用户可以使用 `ip netns exec` 命令来在指定网络命名空间中进行配置，从而配置容器内的网络。

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

### 解释基本的Docker使用工作流程是怎样的？

答：（1）从Dockerfile开始，Dockerfile是镜像的源代码；（2）创建Dockerfile后，可以构建它以创建容器的镜像。图像只是“源代码”的“编译版本”，即Dockerfile；（3）获得容器的镜像后，应使用注册表重新分发容器。注册表就像一个git存储库，可以推送和拉取镜像；接下来，可以使用该图像来运行容器。在许多方面，正在运行的容器与虚拟机（但没有虚拟机管理程序）非常相似。

### 如何在生产中监控Docker？

答：Docker提供docker stats和docker事件等工具来监控生产中的Docker。我们可以使用这些命令获取重要统计数据的报告。

Docker统计数据：当我们使用容器ID调用docker stats时，我们获得容器的CPU，内存使用情况等。它类似于Linux中的top命令。

Docker事件：Docker事件是一个命令，用于查看Docker守护程序中正在进行的活动流。一些常见的Docker事件是：attach，commit，die，detach，rename，destroy等。







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

