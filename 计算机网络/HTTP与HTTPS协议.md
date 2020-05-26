# HTTP协议

## **超文本传输协议**

1. HTTP是基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）
2. HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候，就成了我们常说的HTTPS。如下图![http/https](https://user-gold-cdn.xitu.io/2018/4/16/162cd3df803538f6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
3. HTTP是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型。HTTP是一个无状态的协议。
4. HTTP默认的端口号为80，HTTPS的端口号为443。



## HTTP工作流程

一次HTTP操作称为一个事务，其工作过程大概如下：

1. 用户在浏览器中键入需要访问网页的URL或者点击某个网页中链接；
2. 浏览器根据URL中的域名，通过DNS解析出目标网页的IP地址；

```
浏览器请求这个页面：http://hackr.ip/index.html在这一步，需要域名系统DNS解析域名hackr.ip,得主机的IP地址 20X.189.105.112。然后将上面结合本机自己的信息，封装成一个http请求数据包
```

1. 在HTTP开始工作前，客户端首先会通过TCP/IP协议来和服务端建立链接（TCP三次握手）
2. 建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可内容。
3. 服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。
4. 一般情况下，一旦Web服务器向浏览器发送了请求数据，它就要关闭TCP连接，然后如果浏览器或者服务器在其头信息加入了这行代码：`Connection:keep-alive`，TCP连接在发送后将仍然保持打开状态，于是，浏览器可以继续通过相同的连接发送请求。保持连接节省了为每个请求建立新连接所需的时间，还节约了网络带宽。



## HTTP请求报文

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：

请求行（request line）、请求头部（header）、请求体组成，下图给出了请求报文的一般格式。

![HTTP请求](https://user-gold-cdn.xitu.io/2018/4/16/162cdac93015af8f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080311162578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx,size_16,color_FFFFFF,t_70)

请求行:
    方法:
        GET 获取资源
        POST 向服务器端发送数据，传输实体主体
        PUT 传输文件
        HEAD 获取报文首部
        DELETE 删除文件
        OPTIONS 询问支持的方法
        TRACE 追踪路径
    协议/版本号
    URL
请求头:
    通用首部(General Header)
    请求首部(Request Header)
    响应首部(Response Header)
    实体首部(Entity Header Fields)
请求体

## HTTP状态码

| 类别 | 原因                           |
| :--- | :----------------------------- |
| 1XX  | Informational(信息性状态码)    |
| 2XX  | Success(成功状态码)            |
| 3XX  | Redirection(重定向)            |
| 4XX  | Client Error(客户端错误状态码) |
| 5XX  | Server Error(服务器错误状态吗) |

### 2  成功

* 200(OK 客户端发过来的数据被正常处理
* 204(Not Content 正常响应，没有实体
* 206(Partial Content 范围请求，返回部分数据，响应报文中由Content-Range指定实体内容

### 3  重定向

* 301(Moved Permanently) 永久重定向
* 302(Found) 临时重定向，规范要求，方法名不变，但是都会改变
* 303(See Other) 和302类似，但必须用GET方法
* 304(Not Modified) 状态未改变， 配合(If-Match、If-Modified-Since、If-None_Match、If-Range、If-Unmodified-Since)
* 307(Temporary Redirect) 临时重定向，不该改变请求方法



### 4  客户端错误

* 400(Bad Request) 请求报文语法错误
* 401 (unauthorized) 需要认证
* 403(Forbidden) 服务器拒绝访问对应的资源
* 404(Not Found) 服务器上无法找到资源

### 5  服务器端错误

* 500(Internal Server Error)服务器故障
* 503(Service Unavailable) 服务器处于超负载或正在停机维护



# HTTPS协议



## HTTPS简介

基于HTTP协议，通过SSL或TLS提供加密处理数据、验证对方身份以及数据完整性保护

HTTPS是身披SSL外壳的HTTP。HTTPS是一种通过计算机网络进行安全通信的传输协议，经由HTTP进行通信，利用SSL/TLS建立全信道，加密数据包。HTTPS使用的主要目的是提供对网站服务器的身份认证，同时保护交换数据的隐私与完整性。

PS:TLS是传输层加密协议，前身是SSL协议，由网景公司1995年发布，有时候两者不区分

## HTTPS特点

1. 内容加密：采用混合加密技术，中间者无法直接查看明文内容
2. 验证身份：通过证书认证客户端访问的是自己的服务器
3. 保护数据完整性：防止传输的内容被中间人冒充或者篡改





![这里写图片描述](https://img-blog.csdn.net/20180719094756330?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)