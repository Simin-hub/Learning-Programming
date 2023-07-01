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

### artifacts
使用 artifacts 指定在作业 succeeds, fails, 或 always 时附加到作业的文件和目录列表。

作业完成后，产物将发送到 GitLab。如果大小不大于最大产物大小，它们可以在 GitLab UI 中下载。

默认情况下，后期的作业会自动下载早期作业创建的所有产物。您可以使用 dependencies 控制作业中的产物下载行为。

使用 needs 关键字时，作业只能从 needs 配置中定义的作业下载产物。

默认只收集成功作业的作业产物，产物在缓存后恢复。

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

## 验证`.gitlab-ci.yml` 文件
使用 CI Lint 工具检查极狐GitLab CI/CD 配置的有效性。 您可以从 .gitlab-ci.yml 文件或任何其它示例 CI/CD 配置中验证语法。 该工具检查语法和逻辑错误，并且可以模拟流水线创建，尝试发现更复杂的配置问题。

## 预定义变量参考


