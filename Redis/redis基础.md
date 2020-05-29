# redis整理

  * 事务
  * redis数据开发设计
  * 主从复制
  * 集群分片
  * 数据备份策略
  * 常见reds错误分析
  * 监控redis的服务状态
  * 可视化管理工具
  * [redis防止商品超发](#redis防止商品超发) 
  * redis持久化

1.Redis支持的数据类型？

2.什么是Redis持久化？Redis有哪几种持久化方式？优缺点是什么？

3.Redis 有哪些架构模式？讲讲各自的特点

4.使用过Redis分布式锁么，它是怎么实现的？

5.使用过Redis做异步队列么，你是怎么用的？有什么缺点？

6.什么是缓存穿透？如何避免？什么是缓存雪崩？何如避免？

7.Redis常用命令

8.为什么Redis 单线程却能支撑高并发？

9.说说Redis的内存淘汰策略

10.Redis的并发竞争问题如何解决?

## redis防止商品超发
#### 公用方法 function.php
```php
<?php
//获取商品key名称
function getKeyName($v)
{
    return "send_goods_".$v;
}

//日志写入方法
function writeLog($msg,$v)
{
    $log = $msg.PHP_EOL;
    file_put_contents("log/$v.log",$log,FILE_APPEND);
}
```

#### 自定义redis操作类 myRedis.php
```php
<?php
/**
 * @desc 自定义redis操作类
 * **/
class myRedis{
    public $handler = NULL;
    public function __construct($conf){
        $this->handler = new Redis();
        $this->handler->connect($conf['host'], $conf['port']); //连接Redis
        //设置密码
        if(isset($conf['auth'])){
            $this->handler->auth($conf['auth']); //密码验证
        }
        //选择数据库
        if(isset($conf['db'])){
            $this->handler->select($conf['db']);//选择数据库2
        }else{
            $this->handler->select(0);//默认选择0库
        }
    }

    //获取key的值
    public function get($name){
        return $this->handler->get($name);
    }
    
    //设置key的值
    public function set($name,$value){
        return $this->handler->set($name,$value);
    }

    //判断key是否存在
    public function exists($key){
        if($this->handler->exists($key)){
            return true;
        }
        return false;
    }

    //当key不存在的设置key的值，存在则不设置
    public function setnx($key,$value){
        return $this->handler->setnx($key,$value);
    }

    //将key的数值增加指定数值
    public function incrby($key,$value){
        return $this->handler->incrBy($key,$value);
    }
    
}
```

#### 抢购业务实现 index.php

```php
<?php
require_once './myRedis.php';
require_once './function.php';

class sendAward{
    public $conf = [];
    const V1 = 'way1';//版本一
    const V2 = 'way2';//版本二
    const AMOUNTLIMIT = 5;//抢购数量限制
    const INCRAMOUNT = 1;//redis递增数量值
    
    //初始化调用对应方法执行商品发放
    public function __construct($conf,$type){
        $this->conf = $conf;
        if(empty($type))
            return '';
        if($type==self::V1){
            $this->way1(self::V1);
        }elseif($type==self::V2){
            $this->way2(self::V2);
        }else{
            return '';
        }
    }
    
    //抢购商品方式一
    protected  function way1($v){
        $redis = new myRedis($this->conf);      
        $keyNmae = getKeyName($v);
        if(!$redis->exists($keyNmae)){
            $redis->set($keyNmae,0);
        }
        $currAmount = $redis->get($keyNmae);
        if(($currAmount+self::INCRAMOUNT)>self::AMOUNTLIMIT){
            writeLog("没有抢到商品",$v);
            return;
        }
        $redis->incrby($keyNmae,self::INCRAMOUNT);
        writeLog("抢到商品",$v);
    }
    
    //抢购商品方式二
    protected function way2($v){
        $redis = new myRedis($this->conf);
        $keyNmae = getKeyName($v);
        if(!$redis->exists($keyNmae)){
            $redis->setnx($keyNmae,0);
        }
        if($redis->incrby($keyNmae,self::INCRAMOUNT) > self::AMOUNTLIMIT){
            writeLog("没有抢到商品",$v);
            return;
        }
        writeLog("抢到商品",$v);
    }
            
}

//实例化调用对应执行方法
$type = isset($_GET['v'])?$_GET['v']:'way1';
$conf = [
    'host'=>'192.168.0.214','port'=>'6379',
    'auth'=>'test','db'=>2,
];
new sendAward($conf,$type);
/***
通过ab工具压力测试模拟超发的情况，再结合日志打印的数据说明方法可以有效的防止超发
ab -c 100 -n 200 http://192.168.0.213:8083/index.php?v=way2
ab -c 100 -n 200 http://192.168.0.213:8083/index.php?v=way2
**/
```

####Redis支持哪些数据结构及相应的使用场景

字符串（string）
哈希（map）
列表（list）
集合（sets）
有序集合（sorted sets）
redis特点
单线程
数据结构服务器（易于处理集合运算）
快（每秒10万次set操作）
拥有很多原子操作方法，容易保证数据一致性
兼具临时性和永久性
无中心的分布式集群
(1) 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
(2) 支持丰富数据类型，支持string，list，set，sorted set，hash
(3) 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
(4) 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

redis 实战操作RDB和AOF快照持久化
rdb 和 aof 两种持久化机制
rdb 原理每隔一段时间生成redis内存中的数据的一份完整的快照
触发持久化的机制有手动触发和自动触发
手动触发是执行bgsave命令，bgsave会fork一个子进程，在子进程里备份数据
自动触发，/etc/redis/6379.conf文件,手动配置检查点,一条save配置代表一个检查点，redis默认的机制是RDB所以不涉及开启或不开启。例如,我想每隔5s,只要有一条数据更新就生成快照 save 5 1
AOF持久化机制。
原理有数据写入redis,redis自身就会将数据写入aof日志文件,redis并不是直接写入aof文件,而是先写到os cache,然后到一定时间再从os cache会触发操作系统的fsync操作写到磁盘 AOF会无限制的增加吗? 不会,redis中的数据是有一定限量的,不可能说redis内存中的数据无限增长,进而导致AOF无限增长,内存大小是一定的,到一定时候,redis就会用缓存淘汰算法,LRU,自动将一部分数据从内存中给清除,AOF是存放每条写命令的,所以会不断的膨胀,当大到一定的时候,AOF做rewrite操作,将AOF变得小一些,然后将旧的删掉 例如,当redis进行了清理,清除了一部分数据,AOF量到一定程度时,会rewrite,根据redis中新的数据进行rewrite,从而将aof变小,然后将旧的删掉
手动触发。执行bgrewriteaof命令即可
自动触发
由于redis默认的是RDB的持久化方式,AOF的方式默认是关闭的,所以,我们需要手动开启,我们打开/etc/redis/6379.conf配置文件 a.打开AOF机制 appendonly yes
配置rewrite 在/etc/redis/6379.conf文件中有两个配置,auto-aof-rewrite-percentage和auto-aof-rewrite-min-size 第一条命令的意思是,当aof日志文件中的大小大于上一次的一倍了,那就执行rewrite 第二条意思是,就算满足了第一条的条件,但是还是需要和auto-aof-rewrite-min-size配置的值进行比较,当aof的大小大于64m时才会进行rewrite操作
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
OF的三种写入方式,使用默认的everysec即可,即,每秒写入一次
# appendfsync always
appendfsync everysec
# appendfsync no
####数据恢复方案

a.如果是redis进程挂掉，那么重启redis进程即可，直接基于AOF日志文件恢复数据
b.如果是redis进程所在机器挂掉，那么重启机器后，尝试重启redis进程，尝试直接基于AOF日志文件进行数据恢复
c.如果redis当前最新的AOF和RDB文件出现了丢失/损坏，那么可以尝试基于该机器上当前的某个最新的RDB数据副本进行数据恢复 ①停止redis(命令是redis-cli shutdown), ②在配置文件中关闭aof: appendonly no ③拷贝rdb日志备份到/var/redis/6379目录下 ④启动redis(命令是,先到目录/etc/init.d/目录下, ./redis_6379) ⑤尝试get一个key,确认数据恢复 ⑥命令热修改redis配置,使用redis-cli连接redis,使用命令redis config set appendonly yes打开aof方式,这样aof和rdb数据就一致了 ⑦手动修改6379.conf配置文件中appendonly为yes,因为热修改暂时不会写到配置文件中,所以需要手动修改,然后启动redis,再次确认数据恢复
d.如果当前机器上的所有RDB文件全部损坏，那么从远程的云服务上拉取最新的RDB快照回来恢复数据
c.如果是发现有重大的数据错误,就找最新的并且无错的进行恢复
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"

####redis常见性能问题和解决方案

Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次
为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
尽量避免在压力很大的主库上增加从库
主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1 <- Slave2 <- Slave3... 这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变。
Master写内存快照，save命令调度rdbSave函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以Master最好不要写内存快照。
Master AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响Master重启的恢复速度。Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。
Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。

####MySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。redis 提供 6种数据淘汰策略：
voltile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
no-enviction（驱逐）：禁止驱逐数据

####Redis的同步机制了解么？

主从同步。第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将rdb文件全量同步到复制节点，复制节点接受完成后将rdb镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程

####是否使用过Redis集群，集群的原理是什么？

Redis Sentinel着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。
Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

####memache 和redis 内存分配与碎片回收机制

https://blog.csdn.net/jayxujia123/article/details/88544702
memache使用了slab allocator机制
Slab Allocator的基本原理是按照预先规定的大小，将分配的内存以page为单位，默认情况下一个page是1M，可以通过-I参数在启动时指定，分割成各种尺寸的块（chunk）， 并把尺寸相同的块分成组（chunk的集合），如果需要申请内存时，memcached会划分出一个新的page并分配给需要的slab区域
redis的内存机制
Redis的内存管理主要通过源码中zmalloc.h和zmalloc.c两个文件来实现的。Redis为了方便内存的管理，在分配一块内存之 后，会将这块内存的大小存入内存块的头部。如图 5所示，real_ptr是redis调用malloc后返回的指针。redis将内存块的大小size存入头部，size所占据的内存大小是已知的，为 size_t类型的长度，然后返回ret_ptr。当需要释放内存的时候，ret_ptr被传给内存管理程序。通过ret_ptr，程序可以很容易的算出 real_ptr的值，然后将real_ptr传给free释放内存
Redis通过定义一个数组来记录所有的内存分配情况，这个数组的长度为ZMALLOC_MAX_ALLOC_STAT。数组的每一个元素代表当前 程序所分配的内存块的个数，且内存块的大小为该元素的下标。在源码中，这个数组为zmalloc_allocations。 zmalloc_allocations[16]代表已经分配的长度为16bytes的内存块的个数。zmalloc.c中有一个静态变量 used_memory用来记录当前分配的内存总大小。所以，总的来看，Redis采用的是包装的mallc/free，相较于Memcached的内存 管理方法来说，要简单很多
php中类的使用



####redis 命令列表

del 删除给定的一个或多个 key 。不存在的 key 会被忽略。返回被删除key的数量。O(N)， N 为被删除的 key 的数量。删除单个字符串类型的 key ，时间复杂度为O(1)。删除单个列表、集合、有序集合或哈希表类型的 key ，时间复杂度为O(M)， M 为以上数据结构内的元素数量。
dump 。序列化给定 key ，并返回被序列化的值，使用 RESTORE 命令可以将这个值反序列化为 Redis 键
exists 检查给定 key 是否存在。若 key 存在，返回 1 ，否则返回 0 。
expire key seconds 设置过期时间。设置成功返回 1
expireat key timestamp 设置过期时间，后面跟的是unix 时间戳
KEYS pattern 查找所有符合给定模式 pattern 的 key。
ttl -2表示不存在key -1 表示永不过期
KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS h*llo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。
KEYS 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的 key ，你最好还是用 Redis 的集合结构(set)来代替。
migrate host port key destination-db timeout [COPY] [REPLACE] 迁移实例key
MOVE key db 将当前数据库的 key 移动到给定的数据库 db 当中
object subcommand [arguments [arguments]] OBJECT 命令允许从内部察看给定 key 的 Redis 对象
OBJECT 命令有多个子命令：

OBJECT REFCOUNT <key> 返回给定 key 引用所储存的值的次数。此命令主要用于除错。
OBJECT ENCODING <key> 返回给定 key 锁储存的值所使用的内部表示(representation)。
OBJECT idletime <key> 返回给定 key 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位。
persist key 移除key的时间，转换为永久key
pexpire p-expire跟expire类似，存储毫秒
pexpireat 毫秒的时间戳
pttl 返回毫秒生存时间
randomkey 随机返回一个key
rename key newkey 修改key名字
renamenx key newkey 当且仅当 newkey 不存在时，将 key 改名为 newkey
restore key ttl serialized-value 反序列化给定的序列化值，并将它和给定的 key 关联。个人理解可以将反序列化的值赋值给一个新key
sort key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination] 返回或保存给定列表、集合、有序集合 key 中经过排序的元素
type key 返回 key 所储存的值的类型
scan cursor [MATCH pattern] [COUNT count]
SCAN 命令用于迭代当前数据库中的数据库键。
SSCAN 命令用于迭代集合键中的元素。
HSCAN 命令用于迭代哈希键中的键值对。
ZSCAN 命令用于迭代有序集合中的元素（包括元素成员和元素分值）。
redis 字符串

