title: 为什么用Cmake？
date: 2014-01-09 09:12:19
tags: [Linux,MySQL]
---

>mysql版本5.5以上编译安装时需要用到软件cmake，cmake特性是独立于源码编译，编译工作可以在另外一个目录中而非源码目录中进行，好处是可以保证源码目录不受任何一次编译的影响。

```
[pi@raspi ~]# wget http://www.cmake.org/files/v2.8/cmake-2.8.11.2.tar.gz
[pi@raspi ~]# tar zxvf cmake-2.8.11.2.tar.gz
[pi@raspi ~]# cd  cmake-2.8.11.2
[pi@raspi ~]# ./bootstrap
[pi@raspi ~]# make && make install
```

[Get MySQL5.6.15](http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.15.tar.gz)

``` shell
cmake -DCMAKE_INSTALL_PREFIX=/opt/mysql -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci  -DWITH_EXTRA_CHARSETS=all -DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
-DWITH_SSL=yes  -DENABLED_LOCAL_INFILE=1
```
<!-- more -->
### Setup after Installed

```
chown -R root.mysql /opt/mysql/ 
chown -R mysql.mysql /opt/mysql/data/
```

### Init Database

```
/opt/mysql/scripts/mysql_install_db --user=mysql --basedir=/opt/mysql/ --datadir=/opt/mysql/data/
cp /opt/mysql/support-files/my-default.cnf /etc/my.cnf
```

### Start running

`/opt/mysql/bin/mysqld_safe & `

##cmake编译参数详解：

```
http://dev.mysql.com/doc/refman/5.6/en/source-configuration-options.html
-DCMAKE_INSTALL_PREFIX=dir_name  ：用于指定安装目录
-DWITH_INNOBASE_STORAGE_ENGINE=1
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1  ：常用存储引擎参数设置

-DDEFAULT_CHARSET=charset_name
The server character set. By default, MySQL uses the latin1 (cp1252 West European) character set.
charset_name may be one of binary, armscii8, ascii, big5, cp1250, cp1251, cp1256, 
cp1257, cp850, cp852, cp866, cp932, dec8, eucjpms, euckr, gb2312, gbk, geostd8, greek, 
hebrew, hp8, keybcs2, koi8r, koi8u, latin1, latin2, latin5, latin7, macce, macroman, sjis, 
swe7, tis620, ucs2, ujis, utf8, utf8mb4, utf16, utf16le, utf32. 
The permissible character sets are listed in the cmake/character_sets.cmake file as the value of CHARSETS_AVAILABLE.
-DENABLED_LOCAL_INFILE=bool  ：bool值表示（1表示允许该功能，0表示没有改功能）
 
-DMYSQL_UNIX_ADDR=file_name
The Unix socket file path on which the server listens for socket connections. 
This must be an absolute path name. The default is /tmp/mysql.sock.
```


