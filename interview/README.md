# MyInterview

面试问题以及相关的书籍。

面试问题大都来源是其他博客。

网址：

[golangFamily](https://github.com/xiaobaiTech/golangFamily)

[Go Questions](https://www.bookstack.cn/read/qcrao-Go-Questions/README.md)

[【Golang】Go 3到5年常见的面试题](https://www.cnblogs.com/chenpingzhao/p/13358378.html) 

[Golang面试前二夜准备](https://www.modb.pro/db/159470)

## GO 面试题

面经问题

**新手**

- [Golang开发新手常犯的50个错误](https://blog.csdn.net/gezhonglei2007/article/details/52237582)

**数据类型（map、slice、数组、set）**

[基本数据类型](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md#%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)

- [连nil切片和空切片一不一样都不清楚？那BAT面试官只好让你回去等通知了。](https://mp.weixin.qq.com/s/sW4PD1MiaunURNDIU4BbQQ)
- [golang 中的四种类型转换总结](https://learnku.com/articles/42797)
- [golang面试题：字符串转成byte数组，会发生内存拷贝吗？](https://mp.weixin.qq.com/s/qmlPuGVISx8NYp2b9LrqnA)
- [golang面试题：翻转含有中文、数字、英文字母的字符串](https://mp.weixin.qq.com/s/ssinnUM22PHPWRug8EzAkg)
- [golang面试题：拷贝大切片一定比小切片代价大吗？](https://mp.weixin.qq.com/s/8Dp2eCYzDdBbxAG5-jNevQ)
- [map不初始化使用会怎么样](https://blog.csdn.net/qq_39920531/article/details/88103496)
- [map不初始化长度和初始化长度的区别](https://www.kancloud.cn/kancloud/the-way-to-go/72493)
- [map承载多大，大了怎么办](https://yangxikun.com/golang/2019/10/07/golang-map.html)
- [map的iterator是否安全？能不能一边delete一边遍历？](https://www.bookstack.cn/read/qcrao-Go-Questions/map-%E5%8F%AF%E4%BB%A5%E8%BE%B9%E9%81%8D%E5%8E%86%E8%BE%B9%E5%88%A0%E9%99%A4%E5%90%97.md)
- [字符串不能改，那转成数组能改吗，怎么改](http://c.biancheng.net/view/39.html)
- [怎么判断一个数组是否已经排序](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter03/03.1.html)
- [普通map如何不用锁解决协程安全问题](https://zhuanlan.zhihu.com/p/356739568)
- [array和slice的区别](https://segmentfault.com/a/1190000013148775)
- [golang面试题：json包变量不加tag会怎么样？](https://mp.weixin.qq.com/s/bZlKV_BWSqc-qCa4DrsCbg)
- [golang面试题：reflect（反射包）如何获取字段tag？为什么json包不能导出私有变量的tag？](https://mp.weixin.qq.com/s/P7TEx2mInwEktXTEE6JDWQ)
- [零切片、空切片、nil切片是什么](https://juejin.cn/post/6844903712654098446)
- [slice深拷贝和浅拷贝](https://learnku.com/articles/59163)
- [nil 不同于 null（或是 NULL、nullptr）](https://blog.singee.me/2020/09/24/e8cb67835ea44243b136e3cdf8d5ea84/)
- [map触发扩容的时机，满足什么条件时扩容？](https://www.bookstack.cn/read/qcrao-Go-Questions/map-map%20%E7%9A%84%E6%89%A9%E5%AE%B9%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84.md)
- [map扩容策略是什么](https://www.bookstack.cn/read/qcrao-Go-Questions/map-map%20%E7%9A%84%E6%89%A9%E5%AE%B9%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84.md)
- [自定义类型切片转字节切片和字节切片转回自动以类型切片](https://blog.csdn.net/weixin_42506905/article/details/81359448)
- [make和new什么区别](https://juejin.cn/post/6859145664316571661)
- [slice ，map，chanel创建的时候的几个参数什么含义](https://blog.csdn.net/TCatTime/article/details/111567560)
- [slice，len，cap，共享，扩容](https://cloud.tencent.com/developer/article/1822529)
- [go struct能不能比较？](https://juejin.cn/post/6881912621616857102)
- [使用range 迭代 map 是有序的吗?  map如何顺序读取？](https://www.jianshu.com/p/65d338bb6c82)
- [go中怎么实现set](https://studygolang.com/articles/11179)
- [使用值为 nil 的 sice、map 会发生什么？](https://segmentfault.com/a/1190000038175302)
- [Golang 有没有 this 指针？](https://blog.csdn.net/ma2595162349/article/details/108632865)
- [Golang 语言中局部变量和全局变量的缺省值是什么](https://studygolang.com/articles/15282)
- [Golang 中的引用类型包含哪些?](https://studygolang.com/articles/27252)
- [slice 的扩容机制是什么？](https://juejin.cn/post/6844903812331732999)
- [Golang 中指针运算有哪些?](https://blog.csdn.net/fly910905/article/details/105989267)
- [string 类型的值可以修改吗？](https://www.cnblogs.com/brady-wang/p/15821039.html)
- [array 类型的值作为函数参数是引用传递还是值传递？](https://segmentfault.com/a/1190000037763005)
- [nil](https://segmentfault.com/a/1190000039894167)

**流程控制(for、select、defer、switch)**

- [昨天那个在for循环里append元素的同事，今天还在么？](https://studygolang.com/articles/30844)
- [golang面试官：for select时，如果通道已经关闭会怎么样？如果只有一个case呢？](https://cloud.tencent.com/developer/article/1796708)
- [go defer（for defer）](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-defer/)
- [select可以用于什么？](https://blog.csdn.net/zhaominpro/article/details/77570290)
- [context包的用途？](https://zhuanlan.zhihu.com/p/76555349)
- [switch 中如何强制执行下一个 case 代码块?](https://www.cnblogs.com/gwyy/p/13670090.html)
- [如何从 panic 中恢复?](https://learnku.com/docs/the-way-to-go/133-recovery-from-panic-recover/3676)

**解析**

- [解析 JSON 数据时，默认将数值当做哪种类型](https://zhuanlan.zhihu.com/p/388124005)

**包管理**

- [学go mod就够了！](https://studygolang.com/articles/27293)

**优化**

- [golang面试题：怎么避免内存逃逸？](https://mp.weixin.qq.com/s/VzRTHz1JaDUvNRVB_yJa1A)
- [golang面试题：简单聊聊内存逃逸？](https://mp.weixin.qq.com/s/wJmztRMB1ZAAIItyMcS0tw)
- [给大家丢脸了，用了三年golang，我还是没答对这道内存泄漏题](https://mp.weixin.qq.com/s/-agtdhlW7Yj7S88a0z7KHg)
- [内存碎片化问题](https://segmentfault.com/a/1190000020338427)
- [chan相关的goroutine泄露的问题](https://segmentfault.com/a/1190000040161853)
- [string相关的goroutine泄露的问题](cnblogs.com/ricklz/p/11262069.html)
- [你一定会遇到的内存回收策略导致的疑似内存泄漏的问题](https://colobu.com/2019/08/28/go-memory-leak-i-dont-think-so/)
- [sync.Pool的适用场景](https://geektutu.com/post/hpg-sync-pool.html)

**goroutine**

- [线程、进程、协程、goroutines ](https://zhuanlan.zhihu.com/p/27245377)
- [golang面试题：对已经关闭的的chan进行读写，会怎么样？为什么？](https://mp.weixin.qq.com/s/izbZ3JRqX6jI6Wn7bV6xNQ)
- [golang面试题：对未初始化的的chan进行读写，会怎么样？为什么？](https://juejin.cn/post/6844904196181852173)
- [sync.map 的优缺点和使用场景](https://studygolang.com/articles/22128)
- [主协程如何等其余协程完再操作](https://blog.csdn.net/weixin_42678507/article/details/102786680)
- [有缓存的channel和没有缓存的channel区别是什么？](https://zhuanlan.zhihu.com/p/355487940)
- [协程通信方式有哪些？](https://zhuanlan.zhihu.com/p/36907022)
- [channel底层实现](https://i6448038.github.io/2019/04/11/go-channel/)
- [读写锁底层是怎么实现的？](https://blog.csdn.net/sunxianghuang/article/details/104780010)
- [golang的CSP思想](https://zhuanlan.zhihu.com/p/313763247)
- [channel 是怎么保证线程安全？](http://www.zhoubotong.site/post/25.html)
- [协程和线程的差别](https://segmentfault.com/a/1190000040373756)
- [开多个线程和开多个协程会有什么区别](https://www.kancloud.cn/todo/go_learn/1222804)
- [协程可以自己主动让出 CPU 吗？](https://studygolang.com/articles/26795)
- [GMP](https://learnku.com/articles/41728)
- [GMP模型](https://zboya.github.io/post/go_scheduler/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [动图图解，GMP里为什么要有P](https://mp.weixin.qq.com/s/SEE2TUeZQZ7W1BKkmnelAA)

**反射**

- [golang 面试题：reflect（反射包）如何获取字段 tag？为什么 json 包不能导出私有变量的 tag？](https://mp.weixin.qq.com/s/WK9StkC3Jfy-o1dUqlo7Dg)

**接口（ interface）**

- [开源库里会有一些类似下面这种奇怪的用法：`var _ io.Writer = (*myWriter)(nil)`，是为什么？](https://www.bookstack.cn/read/qcrao-Go-Questions/interface-%E7%BC%96%E8%AF%91%E5%99%A8%E8%87%AA%E5%8A%A8%E6%A3%80%E6%B5%8B%E7%B1%BB%E5%9E%8B%E6%98%AF%E5%90%A6%E5%AE%9E%E7%8E%B0%E6%8E%A5%E5%8F%A3.md)
- [两个interface{} 能不能比较](https://www.jianshu.com/p/a982807819fa)
- [断言时会发生拷贝吗](https://blog.csdn.net/qq_39397165/article/details/115500314)
- [接口是怎么实现的？](https://juejin.cn/post/6844904082453299207)

**unsafe**

- [golang面试题：能说说uintptr和unsafe.Pointer的区别吗？](https://mp.weixin.qq.com/s/PSkz0zj-vqKzmIKa_b-xAA)

**GC**

[GC](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/GC.md)

- [重点](https://www.cnblogs.com/luozhiyun/p/14564903.html)

- [垃圾回收的过程是怎么样的？](https://zhuanlan.zhihu.com/p/297177002)
- [什么是写屏障、混合写屏障，如何实现？](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.9.GC-GC.md)
- [为什么gc会让程序变慢](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.14.GC-GC.md)
- [gc的stw是怎么回事](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.5.GC-GC.md)
- [为什么小对象多了会造成 gc 压力?](https://www.modb.pro/db/148423)
- [两次 GC 周期重叠会引发什么问题，GC 触发机制是什么样的？](https://www.jianshu.com/p/bfc3c65c05d1)

**问题排查**

- [trace](https://mp.weixin.qq.com/s?__biz=MzA4ODg0NDkzOA==&mid=2247487157&idx=1&sn=cbf1c87efe98433e07a2e58ee6e9899e&source=41#wechat_redirect)
- [pprof](https://mp.weixin.qq.com/s/d0olIiZgZNyZsO-OZDiEoA)
- [什么是 goroutine 泄漏?](https://segmentfault.com/a/1190000040161853)
- [当go服务部署到线上了，发现有内存泄露，该怎么处理](https://blog.csdn.net/shudaqi2010/article/details/103362028?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_default&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

**其他**

- 利用golang特性，设计一个QPS为500的服务器
- [必须要手动对齐内存的情况](https://geektutu.com/post/hpg-struct-alignment.html)
- [go栈扩容和栈缩容，连续栈的缺点](https://segmentfault.com/a/1190000019570427)
- [golang怎么做代码优化](https://tangheng1995.github.io/golang/2020/06/03/Golang-optimize.html)
- [golang隐藏技能:怎么访问私有成员](https://www.jianshu.com/p/7b3638b47845)
- [一个协程能保证绑定在一个内核线程上吗？](https://studygolang.com/articles/26795)
- [闭包怎么实现的,闭包的主要应用场景](https://zhuanlan.zhihu.com/p/56750616)
- [Goroutinue 什么时候会被挂起？](https://developer.51cto.com/article/681462.html)
- [Data Race 问题怎么检测？怎么解决?](https://learnku.com/articles/45279)
- [Golang 触发异常的场景有哪些?](http://xueyuan.coder55.com/read/go-senior-learn/go-question-14.2)
- [net/http包中client如何实现长连接？](net/http包中client如何实现长连接？)
- [net/http怎么做连接池和长链接？](net/http怎么做连接池和长链接？)

## 网络基础

[计算机网络](https://github.com/Simin-hub/Golang-Learning-and-Interview/blob/main/Go/%E5%9F%BA%E7%A1%80/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C.md)

- [漫画图解HTTP知识点+面试题](https://mp.weixin.qq.com/s/wNRoDoW_VEqiq8JelePj2g)
- [TCP粘包 数据包：我只是犯了每个数据包都会犯的错](https://mp.weixin.qq.com/s/0-YBxU1cSbDdzcZEZjmQYA)
- [30张图带你搞懂！路由器，集线器，交换机，网桥，光猫有啥区别？](https://mp.weixin.qq.com/s/6eQ00Wzss61XUTO8xeL3iA)
- [既然IP层会分片，为什么TCP层也还要分段？](https://mp.weixin.qq.com/s/YpQGsRyyrGNDu1cOuMy83w)
- [断网了，还能ping通 127.0.0.1 吗？为什么？](https://mp.weixin.qq.com/s/Gml_xxvGjq224L7zCoXm5w)
- [连接一个 IP 不存在的主机时，握手过程是怎样的？](https://mp.weixin.qq.com/s/Czv0CxDKqr2QNItO4aNZMA)
- [动图图解！代码执行send成功后，数据就发出去了吗？](https://mp.weixin.qq.com/s/87BZzLmcntA1snJIIhUR0w)
- [活久见！TCP两次挥手，你见过吗？那四次握手呢？](https://mp.weixin.qq.com/s/Z0EqSihRaRbMscrZJl-zxQ)
- [动图图解！收到RST，就一定会断开TCP连接吗？](https://mp.weixin.qq.com/s/Fr6o6gRiIUIspV9-jR9snw)
- [动图图解！没有accept，能建立TCP连接吗？](https://mp.weixin.qq.com/s/n17NjGRab1u5eXkOCro1gg)
- [来了来了！小白图解网络电子书和博客都来啦！](https://mp.weixin.qq.com/s/yZPorh6js8cq0_6FjfnGZA)
- [HTTP 是无状态的吗？需要保持状态的场景应该怎么做？](https://segmentfault.com/a/1190000009518499)
- [粘包如何解决](https://www.cnblogs.com/cangqinglang/p/11503057.html)
- [RestFul 是什么？RestFul 请求的 URL 有什么特点？](https://www.cnblogs.com/bigsai/p/14099154.html)
- [一次url访问会经历哪些过程](https://blog.csdn.net/Myxyj/article/details/80027700)
- [TCP 三次握手以及四次挥手的流程。为什么需要三次握手以及四次挥手？](https://blog.csdn.net/qzcsu/article/details/72861891)
- [TCP的拥塞控制具体是怎么实现的？UDP有拥塞控制吗？](https://blog.csdn.net/u014465934/article/details/89202797)
- [是否了解中间人劫持原理](https://segmentfault.com/a/1190000041047662)
- [TCP 与 UDP 在网络协议中的哪一层，他们之间有什么区别？](https://blog.fundebug.com/2019/03/22/differences-of-tcp-and-udp/)
- [HTTP 与 HTTPS 有哪些区别？](https://zhuanlan.zhihu.com/p/72616216)
- [select和epoll区别](https://zhuanlan.zhihu.com/p/272891398)
- [TCP 如何实现数据有序性？](https://blog.csdn.net/dccmxj/article/details/52103800?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3&utm_relevant_index=1)
- [TCP长连接和短连接有那么不同的使用场景？](https://blog.csdn.net/qq_16635171/article/details/104312443)
- [TIME_WAIT时长，为什么？](https://blog.csdn.net/yzpbright/article/details/113566357)
- [什么是零拷贝？](https://www.cnblogs.com/xiaolincoding/p/13719610.html)
- [HTTP 简述 HTTP 的 keepalive 的原理和使用场景](https://cloud.tencent.com/developer/news/696654)
- [Cookie 和 Session 的关系和区别是什么？](https://blog.csdn.net/ailunlee/article/details/95939660)
- [DNS 查询服务器的基本流程是什么？DNS 劫持是什么？](https://xxxixxxx.github.io/2021/02/12/1000-009DNS%20%E6%9F%A5%E8%AF%A2%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9FDNS%20%E5%8A%AB%E6%8C%81%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F/)
- [libevent结构，内部实现](https://segmentfault.com/a/1190000019952979)
- [简述对称与非对称加密的概念](https://www.jianshu.com/p/de50d1489359)
- [epoll中的ET和LT模式](https://cloud.tencent.com/developer/article/1636224)
- [JWT 的原理和校验机制](https://www.jianshu.com/p/f111328ea8c4)
- [TCP 怎么保证可靠传输？](https://segmentfault.com/a/1190000022944999)
- [介绍下proactor和reactor](https://www.zhihu.com/question/26943938)
- [Accept发生在三次握手哪个阶段](https://www.cnblogs.com/wuyepeng/p/9735978.html)
- [RPC 的调用过程](https://waylau.com/remote-procedure-calls/)
- [如何解决 TCP 传输丢包问题？](https://blog.csdn.net/qq_30108237/article/details/107057946)
- [什么是 ARP 协议？简述其使用场景](https://www.cnblogs.com/jmilkfan-fanguiju/p/12789810.html)
- [DDOS 攻击原理，如何防范它？](https://blog.csdn.net/huwei2003/article/details/45476743)
- [如何防止传输内容被篡改？](https://juejin.cn/post/6845166890675863559)
- [介绍下滑动窗口](https://blog.csdn.net/wdscq1234/article/details/52444277)
- [TCP 半连接发生场景](https://www.jianshu.com/p/6a0fcb1008d6)
- [reactor的组成](https://jishuin.proginn.com/p/763bfbd58a63)
- [udp包长度](https://cloud.tencent.com/developer/article/1021196)
- [IP为什么要分片](https://cloud.tencent.com/developer/article/1173790)
- [OSI 七层模型，TCP，IP 属于哪一层？](https://cntsp.github.io/2019/12/11/OSI%E4%B8%83%E5%B1%82%E6%A8%A1%E5%9E%8B%E4%B8%8ETCP-IP%E5%9B%9B%E5%B1%82%E7%BB%93%E6%9E%84%E7%9A%84%E5%8C%BA%E5%88%AB-%E8%81%94%E7%B3%BB/)
- [数据包乱序会处理？](https://www.1024sou.com/article/3015.html)
- [什么是 SYN flood，如何防止这类攻击？](https://zhuanlan.zhihu.com/p/29539671)
- [WebSocket 是如何进行传输的](https://segmentfault.com/a/1190000014643900)
- [为什么需要序列化？有什么序列化的方式？](https://zhuanlan.zhihu.com/p/40462507)
- [有chunked的时候contentlength是什么样子](https://zhuanlan.zhihu.com/p/81955498)
- [如何设计一个可靠的udp](https://zhuanlan.zhihu.com/p/129218784)
- [TCP 中常见的拥塞控制算法有哪些？](https://zhuanlan.zhihu.com/p/76023663)
- [如何设置非阻塞](https://blog.51cto.com/e21105834/2887220)
- [什么是跨域，什么情况下会发生跨域请求？](https://segmentfault.com/a/1190000040485198)
- [Udp的接收缓冲区和发送缓冲区和tcp的区别](https://blog.csdn.net/Swallow_he/article/details/84392285)
- [什么时候需要TCP四次挥手？](https://segmentfault.com/a/1190000039165592)
- [traceroute 有什么作用？](https://zhuanlan.zhihu.com/p/36811672)
- [HTTP 的方法有哪些？](https://www.cnblogs.com/weibanggang/p/9454581.html)
- [TIME_WAIT危害](https://zhuanlan.zhihu.com/p/180543251)
- [select什么情况返回0](https://blog.csdn.net/acs713/article/details/17531827)
- [TCP 的 keepalive 了解吗？说一说它和 HTTP 的 keepalive 的区别？](https://zhuanlan.zhihu.com/p/224595048)
- [简述常见的 HTTP 状态码的含义（30从系统层面上，UDP 如何保证尽量可靠？](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
- [指针与引用的区别](https://blog.nowcoder.net/n/58e03b756d924e0681e6765b30836df1)
- [iPv4 和 iPv6 的区别](https://zhuanlan.zhihu.com/p/71684181)
- [项目中说用到线程池，开多大，为什么运用线程池？](https://blog.51cto.com/u_14057963/2938828)
- [如何设计 API 接口使其实现幂等性？](https://cloud.tencent.com/developer/article/1474941)
- [TCP 的 TIME_WAIT 和 CLOSE_WAIT](https://www.cnblogs.com/cangqinglang/p/13185825.html)
- [HTTP 报文头部的组成结构](https://blog.csdn.net/ulike_MFY/article/details/79550241)
- [RestFul 与 RPC 的区别是什么？RestFul 的优点在哪里？](https://zhuanlan.zhihu.com/p/102760613)
- [从输入 URL 到展现页面的全过程](https://juejin.cn/post/6844904194801926157)
- [什么是 TCP 粘包和拆包？](https://cloud.tencent.com/developer/article/1804413)
- [HTTP 中 GET 和 POST 区别](https://learnku.com/articles/25881)
- [讲一下拥塞控制和流量控制](https://zhuanlan.zhihu.com/p/37379780)
- [TCP 协议的延迟 ACK 和累计应答](https://blog.csdn.net/dangzhangjing97/article/details/81485347)
- [ARP协议工作流程](https://cloud.tencent.com/developer/article/1910772)
- [tcp与udp的区别以及应用场景](https://segmentfault.com/a/1190000021815671)
- [HTTPS 的加密与认证过程](https://juejin.cn/post/6987022244509646862)
- [TCP 中 SYN 攻击是什么？如何防止？](https://www.cnblogs.com/sunsky303/p/11811097.html)
- [HTTP 短链接与长链接的区别](https://www.cnblogs.com/0201zcr/p/4694945.html)
- [seq为1000，发送了1000个数据，下一个seq是多少?](https://blog.csdn.net/HappyRocking/article/details/78198776)
- [chunked块了解？介绍下](https://blog.csdn.net/zam183/article/details/103275025)
- [BGP 协议和 OSPF 协议的区别](https://cloud.tencent.com/developer/article/1898179)
- [简述在四层和七层网络协议中负载均衡的原理](https://cloud.tencent.com/developer/article/1082047)
- [http协议格式，几种方法，功能是什么](https://www.jianshu.com/p/8fe93a14754c)
- [syn如果丢了，重传多少次](https://www.csdn.net/tags/NtTagg1sNjU4NjMtYmxvZwO0O0OO0O0O.html)
- [epoll可读情况有哪些](https://zhuanlan.zhihu.com/p/427512269)

## 操作系统

- [如何调试服务器内存占用过高的问题？](https://blog.51cto.com/u_14490033/3133424)
- [简述操作系统如何进行内存管理](https://www.nowcoder.com/questionTerminal/56f6e3d6a4d94361993eba1d2f4664c8)
- [简述创建进程的流程](https://www.csdn.net/tags/MtjaAgwsNzk0NzMtYmxvZwO0O0OO0O0O.html)
- [简述操作系统中 malloc 的实现原理](https://www.nowcoder.com/questionTerminal/5aae63b290c542f0ab0582d293e6c791)
- [简述僵尸进程和孤儿进程及其危害和处理](https://blog.csdn.net/a745233700/article/details/120715371)
- 两个线程交替打印一个共享变量
- [进程通信中的管道实现原理是什么？](https://segmentfault.com/a/1190000009528245)
- [简述同步与异步的区别，阻塞与非阻塞的区别](https://cloud.tencent.com/developer/article/1592939)
- [malloc 创建的对象在堆还是栈中？](https://www.cnblogs.com/will-boot/p/3302936.html)
- [死锁产生的条件、死锁避免方法](https://blog.csdn.net/zhangpower1993/article/details/89518780)
- [进程的三状态模型、五状态模型、七状态模型](https://hsuloong.github.io/operating-system/process-state-model.html)
- [什么情况下，进程会进行切换？](https://blog.csdn.net/Mind_programmonkey/article/details/116722482)
- [Linux 系统态与用户态，什么时候会进入系统态？](https://blog.csdn.net/Shangxingya/article/details/113664779)
- [Linux 下如何查看端口被哪个进程占用？](https://blog.csdn.net/xiangwanpeng/article/details/78804225)
- [共享内存是如何实现的？](https://cloud.tencent.com/developer/article/1878790)
- 进程有多少种状态？
- [线程间有哪些通信方式？](https://www.cnblogs.com/fanguangdexiaoyuer/p/10834737.html)
- Linux 下如何排查 CPU 以及 内存占用过多？
- [操作系统中，虚拟地址与物理地址之间如何映射？](https://blog.51cto.com/u_15169172/2795114)
- [CPU L1, L2缓存是什么](https://zhuanlan.zhihu.com/p/31875174)
- [信号量是如何实现的？](http://c.biancheng.net/view/1232.html)
- [Linux 如何查看实时的滚动日志？](https://cloud.tencent.com/developer/article/1740866)
- [Linux 进程调度的算法](https://www.cnblogs.com/alantu2018/p/8460451.html)
- [简述分页与分段，分页与分段的区别](https://blog.csdn.net/m0_37637511/article/details/123635892)
- [Linux 虚拟内存的页面置换算法](https://blog.csdn.net/weixin_44164489/article/details/109030383)
- [Linux 中虚拟内存和物理内存有什么区别？有什么优点？](https://zhuanlan.zhihu.com/p/486112490)
- [traceroute 命令的原理](https://zhuanlan.zhihu.com/p/36811672)
- [操作系统是通过什么机制触发系统调用的？](https://zhuanlan.zhihu.com/p/505419261)
- [Linux 零拷贝的原理](https://juejin.cn/post/6995519558475841550)
- [系统调用的过程是怎样的？](https://blog.csdn.net/wangquan1992/article/details/108496821)
- [Linux 的 IO模型有哪些](https://zhuanlan.zhihu.com/p/127170201)
- [简述自旋锁与互斥锁的使用场景](https://cloud.tencent.com/developer/news/695304)
- [多线程和多进程的区别是什么？](https://blog.csdn.net/linraise/article/details/12979473)
- [简述几个常用的 Linux 命令以及他们的功能](https://www.jianshu.com/p/0056d671ea6d)
- [进程空间从高位到低位都有些什么？](https://blog.csdn.net/hezhongla0811/article/details/109552107)
- [简述缓冲区溢出及其危害](https://blog.csdn.net/qq_35642036/article/details/82809845)
- [mmap 的使用场景以及原理](https://juejin.cn/post/6956031662916534279)
- [简述操作系统中的缺页中断](https://www.nowcoder.com/questionTerminal/e5671c51ef704d5291a3af70ffc5c255?orderByHotValue=1&page=12&onlyReference=false)
- Linux 下如何查看 CPU 荷载，正在运行的进程，某个端口对应的进程？
- [进程间有哪些通信方式？](https://cloud.tencent.com/developer/article/1690556)
- [为什么进程切换慢，线程切换快？](https://blog.csdn.net/qq_34417408/article/details/110393655)
- [线程从进程继承了哪些资源？线程独享哪些资源？](https://blog.csdn.net/weixin_41983309/article/details/109029494)
- [Linux 页大小是多少？](https://draveness.me/whys-the-design-linux-default-page/)
- [select, poll, epoll 的使用场景以及区别，epoll 中水平触发以及边缘触发有什么不同？](https://zhuanlan.zhihu.com/p/272891398)

## 数据库

- [数据库三大范式是什么](https://www.cnblogs.com/linjiqin/archive/2012/04/01/2428695.html)

- [mysql有关权限的表都有哪几个](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql-%E6%9C%89%E5%85%B3%E6%9D%83%E9%99%90%E7%9A%84%E8%A1%A8%E9%83%BD%E6%9C%89%E5%93%AA%E5%87%A0%E4%B8%AA)

- [MySQL的binlog有有几种录入格式？分别有什么区别？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql-%E7%9A%84-binlog-%E6%9C%89%E6%9C%89%E5%87%A0%E7%A7%8D%E5%BD%95%E5%85%A5%E6%A0%BC%E5%BC%8F)

- [mysql有哪些数据类型](https://m.html.cn/qa/other/1110119351025.html)

- [MySQL存储引擎MyISAM与InnoDB区别](https://segmentfault.com/a/1190000008227211)

- [MyISAM索引与InnoDB索引的区别？](https://segmentfault.com/a/1190000022206424)

- [InnoDB引擎的4大特性](https://zhuanlan.zhihu.com/p/109528131)

- [存储引擎选择](https://segmentfault.com/a/1190000038935511)

- [什么是索引？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E7%B4%A2%E5%BC%95)

- [索引有哪些优缺点？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E7%B4%A2%E5%BC%95%E6%9C%89%E5%93%AA%E4%BA%9B%E4%BC%98%E7%BC%BA%E7%82%B9)

- [索引使用场景（重点）](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E7%B4%A2%E5%BC%95%E7%9A%84%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)

- [索引有哪几种类型？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E7%B4%A2%E5%BC%95%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E7%B1%BB%E5%9E%8B)

- [索引的数据结构（b树，hash）](https://developer.aliyun.com/ask/281188)

- [索引的基本原理](https://zhuanlan.zhihu.com/p/143659858)

- [索引算法有哪些？](https://zhuanlan.zhihu.com/p/422491906)

- [索引设计的原则？](https://juejin.cn/post/6844904009329803277)

- [创建索引的原则](https://developer.aliyun.com/article/6719)

- [创建索引的三种方式，删除索引](https://juejin.cn/post/6897504840583217159)

- [创建索引时需要注意什么？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95%E6%97%B6%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E4%BB%80%E4%B9%88)

- [使用索引查询一定能提高查询的性能吗？为什么](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E6%9F%A5%E8%AF%A2%E4%B8%80%E5%AE%9A%E8%83%BD%E6%8F%90%E9%AB%98%E6%9F%A5%E8%AF%A2%E7%9A%84%E6%80%A7%E8%83%BD%E5%90%97%E4%B8%BA%E4%BB%80%E4%B9%88)

- [百万级别或以上的数据如何删除](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E7%99%BE%E4%B8%87%E7%BA%A7%E5%88%AB%E6%88%96%E4%BB%A5%E4%B8%8A%E7%9A%84%E6%95%B0%E6%8D%AE%E5%A6%82%E4%BD%95%E5%88%A0%E9%99%A4)

- [前缀索引](https://www.jianshu.com/p/fc80445044cc)

- [什么是最左前缀原则？什么是最左匹配原则](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%80%E5%B7%A6%E5%89%8D%E7%BC%80%E5%8E%9F%E5%88%99%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%80%E5%B7%A6%E5%8C%B9%E9%85%8D%E5%8E%9F%E5%88%99)

- [B树和B+树的区别](https://www.jianshu.com/p/ace3cd6526c4)

- [Hash索引和B+树所有有什么区别或者说优劣呢?](https://book.itheima.net/study/1258677827423715330/1277261541762146305/1290887310354763777)

- [数据库为什么使用B+树而不是B树](https://blog.csdn.net/zhoucheng05_13/article/details/79825246)

- [B+树在满足聚簇索引和覆盖索引的时候不需要回表查询数据，](https://six.club/question/1142)

- [什么是聚簇索引？何时使用聚簇索引与非聚簇索引](https://cloud.tencent.com/developer/article/1541265)

- [非聚簇索引一定会回表查询吗？](https://www.cnblogs.com/xxeleanor/p/14998337.html)

- [联合索引是什么？为什么需要注意联合索引中的顺序？](https://book.itheima.net/study/1258677827423715330/1277261541762146305/1290887364041854978)

- [什么是数据库事务？](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1/9744607)

- [事物的四大特性(ACID)介绍一下?](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#acid)

- [什么是脏读？幻读？不可重复读？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E8%84%8F%E8%AF%BB%E5%B9%BB%E8%AF%BB%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB)

- [什么是事务的隔离级别？MySQL的默认隔离级别是什么？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql-%E4%B8%AD-innodb-%E6%94%AF%E6%8C%81%E7%9A%84%E5%9B%9B%E7%A7%8D%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB%E5%90%8D%E7%A7%B0%E4%BB%A5%E5%8F%8A%E9%80%90%E7%BA%A7%E4%B9%8B%E9%97%B4%E7%9A%84%E5%8C%BA%E5%88%AB)

- [对MySQL的锁了解吗](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql-%E4%B8%AD%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E9%94%81)

- [隔离级别与锁的关系](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB%E4%B8%8E%E9%94%81%E7%9A%84%E5%85%B3%E7%B3%BB)

- [按照锁的粒度分数据库锁有哪些？锁机制与InnoDB锁算法](https://www.cnblogs.com/WangMengyi/p/14009018.html)

- [从锁的类别上分MySQL都有哪些锁呢？像上面那样子进行锁定岂不是有点阻碍并发效率了](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql-%E4%B8%AD%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E9%94%81)

- [MySQL中InnoDB引擎的行锁是怎么实现的？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql%E4%B8%ADinnodb%E5%BC%95%E6%93%8E%E7%9A%84%E8%A1%8C%E9%94%81%E6%98%AF%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)

- [InnoDB存储引擎的锁的算法有三种](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql%E4%B8%ADinnodb%E5%BC%95%E6%93%8E%E7%9A%84%E8%A1%8C%E9%94%81%E6%98%AF%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)

- [什么是死锁？怎么解决？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E6%AD%BB%E9%94%81%E6%80%8E%E4%B9%88%E8%A7%A3%E5%86%B3)

- [数据库的乐观锁和悲观锁是什么？怎么实现的？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E4%B9%90%E8%A7%82%E9%94%81%E5%92%8C%E6%82%B2%E8%A7%82%E9%94%81%E6%98%AF%E4%BB%80%E4%B9%88%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)

- [为什么要使用视图？什么是视图？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E4%BD%BF%E7%94%A8%E8%A7%86%E5%9B%BE%E4%BB%80%E4%B9%88%E6%98%AF%E8%A7%86%E5%9B%BE)

- [视图有哪些特点？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%A7%86%E5%9B%BE%E6%9C%89%E5%93%AA%E4%BA%9B%E7%89%B9%E7%82%B9)

- [视图的使用场景有哪些？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%A7%86%E5%9B%BE%E7%9A%84%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E6%9C%89%E5%93%AA%E4%BA%9B)

- [视图的优缺点](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%A7%86%E5%9B%BE%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9)

- [什么是游标？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E6%B8%B8%E6%A0%87)

- 存储过程与函数

- [什么是存储过程？有哪些优缺点？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B%E6%9C%89%E5%93%AA%E4%BA%9B%E4%BC%98%E7%BC%BA%E7%82%B9)

- [什么是触发器？触发器的使用场景有哪些？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E8%A7%A6%E5%8F%91%E5%99%A8%E8%A7%A6%E5%8F%91%E5%99%A8%E7%9A%84%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E6%9C%89%E5%93%AA%E4%BA%9B)

- [MySQL中都有哪些触发器？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql%E4%B8%AD%E9%83%BD%E6%9C%89%E5%93%AA%E4%BA%9B%E8%A7%A6%E5%8F%91%E5%99%A8)

- 常用SQL语句

- [SQL语句主要分为哪几类](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#sql%E8%AF%AD%E5%8F%A5%E4%B8%BB%E8%A6%81%E5%88%86%E4%B8%BA%E5%93%AA%E5%87%A0%E7%B1%BB)

- [超键、候选键、主键、外键分别是什么？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%B6%85%E9%94%AE%E5%80%99%E9%80%89%E9%94%AE%E4%B8%BB%E9%94%AE%E5%A4%96%E9%94%AE%E5%88%86%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)

- [SQL 约束有哪几种？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#sql-%E7%BA%A6%E6%9D%9F%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D)

- [六种关联查询](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%85%AD%E7%A7%8D%E5%85%B3%E8%81%94%E6%9F%A5%E8%AF%A2)

- [什么是子查询](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BB%80%E4%B9%88%E6%98%AF%E5%AD%90%E6%9F%A5%E8%AF%A2)

- [子查询的三种情况](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%AD%90%E6%9F%A5%E8%AF%A2%E7%9A%84%E4%B8%89%E7%A7%8D%E6%83%85%E5%86%B5)

- [mysql中 in 和 exists 区别](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql%E4%B8%AD-in-%E5%92%8C-exists-%E5%8C%BA%E5%88%AB)

- [varchar与char的区别](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#char-%E5%92%8C-varchar-%E7%9A%84%E5%8C%BA%E5%88%AB)

- [varchar(50)中50的涵义、int(20)中20的涵义](https://blog.csdn.net/qq_40634846/article/details/101837838)

- [FLOAT和DOUBLE的区别是什么？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#float%E5%92%8Cdouble%E7%9A%84%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)

- [drop、delete与truncate的区别](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#dropdelete%E4%B8%8Etruncate%E7%9A%84%E5%8C%BA%E5%88%AB)

- [UNION与UNION ALL的区别？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BD%BF%E7%94%A8-union-%E5%92%8C-union-all-%E6%97%B6%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E4%BA%9B%E4%BB%80%E4%B9%88)

- 如何定位及优化SQL语句的性能问题？创建的索引有没有被使用到?或者说怎么才可以知道这条语句运行很慢的原因？

- [SQL的生命周期？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#sql%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

- [大表数据查询，怎么优化](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%A4%A7%E8%A1%A8%E6%95%B0%E6%8D%AE%E6%9F%A5%E8%AF%A2%E6%80%8E%E4%B9%88%E4%BC%98%E5%8C%96)

- [超大分页怎么处理？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%B6%85%E5%A4%A7%E5%88%86%E9%A1%B5%E6%80%8E%E4%B9%88%E5%A4%84%E7%90%86)

- mysql 分页怎么实现

- [慢查询日志怎么看](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97)

- [关心过业务系统里面的sql耗时吗？统计过慢查询吗？对慢查询都怎么优化过？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%85%B3%E5%BF%83%E8%BF%87%E4%B8%9A%E5%8A%A1%E7%B3%BB%E7%BB%9F%E9%87%8C%E9%9D%A2%E7%9A%84sql%E8%80%97%E6%97%B6%E5%90%97%E7%BB%9F%E8%AE%A1%E8%BF%87%E6%85%A2%E6%9F%A5%E8%AF%A2%E5%90%97%E5%AF%B9%E6%85%A2%E6%9F%A5%E8%AF%A2%E9%83%BD%E6%80%8E%E4%B9%88%E4%BC%98%E5%8C%96%E8%BF%87)

- [为什么要尽量设定一个主键？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%B0%BD%E9%87%8F%E8%AE%BE%E5%AE%9A%E4%B8%80%E4%B8%AA%E4%B8%BB%E9%94%AE)

- [主键使用自增ID还是UUID？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%B8%BB%E9%94%AE%E4%BD%BF%E7%94%A8%E8%87%AA%E5%A2%9Eid%E8%BF%98%E6%98%AFuuid)

- [字段为什么要求定义为not null？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%AD%97%E6%AE%B5%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%B1%82%E5%AE%9A%E4%B9%89%E4%B8%BAnot-null)

- [如果要存储用户的密码散列，应该使用什么字段进行存储？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%A6%82%E6%9E%9C%E8%A6%81%E5%AD%98%E5%82%A8%E7%94%A8%E6%88%B7%E7%9A%84%E5%AF%86%E7%A0%81%E6%95%A3%E5%88%97%E5%BA%94%E8%AF%A5%E4%BD%BF%E7%94%A8%E4%BB%80%E4%B9%88%E5%AD%97%E6%AE%B5%E8%BF%9B%E8%A1%8C%E5%AD%98%E5%82%A8)

- [优化查询过程中的数据访问](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%BC%98%E5%8C%96%E6%9F%A5%E8%AF%A2%E8%BF%87%E7%A8%8B%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE)、优化长难的查询语句、优化特定类型的查询语句、优化关联查询、优化子查询、优化LIMIT分页

  、优化UNION查询、优化WHERE子句

- [数据库优化](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E6%95%B0%E6%8D%AE%E5%BA%93%E7%BB%93%E6%9E%84%E4%BC%98%E5%8C%96)

- [为什么要优化](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BC%98%E5%8C%96)

- [MySQL数据库cpu飙升到500%的话他怎么处理？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#mysql%E6%95%B0%E6%8D%AE%E5%BA%93cpu%E9%A3%99%E5%8D%87%E5%88%B0500%E7%9A%84%E8%AF%9D%E4%BB%96%E6%80%8E%E4%B9%88%E5%A4%84%E7%90%86)

- [大表怎么优化？某个表有近千万数据，CRUD比较慢，如何优化？分库分表了是怎么做的？分表分库了有什么问题？有用到中间件么？他们的原理知道么？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%A4%A7%E8%A1%A8%E6%80%8E%E4%B9%88%E4%BC%98%E5%8C%96%E6%9F%90%E4%B8%AA%E8%A1%A8%E6%9C%89%E8%BF%91%E5%8D%83%E4%B8%87%E6%95%B0%E6%8D%AEcrud%E6%AF%94%E8%BE%83%E6%85%A2%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8%E4%BA%86%E6%98%AF%E6%80%8E%E4%B9%88%E5%81%9A%E7%9A%84%E5%88%86%E8%A1%A8%E5%88%86%E5%BA%93%E4%BA%86%E6%9C%89%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%E6%9C%89%E7%94%A8%E5%88%B0%E4%B8%AD%E9%97%B4%E4%BB%B6%E4%B9%88%E4%BB%96%E4%BB%AC%E7%9A%84%E5%8E%9F%E7%90%86%E7%9F%A5%E9%81%93%E4%B9%88)、垂直分表适用场景、水平分表适用场景、水平切分的缺点

- [MySQL的复制原理以及流程](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%85%AD%E5%A4%8D%E5%88%B6)

- [读写分离有哪些解决方案？](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB)

- [备份计划，mysqldump以及xtranbackup的实现原理](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E5%BA%93.md#%E5%A4%87%E4%BB%BD%E8%AE%A1%E5%88%92mysqldump%E4%BB%A5%E5%8F%8Axtranbackup%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)

- [数据表损坏的修复方式有哪些？](https://blog.csdn.net/qq_35440678/article/details/60321689)

## Redis

[地址](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%AE%9E%E6%88%98/Redis.md#%E9%9D%A2%E8%AF%95%E9%A2%98)

## 消息队列

[地址](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%AE%9E%E6%88%98/Kafka.md#%E9%9D%A2%E8%AF%95%E9%A2%98)



