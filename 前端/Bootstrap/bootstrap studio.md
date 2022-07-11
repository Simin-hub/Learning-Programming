# Bootstrap Studio

## the stage

### stage Menu

![image-20220401201621133](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401201621133.png)

大纲网格 - 将显示行和列的边界。 

Show Box Model - 将在您悬停组件时显示组件的边距和填充。 

显示评论 - 将显示/隐藏您或您的队友留下的评论。 

额外的空白 - 启用时，将在页面底部添加额外的填充，以便更容易拖放组件。此额外空间仅在应用程序本身中添加，而不是在导出设计时添加

## component panel

### component Groups

UI group，某些组（例如 UI 组）包含更复杂的组件，这些组件具有内置样式、图像、占位符内容，有时还包含 JavaScript 代码。这些特殊组件非常适合快速构建常见的用户界面部分，如页眉、页脚、画廊等。

User Group，自定义组件。Bootstrap Studio 让您能够以用户组件的形式保存和重用 HTML、CSS 和 JS 片段。这非常适合您经常使用的内容，例如页眉和页脚，可以通过拖放将其包含在任何页面中。 要创建自定义组件，请右键单击页面上的元素并选择“添加到库”选项。

![image-20220401202748551](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401202748551.png)

选中自己需要保存的组件，右键，选择"Add to Library"。

## Options Panel

修改有关设计外观的所有内容

![Options Panel Tabs](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/option-panel-tabs.png)

1. **Appearance** , 保存智能、用户友好的输入，用于更改选定组件的 CSS 属性。
2. **Options**, 可以快速访问引导框架提供的所有设置
3. **Animations** , 定义在滚动、悬停或页面加载时触发的平滑 CSS 动画

### Appearance

在此选项卡中，您可以找到用于调整组件 CSS 的控件。所有更改都会实时反映在stage中，这意味着您可以看到更改属性如何反映在您的设计中。 大多数常用的 CSS 属性都有输入 - 从简单的内容（如填充和边框）到更复杂的 CSS（如变换和过滤器）。所有的输入都是可视化的并且易于使用。

### Options

根据所选组件，Bootstrap Studio 显示引导框架提供的所有设置。这包括文本选项、响应式显示、弹性框、边框和可访问性设置。

与直接使用 CSS 的外观面板相比，选项面板中的设置适用于 Bootstrap 的类。两者之间有一些重复，但建议尽可能使用此选项卡中的选项而不是外观中的选项

## Overview Panel

### Labels

在组件上设置标签。它们将显示在组件名称旁边的概览中。

可以在导出设置中激活导出标签选项。当它打开时，应用程序会将您的组件包装在注释中。

例如设置div 的label 是Avocado

```
<!-- Start: Avocado Card -->
<div>...</div>
<!-- End: Avocado Card -->
```

## Design Panel

设计面板是您可以找到所有设计资源的地方，例如页面、CSS、SASS、字体、图像和 JS 代码。

### importing HTML file

可以通过将外部创建的页面拖放到应用程序窗口来导入它们。对此的一个限制是外部创建的设计作为自定义代码导入并且只能作为 HTML 进行编辑。 Bootstrap Studio 将它们视为黑匣子，不会尝试解析它们。

## Editor Panel

###  Styles Tab

请注意，应用程序生成的样式（标记为“Bootstrap”）用户不可编辑。您可以通过单击块的三点菜单并将其复制到您自己的样式表来覆盖它们。

### JavaScript Tab

可以将漂亮的设计变成功能齐全的网站。您可以使用多个光标和键盘快捷键访问强大的类似 Sublime Text 的编辑。

## workflow

工作流程：

1. 引入文件（import），例如图片、css、js、page（只能作为 HTML 进行编辑）。
2. 导出页面
3. 。。。

### Linked Components

Bootstrap Studio 使您能够同步组件，以便它们一起更新。它称为链接组件，非常适合需要在多个页面上显示的页眉和页脚等元素。

#### Copy and Paste Method

要使用复制和粘贴方法创建链接组件：

1. 选择这个组件
2. 右键选择复制
3. 选择另一个组件，右键然后选择 `Copy & Paste > Paste Linked`

可以通过名称右侧的蓝色图标来识别链接的组件

#### Copy to Multiple Method

创建链接组件的另一种方法是右键单击组件并选择`Copy & Paste > Copy to > Multiple`

检查要将组件复制到的页面并选择链接副本选项。这将创建组件的链接副本并将它们传输到您选择的页面。

### Designing Responsive Pages

提供了许多工具、组件和技术，用于创建在任何设备上看起来都很棒的响应式设计

