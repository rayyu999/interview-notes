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

