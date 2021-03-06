垃圾回收，简称gc。顾名思义，就是废物重利用的意思。再说这个之前先接触一下内存泄露，大概意思就是申请了一块地儿拉了会儿屎，拉完后不收拾，那么那块儿地就算是糟蹋了，地越用越少，最后一地全是屎。说到底一句，用了记得还。一定程度上说，垃圾回收机制就是用来擦屁股的。如果用过C语言，那么申请内存的方式是malloc或者是calloc，然后你用完这个内存后，一定不要忘了用free函数去释放掉，这就是传说中手动垃圾回收，一般都是扫地神僧用这种方式。很多高层次语言中，你这辈子都是接触不到内存管理的，比如世界上最好的语言php，这种语言替你管理了内存，你就安安心心写烂代码即可。写php的，你说你关心内存，我是不怎么相信的，一定是你在装逼。当然了，如果你用的swoole或者wm或者自己发明的常驻内存级php应用,那你将不得不关注内存泄露问题，也就说一定要记得释放无用变量。那么，在用的最普遍地最传统的web开发中，php的自动垃圾回收机制是怎样的呢？这个问题我们先这么想，就是都知道php是C语言实现的，现在把C语言给你放在这里了，然后你想想如何用C语言实现对一个变量的统计以及释放。你不要想如何实现php，你就想C语言如何实现一个变量，从声明开始到最后没人用了，就把这个变量所占的内存给释放掉。你从这个角度出发，就会舒服一些，这不再是一个技术难题，而是一个傻逼产品经理提的一个傻逼需求。好了，步入正题，PHP进行内存管理的核心算法一共两项：一是引用计数，二是写时拷贝，请理（bei）解（song）。当你声明一个PHP变量的时候，C语言就在底层给你搞了一个叫做zval的struct（结构体）；如果你还给这个变量赋值了，比如“hello world”，那么C语言就在底层再给你搞一个叫做zend_value的union（联合体），总体看来就是这样的：

好了，进入代码实战阶段，注意两点：

1. 我用的PHP版本是7.1.17（记住！这个很重要！不同版本的PHP有极大可能会出现不相同的结果！我试过6个版本的PHP，三个PHP5版本，三个HPP7版本，其中PHP7版本变化尤其多，但不影响业务代码不会出bug，放心），运行环境是cli。
2. 下面的原理解只针对PHP7，不再说5了。你面试的时候，只需要说5的我不太了解，7的我深入看过一些即可，面试官不会难为你的。

```php
<?php
// 先不要问为什么非要加mt_rand，不然，绝笔说不过来了，到处都是坑
$a = 'hello'.mt_rand( 1, 1000 );
echo xdebug_debug_zval('a');
$b = a;
echo xdebug_debug_zval('a');
$c = a;
echo xdebug_debug_zval('a');
unset( $c );
echo xdebug_debug_zval('a');
以上代码运行结果是将会是：
a: (refcount=1, is_ref=0)='hello916'
a: (refcount=2, is_ref=0)='hello916'
a: (refcount=3, is_ref=0)='hello916'
a: (refcount=2, is_ref=0)='hello916'
```

其中，zval struct结构体用于保存$a，zend_value union联合体用于保存数据内容也就是'hello916'。由于后面又声明了b和c，所以C不得不又在底层给你搞出两个zval struct结构体来。

其中，zval和zend value的结构大概如下：（注意！！！这并不是完整正确的PHP zval和zend_value在C语言中struct和union实现，仅仅是挑出最重点的部分写出来，强调一下：你没有必要一个字不差背诵过zval和zend_value，你只需要知道原理）

```
zval {  
    string "a"            //变量的名字是a  
    value  zend_value     //变量的值  
    type   string         //变量是字符串类型
}
zend_value {  
    string    "hello916"  //值的内容  
    refcount  1           //引用计数 
}
```

看到上面两个，如果面试官问你php变量为什么能够保存字符串"123"也能保存数字123，你知道该怎么回答了吧？就答出重点zval中有该变量的类型，当是字符串123的时候，type就是string，此时value指向“123”；当是整数123的时候，zval的type为int，value为123。这就是答题的思想，这很重要！而且，通过C语言都是可以实现的！具体真正的val和zend_value的模样，有兴趣的同学可以去网上搜搜，如果你没有C语言的底子，可能比较吃力！前者是一个struct结构体，后者是一个union联合体！

这个refcount就是传说中的引用计数了，初始化的时候a后面的引用次数为1（注意，正确说法应该是a后面的赋值的数组zend_value引用计数为1，而不是a这个变量zval本身）。然后我们将$b = $a，其实相当于又一个变量指向了这个zend_value，所以refcount变为2，最后将\$c = ​\$a，同理，zend_value的refcount再次加1变成了3。然后，我们用unset( \$c )，这会儿，C语言要做的就是把$c的zval给KO free掉，但是并不是free zend_value，这会儿zend_value的refcount就自然而然减1变成2了。

