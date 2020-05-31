# MySQL基础命令


>    本文来源于网络，可能存在错漏之处，仅供参考。

### 1. 连接MySQL： `mysql -h host_address -u user_name -p user_password`

```bash
mysql -h110.110.110.110 -u root -p 123;
```

### 2. 修改密码：`mysqladmin -u user_name -p old_password password new_password`

```bash
mysqladmin -u root -p abc123 password def456;
```

### 3. 增加新用户：`grant select on db_name.* to user_name@login_host identified by 'user_password'`

```bash
/* mysql grant命令添加用户常用的三种模式 */
grant all PRIVILEGES on *.* to 'test'@'localhost' identified by '123';
grant all PRIVILEGES on *.* to 'test'@'%' identified by '123';
grant all PRIVILEGES on *.* to 'test'@'10.22.225.18' identified by '123';
```

>    说明：  
第一条命令添加一个本地用户 'test' ，一般用于web服务器和数据库服务器在一起的情况； 
第二条命令添加一个用户 'test' ，只要能连接数据库服务器的机器都可以使用，这个比较危险，一般不用；  
最后条命令在数据库服务器上给 '10.22.225.18' 机器添加一个用户'test'，一般用于web服务器和数据库服务器分离的情况。


>    注意：  
真正使用的时候不会用 `grant all PRIVILEGES on *.*` ，而是根据实际需要设定相关的权限。
比如 `grant select,insert,delete,update on test.* to 'test'@'localhost' identified by '123';`


### 4. 创建数据库： `create database db_name`

```
create database news;
```

### 5. 显示数据库： `show databases`

### 6. 删除数据库： `drop database db_name`

```mysql
drop database news;
```

### 7. 连接数据库：`use db_name`

```mysql
use news;
```

`use` 语句可以通告MySQL把 `db_name` 数据库作为默认（当前）数据库使用，用于后续语句。该数据库保持为默认数据库，直到语段的结尾，或者直到发布一个不同的 `USE` 语句：

```
mysql> USE db1;
mysql> SELECT COUNT(*) FROM mytable;   # selects from db1.mytable
mysql> USE db2;
mysql> SELECT COUNT(*) FROM mytable;   # selects from db2.mytable
```

### 8. 选择的数据库：`select method()`

MySQL中 `SELECT` 命令类似于其他编程语言里的 `print` 或者 `write`，你可以用它来显示一个字符串、数字、数学表达式的结果等等。如何使用 `MySQL` 中 `SELECT` 命令的特殊功能？

① 显示MYSQL的版本

```mysql
mysql> select version(); 
+-----------------------+ 
| version()             | 
+-----------------------+ 
| 6.0.4-alpha-community | 
+-----------------------+ 
1 row in set (0.02 sec) 
```

② 显示当前时间

```mysql
mysql> select now(); 
+---------------------+ 
| now()               | 
+---------------------+ 
| 2009-09-15 22:35:32 | 
+---------------------+ 
1 row in set (0.04 sec) 
```

③ 显示年月日

```mysql
SELECT DAYOFMONTH(CURRENT_DATE); 
+--------------------------+ 
| DAYOFMONTH(CURRENT_DATE) | 
+--------------------------+ 
|                       15 | 
+--------------------------+ 
1 row in set (0.01 sec) 
  
SELECT MONTH(CURRENT_DATE); 
+---------------------+ 
| MONTH(CURRENT_DATE) | 
+---------------------+ 
|                   9 | 
+---------------------+ 
1 row in set (0.00 sec) 
  
SELECT YEAR(CURRENT_DATE); 
+--------------------+ 
| YEAR(CURRENT_DATE) | 
+--------------------+ 
|               2009 | 
+--------------------+ 
1 row in set (0.00 sec) 
```

④ 显示字符串

```mysql
mysql> SELECT "welecome to my blog!"; 
+----------------------+ 
| welecome to my blog! | 
+----------------------+ 
| welecome to my blog! | 
+----------------------+ 
1 row in set (0.00 sec) 
```

⑤ 当计算器用

```mysql
select ((4 * 4) / 10 ) + 25; 
+----------------------+ 
| ((4 * 4) / 10 ) + 25 | 
+----------------------+ 
|                26.60 | 
+----------------------+ 
1 row in set (0.00 sec) 
```

⑥ 串接字符串