APPEND key value
如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。
如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。
bitcount key [start] [end]计算给定字符串中，被设置为 1 的比特位的数量。
Bitmap 对于一些特定类型的计算非常有效。

假设现在我们希望记录自己网站上的用户的上线频率，比如说，计算用户 A 上线了多少天，用户 B 上线了多少天，诸如此类，以此作为数据，从而决定让哪些用户参加 beta 测试等活动 —— 这个模式可以使用 SETBIT 和 BITCOUNT 来实现。

比如说，每当用户在某一天上线的时候，我们就使用 SETBIT ，以用户名作为 key ，将那天所代表的网站的上线日作为 offset 参数，并将这个 offset 上的为设置为 1 。

举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么执行命令 SETBIT peter 100 1 ；如果明天 peter 也继续阅览网站，那么执行命令 SETBIT peter 101 1 ，以此类推。

当要计算 peter 总共以来的上线次数时，就使用 BITCOUNT 命令：执行 BITCOUNT peter ，得出的结果就是 peter 上线的总天数。
可用于打卡统计天数
bitop operation destkey key [key ...] 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
decr key 数字减1
decrby key decrement 将 key 所储存的值减去减量 decrement 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECRBY 操作
get key 返回 key 所关联的字符串值
getbit key offset 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
getrange key start end 字符串截取，2.0之前版本叫做substr
getset key value 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
incr key 将 key 中储存的数字值增一
incrby key increment 增加相应数量的值
incrbyfloat key increment 增加浮点数,也可以是负数
mget key [key ...] 返回所有(一个或多个)给定 key 的值。复杂度o(n)
mset key value [key value ...] 同时设置一个或多个 key-value 对
msetnx key value [key value ...] 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
psetex key milliseconds value 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
setbit key offset value 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
SETEX key seconds value 将值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)。
SETNX key value 将 key 的值设为 value ，当且仅当 key 不存在。
setrange key offset value 用 value 参数覆写(overwrite)给定 key 所储存的字符串值，从偏移量 offset 开始
hash表

