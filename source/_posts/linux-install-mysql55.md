title: Linux Install MySQL5.5
date: 2014-01-07 14:15:49
tags: MySQL
categories: MySQL
---

## First, Install Cmake

```
$tar zxf cmake-2.6.4.tar.gz
$cd cmake-2.6.4
$./bootstrap
$make
$sudo make install
```
## Make MySQL

* [CMake](http://www.cmake.org/)
* [CMake How to Install](http://www.cmake.org/cmake/help/install.html)

<!-- more -->

```
tar zxf mysql-5.5.11.tar.g
cd mysql-5.5.11
CFLAGS="-O3" CXX=gcc
CXXFLAGS="-O3 -felide-constructors -fno-exceptions -fno-rtti"
cmake . -LH|more 
cmake . -DCMAKE_INSTALL_PREFIX=/home/mysql/mysql -DEXTRA_CHARSETS=all
make -j 8 
make install
```

## Step by step

```
groupadd dba
useradd -g dba mysql
cp mysql-5.5.11.tar.gz /home/mysql/
chown -R mysql:dba /home/mysql/mysql-5.5.11.tar.gz

```

```
su - mysql
tar zxf mysql-5.5.11.tar.gz
cd mysql-5.5.11
CFLAGS="-O3" CXX=gcc 
CXXFLAGS="-O3 -felide-constructors -fno-exceptions -fno-rtti"
cmake . -LH|more        
cmake . -DCMAKE_INSTALL_PREFIX=/home/mysql/mysql -DEXTRA_CHARSETS=all
make -j 8 && make install   
```

### my.cnf

```
cd /home/mysql/mysql
su - root
cp mysql/share/mysql/my-medium.cnf /etc/my.cnf
chown -R mysql:dba /etc/my.cnf

su - mysql
vi /etc/my.cnf
basedir = /home/mysql/mysql
datadir = /home/mysql/mysql/data
socket = /home/mysql/mysql/run/mysql.sock
log-error = /home/mysql/mysql/log/alert.log
log_slow_queries = /home/mysql/mysql/log/slow.log
```


### Start MySQL

```
./scripts/mysql_install_db --basedir=/home/mysql/mysql  \
--datadir=/home/mysql/mysql/data --user=mysql --force

./bin/mysqld_safe &
```

[Via original](http://www.orczhou.com/index.php/2011/06/compile-and-install-mysql-5-5-from-source/)