```mysql
select CONCAT(f_name, " ", l_name) 
AS Name 
from employee_data 
where title = 'Marketing Executive'; 
+---------------+ 
| Name          | 
+---------------+ 
| Monica Sehgal | 
| Hal Simlai    | 
| Joseph Irvine | 
+---------------+ 
3 rows in set (0.00 sec) 
```

>    注意：这里用到 `CONCAT()` 函数，用来把字符串串接起来。另外，我们还用到以前学到的 `AS` 给结果列 `'CONCAT(f_name, " ", l_name)'` 起了个假名。

### 9. 创建数据表：`create table table_name (field_1_name field_1_type [ ,... field_n_name field_n_type ])`


```mysql
CREATE TABLE IF NOT EXISTS `user` (
  `uid` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `user_name` varchar(20) DEFAULT NULL,
  `user_password` varchar(32) DEFAULT NULL,
  `user_email` varchar(40) DEFAULT NULL,
  PRIMARY KEY (`uid`),
) ENGINE=MyISAM DEFAULT CHARSET=utf8 AUTO_INCREMENT=1 ;
```

### 10. 获取表结构：`desc table_name` 或者 `show columns from table_name`

>    使用MySQL数据库desc 表名时，我们看到Key那一栏，可能会有4种值，即 `' '` ， `'PRI'` ， `'UNI'` ， `'MUL'` 。

① 如果 `Key` 是空的, 那么该列值的可以重复, 表示该列没有索引, 或者是一个非唯一的复合索引的非前导列；

② 如果 `Key` 是 `PRI` ,  那么该列是主键的组成部分；

③ 如果 `Key` 是 `UNI` ,  那么该列是一个唯一值索引的第一列(前导列)，并别不能含有空值(`NULL`)；

④ 如果 `Key` 是 `MUL` ,  那么该列的值可以重复, 该列是一个非唯一索引的前导列(第一列)或者是一个唯一性索引的组成部分但是可以含有空值 `NULL`。
如果对于一个列的定义，同时满足上述4种情况的多种，比如一个列既是 `PRI` ，又是`UNI` ，那么 `desc table_name` 的时候，显示的`Key` 值按照优先级来显示 `PRI->UNI->MUL` 。那么此时，显示 `PRI` 。
一个唯一性索引列可以显示为 `PRI` ，并且该列不能含有空值，同时该表没有主键。
一个唯一性索引列可以显示为 `MUL` ，如果多列构成了一个唯一性复合索引，因为虽然索引的多列组合是唯一的，比如 `ID+NAME` 是唯一的，但是没一个单独的列依然可以有重复的值，只要 `ID+NAME` 是唯一的即可。

### 11. 删除表：`drop table table_name`

`DROP TABLE` 用于取消一个或多个表。您必须有每个表的 `DROP`权限。所有的表数据和表定义会被取消，所以使用本语句要小心！

### 12. 表插入数据：`insert into table_name ( field_1_name [ ,... field_n_name ])  values ( value_1 [,...  value_n ] )`

```mysql
INSERT INTO user (`uid`, `user_name`, `user_password`, `user_email`) VALUES (1, 'admin', 'admin',  'admin@example.com');
```
`insert into` 每次只能向表中插入一条记录。

### 13. 查询表数据：`select field_1_name [,... field_n_name ]  from table_name where sql_expression`

① 查询所有行：

查看表 `user` 中所有数据
`select * from user;`

② 查询前几行数据：

查看表 user 中前2行数据
`select * from user order by id limit 0,2;`
>    注：`select` 一般配合 `where` 使用，以查询更精确更复杂的数据。

### 14. 删除表中数据：`delete from table_name where sql_expression`

删除表 user 中编号为1 的记录
`delete from user where uid=1;`

### 15. 修改表中数据：`update table_name set field_name = new_value [ ,…] where sql_expression`

如更新 `id` 为 `1` 的 `user`，设置 `user_name`字段值为 `Mary`。

```mysql
update user set user_name='Mary' where id=1;
```

① 单表的MySQL UPDATE语句：

`UPDATE [LOW_PRIORITY] [IGNORE] tbl_name SET col_name1=expr1 [, col_name2=expr2 ...] [WHERE where_definition] [ORDER BY ...] [LIMIT row_count]`

② 多表的UPDATE语句：

`UPDATE [LOW_PRIORITY] [IGNORE] table_references SET col_name1=expr1 [, col_name2=expr2 ...] [WHERE where_definition]`

