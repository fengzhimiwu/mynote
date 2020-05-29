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