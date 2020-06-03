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