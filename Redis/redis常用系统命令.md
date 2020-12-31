keys [pattern/*] //返回匹配的key，*代表全部
exists keyName   //查询keyName的key是否存在
del              //删除一个key
expire           //设定一个key的过期时间
persist          //移除给定的过期时间
ttl              //查询一个key的当前过期时间
move             //移动key到其他的数据库 redis有15个库，登录默认为0库
randomkey        //随机返回一个key
rename           //重命名key
type             //返回一个key的数据类型   string list set zset
ping             //查询连通情况
select           //选择一个数据库
quit/exit        //退出redis数据库
dbsize           //返回当前数据库中key的数目
info             //获取服务信息和统计
config get       //实时传输收到的请求
flush            //删除当前库中所有的key



```
实例:
 
        127.0.0.1:6379> keys *              //
         1) "name1"
         2) "name"
         3) "name2"
         4) "id5"
         5) "names:002"
         6) "id2"
         7) "sets3"
         8) "names:001"
         9) "list1"
        10) "id3"
        11) "ganggenulichenggong"
        12) "id1"
        13) "name3"
        14) "names:003"
        15) "sets2"
        16) "sets1"
        127.0.0.1:6379> exists name          //   
        (integer) 1
        127.0.0.1:6379> exists sunsuqing
        (integer) 0
        127.0.0.1:6379> del name
        (integer) 1
        127.0.0.1:6379> expire name1 10
        (integer) 1
        127.0.0.1:6379> select 1
        OK
        127.0.0.1:6379[1]> select 2
        OK
        127.0.0.1:6379[2]> select 0 
        OK
        127.0.0.1:6379> select name 2
        (error) ERR wrong number of arguments for 'select' command
        127.0.0.1:6379> move sets3 5
        (integer) 1
        127.0.0.1:6379> select 5
        OK
        127.0.0.1:6379[5]> get sets3
        (error) WRONGTYPE Operation against a key holding the wrong kind of value
        127.0.0.1:6379[5]> select 0
        OK
        127.0.0.1:6379> keys * 
         1) "name2"
         2) "id5"
         3) "names:002"
         4) "id2"
         5) "names:001"
         6) "list1"
         7) "id3"
         8) "ganggenulichenggong"
         9) "id1"
        10) "name3"
        11) "names:003"
        12) "sets2"
        13) "sets1"
        127.0.0.1:6379> expire name3 300
        (integer) 1
        127.0.0.1:6379> ttl name3
        (integer) 295
        127.0.0.1:6379> ttl name3
        (integer) 294
        127.0.0.1:6379> ttl name3
        (integer) 293
        127.0.0.1:6379> 
        127.0.0.1:6379> 
        127.0.0.1:6379> 
        127.0.0.1:6379> 
        127.0.0.1:6379> ttl name3
        (integer) 290
        127.0.0.1:6379> persist age
        (integer) 0
        127.0.0.1:6379> persist name3
        (integer) 1
        127.0.0.1:6379> ttl name3
        (integer) -1
        127.0.0.1:6379> randomkey
        "sets1"
        127.0.0.1:6379> rename name3 niushao
        OK
        127.0.0.1:6379> get niushao
        "gangge"
        127.0.0.1:6379> type id1
        string
        127.0.0.1:6379> type sets1
        set
        127.0.0.1:6379> type sets2
        set
        127.0.0.1:6379> pint
        (error) ERR unknown command 'pint'
        127.0.0.1:6379> ping
        PONG
        127.0.0.1:6379> dbsize
        (integer) 13
        127.0.0.1:6379> info
        # Server
        redis_version:2.8.20
        redis_git_sha1:00000000
        redis_git_dirty:0
        redis_build_id:2f27060cc08b890d
        redis_mode:standalone
        os:Linux 2.6.32-504.30.3.el6.i686 i686
        arch_bits:32
        multiplexing_api:epoll
        gcc_version:4.4.7
        process_id:11595
        run_id:814f8f123f3c5c05f8d5b3e71e678cc88a8a5852
        tcp_port:6379
        uptime_in_seconds:262996
        uptime_in_days:3
        hz:10
        lru_clock:14053484
        config_file:/usr/local/redis/etc/redis.conf
 
        # Clients
        connected_clients:1
        client_longest_output_list:0
        client_biggest_input_buf:0
        blocked_clients:0
 
        # Memory
        used_memory:635248
        used_memory_human:620.36K
        used_memory_rss:1871872
        used_memory_peak:635248
        used_memory_peak_human:620.36K
        used_memory_lua:23552
        mem_fragmentation_ratio:2.95
        mem_allocator:jemalloc-3.6.0
 
        # Persistence
        loading:0
        rdb_changes_since_last_save:5
        rdb_bgsave_in_progress:0
        rdb_last_save_time:1490448253
        rdb_last_bgsave_status:ok
        rdb_last_bgsave_time_sec:0
        rdb_current_bgsave_time_sec:-1
        aof_enabled:0
        aof_rewrite_in_progress:0
        aof_rewrite_scheduled:0
        aof_last_rewrite_time_sec:-1
        aof_current_rewrite_time_sec:-1
        aof_last_bgrewrite_status:ok
        aof_last_write_status:ok
 
        # Stats
        total_connections_received:4
        total_commands_processed:187
        instantaneous_ops_per_sec:0
        total_net_input_bytes:7761
        total_net_output_bytes:4184
        instantaneous_input_kbps:0.00
        instantaneous_output_kbps:0.00
        rejected_connections:0
        sync_full:0
        sync_partial_ok:0
        sync_partial_err:0
        expired_keys:2
        evicted_keys:0
        keyspace_hits:95
        keyspace_misses:15
        pubsub_channels:0
        pubsub_patterns:0
        latest_fork_usec:139
 
        # Replication
        role:master
        connected_slaves:0
        master_repl_offset:0
        repl_backlog_active:0
        repl_backlog_size:1048576
        repl_backlog_first_byte_offset:0
        repl_backlog_histlen:0
 
        # CPU
        used_cpu_sys:147.73
        used_cpu_user:74.83
        used_cpu_sys_children:0.00
        used_cpu_user_children:0.00
 
        # Keyspace
        db0:keys=13,expires=0,avg_ttl=0
        db5:keys=1,expires=0,avg_ttl=0
        127.0.0.1:6379> config get *
          1) "dbfilename"
          2) "dump.rdb"
          3) "requirepass"
          4) ""
          5) "masterauth"
          6) ""
          7) "unixsocket"
          8) ""
          9) "logfile"
         10) ""
         11) "pidfile"
         12) "/var/run/redis.pid"
         13) "maxmemory"
         14) "3221225472"
         15) "maxmemory-samples"
         16) "3"
         17) "timeout"
         18) "0"
         19) "tcp-keepalive"
         20) "0"
         21) "auto-aof-rewrite-percentage"
         22) "100"
         23) "auto-aof-rewrite-min-size"
         24) "67108864"
         25) "hash-max-ziplist-entries"
         26) "512"
         27) "hash-max-ziplist-value"
         28) "64"
         29) "list-max-ziplist-entries"
         30) "512"
         31) "list-max-ziplist-value"
         32) "64"
         33) "set-max-intset-entries"
         34) "512"
         35) "zset-max-ziplist-entries"
         36) "128"
         37) "zset-max-ziplist-value"
         38) "64"
         39) "hll-sparse-max-bytes"
         40) "3000"
         41) "lua-time-limit"
         42) "5000"
         43) "slowlog-log-slower-than"
         44) "10000"
         45) "latency-monitor-threshold"
         46) "0"
         47) "slowlog-max-len"
         48) "128"
         49) "port"
         50) "6379"
         51) "tcp-backlog"
         52) "511"
         53) "databases"
         54) "16"
         55) "repl-ping-slave-period"
         56) "10"
         57) "repl-timeout"
         58) "60"
         59) "repl-backlog-size"
         60) "1048576"
         61) "repl-backlog-ttl"
         62) "3600"
         63) "maxclients"
         64) "10000"
         65) "watchdog-period"
         66) "0"
         67) "slave-priority"
         68) "100"
         69) "min-slaves-to-write"
         70) "0"
         71) "min-slaves-max-lag"
         72) "10"
         73) "hz"
         74) "10"
         75) "repl-diskless-sync-delay"
         76) "5"
         77) "no-appendfsync-on-rewrite"
         78) "no"
         79) "slave-serve-stale-data"
         80) "yes"
         81) "slave-read-only"
         82) "yes"
         83) "stop-writes-on-bgsave-error"
         84) "yes"
         85) "daemonize"
         86) "yes"
         87) "rdbcompression"
         88) "yes"
         89) "rdbchecksum"
         90) "yes"
         91) "activerehashing"
         92) "yes"
         93) "repl-disable-tcp-nodelay"
         94) "no"
         95) "repl-diskless-sync"
         96) "no"
         97) "aof-rewrite-incremental-fsync"
         98) "yes"
         99) "aof-load-truncated"
        100) "yes"
        101) "appendonly"
        102) "no"
        103) "dir"
        104) "/usr/local/redis/bin"
        105) "maxmemory-policy"
        106) "noeviction"
        107) "appendfsync"
        108) "everysec"
        109) "save"
        110) "900 1 300 10 60 10000"
        111) "loglevel"
        112) "notice"
        113) "client-output-buffer-limit"
        114) "normal 0 0 0 slave 268435456 67108864 60 pubsub 33554432 8388608 60"
        115) "unixsocketperm"
        116) "0"
        117) "slaveof"
        118) ""
        119) "notify-keyspace-events"
        120) ""
        121) "bind"
        122) "127.0.0.1"
        127.0.0.1:6379> 
```

