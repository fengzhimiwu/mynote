####1、http状态码都有哪些

> 一二三四五原则:（即一：消息系列；二：成功系列； 三：重定向系列；四：请求错误系列；五：服务器端错误系列。）1** 信息，服务器收到请求，需要请求者继续执行操作
> 2** 成功，操作被成功接收并处理
> 200 OK 请求成功。一般用于GET与POST请求
> 3** 重定向，需要进一步的操作以完成请求
> 300 Multiple Choices 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择
> 301 Moved Permanently 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
> 302 Found 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
> 303 See Other 查看其它地址。与301类似。使用GET和POST请求查看
> 304 Not Modified 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源
> 305 Use Proxy 使用代理。所请求的资源必须通过代理访问
> 4** 客户端错误，请求包含语法错误或无法完成请求
> 400 Bad Request 客户端请求的语法错误，服务器无法理解
> 401 Unauthorized 请求要求用户的身份认证
> 402 Payment Required 保留，将来使用
> 403 Forbidden 服务器理解请求客户端的请求，但是拒绝执行此请求
> 404 Not Found 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面
> 405 Method Not Allowed 客户端请求中的方法被禁止
> 406 Not Acceptable 服务器无法根据客户端请求的内容特性完成请求
> 407 Proxy Authentication Required 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权
> 408 Request Time-out 服务器等待客户端发送的请求时间过长，超时
> 5** 服务器错误，服务器在处理请求的过程中发生了错误
> 500 Internal Server Error 服务器内部错误，无法完成请求
> 501 Not Implemented 服务器不支持请求的功能，无法完成请求
> 502 Bad Gateway 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应
> 503 Service Unavailable 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
> 504 Gateway Time-out 充当网关或代理的服务器，未及时从远端服务器获取请求
> 505 HTTP Version not supported 服务器不支持请求的HTTP协议的版本，无法完成处理

####2、Http 和Https的区别

>第一：http是超文本传输协议，信息是明文传输，https是具有安全性的ssl加密传输协议
第二：http和https使用的是完全不同的连接方式，端口也不一样，前者80 或者443
第三：http连接很简单，是无状态的。https协议是由ssl+http协议构建的可进行加密传输，身份认证的网络协议。

####3、什么方法来加快页面的加载速度

>1，用到服务器资源时在打开，不用时，立即关闭服务器资源。
2，数据库添加索引
3，页面可生成静态
4，图片等大文件单独放在一个服务器
5，能不查询数据库的尽量不去数据取数据，可以放在缓存中。

####4、对于大流量的网站,采用什么样的方法来解决访问量问题?

>①.有效使用缓存，增加缓存命中率.
>②.使用负载均衡.
>③.对静态文件使用CDN进行存储和加速.
>④.想法减少数据库的使用.
>⑤.查看出现统计的瓶颈在哪里.
>反向代理

####5、AJAX的优势是什么？

ajax是异步传输技术，可以通过javascript实现，也可以通过JQuery框架实现，实现局部刷新，减轻了服务器的压力，也提高了用户体验。

对json数据格式的理解？
JSON(javascript object Notation)是一种轻量级的数据交换格式，json数据格式固定，可以被多种语言用作数据的传递。

长连接、短连接的区别和使用
长连接：client 方与 server 方先建立连接，连接建立后不断开，然后再进行报文发送和接收。这种方式下由于通讯连接一直存在。此种方式常用于 P2P 通信。

短连接：Client 方与 server 每进行一次报文收发交易时才进行通讯连接，交易完毕后立即断开连接。此方式常用于一点对多点通讯。C/S 通信。

长连接与短连接的使用时机：

长连接：

短连接多用于操作频繁，点对点的通讯，而且连接数不能太多的情况。每个 TCP 连 接的建立都需要三次握手，每个 TCP 连接的断开要四次握手。如果每次操作都要建立连接然后再操作的话处理速度会降低，所以每次操作下次操作时直接发送数据 就可以了，不用再建立 TCP 连接。例如：数据库的连接用长连接，如果用短连接频繁的通信会造成 socket 错误，频繁的 socket 创建也是对资源的浪 费。