hdel key field [field ...] 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
在Redis2.4以下的版本里， HDEL 每次只能删除单个域，如果你需要在一个原子时间内删除多个域，请将命令包含在 MULTI / EXEC 块内。
hexists key field 判断hash里有没有某个key存在
hget key field 返回哈希表 key 中给定域 field 的值。
hgetall key 返回哈希表 key 中，所有的域和值。
hincrby key field increment 给某个int类型增加或减少数字.如果 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令
hincrbyfloat key field increment 为哈希表 key 中的域 field 加上浮点数增量 increment 。
hkeys key 返回哈希表 key 中的所有域。
hlen key 返回哈希表 key 中域的数量。
hmget key field [field ...] 返回哈希表 key 中，一个或多个给定域的值。
hmset key field value [field value ...] 同时将多个 field-value (域-值)对设置到哈希表 key 中。
hset key field value 将哈希表 key 中的域 field 的值设为 value 。
hsetnx key field value 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在。
hvals key 返回哈希表 key 中所有域的值
hscan
list liebiao

blpop|brpop key [key ...] timeout 它是 LPOP 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。timeout=0,一直堵塞。他会轮询你当前的key，当所有的为nil时再堵塞。b是blocking的意思
brpoplpush | rpoplpush source destination timeout 将列表source最右边元素弹出放到destination列表的最左边。b有堵塞功能，如果source和destination相同，则列表中的表尾元素被移动到表头，并返回该元素，称为列表的旋转
lindex key index 返回列表 key 中，下标为 index 的元素,也可以使用负数下表
linsert key before|after pivot value
将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
当 pivot 不存在于列表 key 时，不执行任何操作。
当 key 不存在时， key 被视为空列表，不执行任何操作。
如果 key 不是列表类型，返回一个错误。
LINSERT mylist BEFORE "World" "There"
llen key 返回列表长度
lpushx key value 当列表不为空且列表存在才插入
lrange key start stop 返回列表指定区间的元素
lrem key count value 根据参数 count 的值，移除列表中与参数 value 相等的元素。
count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
count = 0 : 移除表中所有与 value 相等的值。
lset key index value 将列表 key 下标为 index 的元素的值设置为 value 。
ltrim key start stop 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
set集合

