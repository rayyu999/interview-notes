# 其它内容



## Cookie/Session/Application

Cookie和Session都是客户端与服务器之间保持状态的解决方案，具体来说，Cookie机制采用的是在客户端保持状态的方案，而Session机制采用的是在服务器端保持状态的方案。

Application（Java Web中的ServletContext）：与一个Web应用程序相对应，为应用程序提供了一个全局的状态，所有客户都可以使用该状态。

### Cookie 及其相关API

Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie，而客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器，服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

![](https://images.yingwai.top/picgo/20210901100600.png)

![](https://images.yingwai.top/picgo/20210901100604.png)

### Session 及其相关API

同样地，会话状态也可以保存在服务器端。客户端请求服务器，如果服务器记录该用户状态，就获取Session来保存状态，这时，如果服务器已经为此客户端创建过session，服务器就按照sessionid把这个session检索出来使用；如果客户端请求不包含sessionid，则为此客户端创建一个session并且生成一个与此session相关联的sessionid，并将这个sessionid在本次响应中返回给客户端保存。保存这个sessionid的方式可以采用 cookie机制 ，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器；若浏览器禁用Cookie的话，可以通过 **URL重写机制** 将sessionid传回服务器。

![](https://images.yingwai.top/picgo/20210901100656.png)

### Session 与 Cookie 对比

- **实现机制：**Session的实现常常依赖于Cookie机制，通过Cookie机制回传SessionID；
- **大小限制：**Cookie有大小限制并且浏览器对每个站点也有cookie的个数限制，Session没有大小限制，理论上只与服务器的内存大小有关；
- **安全性：**Cookie存在安全隐患，通过拦截或本地文件找得到cookie后可以进行攻击，而Session由于保存在服务器端，相对更加安全；
- **服务器资源消耗：**Session是保存在服务器端上会存在一段时间才会消失，如果session过多会增加服务器的压力。



## DDos（Distributed Denial of Service）攻击

**客户端不断进行请求链接会怎样？**服务器端会为每个请求创建一个链接，并向其发送确认报文，然后等待客户端进行确认。

### 攻击过程

- 客户端向服务端发送请求链接数据包
- 服务端向客户端发送确认数据包
- 客户端不向服务端发送确认数据包，服务器一直等待来自客户端的确认

### 预防（没有彻底根治的办法，除非不使用TCP）

- 限制同时打开SYN半链接的数目
- 缩短SYN半链接的Time out 时间
- 关闭不必要的服务



## 常见面试题

* [计算机网络面试问题集锦_qq_39322743的博客-CSDN博客_计算机网络面试题](https://blog.csdn.net/qq_39322743/article/details/79700863?ops_request_misc=%7B%22request%5Fid%22%3A%22161542868916780264098812%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=161542868916780264098812&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-4-79700863.first_rank_v2_pc_rank_v29&utm_term=计算机网络面试)
* [计算机网络核心知识点总结&面试笔试要点_HenryLei的博客-CSDN博客_计算机网络核心知识点](https://blog.csdn.net/huanglei305/article/details/99712771?utm_medium=distribute.pc_feed_404.none-task-blog-BlogCommendFromMachineLearnPai2-11.nonecase&dist_request_id=&depth_1-utm_source=distribute.pc_feed_404.none-task-blog-BlogCommendFromMachineLearnPai2-11.nonecas)
* [【学习】计算机网络重点知识点面试突击_小听歌的博客-CSDN博客](https://blog.csdn.net/qq_21407523/article/details/114195378)
* [TCP CLOSE_WAIT 过多解决方案_王卫东 博客-CSDN博客_close wait 过多原因](https://blog.csdn.net/wwd0501/article/details/78674170)

