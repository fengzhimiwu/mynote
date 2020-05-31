要创建一个管理员

grant all on *.* to 'username' identified by 'password' with grant option;﻿

撤销授权

revoke all privileges , grant option from 'username';

创建一个没有任何权限的常规用户

grant useage on datebases.* to 'username'@'localhost' identified by 'password';﻿

为用户授予适当的权限

grant select, insert, update, delete, index, alter, create, drop on databases.* to 'username'@'localhost';

为用户撤销某些权限

revoke alter, create, drop on databases.* from 'username'@'localhost';

当用户不再使用数据库时，撤销所有权限

revoke all on databases.* from 'username'@'localhost';

用户权限

| **权限**       | **应用于**       | **描述**                                                     |
| -------------- | ---------------- | ------------------------------------------------------------ |
| SELECT         | 表、列           | 允许用户从表中选择行（记录）                                 |
| INSERT         | 表、列           | 允许用户在表中插入新行                                       |
| UPDATE         | 表、列           | 允许用户修改现存表里行中的值                                 |
| DELETE         | 表               | 允许用户删除现存表的行                                       |
| INDEX          | 表               | 允许用户创建和拖动特定表索引                                 |
| ALTER          | 表               | 允许用户改变现存表的结构，例如，可添加列、重命名列或表、修改列的数据类型 |
| CREATE         | 数据库、表，索引 | 允许用户创建新数据库、表或索引。如果在GRANT中指定了一个特定的数据库、表或索引，用户就只能够创建它们，即用户必须首先删除它。 |
| DROP           | 数据库、表、试图 | 允许用删除数据库、表或试图                                   |
| EVENT          | 数据库           | 允许用户查看、创建、修改以及删除事件调度器中的事件           |
| TRIGGER        | 表               | 允许用户对已授权表执行创建、执行或删除触发器                 |
| CREATE VIEW    | 视图             | 允许用户创建视图                                             |
| SHOW VIEW      | 视图             | 允许用户查看创建的视图查询                                   |
| PROXY          | 所有对象         | 允许用户切换到其他用户，类似于UNIX的su命令                   |
| CREATE ROUTINE | 存储过程         | 允许用户创建存储过程和函数                                   |
| EXECUTE        | 存储过程         | 允许用户运行存储过程和函数                                   |
| ALTER ROUTINE  | 存储过程         | 允许用修改存储过程和函数的定义                               |

 

管理员权限

| **权限**                | **描述**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| CREATE TABLESPACE       | 允许管理员创建、修改或删除表空间                             |
| CREATE USER             | 允许管理员创建用户                                           |
| CREATE TEMPORARY TABLES | 允许管理员在CREATE TABLE语句中使用TEMPORARY关键字            |
| FILE                    | 允许将数据从文件读入表，或从表中读入文件                     |
| LOCK TABLES             | 允许显示使用LOCK TABLES语句                                  |
| PROCESS                 | 允许管理员查看属于所有用户的服务器进程                       |
| RELOAD                  | 允许管理员重新载入授权表、清空授权、主机、日志和表           |
| REPLICATIOIN CLIENT     | 允许在复制主机（Master）和从机（Slave）上使用SHOW STATUS。   |
| REPLICATIOIN SLAVE      | 允许复制从服务器连接到主服务器                               |
| SHOW DATABASES          | 允许使用SHOW DATABASES语句查看所有数据库列表。没有这个权限，用户只能看到他们能够看到的数据库 |
| SHUTDOWN                | 允许管理员关闭MySQL服务器                                    |
| SUPER                   | 允许管理员关闭属于任何用户的线程                             |

 

特殊权限

| **权限** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| ALL      | 授予用户权限表和管理员权限表列出的所有权限。也可以将ALL写成ALL PRIVILEGES |
| USAGE    | 不授予权限。这将创建一个用户并允许他登录，但是不允许进行任何操作。通常在以后会授予该用户更多的权限。使用GRANT和USAGE语句创建用户并授予权限等同于CREATE USER语句 |