sadd key member [member ...] 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
scard key 返回集合中的基数
sdiff key [key ...] 返回一个集合的全部成员，该集合是所有给定集合之间的差集。
sdiffstore | sinterstore destination key [key ...] 这个命令的作用和 SDIFF 类似，但它将结果保存到 destination 集合，而不是简单地返回结果集。
sinter key [key ...] 返回一个集合的全部成员，该集合是所有给定集合的交集。
sismember key member 判断 member 元素是否集合 key 的成员
smembers 返回集合 key 中的所有成员。
smove source destination member 将 member 元素从 source 集合移动到 destination 集合。
spop key 移除并返回集合中的一个随机元素。
srandmember key [count]
如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。
srem key member [member ...] 移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。
sunion key [key ...] 返回一个集合的全部成员，该集合是所有给定集合的并集。自动去重的
sunionstore destination key [key ...] 集合存储
scan
有序集合

zadd key score member [[score member] [score member] ...] 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
zcard key 返回有序集 key 的基数。
zcount key min max 返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量。
zincrby key increment member 为有序集 key 的成员 member 的 score 值加上增量 increment 。也可以是负数
zrange key start stop [WITHSCORES] 返回有序集 key中，指定区间内的成员。其中成员的位置按 score 值递增(从小到大)来排序。
zrangebyscore key min max [WITHSCORES] [LIMIT offset count] 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。
min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 ZRANGEBYSCORE 这类命令。
ZRANGEBYSCORE salary -inf +inf               # 显示整个有序集
zrank key member 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。排名最低是0
zrem key member [member ...] 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略
zremrangebyrank key start stop 移除有序集 key 中，指定排名(rank)区间内的所有成员。
zremrangebyscore key min max 移除有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。
zrevrange key start stop [WITHSCORES] 返回有序集 key 中，指定区间内的成员。其中成员的位置按 score值递减(从大到小)来排列。rev 就是逆序，reverse的意思
zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count] 返回有序集 key 中， score 值介于 max 和 min 之间(默认包括等于 max 或 min )的所有的成员。有序集成员按 score 值递减(从大到小)的次序排列。
zrevrank key member 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)排序。
zscore key member 返回有序集 key 中，成员 member 的 score 值。
zunionstore destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
计算给定的一个或多个有序集的并集，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination 。
默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之 和 。
WEIGHTS
使用 WEIGHTS 选项，你可以为 每个 给定有序集 分别 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 score 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。
如果没有指定 WEIGHTS 选项，乘法因子默认设置为 1 。
AGGREGATE
使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。
默认使用的参数 SUM ，可以将所有集合中某个成员的 score 值之 和 作为结果集中该成员的 score 值；使用参数 MIN ，可以将所有集合中某个成员的 最小 score 值作为结果集中该成员的 score 值；而参数 MAX 则是将所有集合中某个成员的 最大 score 值作为结果集中该成员的 score 值。
zinterstore destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] 计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 。
zscan
geohash 算法，给经纬度编码，计算周边信息

一致性哈希算法

https://www.jb51.net/article/137523.htm
主要解决问题分布式的负载问题，即使新增或者删除节点，影响的只是一部分的数据负载。减轻系统的压力
红包分配方法

https://blog.csdn.net/dragon_18/article/details/86378979
redis scan 用法

scan cursor match * count 200
实际上scan后面的count不是要输出的数量。而是要迭代的所有key里的个数。我的理解就是redis里有10000个key。如果我想一次性全部scan出来，我必须把这个count大于redis里所有的key的个数。就类似于o(n)遍历，只不过scan不堵塞，是迭代性的，类似于php和python的协程 yield。
cursor比较有意思。当你后面的count数量太小的时候，或者可以理解为比key的总数小，那么他肯定迭代不完你所有的数据。这个输出的cursor就是目前redis迭代到的位置，如果还想继续输出，可以把cursor替换成这个值，继续输出。如果为0，则表示整个库里都迭代完了

####redis rdb恢复要点
必须 shutdown
rdb恢复目录可以通过 config get dir查看
必须appendonly = no

####redis 持久化机制和线上环境容灾

https://blog.csdn.net/zh15732621679/article/details/80307827
https://www.cnblogs.com/chenhaoyu/p/10837723.html 

####mySQL 里有 2000w 数据，redis 中只存 20w 的数据，如何保证 redis 中的数据都是热点数据

相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略（回收策略）。redis 提供 6 种数据淘汰策略：

