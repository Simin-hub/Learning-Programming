# Git

[参考](https://git-scm.com/book/zh/v2)

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