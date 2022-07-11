# HTML学习笔记-01-标签语义化和CSS选择器

标签： HTML 学习笔记 CSS

## HTML标签

[HTML标签主要参考](https://www.w3school.com.cn/tags/html_ref_byfunc.asp)

​	标签列表:

​		基础：

```
<!DOCTYPE>		定义文档类型，告知web浏览器页面，使用了HTML的哪个版本
<html>			定义一个 HTML 文档，标签告知浏览器这是一个 HTML 文档。							
<head>			标签表示文档的头部，其中包含了与该文档有关的信息，在元素内部：<title> 必需、<style>、<link>、<meta>、<script>、<base><noscript>	
<title>			为文档定义一个标题，在所有 HTML 文档中是必需的													主要作用：												
				定义浏览器工具栏中的标题								
				提供页面被添加到收藏夹时的标题							
				显示在搜索引擎结果中的页面标题	
<body>			定义文档的主体
<h1> to <h6>	定义 HTML 标题
<p>				定义一个段落
<br />			定义简单的换行，是一个空标签
<hr />			定义水平线
<!--...-->		定义一个注释
```

​		元信息：

```
<meta>	签提供了 HTML 文档的元数据。元数据不会显示在客户端，当时会被浏览器解析。META元素通常用于指定网页的描述，关键词，的文件的最后修改，作者，和其他元数据。元数据可以被使用浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他 Web 服务调用。

<base>	规定页面上所有链接的默认 URL 和默认目标，在一个文档中，最多能使用一个<base> 元素。<base> 标签必须位于 <head> 元素内部，且应放在第一位。如果使用了 <base> 标签，则必须具备 href 属性或者target 属性或者两个属性都具备。有以下属性：	
		href：	URL 	规定页面所有相对链接的基准URL				
		target：		
			_blank 		浏览器会另开一个新窗口显示链接					
			_parent 	将链接的文件载入含有该链接框架的父框架集或父窗口中				
			_self 		在同一框架或窗口中打开所链接的文档。此参数为默认值，通常不用指定		
			_top 		在当前的整个浏览器窗口中打开所链接的文档，因而会删除所有框架framename 规定页面中所有的超链接和表单在何处打开。该属性会被每个链接中的 target 属性覆盖。
```

​		格式：

```
缩写：
<acronym>		定义只取首字母的缩写。
<abbr>			定义缩写。
信息：
<address>		定义文档作者或拥有者的联系信息。
<progress>		定义任何类型的任务的进度。
文本格式
<b>				定义粗体文本。
<bdi>			定义文本的文本方向，使其脱离其周围文本的方向设置。
<bdo>			定义文字方向。
<big>			定义大号文本。
<small>			定义小号文本。
<em>			定义强调文本。
<strong>		定义语气更为强烈的强调文本。
<code>			定义计算机代码文本。
<samp>			定义计算机代码样本
<del>			定义被删除文本。
<i>				定义斜体文本。
<ins>			定义被插入文本。
<kbd>			定义键盘文本。
<mark>			定义有记号的文本。
<sup>			定义上标文本。
<sub>			定义下标文本。
<pre>			定义预格式文本。
<var>			定义文本的变量部分。
引用：
<blockquote>	定义长的引用。
<q>				定义短的引用
<cite>			定义引用(citation)。
<meter>			定义预定义范围内的度量。

<rp>			定义若浏览器不支持 ruby 元素显示的内容。
<rt>			定义 ruby 注释的解释。
<ruby>			定义 ruby 注释。

<time>			定义日期/时间。
<tt>			定义打字机文本。
<dfn>			定义定义项目。
<wbr>			定义可能的换行符。
```

​		样式：

```
<style>	标签定义 HTML 文档的样式信息													属性：												
		type:text css	规定样式的类型								
        media:media_quary 为样式表规定不同的媒体类型。					
        scoped 如果使用该属性，则样式仅仅应用到 style 元素的父元素及其子元素。												
<div>	标签定义 HTML 文档中的一个分隔区块或者一个区域部分，常用于组合块级元素，以便通过 CSS 来对这些元素进行格式化。																		
<span>	用于对文档中的行内元素进行组合，标签提供了一种将文本的一部分或者文档的一部分独立出来的方式。																											
<header>	标签定义文档或者文档的一部分区域的页眉，元素应该作为介绍内容或者导航链接栏的容器，在一个文档中，您可以定义多个 <header> 元素。<header> 标签不能被放在 <footer>、<address> 或者另一个 <header> 元素内部、																			
<footer>	标签定义文档或者文档的一部分区域的页脚。元素应该包含它所包含的元素的信息。在典型情况下，该元素会包含文档创作者的姓名、文档的版权信息、使用条款的链接、联系信息等等。在一个文档中，您可以定义多个 <footer> 元素。插入联系信息，应该在 <footer> 元素内使用 <address> 标签									
<section>	标签定义了文档的某个区域。比如章节、头部、底部或者文档的其他区域。																
<article>	标签定义独立的内容。定义其所处内容之外的内容。定义的内容本身必须是有意义的且必须是独立于文档的其余部分，例如：论坛帖子、博客文章、新闻故事、评论																							
<aside>		标签用来表示跟当前页面的内容，内容应该与附近的内容相关，通常用于显示侧边栏或者补充的内容，例如：目录、索引等。																					
<details>	定义了用户可见的或者隐藏的需求的补充细节。只有 Chrome 和 Safari 6 支持 <details> 标签。 标签规定了用户可见的或者隐藏的需求的补充细节。用来供用户开启关闭的交互式控件。任何形式的内容都能被放在 <details> 标签里边。元素的内容对用户是不可见的，除非设置了 open 属性，即open=open																
<summary>	为 <details> 元素定义一个可见的标题。当用户点击标题时会显示出详细信息。		
<dialog>	定义一个对话框或者窗口
```

​		表格：

```
<table>		定义一个表格，一个 HTML 表格包括 <table> 元素，一个或多个 <tr>、<th> 以及 <td> 元素。<tr> 元素定义表格行，<th> 元素定义表头，<td> 元素定义表格单元。更复杂的 HTML 表格也可能包括 <caption>、<col>、<colgroup>、<thead>、<tfoot> 以及 <tbody> 元素。									
<caption>	定义表格标题。标签必须直接放置到 <table> 标签之后。																		
<tr>	定义表格中的行。一个 <tr> 元素包含一个或多个 <th> 或 <td> 元素。	
<th>	定义表格中的表头单元格 - 包含头部信息。如果需要将内容横跨多个行或列，请使用 colspan 和 rowspan 属性！													
<td>	定义表格中的标准单元格 - 包含数据。								
<thead>	标签用于组合 HTML 表格的表头内容，应该与 <tbody> 和 <tfoot> 元素结合起来使用，用来规定表格的各个部分（表头、主体、页脚）。通过使用这些元素，使浏览器有能力支持独立于表格表头和表格页脚的表格主体滚动。当包含多个页面的长的表格被打印时，表格的表头和页脚可被打印在包含表格数据的每张页面上。				
<tbody>	标签用于组合 HTML 表格的主体内容。具体同上
<tfoot>	标签用于组合 HTML 表格的页脚内容。具体同上							
<colgroup>	定义表格中供格式化的列组。标签用于对表格中的列进行组合，以便对其进行格式化。通过使用 <colgroup> 标签，可以向整个列应用样式，而不需要重复为每个单元格或每一行设置样式。只能在 <table> 元素之内，在任何一个 <caption> 元素之后，在任何一个 <thead>、<tbody>、<tfoot>、<tr> 元素之前使用 <colgroup> 标签。														
<col>	定义表格中一个或多个列的属性值。通过使用 <col> 标签，可以向整个列应用样式，而不需要重复为每个单元格或每一行设置样式。
示例：
<table border="1">
  <colgroup>
    <col span="2" style="background-color:red">
    <col style="background-color:yellow">
  </colgroup>
  <thead>
  	<tr>
  	  <th>ISBN</th>
  	  <th>Title</th>
 	   <th>Price</th>
 	 </tr>
  </thead>
  <tbody>
 	 <tr>
 	   <td>3476896</td>
	    <td>My first HTML</td>
 	   <td>$53</td>
 	 </tr>
  </tbody>
  <tfoot>
 	 <tr>
 	   <td>5869207</td>
 	   <td>My first CSS</td>
  	  <td>$49</td>
 	 </tr>
  </tfoot>
</table>
```

​		列表：

```
<ul>	定义一个无序列表
<ol>	定义一个有序列表， 列表排序以数字来显示。
<li>	定义一个列表项													
<dl>	标签定义一个描述列表。与 <dt> （定义项目/名字）和 <dd> （描述每一个项目/名字）一起使用。
<dt>	定义一个定义定义列表中的项目。
<dd>	标签被用来对一个描述列表中的项目/名字进行描述。									
<ul>
	<li></li>
	<li></li>
</ul>
<ol>
	<li></li>
	<li></li>
</ol>
<dl>
	<dt></dt>
		<dd></dd>
	<dt></dt>
		<dd></dd>
</dl>
```

​		表单：

```
<form>		定义供用户输入的 HTML 表单。
<input>		定义输入控件。
<label>		定义 input 元素的标注。
<textarea>	定义多行的文本输入控件。

<button>	定义按钮。
<optgroup>	定义选择列表中相关选项的组合。
<option>	定义选择列表中的选项。
<datalist>	定义下拉列表。
<select>	定义选择列表（下拉列表）

<fieldset>	定义围绕表单中元素的边框。
<legend>	定义 fieldset 元素的标题。

<keygen>	定义生成密钥。
<output>	定义输出的一些类型。
```

​		链接：

```
<a>		标签定义超链接,元素最重要的属性是 href 属性，它指定链接的目标。			
		download	filename		指定下载链接									
    	href		URL				规定链接的目标 URL。								
    	hreflang	language_code	规定目标 URL 的基准语言。仅在 href 属性存在时使
    	media	media_query	规定目标 URL 的媒介类型。默认值：all。仅在 href 属性存在时使用target _blank _parent _self _top framename 规定页面中所有的超链接和表单在何处打开type 规定目标URL的MIME类型												
<a href="URL"></a>
        
<link>	定义文档与外部资源的关系。最常见的用途是链接样式表。此元素只能存在于 head 部分，不过它可出现任何次数。														
		href		URL				定义被链接文档的位置	
        hreflang	language_code	定义被链接文档中文本的语言。				
        media		media_query		规定被链接文档将显示在什么设备上。			
        rel			alternate archives author bookmark external first help icon last license next nofollow noreferrer pingback prefetch prev search sidebar stylesheet tagup 必需。定义当前文档与被链接文档之间的关系。						
        type 		MIME_TYPE 		 规定被链接文档的 MIME 类型。
<head>
	<link rel="stylesheet" type="text/css" href="css文件路径">
</head>	

<nav>	标签定义导航链接的部分,并不是所有的 HTML 文档都要使用到 <nav> 元素。<nav> 元素只是作为标注一个导航链接的区域。
```

​		图像：

```
<img>	标签定义 HTML 页面中的图像,标签有两个必需的属性：src 和 alt。
		属性：															
		src URL 规定显示图片的URL									
        alt		规定图像的替代文本。										
        height  规定图像的高度.可用css设置										
        width	规定图像的宽度											
        lsmap 	将图像规定为服务器端图像映射。								
        usemap	将图像定义为客户器端图像映射。
<img src="URL" alt="内容" width="42" height="42">
	
<map>	定义图像映射。<img>中的 usemap 属性可引用 <map> 中的 id 或 name 属性（取决于浏览器），所以我们应同时向 <map> 添加 id 和 name 属性。area 元素永远嵌套在 map 元素内部。area 元素可定义图像映射中的区域。				
<area>	 标签定义图像映射内部的区域（图像映射指的是带有可点击区域的图像）。	元素始终嵌套在 <map> 标签内部。									

<img src="w3cnote://file/getImage?fileId=planetsgif"width="145" height="126"alt="Planets"usemap="#planetmap">

<mapname="planetmap">
	<area shape="rect" coords="0,0,82,126" href="https://www.w3cschool.cn/html5/sun.htm" alt="Sun">
	<area shape="circle" coords="90,58,3" href="https://www.w3cschool.cn/html5/mercur.htm" alt="Mercury">
	<area shape="circle" coords="124,58,8" href="https://www.w3cschool.cn/html5/venus.htm" alt="Venus">
</map>

<canvas>	通过脚本（通常是 JavaScript）来绘制图形（比如图表和其他图像）。
<figcaption>	标签为 <figure> 元素定义标题。元素应该被置于 <figure> 元素的第一个或最后一个子元素的位置。
<figure>	figure 标签用于对元素进行组合。标签规定独立的流内容（图像、图表、照片、代码等等）。元素的内容应该与主内容相关，同时元素的位置相对于主内容是独立的。如果被删除，则不应对文档流产生影响。
```

​		音视频：

```
<audio>		定义声音，比如音乐或其他音频流。
<video>		定义一个音频或者视频
<source>	标签为媒体元素（比如 <video> 和 <audio>）定义媒体资源。					
			属性：															
			media media_query 规定媒体资源的类型，供浏览器决定是否下载			
			src	URL	规定媒体文件的 URL										
			type MIME_type	规定媒体资源的 MIME 类型
<audio controls> 
	<source src="horse.ogg" type="audio/ogg"> 
	<source src="horse.mp3" type="audio/mpeg"> 您的浏览器不支持 audio 元素。
</audio>

<track>	为媒体(<video> 和 <audio>)元素定义外部文本轨道。

```

​		程序：

```
<script>	定义客户端脚本。
<noscript>	元素用来定义在脚本未被执行时的替代内容（文本）。此标签可被用于可识别 <noscript> 标签但无法支持其中的脚本的浏览器。														
<embed>	定义了一个容器，用来嵌入外部应用或者互动程序（插件）					
		src 规定被嵌入内容的 URL										
        type 	规定嵌入内容的 MIME 类型。								
        height 规定嵌入内容的高度										
        width 规定嵌入内容的宽度
<embed src="URL">
	
<object>	标签定义一个嵌入的对象。请使用此元素向您的 XHTML 页面添加多媒体。此元素运行您规定插入 HTML 文档中的对象的数据和参数，以及可用来显示和操作数据的代码。
<param>	标签为<object>标签提供嵌入内容的运行时参数的name与value对。
<object data="horse.wav">
  <param name="autoplay" value="true">
</object>
```

## CSS选择器

```
类选择器
.class				.intro				选择所有class="intro"的元素
id选择器
#id					#firstname			选择所有id="firstname"的元素
通用选择器
*					*					选择所有元素	
元素选择器：
element				p					选择所有<p>元素	
群组选择器
element,element		div,p				选择所有<div>元素和<p>元素
后代选择器
element element		div p				选择<div>元素内的所有<p>元素	 包含子孙元素
子元素选择器
element>element		div>p				选择所有父级是 <div> 元素的 <p> 元素  只能是子元素
相邻兄弟选择器
element+element		div+p				选择所有紧接着<div>元素之后的<p>元素	

属性选择器：																	
[attribute]			[target]			选择所有带有target属性元素	
[attribute=value]	[target=-blank]		选择所有使用target="-blank"的元素	
[attribute~=value]	[title~=flower]		选择标题属性包含单词"flower"的所有元素	
[attribute|=value]	[lang|=en]			选择 lang 属性以 en 为开头的所有元素	
[attribute^=value]	a[src^="https"]		选择每一个src属性的值以"https"开头的元素
[attribute$=value]	a[src$=".pdf"]		选择每一个src属性的值以".pdf"结尾的元素
[attribute*=value]	a[src*="runoob"]	选择每一个src属性的值包含子字符串"runoob"的元素																				
伪类：
:link				a:link				选择所有未访问链接	
:visited			a:visited			选择所有访问过的链接	
:active				a:active			选择活动链接	
:hover				a:hover				选择鼠标在链接上面时
:lang(language)		p:lang(it)			选择带有以 "it" 开头的 lang 属性值的每个 <p> 元素
:focus				input:focus			选择具有焦点的输入元素							
伪元素：
:first-letter		p:first-letter		选择每一个<p>元素的第一个字母	
:first-line			p:first-line		选择每一个<p>元素的第一行	
:first-child		p:first-child		指定只有当<p>元素是其父级的第一个子级的样式。	
:before				p:before			在每个<p>元素之前插入内容	
:after				p:after				在每个<p>元素之后插入内容	

element1~element2	p~ul				选择p元素之后的每一个ul元素	
:first-of-type		p:first-of-type		选择每个p元素是其父级的第一个p元素
:last-of-type		p:last-of-type		选择每个p元素是其父级的最后一个p元素
:only-of-type		p:only-of-type		选择每个p元素是其父级的唯一p元素
:only-child			p:only-child		选择每个p元素是其父级的唯一子元素
:nth-child(n)		p:nth-child(2)		选择每个p元素是其父级的第二个子元素
:nth-last-child(n)	p:nth-last-child(2)	选择每个p元素的是其父级的第二个子元素，从最后一个子项计数
:nth-of-type(n)		p:nth-of-type(2)	选择每个p元素是其父级的第二个p元素
:nth-last-of-type(n)p:nth-last-of-type(2)选择每个p元素的是其父级的第二个p元素，从最后一个子项计数
:last-child			p:last-child		选择每个p元素是其父级的最后一个子级。

:root				:root				选择文档的根元素
:empty				p:empty				选择每个没有任何子级的p元素（包括文本节点）
:target				#news:target		选择当前活动的#news元素（包含该锚名称的点击的URL）
:enabled			input:enabled		选择每一个已启用的输入元素
:disabled			input:disabled		选择每一个禁用的输入元素
:checked			input:checked		选择每个选中的输入元素
:not(selector)		:not(p)				选择每个并非p元素的元素
::selection			::selection			匹配元素中被用户选中或处于高亮状态的部分

:out-of-range		:out-of-range		匹配值在指定区间之外的input元素
:in-range			:in-range			匹配值在指定区间之内的input元素
:read-write			:read-write			用于匹配可读及可写的元素
:read-only			:read-only			用于匹配设置 "readonly"（只读） 属性的元素
:optional			:optional			用于匹配可选的输入元素
:required			:required			用于匹配设置了 "required" 属性的元素
:valid				:valid				用于匹配输入值为合法的元素
:invalid			:invalid			用于匹配输入值为非法的元素
```

​	选择器有着不用的优先级：（[主要参考](https://blog.csdn.net/b954960630/article/details/79560590?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)）

​		不同级别：

​		1.在属性后面使用 !important 会覆盖页面内任何位置定义的元素样式。

​		2.作为style属性写在元素内的样式

​		3.id选择器

​		4.类选择器

​		5.元素选择器

​		6.通配符选择器

​		7.浏览器自定义或继承

​	总结排序：!important > 行内样式 > ID选择器 > 类选择器 > 元素 > 通配符 > 继承 > 浏览器默认属性

​	1.在级别中，先写的会被后写的覆盖

​	2.同一级别css引入方式不同，优先级不同

​		总结排序：内联(行内)样式 > 内部样式表 > 外部样式表 > 导入样式(@import)。



## css布局

[主要参考](https://blog.csdn.net/zhang6223284/article/details/81909600)

### 1.table布局

​		table 的特性决定了它非常适合用来做布局，并且表格中的内容可以自动居中，这是之前用的特别多的一种布局方式。

​		但是它也有自身的局限性，比如 table 比其它 html 标记占更多的字节（造成下载时间延迟,占用服务器更多流量资源），table 会阻挡浏览器渲染引擎的渲染顺序。(会延迟页面的生成速度,让用户等待更久的时间)，但是某些情况下，当采用其他方式不能很好的达到自己的效果时，采用 table 布局能适应当前场景。

### 2.浮动布局

​		float 布局应该是目前各大网站用的最多的一种布局方式了。浮动元素是脱离文档流的，但不脱离文本流，这是什么意思呢，用过 word 的应该知道有一种图片环绕的方式是文字环绕吧，就是这种效果。

​		特点：

​	1.对自身的影响

​		1.1float 元素可以形成块，如 span 元素。可以让行内元素也拥有宽和高，因为块级元素具有宽高
​		1.2浮动元素的位置尽量靠上
​		1.3尽量靠左（float:left）或右（float:right），如果那一行满足不了浮动元素的宽度要求，则元素会往下掉
​	2.对兄弟元素的影响
​		2.1不影响其他块级元素的位置
​		2.2影响其他块级元素的文本
​		2.3上面贴非 float 元素
​		2.4旁边贴 float 元素或者边框
​	3.对父级元素的影响
​		3.1从布局上 “消失”
​		3.2高度塌陷

​		但是当为子元素设置浮动以后，子元素会完全脱离文档流，此时将会导致子元素无法撑起父元素的高度，导致父元素的高度塌陷。

​	解决办法有下面几种

​		1.父元素设置 overflow: auto 或者 overflow: hidden

​		2.给父元素加一个 after 伪类

### 3.定位布局

​	通过position的属性，进行布局，定义各个元素所在的像素位置。

​	position 有如下几个值

​		static（默认情况，存在文档流当中）
​		relative（根据元素本身原来所应该处的位置偏移，不会改变布局的计算）
​		absolute（绝对定位，脱离文档流，不会对别的元素造成影响，相对的是父级最近的 relative 或者 absolute 定位元素）
​		fixed（绝对定位，脱离文档流，相对于的是屏幕，就是那些浮动的广告那样，怎么拉都固定在同一个位置，而 absolute 元素离开屏幕就看不见了）

### 4.弹性盒子布局

flexbox 布局即弹性盒子布局，它的特点是盒子本来就是并列的，只需要指定宽度，来看一个经典的三栏布局的例子

### Grid布局

​	Grid布局，是一个基于网格的二维布局系统，目的是用来优化用户界面设计。例如grid960

## less\sass

​		都是css的拓展语言，less和sass最主要的区别是less是通过Javascript编译，而sass是通过ruby编译的，如果没有引入前端工程化，less会消耗客户端性能，sass会消耗服务端性能，但是引入前端工程化的话，gunt，gulp，webpack等，less和sass在打包阶段都会转化成css，所以不会有区别，只是sass是基于ruby，所以每次npm的时候相对慢一点点

​		CSS是一种标记性语言。如果没有calc()方法，CSS是不能进行真正意义上的计算的，更不提函数、变量这些了。

​		less是一种动态样式语言，对css赋予了动态语言的特性，比如变量、继承、运算、函数，既可以运行在客户端，也可以运行在服务器端，依赖JavaScript。

​		sass是一种动态语言，属于缩排语法，比css多出很多功能，比如变量、嵌套、运算、混入、继承、函数等，更容易阅读；

​		sass与scss关系：sass的缩排语法，对于阅读者很不直观，因此sass对语法进行改良，sass3就变成scss，与原来的语法兼容，只是用{}取代了原来的缩进。



​		区别之处：

​		1.less基于JavaScript，是在客户端处理的

​    		很多开发者不会选择Less因为javaScript引擎需要额外的时间来处理代码然后输出修改过的Css到浏览器， 【解决：只在开发阶段使用Less,一旦开发完成，复制Less输出的到一个压缩器，然后用一个单独的css文件来代替Less文件；另一种方式是使用Less App来编译和压缩你的Less文件；这两种方式都是最小化样式输出】

​		2.sass是基于ruby的，是在服务器端处理的

​		变量在less和sass中唯一的区别就是，less使用@，sass使用$

​		3.变量、使用方面的区别

[主要参考](https://www.jianshu.com/p/6a35a548c9e1)

[主要参考](https://www.cnblogs.com/AmyLin-blogs/p/11453646.html)

### less

[less中文网](https://less.bootcss.com/)

1.变量：使用@变量名

```
@width: 10px;
@height: @width + 10px;

#header {
  width: @width;
  height: @height;
}
```

2.混合

3.嵌套

5.运算

6.转义

7.函数

8.命名空间和访问符

9.映射

10.作用域

11.注释

12.导入

### sass

[sass中文文档](https://www.sass.hk/docs/)

​	1.变量

​	2.注释

​	3.数据类型

​	4.运算

​	5.圆括号

​	6.函数

​	7.插值语句

​	8.&

​	9.变量定义

## 学习心得

​		在这部分学习中，主要学习了语义化标签和css事件。大部分的内容都是之前自己看视频和张凡老师讲解过的，学习起来比较轻松，自己也练习大部分的标签和css事件，练习才会深入理解这些标签和事件。通过练习和老师的讲解对标签的语义化也有了更明确的理解。css事件也主要是通过练习才能知道怎么用，怎么区分这些选择器。

​	在老师讲解布局的时候，也明白了模块化设计的重要意义与便利。CCS模块将作用域限制于组件中，从而避免了全局作用域的问题。我们再也不用操心为组件寻找一个好的命名了，因为编译过程已经帮你完成了这个任务。模块化设计的理念是通用的，学到的不是某一种模式，是整个模块化设计的理念，学习的不再是一个简单的运用而是理解其中的原理，分离常用的样式，设置统一的命名空间，代码的复用，对我们以后学习各种设计模式有所帮助。

​		在写笔记的学习的时候，查了很多的资料，也仔细看了一些文章，对加深了对标签和时间的理解和映象，less和sass在之前张凡老师也讲过，主要的用法还是得通过自学才能完成。以后还需要对这一块的知识进行补充。也还需要通过多运用和理解，不断练习提高自己。

​	