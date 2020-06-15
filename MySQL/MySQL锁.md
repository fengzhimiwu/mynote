### 一、死锁示例

```sql
CREATE TABLE `test` (
  `id` int(20) NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

mysql> SELECT * FROM test;
+----+------+
| id | name |
+----+------+
|  1 | 1    |
|  5 | 5    |
| 10 | 10   |
| 15 | 15   |
| 20 | 20   |
| 25 | 25   |
+----+------+
6 rows in set (0.00 sec)

当数据库的隔离级别为Repeatable Read或Serializable时,我们来看这样的两个并发事务（**场景一**）：

| session1                                        | session2                                        |
| ----------------------------------------------- | ----------------------------------------------- |
| begin;                                          |                                                 |
|                                                 | begin;                                          |
| select * from test where id = 12 for update;    |                                                 |
|                                                 | select * from test where id = 13 for update;    |
| insert into test(id, name) values(12, "test1"); |                                                 |
| 锁等待中                                        | insert into test(id, name) values(13, "test2"); |
| 锁等待解除                                      | 死锁，session 2的事务被回滚                     |

上面两个并发事务一定会发生死锁（这里之所以限定RR和Serializable两个隔离级别，是因为只有这两个级别下才会有间隙锁/临键锁，而这是导致死锁的**根本原因**，后面会详细分析）。

我们再来看另外一个并发场景（**场景二**）：

| session1                                        | session2                                        |
| ----------------------------------------------- | ----------------------------------------------- |
| begin;                                          |                                                 |
|                                                 | begin;                                          |
| select * from test where id = 12 for update;    |                                                 |
|                                                 | select * from test where id = 16 for update;    |
| insert into test(id, name) values(12, "test1"); |                                                 |
| commit;                                         | insert into test(id, name) values(16, "test2"); |
|                                                 | commit;                                         |

在这个并发场景下，两个事务均能成功提交，而不会有死锁。

在上面的示例中，我们发现，select ... for update虽然可以用于解决数据库的并发操作，但在实际项目中却不建议使用，原因是当查询条件对应的记录不存在时，很容易造成死锁。而造成死锁的原因和MySQL的锁机制有关。本文将详细介绍常见的七种锁机制，了解了这些锁机制之后就能理解造成场景一死锁的根本原因以及场景一和场景二差异的原因。

### 二、MySQL的八种锁

> 行锁（Record Locks）
> 间隙锁（Gap Locks）
> 临键锁（Next-key Locks）
> 共享锁/排他锁（Shared and Exclusive Locks）
> 意向共享锁/意向排他锁（Intention Shared and Exclusive Locks）
> 插入意向锁（Insert Intention Locks）
> 自增锁（Auto-inc Locks）
> 空间索引（Predicate Locks for Spatial Indexes）

#### 行锁

这MySQL的官方文档中有以下描述：

A record lock is a lock on an **index record**. **Record locks always lock index records, even if a table is defined with no indexes.** For such cases, InnoDB creates a hidden clustered index and uses this index for record locking.

这句话说明行锁一定是作用在索引上的。

#### 间隙锁

在MySQL的官方文档中有以下描述：

A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record。

这句话表明间隙锁一定是开区间，比如（3，5）或者。在MySQL官网上还有一段非常关键的描述：

Gap locks in InnoDB are “purely inhibitive”, which means that their only purpose is to prevent other transactions from inserting to the gap. Gap locks can co-exist. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap. There is no difference between shared and exclusive gap locks. They do not conflict with each other, and they perform the same function.

这段话表明间隙锁在本质上是不区分共享间隙锁或互斥间隙锁的，而且间隙锁是不互斥的，即两个事务可以同时持有包含共同间隙的间隙锁。这里的共同间隙包括两种场景：其一是两个间隙锁的间隙区间完全一样；其二是一个间隙锁包含的间隙区间是另一个间隙锁包含间隙区间的子集。间隙锁本质上是用于阻止其他事务在该间隙内插入新记录，而自身事务是允许在该间隙内插入数据的。也就是说间隙锁的应用场景包括并发读取、并发更新、并发删除和并发插入。

在MySQL官网上关于间隙锁还有一段重要描述：

Gap locking can be disabled explicitly. This occurs if you change the transaction isolation level to READ COMMITTED. Under these circumstances, gap locking is disabled for searches and index scans and is used only for foreign-key constraint checking and duplicate-key checking.

这段话表明，在RU和RC两种隔离级别下，即使你使用select ... in share mode或select ... for update，也无法防止幻读（读后写的场景）。因为这两种隔离级别下只会有行锁，而不会有间隙锁。这也是为什么示例中要规定隔离级别为RR的原因。

#### 临键锁

在MySQL的官方文档中有以下描述：

A next-key lock is a combination of a record lock on the index record and a gap lock on the gap before the index record.

这句话表明临键锁是行锁+间隙锁，即临键锁是是一个左开右闭的区间，比如（3，5]。

在MySQL的官方文档中还有以下重要描述：

By default, InnoDB operates in REPEATABLE READ transaction isolation level. In this case, InnoDB uses next-key locks for searches and index scans, which prevents phantom rows.

个人觉得这段话描述得不够好，很容易引起误解。这里更正如下：InnoDB的默认事务隔离级别是RR，在这种级别下，如果你使用select ... in share mode或者select ... for update语句，那么InnoDB会使用临键锁，因而可以防止幻读；但即使你的隔离级别是RR，如果你这是使用普通的select语句，那么InnoDB将是快照读，不会使用任何锁，因而还是无法防止幻读。

#### 共享锁/排他锁

在MySQL的官方文档中有以下描述：

InnoDB implements standard row-level locking where there are two types of locks, shared (S) locks and exclusive (X) locks。

A shared (S) lock permits the transaction that holds the lock to read a row.

An exclusive (X) lock permits the transaction that holds the lock to update or delete a row.

这段话明确说名了共享锁/排他锁都只是行锁，与间隙锁无关，这一点很重要，后面还会强调这一点。其中共享锁是一个事务并发读取某一行记录所需要持有的锁，比如select ... in share mode；排他锁是一个事务并发更新或删除某一行记录所需要持有的锁，比如select ... for update。

不过这里需要重点说明的是，尽管共享锁/排他锁是行锁，与间隙锁无关，但一个事务在请求共享锁/排他锁时，获取到的结果却可能是行锁，也可能是间隙锁，也可能是临键锁，这取决于数据库的隔离级别以及查询的数据是否存在。关于这一点，后面分析场景一和场景二的时候还会提到。

#### 意向共享锁/意向排他锁

在MySQL的官方文档中有以下描述：

Intention locks are table-level locks that indicate which type of lock (shared or exclusive) a transaction requires later for a row in a table。

The intention locking protocol is as follows:

Before a transaction can acquire a shared lock on a row in a table, it must first acquire an IS lock or stronger on the table.

Before a transaction can acquire an exclusive lock on a row in a table, it must first acquire an IX lock on the table.

 

这段话说明意向共享锁/意向排他锁属于表锁，且**取得意向共享锁/意向排他锁是取得共享锁/排他锁的前置条件**。

共享锁/排他锁与意向共享锁/意向排他锁的兼容性关系：

|      | X    | IX       | S    | IS   |
| ---- | ---- | -------- | ---- | ---- |
| X    | 互斥 | 互斥     | 互斥 | 互斥 |
| IX   | 互斥 | **兼容** | 互斥 |      |
| S    | 互斥 | 互斥     | 兼容 | 兼容 |
| IS   | 互斥 | 兼容     | 兼容 | 兼容 |

这里需要重点关注的是**IX锁和IX锁是相互兼容**的，这是导致上面场景一发生死锁的前置条件，后面会对死锁原因进行详细分析。

#### 插入意向锁

在MySQL的官方文档中有以下重要描述：

An insert intention lock is a type of gap lock set by INSERT operations prior to row insertion. This lock signals the intent to insert in such a way that multiple transactions inserting into the same index gap need not wait for each other if they are not inserting at the same position within the gap. Suppose that there are index records with values of 4 and 7. Separate transactions that attempt to insert values of 5 and 6, respectively, each lock the gap between 4 and 7 with insert intention locks prior to obtaining the exclusive lock on the inserted row, but do not block each other because the rows are nonconflicting.

这段话表明尽管插入意向锁是一种特殊的间隙锁，但不同于间隙锁的是，该锁只用于并发插入操作。如果说间隙锁锁住的是一个区间，那么插入意向锁锁住的就是一个点。因而从这个角度来说，插入意向锁确实是一种特殊的间隙锁。与间隙锁的另一个非常重要的差别是：尽管插入意向锁也属于间隙锁，但两个事务却不能在同一时间内一个拥有间隙锁，另一个拥有该间隙区间内的插入意向锁（当然，插入意向锁如果不在间隙锁区间内则是可以的）。这里我们再回顾一下共享锁和排他锁：共享锁用于读取操作，而排他锁是用于更新或删除操作。也就是说插入意向锁、共享锁和排他锁涵盖了常用的增删改查四个动作。

#### 示例分析

到此为止，我们介绍了MySQL常用的七种锁的前六种，理解了这六种锁之后，才能很好地分析和理解开头给出的两个场景。

我们先来分析场景一：

| session1                                                     | sessioin2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| begin;                                                       |                                                              |
|                                                              | begin;                                                       |
| select * from test where id = 12 for update;<br /><span style='color:red;'>先请求IX锁并成功获取<br />再请求X锁，但因行记录不存在，故得到的是间隙锁（10，15）</span> |                                                              |
|                                                              | select * from test where id = 13 for update;<br/><span style='color:red;'>先请求IX锁并成功获取<br/>再请求X锁，但因行记录不存在，故得到的是间隙锁（10，15）</span> |
| insert into test(id, name) values(12, "test1");<br/><span style='color:red;'>请求插入意向锁（12），因事务二已有间隙锁，请求只能等待</span> |                                                              |
| 锁等待中                                                     | insert into test(id, name) values(13, "test2");<br/><span style='color:red'>请求插入意向锁（13），因事务一已有间隙锁，请求只能等待</span> |
| 锁等待解除                                                   | 死锁，session 2的事务被回滚                                  |

在场景一中，因为IX锁是表锁且IX锁之间是兼容的，因而事务一和事务二都能同时获取到IX锁和间隙锁。另外，需要说明的是，因为我们的隔离级别是RR，且在请求X锁的时候，查询的对应记录都不存在，因而返回的都是间隙锁。接着事务一请求插入意向锁，这时发现事务二已经获取了一个区间间隙锁，而且事务一请求的插入点在事务二的间隙锁区间内，因而只能等待事务二释放间隙锁。这个时候事务二也请求插入意向锁，该插入点同样位于事务一已经获取的间隙锁的区间内，因而也不能获取成功，不过这个时候，MySQL已经检查到了死锁，于是事务二被回滚，事务一提交成功。

分析并理解了场景一，那场景二理解起来就会简单多了：

| session1                                                     | session2                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| begin;                                                       |                                                              |
|                                                              | begin;                                                       |
| select * from test where id = 12 for update;<br/><span style='color:red;'>先请求IX锁并成功获取<br/>再请求X锁，但因行记录不存在，故得到的是间隙锁（10，15）</span> |                                                              |
|                                                              | select * from test where id = 16 for update;<br/><span style='color:red;'>先请求IX锁并成功获取<br/>再请求X锁，但因行记录不存在，故得到的是间隙锁（15，20）</span> |
| insert into test(id, name) values(12, "test1");<br/><span style='color:red;'>请求插入意向锁（12），获取成功</span> |                                                              |
| commit;                                                      | insert into test(id, name) values(16, "test2");<br/><span style='color:red;'>请求插入意向锁（16），获取成功</span> |
|                                                              | commit;                                                      |

场景二中，两个间隙锁没有交集，而各自获取的插入意向锁也不是同一个点，因而都能执行成功。

#### 自增锁

最后，我们再来介绍下自增锁。在MySQL的官方文档中有以下描述：

An AUTO-INC lock is a special table-level lock taken by transactions inserting into tables with AUTO_INCREMENT columns.The innodb_autoinc_lock_mode configuration option controls the algorithm used for auto-increment locking. It allows you to choose how to trade off between predictable sequences of auto-increment values and maximum concurrency for insert operations.

这段话表明自增锁是一种特殊的表级锁，主要用于事务中插入自增字段，也就是我们最常用的自增主键id。通过innodb_autoinc_lock_mode参数可以设置自增主键的生成策略。为了便于介绍innodb_autoinc_lock_mode参数，我们先将需要用到自增锁的Insert语句进行分类：

**1）Insert语句分类**
“INSERT-like” statements(类INSERT语句) （这种语句实际上包含了下面的2、3、4）

所有可以向表中增加行的语句，包括INSERT, INSERT ... SELECT, REPLACE, REPLACE ... SELECT, and LOAD DATA。包括“simple-inserts”, “bulk-inserts”, and “mixed-mode” inserts.

2. “Simple inserts”

可以预先确定要插入的行数（当语句被初始处理时）的语句。 这包括没有嵌套子查询的单行和多行INSERT和REPLACE语句，但不包括INSERT ... ON DUPLICATE KEY UPDATE。

3. “Bulk inserts”

事先不知道要插入的行数（和所需自动递增值的数量）的语句。 这包括INSERT ... SELECT，REPLACE ... SELECT和LOAD DATA语句，但不包括纯INSERT。 InnoDB在处理每行时一次为AUTO_INCREMENT列分配一个新值。

4. “Mixed-mode inserts”

这些是“Simple inserts”语句但是指定一些（但不是全部）新行的自动递增值。 示例如下，其中c1是表t1的AUTO_INCREMENT列：

INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');

另一种类型的“Mixed-mode inserts”是INSERT ... ON DUPLICATE KEY UPDATE，其在最坏的情况下实际上是INSERT语句随后又跟了一个UPDATE，其中AUTO_INCREMENT列的分配值不一定会在 UPDATE 阶段使用。

**2）InnoDB AUTO_INCREMENT锁定模式分类**
innodb_autoinc_lock_mode = 0 (“traditional” lock mode)

这种锁定模式提供了在MySQL 5.1中引入innodb_autoinc_lock_mode配置参数之前存在的相同行为。传统的锁定模式选项用于向后兼容性，性能测试以及解决“Mixed-mode inserts”的问题，因为语义上可能存在差异。

在此锁定模式下，所有“INSERT-like”语句获得一个特殊的表级AUTO-INC锁，用于插入具有AUTO_INCREMENT列的表。此锁定通常保持到语句结束（不是事务结束），以确保为给定的INSERT语句序列以可预测和可重复的顺序分配自动递增值，并确保自动递增由任何给定语句分配的值是连续的。

在基于语句复制(statement-based replication)的情况下，这意味着当在从服务器上复制SQL语句时，自动增量列使用与主服务器上相同的值。多个INSERT语句的执行结果是确定性的，SLAVE再现与MASTER相同的数据（反之，如果由多个INSERT语句生成的自动递增值交错，则两个并发INSERT语句的结果将是不确定的，并且不能使用基于语句的复制可靠地传播到从属服务器）。

innodb_autoinc_lock_mode = 1 (“consecutive” lock mode)

这是默认的锁定模式。在这个模式下,“bulk inserts”仍然使用AUTO-INC表级锁,并保持到语句结束.这适用于所有INSERT ... SELECT，REPLACE ... SELECT和LOAD DATA语句。同一时刻只有一个语句可以持有AUTO-INC锁。

而“Simple inserts”（要插入的行数事先已知）通过在mutex（轻量锁）的控制下获得所需数量的自动递增值来避免表级AUTO-INC锁， 它只在分配过程的持续时间内保持，而不是直到语句完成。 不使用表级AUTO-INC锁，除非AUTO-INC锁由另一个事务保持。 如果另一个事务保持AUTO-INC锁，则“简单插入”等待AUTO-INC锁，如同它是一个“批量插入”。

此锁定模式确保,当行数不预先知道的INSERT存在时(并且自动递增值在语句过程执行中分配)由任何“类INSERT”语句分配的所有自动递增值是连续的，并且对于基于语句的复制(statement-based replication)操作是安全的。

这种锁定模式显著地提高了可扩展性,并且保证了对于基于语句的复制(statement-based replication)的安全性。此外，与“传统”锁定模式一样，由任何给定语句分配的自动递增数字是连续的。 与使用自动递增的任何语句的“传统”模式相比，语义没有变化，但有个特殊场景需要注意：The exception is for “mixed-mode inserts”, where the user provides explicit values for an AUTO_INCREMENT column for some, but not all, rows in a multiple-row “simple insert”. For such inserts, InnoDB allocates more auto-increment values than the number of rows to be inserted. However, all values automatically assigned are consecutively generated (and thus higher than) the auto-increment value generated by the most recently executed previous statement. “Excess” numbers are lost.

也就说对于混合模式的插入，可能会有部分多余自增值丢失。

在连续锁定模式下，InnoDB可以避免为“Simple inserts”语句使用表级AUTO-INC锁，其中行数是预先已知的，并且仍然保留基于语句的复制的确定性执行和安全性。

innodb_autoinc_lock_mode = 2 (“interleaved” lock mode)

在这种锁定模式下,所有类INSERT(“INSERT-like” )语句都不会使用表级AUTO-INC lock,并且可以同时执行多个语句。这是最快和最可扩展的锁定模式，但是当使用基于语句的复制或恢复方案时，从二进制日志重播SQL语句时，这是不安全的。

在此锁定模式下，自动递增值保证在所有并发执行的“类INSERT”语句中是唯一且单调递增的。但是，由于多个语句可以同时生成数字（即，跨语句交叉编号），为任何给定语句插入的行生成的值可能不是连续的。

如果执行的语句是“simple inserts”，其中要插入的行数已提前知道，则除了“混合模式插入”之外，为单个语句生成的数字不会有间隙。然而，当执行“批量插入”时，在由任何给定语句分配的自动递增值中可能存在间隙。

如果不使用二进制日志作为恢复或复制的一部分来重放SQL语句，则可以使用interleaved lock模式来消除所有使用表级AUTO-INC锁，以实现更大的并发性和性能,其代价是由于并发的语句交错执行,同一语句生成的AUTO-INCREMENT值可能会产生GAP。

innodb_autoinc_lock_mode参数的修改

编辑/etc/my.cnf，加入如下行:

innodb_autoinc_lock_mode=2
直接通过命令修改会报错:

mysql(mdba@localhost:(none) 09:32:19)>set global innodb_autoinc_lock_mode=2;

ERROR 1238 (HY000): Variable 'innodb_autoinc_lock_mode' is a read only variable
**3）InnoDB AUTO_INCREMENT锁定模式含义**
在复制环节中使用自增列

如果你在使用基于语句的复制(statement-based replication)请将innodb_autoinc_lock_mode设置为0或1，并在主从上使用相同的值。 如果使用innodb_autoinc_lock_mode = 2（“interleaved”）或主从不使用相同的锁定模式的配置，自动递增值不能保证在从机上与主机上相同。

如果使用基于行的或混合模式的复制，则所有自动增量锁定模式都是安全的，因为基于行的复制对SQL语句的执行顺序不敏感（混合模式会在遇到不安全的语句是使用基于行的复制模式）。

2. “Lost” auto-increment values and sequence gaps

在所有锁定模式（0,1和2）中，如果生成自动递增值的事务回滚，那些自动递增值将“丢失”。 一旦为自动增量列生成了值，无论是否完成“类似INSERT”语句以及包含事务是否回滚，都不能回滚。 这种丢失的值不被重用。 因此，存储在表的AUTO_INCREMENT列中的值可能存在间隙。

3. Specifying NULL or 0 for the AUTO_INCREMENT column

在所有锁定模式（0,1和2）中，如果用户在INSERT中为AUTO_INCREMENT列指定NULL或0，InnoDB会将该行视为未指定值，并为其生成新值。

4. 为AUTO_INCREMENT列分配一个负值

在所有锁定模式（0,1和2）中，如果您为AUTO_INCREMENT列分配了一个负值，则InnoDB会将该行为视为未指定值，并为其生成新值。

5. 如果AUTO_INCREMENT值大于指定整数类型的最大整数

在所有锁定模式（0,1和2）中，如果值大于可以存储在指定整数类型中的最大整数，则InnoDB会将该值设置为指定类型所允许的最大值。

6. Gaps in auto-increment values for “bulk inserts”

当innodb_autoinc_lock_mode设置为0（“traditional”）或1（“consecutive”）时,任何给定语句生成的自动递增值是连续的，没有间隙，因为表级AUTO-INC锁会持续到 语句结束,并且一次只能执行一个这样的语句。

当innodb_autoinc_lock_mode设置为2（“interleaved”）时，在“bulk inserts”生成的自动递增值中可能存在间隙，但只有在并发执行“INSERT-Like”语句时才会产生这种情况。

对于锁定模式1或2，在连续语句之间可能出现间隙，因为对于批量插入，每个语句所需的自动递增值的确切数目可能不为人所知，并且可能进行过度估计。

7. 由“mixed-mode inserts”分配的自动递增值

考虑一下场景,在“mixed-mode insert”中,其中一个“simple insert”语句指定了一些（但不是全部）行的AUTO-INCREMENT值。 这样的语句在锁模式0,1和2中表现不同。innodb_autoinc_lock_mode=0时,auto-increment值一次只分配一个,而不是在开始时全部分配。当innodb_autoinc_lock_mode=1时，不同于innodb_autoinc_lock_mode=0时的情况，因为auto-increment值在语句一开始就分配了，但实际可能使用不完。当innodb_autoinc_lock_mode=2时，取决于并发语句的执行顺序。

8. 在INSERT语句序列的中间修改AUTO_INCREMENT列值

在所有锁定模式（0,1和2）中，在INSERT语句序列中间修改AUTO_INCREMENT列值可能会导致duplicate key错误。

**4）InnoDB AUTO_INCREMENT计数器初始化**
如果你为一个Innodb表创建了一个AUTO_INCREMENT列，则InnoDB数据字典中的表句柄包含一个称为自动递增计数器的特殊计数器，用于为列分配新值。 此计数器仅存在于内存中，而不存储在磁盘上。

要在服务器重新启动后初始化自动递增计数器，InnoDB将在首次插入行到包含AUTO_INCREMENT列的表时执行以下语句的等效语句。

SELECT MAX(ai_col) FROM table_name FOR UPDATE;
InnoDB增加语句检索的值，并将其分配给表和表的自动递增计数器。 默认情况下，值增加1。此默认值可以由auto_increment_increment配置设置覆盖。

如果表为空，InnoDB使用值1。此默认值可以由auto_increment_offset配置设置覆盖。

如果在自动递增计数器初始化前使用SHOW TABLE STATUS语句查看表, InnoDB将初始化计数器值,但不会递增该值。这个值会储存起来以备之后的插入语句使用。这个初始化过程使用了一个普通的排它锁来读取表中自增列的最大值。InnoDB遵循相同的过程来初始化新创建的表的自动递增计数器。

在自动递增计数器初始化之后，如果您未明确指定AUTO_INCREMENT列的值，InnoDB会递增计数器并将新值分配给该列。如果插入显式指定列值的行，并且该值大于当前计数器值，则将计数器设置为指定的列值。

只要服务器运行，InnoDB就使用内存中自动递增计数器。当服务器停止并重新启动时，InnoDB会重新初始化每个表的计数器，以便对表进行第一次INSERT，如前所述。

服务器重新启动还会取消CREATE TABLE和ALTER TABLE语句中的AUTO_INCREMENT = N表选项的效果（可在建表时可用“AUTO_INCREMENT=n”选项来指定一个自增的初始值，也可用alter table table_name AUTO_INCREMENT=n命令来重设自增的起始值）。

#### 空间锁

官方原文

 `InnoDB` supports `SPATIAL` indexing of columns containing spatial columns (see [Section 11.4.9, “Optimizing Spatial Analysis”](https://dev.mysql.com/doc/refman/8.0/en/optimizing-spatial-analysis.html)).

To handle locking for operations involving `SPATIAL` indexes, next-key locking does not work well to support [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) or [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable) transaction isolation levels. There is no absolute ordering concept in multidimensional data, so it is not clear which is the “next” key.

To enable support of isolation levels for tables with `SPATIAL` indexes, `InnoDB` uses predicate locks. A `SPATIAL` index contains minimum bounding rectangle (MBR) values, so `InnoDB` enforces consistent read on the index by setting a predicate lock on the MBR value used for a query. Other transactions cannot insert or modify a row that would match the query condition.

InnoDB支持包含空间列的列的空间索引（请参见第11.4.9节“优化空间分析”）。

为了处理涉及SPATIAL索引的操作的锁定，临键锁定不能很好地支持REPEATABLE READ或SERIALIZABLE事务隔离级别。 多维数据中没有绝对排序概念，因此尚不清楚哪个是“临”键。

为了支持具有SPATIAL索引的表的隔离级别，InnoDB使用谓词锁。 SPATIAL索引包含最小边界矩形（MBR）值，因此InnoDB通过在用于查询的MBR值上设置谓词锁定来强制对索引进行一致的读取。 其他事务不能插入或修改将匹配查询条件的行。





参考博客：

https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html MySQL官网

https://www.cnblogs.com/rjzheng/p/9950951.html 史上最全的select加锁分析

https://blog.csdn.net/wufaliang003/article/details/81937418 InnoDB并发插入，居然使用意向锁

https://blog.csdn.net/ignorewho/article/details/86423147 插入意向锁

https://www.cnblogs.com/rjzheng/p/9955395.html MySQL事务隔离级别

https://www.jianshu.com/p/68b581481831AUTO-INC锁和AUTO_INCREMENT在InnoDB中处理方式

https://www.jianshu.com/p/10a8d8977aaf  MySQL自增锁模式innodb_autoinc_lock_mode参数详解

原文链接：https://blog.csdn.net/Saintyyu/article/details/91269087



**空间索引函数列表**

| 名称 | 描述                                                         |                                      |
| ---- | ------------------------------------------------------------ | ------------------------------------ |
| 1    | [ST_StartPoint()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-startpoint) | LineString的起始点                   |
| 2    | [ST_EndPoint()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-endpoint) | LineString的终点                     |
| 3    | [ST_Transform()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-transform) | 变换几何的坐标                       |
| 4    | [ST_GeoHash()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html#function_st-geohash) | 产生geohash值                        |
| 5    | [ST_LongFromGeoHash()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html#function_st-longfromgeohash) | 从geohash值返回经度                  |
| 6    | [ST_LatFromGeoHash()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html#function_st-latfromgeohash) | 从geohash值返回纬度                  |
| 7    | [ST_GeomFromGeoJSON()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geojson-functions.html#function_st-geomfromgeojson) | 从GeoJSON对象生成几何                |
| 8    | [Polygon()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_polygon) | 从LineString参数构造多边形           |
| 9    | [ST_PointN()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-pointn) | 从LineString返回第N个点              |
| 10   | [MultiLineString()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_multilinestring) | 从LineString值构造MultiLineString    |
| 11   | [LineString()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_linestring) | 从Point值构造LineString              |
| 12   | [MultiPoint()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_multipoint) | 从Point值构造MultiPoint              |
| 13   | [MultiPolygon()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_multipolygon) | 从Polygon值构造MultiPolygon          |
| 14   | [ST_GeomFromWKB()， ST_GeometryFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-geomfromwkb) | 从WKB返回几何                        |
| 15   | [ST_GeomCollFromWKB()， ST_GeometryCollectionFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-geomcollfromwkb) | 从WKB返回几何集合                    |
| 16   | [ST_LineFromWKB()， ST_LineStringFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-linefromwkb) | 从WKB构造LineString                  |
| 17   | [ST_MLineFromWKB()， ST_MultiLineStringFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-mlinefromwkb) | 从WKB构造MultiLineString             |
| 18   | [ST_MPointFromWKB()， ST_MultiPointFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-mpointfromwkb) | 从WKB构造MultiPoint                  |
| 19   | [ST_MPolyFromWKB()， ST_MultiPolygonFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-mpolyfromwkb) | 从WKB构造MultiPolygon                |
| 20   | [ST_PointFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-pointfromwkb) | 从WKB构造点                          |
| 21   | [ST_PolyFromWKB()， ST_PolygonFromWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-polyfromwkb) | 从WKB构造多边形                      |
| 22   | [ST_GeomFromText()， ST_GeometryFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-geomfromtext) | 从WKT返回几何                        |
| 23   | [ST_GeomCollFromText()，ST_GeometryCollectionFromText()，ST_GeomCollFromTxt()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-geomcollfromtext) | 从WKT返回几何集合                    |
| 24   | [ST_PointFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-pointfromtext) | 从WKT构建点                          |
| 25   | [ST_LineFromText()， ST_LineStringFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-linefromtext) | 从WKT构造LineString                  |
| 26   | [ST_MLineFromText()， ST_MultiLineStringFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-mlinefromtext) | 从WKT构造MultiLineString             |
| 27   | [ST_MPointFromText()， ST_MultiPointFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-mpointfromtext) | 从WKT构造MultiPoint                  |
| 28   | [ST_MPolyFromText()， ST_MultiPolygonFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-mpolyfromtext) | 从WKT构造MultiPolygon                |
| 29   | [ST_PolyFromText()， ST_PolygonFromText()](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-polyfromtext) | 从WKT构造多边形                      |
| 30   | [GeomCollection()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_geomcollection) | 从几何构造几何集合                   |
| 31   | [GeometryCollection()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_geometrycollection) | 从几何构造几何集合                   |
| 32   | [ST_GeometryN()](https://dev.mysql.com/doc/refman/8.0/en/gis-geometrycollection-property-functions.html#function_st-geometryn) | 从几何集合中返回第N个几何            |
| 33   | [ST_AsGeoJSON()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geojson-functions.html#function_st-asgeojson) | 从几何体生成GeoJSON对象              |
| 34   | [ST_AsBinary()， ST_AsWKB()](https://dev.mysql.com/doc/refman/8.0/en/gis-format-conversion-functions.html#function_st-asbinary) | 从内部几何格式转换为WKB              |
| 35   | [ST_AsText()， ST_AsWKT()](https://dev.mysql.com/doc/refman/8.0/en/gis-format-conversion-functions.html#function_st-astext) | 从内部几何格式转换为WKT              |
| 36   | [Point()](https://dev.mysql.com/doc/refman/8.0/en/gis-mysql-specific-functions.html#function_point) | 从坐标构造点                         |
| 37   | [ST_Length()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-length) | 返回LineString的长度                 |
| 38   | [ST_NumPoints()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-numpoints) | 返回LineString中的点数               |
| 39   | [ST_X()](https://dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html#function_st-x) | 返回Point的X坐标                     |
| 40   | [ST_Y()](https://dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html#function_st-y) | 返回Point的Y坐标                     |
| 41   | [ST_Longitude()](https://dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html#function_st-longitude) | 返回Point的经度                      |
| 42   | [ST_Latitude()](https://dev.mysql.com/doc/refman/8.0/en/gis-point-property-functions.html#function_st-latitude) | 返回Point的纬度                      |
| 43   | [ST_InteriorRingN()](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html#function_st-interiorringn) | 返回Polygon的第N个内环               |
| 44   | [ST_ExteriorRing()](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html#function_st-exteriorring) | 返回Polygon的外环                    |
| 45   | [ST_Area()](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html#function_st-area) | 返回Polygon或MultiPolygon区域        |
| 46   | [ST_Union()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-union) | 返回点集两个几何的并集               |
| 47   | [ST_SymDifference()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-symdifference) | 返回点设置两个几何的对称差异         |
| 48   | [ST_Intersection()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-intersection) | 返回点设置两个几何的交集             |
| 49   | [ST_NumInteriorRing()， ST_NumInteriorRings()](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html#function_st-numinteriorrings) | 返回多边形内圈的数量                 |
| 50   | [ST_Envelope()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-envelope) | 返回几何的MBR                        |
| 51   | [ST_SRID()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-srid) | 返回几何的空间参考系统ID             |
| 52   | [ST_NumGeometries()](https://dev.mysql.com/doc/refman/8.0/en/gis-geometrycollection-property-functions.html#function_st-numgeometries) | 返回几何集合中的几何数量             |
| 53   | [ST_GeometryType()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-geometrytype) | 返回几何类型的名称                   |
| 54   | [ST_ConvexHull()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-convexhull) | 返回几何体的凸包                     |
| 55   | [ST_Simplify()](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html#function_st-simplify) | 返回简化几何                         |
| 56   | [ST_Buffer()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-buffer) | 返回距离几何体的给定距离内的点的几何 |
| 57   | [ST_Validate()](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html#function_st-validate) | 返回验证的几何体                     |
| 58   | [ST_Centroid()](https://dev.mysql.com/doc/refman/8.0/en/gis-polygon-property-functions.html#function_st-centroid) | 返回质心作为一个点                   |
| 59   | [ST_Dimension()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-dimension) | 几何尺寸                             |
| 60   | [ST_IsClosed()](https://dev.mysql.com/doc/refman/8.0/en/gis-linestring-property-functions.html#function_st-isclosed) | 几何是否封闭且简单                   |
| 61   | [ST_IsSimple()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-issimple) | 几何是否简单                         |
| 62   | [ST_IsValid()](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html#function_st-isvalid) | 几何是否有效                         |
| 63   | [ST_PointFromGeoHash()](https://dev.mysql.com/doc/refman/8.0/en/spatial-geohash-functions.html#function_st-pointfromgeohash) | 将geohash值转换为POINT值             |
| 64   | [ST_SwapXY()](https://dev.mysql.com/doc/refman/8.0/en/gis-format-conversion-functions.html#function_st-swapxy) | 交换X / Y坐标的返回参数              |
| 65   | [ST_MakeEnvelope()](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html#function_st-makeenvelope) | 两点左右的矩形                       |
| 66   | [MBREquals()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrequals) | 两个几何的MBR是否相等                |
| 67   | [MBRIntersects()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrintersects) | 两个几何的MBR是否相交                |
| 68   | [MBROverlaps()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbroverlaps) | 两个几何的MBR是否重叠                |
| 69   | [ST_Difference()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-difference) | 两个几何的返回点集差异               |
| 70   | [MBRDisjoint()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrdisjoint) | 两个几何形状的MBR是否不相交          |
| 71   | [ST_Distance_Sphere()](https://dev.mysql.com/doc/refman/8.0/en/spatial-convenience-functions.html#function_st-distance-sphere) | 两个几何形状之间的最小地球距离       |
| 72   | [MBRTouches()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrtouches) | 两种几何形状的MBR是否接触            |
| 73   | [ST_Buffer_Strategy()](https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html#function_st-buffer-strategy) | 为ST_Buffer（）生成策略选项          |
| 74   | [MBRCoveredBy()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrcoveredby) | 一个MBR是否被另一个MBR覆盖           |
| 75   | [MBRCovers()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrcovers) | 一个MBR是否涵盖另一个MBR             |
| 76   | [MBRContains()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrcontains) | 一个几何的MBR是否包含另一个几何的MBR |
| 77   | [MBRWithin()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-mbr.html#function_mbrwithin) | 一个几何的MBR是否在另一个几何的MBR内 |
| 78   | [ST_Contains()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-contains) | 一个几何是否包含另一个               |
| 79   | [ST_Touches()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-touches) | 一个几何是否接触另一个               |
| 80   | [ST_Disjoint()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-disjoint) | 一个几何是否与另一个几何脱节         |
| 81   | [ST_Equals()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-equals) | 一个几何是否与另一个几何相等         |
| 82   | [ST_Crosses()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-crosses) | 一个几何是否与另一个几何相交         |
| 83   | [ST_Intersects()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-intersects) | 一个几何是否与另一个相交             |
| 84   | [ST_Overlaps()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-overlaps) | 一个几何是否与另一个重叠             |
| 85   | [ST_Within()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-within) | 一个几何是否在另一个之内             |
| 86   | [ST_Distance()](https://dev.mysql.com/doc/refman/8.0/en/spatial-relation-functions-object-shapes.html#function_st-distance) | 一个几何与另一个几何的距离           |
| 87   | [ST_IsEmpty()](https://dev.mysql.com/doc/refman/8.0/en/gis-general-property-functions.html#function_st-isempty) | 占位符功能                           |