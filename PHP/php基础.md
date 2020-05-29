#### php的内核原理

**php的内核组成模块和运行原理**

PHP 三大模块的 理解：

> 1.PHP内核：其功能是用来处理 (请求，文件流，错误） 等相关操作。
> 2.Zend引擎：将PHP文件转为机器语言，然后在虚拟机上运行。
> 3.扩展层：函数、类库、流。PHP可以加载扩展实现一些特定操作。
> 推荐：《[PHP教程](https://www.php.cn/course/list/29.html)》

三者关系：

> Zend执行程序时候需要连接若干扩展，
> 它将控制权交由扩展层，
> 等扩展层处理完毕，
> 将结果返还给Zend。
> 最终Zend将程序运行结果返回给PHP内核。
> PHP内核再将结果传给SAPI层。
> 最终输出到浏览器。

PHP设计理念及特点：

> 多进程模型：由于PHP是多进程模型，不同请求间互不干涉，这样保证了一个请求挂掉不会对全盘服务造成影响，当然，随着时代发展，PHP也早已支持多线程模型。
> 弱类型语言：和C/C++、Java、C#等语言不同，PHP是一门弱类型语言。一个变量的类型并不是一开始就确定不变，运行中才会确定并可能发生隐式或显式的类型转换，这种机制的灵活性在web开发中非常方便、高效，具体会在后面PHP变量中详述。
> 引擎(Zend)+组件(ext)的模式降低内部耦合。
> 中间层(sapi)隔绝web server和PHP。
> 语法简单灵活，没有太多规范。缺点导致风格混杂。

####php是什么，简述一下优势和劣势

>- PHP全称“Hypertext Preprocessor”，中文名称“超文本预处理器”，是一种用于创建动态和交互式HTML网页的开源脚本语言。
>- PHP独特的语法混合了C、Java、Perl以及 PHP 自创的语法。
>- 主要特点：开源性和免费性、快捷性、数据库连接的广泛性、面向过程和面向对象并用
>- 优点：流行，容易上手、开发职位很多、仍然在不断发展、可植入性强、拓展性强
>- 缺点：PHP的解释运行机制、语法不太严谨，实时编译，运行速度相对比较慢
>- 开源简单，开发速度比较快，适合web开发

####php5和php7的区别有哪些

>- 性能提升：php7比php5性能提升了两倍
>- 以前许多致命错误，现在改成抛出异常
>- php7新增了标量类型声明
>- php7增加了匿名类
>- php7增加了函数的返回类型声明
>- php7增加了结合比较运算符
>- php7增加了空接合操作符
>- use 加强：从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了
>- 错误处理和64位支持
>- php7移除了一些老的不再支持的SAPI（服务器端应用编程端口）和扩展

> 改进的性能 - 将 PHPNG 代码合并到 PHP7 中，速度是 PHP 5 的两倍。
> 降低内存消耗 - 优化的 PHP 7 使用较少的资源。
> 标量类型声明 - 现在可以强制执行参数和返回类型。
> 一致的 64 位支持 - 对 64 位体系结构机器的一致支持。
> 改进了异常层次 - 异常层次得到了改进
> 许多致命的错误转换为例外 - 例外范围增加，涵盖许多致命的错误转换为例外。
> 安全随机数发生器 - 增加新的安全随机数发生器 API。
> 已弃用的 SAPI 和扩展已删除 - 各种旧的和不受支持的 SAPI 和扩展从最新版本中删除。
> 空合并运算符（？） - 添加了新的空合并运算符。
> 返回和标量类型声明 - 支持所添加的返回类型和参数类型。
> 匿名类 - 支持匿名添加。
> 零成本断言 - 支持零成本断言增加。

####为什么php7比php5性能提升了

>- 变量存储字节减小，减少内存占用，提升变量操作速度
>- 改善数组结构，数组元素和hash映射表被分配到同一块内存里，降低了内存占用、提升了CPU缓存命中率
>- 改进了函数的调用机制，通过优化参数传递的环节，减少了一些指令，提高执行效率

####php的生命周期有哪几个阶段

>- 模块初始化阶段（module startup）
>- 请求初始化阶段（request startup）
>- 脚本执行阶段（execute sccript）
>- 请求关闭阶段（request shutdown）
>- 模块关闭阶段（module shutdown）
>- 注：根据不同SAPI的实现，各阶段的执行情况会有一些差异，比如命令行模式下，每次执行一个脚本都会完成的经历这些阶段。而FastCgi模式下则在启动时执行一次模块初始化，然后各个请求只经历请求初始化，脚本执行，请求关闭几个阶段，在SAPI关闭时，才经历模块关闭阶段。

####mvc是什么?相互间有什么关系?

> mvc是一种开发模式,主要分为三部分:m(model),也就是模型,负责数据的操作;v(view),也就是视图,负责前后台的显示;c(controller),也就是控制器,负责业务逻辑客户端请求项目的控制器,如果执行过程中需要用到数据,控制器就会到模型中获取数据,再将获取到的数据通过视图显示出来
>
> MVC三层分别指：业务模型、视图、控制器，由控制器层调用模型处理数据，然后将数据映射到视图层进行显示，优点是：①可以实现代码的重用性，避免产生代码冗余；②M和V的实现代码分离，从而使同一个程序可以使用不同的表现形式

#### 简述下闭包

> 个人理解闭包是指函数里面定义一个私有的函数，可以调用父函数里的变量（定义在一个函数内部的函数）
> 一个外部函数执行完毕后，由于其内部函数被外部引用，导致其作用域的变量继续存活，而不能在函数执行完毕后销毁，包含这些变量的那个对象就叫闭包。

#### SESSION 与 COOKIE的区别是什么，请从协议，产生的原因与作用说明?

* http无状态协议，不能区分用户是否是从同一个网站上来的，同一个用户请求不同的页面不能看做是同一个用户。
* SESSION存储在服务器端，COOKIE保存在客户端（有最大存储限制，不能超过4K）。
* Session比较安全，cookie用某些手段可以修改，不安全
* Session依赖于cookie进行传递。
* 禁用cookie后，session不能正常使用。
* Session的缺点：保存在服务器端，每次读取都从服务器进行读取，对服务器有资源消耗。
* Session保存在服务器端的文件或数据库中，默认保存在文件中，文件路径由php配置文件的session.save_path指定。Session文件是公有的。

#### session跨域共享解决方案

> 要让session跨域共享，需要**解决三个问题**：
> 1、通过什么方法来传递session_id？
> 2、通过什么方法来保存session信息？
> 3、通过什么方法来进行跨域?
> 一、**传递session_id有4种方法**
> 1、 通过cookie
> 2、 设置php.ini中的session.use_trans_sid=1，让PHP自动跨页传递session id
> 3、 手动通过url或隐藏表单传值
> 4、 用文件或数据库方式传递，在通过其他key对应取值
> 二、**保存session信息有3种方法**
> 1、数据库
> 2、memcache
> 3、共享文件
> **三、跨域访问的方法**
> 1、通过服务器（php脚本）
> 2、通过JavaScript

####如何修改 SESSION 的生存时间

> 设置浏览器保存的 sessionid 失效时间 setcookie (session_name (), session_id (), time () + $lifeTime, "/");
> 可以使用 SESSION 自带的 session_set_cookie_params (86400); 来设置 Session 的生存期
> 通过修改 php.ini 中的 session.gc_maxlifetime 参数的值就可以改变 session 的生存时间

####如何修改session的生存时间

> 在php.ini 中设置 session.gc_maxlifetime = 1440 //默认时间
> \$lifeTime = 24 * 3600; // 保存一天
> session_set_cookie_params($lifeTime);
> session_start();

#### isset() 和 empty() 区别

> isset判断变量是否存在，可以传入多个变量，若其中一个变量不存在则返回假；
> empty判断变量是否为空为假，只可传一个变量，如果为空为假则返回真。

#### 请说明 PHP 中传值与传引用的区别。什么时候传值什么时候传引用？

> 按值传递：函数范围内对值的任何改变在函数外部都会被忽略
> 按引用传递：函数范围内对值的任何改变在函数外部也能反映出这些修改
> 优缺点：按值传递时，php必须复制值。特别是对于大型的字符串和对象来说，这将会是一个代价很大的操作。按引用传递则不需要复制值，对于性能提高很有好处。

#### 如何获取客户端的ip（要求取得一个int）和服务器ip的代码

> 客户端：$_SERVER[“REMOTE_ADDR”];或者getenv（‘REMOTE_ADDR’）
> 服务器端：gethostbyname（‘www.baidu.com’）

#### 简述php的垃圾收集机制

> php中的变量存储在变量容器zval中，zval中除了存储变量类型和值外，还有is_ref和refcount字段。refcount表示指向变量的元素个数，is_ref表示变量是否有别名。如果refcount为0时，就回收该变量容器。如果一个zval的refcount减1之后大于0，它就会进入垃圾缓冲区。当缓冲区达到最大值后，回收算法会循环遍历zval，判断其是否为垃圾，并进行释放处理。

####在程序的开发中，如何提高程序的运行效率？

> A、优化SQL语句，查询语句中尽量不使用select *，用哪个字段查哪个字段；少用子查询可用表连接代替；少用模糊查询；
> B、数据表中创建索引；
> C、对程序中经常用到的数据生成缓存。

####语句include和require的区别是什么?为避免多次包含同一文件，可用什么语句代替它们?

> 区别：
> 在失败的时候：
> include产生一个warning，而require产生直接产生错误中断
> require在运行前载入
> include在运行时载入
> 代替：
> require_once
> include_once

#### include和require的区别是什么？

> require是无条件包含，也就是如果一个流程里加入require，无论条件成立与否都会先执行require，当文件不存在或者无法打开的时候，会提示错误，并且会终止程序执行
> include有返回值，而require没有(可能因为如此require的速度比include快)，如果被包含的文件不存在的化，那么会提示一个错误，但是程序会继续执行下去

####安全对一套程序来说至关重要，请说说在开发中应该注意哪些安全机制？

> A、防远程提交；
> B、防SQL注入，对特殊代码进行过滤；
> C、防止注册机灌水，使用验证码。

####PHP 的控制反转 (IOC) 和依赖注入 (DI) 概念

> IOC（inversion of control）控制反转模式；控制反转是将组件间的依赖关系从程序内部提到外部来管理；
DI（dependency injection）依赖注入模式；依赖注入是指将组件的依赖通过外部以参数或其他形式注入；

####堆和栈的区别？

> A、堆是程序运行期间动态分配的内存空间，你可以根据程序的运行情况确定要分配的堆内存的大小；
> B、栈是编译期间就分配好的内存空间，因此你的代码中必须就栈的大小有明确的定义。

####PHP如何实现多继承吗
> 可以使用 interface 或 trait 实现 。
> 1.使用 interface 声明类不能被实例化，并且属性必须是常量，方法不能有方法体
> 2.trait 声明的类不能被实例化，由use引入，会覆盖父类的相同属性及方法，如果有多个use,那么按顺序下面的覆盖最上面的相同的属性及方法

####array_map和array_walk区别

> 关键点
> map 主要是为了得到你的回调函数处理后的新数组，要的是结果。
> walk 主要是对每个参数都使用一次你的回调函数，要的是处理的过程。
> walk 可以认为提供额外参数给回调函数，map不可以
> walk 主要是要对数组内的每个值进行操作，操作结果影响原来的数组
> map 主要是对数组中的值进行操作后返回数组，以得到一个新数组
> walk 可以没有返回值
> map要有，因为要填充数组

####简述一下php5、php7 zval struct有什么特点，如果需要操作数组，他们的处理方式有什么不同？

####单引号和双引号的区别是什么？

（1）双引号可以解析变量，单引号不能解析变量
（2）双引号和单引号可以互相嵌套
（3）双引号当中的变量可以使用特殊字符分隔开，但是特殊 字符会原样输出，使用{}不会输出
（4）双引号当中包含单引号，单引号当中包含变量，变量会被解析，单引号会被原样输出
（5）双引号可以解析转义字符，单引号不会解析转义字符，单引号只会解析本身和’单引号本身的转义
（6）单引号当中嵌套单引号，双引号当中嵌套双引号，当中的单引号和双引号需要使用转义符合
（7）单引号效率要高于双引号

php 错误处理

try catch finally
set_error_handle //处理默认notice，warning等错误
register_shutdown_function
register_shutdown_function(function(){
    if($error = error_get_last()){
        print_r($error);
    }
}

简述php的错误机制

error_reporting(E_ALL);
ini_set('display_errors', 'On');
try catch finally
set_error_handle //处理默认notice，warning等错误
register_shutdown_function
register_shutdown_function(function(){
    if($error = error_get_last()){
        print_r($error);
    }
}

####trait特性

主要用来解决单继承引起的代码复用问题
类中使用trait类似于include
trait支持所有修饰符例如final,static,abstract
trait不能通过new实例化
trait中的方法冲突可以通过insteadof或者as 关键词解决，通过as还可以修改访问权限
use atrait, btrait;
trait里也可以使用trait
insteadof代替的意思，as 别名的意思
use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
        B::bigTalk as talk;
    }
reflaction 反射。反射的意思是提取出关于类，方法，属性参数等的详细信息，包括注释

####在php手册中，经常能看到xxx函数是二进制安全的，比如转换大小写的strtoupper、strtolower俩函数，你如何理解“二进制安全”？

因为有了对字符串长度定义len, 所以在处理字符串时候不会以零值字节(\0)为字符串结尾标志.
二进制安全就是输入任何字节都能正确处理, 即使包含零值字节.

redis不像C语言那样,使用'\0'作为判定一个字符串的结尾,而是使用了独立的len,这样可以保证即使存储的数据中有'\0'这样的字符,它也是可以支持读取的.



####PHP遍历文件夹下所有文件

`<?php`
`function read_all($dir){
    if(!is_dir($dir)) return false;
    $handle = opendir($dir);
    if($handle){
        while(($fl = readdir($handle)) !== false){
            $temp = $dir.$fl;
            //$fl !='.' && $fl != '..' 排除当前目录及父级目录
            if(is_dir($temp) && $fl!='.' && $fl != '..'){
                echo '目录：'.$temp.'<br>';
                read_all($temp);
            }else{
                if($fl !='.' && $fl != '..'){
                    echo '文件：'.$temp.'<br>';`
                `}`
            `}`
        `}`
    `}`
`}`
`read_all("./dir/");`
`?>`

39、在命令行中运行php程序
php indx.php
A、从命令行运行php非常简单。但有些注意事项需要各位了解下，诸如$_SESSION之类的服务器变量是无法在命令行中使用的，其他代码的运行则和web服务器中完全一样；
B、在命令行中执行php文件的好处之一就是可以通过脚本实现一些计划任务（crontab）的执行，而无须通过web服务器。

延伸1：
php -v 显示当前PHP版本
php -m 显示当前php加载的有效模块
php -i 输出无html格式的phpinfo
php --rf function

延伸2：向php脚本传递参数：
提示：命令行下执行php，是不走Apache/Nginx等这类东西的，没有什么http协议，所以get,post传参数根本不起作用，并且还会报错。有些时候需要在shell命令下把PHP当作脚本执行，比如定时任务。这就涉及到在shell命令下如何给php传参的问题，通常有三种方式传参。

A、使用\$argv or $argc参数接收
echo "接收到{\$argc}个参数";
print_r(\$argv);
?>

B、使用getopt函数
\$param_arr = getopt('a\:b\:');
print_r($param_arr);
?>

C、提示用户输入
fwrite(STDOUT,'Please enter your name：');
echo 'Your name is：'.fgets(STDIN);
?>

40、你用什么方法检查PHP脚本的执行效率（通常是脚本执行时间）和数据库SQL的效率（通常是数据库Query时间），并定位和分析脚本执行和数据库查询的瓶颈所在？
A、PHP脚本的执行效率
a、代码脚本里计时；
b、xdebug统计函数执行次数和具体时间进行分析，最好使用工具winCacheGrind分析；
c、在线系统用strace跟踪相关进程的具体系统调用。

B、数据库SQL的效率
a、sql的explain(mysql)，启用slow query log记录慢查询；
b、通常还要看数据库设计是否合理，需求是否合理等。



在PHP中error_reporting这个函数有什么作用？
设置PHP的报错级别并返回当前级别。

####PHP如何实现页面跳转
方法一：php函数跳转，缺点，header头之前不能有输出，跳转后的程序继续执行，可用exit中断执行后面的程序。
header("Location:网址");//直接跳转
header("refresh:3;url=http://www.jsdaima.com");//三秒后跳转
方法二：利用meta
echo"";

####如何把一个GB2312格式的字符串装换成UTF-8格式？
iconv('GB2312','UTF-8','js代码（www.jsdaima.com）是IT资源下载与IT技能学习平台。');
?>

####如果需要原样输出用户输入的内容，在数据入库前，要用哪个函数处理？
htmlspecialchars或者htmlentities

####PDO、adoDB、PHPLib 数据库抽象层比较
PHP 数据库抽象层就是指，封装了数据库底层操作的介于 PHP 逻辑程序代码和数据库之间的中间件。
PDO 以 PHP 5.1 为基础进行设计，它使用 C 语言做底层开发，设计沿承 PHP 的特点，以简洁易用为准，从严格意义上讲，PDO 应该归为 PHP 5 的 SPL 库之一，而不应该归于数据抽象层，因为其本身和 MySQL 和 MySQLi 扩展库的功能类似。PDO 是不适合用在打算或者有可能会变更数据库的系 统中的。
ADODB 不管后端数据库如何，存取数据库的方式都是一致的；
转移数据库平台时，程序代码也不必做太大的更动，事实上只需要改动数据库配置文 件。提供了大量的拼装方法，目的就是针对不同的数据库在抽象层的底层对这些语句进行针对性的翻译，以适应不同的数据库方言！但是这个抽象层似乎体积过于庞 大了，全部文件大概有 500K 左右，如果你做一个很小的网站的话，用这个似乎大材小用了
PHPLib 可能是伴随 PHP 一同成长最老的数据库抽象层（但和 ADODB 相比，它只算是一个 MySQL 抽象类库），这个抽象类使用方法相当简单，体积小，是小型网站开发不错的选择。
PDO 提供预处理语句查询、错误异常处理、灵活取得查询结果（返回数组、字符串、对象、回调函数）、字符过滤防止 SQL 攻击、事务处理、存储过程。
ADODB 支持 缓存查询、移动记录集、（HTML、分页、选择菜单生成）、事务处理、输出到文件。 

####预定义变量、魔术变量、魔术方法比较，及作用举例
预定义变量（超级全局变量）

\$GLOBALS
\$\_SERVER
\$\_GET
\$\_POST
\$\_COOKIE
\$\_SESSION
\$\_REQUEST
\$\_ENV

魔术方法

\_\_construct 
\_\_destruct
\_\_autoload
\_\_get
\_\_set
\_\_call
\_\_callStatic
\_\_clone
\_\_toString
\_\_sleep
\_\_wakeup
\_\_invoke

魔术常量 
LINE
FILE
DIR
CLASS
FUNCTION
METHOD
NAMESPACE

简述两种屏蔽php程序的notice警告的方法?
初始化变量，文件开始设置错误级别或者修改php.ini设置error_reportingset_error_handler和@抑制错误：
①在程序中添加：error_reporting(E_ALL&~E_NOTICE);
②.或者修改php.ini中的：error_reporting=E_ALL改为：error_reporting=E_ALL&~E_NOTICE③.error_reporting(0);或者修改php.inidisplay_errors=Off

####写一个函数，尽可能高效的，从一个标准url里取出文件的扩展名?
例如://www.sina.com.cn/abc/de/fg.php?id=1需要取出php或.php?
\$url="//www.sina.com.cn/abc/de/fg.php?id=1";
arr=parseurl(url);
pathArr=pathinfo(arr['path']);
print_r($pathArr['extension']);



一个文件的路径为/wwwroot/include/page.class.php，写出获得该文件扩展名的方法（5分）

$arr = pathinfo(“/wwwroot/include/page.class.php”);

\$str = substr(​\$arr[‘basename’],strrpos($arr[‘basename’],’.’));

写出一个函数，参数为年份和月份，输出结果为指定月的天数（5分）

Function day_count(\$year,$month){

Echo date(“t”,strtotime(\$year.”-”.$month.”-1”));

#### spl 常用数据结构类

![1](.\images\1.jpeg)

**堆(Heap)**就是为了实现优先队列而设计的一种数据结构，它是通过构造二叉堆(二叉树的一种)实现。根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。二叉堆还常用于排序(堆排序)。

**如下：最小堆（任意节点的优先级不小于它的子节点）**

![](.\images\2.png)

PHP的SPL提供了些数据结构基本类型的实现，虽然我们可以使用传统的变量类型来描述数据结构，例如用数组来描述堆栈（Strack）然后使用对应的方式 pop 和 push（array_pop()、array_push()），但你得时刻小心，因为毕竟它们不是专门用于描述数据结构的，一次误操作就有可能破坏该堆栈。而SPL的 SplStack 对象则严格以堆栈的形式描述数据，并提供对应的方法。同时，这样的代码应该也能理解它在操作堆栈而非某个数组，从而能让你的同伴更好的理解相应的代码，并且它更快。

- **栈的实现**

- $stack = new SplStack();

- //入栈

- $stack->push('a');

- $stack->push('b');

- //出栈

- echo $stack->pop();

- echo $stack->pop();

- **队列的实现**

- $queue = new SplQueue();

- //入队列

- $queue->enqueue('a');

- $queue->enqueue('b');

- $queue->enqueue('c');

- //出队列

- echo $queue->dequeue();

- echo $queue->dequeue();

- echo $queue->dequeue();

- **最小堆的实现**

- $heap = new SplMinHeap();

- //插入到堆

- $heap->insert('a');

- $heap->insert('b');

- //从堆中提取数据

- echo $heap->extract();

- echo $heap->extract();

- **固定长度的数组**

- $array = new SplFixedArray(5);

- $array[1] = 5;

- var_dump($array);

- 

####PHP 做好防盗链的基本思想 防盗链

什么是盗链？

盗链是指服务提供商自己不提供服务的内容，通过技术手段绕过其它有利益的最终用户界面 (如广告)，直接在自己的网站上向最终用户提供其它服务提供商的服务内容，骗取最终用户的浏览和点击率。受益者不提供资源或提供很少的资源，而真正的服务提供商却得不到任何的收益。

网站盗链会大量消耗被盗链网站的带宽，而真正的点击率也许会很小，严重损害了被盗链网站的利益。 如何做防盗链？

不定期更名文件或者目录

限制引用页

原理是，服务器获取用户提交信息的网站地址，然后和真正的服务端的地址相比较， 如果一致则表明是站内提交，或者为自己信任的站点提交，否则视为盗链。实现时可以使用 HTTP_REFERER1 和 htaccess 文件 (需要启用 mod_Rewrite)，结合正则表达式去匹配用户的每一个访问请求。

文件伪装

文件伪装是目前用得最多的一种反盗链技术，一般会结合服务器端动态脚本 (PHP/JSP/ASP)。实际上用户请求的文件地址，只是一个经过伪装的脚本文件，这个脚本文件会对用户的请求作认证，一般会检查 Session，Cookie 或 HTTP_REFERER 作为判断是否为盗链的依据。而真实的文件实际隐藏在用户不能够访问的地方，只有用户通过验证以后才会返回给用户

加密认证

这种反盗链方式，先从客户端获取用户信息，然后根据这个信息和用户请求的文件名 字一起加密成字符串 (Session ID) 作为身份验证。只有当认证成功以后，服务端才会把用户需要的文件传送给客户。一般我们会把加密的 Session ID 作为 URL 参数的一部分传递给服务器，由于这个 Session ID 和用户的信息挂钩，所以别人就算是盗取了链接，该 Session ID 也无法通过身份认证，从而达到反盗链的目的。这种方式对于分布式盗链非常有效。

随机附加码

每次，在页面里生成一个附加码，并存在数据库里，和对应的图片相关，访问图片时和此附加码对比，相同则输出图片，否则输出 404 图片

加入水印

####smarty模板的特点

速度快，编译型，缓存技术，插件机制，强大的表现逻辑

####Smarty的原理

smarty是一个模板引擎，使用smarty主要是为了实现逻辑和外在内容的分离，如果不使用模板的话，通常的做法就是php代码和html代码混编。使用了模板之后，则可以将业务逻辑都放到php文件中，而负责显示内容的模板则放到html文件中。
Smarty在执行display方法的时候，读取模板文件，并进行数据替换，生成编译文件，之后每次访问都会直接访问编译文件，读取编译文件省去了读取模板文件，和字符串替换的时间，所以可以更快，编译文件里时间戳记录模板文件修改时间，如果模板被修改过就可以检测到，然后重新编译（编译是把静态内容保存起来，动态内容根据传入的参数不同而不同）。
如果启用了缓存，则会根据编译文件生成缓存文件，在访问的时候如果有缓存文件并且缓存文件没有过期，则直接访问缓存文件。

####请写一个函数验证电子邮件的格式是否正确（要求使用正则）

preg_match('/^[\w\-\.]+@[\w\-]+(\.\w+)+\$/',\$email);



####写一个函数，获取一篇文章内容中的全部图片，并下载

```php
public function searchImgDownload($url, $download_dir = '')
{
    //判断url是否正确
    if (!filter_var($url, FILTER_VALIDATE_URL)) {
        return false;
    }

    if (!$download_dir) {
        $download_dir = _CACHE_DIR_ . '/download/';
    }

    //读取页面内容
    $html = file_get_contents($url);
    //正则匹配img标签
    preg_match_all('/<img[^>]*src="([^"]*)"[^>]*>/i',$html, $matchs);  
    $images = $matchs[1];
    //判断下载目录是否存在
    if (!is_dir($download_dir)) {
        mkdir($download_dir, 0777, true);
    }
    $allImg = [];
    foreach ($images as $img) {
        $imgName = $download_dir.md5($img) . '.png';
        file_put_contents($imgName, file_get_contents($img));
        array_push($allImg, $imgName);
    }
    var_dump($allImg);exit;
}
```

### 2. 获取当前客户端的IP地址，并判断是否在（1.1.1.1,255.255.255.254)
```php
function getip()
{
    $unknown = 'unknown';
    if (
        isset($_SERVER['HTTP_X_FORWARDED_FOR']) && 
        $_SERVER['HTTP_X_FORWARDED_FOR'] && 
        strcasecmp($_SERVER['HTTP_X_FORWARDED_FOR'], $unknown)
    ) {
        $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    } elseif (
        isset($_SERVER['REMOTE_ADDR']) && 
        $_SERVER['REMOTE_ADDR'] && 
        strcasecmp($_SERVER['REMOTE_ADDR'], $unknown)
    ) {
        $ip = $_SERVER['REMOTE_ADDR'];
    }
    /*
    处理多层代理的情况
    或者使用正则方式：
    $ip = preg_match("/[\d\.]{7,15}/", $ip, $matches) ? $matches[0] : $unknown;
    */
    if (false !== strpos($ip, ','))
        $ip = reset(explode(',', $ip));
    return $ip;
}

$client_ip = getip();
$client_ip = sprintf('%u', ip2long($client_ip));   //64位系统无压力

/**
 * plan A
 */
$range_min = sprintf('%u', ip2long('1.1.1.1'));
$range_max = sprintf('%u', ip2long('255.255.255.255'));

/**
 * plan B
 */
$range_min = bindec(decbin(ip2long('1.1.1.1')));
$range_max = bindec(decbin(ip2long('255.255.255.255')));


if ($client_ip >= $range_min and $client_ip <= $range_max) {
    echo 'true';
} else {
    echo 'false';
}
```

####从扑克牌中随机抽5张牌，判断是不是一个顺子,即这5张牌是连续的,JQK用11、12、13表示

```php
function eatDuck(array $arr)
{
    $count = count($arr);
    if (count(array_unique($arr)) != $count) {
        return false;//对子
    }
    if (max($arr) - min($arr) != $count - 1) {
        return false;
    }
    return true;
}


function eatDuck(array $arr)
{
    $count = count($arr);
    if (count(array_unique($arr)) != $count) {
        return false;//对子
    }
    if (max($arr) - min($arr) != $count - 1) {
        return false;
    }
    return true;
}

var_dump(eatDuck([1, 3, 5, 2, 4]));
var_dump(eatDuck([10, 13, 11, 12, 14]));
var_dump(eatDuck([1, 3, 5, 7, 9]));

```

### 13. array_map array_reduce array_walk array_filter的区别

array_map 函数将用户自定义函数作用到数组中的每个值上，并返回用户自定义函数作用后的带有新值的数组,数组的键名不变

array_reduce 函数用回调函数迭代地将数组简化为单一的值。如果指定第三个参数，则该参数将被当成是数组中的第一个值来处理，或者如果数组为空的话就作为最终返回值

array_walk 函数对数组中的每个元素应用用户自定义函数。在函数中，数组的键名和键值是参数。

array_filter 函数把输入数组中的每个键值传给回调函数。如果回调函数返回 true，则把输入数组中的当前键值返回结果数组中。数组键名保持不变。



### 扩展阅读

- [3年PHPer的面试总结](http://coffeephp.com/articles/4?utm_source=laravel-china.org)
- [垃圾回收机制](http://docs.php.net/manual/zh/features.gc.performance-considerations.php)
- [S.O.L.I.D 面向对象设计](https://laravel-china.org/articles/4160/solid-object-oriented-design-and-programming-oodoop-notes?order_by=created_at&)
- [浅谈IOC--说清楚IOC是什么](http://www.cnblogs.com/DebugLZQ/archive/2013/06/05/3107957.html)
- [Redis和Memcached的区别](https://www.biaodianfu.com/redis-vs-memcached.html)