本文把php程序员划分为中、高级程序员两大类程序员，并针对这两大程序员应具备的技能进行分类探索。

**中级PHP程序员** 

**1.Linux**
 能够流畅的使用Shell脚本来完成很多自动化的工作；awk/sed/perl 也操作的不错，能够完成很多文本处理和数据统计等工作；基本能够安装大 部分非特殊的Linux程序（包括各种库、包、第三方依赖等等，比如MongoDB/Redis/Sphinx/Luncene/SVN之类的）；了解基 本的Linux服务，知道如何查看Linux的性能指标数据，知道基本的Linux下面的问题跟踪等 

**2. Nginx:**

 在第一阶段的基础上面，了解复杂一些的Nginx配置；包括多核配置、events、proxy_pass，sendfile/tcp_*配置，知道超 时等相关配置和性能影响；知道nginx除了web server，还能够承担代理服务器、反向静态服务器等配置；知道基本的nginx配置调优；知道如 何配置权限、编译一个nginx扩展到nginx；知道基本的nginx运行原理（master/worker机制，epoll），知道为什么nginx 性能比apache性能好等知识；

**3. MySQL/MongoDB：**

 在第一阶段的基础上面，在MySQL开发方面，掌握很多小技巧，包括常规SQL优化（group by/order by/rand优化等）；除了能够搭 建MySQL，还能够冷热备份MySQL数据，还知道影响innodb/myisam性能的配置选项（比如key_buffer/query_cache /sort_buffer/innodb_buffer_pool_size/innodb_flush_log_at_trx_commit等），也知 道这些选项配置成为多少值合适；另外也了解一些特殊的配置选项，比如 知道如何搭建mysql主从同步的环境，知道各个binlog_format的区 别；知道MySQL的性能追查，包括slow_log/explain等，还能够知道基本的索引建立处理等知识；原理方面了解基本的MySQL的架构 （Server+存储引擎），知道基本的InnoDB/MyISAM索引存储结构和不同（聚簇索引，B树）；知道基本的InnoDB事务处理机制；了解大 部分MySQL异常情况的处理方案（或者知道哪儿找到处理方案）。条件允许的情况，建议了解一下NoSQL的代表MongoDB数据库，顺便对比跟 MySQL的差别，同事能够在合适的应用场景安全谨慎的使用MongoDB，知道基本的PHP与MongoDB的结合开发。

**4. Redis/Memcached：**

 在大部分中型系统里面一定会涉及到缓存处理，所以一定要了解基本的缓存；知道Memcached和Redis的异同和应用场景，能够独立安 装 Redis/Memcached，了解Memcahed的一些基本特性和限制，比如最大的value值，知道PHP跟他们的使用结合；Redis了解 基本工作原理和使用，了解常规的数据类型，知道什么场景应用什么类型，了解Redis的事务等等。原理部分，能够大概了解Memcached的内存结构 （slab机制），redis就了解常用数据类型底层实现存储结构（SDS/链表/SkipList/HashTable）等等，顺便了解一下Redis 的事务、RDB、AOF等机制更好

**5. PHP：**

 除了第一阶段的能力，安装配置方面能够随意安装PHP和各种第三方扩展的编译安装配置；了解php-fpm的大部分配置选项和含义（如 max_requests/max_children/request_terminate_timeout之类的影响性能的配置），知道mod_php /fastcgi的区别；在PHP方面已经能够熟练各种基础技术，还包括各种深入些的PHP，包括对PHP面向对象的深入理解/SPL/语法层面的特殊特 性比如反射之类的；在框架方面已经阅读过最少一个以上常规PHP MVC框架的代码了，知道基本PHP框架内部实现机制和设计思想；在PHP开发中已经能 够熟练使用常规的设计模式来应用开发（抽象工厂/单例/观察者/命令链/策略/适配器 等模式）；建议开发自己的PHP MVC框架来充分让开发自由化， 让自己深入理解MVC模式，也让自己能够在业务项目开发里快速升级；熟悉PHP的各种代码优化方法，熟悉大部分PHP安全方面问题的解决处理；熟悉基本的 PHP执行的机制原理（Zend引擎/扩展基本工作机制）；

**6. 系统设计：**

 能够设计大部分中型系统的网站架构、数据库、基本PHP框架选型；性能测试排查处理等；能够完成类似：浏览 器 -> CDN(Squid) -> Nginx+PHP -> 缓存 -> 数据库 结构网站的基本设计开发维护；能够支撑 每天数百万到千万流量基本网站的开发维护工作； 

**高级PHP程序员**

 **重点：**除了基本的LNMP程序，还能够在某个方向或领域有深入学习。（纵深维度发展）

 **目标：**除了能够完成基本的PHP业务开发，还能够解决大部分深入复杂的技术问题，并且可以独立设计完成中大型的系统设计和开发工作；自己能够独立hold深入某个技术方向，在这块比较专业。（比如在MySQL、Nginx、PHP、Redis等等任一方向深入研究）