短连接：

web 网站的 http 服务一般都用短连接。因为长连接对于服务器来说要耗费一定 的资源。像 web 网站这么频繁的成千上万甚至上亿客户端的连接用短连接更省一些资源。试想如果都用长连接，而且同时用成千上万的用户，每个用户都占有一个 连接的话，可想而知服务器的压力有多大。所以并发量大，但是每个用户又不需频繁操作的情况下需要短连接。

socket 连接步骤
Socket（套接字）概念

套接字（socket）是通信的基石，是支持 TCP/IP 协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议，本地主机的 IP 地址，本地进程的协议端口，远地主机的 IP 地址，远地进程的协议端口。

Socket 连接过程

建立 Socket 连接至少需要一对套接字，其中一个运行于客户端，称为 ClientSocket ，另一个运行于服务器端，称为 ServerSocket

套接字之间的连接过程可以分为三个步骤：服务器监听，客户端请求，连接确认。

服务器监听：是服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态。

客户端请求：是指由客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

连接确认：是指当服务器端套接字监听到或者说接收到客户端套接字的连接请求，它就响应客户端

套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，连接就建立好了。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。



10. TCP 协议，三次握手、四次挥手
TCP 协议 (Transmission Control Protocol) 是主机对主机层的传输控制协议，提供可靠的连接服务，采用三次握手确认建立一个连接，四次挥手断开连接。

位码即 tcp 标志位，有 6 种标示:

SYN (synchronous 建立联机) 同步

ACK (acknowledgement 确认)

PSH (push 传送)

FIN (finish 结束)

RST (reset 重置)

URG (urgent 紧急)

如何优化前端性能
1) 页面内容的优化

a) 降低请求数

合并 css、js 文件，集成 CSS 图片

b) 减少交互通信量

压缩技术：压缩 css、js 文件，优化图像，减少 cookie 体积；

合理利用缓存：使用外部 js/css 文件，缓存 ajax；

减少不必要的通信量：剔除无用脚本和样式、推迟加载内容、使用 GET 请求

c) 合理利用 “并行” 尽量避免重定向

慎用 Iframe 样式表置于顶部 脚本放到样式后面加载

d) 节约系统消耗

避免 CSS 表达式、滤镜

2) 服务器的优化

a) b)

c)

d)



19. yahoo 的 34 条前端优化法则
减少 HTTP 请求、利用 CDN 技术、 设置头文件过期或者静态缓存、Gzip 压缩、把 CSS 放顶部、 把 JS 放底部、避免 CSS 表达式、将 JS 和 CSS 外链、减少 DNS 查找、减小 JS 和 CSS 的体积、 避免重定向、删除重复脚本、 配置 ETags、缓存 Ajax、尽早的释放缓冲、

用 GET 方式进行 AJAX 请求、延迟加载组件、 预加载组件、减少 DOM 元素数量、跨域分离组件、

减少 iframe 数量、不要出现 404 页面、减小 Cookie、 对组件使用无 Cookie 的域名、减少 DOM 的访问次数、开发灵活的事件处理句柄、使用 <link> 而非 @import、避免过滤器的使用、优化图片、优化 CSS Sprites、 不要在 HTML 中缩放图片、缩小 favicon. ico 的大小并缓存它、保证组件在 25K 以下、将组件打包进一个多部分的文档中

304状态码详解

https://blog.csdn.net/Dongguabai/article/details/84323511
个人理解本地浏览器如果有缓存会自动带一些校验参数传递给服务器，服务器根据参数校验这个接口有没有变动过，有变动正常请求数据，返回200。没变动返回304，客户端展示缓存数据
磁盘满了怎么查询

#### [html5新特性总结](https://www.cnblogs.com/binguo666/p/10928907.html)

html5总的来说比html4多了十个新特性，但其不支持ie8及ie8以下版本的浏览器
一、语义标签
二、增强型表单
三、视频和音频
四、Canvas绘图
五、SVG绘图
六、地理定位
七、拖放API
八、WebWorker
九、WebStorage
十、WebSocket

