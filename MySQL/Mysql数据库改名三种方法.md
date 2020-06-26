如果是MyISAM引擎的表可以直接移动表文件，InnoDB的不可以，会提示表不存在

1、

```sql
RENAME database olddbname TO newdbname
```

5.1.7到5.1.23版本可以用的，官方不推荐，会有丢失数据的危险

2、

创建新库，使用mysqldump导出数据，但如果数据量比较大，会比较耗时



3、

使用shell脚本

```shell
#!/bin/bash
# 假设将old_database数据库名改为new_database
# MyISAM直接更改数据库目录下的文件即可

mysql -uroot -p123456 -e 'create database if not exists new_database'
list_table=$(mysql -uroot -p123456 -Nse "select table_name from information_schema.TABLES where TABLE_SCHEMA='old_database'")

for table in $list_table
do
    mysql -uroot -p123456 -e "rename table old_database.$table to new_database.$table"
done
```

rename table   操作在表名前增加库名时，如果指向一个新的库名，则会复制表数据到新库，原库数据继续存在