**1. Linux：**

 除了第二阶段的能力，在Linux下面除了常规的操作和性能监控跟踪，还能够使用很多高级复杂的命令完成工作（watch/tcpdump/starce /ldd/ar等)；在shell脚本方面，已经能够编写比较复杂的shell脚本（超过500行）来协助完成很多包括备份、自动化处理、监控等工作的 shell；对awk/sed/perl 等应用已经如火纯青，能够随意操作控制处理文本统计分析各种复杂格式的数据；对Linux内部机制有一些了解， 对内核模块加载，启动错误处理等等有个基本的处理；同时对一些其他相关的东西也了解，比如NFS、磁盘管理等等；

**2. Nginx:**

 在第二阶段的基础上面，已经能够把Nginx操作的很熟练，能够对Nginx进行更深入的运维工作，比如监控、性能优化，复杂问题处理等等；看个人兴趣， 更多方面可以考虑侧重在关于Nginx工作原理部分的深入学习，主要表现在阅读源码开始，比如具体的master/worker工作机制，Nginx内部 的事件处理，内存管理等等；同时可以学习Nginx扩展的开发，可以定制一些自己私有的扩展；同时可以对Nginx+Lua有一定程度的了解，看看是否可 以结合应用出更好模式；这个阶段的要求是对Nginx原理的深入理解，可以考虑成为Nginx方向的深入专业者。

**3. MySQL/MongoDB：**

 在第二阶段的基础上面，在MySQL应用方面，除了之前的基本SQL优化，还能够在完成一些复杂操作，比如大批量数据的导入导出，线上大批量数据的更改表 结构或者增删索引字段等等高危操作；除了安装配置，已经能够处理更多复杂的MySQL的问题，比如各种问题的追查，主从同步延迟问题的解决、跨机房同步数 据方案、MySQL高可用架构等都有涉及了解；对MySQL应用层面，对MySQL的核心关键技术比较熟悉，比如事务机制（隔离级别、锁等）、对触发器、 分区等技术有一定了解和应用；对MySQL性能方面，有包括磁盘优化（SAS迁移到SSD）、服务器优化（内存、服务器本身配置）、除了二阶段的其他核心 性能优化选项（innodb_log_buffer_size/back_log/table_open_cache /thread_cache_size/innodb_lock_wait_timeout等）、连接池软件选择应用，对show * （show status/show profile）类的操作语句有深入了解，能够完成大部分的性能问题追查；MySQL备份技术的深入熟悉，包括灾备 还原、对Binlog的深入理解，冷热备份，多IDC备份等；在MySQL原理方面，有更多了解，比如对MySQL的工作机制开始阅读部分源码，比如对主 从同步（复制）技术的源码学习，或者对某个存储引擎（MyISAM/Innodb/TokuDB）等等的源码学习理解，如果条件允许，可以参考CSV引擎 开发自己简单的存储引擎来保存一些数据，增强对MySQL的理解；在这个过程，如果自己有兴趣，也可以考虑往DBA方向发展。MongoDB层面，可以考 虑比如说在写少读多的情况开始在线上应用MongoDB，或者是做一些线上的数据分析处理的操作，具体场景可以按照工作来，不过核心是要更好的深入理解 RMDBS和NoSQL的不同场景下面的应用，如果条件或者兴趣允许，可以开始深入学习一下MongoDB的工作机制。 

**4. Redis/Memcached：**

 在第二阶段的基础上面，能够更深入的应用和学习。因为Memcached不是特别复杂，建议可以把源码进行阅读，特别是内存管理部分，方便深入理 解；Redis部分，可以多做一些复杂的数据结构的应用（zset来做排行榜排序操作/事务处理用来保证原子性在秒杀类场景应用之类的使用操作）；多涉及 aof等同步机制的学习应用，设计一个高可用的Redis应用架构和集群；建议可以深入的学习一下Redis的源码，把在第二阶段积累的知识都可以应用 上，特别可以阅读一下包括核心事件管理、内存管理、内部核心数据结构等充分学习了解一下。如果兴趣允许，可以成为一个Redis方面非常专业的使用者。

