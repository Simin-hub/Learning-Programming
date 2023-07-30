# Git

[参考](https://git-scm.com/book/zh/v2)

## Git 是什么？

Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方式。 从概念上来说，其它大部分系统以文件变更列表的方式存储信息，这类系统（CVS、Subversion、Perforce、Bazaar 等等） 将它们存储的信息看作是一组基本文件和每个文件随时间逐步累积的差异 （它们通常称作 **基于差异（delta-based）** 的版本控制）。

为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

### 三种状态

现在请注意，如果你希望后面的学习更顺利，请记住下面这些关于 Git 的概念。 Git 有三种状态，你的文件可能处于其中之一： **已提交（committed）**、**已修改（modified）** 和 **已暂存（staged）**。

- 已修改表示修改了文件，但还没保存到数据库中。
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
- 已提交表示数据已经安全地保存在本地数据库中。

这会让我们的 Git 项目拥有三个阶段：工作区、暂存区以及 Git 目录。

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/areas.png" alt="工作区、暂存区以及 Git 目录。" style="zoom:50%;" />

**工作区是对项目的某个版本独立提取出来的内容**。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。对应不同分支的本地所看到的目录，即资源管理器所看到的目录。

暂存区是一个文件，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。 按照 Git 的术语叫做“索引”，不过一般说法还是叫“暂存区”。`git add .`加入暂存区。

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。`git commit`

基本的 Git 工作流程如下：

1. 在工作区中修改文件。
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。

如果 Git 目录中保存着特定版本的文件，就属于 **已提交** 状态。 如果文件已修改并放入暂存区，就属于 **已暂存** 状态。 如果自上次检出后，作了修改但还没有放到暂存区域，就是 **已修改** 状态。 

### 本地仓库

版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

### 远程仓库

 远程仓库是指托管在因特网或其他网络中的你的项目的版本库。github和github都是基于git的web代码仓库管理软件。

一个本地仓库可以对应多个远程仓库。

```
# 查看本地仓库对应的远程仓库
git remote -v
```

远程仓库通常只是一个裸仓库（bare repository）——即一个没有当前工作目录的仓库。 因为该仓库仅仅作为合作媒介，不需要从磁盘检查快照；存放的只有 Git 的资料。 简单的说，裸仓库就是你工程目录内的 `.git` 子目录内容，不包含其他资料。

### 分支

Git 的分支，其实本质上仅仅是**指向提交对象的可变指针**。 Git 的默认分支名字是 `master`。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 `master` 分支。 `master` 分支会在每次提交时自动向前移动。

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/7251bd4d7b1af4aaaa3ffaf9ed004c48.png" alt="深层次的分支概念图" style="zoom:50%;" />

#### 分支管理

创建、删除、合并。

#### 合并

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/basic-merging-1.png" alt="一次典型合并中所用到的三个快照。" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/basic-merging-2.png" alt="一个合并提交。" style="zoom:50%;" />

#### 删除

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20230702114156558.png" alt="image-20230702114156558" style="zoom:50%;" />

#### 远程分支

远程分支是开发者在同一个项目上同时协作的方式。

远程分支存在于一个远程仓库中（通常称为 `origin`），托管在 [GitHub](https://github.com/) 等平台上。

远程分支是对远程仓库的引用（指针），包括分支、标签等等。 你可以通过 `git ls-remote <remote>` 来显式地获得远程引用的完整列表， 或者通过 `git remote show <remote>` 获得远程分支的更多信息。 然而，一个更常见的做法是利用远程跟踪分支。

远程跟踪分支是远程分支状态的引用。它们是你无法移动的本地引用。一旦你进行了网络通信， Git 就会为你移动它们以精确反映远程仓库的状态。请将它们看做书签， 这样可以提醒你该分支在远程仓库中的位置就是你最后一次连接到它们的位置。

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/remote-branches-1.png" alt="克隆之后的服务器与本地仓库。" style="zoom:80%;" />

> “origin” 并无特殊含义
>
> 远程仓库名字 “origin” 与分支名字 “master” 一样，在 Git 中并没有任何特别的含义一样。 同时 “master” 是当你运行 `git init` 时默认的起始分支名字，原因仅仅是它的广泛使用， “origin” 是当你运行 `git clone` 时默认的远程仓库名字。 如果你运行 `git clone -o booyah`，那么你默认的远程分支名字将会是 `booyah/master`。

### 本地仓库、远程仓库、本地分支、远程分支

在Git中，远程仓库和本地仓库之间是相互关联的，它们可以进行代码的同步和交互。

远程仓库是位于远程服务器上的代码仓库，可以与多个开发者共享。常见的远程仓库服务提供商包括GitHub、GitLab和Bitbucket等。你可以将本地仓库与一个或多个远程仓库进行关联，以便进行代码的推送和拉取操作。

本地仓库是位于你的本地计算机上的代码仓库，用于存储你的项目代码。当你在本地进行代码开发时，你可以在本地仓库中创建分支、修改代码等操作。本地仓库中的分支和提交记录等信息都保存在本地。

本地分支和远程分支是Git中的两个概念，它们之间也存在关联。

本地分支是基于本地仓库创建的分支，用于在本地进行代码的开发和管理。你可以创建新的本地分支，切换到不同的分支，合并分支等操作。本地分支可以用于并行开发和实验性的功能开发。

远程分支是远程仓库中的分支，它是由其他开发者推送到远程仓库的分支。当你克隆或者关联一个远程仓库时，会在本地仓库中自动创建对应的远程分支引用。你可以使用`git fetch`命令获取最新的远程分支信息，然后通过合并或者创建本地分支来将远程分支的修改应用到本地。

本地分支和远程分支之间可以建立追踪关系。当你在本地从远程分支检出一个新分支时，这个新的本地分支会与远程分支建立追踪关系。这样，在使用`git pull`命令时，Git会自动将远程分支的修改拉取到本地分支，并进行合并。

综上所述，本地仓库和远程仓库之间通过推送(push)和拉取(fetch/pull)进行代码的同步，本地分支和远程分支之间通过追踪关系进行代码的交互。

## 常用命令

[参考](https://mp.weixin.qq.com/s/JUya4OMMTZBZwNFbO-gLCw)

现在做项目 Git 代码管理是一定少不了的。多年以前可能是 SVN，我想如今的公司里面基本都转型到用 Git 了吧。

虽然如今已有很多可视化的 Git 工具，但是很多时候我们还是需要用到命令直接操作。所以我就把 Git 的相关命令汇集了起来，即方便自己，也方便大家。

**相关名词解释**

- master: 默认开发分支
- origin: 默认远程版本库
- Index / Stage：暂存区
- Workspace：工作区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

### 一、新建代码库

```
# 在当前目录新建一个Git代码库
$ git init
# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]
# 下载一个项目和它的整个代码历史
$ git clone [url]
```

这两个项目初始化的命令，一般只在项目新建时使用到，比如我们在 Github 里面新建了一个仓库：

![图片](https://mmbiz.qpic.cn/mmbiz_png/56U6KCydWqkadjTDdticsDlTGYOUViakf4H1micibUuhhc0ulicrBZK5ptUofaZmNt0YVRHJNckibGKspCl4YHS2VsBg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

你能看到他的引导流程里面，最开始的就是 git init。

### 二、配置

我们时常会遇到并不想把某些文件提交到 Git 库里面去，比如 .idea、node_models 等，这类开发工具的配置目录，三方库依赖等。

这类需求就可以通过 `.gitconfig` 文件来进行配置。它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

我们以 `kubernetes` 的代码为例：

![图片](https://mmbiz.qpic.cn/mmbiz_png/56U6KCydWqkadjTDdticsDlTGYOUViakf4k0N6AVhuCJahLeHibX28zu11eCPYLufJ3wD4PdWLfupYhL79IAKX0Nw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



你能看到他里面忽略了很多文件。

其他的 git 配置命令如下，基本不太常用：

```
# 显示当前的Git配置
$ git config --list
# 编辑Git配置文件
$ git config -e [--global]
# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

### 三、增加/删除/修改文件

```
# 查看状态
$ git status
# 查看变更内容
$ git diff
# 添加指定文件到暂存区
$ git add [file1] [file2]
# 添加指定目录到暂存区，包括子目录
$ git add [dir]
# 添加当前目录的所有文件到暂存区
$ git add
# 添加每个变化前，都会要求确认，对于同一个文件的多处变化，可以实现分次提交
$ git add -p
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2]
# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]
# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed
```

这里面我们用得最多就是 git add 和  git rm 命令。当我们在工程里面新建了一个文件，默认他是不会自动添加到 git 里面进行管理，而是需要你通过  git add 到里面。

如果是在项目初始化时，我们一般都直接使用 git add . 来把整合项目添加进去。

### 四、代码提交

```
# 提交暂存区到仓库区
$ git commit -m [message]
# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]
# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a
# 提交时显示所有diff信息
$ git commit -v
# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]
# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2]
```

### 五、分支

```
# 显示所有本地分支
$ git branch
# 列出所有远程分支
$ git branch -r
# 列出所有本地分支和远程分支
$ git branch -a
# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]
# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]
# 删除分支
$ git branch -d [branch-name]
# 删除远程分支
$ git push origin --delete [branch-namel
$ git branch -dr [remote/branch]
# 新建一个分支，并切换到该分支
$ git checkout -b [branch]
# 切换到指定分支，并更新工作区
$ git checkout [branch-name]
# 切换到上一个分支
$ git checkout -
# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]
# 合并指定分支到当前分支
$ git merge [branch]
# 衍合指定分支到当前分支
$ git rebase <branch>
# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]
```

关于分支我更喜欢使用工具去处理切换合并这些。

### 六、标签

```
# 列出所有本地标签 
$ git tag 
# 基于最新提交创建标签 
$ git tag <tagname> 
# 删除标签
$ git tag -d <tagname> 
# 删除远程tag
$ git push origin :refs/tags/[tagName]
# 查看tag信息
$ git show [tag]
# 提交指定tag
$ git push [remote] [tag]
# 提交所有tag
$ git push [remote] --tags
# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

标签一般在我们需要发版，或者代码里程碑时使用到，目的是标记一个时间点，等后期可以快速定位到这个点的代码。

### 七、查看信息

```
# 显示有变更的文件
$ git status
# 显示当前分支的版本历史
$ git log
# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat
# 搜索提交历史，根据关键词
$ git log -s [keyword]
# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s
# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature
# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]
# 显示指定文件相关的每一次diff
$ git log -p [file]
# 显示过去5次提交
$ git log -5 --pretty--oneline
# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn
# 显示指定文件是什么人在什么时间修改过
$ git blame [file]
# 显示暂存区和工作区的差异
$ git diff
# 显示暂存区和上一个commit的差异
$ git diff --cached [file]
# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD
# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]
# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
# 显示某次提交的元数据和内容变化
$ git show [commit]
# 显示某次提交发生变化的文件
$ git show --name-only [commit]
# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]
# 显示当前分支的最近几次提交
$ git reflog
```

这部分的命令用的并不太多，作为了解就好！

### 八、远程操作

```
# 下载远程仓库的所有变动
$ git fetch [remote]
# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]
# 显示所有远程仓库
$ git remote -v
# 显示某个远程仓库的信息
$ git remote show [remote]
# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]
# 上传本地指定分支到远程仓库
$ git push [remote] [branch]
# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force
# 推送所有分支到远程仓库
$ git push [remote] --all
# 删除远程分支或标签 
$ git push <remote> :<branch/tag-name> 
# 上传所有标签
$ git push --tags 
```

这里面的命令我们用得最多就是 git push，其他的都用得非常少，作为了解就可以了。

### 九、撤销

```
# 撤销工作目录中所有未提交文件的修改内容 
$ git reset --hard HEAD 
# 撤销指定的未提交文件的修改内容 
$ git checkout HEAD <file> 
# 撤销指定的提交 
$ git revert <commit> 
# 退回到之前1天的版本
$ git log --before="1 days"
# 恢复暂存区的指定文件到工作区
$ git checkout [file]
# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]
# 恢复暂存区的所有文件到工作区
$ git checkout.
# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]
# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard
# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]
# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]
# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]
# 暂时将未提交的变化移除，稍后再移入 
$ git stash
$ git stash pop
```

像代码回滚这类危险操作，个人建议新手还是使用工具操作，别问为什么，问就说明你坑踩得不够。

### 十、其他

生成一个可供发布的压缩包。

```
$ git archive
```

这个一般用得很少，现在基本都用 CICD 自动构建了，发布这些基本都是使用镜像发布了，Git 现在大都只用作代码托管用。

## 服务器上的Git

Git 可以使用四种不同的协议来传输资料：本地协议（Local），HTTP 协议，SSH（Secure Shell）协议及 Git 协议。

### 在服务器上搭建Git

#### 导出裸仓库

为了通过克隆你的仓库来创建一个新的裸仓库，你需要在克隆命令后加上 `--bare` 选项。 按照惯例，裸仓库的目录名以 .git 结尾.

```
$ git clone --bare my_project my_project.git
Cloning into bare repository 'my_project.git'...
done.
```

#### 把裸仓库放到服务器上

把裸仓库放到服务器上并设置你的协议。 假设一个域名为 `git.example.com` 的服务器已经架设好，并可以通过 SSH 连接， 你想把所有的 Git 仓库放在 `/srv/git` 目录下。 假设服务器上存在 `/srv/git/` 目录，你可以通过以下命令复制你的裸仓库来创建一个新仓库：

```console
$ scp -r my_project.git user@git.example.com:/srv/git
```

此时，其他可通过 SSH 读取此服务器上 `/srv/git` 目录的用户，可运行以下命令来克隆你的仓库。

```console
$ git clone user@git.example.com:/srv/git/my_project.git
```

如果一个用户，通过使用 SSH 连接到一个服务器，并且其对 `/srv/git/my_project.git` 目录拥有可写权限，那么他将自动拥有推送权限。

如果到该项目目录中运行 `git init` 命令，并加上 `--shared` 选项， 那么 Git 会自动修改该仓库目录的组权限为可写。 注意，运行此命令的工程中不会摧毁任何提交、引用等内容。

```console
$ ssh user@git.example.com
$ cd /srv/git/my_project.git
$ git init --bare --shared
```