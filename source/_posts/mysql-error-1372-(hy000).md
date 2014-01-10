title: MYSQL ERROR 1372 (HY000)
date: 2014-01-09 10:46:42
tags: MySQL
---

```
GRANT ALL PRIVILEGES ON KETTLE_DEMO.* TO 'dba'@'%' IDENTIFIED BY PASSWORD 'dba';
ERROR 1372 (HY000): Password hash should be a 41-digit hexadecimal number
```

### fix

```
mysql> select password('dba');
+-------------------------------------------+
| password('dba')                           |
+-------------------------------------------+
| *** |
+-------------------------------------------+
1 row in set (0.00 sec)

GRANT ALL PRIVILEGES ON KETTLE_DEMO.* TO 'dba'@'%' IDENTIFIED BY PASSWORD '***';
Query OK, 0 rows affected (0.00 sec)
```