`UPDATE` 语法可以用新值更新原有表行中的各列。`SET` 子句指示要修改哪些列和要给予哪些值。`WHERE` 子句指定应更新哪些行。如果没有 `WHERE` 子句，则更新所有的行。如果指定了 `ORDER BY` 子句，则按照被指定的顺序对行进行更新。 `LIMIT` 子句用于给定一个限值，限制可以被更新的行的数目。

### 16. 增加字段：`alter table table_name [ add field_name field_type / other_sql_expression ]`

在表 `user` 中添加了一个字段 `user_pic` ，类型为 `varchar(40)`，默认值为 `NULL`
`alter table user add user_pic varchar(40) default NULL;`

加索引
`alter table employee add index emp_name (name);`

加主关键字的索引
`alter table employee add primary key(id);`

加唯一限制条件的索引
`alter table employee add unique emp_name2(cardnumber);`

删除某个索引
`alter table employee drop index emp_name;`

增加字段
`alter table user add user_pic varchar(40) default NULL;`

修改原字段名称及类型
`ALTER TABLE  table_name CHANGE old_field_name new_field_name field_type;`

删除字段
`ALTER TABLE table_name DROP field_name;`

### 17. 修改表名：`rename table old_table_name to new_table_name`

当你执行 `RENAME` 时，你不能有任何锁定的表或活动的事务。你同样也必须有对原初表的 `ALTER` 和 `DROP` 权限，以及对新表的 `CREATE` 和 `INSERT` 权限。
如果在多表更名中，`MySQL` 遭遇到任何错误，它将对所有被更名的表进行倒退更名，将每件事物退回到最初状态。

### 18.备份数据库：

① 导出整个数据库，导出文件默认是存在mysql\bin目录下
    `mysqldump -u user_name -p user_password db_name > new_db_name.sql`

② 导出一个表
    `mysqldump -u user_name -p user_password database_name table_name > outfile_name.sql`

③ 导出一个数据库结构
    `mysqldump -u user_name -puser_password -d -add-drop-table database_name > outfile_name.sql`
    `-d` 没有数据 `-add-drop-table` 在每个 `create` 语句之前增加一个 `drop table`

④带语言参数导出
    `mysqldump -u user_name -p user_password -default-character-set=latin1 -set-charset=gbk -skip-opt database_name > outfile_name.sql`

### 19. 建库建表示例：

```mysql
drop database if exists school; //如果存在SCHOOL则删除
create database school; //建立库SCHOOL
use school; //打开库SCHOOL
create table teacher //建立表TEACHER
(
    id int(3) auto_increment not null primary key,
    name char(10) not null,
    address varchar(50) default ‘深圳’,
    year date
); //建表结束

//以下为插入字段
insert into teacher values(”,’allen’,'大连一中’,'1976-10-10′);
insert into teacher values(”,’jack’,'大连二中’,'1975-12-23′);
```

如果你在 `mysql` 提示符键入上面的命令也可以，但不方便调试。

