# sessions

[参考地址](https://learnku.com/docs/build-web-application-with-golang/061-session-and-cookie/3189)

[参考地址2](https://juejin.cn/post/6844903892757512205)

session 和 cookie 是网站浏览中较为常见的两个概念，也是比较难以辨析的两个概念，但它们在浏览需要认证的服务页面以及页面统计中却相当关键。我们先来了解一下 session 和 cookie 怎么来的？考虑这样一个问题：

如何抓取一个访问受限的网页？如新浪微博好友的主页，个人微博页面等。

显然，通过浏览器，我们可以手动输入用户名和密码来访问页面，而所谓的 “抓取”，其实就是使用程序来模拟完成同样的工作，因此我们需要了解 “登陆” 过程中到底发生了什么。

当用户来到微博登陆页面，输入用户名和密码之后点击 “登录” 后浏览器将认证信息 POST 给远端的服务器，服务器执行验证逻辑，如果验证通过，则浏览器会跳转到登录用户的微博首页，在登录成功后，服务器如何验证我们对其他受限制页面的访问呢？因为 **HTTP 协议是无状态的，所以很显然服务器不可能知道我们已经在上一次的 HTTP 请求中通过了验证**。当然，最简单的解决方案就是所有的请求里面都带上用户名和密码，这样虽然可行，但大大加重了服务器的负担（对于每个 request 都需要到数据库验证），也大大降低了用户体验 (每个页面都需要重新输入用户名密码，每个页面都带有登录表单)。既然直接在请求中带上用户名与密码不可行，那么就只有在服务器或客户端保存一些类似的可以代表身份的信息了，所以就有了 cookie 与 session。

cookie，简而言之就是**在本地计算机保存一些用户操作的历史信息**（当然包括登录信息），并在用户再次访问该站点时浏览器**通过 HTTP 协议将本地 cookie 内容发送给服务器**，从而完成验证，或继续上一步操作。

![img](https://cdn.learnku.com/build-web-application-with-golang/images/6.1.cookie2.png?raw=true)

session，简而言之就是在**服务器上保存用户操作的历史信息**。服务器使用 session id 来标识 session，**session id 由服务器负责产生，保证随机性与唯一性**，相当于一个随机密钥，避免在握手或传输中暴露用户真实密码。但该方式下，仍然需要将发送请求的客户端与 session 进行对应，所以可以借助 cookie 机制来获取客户端的标识（即 session id），也可以通过 GET 方式将 id 提交给服务器。

![6.1.session.png](https://cdn.learnku.com/build-web-application-with-golang/images/6.1.session.png?raw=true)

**cookie 是有时间限制的**，根据生命期不同分成两种：**会话 cookie** 和**持久 cookie**；

如果**不设置过期时间**，则表示这个 cookie 的生命周期为**从创建到浏览器关闭为止**，只要关闭浏览器窗口，cookie 就消失了。这种生命期为浏览会话期的 cookie 被称为会话 cookie。会话 cookie 一般不保存在硬盘上而是保存在内存里。

如果**设置了过期时间** (setMaxAge (606024))，**浏览器就会把 cookie 保存到硬盘上，关闭后再次打开浏览器，这些 cookie 依然有效直到超过设定的过期时间。**存储在硬盘上的 cookie 可以在不同的浏览器进程间共享，比如两个 IE 窗口。而对于保存在内存的 cookie，不同的浏览器有不同的处理方式。

**session 机制是一种服务器端的机制，服务器使用一种类似于散列表的结构 (也可能就是使用散列表) 来保存信息。**

但程序需要为某个客户端的请求创建一个 session 的时候，服务器首先检查这个客户端的请求里是否包含了一个 session 标识－称为 session id，如果已经包含一个 session id 则说明以前已经为此客户创建过 session，服务器就按照 session id 把这个 session 检索出来使用 (如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的 session 对象，但用户人为地在请求的 URL 后面附加上一个 JSESSION 的参数)。如果客户请求不包含 session id，则为此客户创建一个 session 并且同时生成一个与此 session 相关联的 session id，这个 session id 将在本次响应中返回给客户端保存。

session 机制本身并不复杂，然而其实现和配置上的灵活性却使得具体情况复杂多变。这也要求我们不能把仅仅某一次的经验或者某一个浏览器，服务器的经验当作普遍适用的。

## gorilla/sessions

[`gorilla/sessions`](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgorilla%2Fsessions)是 gorilla web 开发工具包中管理 session 的库。它提供了基于 cookie 和本地文件系统的 session。同时预留扩展接口，可以使用其它的后端存储 session 数据。

[地址](https://juejin.cn/post/6989043108012884004)

