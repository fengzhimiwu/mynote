1、service是一个脚本命令，分析service可知是去/etc/init.d目录下执行相关程序。service和chkconfig结合使用。 
服务配置文件存放目录/etc/init.d/

2、systemd 
centos7版本中使用了systemd，systemd同时兼容service,对应的命令就是systemctl 

systemctl命令是系统服务管理器指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。

任务	旧指令	新指令
使某服务自动启动	chkconfig --level 3 httpd on	systemctl enable httpd.service

使某服务不自动启动	chkconfig --level 3 httpd off	systemctl disable httpd.service

检查服务状态	service httpd status	systemctl status httpd.service
（服务详细信息） systemctl is-active httpd.service （仅显示是否 Active)

显示所有已启动的服务	chkconfig --list	systemctl list-units --type=service

启动某服务	service httpd start	systemctl start httpd.service

停止某服务	service httpd stop	systemctl stop httpd.service

重启某服务	service httpd restart	systemctl restart httpd.service



du -h --max-depth=1
du -h
du -s /* | sort -nr 命令查看那个目录占用空间大
执行 du -sh /usr/local/* | sort -r | head -n 10 查看 /usr/local 目录下占用空间较大的10个文件，并按照降序排列

如何把文件的权限设定成777，其中每个7代表什么？ chmod -R 777 ./* 从左至右用0-9表示10个字符，第0位确定文件类型， 第1-3位确定属主权限拥有该文件的权限（该文件的所有者的权限）， 第4-6位确定属组权限拥有该文件的权限（所有者的同组用户）， 第7-9位确定其他用户拥有该文件的权限。 用数字来代表各个权限，则：

读，权限是二进制的100，十进制是4；

写，权限是二进制的010，十进制是2；

执行，权限是二进制的001，十进制是1；

即：各权限的对应数字为：r:4，w:2，x:1

####MIME是什么，列举一些常见的类型 mime类型是多用途互联网邮件扩展类型

常见的MIME类型(通用型)：

[超文本标记语言]文本 .html text/html
xml文档 .xml text/xml
XHTML文档 .xhtml application/xhtml+xml
普通文本 .txt text/plain
RTF文本 .rtf application/rtf
[PDF文档].pdf application/pdf
Microsoft Word[文件] .word application/msword
PNG图像 .png image/png
GIF图形 .gif image/gif
JPEG图形 .jpeg,.jpg image/jpeg
au声音[文件] .au audio/basic
MIDI音乐[文件] mid,.midi audio/midi,audio/x-midi
RealAudio音乐[文件] .ra, .ram audio/x-pn-realaudio
MPEG[文件] .mpg,.mpeg video/mpeg
AVI[文件] .avi video/x-msvideo
GZIP[文件] .gz application/[x-gzip]
TAR[文件] .tar application/x-tar
任意的二进制数据 application/octet-stream

| **用于**[**WAP**](https://baike.baidu.com/item/WAP)[**服务器**]**的MIME类型有：** |                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| [MRP]文件（国内普遍的手机）                                  | .mrp application/octet-stream                          |
| IPA文件([IPHONE])                                            | .ipa application/iphone-package-archive                |
|                                                              | .deb application/x-debian-package-archive              |
| APK文件([安卓系统])                                          | .apk application/vnd.android.package-archive           |
| CAB文件([Windows Mobile])                                    | .cab application/vnd.cab-com-archive                   |
| XAP文件([Windows Phone 7])                                   | .xap application/x-silverlight-app                     |
| SIS文件([symbian]平台/S60V1)                                 | .sis application/vnd.symbian.install-archive *（下有） |
| SISX文件(symbian平台/S60V3/V5)                               | .sisx application/vnd.symbian.epoc/x-sisx-app          |
| JAR、JAD文件([JAVA平台]手机通用格式)                         | .jar .jad下面有                                        |

ASCII、Unicode、GBK、UTF-8 ascii英文编码，gbk是中文编码，unicode全世界统一编码，utf-8基于unicode,unicode比较浪费带宽和 硬盘，所以将unicode编码解码一套编码规则，就是utf-8



[Linux网络编程之select、poll、epoll的比较，以及epoll的水平触发(LT)和边缘触发(ET)](https://www.cnblogs.com/heluan/p/9589086.html)

[I/O多路复用技术（multiplexing）是什么？](https://www.zhihu.com/question/28594409)

Linux的网络通信先后推出了select、poll、epoll三种模式。
select有以下三个问题：
（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大。
（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大。
（3）select支持的文件描述符数量太小了，默认是1024。
poll解决了第三个问题，select保存描述符fd的数据结构是数组，poll改成了链表，突破了fd的个数限制。
但是第1和第2个问题依然存在。
epoll在poll的基础上，又解决了前两个问题：
（1）对第一个问题，epoll每次注册新的事件到epoll句柄中时（在epoll_ctl中指定EPOLL_CTL_ADD），会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝。这样epoll保证了每个fd在整个过程中只会拷贝一次。
（2）对第二个问题，epoll单独设置了一个就绪链表，当fd就绪(可读/可写)之后，放入就绪链表。epoll_wait只需要遍历就绪链表，而不需要遍历所有的fd，从而节省大量的CPU时间。
epoll有LT和ET两种工作模式，默认工作模式是LT（水平触发），高速工作模式是ET（边缘触发）。
LT是fd只要处于可读或可写状态，就会通知用户；ET只有不可读变为可读，或不可写变为可写之时，才会通知用户。
ET对系统的调用，比LT要少得多，所以ET是高速工作模式，效率高很多。
用户使用ET模式时，读/写fd的时候，必须连续读/写完（直到返回EAGAIN错误）。否则如果未读/写完，系统会认为状态没有变化，就不会再重复通知，这样这个fd就死掉了。
epoll的内核实现以及它的数据结构

epoll_create:创建内核事件表用来存放描述符和事件。它的数据结构为：struct eventpoll。其中包括了两个重要成员，一个是红黑树，也就是内核事件表。另一个重要成员是用于存放就绪事件的队列。
epoll_ctl:红黑树添加节点操作：ep_insert。红黑树移除节点操作：ep_remove。红黑树修改节点操作：ep_modify。每个节点都是一个描述符和事件的结构体。
epoll_wait：负责收集就绪事件。
注意：关于epoll_wait是如何收集就绪事件的，大致是这样一个思路：添加事件和描述符时，注册回调函数。当描述符上有事件就绪时，把描述符和事件添加到队列中
select 有描述符限制吗？是多少？

https://blog.csdn.net/H2OzzZZ/article/details/89523601
关于文件描述符上限的这个问题，首先就要理解提问者所关注的重点，而且一般来说讨论的是select本身的FD_SETSIZE=1024这个限制，如果是选择题poll和epoll应该没有上限，但如果是问答，最好能讲清楚

### 8. linux中怎么查看系统资源占用情况,同时会问到怎么查看TCP端口,TCP连接状态
```
top         //可能会问得更深,比如显示出来有哪些信息、你关心哪些信息、查看某个进程等
iostat      //磁盘cpu
free        //内存剩余
df          //磁盘使用情况
du          //文件占用信息

ps              //查看进程信息
netstat -anptol //查看端口占用情况,参数细节建议查文档,小心被问倒
```