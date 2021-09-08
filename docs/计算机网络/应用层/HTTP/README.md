# HTTP



## 基础概念

### 请求和响应报文

客户端发送一个请求报文给服务器，服务器根据请求报文中的信息进行处理，并将处理结果放入响应报文中返回给客户端。

请求报文结构：

- 第一行是包含了请求方法、URL、协议版本；
- 接下来的多行都是请求首部 Header，每个首部都有一个首部名称，以及对应的值。
- 一个空行用来分隔首部和内容主体 Body
- 最后是请求的内容主体

```
GET http://www.example.com/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cache-Control: max-age=0
Host: www.example.com
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947+gzip"
Proxy-Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 xxx

param1=1&param2=2
```

响应报文结构：

- 第一行包含协议版本、状态码以及描述，最常见的是 200 OK 表示请求成功了
- 接下来多行也是首部内容
- 一个空行分隔首部和内容主体
- 最后是响应的内容主体

```html
HTTP/1.1 200 OK
Age: 529651
Cache-Control: max-age=604800
Connection: keep-alive
Content-Encoding: gzip
Content-Length: 648
Content-Type: text/html; charset=UTF-8
Date: Mon, 02 Nov 2020 17:53:39 GMT
Etag: "3147526947+ident+gzip"
Expires: Mon, 09 Nov 2020 17:53:39 GMT
Keep-Alive: timeout=4
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Proxy-Connection: keep-alive
Server: ECS (sjc/16DF)
Vary: Accept-Encoding
X-Cache: HIT

<!doctype html>
<html>
<head>
    <title>Example Domain</title>
	// 省略... 
</body>
</html>
```



### URL

HTTP 使用 URL（ **U**niform **R**esource **L**ocator，统一资源定位符）来定位资源，它是 URI（**U**niform **R**esource **I**dentifier，统一资源标识符）的子集，URL 在 URI 的基础上增加了定位能力。URI 除了包含 URL，还包含 URN（Uniform Resource Name，统一资源名称），它只是用来定义一个资源的名称，并不具备定位该资源的能力。例如 urn:isbn:0451450523 用来定义一个书籍名称，但是却没有表示怎么找到这本书。



## HTTP/1.1、HTTP/2、HTTP/3 演变

### HTTP/1.1 对比 HTTP/1.0

性能上的改进：

* 使用 TCP 长连接的方式改善了 HTTP/1.0 短连接造成的性能开销。
* 支持管道（pipeline）网络传输，只要第一个请求发出去了，不必等其回来，就可以发第二个请求出去，可以减少整体的响应时间。

HTTP/1.1 的性能瓶颈：

* 请求/响应头部（Header）未经压缩就发送，首部信息越多延迟越大。只能压缩 Body 的部分；
* 发送冗长的首部。每次互相发送相同的首部造成的浪费较多；
* 服务器是按请求的顺序相应的，如果服务器响应慢，会招致客户端一直请求不到数据，也就是队头阻塞；
* 没有请求优先级控制；
* 请求只能从客户端开始，服务器只能被动响应。

### HTTP/2 对比 HTTP/1.1

HTTP/2 协议基于 HTTPS，安全性有保障。

性能上的改进：

* **头部压缩：**HTTP/2 会压缩头（Header），如果你同时发出多个请求，他们的头是一样的或是相似的，那么协议会帮你消除重复的部分。

  （`HPACK` 算法：在客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段，只发送索引号，达到提高速度的目的）

