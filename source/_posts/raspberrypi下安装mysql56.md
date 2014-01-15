title: RaspberryPi下编译安装MySQL5.6
date: 2014-01-15 11:30:42
tags: [RaspberryPi, MySQL, Linux]
---

### 遇到的问题还是蛮多的 首先是时间 Cmake编译的时间差不多是12小时左右

```
sudo apt-get install openssl-dev*
sudo apt-get install  libncurses5-dev
sudo cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \ 
-DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci  -DWITH_EXTRA_CHARSETS=all \ 
-DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1  \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \ 
-DWITH_SSL=yes  -DENABLED_LOCAL_INFILE=1
```

### 一开始启动都是正常的 但我做了一些破坏测试以后就有点问题了


```
pi@raspberrypi /usr/local/mysql/data $ ls -lah
total 85M
drwxr-sr-x  5 mysql dba   4.0K Jan 10 15:55 .
drwxr-sr-x 15 root  staff 4.0K Jan 10 06:09 ..
-rw-rw----  1 mysql dba     56 Jan 10 15:55 auto.cnf
-rw-rw----  1 mysql dba    74M Jan 10 15:55 ibdata1
-rw-rw----  1 mysql dba   5.0M Jan 10 15:55 ib_logfile0
-rw-rw----  1 mysql dba   5.0M Jan 10 15:43 ib_logfile1
drwx--S---  2 mysql dba   4.0K Jan 10 15:54 mysql
drwx--S---  2 mysql dba   4.0K Jan 10 15:54 performance_schema
drwxr-sr-x  2 mysql dba   4.0K Jan 10 02:27 test
```

5.6开始 Innodb的文件你如果手动删除的话 有几张表是不会重建的 这个比较麻烦 我是重新install初始化DB以后恢复的

#### Mysql 5.6 Can't find messagefile '/usr/share/mysql/errmsg.sys'解决方法

`sudo cp share/english/errmsg.sys /usr/share/mysql/`

### 测试阶段 参数还没设置太多 个别和内存有关的参数要小心 不要设置太大

```
[mysqld]

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
 basedir = /usr/local/mysql
 datadir = /usr/local/mysql/data
 port = 3306
 server_id = 1
 socket = /tmp/mysql.sock
 log-error = /usr/local/mysql/log/alert.log

 explicit_defaults_for_timestamp=true

#-- Innodb
innodb_data_home_dir = /usr/local/mysql/data
innodb_data_file_path = ibdata1:10M:autoextend
innodb_log_group_home_dir = /usr/local/mysql/data

 innodb_buffer_pool_size = 12M
# innodb_additional_mem_pool_size = 2M

innodb_log_file_size = 5M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50
```


