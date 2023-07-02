# .gitlab-ci.yml 文件
[参考](https://docs.gitlab.cn/)
`.gitlab-ci.yml` 文件是一个YAML 文件，您可以在其中配置 GitLab CI/CD 的特定指令

在此文件中，您定义：

*   runner 应执行的作业的结构和顺序
*   runner 在遇到特定条件时应做出的决定

例如，您可能希望在提交到除默认分支之外的任何分支时运行一组测试。当您提交到默认分支时，您希望运行相同的套件，但还要发布您的应用程序

在.gitlab-ci.yml文件中，可以定义：

* 要运行的脚本。
* 要包含的其他配置文件和模板。
* 依赖项和缓存。
* 要按顺序运行的命令和要并行运行的命令。
* 将应用程序部署到的位置。
* 无论您是想自动运行脚本还是手动触发它们中的任何一个

所有这些都 `.gitlab-ci.yml` 文件中定义

**实例**

```bash
build-job: ## 作业名称
  stage: build ## 定义作业的阶段
  image: node ## 从什么镜像创建
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"

test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20

deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```
`$GITLAB_USER_LOGIN` 和 `$CI_COMMIT_BRANCH` 是在作业运行时填充的预定义变量。

## Gitlab CI/CD pipeline 的配置包含：

[参考](https://docs.gitlab.com/16.1/ee/ci/yaml/index.html)

1. 使用全局 Keyword 来配置 pipeline 行为；
2. 使用 Job 特有 Keyword 来配置 Job；

### 配置 pipeline 行为的 全局 Keyword 有：

| Keyword                                                      | 描述                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| [default](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23default) | 定义Job 关键字的自定义默认值                            |
| [include](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23include) | 定义导入其它的 `*.yml` 文件来配置，本地和远程文件都可以 |
| [stages](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23stages) | 定义pipeline 中 stage 的名称和顺序                      |
| [variables](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23variables) | 定义 pipeline 中 所有 job 可以使用的全局变量            |
| [workflow](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23workflow) | 控制可以运行的 pipeline 类型                            |

### 配置 Job 使用的 Job Keyword 有：

| Keyword                                                      | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [after_script](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23after_script) | 定义Job 完成后执行的 script，执行失败的 Job 也可以执行 script |
| **[allow_failure](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23allow_failure)** | 定义是否允许 Job 失败，如是是 true，则 当前 Job 执行失败后，不会导致 pipeline 失败，会继续往后执行，否则 pipeline 终止 |
| **[artifacts](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23artifacts)** | 定义 Job 产物，也就是 Job 执行完成后（成功和失败都行），将哪些文件或目录添加到产物中或者构建结果中 |
| [before_script](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23before_script) | 定义Job 开始前执行的 script                                  |
| **[cache](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23cache)** | 定义 Job 之前缓存的文件或目录                                |
| [coverage](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23coverage) | 给定 Job 的代码覆盖率设置                                    |
| [dast_configuration](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23dast_configuration) | 在 Job 级别上使用DAST配置文件中的配置                        |
| [dependencies](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23dependencies) | 控制 Job 产物的下载行为，也就是从哪些 Job 获取产物；如果不定义，则前几个 stage 的所有产物都会传递给每个 Job |
| [environment](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23environment) | Job 部署到的环境的名称                                       |
| [except](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23except) | 使用 only 和 except 控制 Job 何时被添加到 pipeline，except 控制 Job 何时不运行；可以使用新 Keyword:rules 来替代 only 和 except |
| [only](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23only) | 使用 only 和 except 控制 Job 何时被添加到 pipeline，except 控制 Job 何时运行；可以使用新 Keyword:rules 来替代 only 和 except |
| **[rules](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23rules)** | 定义在 pipeline 中包含和不包含 Job 的规则，rules 与 only 和 except 的作用一样，所以不能同时出现 |
| [extends](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23extends) | 扩展某个 Job                                                 |
| [image](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23image) | 指定 Job 在其中运行的 Docker image                           |
| [inherit](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23inherit) | 定义继承的全局设置的默认变量                                 |
| [interruptible](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23interruptible) | 定义在 Job 执行完成前启动了新 pipeline 的时候，是否需要取消 Job，ture 取消，否则不取消 |
| [needs](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23needs) | 该关键字，可以让 Job 不需要按照 Stage 顺序来执行，可以不等待，按照需要来执行 |
| [pages](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23pages) | 定义一个 将静态内容上传到 GitLab 的 GitLab pages Job，然后会将内容作为网址发布 |
| [parallel](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23parallel) | 单个 pipeline 中并以并行的方式运行 Job 多次                  |
| [release](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23release) | 让 Runner 生成 release                                       |
| [resource_group](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23resource_group) | 限制 Job 并发运行                                            |
| [retry](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23retry) | Job 执行失败，重试的次数，默认为0                            |
| [script](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23script) | Job 执行的脚本                                               |
| [secrets](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23secrets) | 指定 CI/CD secrets                                           |
| [services](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23services) | 指定额外的 Docker image 来运行 script                        |
| [stage](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23stage) | 定义 Job 所处的 stage                                        |
| [tags](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23tags) | 通过 tag 来选择 Runner 运行，Runner 建立的时候会创建 tag 列表 |
| [timeout](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23timeout) | Job 执行的超时时间                                           |
| [trigger](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23trigger) | 定义下游 pipeline 触发器                                     |
| [variables](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23variables) | 定义 Job 可以使用的局部变量                                  |
| [when](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fci%2Fyaml%2F%23variables) | 何时运行 Job 的条件                                          |



在 全局 Keyword 中的 `default` 可以 用来配置所有 Job 需要执行通用配置，它支持的 Job 配置有：

- `after_script`
- `artifacts`
- `before_script`
- `cache`
- `image`
- `interruptible`
- `retry`
- `services`
- `tags`
- `timeout`

## 关键字

### stages

使用 stages 来定义包含作业组的阶段。stages 是为流水线全局定义的。在作业中使用 stage 来定义作业属于哪个阶段。

如果 .gitlab-ci.yml 文件中没有定义 stages，那么默认的流水线阶段是：

* .pre
* build
* test
* deploy
* .post

stages 项的顺序定义了作业的执行顺序：

* 同一阶段的作业并行运行。
* 下一阶段的作业在上一阶段的作业成功完成后运行。

```
stages:
  - build
  - test
  - deploy
```
* build 中的所有作业并行执行。
* 如果 build 中的所有作业都成功，test 作业将并行执行。
* 如果 test 中的所有作业都成功，deploy 作业将并行执行。
* 如果 deploy 中的所有作业都成功，则流水线被标记为 passed。

如果任何作业失败，流水线将被标记为 failed 并且后续阶段的作业不会启动。**当前阶段的作业不会停止并继续运行**。

如果作业未指定 stage，则作业被分配到 test 阶段。

如果定义了一个阶段，但没有作业使用它，则该阶段在流水线中不可见。 这对合规流水线配置很有用，因为：

* 阶段可以在合规性配置中定义，但如果不使用则保持隐藏。
* 当开发人员在作业定义中使用它们时，定义的阶段变得可见。

要使作业更早开始并忽略阶段顺序，请使用 needs 关键字。

### image
使用 image 指定运行作业的 Docker 镜像。

关键字类型：作业关键字。您只能将其用作作业的一部分或在 default: 部分 中使用。

可能的输入：镜像的名称，包括镜像库路径（如果需要），采用以下格式之一：

* `image-name`（与使用带有 latest 标签的 <image-name> 相同）
* `image-name:tag`
* `image-name@digest`

### services

`services`指定任何你的脚本需要成功运行的**额外Docker镜像**。`services` 镜像链接到 [`image`](https://docs.gitlab.cn/jh/ci/yaml/#image) 关键字中指定的镜像。例如需要其他的数据库容器。

**关键字类型**：作业关键字。您只能将其用作作业的一部分或在 [`default:` 部分](https://docs.gitlab.cn/jh/ci/yaml/#default) 中使用。

**可能的输入**：服务镜像的名称，包括镜像库路径（如果需要），采用以下格式之一：

- `<image-name>`（与使用带有 `latest` 标签的 `<image-name>` 相同）
- `<image-name>:<tag>`
- `<image-name>@<digest>`

### script

使用 script 指定 runner 要执行的命令。

除了 trigger jobs 之外的所有作业都需要一个 script 关键字。

关键字类型：作业关键字。您只能将其用作作业的一部分。

可能的输入：一个数组，包括：

* 单行命令。
* 长命令拆分多行。
* YAML 锚点。

### rules
使用 rules 来包含或排除流水线中的作业。

创建流水线时会评估规则，并按顺序评估，直到第一次匹配。找到匹配项后，该作业将包含在流水线中或从流水线中排除，具体取决于配置。

您不能在规则中使用作业脚本中创建的 dotenv 变量，因为在任何作业运行之前都会评估规则。

rules 接受以下规则：

* if
* changes
* exists
* allow_failure
* variables
* when

您可以为复杂规则，将多个关键字组合在一起。

作业被添加到流水线中：

* 如果 if、changes 或 exists 规则匹配并且还具有 when: on_success（默认）、when: delay 或 when: always。
* 如果达到的规则只有 when: on_success、when: delay 或 when: always。

作业未添加到流水线中：

* 如果没有规则匹配。
* 如果规则匹配并且有 when: never。
  

您可以在不同的工作中使用 !reference 标签 来重用 rules 配置。

### tags

使用 tags 从项目可用的所有 runner 列表中选择一个特定的 runner。

注册 runner 时，您可以指定 runner 的标签，例如 ruby、postgres 或 development。要获取并运行作业，必须为 runner 分配作业中列出的每个标签。

关键字类型：作业关键字。您只能将其用作作业的一部分或在 default: 部分 中使用。

可能的输入：
* 标签名称数组。
* 14.1 及更高版本中的 CI/CD 变量。

### allow_failure
使用 allow_failure 来确定当作业失败时，流水线是否应该继续运行。

* 要让流水线继续运行后续作业，请使用 allow_failure: true。
* 要停止流水线运行后续作业，请使用 allow_failure: false。
当允许作业失败 (allow_failure: true) 时，橙色警告 () 表示作业失败。但是，流水线是成功的，并且关联的提交被标记为已通过且没有警告。

在以下情况下会显示相同的警告：

* 阶段中的所有其他工作均成功。
* 流水线中的所有其他作业均成功。

allow_failure 的默认值为：

* 手动作业为 true。
* 对于在 rules 中使用 when:manual 的作业为 false。
* 在所有其它情况下为 false。

关键字类型：作业关键字。您只能将其用作作业的一部分。
可能的输入：true 或 false。

### when
使用 when 配置作业运行的条件。如果未在作业中定义，则默认值为 when: on_success。

关键字类型：作业关键字。您可以将其用作作业的一部分。when: always 和 when: never 也可以在 workflow:rules 中使用。

可能的输入：

* on_success（默认）：仅当早期阶段的所有作业都成功或具有 allow_failure: true 时才运行作业。
* manual：仅在手动触发时运行作业。
* always：无论早期阶段的作业状态如何，都运行作业。也可以在 workflow:rules 中使用。
* on_failure：只有在早期阶段至少有一个作业失败时才运行作业。带有 allow_failure: true 的作业总是被认为是成功的。
* delayed：作业的执行延迟指定的持续时间。
* never：不要运行作业。只能在 rules 部分或 workflow: rules 中使用。

### environment
使用 environment 定义作业部署到的环境。

关键字类型：作业关键字。您只能将其用作作业的一部分。

可能的输入：作业部署到的环境的名称，采用以下格式之一：

* 纯文本，包括字母、数字、空格和以下字符：-、_、/、$、{、}。
* CI/CD 变量，包括在.gitlab-ci.yml 文件中定义的预定义、安全或变量。您不能使用在 script 部分中定义的变量。



### cache

cache 是 Job 下载并保存的一个或多个文件。使用相同 cahce 的后续 Job 不必再次下载文件，因此执行速度更快。

cache：

- 使用 `cache` 来定义每个 Job 的缓存；
- 后续的 Pipeline 可以使用缓存；
- 如果依赖相同的，同一个 Pipeline 的后续 Job 可以使用缓存；
- 不同的项目不能共用缓存。

#### cache: paths

使用`paths`指令选择要缓存的文件或目录。也可以使用通配符。

#### cache: key

`key`指令允许我们定义缓存的作用域(亲和性)，可以是所有 jobs 的单个缓存，也可以是每个 job，也可以是每个分支或者是任何你认为合适的地方。它也可以让你很好的调整缓存，允许你设置不同 jobs 的缓存，甚至是不同分支的缓存。

`cache:key`可以使用任何的[预定义变量](https://docs.gitlab.com/ce/ci/variables/README.html)。给每个缓存一个唯一的识别密钥。所有使用相同缓存密钥的作业都使用相同的缓存，包括在不同管道中。

如果没有设置，默认键为default。所有带有cache关键字但没有cache:key的作业都共享默认缓存。

### artifacts

使用 artifacts 指定在作业 succeeds, fails, 或 always 时附加到作业的文件和目录列表。

作业完成后，产物将发送到 GitLab。如果大小不大于最大产物大小，它们可以在 GitLab UI 中下载。

默认情况下，后期的作业会自动下载早期作业创建的所有产物。您可以使用 dependencies 控制作业中的产物下载行为。

使用 needs 关键字时，作业只能从 needs 配置中定义的作业下载产物。

默认只收集成功作业的作业产物，产物在缓存后恢复。

### cache 和 artifacts 的区别：

- 对依赖项使用 cache，比如从网络上下载的包。缓存存储在安装 GitLab Runner的地方，如果启用了分布式缓存，则将其安装并上传载到S3；
- **使用 artifacts 在 stage 之间传递中间构建结果。artifacts 由 Job 生成，存储在GitLab中，可以下载**；
- artifacts 和 cache 都定义了它们相对于项目目录的路径，并且不能链接到项目目录之外的文件。

### coverage

`coverage`使用覆盖率与自定义正则表达式来配置如何从作业输出中提取代码覆盖率。如果作业输出中至少有一行与正则表达式相匹配，覆盖率就会显示在用户界面上。

为了从匹配中提取代码覆盖率值，GitLab使用这个小的正则表达式：`\d+（\.\d+）?`



### release

使用 release 创建一个发布。

发布作业必须有权访问 release-cli，其必须在 $PATH 中。

关键字类型：作业关键字。您只能将其用作作业的一部分。

可能的输入：release: 子键：
* tag_name
* description
* tag_message（可选）
* name（可选）
* ref（可选）
* milestones（可选）
* released_at（可选）
* assets:links（可选）

### pages
使用 pages 定义一个 GitLab Pages 作业，将静态内容上传到极狐GitLab，然后将内容发布为网站。

关键字类型：作业名称。


### variables
使用 variables 为作业定义自定义变量。

变量在 script、before_script 和 after_script 命令中始终可用。 您还可以在某些作业关键字中使用变量作为输入。

关键字类型：全局和工作关键字。您可以在全局级别使用它，也可以在作业级别使用它。

如果您将 variables 定义为全局关键字，它的行为类似于所有作业的默认变量。创建流水线时，每个变量都会复制到每个作业配置。 如果作业已经定义了该变量，则作业级别变量优先。

在全局级别定义的变量不能用作其他全局关键字的输入，例如 include。这些变量只能在作业级别使用，在 script、before_script 和 after_script 部分，以及一些作业关键字中的输入，例如 rules。

可能的输入：变量名和值对：

* 名称只能使用数字、字母和下划线 (_)。
* 值必须是字符串。

### workflow

控制pipeline行为。



## 验证`.gitlab-ci.yml` 文件

使用 CI Lint 工具检查极狐GitLab CI/CD 配置的有效性。 您可以从 .gitlab-ci.yml 文件或任何其它示例 CI/CD 配置中验证语法。 该工具检查语法和逻辑错误，并且可以模拟流水线创建，尝试发现更复杂的配置问题。

## 预定义变量参考

## Docker 运行ＣＩ／ＣＤ

[参考](https://docs.gitlab.com/ee/ci/docker/using_docker_images.html)、[参考２](https://blog.csdn.net/Sherlock1020/article/details/126303270)、

将 [Docker](https://www.docker.com/) 合并到 CI/CD 工作流程中有两种主要方法：

- **在 Docker 容器中[运行CI/CD 作业](https://docs.gitlab.cn/jh/ci/docker/using_docker_images.html)。**

  可以创建 CI/CD 作业来执行测试、构建或发布应用程序等操作。这些作业可以在 Docker 容器中运行。

  例如，您可以告诉 GitLab CI/CD 使用托管在 Docker Hub 或 GitLab 容器镜像库中的节点镜像。然后，您的作业在基于镜像的容器中运行。该容器具有构建应用程序所需的所有 Node 依赖项。

- **使用 [Docker](https://docs.gitlab.cn/jh/ci/docker/using_docker_build.html) 或 [kaniko](https://docs.gitlab.cn/jh/ci/docker/using_kaniko.html) 构建 Docker 镜像。**

  您可以创建 CI/CD 作业来构建 Docker 镜像并将它们发布到容器镜像库。

### 在Docker容器中运行CI/CD工作

####　注册一个使用Docker executor 器的runner

要将 GitLab Runner 与 Docker 一起使用，您需要注册一个使用 Docker executor 的 runner。

#### 脚本在哪里执行

当 CI 作业在 Docker 容器中运行时，`before_script`、`script` 和 `after_script` 命令在 `/builds/<project-path>/` 目录中运行。 您的镜像可能定义了不同的默认 `WORKDIR`。要移动到您的 `WORKDIR`，请将 `WORKDIR` 保存为环境变量，以便您可以在作业运行时在容器中引用它。