**5. PHP：**

 作为基础核心技能，我们在第二阶段的基础上面，需要有更深入的学习和应用。从基本代码应用上面来说，能够解决在PHP开发中遇到95%的问题，了解大部分 PHP的技巧；对大部分的PHP框架能够迅速在一天内上手使用，并且了解各个主流PHP框架的优缺点，能够迅速方便项目开发中做技术选型；在配置方面，除 了常规第二阶段会的知识，会了解一些比较偏门的配置选项（php auto_prepend_file/auto_append_file），包括扩展中 的一些复杂高级配置和原理（比如memcached扩展配置中的memcache.hash_strategy、apc扩展配置中的 apc.mmap_file_mask/apc.slam_defense/apc.file_update_protection之类的）；对php的 工作机制比较了解，包括php-fpm工作机制（比如php-fpm在不同配置机器下面开启进程数量计算以及原理），对zend引擎有基本熟悉 （vm/gc/stream处理），阅读过基本的PHP内核源码（或者阅读过相关文章），对PHP内部机制的大部分核心数据结构（基础类型/Array /Object）实现有了解，对于核心基础结构（zval/hashtable/gc）有深入学习了解；能够进行基本的PHP扩展开发，了解一些扩展开发 的中高级知识（minit/rinit等），熟悉php跟apache/nginx不同的通信交互方式细节（mod_php/fastcgi）；除了开发 PHP扩展，可以考虑学习开发Zend扩展，从更底层去了解PHP。

**6. C/C++：**

 在第二阶段基础上面，能够在C/C++语言方面有更深入的学习了解，能够完成中小型C/C++系统的开发工作；除了基本第二阶段的基础C/C++语法和数 据结构，也能够学习一些特殊数据结构（b-tree/rb-tree/skiplist/lsm-tree/trie-tree等）方便在特殊工作中需 求；在系统编程方面，熟悉多进程、多线程编程；多进程情况下面了解大部分多进程之间的通信方式，能够灵活选择通信方式（共享内存/信号量/管道等）；多线 程编程能够良好的解决锁冲突问题，并且能够进行多线程程序的开发调试工作；同时对网络编程比较熟悉，了解多进程模型/多线程模型/异步网络IO模型的差别 和选型，熟悉不同异步网络IO模型的原理和差异（select/poll/epoll/iocp等），并且熟悉常见的异步框架（ACE/ICE /libev/libevent/libuv/Boost.ASIO等）和使用，如果闲暇也可以看看一些国产自己开发的库（比如muduo）；同时能够设 计好的高并发程序架构（leader-follow/master-worker等）；了解大部分C/C++后端Server开发中的问题（内存管理、日 志打印、高并发、前后端通信协议、服务监控），知道各个后端服务RPC通信问题（struct/http/thirft/protobuf等）；能够更熟 络的使用GCC和GDB来开发编译调试程序，在线上程序core掉后能够迅速追查跟踪解决问题；通用模块开发方面，可以积累或者开发一些通用的工具或库 （比如异步网络框架、日志库、内存池、线程池等），不过开发后是否应用要谨慎，省的埋坑去追bug；

**7. 前端：**

 深入了解HTTP协议（包括各个细致协议特殊协议代码和背后原因，比如302静态文件缓存了，502是nginx后面php挂了之类的）；除了之前的前端 方面的各种框架应用整合能力，前端方面的学习如果有兴趣可以更深入，表现形式是，可以自己开发一些类似jQuery的前端框架，或者开发一个富文本编辑器 之类的比较琐碎考验JavaScript功力；

**8. 其他领域语言学习：**

 在基础的PHP/C/C++语言方面有基本积累，建议在当前阶段可以尝试学习不同的编程语言，看个人兴趣爱好，脚本类语言可以学学 Python /Ruby 之类的，函数式编程语言可以试试 Lisp/Haskell/Scala/Erlang 之类的，静态语言可以试试 Java /Golang，数据统计分析可以了解了解R语言，如果想换个视角做后端业务，可以试试 Node.js还有前面提到的跟Nginx结合的 Nginx_Lua等。学习不同的语言主要是提升自己的视野和解决问题手段的差异，比如会了解除了进程/线程，还有轻量级协程；比如在跨机器通信场景下 面，Erlang的解决方案简单的惊人；比如在不想选择C/C++的情况下，还有类似高效的Erlang/Golang可用等等；主要是提升视野。

**9. 其他专业方向学习：**

 在本阶段里面，会除了基本的LNMP技能之外，会考虑一些其他领域知识的学习，这些都是可以的，看个人兴趣和长期的目标方向。目前情况能够选择的领域比较 多，比如、云计算（分布式存储、分布式计算、虚拟机等），机器学习（数据挖掘、模式识别等，应用到统计、个性化推荐），自然语言处理（中文分词等），搜索 引擎技术、图形图像、语音识别等等。除了这些高大上的，也有很多偏工程方面可以学习的地方，比如高性能系统、移动开发（Android/IOS）、计算机 安全、嵌入式系统、硬件等方向。

**10. 系统设计：**

 系统设计在第二阶段的基础之上，能够应用掌握的经验技能，设计出比较复杂的中大型系统，能够解决大部分线上的各种复杂系统的问题，完成类似 浏览 器 -> CDN -> 负载均衡 ->接入层 -> Nginx+PHP -> 业务缓存 -> 数据 库 -> 各路复杂后端RPC交互（存储后端、逻辑后端、反作弊后端、外部服务） -> 更多后端 酱紫的复杂业务；能够支撑每天数千万到数 亿流量网站的正常开发维护工作。