① 你可以将以上命令原样写入一个文本文件中，假设为 `school.sql` ，然后复制到 `c:\` 下，并在 `DOS` 状态进入目录 `mysql\bin` ，然后键入以下命令：
`mysql -u user_name -p user_password < c:\school.sql`

如果成功，空出一行无任何显示；如有错误，会有提示。（以上命令已经调试，你只要将//的注释去掉即可使用）。

② 或者进入命令行后使用 `mysql> source c:\school.sql; ` 也可以将 `school.sql` 文件导入数据库中。



DML（data manipulation language）数据操纵语言：

就是我们最经常用到的 SELECT、UPDATE、INSERT、DELETE。 主要用来对数据库的数据进行一些操作。

SELECT 列名称 FROM 表名称
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
DELETE FROM 表名称 WHERE 列名称 = 值



DDL（data definition language）数据库定义语言：
　　　　其实就是我们在创建表的时候用到的一些sql，比如说：CREATE、ALTER、DROP等。DDL主要是用在定义或改变表的结构，数据类型，表之间的链接和约束等初始化工作上


CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)

ALTER TABLE table_name
ALTER COLUMN column_name datatype

DROP TABLE 表名称
DROP DATABASE 数据库名称



DCL（Data Control Language）数据库控制语言：
　　　　是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。这个比较少用到。









1 SHOW DATABASES;  默认数据库：
　   　　  mysql - 用户权限相关数据
　　　　　　test - 用于用户测试数据
　　　　　　information_schema - MySQL本身架构相关数据
 1.2、创建数据库
\# utf-8
CREATE DATABASE 数据库名称 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

\# gbk
CREATE DATABASE 数据库名称 DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci; 1.3、打开数据库
USE db_name;
注：每次使用数据库必须打开相应数据库显示当前使用的数据库中所有表：SHOW TABLES;
 1.4、用户管理
         用户设置:
创建用户
create user '用户名'@'IP地址' identified by '密码';
删除用户
drop user '用户名'@'IP地址';
修改用户
rename user '用户名'@'IP地址'; to '新用户名'@'IP地址';;
修改密码
set password for '用户名'@'IP地址' = Password('新密码')

PS：用户权限相关数据保存在mysql数据库的user表中，所以也可以直接对其进行操作（不建议）
用户权限设置:
show grants for '用户'@'IP地址' -- 查看权限
grant 权限 on 数据库.表 to '用户'@'IP地址' -- 授权
revoke 权限 on 数据库.表 from '用户'@'IP地址' -- 取消权限
all privileges 除grant外的所有权限
select 仅查权限
select,insert 查和插入权限
...
usage 无访问权限
alter 使用alter table
alter routine 使用alter procedure和drop procedure
create 使用create table
create routine 使用create procedure
create temporary tables 使用create temporary tables
create user 使用create user、drop user、rename user和revoke all privileges
create view 使用create view
delete 使用delete
drop 使用drop table
execute 使用call和存储过程
file 使用select into outfile 和 load data infile
grant option 使用grant 和 revoke
index 使用index
insert 使用insert
lock tables 使用lock table
process 使用show full processlist
select 使用select
show databases 使用show databases
show view 使用show view
update 使用update
reload 使用flush
shutdown 使用mysqladmin shutdown(关闭MySQL)
super 使用change master、kill、logs、purge、master和set global。还允许mysqladmin调试登陆
replication client 服务器位置的访问
replication slave 由复制从属使用
对于目标数据库以及内部其他：
数据库名.* 数据库中的所有
数据库名.表 指定数据库中的某张表
数据库名.存储过程 指定数据库中的存储过程
*.* 所有数据库

用户名@IP地址 用户只能在改IP下才能访问
用户名@192.168.1.% 用户只能在改IP段下才能访问(通配符%表示任意)
用户名@% 用户可以再任意IP下访问(默认IP地址为%)

grant all privileges on db1.tb1 TO '用户名'@'IP'

grant select on db1.* TO '用户名'@'IP'

grant select,insert on *.* TO '用户名'@'IP'

revoke select on db1.tb1 from '用户名'@'IP'
1.4、备份库和恢复库
     备份库：
     MySQL备份和还原,都是利用mysqldump、mysql和source命令来完成。
   1.在Windows下MySQL的备份与还原
备份
**1**、开始菜单 | 运行 | cmd |利用“cd /Program Files/MySQL/MySQL Server **5.0**/bin”命令进入bin文件夹
**2**、利用“mysqldump -u 用户名 -p databasename >exportfilename”导出数据库到文件，如mysqldump -u root -p voice>voice.sql，然后输入密码即可开始导出。

还原
**1**、进入MySQL Command Line Client，输入密码，进入到“mysql>”。
**2**、输入命令"show databases；"，回车，看看有些什么数据库；建立你要还原的数据库，输入"create database voice；"，回车。
**3**、切换到刚建立的数据库，输入"use voice；"，回车；导入数据，输入"source voice.sql；"，回车，开始导入，再次出现"mysql>"并且没有提示错误即还原成功。
2、在linux下MySQL的备份与还原
**2.1** 备份(利用命令mysqldump进行备份)
[root@localhost mysql]# mysqldump -u root -p voice>voice.sql，输入密码即可。
**2.2** 还原
方法一：
[root@localhost ~]# mysql -u root -p 回车，输入密码，进入MySQL的控制台"mysql>"，同1.2还原。
方法二：
[root@localhost mysql]# mysql -u root -p voice<voice.sql，输入密码即可。  3、更多备份及还原命令
备份：
**1**.备份全部数据库的数据和结构

mysqldump -uroot -p123456 -A >F:\all.sql

**2**.备份全部数据库的结构（加 -d 参数）

mysqldump -uroot -p123456 -A -d>F:\all_struct.sql

**3**.备份全部数据库的数据(加 -t 参数)

mysqldump -uroot -p123456 -A -t>F:\all_data.sql

**4**.备份单个数据库的数据和结构(,数据库名mydb)

mysqldump -uroot -p123456 mydb>F:\mydb.sql

**5**.备份单个数据库的结构

mysqldump -uroot -p123456 mydb -d>F:\mydb.sql

**6**.备份单个数据库的数据

mysqldump -uroot -p123456 mydb -t>F:\mydb.sql

**7**.备份多个表的数据和结构（数据，结构的单独备份方法与上同）

mysqldump -uroot -p123456 mydb t1 t2 >f:\multables.sql

**8**.一次备份多个数据库

mysqldump -uroot -p123456 --databases db1 db2 >f:\muldbs.sql
还原：
还原部分分（**1**）mysql命令行source方法 和 （**2**）系统命令行方法

**1**.还原全部数据库:

(**1**) mysql命令行：mysql>source f:\all.sql

(**2**) 系统命令行： mysql -uroot -p123456 <f:\all.sql

**2**.还原单个数据库(需指定数据库)

(**1**) mysql>use mydb

mysql>source f:\mydb.sql

(**2**) mysql -uroot -p123456 mydb <f:\mydb.sql

**3**.还原单个数据库的多个表(需指定数据库)

(**1**) mysql>use mydb

mysql>source f:\multables.sql

(**2**) mysql -uroot -p123456 mydb <f:\multables.sql

**4**.还原多个数据库，（一个备份文件里有多个数据库的备份，此时不需要指定数据库）

(**1**) mysql命令行：mysql>source f:\muldbs.sql

(**2**) 系统命令行： mysql -uroot -p123456 <f:\muldbs.sql
更多备份知识：
http://www.jb51.net/article/41570.htm
（二）数据表的创建
 1.1、显示数据表
show tables; 1.2、创建数据表
create table 表名(
列名 类型 是否可以为空，
列名 类型 是否可以为空
)ENGINE=InnoDB DEFAULT CHARSET=utf8 
是否可空，null表示空，非字符串
not null - 不可空
null - 可空

默认值，创建列时可以指定默认值，当插入数据时如果未主动设置，则自动添加默认值
create table tb1(
nid int not null defalut 2,
num int not null
)
自增，如果为某列设置自增列，插入数据时无需设置此列，默认将自增（表中只能有一个自增列）
create table tb1(
nid int not null auto_increment primary key,
num int null
)
或
create table tb1(
nid int not null auto_increment,
num int null,
index(nid)
)
注意：1、对于自增列，必须是索引（含主键）。
2、对于自增可以设置步长和起始值
show session variables like 'auto_inc%';
set session auto_increment_increment=2;
set session auto_increment_offset=10;

shwo global variables like 'auto_inc%';
set global auto_increment_increment=2;
set global auto_increment_offset=10;
主键，一种特殊的唯一索引，不允许有空值，如果主键使用单个列，则它的值必须唯一，如果是多列，则其组合必须唯一。
create table tb1(
nid int not null auto_increment primary key,
num int null
)
或
create table tb1(
nid int not null,
num int not null,
primary key(nid,num)
)
外键，一个特殊的索引，只能是指定内容
creat table color(
nid int not null primary key,
name char(16) not null
)

create table fruit(
nid int not null primary key,
smt char(32) null ,
color_id int not null,
constraint fk_cc foreign key (color_id) references color(nid)
)主键与外键关系(非常重要)
http://www.cnblogs.com/programmer-tlh/p/5782451.html
 1.3删除表
drop table 表名1.4、清空表
delete from 表名
truncate table 表名1.5、基本数据类型
MySQL的数据类型大致分为：数值、时间和字符串
bit[(M)]
二进制位（101001），m表示二进制位的长度（1-64），默认m＝1

tinyint[(m)] [unsigned] [zerofill]

小整数，数据类型用于保存一些范围的整数数值范围：
有符号：
-128 ～ 127.
无符号：
0 ～ 255

特别的： MySQL中无布尔值，使用tinyint(1)构造。

int[(m)][unsigned][zerofill]

整数，数据类型用于保存一些范围的整数数值范围：
有符号：
-2147483648 ～ 2147483647
无符号：
0 ～ 4294967295

特别的：整数类型中的m仅用于显示，对存储范围无限制。例如： int(5),当插入数据2时，select 时数据显示为： 00002

bigint[(m)][unsigned][zerofill]
大整数，数据类型用于保存一些范围的整数数值范围：
有符号：
-9223372036854775808 ～ 9223372036854775807
无符号：
0 ～ 18446744073709551615

decimal[(m[,d])] [unsigned] [zerofill]
准确的小数值，m是数字总个数（负号不算），d是小数点后个数。 m最大值为65，d最大值为30。

特别的：对于精确数值计算时需要用此类型
decaimal能够存储精确值的原因在于其内部按照字符串存储。

FLOAT[(M,D)] [UNSIGNED] [ZEROFILL]
单精度浮点数（非准确小数值），m是数字总个数，d是小数点后个数。
无符号：
-3.402823466E+38 to -1.175494351E-38,
0
1.175494351E-38 to 3.402823466E+38
有符号：
0
1.175494351E-38 to 3.402823466E+38

**** 数值越大，越不准确 ****

DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]
双精度浮点数（非准确小数值），m是数字总个数，d是小数点后个数。

无符号：
-1.7976931348623157E+308 to -2.2250738585072014E-308
0
2.2250738585072014E-308 to 1.7976931348623157E+308
有符号：
0
2.2250738585072014E-308 to 1.7976931348623157E+308
**** 数值越大，越不准确 ****


char (m)
char数据类型用于表示固定长度的字符串，可以包含最多达255个字符。其中m代表字符串的长度。
PS: 即使数据小于m长度，也会占用m长度
varchar(m)
varchars数据类型用于变长的字符串，可以包含最多达255个字符。其中m代表该数据类型所允许保存的字符串的最大长度，只要长度小于该最大值的字符串都可以被保存在该数据类型中。

注：虽然varchar使用起来较为灵活，但是从整个系统的性能角度来说，char数据类型的处理速度更快，有时甚至可以超出varchar处理速度的50%。因此，用户在设计数据库时应当综合考虑各方面的因素，以求达到最佳的平衡

text
text数据类型用于保存变长的大字符串，可以组多到65535 (2**16 − 1)个字符。

mediumtext
A TEXT column with a maximum length of 16,777,215 (2**24 − 1) characters.

longtext
A TEXT column with a maximum length of 4,294,967,295 or 4GB (2**32 − 1) characters.


enum
枚举类型，
An ENUM column can have a maximum of 65,535 distinct elements. (The practical limit is less than 3000.)
示例：
CREATE TABLE shirts (
name VARCHAR(40),
size ENUM('x-small', 'small', 'medium', 'large', 'x-large')
);
INSERT INTO shirts (name, size) VALUES ('dress shirt','large'), ('t-shirt','medium'),('polo shirt','small');

set
集合类型
A SET column can have a maximum of 64 distinct members.
示例：
CREATE TABLE myset (col SET('a', 'b', 'c', 'd'));
INSERT INTO myset (col) VALUES ('a,d'), ('d,a'), ('a,d,a'), ('a,d,d'), ('d,a,d');

DATE
YYYY-MM-DD（1000-01-01/9999-12-31）

TIME
HH:MM:SS（'-838:59:59'/'838:59:59'）

YEAR
YYYY（1901/2155）

DATETIME

YYYY-MM-DD HH:MM:SS（1000-01-01 00:00:00/9999-12-31 23:59:59 Y）

TIMESTAMP

YYYYMMDD HHMMSS（1970-01-01 00:00:00/2037 年某时）
1.6、修改表(alter)  
添加列：alter table 表名 add 列名 类型
删除列：alter table 表名 drop column 列名
修改列：
alter table 表名 modify column 列名 类型; -- 类型
alter table 表名 change 原列名 新列名 类型; -- 列名，类型

添加主键：
alter table 表名 add primary key(列名);
删除主键：
alter table 表名 drop primary key;
alter table 表名 modify 列名 int, drop primary key;

添加外键：alter table 从表 add constraint 外键名称（形如：FK_从表_主表） foreign key 从表(外键字段) references 主表(主键字段);
删除外键：alter table 表名 drop foreign key 外键名称

修改默认值：ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000;
删除默认值：ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
更多参考：
• http://www.runoob.com/mysql/mysql-data-types.html
1.7、数据表关系
关联映射：一对多/多对一
 存在最普遍的映射关系，简单来讲就如球员与球队的关系； 一对多：从球队角度来说一个球队拥有多个球员 即为一对多 多对一：从球员角度来说多个球员属于一个球队 即为多对一数据表间一对多关系如下图：

![file:///tmp/ct_tmp/5.png](https://note.daiem.cn/api/file/getImage?fileId=5adef03f406b8b084a000021)

关联映射：一对一 一对一关系就如球队与球队所在地址之间的关系，一支球队仅有一个地址，而一个地址区也仅有一支球队。 数据表间一对一关系的表现有两种，一种是外键关联，一种是主键关联。图示如下： 一对一外键关联：

![file:///tmp/ct_tmp/6.png](https://note.daiem.cn/api/file/getImage?fileId=5adef03f406b8b084a00001b)

一对一主键关联：要求两个表的主键必须完全一致，通过两个表的主键建立关联关系![file:///tmp/ct_tmp/7.png](https://note.daiem.cn/api/file/getImage?fileId=5adef03f406b8b084a00001d)

关联映射：多对多 多对多关系也很常见，例如学生与选修课之间的关系，一个学生可以选择多门选修课，而每个选修课又可以被多名学生选择。 数据库中的多对多关联关系一般需采用中间表的方式处理，将多对多转化为两个一对多

![file:///tmp/ct_tmp/8.png](https://note.daiem.cn/api/file/getImage?fileId=5adef03f406b8b084a000020)

1.8、数据表之间的约束约束是一种限制，它通过对表的行或列的数据做出限制，来确保表的数据的完整性、唯一性。
MYSQL中，常用的几种约束：
![file:///tmp/ct_tmp/9.png](https://note.daiem.cn/api/file/getImage?fileId=5adef03f406b8b084a00001f)
===================================================
主键(PRIMARY KEY)是用于约束表中的一行，作为这一行的标识符，在一张表中通过主键就能准确定位到一行，因此主键十分重要。主键要求这一行的数据不能有重复且不能为空。
 还有一种特殊的主键——复合主键。主键不仅可以是表中的一列，也可以由表中的两列或多列来共同标识
===================================================
默认值约束(DEFAULT)规定，当有DEFAULT约束的列，插入数据为空时该怎么办。
DEFAULT约束只会在使用INSERT语句（上一实验介绍过）时体现出来，INSERT语句中，如果被DEFAULT约束的位置没有值，那么这个位置将会被DEFAULT的值填充
===================================================
唯一约束(UNIQUE)比较简单，它规定一张表中指定的一列的值必须不能有重复值，即这一列每个值都是唯一的。
当INSERT语句新插入的数据和已有数据重复的时候，如果有UNIQUE约束，则INSERT失败.
===================================================
外键(FOREIGN KEY)既能确保数据完整性，也能表现表之间的关系。
一个表可以有多个外键，每个外键必须REFERENCES(参考)另一个表的主键，被外键约束的列，取值必须在它参考的列中有对应值。
在INSERT时，如果被外键约束的值没有在参考列中有对应，比如以下命令，参考列(department表的dpt_name)中没有dpt3，则INSERT失败
===================================================
非空约束(NOT NULL),听名字就能理解，被非空约束的列，在插入值时必须非空。
在MySQL中违反非空约束，不会报错，只会有警告.
CREATE DATABASE mysql_shiyan;

use mysql_text;

CREATE TABLE department
(
dpt_name CHAR(**20**) NOT NULL,
people_num INT(**10**) DEFAULT '10',
CONSTRAINT dpt_pk PRIMARY KEY (dpt_name)
);

CREATE TABLE employee
(
id INT(**10**) PRIMARY KEY,
name CHAR(**20**),
age INT(**10**),
salary INT(**10**) NOT NULL,
phone INT(**12**) NOT NULL,
in_dpt CHAR(**20**) NOT NULL,
UNIQUE (phone),
CONSTRAINT emp_fk FOREIGN KEY (in_dpt) REFERENCES department(dpt_name)
);

CREATE TABLE project
(
proj_num INT(**10**) NOT NULL,
proj_name CHAR(**20**) NOT NULL,
start_date DATE NOT NULL,
end_date DATE DEFAULT '2015-04-01',
of_dpt CHAR(**20**) REFERENCES department(dpt_name),
CONSTRAINT proj_pk PRIMARY KEY (proj_num,proj_name)
);