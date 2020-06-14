# mysql知识整理

- 基础知识    
  - 普通查询语句
  - join
  - 子查询
  - union
  - union all
  - [触发器](#触发器)
  - 存储过程
- 数据库优化
  - 应用系统sql优化
    - 分表策略
    - 分库策略
    - 索引
    - 普通索引，联合索引
    - 如何选择字段创建索引
    - 索引失效原因 
    - 查询缓存
  - mysql服务器优化
    - 开启慢查询定位问题
    - mysql连接数
  - 操作系统与硬件优化
  - 系统架构整体优化
    - 负载均衡
    - 缓存
    - 分布式优化
- 数据管理与维护
  - 数据备份
  - mysqldump备份数据库
  - mysldump备份数据表
  - mysqldump备份数据表结构
  - 数据恢复
  - mysql日志
  - mysql监控
    - mysql常用工具



## 触发器
触发器的定义：触发器（TRIGGER）是MySQL的数据库对象之一，从5.0.2版本开始支持。该对象与编程语言中的函数非常类似，都需要声明、执行等。但是触发器的执行不是由程序调用，也不是由手工启动，而是由事件来触发、激活从而实现执行。

#### 触发器的基本语法
```
CREATE TRIGGER trigger_name
trigger_time
trigger_event ON tbl_name
FOR EACH ROW
begin
……
end
```

这里触发器的有两种:before,after
触发的事件类型为：update，insert，delete
#### 创建两张表并插入一些数据演示下触发器的使用
```
CREATE TABLE goods (
        id INT(11) UNSIGNED NOT NULL auto_increment,
        name VARCHAR(40) not null COMMENT '商品名称',
        stock SMALLINT(11) UNSIGNED NOT NULL COMMENT '商品库存',
        PRIMARY KEY(`id`)
        )ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '商品表' ;
```

```
INSERT INTO goods (`name`,`stock`) values ('iphonex',50),('小米2',30),('联想手机',40);
```

```
CREATE TABLE goods_order(
        oid int(11) UNSIGNED NOT NULL auto_increment COMMENT '订单id,自增id',
        gid INT(11) UNSIGNED NOT NULL COMMENT 'goods表的商品id',
        nums SMALLINT(11) UNSIGNED NOT NULL COMMENT '订单购买数量',
        PRIMARY KEY(`oid`)
        ) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '商品订单';
```

#### 创建触发器当有新订单的时候减少goods表的库存数量
```
CREATE TRIGGER t1
AFTER
INSERT 
ON goods_order
FOR EACH ROW
BEGIN
UPDATE goods SET stock = stock-new.nums WHERE id=new.gid;
END
```

#### 运行创建触发器语句后显示触发器
```
MySQL [test]> show triggers\G;
*************************** 1. row ***************************
Trigger: t1
Event: INSERT
Table: goods_order
Statement: BEGIN
UPDATE goods SET stock = stock-new.nums WHERE id=new.gid;
END
Timing: AFTER
Created: NULL
sql_mode: NO_ENGINE_SUBSTITUTION
Definer: root@%
character_set_client: utf8
collation_connection: utf8_general_ci
    Database Collation: utf8_unicode_ci
1 row in set (0.00 sec)
```

#### 查询当前数据库商品表和订单表的数据
```
MySQL [test]> select * from goods;
+----+--------------+-------+
| id | name         | stock |
+----+--------------+-------+
|  1 | iphonex      |    50 |
|  2 | 小米2        |    30 |
|  3 | 联想手机     |    40 |
+----+--------------+-------+
3 rows in set (0.00 sec)

    MySQL [test]> select * from goods_order;
Empty set (0.00 sec)
```

#### 向订单表中插入数据查看商品表库存是否自动减掉了
```
MySQL [test]> insert into goods_order (`gid`,`nums`) values (1,2);
Query OK, 1 row affected (0.00 sec)

MySQL [test]> select * from goods;
+----+--------------+-------+
| id | name         | stock |
+----+--------------+-------+
|  1 | iphonex      |    48 |
|  2 | 小米2        |    30 |
|  3 | 联想手机     |    40 |

MySQL [test]> select * from goods_order;
+-----+-----+------+
| oid | gid | nums |
+-----+-----+------+
|   1 |   1 |    2 |
+-----+-----+------+
```
经过对照发现当添加一条商品，购买数量为2时商品表的库存数由50变为了48说明触发器运行成果，和我们所猜想的一致。

#### 想订单表中插入超过库存总量的数据时测试效果
```
MySQL [test]> insert into goods_order (`gid`,`nums`) values (2,32);
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(`test`.`goods`.`stock` - NEW.nums)'

插入超过库存数量的商品时sql报错，因为我们设置的字段是无符号的，如果当前字段是有符号此时，字段会更新为-2，这肯定是不符合我们要求的，此时需要修改t1触发器来，当超过库存总数量的时候我们减掉最大库存即可。
```
#### 修改t1触发器
```
DROP TRIGGER t1;
CREATE TRIGGER t1
BEFORE
INSERT 
ON goods_order
FOR EACH ROW
BEGIN
DECLARE rnum int;
##查询插入之前商品的库存
SELECT stock into rnum from goods where id=new.gid;
##如果即购买订单的商品数量大于总库存，则设置为购买的数量为当前的商品库存数量
if new.nums>rnum THEN
    SET new.nums = rnum;
END IF;
UPDATE goods SET stock = stock-new.nums WHERE id=new.gid;
END
```

#### 插入订单查看效果
```
MySQL [test]> select * from goods;
+----+--------------+-------+
| id | name         | stock |
+----+--------------+-------+
|  1 | iphonex      |    48 |
|  2 | 小米2        |    30 |
|  3 | 联想手机     |    40 |
+----+--------------+-------+
3 rows in set (0.00 sec)

MySQL [test]> select * from goods_order;
+-----+-----+------+
| oid | gid | nums |
+-----+-----+------+
|   1 |   1 |    2 |
+-----+-----+------+
1 row in set (0.00 sec)


MySQL [test]> insert into goods_order (`gid`,`nums`) values (2,32);
Query OK, 1 row affected (0.00 sec)

MySQL [test]> select * from goods;
+----+--------------+-------+
| id | name         | stock |
+----+--------------+-------+
|  1 | iphonex      |    48 |
|  2 | 小米2        |     0 |
|  3 | 联想手机     |    40 |
+----+--------------+-------+
3 rows in set (0.00 sec)

MySQL [test]> select * from goods_order;
+-----+-----+------+
| oid | gid | nums |
+-----+-----+------+
|   1 |   1 |    2 |
|   3 |   2 |   30 |
+-----+-----+------+
2 rows in set (0.00 sec

```
从上门可以看出，当我们订单中购买数量大于商品库存总数的时候，商品库存只会扣除最大库存数

#### 创建触发t2，实现当删除订单表时恢复商品库存数量
```
CREATE TRIGGER t2
AFTER
DELETE
ON goods_order
FOR EACH ROW
BEGIN
UPDATE goods SET stock=stock+old.nums where id=old.gid; 
END
```

查询当前所有触发器
```
MySQL [test]> show triggers\G;
*************************** 1. row ***************************
             Trigger: t1
               Event: INSERT
               Table: goods_order
           Statement: BEGIN
        DECLARE rnum int;
 ##查询插入之前商品的库存
        SELECT stock into rnum from goods where id=new.gid;
 ##如果即购买订单的商品数量大于总库存，则设置为购买的数量为当前的商品库存数量
        if new.nums>rnum THEN
                SET new.nums = rnum;
        END IF;
        UPDATE goods SET stock = stock-new.nums WHERE id=new.gid;
END
              Timing: BEFORE
             Created: NULL
            sql_mode: NO_ENGINE_SUBSTITUTION
             Definer: root@%
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8_unicode_ci
*************************** 2. row ***************************
             Trigger: t2
               Event: DELETE
               Table: goods_order
           Statement: BEGIN
UPDATE goods SET stock=stock+old.nums where id=old.gid; 
END
              Timing: AFTER
             Created: NULL
            sql_mode: NO_ENGINE_SUBSTITUTION
             Definer: root@%
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8_unicode_ci
2 rows in set (0.00 sec)
```

删除订单表id=1的数据
```
MySQL [test]> delete from goods_order where id=1;
ERROR 1054 (42S22): Unknown column 'id' in 'where clause'
MySQL [test]> delete from goods_order where oid=1;
Query OK, 1 row affected (0.00 sec)

MySQL [test]> select * from goods;
+----+--------------+-------+
| id | name         | stock |
+----+--------------+-------+
|  1 | iphonex      |    50 |
|  2 | 小米2        |     0 |
|  3 | 联想手机     |    40 |
+----+--------------+-------+
3 rows in set (0.00 sec)

MySQL [test]> select * from goods_order;
+-----+-----+------+
| oid | gid | nums |
+-----+-----+------+
|   3 |   2 |   30 |
+-----+-----+------+
1 row in set (0.00 sec)
```
由上门可以看出，删除id=1的数据后，订单表id=1的商品库存数由之前48增加到了50说明触发器成功执行

#### 最后解释下FOR EACH ROW
for each row表示的是执行的触发起的动作影响了多少行数据就执行多少行的数据

- 
- - 什么是索引，作用是什么？常见索引类型有那些？Mysql 建立索引的原则？

> 索引是一种特殊的文件,它们包含着对数据表里所有记录的引用指针，相当于书本的目录。其作用就是加快数据的检索效率。常见索引类型有主键、唯一索引、复合索引、全文索引。
> 
> - 索引创建的原则
>   - 最左前缀原理
>   - 选择区分度高的列作为索引
>   - 尽量的扩展索引，不要新建索引

- SQL 语句的优化原则？

> 1. 避免使用 Like 模糊查询
> 2. 只列出需要查询的字段，而不是所有
> 3. 避免使用 MySQL 函数，尽量让 MySQL 做更少的事情，减轻 MySQL 的压力
> 4. 经常查询的字段，创建合适的索引，提高查询效率

- 什么是 MySQL 慢查询？又该如何优化？

> MySQL 中查询超过指定时间的语句，被称之为「慢查询」。该如何优化呢？优化 SQL 语句，创建合适的索引，如以上两个问题。

- MySQL 分库分表怎么设计

> #### 1. 垂直分表
> 
> 垂直分表在日常开发和设计中比较常见，通俗的说法叫做“大表拆小表”，某个表中的字段比较多，可以新建立一张“扩展表”，将不经常使用或者长度较大的字段，拆分出去放到“扩展表”中。
> 
> #### 2. 垂直分库
> 
> 基本的思路就是按照业务模块来划分出不同的数据库，而不是像早期一样将所有的数据表都放到同一个数据库中。  
> 
> #### 3. 水平分表
> 
> 水平分表也称为横向分表，比较容易理解，就是将表中不同的数据行按照一定规律分布到不同的数据库表中（这些表保存在同一个数据库中），这样来降低单表数据量，优化查询性能。 
> 
> #### 4. 水平分库分表
> 
> 水平分库分表与上面讲到的水平分表的思想相同，唯一不同的就是将这些拆分出来的表保存在不同的数据库中。

- 什么是 MySQL 死锁？如何有效降低死锁？

> 死锁：死锁一般是事务相互等待对方资源，最后形成环路，而无法继续运行。
> 
> 产生死锁的原因：
> 
> 1. 系统资源不足；
> 2. 进程运行推进的顺序不合适；
> 3. 资源分配不当等；
> 
> 如何有效降低死锁：
> 
> 1. 按同一顺序访问资源；
> 2. 避免事务中的用户交互；
> 3. 保持事务简短并在一个批处理中；
> 4. 使用低隔离级别；
> 5. 使用绑定连接；

### 9. SQL注入的原理是什么？如何防止SQL注入
```
通常都是低级程序员写的低级代码,未过滤用户输入导致的,现代框架的ORM一般都做过相应处理,如果需要自己处理,有两种解决方式:
1:转义用户输入(htmlentities/htmlspecialchars),用mysql_real_escape_string方法过滤SQL语句的参数
2:预编译sql    (最佳方式)
```
### 10. mysql 主键和唯一索引的区别
```
主键是一种约束，唯一索引是一种索引，两者在本质上是不同的
主键创建后一定包含一个唯一性索引，唯一性索引并不一定就是主键
唯一性索引列允许空值，而主键列不允许为空值
主键列在创建时，已经默认为空值 + 唯一索引了
主键可以被其他表引用为外键，而唯一索引不能
一个表最多只能创建一个主键，但可以创建多个唯一索引

https://blog.csdn.net/baoqiangwang/article/details/4832814
```
### 11. mysql 数据库中的事务是什么？
```
事务的特征：ACID
原子性Atomicity 一组DML语句要么全部成功要么全部失败
一致性Consistency 事务必须由一个状态到另一个状态
隔离性Isolation 多个事务之间能够根据事务的隔离级别表现不同
持久性Durability 提交后的事务，一旦提交，它对数据库中的数据修改是永久性的

Q:当没有事务的情况会出现什么问题?
A:当在控制台，操作mysql数据库时候，如果没有事务控制，误操作就会造成数据的永久损失。
事务的隔离级别:
```
事务的隔离级别:

| 隔离级别                   | 脏读 | 不可重复读 |               幻读 |        加锁读 |
| -------------------------- | :--: | ---------: | -----------------: | ------------: |
| 读未提交(Read uncommitted) |  o   |          o |                  o |        不加锁 |
| 读已提交(Read committed)   |  x   |          o |                  o |        不加锁 |
| 可重复读(Repeatable read)  |  x   |          x | o(mysql不会出现 x) |        不加锁 |
| 可串行读(Serializable)     |  x   |          x |                  x | 加锁 (全表锁) |


```
脏读：当某个客户端查询出了另外一个事务还没有提交的修改数据，即为脏读。
不可重复读：[同一查询]在[同一事务]中多次进行，由其它提交事务所做的修改或删除，每次返回不同的的结果集，此时发生非重复读。
幻读：[同一查询]在[同一事务]中多次进行，由于其它提交事务(事务可能没提交)所做的插入操作，每次返回不同的结果集，此时发生幻读。
```

### 扩展阅读

- [MySQL索引原理及慢查询优化](https://tech.meituan.com/mysql-index.html)
- [分库分表的几种常见形式](http://www.infoq.com/cn/articles/key-steps-and-likely-problems-of-split-table)
- [大众点评订单系统分库分表实践](https://tech.meituan.com/dianping_order_db_sharding.html)
- [MySQL 死锁问题及解决](http://onwise.xyz/2017/04/20/mysql-%E6%AD%BB%E9%94%81%E9%97%AE%E9%A2%98%E5%8F%8A%E8%A7%A3%E5%86%B3/)
- [MySQL索引背后的数据结构及算法原理](https://www.kancloud.cn/kancloud/theory-of-mysql-index/41846)





1>.InnoDB支持事物，而MyISAM不支持事物

2>.InnoDB支持行级锁，而MyISAM支持表级锁

3>.InnoDB支持MVCC, 而MyISAM不支持

4>.InnoDB支持外键，而MyISAM不支持

5>.InnoDB不支持全文索引，而MyISAM支持

### 存放索引的方式

- InnoDB是聚集索引，
  数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。
  因此，主键不应该过大，因为主键太大，其他索引也都会很大。
- MyISAM是非聚集索引，数据文件是分离的，
  索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

### 存储引擎管理

- 查看数据库支持的存储引擎
  show engines
- 查看数据库当前使用的存储引擎,就是默认引擎是什么。
  show variables like '%storage_engine%'
  也可以在MySQL配置文件中查看。
  windows - my.ini。
  Linux - my.cnf
- 查看数据库表所用的存储引擎
  show create table table_name
- 创建表指定存储引擎
  create table table_name (column_name column_type) engine = engine_name
- 修改表的存储引擎
  alter table table_name engine=engine_name
- 修改默认的存储引擎
  在MySQL配置文件中修改下述内容：
  default-storage-engine=INNODB
  MySQL配置文件：
  windows系统 - MySQL安装目录/my.ini （5.7版本my.ini文件在数据目录中。 C:/programdata/MySQL Server 5.7/mysql/）
  linux系统 - /etc/my.cnf

 

\1. InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务； 

\2. InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败； 

\3. InnoDB是聚集索引，使用B+Tree作为索引结构，数据文件是和（主键）索引绑在一起的（表数据文件本身就是按B+Tree组织的一个索引结构），必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。

​    MyISAM是非聚集索引，也是使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

​    也就是说：InnoDB的B+树主键索引的叶子节点就是数据文件，辅助索引的叶子节点是主键的值；而MyISAM的B+树主键索引和辅助索引的叶子节点都是数据文件的地址指针。

\4. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快（注意不能加有任何WHERE条件）；

那么为什么InnoDB没有了这个变量呢？

  因为InnoDB的事务特性，在同一时刻表中的行数对于不同的事务而言是不一样的，因此count统计会计算对于当前事务而言可以统计到的行数，而不是将总行数储存起来方便快速查询。InnoDB会尝试遍历一个尽可能小的索引除非优化器提示使用别的索引。如果二级索引不存在，InnoDB还会尝试去遍历其他聚簇索引。
  如果索引并没有完全处于InnoDB维护的缓冲区（Buffer Pool）中，count操作会比较费时。可以建立一个记录总行数的表并让你的程序在INSERT/DELETE时更新对应的数据。和上面提到的问题一样，如果此时存在多个事务的话这种方案也不太好用。如果得到大致的行数值已经足够满足需求可以尝试SHOW TABLE STATUS


\5. Innodb不支持全文索引，而MyISAM支持全文索引，在涉及全文索引领域的查询效率上MyISAM速度更快高；PS：5.7以后的InnoDB支持全文索引了

\6. MyISAM表格可以被压缩后进行查询操作

\7. InnoDB支持表、行(默认)级锁，而MyISAM支持表级锁

​    InnoDB的行锁是实现在索引上的，而不是锁在物理行记录上。潜台词是，如果访问没有命中索引，也无法使用行锁，将要退化为表锁。
例如：

  t_user(uid, uname, age, sex) innodb;

  uid PK
  无其他索引
  update t_user set age=10 where uid=1;       命中索引，行锁。

  update t_user set age=10 where uid != 1;      未命中索引，表锁。

  update t_user set age=10 where name='chackca';  无索引，表锁。

8、InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有

9、Innodb存储文件有frm、ibd，而Myisam是frm、MYD、MYI

​    Innodb：frm是表定义文件，ibd是数据文件

​    Myisam：frm是表定义文件，myd是数据文件，myi是索引文件

 

如何选择：
  \1. 是否要支持事务，如果要请选择innodb，如果不需要可以考虑MyISAM；

  \2. 如果表中绝大多数都只是读查询，可以考虑MyISAM，如果既有读也有写，请使用InnoDB。

  \3. 系统奔溃后，MyISAM恢复起来更困难，能否接受；

  \4. MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用InnoDB，至少不会差。

 

InnoDB为什么推荐使用自增ID作为主键？

  答：自增ID可以保证每次插入时B+索引是从右边扩展的，可以避免B+树和频繁合并和分裂（对比使用UUID）。如果使用字符串主键和随机主键，会使得数据随机插入，效率比较差。

 

innodb引擎的4大特性

​    插入缓冲（insert buffer),二次写(double write),自适应哈希索引(ahi),预读(read ahead)



innodb引擎的4大特性

**插入缓冲（insert buffer)**

提升插入性能，change buffering是insert buffer的加强，insert buffer只针对insert有效，change buffering对insert、delete、update(delete+insert)、purge都有效

只对于非聚集索引（非唯一）的插入和更新有效，对于每一次的插入不是写到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，如果在则直接插入；若不在，则先放到Insert Buffer 中，再按照一定的频率进行合并操作，再写回disk。这样通常能将多个插入合并到一个操作中，目的还是为了减少随机IO带来性能损耗。

使用插入缓冲的条件：
\* 非聚集索引
\* 非唯一索引

上面提过在一定频率下进行合并，那所谓的频率是什么条件？

1）辅助索引页被读取到缓冲池中。正常的select先检查Insert Buffer是否有该非聚集索引页存在，若有则合并插入。

2）辅助索引页没有可用空间。空间小于1/32页的大小，则会强制合并操作。

3）Master Thread 每秒和每10秒的合并操作。

**二次写(double write)**

Doublewrite缓存是位于系统表空间的存储区域，用来缓存InnoDB的数据页从innodb buffer pool中flush之后并写入到数据文件之前，所以当操作系统或者数据库进程在数据页写磁盘的过程中崩溃，Innodb可以在doublewrite缓存中找到数据页的备份而用来执行crash恢复。数据页写入到doublewrite缓存的动作所需要的IO消耗要小于写入到数据文件的消耗，因为此写入操作会以一次大的连续块的方式写入

在应用（apply）重做日志前，用户需要一个页的副本，当写入失效发生时，先通过页的副本来还原该页，再进行重做，这就是double write

doublewrite组成：
内存中的doublewrite buffer,大小2M。
物理磁盘上共享表空间中连续的128个页，即2个区（extend），大小同样为2M。
对缓冲池的脏页进行刷新时，不是直接写磁盘，而是会通过memcpy()函数将脏页先复制到内存中的doublewrite buffer，之后通过doublewrite 再分两次，每次1M顺序地写入共享表空间的物理磁盘上，在这个过程中，因为doublewrite页是连续的，因此这个过程是顺序写的，开销并不是很大。在完成doublewrite页的写入后，再将doublewrite buffer 中的页写入各个 表空间文件中，此时的写入则是离散的。如果操作系统在将页写入磁盘的过程中发生了崩溃，在恢复过程中，innodb可以从共享表空间中的doublewrite中找到该页的一个副本，将其复制到表空间文件，再应用重做日志。

**自适应哈希索引(ahi)**

Adaptive Hash index属性使得InnoDB更像是内存数据库。

Innodb存储引擎会监控对表上二级索引的查找，如果发现某二级索引被频繁访问，二级索引成为热数据，建立哈希索引可以带来速度的提升

经常访问的二级索引数据会自动被生成到hash索引里面去(最近连续被访问三次的数据)，自适应哈希索引通过缓冲池的B+树构造而来，因此建立的速度很快。
哈希（hash）是一种非常快的等值查找方法，在一般情况下这种查找的时间复杂度为O(1),即一般仅需要一次查找就能定位数据。而B+树的查找次数，取决于B+树的高度，在生产环境中，B+树的高度一般3-4层，故需要3-4次的查询。

innodb会监控对表上索引页的查询。如果观察到建立哈希索引可以带来速度提升，则自动建立哈希索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）。
AHI有一个要求，就是对这个页的连续访问模式必须是一样的。
例如对于（a,b）访问模式情况：
where a = xxx
where a = xxx and b = xxx

特点
　　1、无序，没有树高
　　2、降低对二级索引树的频繁访问资源，索引树高<=4，访问索引：访问树、根节点、叶子节点
　　3、自适应
3、缺陷
　　1、hash自适应索引会占用innodb buffer pool；
　　2、自适应hash索引只适合搜索等值的查询，如select * from table where index_col='xxx'，而对于其他查找类型，如范围查找，是不能使用的；
　　3、极端情况下，自适应hash索引才有比较大的意义，可以降低逻辑读。

 

**预读(read ahead)**

介绍预读机制首先要知道InnoDB引擎,他是现在版本MySQL的主要引擎，它将所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引，缓冲池用于高速缓冲数据和索引。

InnoDB在I/O的优化上相对之前有着独特的预读机制，预读机制就是发起一个i/o请求，异步地在缓冲池中预先回迁若干页面，预计将会用到回迁的页面，这些请求在一个范围内引入所有页面。InnoDB以64个page为一个extent，而page和extent则是接下来要介绍的两种预读算法的基本单位。

数据库发起数据请求，请求被提交到文件读取系统，放入相关请求队列中；读取进程从请求队列把请求取出，根据读取请求到相关的数据存储介质读取数据；返回数据，放入响应队列，最后数据库从响应队列中将数据取走，完成数据读取。

相关流程

接着进程继续处理请求队列，(如果数据库是全表扫描的话，数据读请求将会占满请求队列)，判断后面几个数据读请求的数据是否相邻，再根据自身系统IO带宽处理量，进行预读，进行读请求的合并处理，一次性读取多块数据放入响应队列中，再被数据库取走。(如此，一次物理读操作，实现多页数据读取，rrqm>0（# iostat -x），假设是4个读请求合并，则rrqm参数显示的就是4)

InnoDB使用两种预读算法来提高I/O性能：线性预读（linear read-ahead）和随机预读（randomread-ahead）

　　为了区分这两种预读的方式，我们可以把线性预读放到以extent为单位，而随机预读放到以extent中的page为单位。线性预读着眼于将下一个extent提前读取到buffer pool中，而随机预读着眼于将当前extent中的剩余的page提前读取到buffer pool中。

## 线性预读

线性预读方式有一个很重要的变量控制是否将下一个extent预读到buffer pool中，通过使用配置参数innodb_read_ahead_threshold，控制触发innodb执行预读操作的时间。

　　如果一个extent中的被顺序读取的page超过或者等于该参数变量时，Innodb将会异步的将下一个extent读取到buffer pool中，innodb_read_ahead_threshold可以设置为0-64的任何值(因为一个extent中也就只有64页)，默认值为56，值越高，访问模式检查越严格。

例如，如果将值设置为48，则InnoDB只有在顺序访问当前extent中的48个pages时才触发线性预读请求，将下一个extent读到内存中。如果值为8，InnoDB触发异步预读，即使程序段中只有8页被顺序访问。

　　可以在MySQL配置文件中设置此参数的值，或者使用SET GLOBAL需要该SUPER权限的命令动态更改该参数。

　　在没有该变量之前，当访问到extent的最后一个page的时候，innodb会决定是否将下一个extent放入到buffer pool中。

## 随机预读

随机预读方式则是表示当同一个extent中的一些page在buffer pool中发现时，Innodb会将该extent中的剩余page一并读到buffer pool中。

由于随机预读方式给innodb code带来了一些不必要的复杂性，同时在性能也存在不稳定性，在5.5版本以及之后中已经将这种预读方式废弃，默认是OFF。若要启用此功能，即将配置变量设置innodb_random_read_ahead为ON。





MySQL日志系统：redo log、binlog、undo log 区别与作用

日志系统主要有redo log(重做日志)和binlog(归档日志)。

redo log是InnoDB存储引擎层的日志，binlog是MySQL Server层记录的日志， 两者都是记录了某些操作的日志(不是所有)自然有些重复（但两者记录的格式不同）。

图来自极客时间的mysql实践,该图是描述的是MySQL的逻辑架构。

### redo log日志模块

redo log是InnoDB存储引擎层的日志，又称重做日志文件，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来。在实例和介质失败（media failure）时，redo log文件就能派上用场，如数据库掉电，InnoDB存储引擎会使用redo log恢复到掉电前的时刻，以此来保证数据的完整性。

在一条更新语句进行执行的时候，InnoDB引擎会把更新记录写到redo log日志中，然后更新内存，此时算是语句执行完了，然后在空闲的时候或者是按照设定的更新策略将redo log中的内容更新到磁盘中，这里涉及到`WAL`即`Write Ahead logging`技术，他的关键点是先写日志，再写磁盘。

有了redo log日志，那么在数据库进行异常重启的时候，可以根据redo log日志进行恢复，也就达到了`crash-safe`。

redo log日志的大小是固定的，即记录满了以后就从头循环写。

图片来自极客时间，该图展示了一组4个文件的redo log日志，checkpoint之前表示擦除完了的，即可以进行写的，擦除之前会更新到磁盘中，write pos是指写的位置，当write pos和checkpoint相遇的时候表明redo log已经满了，这个时候数据库停止进行数据库更新语句的执行，转而进行redo log日志同步到磁盘中。

### binlog日志模块

binlog是属于MySQL Server层面的，又称为归档日志，属于逻辑日志，是以二进制的形式记录的是这个语句的原始逻辑，依靠binlog是没有`crash-safe`能力的

### redo log和binlog区别

- redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。
- redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
- redo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
- binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。

### 一条更新语句执行的顺序

update T set c=c+1 where ID=2;

- 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
- 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
- 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
- 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
- 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

这个update语句的执行流程图，图中浅色框表示是在 InnoDB 内部执行的，深色框表示是在执行器中执行的。

innodb事务日志包括redo log和undo log。redo log是重做日志，提供前滚操作，undo log是回滚日志，提供回滚操作。

undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
1.redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。
2.undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。

 

# 一、重做日志（redo log）

作用：

确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

# 二、回滚日志（undo log）

作用：

保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

# 三、二进制日志（binlog）：

作用：

用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。 
用于数据库的基于时间点的还原。

 

数据库数据存放的文件称为data file；日志文件称为log file；数据库数据是有缓存的，如果没有缓存，每次都写或者读物理disk，那性能就太低下了。数据库数据的缓存称为data buffer，日志（redo）缓存称为log buffer；既然数据库数据有缓存，就很难保证缓存数据（脏数据）与磁盘数据的一致性。比如某次数据库操作：

update driver_info set driver_status = 2 where driver_id = 10001;

更新driver_status字段的数据会存放在缓存中，等待存储引擎将driver_status刷新data_file，并返回给业务方更新成功。如果此时数据库宕机，缓存中的数据就丢失了，业务方却以为更新成功了，数据不一致，也没有持久化存储。

上面的问题就可以通过事务的ACID特性来保证。

BEGIN trans；

update driver_info set driver_status = 2 where driver_id = 10001;

COMMIT;

这样执行后，更新要么成功，要么失败。业务方的返回和数据库data file中的数据保持一致。要保证这样的特性这就不得不说存储引擎innodb的redo和undo日志。

redo日志、undo日志：

存储引擎也会为redo undo日志开辟内存缓存空间，log buffer。磁盘上的日志文件称为log file，是顺序追加的，性能非常高，注：磁盘的顺序写性能比内存的写性能差不了多少。

undo日志用于记录事务开始前的状态，用于事务失败时的回滚操作；redo日志记录事务执行后的状态，用来恢复未写入data file的已成功事务更新的数据。例如某一事务的事务序号为T1，其对数据X进行修改，设X的原值是5，修改后的值为15，那么Undo日志为<T1, X, 5>，Redo日志为<T1, X, 15>。

梳理下事务执行的各个阶段：

（1）写undo日志到log buffer；

（2）执行事务，并写redo日志到log buffer；

（3）如果innodb_flush_log_at_trx_commit=1，则将redo日志写到log file，并刷新落盘。

（4）提交事务。

可能有同学会问，为什么没有写data file，事务就提交了？

在数据库的世界里，数据从来都不重要，日志才是最重要的，有了日志就有了一切。

因为data buffer中的数据会在合适的时间 由存储引擎写入到data file，如果在写入之前，数据库宕机了，根据落盘的redo日志，完全可以将事务更改的数据恢复。好了，看出日志的重要性了吧。先持久化日志的策略叫做Write Ahead Log，即预写日志。

分析几种异常情况：

- innodb_flush_log_at_trx_commit=2（[innodb_flush_log_at_trx_commit和sync_binlog参数详解](http://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NjQwNTE5Ng%3D%3D%26mid%3D2247483681%26idx%3D1%26sn%3D03adfb89521568013f6a1efd9ca1af6a%26scene%3D21%23wechat_redirect)）时，将redo日志写入logfile后，为提升事务执行的性能，存储引擎并没有调用文件系统的sync操作，将日志落盘。如果此时宕机了，那么未落盘redo日志事务的数据是无法保证一致性的。
- undo日志同样存在未落盘的情况，可能出现无法回滚的情况。

checkpoint：

checkpoint是为了定期将db buffer的内容刷新到data file。当遇到内存不足、db buffer已满等情况时，需要将db buffer中的内容/部分内容（特别是脏数据）转储到data file中。在转储时，会记录checkpoint发生的”时刻“。在故障回复时候，只需要redo/undo最近的一次checkpoint之后的操作。

 

 

**一、redo log** 

　　 重做日志

　　作用：确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

　　内容：物理格式的日志，记录的是物理数据页面的修改的信息，其redo log是顺序写入redo log file的物理文件中去的。

**二、bin log**

　　归档日志（二进制日志）

　　作用：用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。
用于数据库的基于时间点的还原。

　　内容：逻辑格式的日志，可以简单认为就是执行过的事务中的sql语句。

但又不完全是sql语句这么简单，而是包括了执行的sql语句（增删改）反向的信息，也就意味着delete对应着delete本身和其反向的insert；update对应着update执行前后的版本的信息；insert对应着delete和insert本身的信息。

　　binlog 有三种模式：Statement（基于 SQL 语句的复制）、Row（基于行的复制） 以及 Mixed（混合模式）

**三、undo log**

　　回滚日志

　　作用：保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

　　内容：逻辑格式的日志，在执行undo的时候，仅仅是将数据从逻辑上恢复至事务之前的状态，而不是从物理页面上操作实现的，这一点是不同于redo log的。

------

**redo log**

　　mysql，如果每次更新操作都要写进磁盘，然后磁盘要找到对应记录，然后再更细，整个过程io成本、查找成本都很高。

　　解决方案：WAL技术（Write-Ahead Logging）。先写日志，再写磁盘。

　　具体来说，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做。InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写，如下面这个图所示。

write pos 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

　　write pos 和 checkpoint 之间的是log上还空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，表示log满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint 推进一下。

　　有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 crash-safe。

 

**bin log**

　　MySQL 整体来看，其实就有两块：一块是 Server 层，它主要做的是 MySQL 功能层面的事情；还有一块是引擎层，负责存储相关的具体事宜。redo log 是 InnoDB 引擎特有的日志，而 Server 层也有自己的日志，称为 binlog（归档日志）。

　　这两种日志有以下三点不同。

　　1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

　　2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

　　3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

update 语句时的内部流程：

　　1. 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。

　　2. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。

　　3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。

　　4. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。

　　5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

最后三步，将 redo log 的写入拆成了两个步骤：prepare 和 commit，这就是"两阶段提交"。

**两阶段提交**

　　两阶段提交，是为了binlog和redolog两分日志之间的逻辑一致。redo log 和 binlog 都可以用于表示事务的提交状态，而两阶段提交就是让这两个状态保持逻辑上的一致。

　　由于 redo log 和 binlog 是两个独立的逻辑，如果不用两阶段提交，要么就是先写完 redo log 再写 binlog，或者采用反过来的顺序。可能造成的问题：

 　update 语句来做例子。假设当前 ID=2 的行，字段 c 的值是 0，再假设执行 update 语句过程中在写完第一个日志后，第二个日志还没有写完期间发生了 crash，会出现什么情况呢？

　　1. 先写 redo log 后写 binlog。

　　假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。由于，redo log 写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行 c 的值是 1。但是由于 binlog 没写完就 crash 了，这时候 binlog 里面就没有记录这个语句。因此，之后备份日志的时候，存起来的 binlog 里面就没有这条语句。然后你会发现，如果需要用这个 binlog 来恢复临时库的话，由于这个语句的 binlog 丢失，这个临时库就会少了这一次更新，恢复出来的这一行 c 的值就是 0，与原库的值不同。

　　2. 先写 binlog 后写 redo log。

　　如果在 binlog 写完之后 crash，由于 redo log 还没写，崩溃恢复以后这个事务无效，所以这一行 c 的值是 0。但是 binlog 里面已经记录了“把 c 从 0 改成 1”这个日志。所以，在之后用 binlog 来恢复的时候就多了一个事务出来，恢复出来的这一行 c 的值就是 1，与原库的值不同。

　　如果不使用“两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。

　　扩容的时候，也就是需要再多搭建一些备库来增加系统的读能力的时候，现在常见的做法也是用全量备份加上应用 binlog 来实现的，这个“不一致”就会导致线上出现主从数据库不一致的情况。

 

**undo log**

　　undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。

　　undo日志用于记录事务开始前的状态，用于事务失败时的回滚操作；redo日志记录事务执行后的状态，用来恢复未写入data file的已成功事务更新的数据。例如某一事务的事务序号为T1，其对数据X进行修改，设X的原值是0，修改后的值为1，那么Undo日志为<T1, X, 0>，Redo日志为<T1, X, 1>。