那么写时拷贝是什么意思呢？看下面代码：

```
<?php
// 先不要问为什么非要加mt_rand，不然，绝笔说不过来了，到处都是坑
$a = 'hello'.mt_rand( 1, 1000 );
$b = $a;
$a = 123;
echo $b.PHP_EOL;
// 运行结果，不用我说吧，脚趾头都知道是'hello'.mt_rand( 1, 1000 )的结果，绝对不可能是123
```

其实，当你把\$a赋值给​\$b的时候，​\$a的值并没有真的复制了一份，这样是对内存的极度不尊重，也是对时间复杂度的极度不尊重，计算机仅仅是将\$b指向了\$a的值而已，这就叫多快好省。那么，什么时候真正的发生复制呢？就是当我们修改\$a的值为123的时候，这个时候就不得已进行复制，避免\$b的值和\$a的一样。

```
<?php
$a = 'hello'.mt_rand( 1, 1000 );
$b = $a;
echo xdebug_debug_zval('a');
$a = 'world'.mt_rand( 2, 2000 );
echo xdebug_debug_zval('a');
// 运行结果为1，其中的原理你自己应该能理顺了昂
```

叨逼叨了这么长，通过简单的案例解释清楚了两个要点：引用计数和写时拷贝，那么垃圾回收也该来了。当一个zval在被unset的时候、或者从一个函数中运行完毕出来（就是局部变量）的时候等等很多地方，都会产生zval与zend_value发生断开的行为，这个时候zend引擎需要检测的就是zend_value的refcount是否为0，如果为0，则直接KO free空出内容来。如果zend_value的recount不为0（废话一定是大于0），这个value不能被释放，但是也不代表这个zend_value是清白的，因为此zend_value依然可能是个垃圾。

什么样的情况会导致zend_value的refcount不为0，但是这个zend_value却是个垃圾呢？PHP7种两种情况：

1. 数组：a数组的某个成员使用&引用a自己

2. 对象：对象的某个成员引用对象自己

   请理（bei）解（song），一般面试，你能回答到这一步，已经非常屌了。

```
<?php
$arr = [ 1 ];
$arr[] = &$arr;
unset( $arr );
```

这种情况下，zend_value不会能释放，但也不能放过它，不然一定会产生内存泄漏，所以这会儿zend_value会被扔到一个叫做垃圾回收堆中，然后zend引擎会依次对垃圾回收堆中的这些zend_value进行二次检测，检测是不是由于上述两种情况造成的refcount为1但是自身却确实没有人再用了，如果一旦确定是上述两种情况造成的，那么就会将zend_value彻底抹掉释放内存。

那么垃圾回收发生在什么时候？有些同学可能有疑问，就是php不是运行一次就销毁了吗，我要着gc有何用？并不是啦，首先当一次fpm运行完毕后，最后一定还有gc的，这个销毁就是gc；其次是，内存都是即用即释放的，而不是攒着非得到最后，你想想一个典型的场景，你的控制器里的某个方法里用了一个函数，函数需要一个巨大的数组参数，然后函数还需要修改这个巨大的数组参数，你们应该是函数的运行范围里面修改这个数组，所以此时会发生写时拷贝了，当函数运行完毕后，就得赶紧释放掉这块儿内存以供给其他进程使用，而不是非得等到本地fpm request彻底完成后才销毁。

说到最后，说些自己的话：大多数情况下，面试官问你问题主要是想一是要你个思维思路，二是看你学习程度。就像gc这个问题，其实很多脚本语言的垃圾回收机制基本上都是靠引用计数和写时拷贝这两种算法结合完成的，所以如果你设计一门脚本语言，gc机制就按照这两种算法进行设计即可。其次是大多数phper不会看这些东西的，面试官问你这个问题不是要你死记硬背那么多细节，你背不过的，他还是想探测你平时有没有更积极地往深层发展的心态。

第一次写面试题答案，但是发现篇幅真的不够，经验不太够，感觉写不是特别如心意，可能是我想说的太多，总想当成博客写，但是并不能，不然就弄不完，所以只能就着可以应付面试官的主题思想来写这种，注重体现重点，很多细节实在没法写，比如我举个例子\$a=[]，然后xdebug_debug_zval( $a )的refcount值你猜是多少？7.1.17下竟然是2，你是不是以为是1，然而并不是。不过你不用纠结这些细节，gc的关键就是能说出引用计数的原理和写时拷贝，很多细节深处都各种奇奇怪怪的东西，面试官自己都不一定知道。