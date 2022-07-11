## HTML事件

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

## CSS选择器