title: "MYSQL Replication学习总结"
categories: MySQL
month: 12
year: 2011
tags: MySQL
---

>复制的本质：将DDL DML(除查询以外)的动作记录在slave上replay。
##一个slave只能有一个master

###grant： replication slave

###获取快照：
```
flush tables with read lock; show master status; 
tar -xcvf __.tar.gz /data ; scp ; unlock tables；
```

###slave的关键启动项： --skip--slave--start
```
CHANGE MASTER TO
MASTER_HOST = '192.168.200.200',
	    MASTER_PORT = 3306,
	    MASTER_USER = 'rep1',
	    MASTER_PASSWORD = '12345',
	    MASTER_LOG_FILE = 'mysql-bin.000001',
	    MASTER_LOG_POS = 109;

start slave;
```
<!-- more -->

###配置相关：
```
log-slave-updates = 1 #slave上也打开中继日志 
master-connect-retry = 3 # 断开连接重连的时间间隔
read-only = 1 # 只读模式开关
```
###主从同步维护
```
master：
----------flush tables with read lock;
----------show master status;
slave:
----------select master_pos_wait('mysql-bin.00001','975');
-- 这个函数会让slave达到指定的偏移量后返回0 返回-1表示超时
master:
----------unlock tables;
```

###大文本传输[要注意的是master和slave上的这个设置必须完全一样 否则会有误差]
```
show variables like 'max_allowed_packet';
set @@global.max_allowed_packet = 1024*1024*16
```


###多主复制时自增长列冲突的问题。
```
auto_increment_increment = 10
auto_increment_offset = 1
```
#####从1开始 每次增加10
查看 
`show variables like 'auto_inc%';`

###复制进度，当前执行SQL进程时间戳和系统时间之间的差距。 Time 单位是秒 (show processlist \G);

* 复制的模式：基于语句和基于行
* 向后兼容：新版本的MYSQL可以做旧版本MYSQL的slave
* IO线程和SQL线程是独立分开的。
* slave上的复制是串行的，master上的并行更新操作不能在slave上并行进行。

###单独STOP一个进程： STOP slave IO_THREAD;
```
[my.cnf]
sync_binlog = 1 #将二进制日志的内容同步于磁盘之上。
innodb的前提是 innodb_flush_logs_at_trx_commit = 1
```

###如果不想在server崩溃后表被破坏，可以使用innodb，如果确定使用innodb的话，master上要设置相关参数
```
[my.cnf]
innodb_flush_logs_at_trx_commit = 1
innodb_support_xa = 1
innodb_safe_binlog = 1
```

###M-S差距越来越大 IO进程会产生很多中继日志。复制完的sql语句是否立即从中继日志中清除，1表示立即清除
`relay-log-purge=1`


###基于命令的复制就是语句复制 逻辑复制的一种 5.0之前的版本都支持。
* `[好处]`实现比较简单， 节省带宽 上GB的数据查询在日志中只有几个字节的长度。 有binlog生成，而mysqlbinlog是使用命令复制的最佳工具。
* `[劣势]`M-S上执行的系统时间会有不同 比如current_user()的查询会不同，存储过程 触发器在基于命令复制下也会有问题。修改必须是串行的，需要大量特殊代码。

>ROW行复制 是5.1版本独有的,将实际的数据更改记录到二进制日志中。
* [好]正确复制
* [缺陷]日志会变得很大 不能用mysqlbinlog检查日志

###基于行的复制可以高效的复制数据 因为slave不用replay master上的数据查询。

>[比如]UPDATE动作 基于行复制 代价就会很高 逐行修改 每一行的修改都会写入二进制日志中  基于命令就会很小 。

`5.1的MYSQL`会自动切换命令复制和行复制模式，默认是基于命令复制，当探测到当前事件不能使用命令正确复制的时候就切换到行模式下。
可以通过binlog_format控制复制模式。

###基于行的复制事件在二进制日志里很难进行时间点恢复，但不等于不能做到。
```
[files]
mysql-bin.index 每一行包含了二进制日志文件的文件名
mysql-relay-bin.index 中继日志 格式同上。
master.info slave连接master的所有信息都在这里 文本文件 一行一条内容。
relat-log.info slave上当前的二进制日志和中继日志的坐标
问题：这些过程都不是同步的 过程中失去电力或其他原因 数据将会不完整。

[标准设置]
log_bin = mysql-bin #safe的 不要以主机名来命名二进制日志文件
log_bin_index = mysql-bin.index
relay_log = mysql-relay-bin
relay_log_index = mysql-relay-bin.index

log_slave_updates = 1 将一台slave变成master
serverID是为了防止无限循环而成的 建议将master的ID设置城10 因为1是默认。

复制的过滤设置
控制二进制日志的生成： binlog_do_db binlog_ignore_db 
默认不需要开启  根据自己的需求来设置。比如忽略某一个库的二进制日志生成。

replicate_do_db 复制特定的库
replicate_do_table 复制指定的表
replicate_ignore_db 忽略某个库
replicate_ignore_table 忽略某张表
replicate_rewrite_db 重写某一个库 replicate-rewrite-db="master_db->master_db_foo"
replicate_wild_do_table = mysql.% 通配符下的 所有mysql库的表被复制
replicate_wild_ignore_table 同上  是忽略。

[禁止权限被复制]
replicate_ignore_table = mysql.columns_priv
replicate_ignore_table = mysql.db
replicate_ignore_table = mysql.host
replicate_ignore_table = mysql.procs_priv
replicate_ignore_table = mysql.tables_priv
replicate_ignore_table = mysql.user
```
##被动模式下的主主复制 ： 容错 高可用性
>google的补丁： code.google.com/p/mysql-master-master

分发master 提供master上的binlog

ignore要慎用 比如ignore一个test库 但下面的操作就会麻烦
```
USE TEST;
UPDATE bbs.emp SET uname = 'aaa' where uid = 10;
这个update的动作就会被省略。
```
如果带宽有限制：slave上激活 slave_compressed_protocol 文本在传输时就会压缩 代价是消耗CPU。