#### The Bootstrap Grid

![image-20220401225247276](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401225247276.png)

**Rows and Column**

在 Grid 组中，您可以找到 Row 和 Column 组件。这些是任何网格布局的主要构建块。

以下是基本概念：

-  网格由一个 Row 和其中的一个或多个 Column 组件组成。 
- 您可以将行放置在页面上的任何位置，包括列内部，从而创建嵌套网格。
-  Row 占据其父级的全部宽度。 列可以占据行的某个部分，从 1/12 一直到 12/12（占据行的整个宽度）。
-  列在行内的同一行上彼此相邻，但如果它们的宽度超过 12 部分，则不适合的第一列会换行。
-  对于每个引导网格断点，列可以具有不同的宽度。这使得可以根据手机、笔记本电脑和台式显示器等不同设备上的可用空间来调整布局。

**Column Toolbar**

![image-20220401230302075](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401230302075.png)

布局的大致轮廓准备好后，您可以开始调整列的大小和响应能力。选择一列后，将出现一个工具栏，其中包含用于更改列的顺序和大小的选项，以及用于快速向该行添加更多列的按钮。

**Options Panel**

要进行更精细的调整，请选择一列并转到“选项面板”。在这里，您可以对列的显示方式进行更多设置，并且可以针对特定断点调整每个选项。

**Conditional Visibility**

响应式显示选项提供了一种根据屏幕大小隐藏、显示或更改任何元素的显示类型的快速方法。您可以在选项面板的响应式显示组中找到它们。

**将列拆分为新行（清除修复）**

对于某些布局，您需要将列清除为单独的行。这就是 Bootstrap Studio 为您提供 Column Helper 组件的原因。您只需要在两列之间拖放它并为其提供正确的响应式显示类以限制它何时处于活动状态

**通过在css中实现媒体查询**

可以通过在 CSS 编辑器中编写代码来实现以上的任何目标。每个 CSS 块都可以应用媒体查询。

### Custom Code

是一个功能强大的组件，可让您自由地将其内容编辑为 HTML。 Bootstrap Studio 不会尝试以任何方式解析或验证代码，这为诸如嵌入应用程序本身不支持的服务器端语言或标签之类的事情打开了大门。

#### 基本使用

![image-20220401231703692](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401231703692.png)

将自定义代码组件从组件面板拖放到舞台。双击它打开它进行编辑。

![image-20220401232058476](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401232058476.png)

可以使用VScode、Sublime Text等编辑器

#### Converting Components to Custom Code（将组件转换为自定义代码）

如果您希望以 Bootstrap Studio 中无法实现的方式自定义组件或设计的一部分，您可以通过**将其转换为自定义代码来实现**。只需右键单击一个组件并选择转换为 HTML。请记住，**转换是一种方式；您将无法将修改转换回组件**。

### Custom Options

![image-20220401234234992](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220401234234992.png)

#### 自定义选项编辑器

![Custom Options Panel](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/custom-options-panel.png)

在这里您可以创建两种类型的选项： 

- toggle切换      在组件选项中提供一个可切换的开关。启用后，将向元素添加类名或属性，否则将被删除。 
- **Dropdown** 下拉列表 - 以下拉列表的形式显示可能的选项列表。通过选择其中一个选项，所选的类名或属性将被添加到元素中。

**toggles**

![](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/add-checkbox.png)

您可以自定义以下属性： 

- name 名称- 将在此选项的选项面板中显示的标签（默认为类/属性名称） 
- Tooltip  工具提示- 将在标签之后显示的工具提示 
- Action操作 - 选择该选项是否应在元素处于活动状态时为其添加类名或属性 
- class name  类名 - 将添加到元素的类名 
- **Attribute Name** 属性名称 - 将添加到元素的属性的名称 
- **Attribute Value** 属性值 - 将添加到元素的属性值

**Dropdowns**

![add-dropdown](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/add-dropdown.png)



您可以自定义以下属性： 

**Name**  名称 - 将在此选项的选项面板中显示的标签 

Tooltip - 将在标签之后显示的工具提示 

**Action**  操作 - 选择该选项是否应在元素处于活动状态时为其添加类名或属性 

**Attribute Name** 属性名称 - 将添加到元素的属性名称（如果操作是“添加属性”） 

**Removable** 可移除 - 这将在下拉列表中添加一个选项“无”，这将删除此选项设置的类名/属性 

**Options**  选项 - 将在下拉列表中显示的选项 

- label 标签 - 用户将在下拉列表中看到的名称 
-  **Class Name/Attribute Value** 类名/属性值 - 将添加到元素的类名/属性值