####web开发中一般有哪些安全问题并简述一下原理和处理方式。

xss跨站脚本攻击
csrf跨站请求伪造（cross-site request forgery）
csrf实际上是用户登录网站a，然后登录钓鱼网站b的时候，不知情的情况下访问了b提供的访问a的链接。
防御方法
1.http refer,refer里记录来源网址，但是有的浏览器（ie ff2）可以自定义修改
2.请求地址里加上token验证，token存在session中，每次请求都带上token，验证用户合法性。缺点是改动量大，token也有可能会被获取到，有些论坛允许用户添加自己的网站链接，有可能也会在连接上带上用户token。
3.http头里增加自定义属性，通过xmlhttprequest类添加

####如何处理tcp粘包问题

https://www.jianshu.com/p/bb1377163ca7
TCP 是流式协议没有消息边界，客户端向服务器端发送一次数据，可能会被服务器端分成多次收到。
客户端向服务器端发送多条数据。服务器端可能一次全部收到。 保证传输的可靠性，顺序。
TCP拥有拥塞控制，所以数据包可能会延后发送。
TCP粘包
介绍
TCP 粘包是指发送方发送的若干包数据 到 接收方接收时粘成一包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾。
原因
发送方：发送方需要等缓冲区满才发送出去，造成粘包
接收方：接收方不及时接收缓冲区的包，造成多个包接收
swoole处理粘包
EOF结束协议：
EOF切割需要遍历整个数据包的内容，查找EOF，因此会消耗大量CPU资源。上手比较简单
固定包头+包体协议
长度检测协议，只需要计算一次长度，数据处理仅进行指针偏移，性能非常高

26. HTTP 请求头信息和响应头信息
请求头信息

POST /scp1.1.0/prs/new_rnaseqtask/run_go HTTP/1.1

Host: 172.30.4.102

User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:6.0) Gecko/20100101 Firefox/6.0

Accept: /

Accept-Language: zh-cn,zh;q=0.5

Accept-Encoding: gzip, deflate

Accept-Charset: GB2312,utf-8;q=0.7,*;q=0.7

Connection: keep-alive

Content-Type: application/x-www-form-urlencoded; charset=UTF-8

X-Requested-With: XMLHttpRequest

Referer: http://172.30.4.102/scp1.1.0/index.php/prs... Content-Length: 1819

Cookie:

ci_session=a%3A4%3A%7Bs%3A10%3A%22session_id%22%3Bs%3A32%3A%22e31556053ff9407a454f6a1e146d43eb%22%3Bs%3A10%3A%22ip_address%22%3Bs%3A12%3A%22172.16.23.42%22%3Bs%3A10%3A%22user_agent%22%3Bs%3A50%3A%22Mozilla%2F5.0+%28Windows+NT+6.1%3B+rv%3A6.0%29+Gecko%2F2010010%22%3Bs%3A13%3A%22last_activity%22%3Bi%3A1314955607%3B%7D664b51a01ef99bac95f3e2206e79cb00;PHPSESSID=v33mlm1437lmop1fquta675vv4;username=linjinming; tk=1314955601855 Pragma: no-cache

Cache-Control: no-cache

响应头信息

HTTP/1.1 200 OK

Date: Fri, 02 Sep 2011 09:27:07 GMT

Server: Apache/2.2.3 (Red Hat)

X-Powered-By: PHP/5.1.6

Expires: Thu, 19 Nov 1981 08:52:00 GMT

Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0

Pragma: no-cache

Vary: Accept-Encoding

Content-Encoding: gzip

Content-Length: 31

Connection: close

Content-Type: text/html; charset=UTF-8



### 6. http与https的主要区别

```
1、https协议需要到CA申请证书，一般免费证书较少，因而需要一定费用。

2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl/tls加密传输协议。

3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

4、http的连接很简单，是无状态的；HTTPS协议是由SSL/TLS+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

```

- [HTTP协议：工作原理](http://blog.csdn.net/anndy_/article/details/77198883)
- [SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)