* **二进制格式：**HTTP/2 不像 HTTP/1.1 里的纯文本形式的报文，而是全面采用了二进制格式，头信息和数据体都是二进制，并且统称为帧（frame）：头信息帧和数据帧。

  ![](https://images.yingwai.top/picgo/20210812172530.png)

  收到报文后无需再将明文的报文转成二进制，而是直接解析二进制报文，增加了数据传输的效率。

* **数据流：**HTTP/2 的数据包不是按顺序发送的，同一个连接里面连续的数据包可能属于不同的回应。

  每个请求或回应的所有数据包，称为⼀个数据流（ Stream ）。每个数据流都标记着⼀个独⼀⽆⼆的编号，其中规
  定客户端发出的数据流编号为奇数， 服务器发出的数据流编号为偶数。

  客户端还可以指定数据流的优先级。优先级⾼的请求，服务器就先响应该请求。

  ![](https://images.yingwai.top/picgo/202109080948713.png ':size=60%')

* **多路复用：**HTTP/2 是可以在⼀个连接中并发多个请求或回应，⽽不⽤按照顺序⼀⼀对应。

  移除了 HTTP/1.1 中的串⾏请求，不需要排队等待，也就不会再出现「队头阻塞」问题，降低了延迟，⼤幅度提⾼了连接的利⽤率。

  举例来说，在⼀个 TCP 连接⾥，服务器收到了客户端 A 和 B 的两个请求，如果发现 A 处理过程⾮常耗时，于是就回应 A 请求已经处理好的部分，接着回应 B 请求，完成后，再回应 A 请求剩下的部分。

  ![](https://images.yingwai.top/picgo/202109080949573.png)

* **服务器推送：**HTTP/2 还在⼀定程度上改善了传统的「请求 - 应答」⼯作模式，服务不再是被动地响应，也可以主动向客户端发送消息。

  举例来说，在浏览器刚请求 HTML 的时候，就提前把可能会⽤到的 JS、CSS ⽂件等静态资源主动发给客户端，减少延时的等待，也就是服务器推送（Server Push，也叫 Cache Push）。

### HTTP/3 对比 HTTP/2

HTTP/2 主要的问题在于，多个 HTTP 请求在复⽤⼀个 TCP 连接，下层的 TCP 协议是不知道有多少个 HTTP 请求的。所以⼀旦发⽣了丢包现象，就会触发 TCP 的重传机制，这样在⼀个 TCP 连接中的**所有的 HTTP 请求都必须等待这个丢了的包被重传回来。**

* HTTP/1.1 中的管道（ pipeline）传输中如果有⼀个请求阻塞了，那么队列后请求也统统被阻塞住了；
* HTTP/2 多个请求复⽤⼀个TCP连接，⼀旦发⽣丢包，就会阻塞住所有的 HTTP 请求。

这都是基于 TCP 传输层的问题，所以 **HTTP/3 把 HTTP 下层的 TCP 协议改成了 UDP**！

![](https://images.yingwai.top/picgo/202109080952743.png)

UDP 发⽣是不管顺序，也不管丢包的，所以不会出现 HTTP/1.1 的队头阻塞 和 HTTP/2 的⼀个丢包全部重传问题。

UDP 是不可靠传输的，但基于 UDP 的 QUIC 协议 可以实现类似 TCP 的可靠性传输。

* QUIC 有⾃⼰的⼀套机制可以保证传输的可靠性的。当某个流发⽣丢包时，只会阻塞这个流，其他流不会受到影响。
* TLS3 升级成了最新的 1.3 版本，头部压缩算法也升级成了 QPack 。
* HTTPS 要建⽴⼀个连接，要花费 6 次交互，先是建⽴三次握⼿，然后是 TLS/1.3 的三次握⼿。QUIC 直接把以往的 TCP 和 TLS/1.3 的 6 次交互合并成了 3 次，减少了交互次数。

![](https://images.yingwai.top/picgo/202109080953250.png)

所以 QUIC 是⼀个在 UDP 之上的伪 TCP + TLS + HTTP/2 的多路复⽤的协议。

QUIC 是新协议，对于很多⽹络设备，根本不知道什么是 QUIC，只会当做 UDP，这样会出现新的问题。所以HTTP/3 现在普及的进度⾮常的缓慢，不知道未来 UDP 是否能够逆袭 TCP。



## HTTP 方法

客户端发送的 **请求报文** 第一行为请求行，包含了方法字段。

### GET

> 获取资源

当前网络请求中，绝大部分使用的是 GET 方法。

### HEAD

> 获取报文首部

和 GET 方法类似，但是不返回报文实体主体部分。

主要用于确认 URL 的有效性以及资源更新的日期时间等。

### POST

> 传输实体主体

POST 主要用来传输数据，而 GET 主要用来获取资源。

### PUT

> 上传文件

由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。

```html
PUT /new.html HTTP/1.1
Host: example.com
Content-type: text/html
Content-length: 16

<p>New File</p>
```

### PATCH

> 对资源进行部分修改

PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改。

```
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

### DELETE

> 删除文件

与 PUT 功能相反，并且同样不带验证机制。

