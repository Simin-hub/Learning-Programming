# CI/CD



## GitLab CI/CD

GitLab CI/CD 是一个内置在GitLab中的工具，用于通过持续方法进行软件开发。GitLab CI/CD 由一个名为 .gitlab-ci.yml 的文件进行配置，改文件位于仓库的根目录下。文件中指定的脚本由GitLab Runner执行。

### 工作原理

为了使用GitLab CI/CD，你需要一个托管在GitLab上的应用程序代码库，并且在根目录中的.gitlab-ci.yml文件中指定构建、测试和部署的脚本。

在这个文件中，你可以定义要运行的脚本，定义包含的依赖项，选择要按顺序运行的命令和要并行运行的命令，定义要在何处部署应用程序，以及指定是否 要自动运行脚本或手动触发脚本。 

为了可视化处理过程，假设添加到配置文件中的所有脚本与在计算机的终端上运行的命令相同。

一旦你已经添加了.gitlab-ci.yml到仓库中，GitLab将检测到该文件，并使用名为GitLab Runner的工具运行你的脚本。该工具的操作与终端类似。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/874963-20200203191024392-1340530589.png)

1. Verify

   - 通过持续集成自动构建和测试你的应用程序

   - 使用GitLab代码质量（GitLab Code Quality）分析你的源代码质量

   - 通过浏览器性能测试（Browser Performance Testing）确定代码更改对性能的影响

   - 执行一系列测试，比如Container Scanning , Dependency Scanning , JUnit tests

   - 用Review Apps部署更改，以预览每个分支上的应用程序更改 

2. Package

   - 用Container Registry存储Docker镜像

   - 用NPM Registry存储NPM包

   - 用Maven Repository存储Maven artifacts

   - 用Conan Repository存储Conan包 

3. Release

   - 持续部署，自动将你的应用程序部署到生产环境

   - 持续交付，手动点击以将你的应用程序部署到生产环境

   - 用GitLab Pages部署静态网站

   - 仅将功能部署到一个Pod上，并让一定比例的用户群通过Canary Deployments访问临时部署的功能（PS：即灰度发布）

   - 在Feature Flags之后部署功能

   - 用GitLab Releases将发布说明添加到任意Git tag

   - 使用Deploy Boards查看在Kubernetes上运行的每个CI环境的当前运行状况和状态

   - 使用Auto Deploy将应用程序部署到Kubernetes集群中的生产环境 

使用GitLab CI/CD，还可以：

- 通过Auto DevOps轻松设置应用的整个生命周期
- 将应用程序部署到不同的环境
- 安装你自己的GitLab Runner
- Schedule pipelines
- 使用安全测试报告（Security Test reports）检查应用程序漏洞 