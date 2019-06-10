# 浏览器原理之http协议s版

浏览器是[b/s](https://www.cnblogs.com/zl1991/p/6951788.html)结构最主要的组成部分，是新时代对c/s结构的一种进化，这种模式统一了[客户端](https://baike.baidu.com/item/客户端/101081)，将系统功能实现的核心部分集中到[服务器](https://baike.baidu.com/item/服务器/100571)上。

目前使用的主流浏览器有五个：Internet Explorer、Firefox、Safari、Chrome 浏览器和 Opera。同样一行代码，不同的浏览器会有不同的解读和呈现，而且速度也不一样，给人的体验也不一样。而决定如何解释和执行代码的核心就是浏览器的内核。

对于前端来说，浏览器是必须接触的，也是个黑盒一般的存在。

在前端面试中经常会有这样一道题：输入一个url到浏览器页面展示，中间发生了什么。

简要回答：

1. url使用dns解析成ip，

2. 浏览器使用http协议或者https协议，向服务器发送请求

3. tcp三次握手

4. 服务器处理

5. 请求回来的html代码经过解析，构建dom树

6. 计算dom上css属性，根据css属性逐个渲染得到内存中的位图。

7. 绘制界面。

   

   首先呢我们先介绍一下网络通信部分：

   ##http协议

   要确定http协议属于应用层协议，可以说是iso模型中最顶层的，它建立在tcp协议之上。

   ### http特点：

   1. 请求应答模式，每个请求建立一次连接。
   2. 无连接 指的是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即完成后断开连接。采用这种方式可以节省传输时间。

   ![http长连接](https://img-blog.csdn.net/20180511155036709?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzY3MjE2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   3. 无状态 是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。即我们给服务器发送 HTTP 请求之后，服务器根据请求，会给我们发送数据过来，但是，发送完，不会记录任何信息。（淘宝登陆记录购物车信息，cookie，session）

   4. http请求是明文的。HTTP数据在网络中裸奔，安全性很有问题，导致数据泄露、数据篡改、流量劫持、钓鱼攻击等安全问题的重要原因。解决办法就是TLS协议，即https。内容容易被查看，无法保证数据完整性，无法保证数据来源的可靠性

### http协议现行标准

http协议是基于tcp协议出现的，tcp协议位于传输层，提供一个**可靠**的双向通信通道.

http 0.9 短连接，接收数据就断开连接。

http1.1 半双工，长连接。

http2 全双工,一个消息发送后不用等待接受,第二个消息可以直接发送.多路复用，帧和流传输。

在0.9版本里，每次请求一次都要建立一个http请求，请求过程占用很多的时间，现在网站各种图和文字都是要发请求，效率很低，那么1.1版本出现的keep-alive就是有效的解决了这一问题，一定时间内，同域名只建立一次http请求，这样效率大大提升。

但是，1.1版本仍有缺点：

1. 串型文件传输，请求a文件，b文件要等待，请求响应完再请求。

2. 连接数浏览器和服务器都有限制

   

   所以又出现了http2.0版本，因为请求响应都是基于文本的，传输helloworld，只能h-d传输，不能并行传输，因为没有顺序。2.0引入二进制的帧和流，帧就是对数据进行顺序标识，流进行并行传输。

   因为有了流的概念，所以同一域名不管多少文件，都只建立一条tcp连接。（和长连接区分，长：是一条http能一直保持，流：是一个tcp上加n个http请求）

![img](https://img-blog.csdnimg.cn/20190124110554546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlbG9uZ2Zvcg==,size_16,color_FFFFFF,t_70)



### http协议格式

请求报文和响应报文。分开来讲述：

#### 请求/响应报文

请求的第一行叫请求行，它包含方法，路径和协议版本。

![请求报文](https://images0.cnblogs.com/i/116165/201407/121521541766071.png)

响应的第一行是响应行，协议版本 状态码和状态文本

紧随请求/响应行的是请求/响应头，它类似键值对，相当于http的配置或者规定，

接着是个空行，然后是请求/响应体，相当于传输的数据。

![响应报文](https://images0.cnblogs.com/i/116165/201407/111156110203396.png)

#### http method & status

其实http最主要的是三个一个是请求方法，一个是响应状态码，还有一个在下面。

简单的说一下就是get和post请求。一般呢根据restful规定，get用于获取数据，post用于发送给后端数据（表单）。

1xx 临时回应，客户端继续请求

2xx请求成功 200

3xx请求目标变化 301永久重定向 302临时重定向 304缓存

4xx客户端错误 403没权限 404请求页面不存在（可能是路径输入错误）

5xx 服务器错误 500服务器错误 503服务器暂时故障，请稍后再试　（please call me later）

#### http头文件

比如：connection：keep-alive，cookie

![这里写图片描述](https://img-blog.csdn.net/20180719094739178?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### https

我们说http明文的特点，所以极其不安全，那么解决安全的办法就是在http和tcp层之间加安全层套接字，可以理解为在把http建立起来的时候对数据加一层处理，现在基本都是用tsl。为了解决http明文的问题。

![img](https://images0.cnblogs.com/i/116165/201407/111703047392802.png)

但https也有缺点

1. 需要申请购买，成本。
2. 性能 HTTPS连接缓存不如HTTP高效，流量成本高。
3. HTTPS连接服务器端资源占用高很多。
4. 对网站的响应速度有影响，影响用户体验。