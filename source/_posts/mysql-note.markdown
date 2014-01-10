title: "MySQL Note"
date: 2010-05-23 23:43
comments: true
categories: MySQL
tags: MySQL
---

##Select查询细节：
###【匹配多个字符】
`SELECT __ FROM __ WHERE ___ LIKE 'Smith% Corp.'`
####Smith Corp. Smithson Corp ............  %通配符可以匹配0个或多个字符
###【匹配单个字符】
`SELECT __ FROM __ WHERE ___ LIKE 'Smiths_n Corp.'`
####Smithson Smithsen
###【以上模式的组合使用】
####'Smiths_n %'
###【转义字符】
比如搜索“A%BC”开始的产品   `'A$%BC%' ESCAPE '$'`
###【判断null】
不能使用=来判断 IS NULL  IS NOT NULL
###【NOT判断】
`AND NOT SALES < 1500 不小于1500的`

###【索引使用情况】
* `Handler_read_key` 如果索引正在工作 这个值会很高
* `Handler_read_rnd_next` 值如果高 证明查询运行低效
<!-- more -->
###【索引缓存优化】
* `Key_read_requests` 从缓存中读取索引的次数
* `Key_reads` 从硬盘中读取索引的请求次数
* [my.cnf] `key_buffer_size = 256M` 只缓存索引键
或者直接修改 `SET GLOBAL key_buffer_size = 256M`

###【表缓存优化】
* `Open_tables`  当前打开表的缓存数 flush tables会关闭一些当前没有使用的表缓存
* `Opened_file`s 曾经打开过的表缓存数 flush tables 不会改变大小
* `[my.cnf]  table_open_cache = 128` #这个值与系统的`max_connections`有关
* `SET GLOBAL table_open_cache = 12`8

###【innodb的优化设置】
* `innodb_buffer_pool_size` #同时为数据块和索引块做缓存 和oracle一样 值越高 IO访问就越少 建议物理内存的80%
* `innodb_buffer_pool_size = 64M`
* innodb_flush_log_at_trx_commit  

控制缓冲区中的数据写入到日志文件以及日志文件刷新磁盘的时机。
* 0的时候 每秒一次的被写入到日志文件,并且对日志文件做向磁盘刷新的操作 但是一个事务提交不做任何操作。崩溃的时候数据库会丢失没有写到日志文件中的事务。最多丢失一秒钟的事务。最不安全 效率最高。
* 1的时候 每个事务提交的时候 日志缓冲被写到日志文件 并且对日志文件做向磁盘刷新的操作。默认。每个事务提交的时候都会从log buffer写到日志文件 实际刷新磁盘 有性能上一定消耗。
* 2的时候 每个事务提交时 日志缓冲被写到日志文件 但不对日志文件做向磁盘刷新的操作。没有刷新磁盘，但已经写入到日志文件，只要操作系统没有崩溃，那么并没有丢失数据，比0安全。
* `innodb_additional_mem_pool_size`  状态值怎么查？ status
* Innodb引擎用来储存引擎数据库结构和其他内部数据结构的内存池的大小，默认1M。应用程序里的表越多就应该分配越多的内存，如果innodb用光了这个内存就会向系统内存要。并且写入警告日志。不要太大内存 16,足以。
`innodb_lock_wait_timeout`

自动监测行锁导致的死锁并进行相应的处理，但是不能对表锁导致的死锁进行自动监测。参数主要是被用于在出现类似情况的时候等待指定的时间后回滚，默认是50秒。
`innodb_support_xa`
是否支持分布式事务 默认是1或ON 如果应用中不使用分布式事务就关闭该参数 减少磁盘刷新的次数并获得innodb性能。
`innodb_log_buffer_size`
日志缓存的大小 默认的设置在中等强度写入负载以及较短事务的情况下，一般可以满足服务器的性能要求。如果更新操作峰值或者负载较大就应该加大这个值。8-16M即可。
`innodb_log_file_size`
日志组log group中每个日志文件的大小 在高写入负载尤其是大数据集的情况下很重要。这个值越大性能就越高，副作用是恢复时时间会加长。`默认是5M`。 
Javaeye推荐`innodb_log_file_size = 64M`
####需要注意的是 修改完以后要STOP服务 接着删除原来的日志ib_logfile0和ib_logfile1，然后启动服务，看新日志的大小是否为设置的值。
##整体性能分析报告

```
mysql> show engine innodb status \G
innodb_file_per_table
```

取值为ON或者OFF。是否为每个table使用单独的数据文件保存。如果系统中表的个数不多，并且没有超大表，使用该参数可以使得各个表之间的 维护相对独立，有一定的好处。innodb_flush_method究竟应不应该使用O_DIRECT？
####innodb_flush_method究竟应不应该使用O_DIRECT？ 

所有MySQL调优的建议都说，如果硬件没有预读功能，那么使用O_DIRECT将极大降低InnoDB的性能，因为O_DIRECT跳过了操作 系统的文件系统Disk Cache，让MySQL直接读写磁盘了。 

但是在我的实践中来看，如果不使用O_DIRECT，操作系统被迫开辟大量的Disk Cache用于innodb的读写缓存，不但没有提高读写性能，反而造成读写性能急剧下降。而且buffer pool的数据缓存和操作系统Disk Cache缓存造成了Double buffer的浪费，显然从我这个实践来看，浪费得非常厉害。 

说O_DIRECT造成MySQL直接读写磁盘造成得性能下降问题，我觉得完全是杞人忧天。因为从JavaEye的数据库监测来看，Innodb 的buffer pool命中率非常高，有98%以上，真正的磁盘操作是微乎其微的。为了1%的磁盘操作能够得到Disk Cache，而浪费了98%的double buffer内存空间，无论从性能上看，还是从内存资源的消耗来看，都是非常不明智的。 
###【查询缓存 MySQL Query Cache】
* 存储SELECT查村的文本以及相应结果作为缓存。
* `have_query_cache`  是否配置高速缓存
* `query_cache_size` 缓存大小
* `query_cache_type` #0和OFF 缓存关闭； 1或者ON 缓存打开 使用SQL_NO_CACHE的SELECT除外；2或者demand 只有带SQL_CACHE的SELECT语句提供高速缓存。
* `query_cache_limit` 单个查询能够使用的缓存区大小  如果超过就不使用cache了
* `query_cache_min_res_unit` 单个存储块的内存分配大小 Qcache_queries_in_cache和Qcache_total_blocks的比例如果是1:2是正合适的，如果再大就需要调整了。
* `Qcache_free_blocks` 查询缓存中的空闲内存块的数目
* `Qcache_free_memory` 查询缓存中的空闲内存总数
* `Qcache_hits` 缓存采样数数目
* `Qcache_inserts` 被加入到缓存中的查询数目。
* `Qcache_lowmem_prunes` 因为缺少内存而被从缓存中删除的查询数目。
* `Qcache_not_cached` 没有被缓存的查询数目，由query_cache_type带来的。
* `Qcache_queries_in_cache` 在缓存中已注册的查询数目。
* `Qcache_total_blocks` 查询缓存中的块的总数目。

###【mysqld读取my.cnf的顺序】
* 第一搜，首先读取`/etc/my.cnf`，多实例这个配置 文件不会存在。
* 第二搜，`$datadir/my.cnf`，在data目录下寻找此配置文件。
* 第三搜，`defaultfile=/path/my.cnf` 通常写在命令行上，mysqld_safe defaultfile=/tmp/my.cnf &等执行。
* 第四搜，`~/my.cnf` 当前用户下的配置文件。
