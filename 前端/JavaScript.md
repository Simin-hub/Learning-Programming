# JavaScript

[参考](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/01-JavaScript%E7%AE%80%E4%BB%8B.html#%E6%88%91%E7%9A%84%E5%85%AC%E4%BC%97%E5%8F%B7)

## 基础

### JavaScript 的组成

JavaScript 基础分为三个部分：

- **ECMAScript**：JavaScript 的**语法标准**。包括变量、表达式、运算符、函数、if 语句、for 语句等。
- **DOM**：Document Object Model（文档对象模型），JS 操作**页面上的元素**（标签）的 API。比如让盒子移动、变色、改变大小、轮播图等等。
- **BOM**：Browser Object Model（浏览器对象模型），JS 操作**浏览器部分功能**的 API。通过 BOM 可以操作浏览器窗口，比如弹框、控制浏览器跳转、获取浏览器分辨率等等。

通俗理解就是：ECMAScript 是 JS 的语法；DOM 和 BOM 是浏览器运行环境为 JS 提供的 API。

### JavaScript 的特点

#### 特点 1：解释型语言

JavaScript 简称 JS，是前端开发的一门脚本语言（解释型语言）。

**解释型语言**的意思是：程序执行之前，不需要事先被翻译为机器码；而是**在运行时，边翻译边执行**（翻译一行，执行一行）。关于解释型语言的详细介绍。

为什么 JS 是解释型语言呢？这和浏览器的工作原理有关。浏览器中有一个专门的“JS 解析器”可以让 JS 边解析边执行。

由于少了事先编译这一步骤，所以解释型语言开发起来尤为方便，但是解释型语言运行较慢也是它的劣势。不过解释型语言中使用了 JIT 技术，使得运行速度得以改善。

#### 特点 2：单线程

#### 特点 3：ECMAScript 标准

ECMAScript 是一种由 ECMA 国际（前身为欧洲计算机制造商协会,英文名称是 European Computer Manufacturers Association）制定和发布的脚本语言规范。

JavaScript 是由公司开发而成的，问题是不便于其他的公司拓展和使用。所以欧洲的这个 ECMA 的组织，牵头制定 JavaScript 的标准，取名为 ECMAScript。

简单来说，**ECMAScript 不是一门语言，而是一个标准**。ECMAScript 规定了 JS 的编程语法和基础核心知识，是所有浏览器厂商共同遵守的一套 JS 语法工业标准。

ECMAScript 在 2015 年 6 月，发布了 ECMAScript 6 版本（ES6），语言的能力更强，包含了很多新特性），但也要考虑它的浏览器兼容性问题。

ECMA 赋予了 JavaScript 新的能力和活力。

### 引入 JS 代码

引入 JS 代码，有哪几种方式？有三种方式：（和 CSS 的引入方式类似）

1. **行内式**：写在标签内部。
2. **内嵌式**（内联式）：写在 head 标签中。
3. **外链式**：引入外部 JS 文件

#### JavaScript 一些简单的语法规则

学习程序，是有规律可循的，程序会有有相同的部分，这些部分就是一种规定，不能更改，我们称之为：语法。我们先来了解一个简单的语法规则。

1、JS 对换行、缩进、空格不敏感。每一条语句以分号结尾。

2、每一条语句末尾要加上**分号**。虽然分号不是必须加的，但如果不写分号，浏览器会自动添加分号，导致消耗一些系统资源和性能，甚至有可能**添加错误**。

3、所有的符号，都是英文的。比如**括号**、引号、分号。

如果你用的是搜狗拼音，**建议不要用 shift 切换中英文**（可以在搜狗软件里进行设置），不然很容易输入中文的分号；建议用 ctrl+space 切换中英文输入法。

4、JS 严格区分大小写。

### 事件

#### 事件简介

事件：就是文档或浏览器窗口中发生的一些特定的交互瞬间。对于 Web 应用来说，有下面这些代表性的事件：点击某个元素、将鼠标移动至某个元素上方、关闭弹窗等等。

JavaScript 是以**事件驱动为核心**的一门语言。JavaScript 与 HTML 之间的交互是通过事件实现的。

#### 事件的三要素

**事件的三要素：事件源、事件、事件驱动程序**。

比如，我用手去按开关，灯亮了。这件事情里，事件源是：手。事件是：按开关。事件驱动程序是：灯开了或者关了。

再比如，网页上弹出一个广告，我点击右上角的`X`，广告就关闭了。这件事情里，事件源是：`X`。事件是：onclick。事件驱动程序是：广告关闭了。

于是我们可以总结出：谁引发的后续事件，谁就是事件源。

**总结如下：**

- 事件源：引发后续事件的html标签。
- 事件：js已经定义好了（见下图）。
- 事件驱动程序：对样式和html的操作。也就是DOM。

也就是说，我们可以在时间对应的属性中写一些js代码，当事件被触发时，这些代码将会执行。

**代码书写步骤如下：**（重要）

（1）获取事件源：document.getElementById(“box”); // 类似于Android里面的findViewById

（2）绑定事件： 事件源box.事件onclick = function(){ 事件驱动程序 };

（3）书写事件驱动程序：关于DOM的操作。

最简单的代码举例：（点击box1，然后弹框）

```html
<body>
<div id="box1"></div>

<script type="text/javascript">
    // 1、获取事件源
    var div = document.getElementById("box1");
    // 2、绑定事件
    div.onclick = function () {
        // 3、书写事件驱动程序
        alert("我是弹出的内容");
    }
</script>

</body>
```

常见的事件如下：

![img](http://img.smyhvae.com/20180126_1553.png)

##### 1、获取事件源的方式（DOM节点的获取）

```
var div1 = document.getElementById("box1");      //方式一：通过id获取单个标签

var arr1 = document.getElementsByTagName("div");     //方式二：通过 标签名 获得 标签数组，所以有s

var arr2 = document.getElementsByClassName("hehe");  //方式三：通过 类名 获得 标签数组，所以有s
```

##### 2、绑定事件的方式

方式一：直接绑定匿名函数

```html
<div id="box1" ></div>

<script type="text/javascript">
    var div1 = document.getElementById("box1");
    //绑定事件的第一种方式
    div1.onclick = function () {
        alert("我是弹出的内容");
    }
</script>
```

方式二：先单独定义函数，再绑定

```html
 <div id="box1" ></div>

<script type="text/javascript">
    var div1 = document.getElementById("box1");
    //绑定事件的第二种方式
    div1.onclick = fn;   //注意，这里是fn，不是fn()。fn()指的是返回值。
    //单独定义函数
    function fn() {
        alert("我是弹出的内容");
    }
</script>
```

注意上方代码的注释。**绑定的时候，是写fn，不是写fn()**。fn代表的是整个函数，而fn()代表的是返回值。

方式三：行内绑定

```html
<!--行内绑定-->
<div id="box1" onclick="fn()"></div>

<script type="text/javascript">

    function fn() {
        alert("我是弹出的内容");
    }

</script>
```

注意第一行代码，绑定时，是写的`"fn()"`，不是写的`"fn"`。因为绑定的这段代码不是写在js代码里的，而是被识别成了**字符串**。

##### 3、事件驱动程序

我们在上面是拿alert举例，不仅如此，我们还可以操作标签的属性和样式。举例如下：

点击鼠标时，原本粉色的div变大了，背景变红：

```html
    <style>
        #box1 {
            width: 100px;
            height: 100px;
            background-color: pink;
            cursor: pointer;
        }
    </style>
</head>

<body>

<div id="box1" ></div>

<script type="text/javascript">
    var div1 = document.getElementById("box1");
    //点击鼠标时，原本粉色的div变大了，背景变红了
    div1.onclick = function () {
        div1.style.width = "200px";   //属性值要写引号
        div1.style.height = "200px";
        div1.style.backgroundColor = "red";   //属性名是backgroundColor，不是background-color
    }
</script>
```

上方代码的注意事项：

- 在js里写属性值时，要用引号
- 在js里写属性名时，是`backgroundColor`，不是CSS里面的`background-color`。

实现效果如下：

![img](http://img.smyhvae.com/20180126_1720.gif)

### DOM

JavaScript基础分为三个部分：

- ECMAScript：JavaScript的语法标准。包括变量、表达式、运算符、函数、if语句、for语句等。
- **DOM**：文档对象模型（Document object Model），操作**网页上的元素**的API。比如让盒子移动、变色、轮播图等。
- **BOM**：浏览器对象模型（Browser Object Model），操作**浏览器部分功能**的API。比如让浏览器自动滚动。

#### 节点

**节点**（Node）：**构成 HTML 网页的最基本单元**。网页中的每一个部分都可以称为是一个节点，比如：html标签、属性、文本、注释、整个文档等都是一个节点。

虽然都是节点，但是实际上他们的具体类型是不同的。常见节点分为四类：

- 文档节点（文档）：整个 HTML 文档。整个 HTML 文档就是一个文档节点。
- 元素节点（标签）：HTML标签。
- 属性节点（属性）：元素的属性。
- 文本节点（文本）：HTML标签中的文本内容（包括标签之间的空格、换行）。

节点的类型不同，属性和方法也都不尽相同。所有的节点都是Object。

#### 什么是DOM

**DOM**：Document Object Model，文档对象模型。DOM 为文档提供了结构化表示，并定义了如何通过脚本来访问文档结构。目的其实就是为了能让js操作html元素而制定的一个规范。

DOM就是由节点组成的。

**解析过程**： HTML加载完毕，渲染引擎会在内存中把HTML文档，生成一个DOM树，getElementById是获取内中DOM上的元素节点。然后操作的时候修改的是该元素的**属性**。

**DOM树**：（一切都是节点）

DOM的数据结构如下：

![img](http://img.smyhvae.com/20180126_2105.png)

上图可知，**在HTML当中，一切都是节点**（非常重要）。节点的分类，在上一段中，已经讲了。

整个html文档就是一个文档节点。所有的节点都是Object。

#### DOM可以做什么

- 找对象（元素节点）
- 设置元素的属性值
- 设置元素的样式
- 动态创建和删除元素
- 事件的触发响应：事件源、事件、事件的驱动程序

#### 元素节点的获取

DOM节点的获取方式其实就是**获取事件源的方式**。关于事件，上一篇文章中已经讲到了。

想要操作元素节点，必须首先要找到该节点。有三种方式可以获取DOM节点：

```javascript
var div1 = document.getElementById("box1"); //方式一：通过 id 获取 一个 元素节点（为什么是一个呢？因为 id 是唯一的）

var arr1 = document.getElementsByTagName("div"); //方式二：通过 标签名 获取 元素节点数组，所以有s

var arr2 = document.getElementsByClassName("hehe"); //方式三：通过 类名 获取 元素节点数组，所以有s
```

既然方式二、方式三获取的是标签数组，那么习惯性是**先遍历之后再使用**。

特殊情况：数组中的值只有1个。即便如此，这一个值也是包在数组里的。这个值的获取方式如下：

```javascript
document.getElementsByTagName("div1")[0];    //取数组中的第一个元素

document.getElementsByClassName("hehe")[0];  //取数组中的第一个元素
```

#### DOM访问关系的获取

DOM的节点并不是孤立的，因此可以通过DOM节点之间的相对关系对它们进行访问。如下：

![img](http://img.smyhvae.com/20180126_2140.png)

节点的访问关系，是以**属性**的方式存在的。

JS中的**父子兄**访问关系：

![img](http://img.smyhvae.com/20180126_2145.png)

这里我们要重点知道**parentNode**和**children**这两个属性的用法。下面分别介绍。

##### 获取父节点

调用者就是节点。一个节点只有一个父节点，调用方式就是

```javascript
	节点.parentNode
```

##### 获取兄弟节点

**1、下一个节点 | 下一个元素节点**：

> Sibling的中文是**兄弟**。

（1）nextSibling：

- 火狐、谷歌、IE9+版本：都指的是下一个节点（包括标签、空文档和换行节点）。
- IE678版本：指下一个元素节点（标签）。

（2）nextElementSibling：

- 火狐、谷歌、IE9+版本：都指的是下一个元素节点（标签）。

**总结**：为了获取下一个**元素节点**，我们可以这样做：在IE678中用nextSibling，在火狐谷歌IE9+以后用nextElementSibling，于是，综合这两个属性，可以这样写：

```javascript
	下一个兄弟节点 = 节点.nextElementSibling || 节点.nextSibling
```

**2、前一个节点 | 前一个元素节点**：

> previous的中文是：前一个。

（1）previousSibling：

- 火狐、谷歌、IE9+版本：都指的是前一个节点（包括标签、空文档和换行节点）。
- IE678版本：指前一个元素节点（标签）。

（2）previousElementSibling：

- 火狐、谷歌、IE9+版本：都指的是前一个元素节点（标签）。

**总结**：为了获取前一个**元素节点**，我们可以这样做：在IE678中用previousSibling，在火狐谷歌IE9+以后用previousElementSibling，于是，综合这两个属性，可以这样写：

```javascript
	前一个兄弟节点 = 节点.previousElementSibling || 节点.previousSibling
```

**3、补充**：获得任意一个兄弟节点：

```javascript
	节点自己.parentNode.children[index];  //随意得到兄弟节点
```

##### 获取单个的子节点

**1、第一个子节点 | 第一个子元素节点**：

（1）firstChild：

- 火狐、谷歌、IE9+版本：都指的是第一个子节点（包括标签、空文档和换行节点）。
- IE678版本：指第一个子元素节点（标签）。

（2）firstElementChild：

- 火狐、谷歌、IE9+版本：都指的是第一个子元素节点（标签）。

**总结**：为了获取第一个**子元素节点**，我们可以这样做：在IE678中用firstChild，在火狐谷歌IE9+以后用firstElementChild，于是，综合这两个属性，可以这样写：

```javascript
	第一个子元素节点 = 节点.firstElementChild || 节点.firstChild
```

**2、最后一个子节点 | 最后一个子元素节点**：

（1）lastChild：

- 火狐、谷歌、IE9+版本：都指的是最后一个子节点（包括标签、空文档和换行节点）。
- IE678版本：指最后一个子元素节点（标签）。

（2）lastElementChild：

- 火狐、谷歌、IE9+版本：都指的是最后一个子元素节点（标签）。

**总结**：为了获取最后一个**子元素节点**，我们可以这样做：在IE678中用lastChild，在火狐谷歌IE9+以后用lastElementChild，于是，综合这两个属性，可以这样写：

```javascript
	最后一个子元素节点 = 节点.lastElementChild || 节点.lastChild
```

##### 获取所有的子节点

（1）**childNodes**：标准属性。返回的是指定元素的**子节点**的集合（包括元素节点、所有属性、文本节点）。是W3C的亲儿子。

- 火狐 谷歌等高本版会把换行也看做是子节点。

用法：

```javascript
	子节点数组 = 父节点.childNodes;   //获取所有节点。
```

（2）**children**：非标准属性。返回的是指定元素的**子元素节点**的集合。【重要】

- 它只返回HTML节点，甚至不返回文本节点。
- 在IE6/7/8中包含注释节点（在IE678中，注释节点不要写在里面）。

虽然不是标准的DOM属性，但它和innerHTML方法一样，得到了几乎所有浏览器的支持。

用法：（**用的最多**）

```javascript
	子节点数组 = 父节点.children;   //获取所有节点。用的最多。
```

#### DOM节点的操作（重要）

上一段的内容：节点的**访问关系**都是**属性**。

本段的内容：节点的**操作**都是**函数**（方法）。

##### 创建节点

格式如下：

```javascript
	新的标签(元素节点) = document.createElement("标签名");
```

比如，如果我们想创建一个li标签，或者是创建一个不存在的adbc标签，可以这样做：

```html
<script type="text/javascript">
    var a1 = document.createElement("li");   //创建一个li标签
    var a2 = document.createElement("adbc");   //创建一个不存在的标签

    console.log(a1);
    console.log(a2);

    console.log(typeof a1);
    console.log(typeof a2);
</script>
```

打印结果：

![img](http://img.smyhvae.com/20180127_1135.png)

##### 插入节点

插入节点有两种方式，它们的含义是不同的。

方式1：

```javascript
	父节点.appendChild(新的子节点);
```

解释：父节点的最后插入一个新的子节点。

方式2：

```javascript
	父节点.insertBefore(新的子节点,作为参考的子节点)
```

解释：

- 在参考节点前插入一个新的节点。
- 如果参考节点为null，那么他将在父节点里面的最后插入一个子节点。

![img](http://img.smyhvae.com/20180127_1257.png)

我们可以看到，li标签确实被插入到了box1标签的里面，和box2并列了。

方式2的举例：

![img](http://img.smyhvae.com/20180127_1302.png)

我们可以看到，b1标签被插入到了box1标签的里面，和a1标签并列，在a1标签的前面。

**特别强调：**

关于方式1的appendChild方法，这里要强调一下。比如，现在有下面这样一个div结构：

```html
<div class="box11">
    <div class="box12">生命壹号</div>
</div>

<div class="box21">
    <div class="box22">永不止步</div>

</div>
```

上方结构中，子盒子box12是在父亲box11里的，子盒子box22是在父亲box21里面的。现在，如果我调用方法`box11.appendChild(box22)`，**最后产生的结果是：box22会跑到box11中**（也就是说，box22不在box21里面了）。这是一个很神奇的事情：

![img](http://img.smyhvae.com/20180129_2125.png)

##### 删除节点

格式如下：

```javascript
	父节点.removeChild(子节点);
```

解释：**用父节点删除子节点**。必须要指定是删除哪个子节点。

如果我想删除自己这个节点，可以这么做：

```javascript
	node1.parentNode.removeChild(node1);
```

##### 复制节点（克隆节点）

格式如下：

```javascript
	要复制的节点.cloneNode();       //括号里不带参数和带参数false，效果是一样的。

	要复制的节点.cloneNode(true);
```

括号里带不带参数，效果是不同的。解释如下：

- 不带参数/带参数false：只复制节点本身，不复制子节点。
- 带参数true：既复制节点本身，也复制其所有的子节点。

#### 设置节点的属性

我们可以获取节点的属性值、设置节点的属性值、删除节点的属性。

我们就统一拿下面这个标签来举例：

```html
	<img src="images/1.jpg" class="image-box" title="美女图片" alt="地铁一瞥" id="a1">
```

下面分别介绍。

##### 1、获取节点的属性值

**方式1**：

```javascript
	元素节点.属性名;
	元素节点[属性名];
```

举例：（获取节点的属性值）

```html
<body>
<img src="images/1.jpg" class="image-box" title="美女图片" alt="地铁一瞥" id="a1">

<script type="text/javascript">
    var myNode = document.getElementsByTagName("img")[0];

    console.log(myNode.src);
    console.log(myNode.className);    //注意，是className，不是class
    console.log(myNode.title);

    console.log("------------");

    console.log(myNode["src"]);
    console.log(myNode["className"]); //注意，是className，不是class
    console.log(myNode["title"]);
</script>
</body>
```

上方代码中的img标签，有各种属性，我们可以逐一获取，打印结果如下：

![img](http://img.smyhvae.com/20180127_1340.png)

**方式2**：

```javascript
	元素节点.getAttribute("属性名称");
```

举例：

```javascript
    console.log(myNode.getAttribute("src"));
    console.log(myNode.getAttribute("class"));   //注意是class，不是className
    console.log(myNode.getAttribute("title"));
```

打印结果：

![img](http://img.smyhvae.com/20180127_1345.png)

方式1和方式2的区别在于：前者是直接操作标签，后者是把标签作为DOM节点。推荐方式2。

##### 2、设置节点的属性值

方式1举例：（设置节点的属性值）

```javascript
    myNode.src = "images/2.jpg"   //修改src的属性值
    myNode.className = "image2-box";  //修改class的name
```

方式2：

```javascript
	元素节点.setAttribute("属性名", "新的属性值");
```

方式2举例：（设置节点的属性值）

```javascript
    myNode.setAttribute("src","images/3.jpg");
    myNode.setAttribute("class","image3-box");
    myNode.setAttribute("id","你好");
```

##### 3、删除节点的属性

格式：

```javascript
	元素节点.removeAttribute(属性名);
```

举例：（删除节点的属性）

```javascript
    myNode.removeAttribute("class");
    myNode.removeAttribute("id");
```

##### 总结

获取节点的属性值和设置节点的属性值，都有两种方式。

**如果是节点的“原始属性”**（比如 普通标签的`class/className`属性、普通标签的`style`属性、普通标签的 title属性、img 标签的`src`属性、超链接的`href`属性等），**方式1和方式2是等价的**，可以混用。怎么理解混用呢？比如说：用 `div.title = '我是标题'`设置属性，用 `div.getAttribute('title')`获取属性，就是混用。

但如果是节点的“非原始属性”，比如：

```javascript
div.aaa = 'qianguyihao';

div.setAttribute('bbb', 'qianguyihao');
```

上面的这个“非原始属性”，在使用这两种方式时，是有区别的。区别如下：

- 方式1 的`元素节点.属性`和`元素节点[属性]`：绑定的属性值不会出现在标签上。
- 方式2 的`get/set/removeAttribut`：绑定的属性值会出现在标签上。
- **这两种方式不能交换使用**，get值和set值必须使用同一种方法。

举例：

```html
<body>
<div id="box" title="主体" class="asdfasdfadsfd">我爱你中国</div>
<script>

    var div = document.getElementById("box");

    //采用方式一进行set
    div.aaaa = "1111";
    console.log(div.aaaa);    //打印结果：1111。可以打印出来，但是不会出现在标签上

    //采用方式二进行set
    div.setAttribute("bbbb","2222");    //bbbb作为新增的属性，会出现在标签上

    console.log(div.getAttribute("aaaa"));   //打印结果：null。因为方式一的set，无法采用方式二进行get。
    console.log(div.bbbb);                   //打印结果：undefined。因为方式二的set，无法采用方式一进行get。

</script>
</body>
```



#### DOM对象的属性-补充

##### innerHTML和innerText的区别

- value：标签的value属性。
- **innerHTML**：双闭合标签里面的内容（包含标签）。
- **innerText**：双闭合标签里面的内容（不包含标签）。（老版本的火狐用textContent）

**获取内容举例：**

如果我们想获取innerHTML和innerText里的内容，看看会如何：（innerHTML会获取到标签本身，而innerText则不会）

![img](http://img.smyhvae.com/20180127_1652.png)

**修改内容举例：**（innerHTML会修改标签本身，而innerText则不会）

![img](http://img.smyhvae.com/20180127_1657.png)

##### nodeType属性

这里讲一下nodeType属性。

- **nodeType == 1 表示的是元素节点**（标签） 。记住：在这里，元素就是标签。
- nodeType == 2 表示是属性节点。
- nodeType == 3 是文本节点。

##### nodeType、nodeName、nodeValue

我们那下面这个标签来举例：

```html
<div id="box" value="111">
    生命壹号
</div>
```

上面这个标签就包含了三种节点：

- 元素节点（标签）
- 属性节点
- 文本节点

获取这三个节点的方式如下：

```javascript
    var element = document.getElementById("box1");  //获取元素节点（标签）
    var attribute = element.getAttributeNode("id"); //获取box1的属性节点
    var txt = element.firstChild;                   //获取box1的文本节点

    var value = element.getAttribute("id");         //获取id的属性值

    console.log(element);
    console.log("--------------");
    console.log(attribute);
    console.log("--------------");
    console.log(txt);
    console.log("--------------");
    console.log(value);
```

打印结果如下：

![img](http://img.smyhvae.com/20180128_1935.png)

既然这三个都是节点，如果我想获取它们的nodeType、nodeName、nodeValue，代码如下：

```javascript
    var element = document.getElementById("box1");  //获取元素节点（标签）
    var attribute = element.getAttributeNode("id"); //获取box1的属性节点
    var txt = element.firstChild;                   //获取box1的文本节点

    //获取nodeType
    console.log(element.nodeType);       //1
    console.log(attribute.nodeType);     //2
    console.log(txt.nodeType);           //3

    console.log("--------------");

    //获取nodeName
    console.log(element.nodeName);       //DIV
    console.log(attribute.nodeName);     //id
    console.log(txt.nodeName);           //#text

    console.log("--------------");

    //获取nodeValue
    console.log(element.nodeValue);     //null
    console.log(attribute.nodeValue);   //box1
    console.log(txt.nodeValue);         //生命壹号
```

打印结果如下：

![img](http://img.smyhvae.com/20180128_1939.png)

#### 文档的加载

**浏览器在加载一个页面时，是按照自上向下的顺序加载的，读取到一行就运行一行**。如果将script标签写到页面的上边，在代码执行时，页面还没有加载，页面没有加载DOM对象也没有加载，会导致无法获取到DOM对象。

**onload 事件**：

onload 事件会在整个页面加载完成之后才触发。为 window 绑定一个onload事件，该事件对应的响应函数将会在页面加载完成之后执行，这样可以确保我们的代码执行时所有的DOM对象已经加载完毕了。

代码举例：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title></title>
    <script type="text/javascript">
      // 【方式一：先加载，后执行】这段 js 代码是写在 <head> 标签里的，所以建议放在 window.onload 里面。
      window.onload = function() {
        // 获取id为btn的按钮
        var btn = document.getElementById("btn");
        // 为按钮绑定点击事件
        btn.onclick = function() {
          alert("hello");
        };
      };
    </script>
  </head>
  <body>
    <button id="btn">点我一下</button>

    <script type="text/javascript">
      // 【方式二：后加载，后执行】这段 js 代码是写在 <body> 标签里的，代码的位置是处在页面的下方。这么做，也可以确保：在页面加载完毕后，再执行 js 代码。

      // 获取id为btn的按钮
      var btn = document.getElementById("btn");
      // 为按钮绑定点击事件
      btn.onclick = function() {
        alert("hello");
      };
    </script>
  </body>
</html>
```

上方代码中，方式一和方式二均可以确保：在页面加载完毕后，再执行 js 代码。

### 设置行内样式

#### style属性的获取和修改

在DOM当中，如果想设置样式，有两种形式：

- className（针对内嵌样式）
- style（针对行内样式）

#### 通过 js 读取元素的样式

语法：（方式一）

```javascript
    元素.style.样式名
```

备注：我们通过style属性读取的样式都是**行内样式**。

语法：（方式二）

```javascript
    元素.style["属性"];  //格式

    box.style["width"];  //举例
```

方式二最大的优点是：可以给属性传递参数。

### BOM

## 事件

### 窗口事件：window 对象触发的事件。

​	适用于 <body> 标签

```
onbeforeonload		在文档加载之前执行脚本
onerror				当错误发生时运行脚本
onblur				当窗口失去焦点时运行脚本
onhaschange			当文档改变时运行脚本
onload				当文档加载是运行脚本
onmessage			当触发消息时运行脚本
onoffline			当文档离线时运行脚本
ononline			当文档上线时运行脚本
onpagehide 			当窗口隐藏时运行脚本
onpageshow			当窗口显示时运行脚本
```

### 表单事件：由 HTML 表单内部的动作触发的事件。

​	适用于所有 HTML 5 元素，不过最常用于表单元素中

```
onblur			当元素失去焦点时运行脚本
onblur			当元素失去焦点时运行脚本
onchange		当元素改变时运行脚本
oncontextmenu	当触发上下文菜单时运行脚本
onfocus			当元素获得焦点时运行脚本
onformchange	当表单改变时运行脚本
onforminput		当表单获得用户输入时运行脚本
oninput			当元素获得用户输入时运行脚本
oninvalid		当元素无效时运行脚本
onselect		当选取元素时运行脚本
onsubmit		当提交表单时运行脚本
```

### 键盘事件：由键盘触发的事件

​	适用于所有HTML 5元素

```
onkeydown	当按下按键时运行脚本
onkeypress	当按下并松开按键时运行脚本
onkeyup		当松开按键时运行脚本
```

### 鼠标事件：由鼠标或相似的用户动作触发的事件

​	适用于所有HTML 5元素

```
onclick			当单击鼠标时运行脚本
ondblclick		当双击鼠标时运行脚本
ondrag			当拖动元素时运行脚本
ondragend		当拖动操作结束时运行脚本
ondragenter		当元素被拖动至有效的拖放目标时运行脚本
ondragleave		当元素离开有效拖放目标时运行脚本
ondragover		当元素被拖动至有效拖放目标上方时运行脚本
ondragstart		当拖动操作开始时运行脚本
ondrop			当被拖动元素正在被拖放时运行脚本
onmousedown		当按下鼠标按钮时运行脚本
onmousemove		当鼠标指针移动时运行脚本
onmouseout		当鼠标指针移出元素时运行脚本
onmouseover		当鼠标指针移至元素之上时运行脚本
onmouseup		当松开鼠标按钮时运行脚本
onmousewheel	当转动鼠标滚轮时运行脚本
onscroll		当滚动元素滚动元素的滚动条时运行脚本
```

### 多媒体事件：通过视频（videos），图像（images）或者音频（audio） 触发该事件

​	多应用于HTML媒体元素

```
onabort				当发生中止事件时运行脚本
oncanplay			当媒介能够开始播放但可能因缓冲而需要停止时运行脚本
oncanplaythrough	当媒介能够无需因缓冲而停止即可播放至结尾时运行脚本
ondurationchange	当媒介长度改变时运行脚本
onemptied			当媒介资源元素突然为空时（网络错误、加载错误等）运行脚本
onendedNew			当媒介已抵达结尾时运行脚本
onerrorNew			当在元素加载期间发生错误时运行脚本
onloadeddata		当加载媒介数据时运行脚本
onloadedmetadata	当媒介元素的持续时间以及其他媒介数据已加载时运行脚本
onloadstart			当浏览器开始加载媒介数据时运行脚本
onpause				当媒介数据暂停时运行脚本
onplay				当媒介数据将要开始播放时运行脚本
onplaying			当媒介数据已开始播放时运行脚本
onprogress			当浏览器正在取媒介数据时运行脚本
onratechange		当媒介数据的播放速率改变时运行脚本
onreadystatechange	当就绪状态（ready-state）改变时运行脚本
onseeked			当媒介元素的定位属性 [1] 不再为真且定位已结束时运行脚本
onseeking			当媒介元素的定位属性为真且定位已开始时运行脚本
onstalled			当取回媒介数据过程中（延迟）存在错误时运行脚本
onsuspend			当浏览器已在取媒介数据但在取回整个媒介文件之前停止时运行脚本
ontimeupdate		当媒介改变其播放位置时运行脚本
onvolumechange		当媒介改变音量亦或当音量被设置为静音时运行脚本
onwaiting			当媒介已停止播放但打算继续播放时运行脚本
```

### 其他事件

```
onshow		当元素在上下文显示时触发
ontoggle	当用户打开或关闭 元素时触发
```



### 事件的绑定和事件对象Event



### 事件的传播和事件冒泡

### DOM事件流

事件传播的三个阶段是：事件捕获、事件冒泡和目标。

- 事件捕获阶段：事件从祖先元素往子元素查找（DOM树结构），直到捕获到事件目标 target。在这个过程中，默认情况下，事件相应的监听函数是不会被触发的。
- 事件目标：当到达目标元素之后，执行目标元素该事件相应的处理函数。如果没有绑定监听函数，那就不执行。
- 事件冒泡阶段：事件从事件目标 target 开始，从子元素往冒泡祖先元素冒泡，直到页面的最上一级标签。

如下图所示：

![img](http://img.smyhvae.com/20180204_1218.jpg)

#### 事件捕获

addEventListener可以捕获事件：

```javascript
    box1.addEventListener("click", function () {
        alert("捕获 box3");
    }, true);
```

上面的方法中，参数为true，代表事件在捕获阶段执行。

**重点**：捕获阶段，事件依次传递的顺序是：window --> document --> html--> body --> 父元素、子元素、目标元素。

说明：

（1）第一个接收到事件的对象是 **window**（有人会说body，有人会说html，这都是错误的）。

（2）JS中涉及到DOM对象时，有两个对象最常用：window、doucument。它们俩是最先获取到事件的。

**补充一个知识点：**

在 js中：

- 如果想获取 `html`节点，方法是`document.documentElement`。
- 如果想获取 `body` 节点，方法是：`document.body`。

二者不要混淆了

#### 事件冒泡

**事件冒泡**: 当一个元素上的事件被触发的时候（比如说鼠标点击了一个按钮），同样的事件将会在那个元素的所有**祖先元素**中被触发。这一过程被称为事件冒泡；这个事件从原始元素开始一直冒泡到DOM树的最上层。

通俗来讲，冒泡指的是：**子元素的事件被触发时，父元素的同样的事件也会被触发**。取消冒泡就是取消这种机制。

**冒泡顺序**：

一般的浏览器: （除IE6.0之外的浏览器）

- div -> body -> html -> document -> window

IE6.0：

- div -> body -> html -> document

##### 不是所有的事件都能冒泡

以下事件不冒泡：blur、focus、load、unload、onmouseenter、onmouseleave。意思是，事件不会往父元素那里传递。

我们检查一个元素是否会冒泡，可以通过事件的以下参数：

```javascript
    event.bubbles
```

如果返回值为true，说明该事件会冒泡；反之则相反。

##### 阻止冒泡

大部分情况下，冒泡都是有益的。当然，如果你想阻止冒泡，也是可以的。可以按下面的方法阻止冒泡。

##### 阻止冒泡的方法

w3c的方法：（火狐、谷歌、IE11）

```javascript
    event.stopPropagation();
```

IE10以下则是：

```javascript
event.cancelBubble = true
```

兼容代码如下：

```javascript
   box3.onclick = function (event) {

        alert("child");

        //阻止冒泡
        event = event || window.event;

        if (event && event.stopPropagation) {
            event.stopPropagation();
        } else {
            event.cancelBubble = true;
        }
    }
```

上方代码中，我们对box3进行了阻止冒泡，产生的效果是：事件不会继续传递到 father、grandfather、body了。

### 事件委托

事件委托，通俗地来讲，就是**把一个元素响应事件（click、keydown......）的函数委托到另一个元素**。

比如说有一个列表 ul，列表之中有大量的列表项 `<a>`标签：

```html
<ul id="parent-list">
    <li><a href="javascript:;" class="my_link">超链接一</a></li>
    <li><a href="javascript:;" class="my_link">超链接二</a></li>
    <li><a href="javascript:;" class="my_link">超链接三</a></li>
</ul>
```

当我们的鼠标移到`<a>`标签上的时候，需要获取此`<a>`的相关信息并飘出悬浮窗以显示详细信息，或者当某个`<a>`被点击的时候需要触发相应的处理事件。我们通常的写法，是为每个`<a>`都绑定类似onMouseOver或者onClick之类的事件监听：

```javascript
    window.onload = function(){
        var parentNode = document.getElementById("parent-list");
        var aNodes = parentNode.getElementByTagName("a");
        for(var i=0, l = aNodes.length; i < l; i++){

            aNodes[i].onclick = function() {
                console.log('我是超链接 a 的单击相应函数');
            }
        }
    }
```

但是，上面的做法过于消耗内存和性能。**我们希望，只绑定一次事件，即可应用到多个元素上**，即使元素是后来添加的。

因此，比较好的方法就是**把这个点击事件绑定到他的父层**，也就是 `ul` 上，然后在执行事件函数的时候再去匹配判断目标元素。如下：

```html
    <!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title></title>
        <script type="text/javascript">
            window.onload = function() {

                // 获取父节点，并为它绑定click单击事件。 false 表示事件在冒泡阶段触发（默认）
                document.getElementById('parent-list').addEventListener('click', function(event) {
                    event = event || window.event;

                    // e.target 表示：触发事件的对象
                    //如果触发事件的对象是我们期望的元素，则执行否则不执行
                    if (event.target && event.target.className == 'link') {
                    // 或者写成 if (event.target && event.target.nodeName.toUpperCase() == 'A') {
                        console.log('我是ul的单击响应函数');
                    }
                }, false);
            };
        </script>
    </head>
    <body>
        <ul id="parent-list" style="background-color: #bfa;">
            <li>
                <p>我是p元素</p>
            </li>
            <li><a href="javascript:;" class="link">超链接一</a></li>
            <li><a href="javascript:;" class="link">超链接二</a></li>
            <li><a href="javascript:;" class="link">超链接三</a></li>
        </ul>
    </body>
```



上方代码，为父节点注册 click 事件，当子节点被点击的时候，click事件会从子节点开始**向父节点冒泡**。**父节点捕获到事件**之后，开始执行方法体里的内容：通过判断 event.target 拿到了被点击的子节点`<a>`。从而可以获取到相应的信息，并作处理。

换而言之，参数为false，说明事件是在冒泡阶段触发（子元素向父元素传递事件）。而父节点注册了事件函数，子节点没有注册事件函数，此时，会在父节点中执行函数体里的代码。

**总结**：事件委托是利用了冒泡机制，减少了事件绑定的次数，减少内存消耗，提高性能。

事件委托的参考链接：

- [荐 | JavaScript事件代理和委托（Delegation）](https://www.cnblogs.com/owenChen/archive/2013/02/18/2915521.html)
- [JavaScript 事件委托详解](https://zhuanlan.zhihu.com/p/26536815)

### 键盘事件

#### 鼠标的拖拽事件

拖拽的流程：

（1）`onmousedown`：当鼠标在被拖拽元素上按下时，开始拖拽；

（2）`onmousemove`：当鼠标移动时被拖拽元素跟随鼠标移动；

（3）`onmouseup`：当鼠标松开时，被拖拽元素固定在当前位置。

#### 鼠标的滚轮事件

`onmousewheel`：鼠标滚轮滚动的事件，会在滚轮滚动时触发。但是火狐不支持该属性。

`DOMMouseScroll`：在火狐中需要使用 DOMMouseScroll 来绑定滚动事件。注意该事件需要通过addEventListener()函数来绑定。

#### 键盘事件

**事件名**

`onkeydown`：按键被按下。

`onkeyup`：按键被松开。

**注意**：

- 如果一直按着某一个按键不松手，那么，`onkeydown`事件会一直触发。此时，松开键盘，`onkeyup`事件会执行一次。
- 当`onkeydown`连续触发时，第一次和第二次之间会间隔稍微长一点，后续的间隔会非常快。这种设计是为了防止误操作的发生。

### 定时器

#### 定时器的常见方法

- setInterval()：循环调用。将一段代码，**每隔一段时间**执行一次。（循环执行）
- setTimeout()：延时调用。将一段代码，等待一段时间之后**再执行**。（只执行一次）

## ES6



## 异步编程

### 前端使用异步的场景

什么时候需要**等待**，就什么时候用异步。常见的异步场景如下：

- 1、定时器：setTimeout（定时炸弹）、setInterval（循环执行）
- 2、事件绑定（比如说，按钮绑定点击事件之后，用户爱点不点。我们不可能卡在按钮那里，什么都不做。所以，应该用异步）
- 3、网络请求（含接口请求）：ajax 请求、网络图片加载
- 4、ES6 中的 Promise

### Ajax

#### Ajax 的概念

在浏览器中，我们可以在不刷新页面的情况下，通过 Ajax 的方式去获取一些新的内容。

Ajax：Asynchronous Javascript And XML（异步 JavaScript 和 XML）。它并不是凭空出现的新技术，而是对于现有技术的结合。Ajax 的核心是 js 对象：**XMLHttpRequest**。

#### Ajax 原理（发送 Ajax 请求的五个步骤）

> 其实也就是 使用 XMLHttpRequest 对象的五个步骤。

我们先回忆一下，一个完整的 HTTP 请求需要的是：

- 请求的网址、请求方法 get/post。
- 提交请求的内容数据、请求主体等。
- 接收响应回来的内容。

发送 Ajax 请求的五个步骤：

（1）创建异步对象，即 XMLHttpRequest 对象。

（2）使用 open 方法设置请求参数。`open(method, url, async)`。参数解释：请求的方法、请求的 url、是否异步。第三个参数如果不写，则默认为 true。

（3）发送请求：`send()`。

（4）注册事件：注册 onreadystatechange 事件，状态改变时就会调用。

如果要在数据完整请求回来的时候才调用，我们需要手动写一些判断的逻辑。

（5）服务端响应，获取返回的数据。

#### XMLHttpRequest 对象详解

##### 发送请求

发送请求的方法：

```javascript
open(method, url, async);
```

参数解释：

- method：请求的类型；GET 或 POST
- url：文件在服务器上的位置
- async：true（异步）或 false（同步）

另外还有个方法：（仅用于 POST 请求）

```javascript
send(string);
```

##### POST 请求时注意

如果想让 像 form 表单提交数据那样使用 POST 请求，就需要使用 XMLHttpRequest 对象的 setRequestHeader()方法 来添加 HTTP 头。然后在 send() 方法中添加想要发送的数据：

```javascript
xmlhttp.open('POST', 'ajax_test.php', true);

xmlhttp.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');

xmlhttp.send('name=smyhvae&age=27');
```

##### onreadystatechange 事件

注册 onreadystatechange 事件后，每当 readyState 属性改变时，就会调用 onreadystatechange 函数。

readyState：（存有 XMLHttpRequest 的状态。从 0 到 4 发生变化）

- 0: 请求未初始化
- 1: 服务器连接已建立
- 2: 请求已接收
- 3: 请求处理中
- 4: 请求已完成，且响应已就绪

status：

- 200: "OK"。
- 404: 未找到页面。

在 onreadystatechange 事件中，**当 readyState 等于 4，且状态码为 200 时，表示响应已就绪**。

##### 服务器响应的内容

- responseText：获得字符串形式的响应数据。
- responseXML：获得 XML 形式的响应数据。

如果响应的是普通字符串，就使用 responseText；如果响应的是 XML，使用 responseXML。

#### 手写第一个 Ajax 请求

get 请求：

```js
//【发送ajax请求需要五步】
//（1）创建XMLHttpRequest对象
var xmlhttp = new XMLHttpRequest();

//（2）设置请求的参数。包括：请求的方法、请求的url。
xmlhttp.open('get', '02-ajax.php');

//（3）发送请求
xmlhttp.send();

//（4）注册事件。 onreadystatechange事件，状态改变时就会调用。
//如果要在数据完整请求回来的时候才调用，我们需要手动写一些判断的逻辑。
xmlhttp.onreadystatechange = function () {
    // 为了保证 数据 完整返回，我们一般会判断 两个值
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
        //（5）服务端相应：如果能够进入这个判断，说明数据请求成功了
        console.log('数据返回成功：' + JSON.stringify(xmlhttp.responseText));

        // 伪代码：按业务需要，将接口返回的内容显示在页面上
        // document.querySelector('h1').innerHTML = xmlhttp.responseText;
    }
};
```

post 请求：

```js
//（1）异步对象
var xmlhttp = new XMLHttpRequest();

//（2）设置请求参数。包括：请求的方法、请求的url。
xmlhttp.open('post', '02.post.php');

// 如果想要使用post提交数据,必须添加此行
xmlhttp.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');

//（3）发送请求
xmlhttp.send('name=fox&age=18');

//（4）注册事件
xmlhttp.onreadystatechange = function () {
    //（5）服务端相应
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
        alert(xmlhttp.responseText);
    }
};
```

##### 封装 Ajax 请求（重要）

上面的代码，执行顺序很好理解，但在实战开发中，是不会这么写的。假如你的页面中，需要调十次接口，那岂不是要手写十遍 Ajax 请求？这样会导致大量的重复代码。

所以，我们需要把重复代码封装成一个公共函数，然后通过**回调函数**处理成功和失败的逻辑。

封装 Ajax 请求的代码如下：(get 请求为例)

```js
// 封装 Ajax为公共函数：传入回调函数 success 和 fail
function myAjax(url, success, fail) {
    // 1、创建XMLHttpRequest对象
    var xmlhttp;
    if (window.XMLHttpRequest) {
        xmlhttp = new XMLHttpRequest();
    } else {
        // 兼容IE5、IE6浏览器。不写也没关系
        xmlhttp = new ActiveXObject('Microsoft.XMLHTTP');
    }
    // 2、发送请求
    xmlhttp.open('GET', url, true);
    xmlhttp.send();
    // 3、服务端响应
    xmlhttp.onreadystatechange = function () {
        if (xmlhttp.readyState === 4 && xmlhttp.status === 200) {
            var obj = JSON.parse(xmlhttp.responseText);
            console.log('数据返回成功：' + obj);
            success && success(xmlhttp.responseText);
        } else {
            // 这里的 && 符号，意思是：如果传了 fail 参数，就调用后面的 fail()；如果没传 fail 参数，就不调用后面的内容。因为 fail 参数不一定会传。
            fail && fail(new Error('接口请求失败'));
        }
    };
}

// 单次调用 ajax
myAjax('a.json', (res) => {
    console.log(res);
});

// 多次调用 ajax。接口请求顺序：a --> b --> c
myAjax('a.json', (res) => {
    console.log(res);
    myAjax('b.json', (res) => {
        console.log(res);
        myAjax('c.json', (res) => {
            console.log(res);
        });
    });
});
```

学会了封装 get 请求之后，封装 post请求也是类似的写法。

##### Ajax 请求：get 请求举例

（1）index.html：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Document</title>
    </head>
    <body>
        <h1>Ajax 发送 get 请求</h1>
        <input type="button" value="发送get_ajax请求" id="btnAjax" />

        <script type="text/javascript">
            // 绑定点击事件
            document.querySelector('#btnAjax').onclick = function () {
                // 这里直接使用上面封装的 myAjax() 方法即可
                myAjax('02-ajax.php', (res) => {
                    console.log(res);
                    console.log('数据返回成功');
                    // 显示在页面上
                    document.querySelector('h1').innerHTML = res;
                    // alert(xhr.responseText);
                });
            };
        </script>
    </body>
</html>
```

（2）02-ajax.php：

```php
<?php
	echo 'smyhvae';
 ?>
```

效果如下：

![img](http://img.smyhvae.com/20180228_1605.gif)

## JQuery

jQuery 是 js 的一个库，封装了我们开发过程中常用的一些功能，方便我们调用，提高开发效率。

js库是把我们常用的功能放到一个单独的文件中，我们用的时候，直接引用到页面里即可。

以下是jQuery的相关信息：

- 官网：http://jquery.com/
- 官网API文档：http://api.jquery.com/
- 中文汉化API文档：http://www.css88.com/jqapi-1.9/