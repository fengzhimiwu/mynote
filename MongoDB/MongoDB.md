MongoDB 应用场景

mongodb 支持副本集、索引、自动分片，可以保证较高的性能和可用性。

更高的写入负载

默认情况下，MongoDB 更侧重高数据写入性能，而非事务安全，MongoDB 很适合业务系统中有大量 “低价值” 数据的场景。但是应当避免在高事务安全性的系统中使用 MongoDB，除非能从架构设计上保证事务安全。

高可用性

MongoDB 的复副集 (Master-Slave) 配置非常简洁方便，此外，MongoDB 可以快速响应的处理单节点故障，自动、安全的完成故障转移。这些特性使得 MongoDB 能在一个相对不稳定（如云主机）的环境中，保持高可用性。

数据量很大或者未来会变得很大

依赖数据库 (MySQL) 自身的特性，完成数据的扩展是较困难的事，在 MySQL 中，当一个单达表到 5-10GB 时会出现明显的性能降级，此时需要通过数据的水平和垂直拆分、库的拆分完成扩展，使用 MySQL 通常需要借助驱动层或代理层完成这类需求。而 MongoDB 内建了多种数据分片的特性，可以很好的适应大数据量的需求。

基于位置的数据查询

MongoDB 支持二维空间索引，因此可以快速及精确的从指定位置获取数据。

表结构不明确

在一些传统 RDBMS 中，增加一个字段会锁住整个数据库 / 表，或者在执行一个重负载的请求时会明显造成其它请求的性能降级。通常发生在数据表大于 1G 的时候（当大于 1TB 时更甚）。 因 MongoDB 是文档型数据库，为非结构货的文档增加一个新字段是很快速的操作，并且不会影响到已有数据。另外一个好处当业务数据发生变化时，是将不在需要由 DBA 修改表结构。

/usr/local/mongodb/mongodb-3.6.2/bin/mongod --dbpath=/usr/local/mongodb/mongodb-3.6.2/data --logpath=/usr/local/mongodb/mongodb-3.6.2/logs/mongo.log

 

\#! /bin/bash
nohup /usr/local/mongodb/mongodb-3.6.2/bin/mongod -f /usr/local/mongodb/mongodb-3.6.2/mongodb.conf >/dev/null 2>&1 &

nohup mongod --auth --port 27017 --dbpath /data/db --bind_ip 0.0.0.0

db.createUser({ user: 'daiem', pwd: 'daienmin123', roles: [ { role: "readWrite", db: "leanote" } ] });

 



sudo ./bin/mongod --config mongodb.conf

db.help() help on db methods
db.mycoll.help() help on collection methods
sh.help() sharding helpers
rs.help() replica set helpers
help admin administrative help
help connect connecting to a db help
help keys key shortcuts
help misc misc things to know
help mr mapreduce

show dbs show database names
show collections show collections in current database
show users show users in current database
show profile show most recent system.profile entries with time >= 1ms
show logs show the accessible logger names
show log [name] prints out the last segment of log in memory, 'global' is default
use <db_name> set current database
db.foo.find() list objects in collection foo
db.foo.find( { a : 1 } ) list objects in foo where a == 1
it result of the last line evaluated; use to further iterate
DBQuery.shellBatchSize = x set default number of items to display on shell
exit quit the mongo shell

 \# mongodb.conf

\# Where to store the data.
dbpath=/var/lib/mongodb

\#where to log
logpath=/var/log/mongodb/mongodb.log

logappend=true

bind_ip = 127.0.0.1
port = 27017

\# Enable journaling, http://www.mongodb.org/display/DOCS/Journaling
journal=true

\# Enables periodic logging of CPU utilization and I/O wait
\#cpu = true

\# Turn on/off security. Off is currently the default
\#noauth = true
\#auth = true
\# Verbose logging output.
\#verbose = true

\# Inspect all client data for validity on receipt (useful for
\# developing drivers)
\#objcheck = true

\# Enable db quota management
\#quota = true

\# Set oplogging level where n is
\# 0=off (default)
\# 1=W
\# 2=R
\# 3=both
\# 7=W+some reads
\#oplog = 0
\# Diagnostic/debugging option
\#nocursors = true

\# Ignore query hints
\#nohints = true

\# Disable the HTTP interface (Defaults to localhost:27018).
\#nohttpinterface = true

\# Turns off server-side scripting. This will result in greatly limited
\# functionality
\#noscripting = true

\# Turns off table scans. Any query that would do a table scan fails.
\#notablescan = true

\# Disable data file preallocation.
\#noprealloc = true

\# Specify .ns file size for new databases.
\# nssize = <size>
\# Accout token for Mongo monitoring server.
\#mms-token = <token>

\# Server name for Mongo monitoring server.
\#mms-name = <server-name>

\# Ping interval for Mongo monitoring server.
\#mms-interval = <seconds>

\# Replication Options

\# in replicated mongo databases, specify here whether this is a slave or master
\#slave = true
\#source = master.example.com
\# Slave only: specify a single database to replicate
\#only = master.example.com
\# or
\#master = true
\#source = slave.example.com
\# Address of a server to pair with.
\#pairwith = <server:port>
\# Address of arbiter server.
\#arbiter = <server:port>
\# Automatically resync if slave data is stale
\#autoresync
\# Custom size for replication operation log.
\#oplogSize = <MB>
\# Size limit for in-memory storage of op ids.
\#opIdMem = <bytes>

\# SSL options
\# Enable SSL on normal ports
\#sslOnNormalPorts = true
\# SSL Key file and password
\#sslPEMKeyFile = /etc/ssl/mongodb.pem
\#sslPEMKeyPassword = pass