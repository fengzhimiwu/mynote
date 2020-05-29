####对比LAMP、LNMP、LNAMP之间的区别

> LAMP是Linux+Apache+MySQL+PHP的组合方式
> 优点是开源、稳定、模块丰富是 Apache 的优势
> 但缺点是有些臃肿，内存和 CPU 开销大，性能上有损耗。

> LNMP是Linux+Nginx+Mysql+PHP的组合方式，其特点是利用Nginx的快速与轻量级，替代以前的LAMP(Linux+Apache+Mysql+PHP)的方式。由于安装方便，并且安装脚本也随时更新，LNMP成为很多站长首选的一键安装包。

> LNMP方式的优点：占用VPS资源较少，Nginx配置起来也比较简单，利用fast-cgi的方式动态解析PHP脚本。
> LNMP方式的缺点：php-fpm组件的负载能力有限，在访问量巨大的时候，php-fpm进程容易僵死，容易发生502 bad gateway错误。

> LNAMP是Linux+Nginx+Apache+Mysql+PHP的组合方式，其特点是利用Nginx来作为静态脚本的解析，而利用Nginx的转发特性，将动态脚本的解析转交给Apache来处理，这样，能充分利用两种Web服务器的特点，对于访问量需求较大的站点来说，是一个很不错的选择。

> LNAMP方式的优点：由于Apache本身处理PHP的能力比起php-fpm要强，所以不容易出现类似502 bad gateway的错误。适合访问量较大的站点使用。
> LNAMP方式的缺点：相比LNMP方式会多占用一些资源，另外，配置虚拟主机需要同时修改Nginx和Apache的配置文件，要稍微麻烦一些。

####CGI、FastCGI、PHP-CGI、PHP-FPM分别是什么

* **CGI**

  > CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。
  > CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。

* **FastCGI**

  > FastCGI具有语言无关性.
  > FastCGI在进程中的应用程序，独立于核心web服务器运行，提供了一个比API更安全的环境。APIs把应用程序的代码与核心的web服务器链接在一起，这意味着在一个错误的API的应用程序可能会损坏其他应用程序或核心服务器。 恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥。
  > FastCGI技术目前支持语言有：C/C++、Java、Perl、Tcl、Python、SmallTalk、Ruby等。相关模块在Apache, ISS, Lighttpd等流行的服务器上也是可用的。
  > FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。
  > **FastCGI的工作原理**
  > Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
  > FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
  > 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
  > FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。
  > 在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。
  > **FastCGI的不足**
  > 因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。
  > **PHP-CGI**
  > PHP-CGI是PHP自带的FastCGI管理器。
  > PHP-CGI的不足：
  > php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
  > 直接杀死php-cgi进程，php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题，守护进程会平滑从新生成新的子进程。）
  > **PHP-FPM**
  > PHP-FPM是一个PHP FastCGI管理器，是只用于PHP

####如何确定一台机器应该开启多少个php-fpm进程

>使用netstat -napo |grep "php-fpm" | wc -l查看当前进程数
>依据每个php-fpm 30MB的占用量计算最大可开启数量，还要根据CPU核数选择合适的开启数量，需要使用压测找到一个最合理的数值