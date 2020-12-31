PHP的所有应用程序都是通过WEB服务器(如IIS，Nginx或Apache)和PHP引擎程序解释执行完成的，工作过程：

(1)当用户在浏览器地址中输入要访问的PHP页面文件名，然后触发一个web请求，并将请求传送到WEB服务器。

(2)WEB服务器接受这个请求，并根据其后缀进行判断是一个PHP请求，WEB服务器从硬盘或内存中调出用户要访问的PHP应用程序，并将其发送给PHP引擎程序。

(3)PHP引擎程序将会对WEB服务器传送过来的文件从头到尾进行扫描并根据命令从后台读取，处理数据，并动态地生成相应的HTML页面。

(4)PHP引擎将生成HTML页面返回给WEB服务器。WEB服务器再将HTML页面返回给客户端浏览器。

\2. php运行模式：

1）cgi 通用网关接口（Common Gateway Interface)）

2） fast-cgi 常驻 (long-live) 型的 CGI

3） cli 命令行运行 （Command Line Interface）

4）web模块模式 （apache等web服务器运行的模块模式）

额外解释：

1）模块模式：

模块模式是以mod_php5模块的形式集成，此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（mod_php5）， PHP模块通过注册apache2的ap_hook_post_config挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载，这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号HUP或者AP_SIG_GRACEFUL通知服务器重新载入模块。但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 Apache中对动态链接库的处理是通过模块mod_so来完成的，因此mod_so模块不能被动态加载，它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

2）php在Nginx中运行模式（Nginx+ PHP-FPM）

详细请看 nginx + php 原理一节

补充：

1、cgi、fast-cgi协议

cgi的历史

CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。

早期的webserver只处理html等静态文件，但是随着技术的发展，出现了像php等动态语言。 webserver处理不了了，怎么办呢？那就交给php解释器来处理吧！但是，php解释器如何与webserver进行通信呢？

为了解决不同的语言解释器(如php、python解释器)与webserver的通信，于是出现了cgi协议。只要你按照cgi协议去编写程序，就能实现语言解释器与webserver的通信。如php-cgi程序。

fast-cgi的改进

有了cgi协议，解决了php解释器与webserver通信的问题，webserver终于可以处理动态语言了。但是，webserver每收到一个请求，都会去fork一个cgi进程，请求结束再kill掉这个进程。这样有10000个请求，就需要fork、kill php-cgi进程10000次。有没有发现很浪费资源？于是，出现了cgi的改良版本，fast-cgi。

fast-cgi每次处理完请求后，不会kill掉这个进程，而是保留这个进程，使这个进程可以一次处理多个请求。这样每次就不用重新fork一个进程了，大大提高了效率。FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。

2、php-fpm是什么

PHP-FPM是一个PHP FastCGI管理器，是只用于PHP的。PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置。

进程包含 master 进程和 worker 进程两种进程。 

master 进程只有一个，负责监听端口，接收来自 Web Server 的请求，而 worker 进程则一般有多个(具体数量根据实际需要配置)，每个进程内部都嵌入了一个 PHP 解释器，是 PHP 代码真正执行的地方。