volatile-lru：从已设置过期时间的数据集（server.db [i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db [i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db [i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db [i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db [i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

- Redis、Memecached 这两者有什么区别？

> 1.  Redis 支持更加丰富的数据存储类型，String、Hash、List、Set 和 Sorted Set。Memcached 仅支持简单的 key-value 结构。
> 2.  Memcached key-value存储比 Redis 采用 hash 结构来做 key-value 存储的内存利用率更高。
> 3.  Redis 提供了事务的功能，可以保证一系列命令的原子性
> 4.  Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中
> 5.  Redis 只使用单核，而 Memcached 可以使用多核，所以平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。

- Redis 如何实现持久化？

> 1. RDB 持久化，将 Redis 在内存中的的状态保存到硬盘中，相当于备份数据库状态。
> 2. AOF 持久化（Append-Only-File），AOF 持久化是通过保存 Redis 服务器锁执行的写状态来记录数据库的。相当于备份数据库接收到的命令，所有被写入 AOF 的命令都是以 Redis 的协议格式来保存的。









**Redis 是什么**

Redis 是 C 语言开发的一个开源的（遵从 BSD 协议）高性能键值对（key-value）的内存数据库，可以用作数据库、缓存、消息中间件等。

它是一种 NoSQL（not-only sql，泛指非关系型数据库）的数据库。

Redis 作为一个内存数据库：

性能优秀，数据在内存中，读写速度非常快，支持并发 10W QPS。单进程单线程，是线程安全的，采用 IO 多路复用机制。
丰富的数据类型，支持字符串（strings）、散列（hashes）、列表（lists）、集合（sets）、有序集合（sorted sets）等。
支持数据持久化。可以将内存中数据保存在磁盘中，重启时加载。主从复制，哨兵，高可用。
可以用作分布式锁。
可以作为消息中间件使用，支持发布订阅。

**五种数据类型**

说之前，我觉得有必要先来了解下 Redis 内部内存管理是如何描述这 5 种数据类型的。

<img src="./images/redis-1.jpeg" style="zoom:80%;" />

redisObject 最主要的信息如上图所示：type 表示一个 value 对象具体是何种数据类型，encoding 是不同数据类型在 Redis 内部的存储方式。

比如：type=string 表示 value 存储的是一个普通字符串，那么 encoding 可以是 raw 或者 int。

下面我简单说下 5 种数据类型：

①String 是 Redis 最基本的类型，可以理解成与 Memcached一模一样的类型，一个 Key 对应一个 Value。Value 不仅是 String，也可以是数字。

String 类型是二进制安全的，意思是 Redis 的 String 类型可以包含任何数据，比如 jpg 图片或者序列化的对象。String 类型的值最大能存储 512M。

②Hash是一个键值（key-value）的集合。Redis 的 Hash 是一个 String 的 Key 和 Value 的映射表，Hash 特别适合存储对象。常用命令：hget，hset，hgetall 等。

③List 列表是简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部（左边）或者尾部（右边） 常用命令：lpush、rpush、lpop、rpop、lrange（获取列表片段）等。

应用场景：List 应用场景非常多，也是 Redis 最重要的数据结构之一，比如 Twitter 的关注列表，粉丝列表都可以用 List 结构来实现。

数据结构：List 就是链表，可以用来当消息队列用。Redis 提供了 List 的 Push 和 Pop 操作，还提供了操作某一段的 API，可以直接查询或者删除某一段的元素。

实现方式：Redis List 的是实现是一个双向链表，既可以支持反向查找和遍历，更方便操作，不过带来了额外的内存开销。

④Set 是 String 类型的无序集合。集合是通过 hashtable 实现的。Set 中的元素是没有顺序的，而且是没有重复的。常用命令：sdd、spop、smembers、sunion 等。

应用场景：Redis Set 对外提供的功能和 List 一样是一个列表，特殊之处在于 Set 是自动去重的，而且 Set 提供了判断某个成员是否在一个 Set 集合中。

⑤Zset 和 Set 一样是 String 类型元素的集合，且不允许重复的元素。常用命令：zadd、zrange、zrem、zcard 等。

使用场景：Sorted Set 可以通过用户额外提供一个优先级（score）的参数来为成员排序，并且是插入有序的，即自动排序。

当你需要一个有序的并且不重复的集合列表，那么可以选择 Sorted Set 结构。

和 Set 相比，Sorted Set关联了一个 Double 类型权重的参数 Score，使得集合中的元素能够按照 Score 进行有序排列，Redis 正是通过分数来为集合中的成员进行从小到大的排序。

实现方式：Redis Sorted Set 的内部使用 HashMap 和跳跃表（skipList）来保证数据的存储和有序，HashMap 里放的是成员到 Score 的映射。

而跳跃表里存放的是所有的成员，排序依据是 HashMap 里存的 Score，使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。

数据类型应用场景总结：

![](./images/redis-2.png)

**Redis 缓存**



缓存和数据库数据一致性问题：分布式环境下非常容易出现缓存和数据库间数据一致性问题，针对这一点，如果项目对缓存的要求是强一致性的，那么就不要使用缓存。

我们只能采取合适的策略来降低缓存和数据库间数据不一致的概率，而无法保证两者间的强一致性。

合适的策略包括合适的缓存更新策略，更新数据库后及时更新缓存、缓存失败时增加重试机制。



**Redis 雪崩了解吗？**

目前电商首页以及热点数据都会去做缓存，一般缓存都是定时任务去刷新，或者查不到之后去更新缓存的，定时任务刷新就有一个问题。

举个栗子：如果首页所有 Key 的失效时间都是 12 小时，中午 12 点刷新的，我零点有个大促活动大量用户涌入，假设每秒 6000 个请求，本来缓存可以抗住每秒 5000 个请求，但是缓存中所有 Key 都失效了。

此时 6000 个/秒的请求全部落在了数据库上，数据库必然扛不住，真实情况可能 DBA 都没反应过来直接挂了。

此时，如果没什么特别的方案来处理，DBA 很着急，重启数据库，但是数据库立马又被新流量给打死了。这就是我理解的缓存雪崩。

同一时间大面积失效，瞬间 Redis 跟没有一样，那这个数量级别的请求直接打到数据库几乎是灾难性的。

你想想如果挂的是一个用户服务的库，那其他依赖他的库所有接口几乎都会报错。

如果没做熔断等策略基本上就是瞬间挂一片的节奏，你怎么重启用户都会把你打挂，等你重启好的时候，用户早睡觉去了，临睡之前，骂骂咧咧“什么垃圾产品”。

处理缓存雪崩简单，在批量往 Redis 存数据的时候，把每个 Key 的失效时间都加个随机值就好了，这样可以保证数据不会再同一时间大面积失效。

setRedis（key, value, time+Math.random()*10000）;

如果 Redis 是集群部署，将热点数据均匀分布在不同的 Redis 库中也能避免全部失效。

或者设置热点数据永不过期，有更新操作就更新缓存就好了（比如运维更新了首页商品，那你刷下缓存就好了，不要设置过期时间），电商首页的数据也可以用这个操作，保险。

**那你了解缓存穿透和击穿么，可以说说他们跟雪崩的区别吗？**

先说下缓存穿透吧，缓存穿透是指缓存和数据库中都没有的数据，而用户（黑客）不断发起请求。

举个栗子：我们数据库的 id 都是从 1 自增的，如果发起 id=-1 的数据或者 id 特别大不存在的数据，这样的不断攻击导致数据库压力很大，严重会击垮数据库。

至于缓存击穿嘛，这个跟缓存雪崩有点像，但是又有一点不一样，缓存雪崩是因为大面积的缓存失效，打崩了 DB。

而缓存击穿不同的是缓存击穿是指一个 Key 非常热点，在不停地扛着大量的请求，大并发集中对这一个点进行访问，当这个 Key 在失效的瞬间，持续的大并发直接落到了数据库上，就在这个 Key 的点上击穿了缓存。

缓存穿透我会在接口层增加校验，比如用户鉴权，参数做校验，不合法的校验直接 return，比如 id 做基础校验，id<=0 直接拦截。

Redis 里还有一个高级用法布隆过滤器（Bloom Filter）这个也能很好的预防缓存穿透的发生。

它的原理也很简单，就是利用高效的数据结构和算法快速判断出你这个 Key 是否在数据库中存在，不存在你 return 就好了，存在你就去查 DB 刷新 KV 再 return。

缓存击穿的话，设置热点数据永不过期，或者加上互斥锁就搞定了。

**Redis 为何这么快**

官方提供的数据可以达到 100000+ 的 QPS（每秒内的查询次数），这个数据不比 Memcached 差！

Redis 这么快，为什么还是单线程的吧。

Redis 确实是单进程单线程的模型，因为 Redis 完全是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。

既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章的采用单线程的方案了（毕竟采用多线程会有很多麻烦）。

Redis 是单线程的，为什么还能这么快吗？

有如下四点：

Redis 完全基于内存，绝大部分请求是纯粹的内存操作，非常迅速，数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度是 O(1)。

数据结构简单，对数据操作也简单。

采用单线程，避免了不必要的上下文切换和竞争条件，不存在多线程导致的 CPU 切换，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有死锁问题导致的性能消耗。

使用多路复用 IO 模型，非阻塞 IO。

Redis 和 Memcached 的区别

为什么选择 Redis 的缓存方案而不用 Memcached 呢？

原因有如下四点：

存储方式上：Memcache 会把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。Redis 有部分数据存在硬盘上，这样能保证数据的持久性。

数据支持类型上：Memcache 对数据类型的支持简单，只支持简单的 key-value，，而 Redis 支持五种数据类型。

使用底层模型不同：它们之间底层实现方式以及与客户端之间通信的应用协议不一样。Redis 直接自己构建了 VM 机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

Value 的大小：Redis 可以达到 1GB，而 Memcache 只有 1MB。

**淘汰策略**

Redis 有六种淘汰策略，如下图：

![](./images/redis-3.png)

Redis 4.0 加入了 LFU（least frequency use）淘汰策略，包括 volatile-lfu 和 allkeys-lfu，通过统计访问频率，将访问频率最少，即最不经常使用的 KV 淘汰。

**Redis持久化机制**

Redis 为了保证效率，数据缓存在了内存中，但是会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件中，以保证数据的持久化。

**Redis 的持久化策略有两种：**

**RDB：**

快照形式是直接把内存中的数据保存到一个 dump 的文件中，定时保存，保存策略。AOF：把所有的对 Redis 的服务器进行修改的命令都存到一个文件里，命令的集合。Redis 默认是快照 RDB 的持久化方式。

当 Redis 重启的时候，它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。你甚至可以关闭持久化功能，让数据只在服务器运行时存。

默认 Redis 是会以快照"RDB"的形式将数据持久化到磁盘的一个二进制文件 dump.rdb。

当 Redis 需要做持久化时，Redis 会 fork 一个子进程，子进程将数据写到磁盘上一个临时 RDB 文件中。

当子进程完成写临时文件后，将原来的 RDB 替换掉，这样的好处是可以 copy-on-write。

RDB 的优点是：这种文件非常适合用于备份：比如，你可以在最近的 24 小时内，每小时备份一次，并且在每个月的每一天也备份一个 RDB 文件。

这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。RDB 非常适合灾难恢复。

RDB 的缺点是：如果你需要尽量避免在服务器故障时丢失数据，那么RDB不合适你。

**AOF：**

使用 AOF 做持久化，每一个写命令都通过 write 函数追加到 appendonly.aof 中，配置方式如下：

appendfsyncyesappendfsync always #每次有数据修改发生时都会写入AOF文件。appendfsync everysec #每秒钟同步一次，该策略为AOF的缺省策略。

AOF 可以做到全程持久化，只需要在配置中开启 appendonly yes。这样 Redis 每执行一个修改数据的命令，都会把它添加到 AOF 文件中，当 Redis 重启时，将会读取 AOF 文件进行重放，恢复到 Redis 关闭前的最后时刻。

使用 AOF 的优点是会让 Redis 变得非常耐久。可以设置不同的 Fsync 策略，AOF的默认策略是每秒钟 Fsync 一次，在这种配置下，就算发生故障停机，也最多丢失一秒钟的数据。

缺点是对于相同的数据集来说，AOF 的文件体积通常要大于 RDB 文件的体积。根据所使用的 Fsync 策略，AOF 的速度可能会慢于 RDB。

**RDB和AOF如何选择**

如果你非常关心你的数据，但仍然可以承受数分钟内的数据丢失，那么可以额只使用 RDB 持久。

AOF 将 Redis 执行的每一条命令追加到磁盘中，处理巨大的写入会降低Redis的性能，不知道你是否可以接受。

数据库备份和灾难恢复：定时生成 RDB 快照非常便于进行数据库备份，并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度快。

当然了，Redis 支持同时开启 RDB 和 AOF，系统重启后，Redis 会优先使用 AOF 来恢复数据，这样丢失的数据会最少。

**主从复制**

Redis 单节点存在单点故障问题，为了解决单点问题，一般都需要对 Redis 配置从节点，然后使用哨兵来监听主节点的存活状态，如果主节点挂掉，从节点能继续提供缓存功能，你能说说 Redis 主从复制的过程和原理吗？

主从配置结合哨兵模式能解决单点故障问题，提高 Redis 可用性。

从节点仅提供读操作，主节点提供写操作。对于读多写少的状况，可给主节点配置多个从节点，从而提高响应效率。

关于复制过程，是这样的：

从节点执行 slaveof[masterIP][masterPort]，保存主节点信息。从节点中的定时任务发现主节点信息，建立和主节点的 Socket 连接。从节点发送 Ping 信号，主节点返回 Pong，两边能互相通信。连接建立后，主节点将所有数据发送给从节点（数据同步）。主节点把当前的数据同步给从节点后，便完成了复制的建立过程。接下来，主节点就会持续的把写命令发送给从节点，保证主从数据一致性。

**那你能详细说下数据同步的过程吗？**

Redis 2.8 之前使用 sync[runId][offset] 同步命令，Redis 2.8 之后使用 psync[runId][offset] 命令。

两者不同在于，Sync 命令仅支持全量复制过程，Psync 支持全量和部分复制。

介绍同步之前，先介绍几个概念：

runId：每个 Redis 节点启动都会生成唯一的 uuid，每次 Redis 重启后，runId 都会发生变化。offset：主节点和从节点都各自维护自己的主从复制偏移量 offset，当主节点有写入命令时，offset=offset+命令的字节长度。从节点在收到主节点发送的命令后，也会增加自己的 offset，并把自己的 offset 发送给主节点。这样，主节点同时保存自己的 offset 和从节点的 offset，通过对比 offset 来判断主从节点数据是否一致。repl_backlog_size：保存在主节点上的一个固定长度的先进先出队列，默认大小是 1MB。

主节点发送数据给从节点过程中，主节点还会进行一些写操作，这时候的数据存储在复制缓冲区中。

从节点同步主节点数据完成后，主节点将缓冲区的数据继续发送给从节点，用于部分复制。

主节点响应写命令时，不但会把命名发送给从节点，还会写入复制积压缓冲区，用于复制命令丢失的数据补救。

![](./images/redis-4.jpeg)

上面是 Psync 的执行流程，从节点发送 psync[runId][offset] 命令，主节点有三种响应：

FULLRESYNC：第一次连接，进行全量复制CONTINUE：进行部分复制ERR：不支持 psync 命令，进行全量复制

**全量复制和部分复制的过程**

![](./images/redis-5.jpeg)

上面是全量复制的流程。主要有以下几步：

从节点发送 psync ? -1 命令（因为第一次发送，不知道主节点的 runId，所以为?，因为是第一次复制，所以 offset=-1）。主节点发现从节点是第一次复制，返回 FULLRESYNC {runId} {offset}，runId 是主节点的 runId，offset 是主节点目前的 offset。从节点接收主节点信息后，保存到 info 中。主节点在发送 FULLRESYNC 后，启动 bgsave 命令，生成 RDB 文件（数据持久化）。主节点发送 RDB 文件给从节点。到从节点加载数据完成这段期间主节点的写命令放入缓冲区。从节点清理自己的数据库数据。从节点加载 RDB 文件，将数据保存到自己的数据库中。如果从节点开启了 AOF，从节点会异步重写 AOF 文件。

关于部分复制有以下几点说明：

①部分复制主要是 Redis 针对全量复制的过高开销做出的一种优化措施，使用 psync[runId][offset] 命令实现。

当从节点正在复制主节点时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向主节点要求补发丢失的命令数据，主节点的复制积压缓冲区将这部分数据直接发送给从节点。

这样就可以保持主从节点复制的一致性。补发的这部分数据一般远远小于全量数据。

②主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内的复制积压缓冲区依然可以保存最近一段时间的写命令数据。

③当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行 ID。因此会把它们当做 psync 参数发送给主节点，要求进行部分复制。

④主节点接收到 psync 命令后首先核对参数 runId 是否与自身一致，如果一致，说明之前复制的是当前主节点。

之后根据参数 offset 在复制积压缓冲区中查找，如果 offset 之后的数据存在，则对从节点发送+COUTINUE 命令，表示可以进行部分复制。因为缓冲区大小固定，若发生缓冲溢出，则进行全量复制。

⑤主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。

**哨兵**

**主从复制会存在的问题**

一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预。主节点的写能力受到单机的限制。主节点的存储能力受到单机的限制。原生复制的弊端在早期的版本中也会比较突出，比如：Redis 复制中断后，从节点会发起 psync。此时如果同步不成功，则会进行全量同步，主库执行全量备份的同时，可能会造成毫秒或秒级的卡顿。

**比较主流的解决方案**

是哨兵

哨兵有哪些功能？

![](./images/redis-6.jpeg)

如图，是 Redis Sentinel（哨兵）的架构图。Redis Sentinel（哨兵）主要功能包括主节点存活检测、主从运行情况检测、自动故障转移、主从切换。

Redis Sentinel 最小配置是一主一从。Redis 的 Sentinel 系统可以用来管理多个 Redis 服务器。

该系统可以执行以下四个任务：

监控：不断检查主服务器和从服务器是否正常运行。通知：当被监控的某个 Redis 服务器出现问题，Sentinel 通过 API 脚本向管理员或者其他应用程序发出通知。自动故障转移：当主节点不能正常工作时，Sentinel 会开始一次自动的故障转移操作，它会将与失效主节点是主从关系的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点，这样人工干预就可以免了。配置提供者：在 Redis Sentinel 模式下，客户端应用在初始化时连接的是 Sentinel 节点集合，从中获取主节点的信息。

**哨兵的工作原理**

直接上图：

![](./images/redis-7.jpeg)

①每个 Sentinel 节点都需要定期执行以下任务：每个 Sentinel 以每秒一次的频率，向它所知的主服务器、从服务器以及其他的 Sentinel 实例发送一个 PING 命令。（如上图）

![](./images/redis-8.jpeg)

②如果一个实例距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 所指定的值，那么这个实例会被 Sentinel 标记为主观下线。（如上图）

![](./images/redis-9.jpeg)

③如果一个主服务器被标记为主观下线，那么正在监视这个服务器的所有 Sentinel 节点，要以每秒一次的频率确认主服务器的确进入了主观下线状态。

![](./images/redis-10.jpeg)

④如果一个主服务器被标记为主观下线，并且有足够数量的 Sentinel（至少要达到配置文件指定的数量）在指定的时间范围内同意这一判断，那么这个主服务器被标记为客观下线。

![](./images/redis-11.jpeg)

⑤一般情况下，每个 Sentinel 会以每 10 秒一次的频率向它已知的所有主服务器和从服务器发送 INFO 命令。

当一个主服务器被标记为客观下线时，Sentinel 向下线主服务器的所有从服务器发送 INFO 命令的频率，会从 10 秒一次改为每秒一次。

![](./images/redis-12.jpeg)

⑥Sentinel 和其他 Sentinel 协商客观下线的主节点的状态，如果处于 SDOWN 状态，则投票自动选出新的主节点，将剩余从节点指向新的主节点进行数据复制。

![](./images/redis-13.jpeg)

⑦当没有足够数量的 Sentinel 同意主服务器下线时，主服务器的客观下线状态就会被移除。

当主服务器重新向 Sentinel 的 PING 命令返回有效回复时，主服务器的主观下线状态就会被移除。

**总结**

本文在一次面试的过程中讲述了 Redis 是什么，Redis 的特点和功能，Redis 缓存的使用，Redis 为什么能这么快，Redis 缓存的淘汰策略，持久化的两种方式，Redis 高可用部分的主从复制和哨兵的基本原理。

### Redis命令行批量删除相同前缀的key

redis-cli KEYS "topic*" | xargs redis-cli DEL

此种方式不适用与生产环境执行，容易导致服务不可用，或者服务器宕机风险

推荐使用redis SCAN、SSCAN、ZSCAN、HSCAN迭代的方式删除

一下为php示例代码

**SCAN** 用于迭代

- `SCAN cursor [MATCH pattern] [COUNT count]`
- 作用：迭代当前数据库中的数据库键
- SCAN 使用 demo



```php
<?php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
/* Options for the SCAN family of commands, indicating whether to abstract
   empty results from the user.  If set to SCAN_NORETRY (the default), phpredis
   will just issue one SCAN command at a time, sometimes returning an empty
   array of results.  If set to SCAN_RETRY, phpredis will retry the scan command
   until keys come back OR Redis returns an iterator of zero */
$redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);
$iterator = null;
$count = 1000; // 测试时redis中大概有20w个key，用时约2s，当count为500时用时约4s，count越大时扫描用时越短（当然需要根据你的业务需要来定）
$prefix = date('Ymd');
$time1 = msectime();
$total = [];
while ($arrKeys = $redis->scan($iterator, $prefix . '*', $count)) {
    $arrValues = $redis->mget($arrKeys);
    $ret = array_combine($arrKeys, $arrValues);
    $total = array_merge($total, $ret);
}
$time2 = msectime();
$time = $time2 - $time1;
echo 'time : ' . $time  . ' ms; total keys : ' . count($total) . PHP_EOL;
// time : 2009 ms; total keys : 129798  （用时2009 ms，20w中共有129798个前缀为$prefix的key）
function msectime() {
    list($msec, $sec) = explode(' ', microtime());
    $msectime = (float)sprintf('%.0f', (floatval($msec) + floatval($sec)) * 1000);
    return $msectime;
}
```

**SSCAN**

- `SCAN cursor [MATCH pattern] [COUNT count]`
- 作用：用于迭代集合键中的元素
- SSCAN使用demo



```php
<?php
$iterator = null;
$count = 1000;
$mainKey = date('Ymd');
$prefix = $mainKey;
$total = [];
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);
$time1 = msectime();
while ($arrKeys = $redis->sscan($mainKey, $iterator, $prefix . '*', $count)) { // 匹配前缀为当前日期的key
    $total += $arrKeys;
}
$time2 = msectime();
$time = $time2 - $time1;
echo 'use time : ' . $time  . ' ms; total keys : ' . count($total) . PHP_EOL;
// use time : 649 ms; total keys : 90010 （这个集合中有20w个元素）
....... other code ......
```

**ZSCAN**

- `ZSCAN cursor [MATCH pattern] [COUNT count]`
- 作用：用于迭代有序集合中的元素
- ZSCAN使用demo



```php
<?php
$iterator = null;
$count = 1000;
$mainKey = 'test';
$prefix = '10';
$total = [];
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);
$time1 = msectime();
while ($arrKeys = $redis->zscan($mainKey, $iterator, $prefix . '*', $count)) { // 匹配key前缀是 10 的所有key
    var_dump($arrKeys);
    $total += $arrKeys;
}
$time2 = msectime();
$time = $time2 - $time1;
echo 'use time : ' . $time  . ' ms; total keys : ' . count($total) . PHP_EOL;
// use time : 317 ms; total keys : 1111 （这个集合中有10w个元素）
...... other code ......
```

**HSCAN**

- `HSCAN cursor [MATCH pattern] [COUNT count]`
- 作用：用于迭代哈希中的元素
- HSACN 使用demo



```php
<?php
$iterator = null;
$count = 1000;
$mainKey = 'my_hash_key';
$match = "*key*";
$total = [];
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);
$time1 = msectime();
while ($arrKeys = $redis->hscan($mainKey, $iterator, $match, $count)) { // 匹配含有'key'的键
    var_dump($arrKeys);
    $total += $arrKeys;
}
$time2 = msectime();
$time = $time2 - $time1;
echo 'use time : ' . $time  . ' ms; total keys : ' . count($total) . PHP_EOL;
// use time : 1484 ms; total keys : 100000
...... other code ......
```

redis pipe 代码demo (快速向有序集合添加100000个key， 其他操作类似， 只要修改for中的操作即可)



```php
$pipe = $redis->multi(Redis::PIPELINE);
for ($i = 0; $i < 100000; $i++) {
    $redis->zAdd($key, $i, $i);
}
$curValues = $pipe->exec();
$val = $redis->zRange($key, 0, -1, true);
var_dump($val);
```

传送门

- [redis 指令参考 scan](https://links.jianshu.com/go?to=http%3A%2F%2Fdoc.redisfans.com%2Fkey%2Fscan.html%23scan)
- [Redis删除相同前缀的key](https://www.cnblogs.com/east7/p/11665392.html)
- [PHP redis SCAN、SSCAN、ZSCAN、HSCAN 的使用](https://www.jianshu.com/p/68ec98d2e730)
- [php使用redis的scan命令时遇到的坑](https://blog.csdn.net/yaoxiaofeng_000/article/details/79810413)
- [Redis删除特定前缀key的优雅实现](https://www.cnblogs.com/cheyunhua/p/11037824.html)

