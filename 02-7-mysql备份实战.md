# 02-7-mysql备份实战

# 1.备份恢复演练(mysqldump+binlog)

## 知识储备

如下内容。。

## 全量备份

全量数据，指的是某一整个数据库（如kings）中所有的表、以及表数据，进行备份。

例如备份所有数据库、以及所有数据，上面也讲了mysqldump的全量备份操作。

```
备份所有库
mysqldump -uroot -pwww.yuchaoit.cn -S /data/3306/mysql.sock -F -A -B |gzip >/server/backup/mysqlbak_$(date+%F).sql.gz

备份某个库
mysqldump -uroot -pwww.yuchaoit.cn -S /data/3306/mysql.sock -F -B oldboy|gzip >/server/backup/mysqlbak_$(date+%F).sql.gz


-F 备份前，刷新binlog日志，用于增量恢复
-B 备份指定的某些数据库
-A 备份所有库、表、数据
```

## 图解增量备份

![image-20220724152841392](http://book.bikongge.com/sre/2024-linux/image-20220724152841392.png)

## binlog

binlog是mysql一大重点，Binlog是一个二进制格式的文件，用于`记录用户对数据库更新的SQL语句信息`，例如`更改数据库库表`和`更改表内容的SQL语句`都会记录到binlog里，但是对库表等内容的查询则不会记录到日志中。

## binlog的作用

当有数据写入到数据库时，还会同时把`更新的SQL语句`写入到对应的binlog文件里，这个文件就是上文所说的binlog文件。

使用mysqldump备份时，例如我们一般会写crontab，例如夜里0点整，进行数据库备份。

## 使用binlog的背景

问题是，每天只有到0点整才进行备份，那未备份之前，也就是两次数据库备份的间隔是24小时。

一旦在这个期间发生故障，那么数据此时就是丢失的，即使使用mysqldump的备份，也只能找回当日0点的数据。

> 使用binlog功能，可以解决该问题

使用binlog文件，可以将两次完整备份间隔之间的数据还原

因为binlog文件的数据就是，写入数据库，的数据

因此可以使用binlog来恢复数据，这种方式称之为二进制增量数据恢复

### 切割binlog图解

切割binlog作用是很有必要的

因为要确定，全量备份mysqldump和增量备份binlog的一个临界点。

当全量备份完成之后，上一次的binlog文件就没有作用了（因为此时的mysqldump全量备份，导出的数据已经包含了上一次的binlog数据）

![image-20220724154236496](http://book.bikongge.com/sre/2024-linux/image-20220724154236496.png)

下一次的全量备份中间的数据依然很重要，就还得存储在binlog文件里

因此需要要求，mysqldump的全量备份数据，和binlog的数据确定临界点

- binlog不能和mysqldump重复
- binlog和mysqldump也不能丢失

## mysqldump的参数

### -F参数切割binlog日志

> -F 参数用于mysqldump全量备份后立即对binlog日志文件切割，生成一个新日志文件，且重新记录binlog日志，用于将来增量恢复，从新的binlog日志文件开始
>
> 利用-F能够立即切割出新的binlog文件

```
mysqldump -uroot -pyuchao7777   -F -B kings|gzip > /data/3307/$(date +%F).sql.gz
```

后面超哥会带着大家实际进行数据故障恢复操作。

### --master-data参数

刚才的-F参数，是给mysqldump提供的切割binlog，但是这也需要不断的执行，不断的切割。`

mysqldump也提供了`--master-data`参数，能够在备份的SQL文件中，添加`CHANGE MASTER`语句，以及`binlog`文件的`pos`位置，也就是记录数据的写入位置。

参数解释

```
--master-data[=value]
该选项将二进制日志的位置和文件名写入到输出中。该选项要求有RELOAD权限，并且必须启用二进制日志。

如果该选项值等于1，位置和文件名被写入CHANGE MASTER语句形式的转储输出，如果你使用该SQL转储主服务器以设置从服务器，从服务器从主服务器二进制日志的正确位置开始。

如果选项值等于2，CHANGE MASTER语句被写成SQL注释。
```

测试执行效果

```
[root@mysql-server56 3307]# mysqldump -uroot -pyuchao7777 -P3307 -h127.0.0.1 --master-data=1  kings --compact  > /data/3307/master-data.sql
Warning: Using a password on the command line interface can be insecure.
[root@mysql-server56 3307]#
[root@mysql-server56 3307]# mysqldump -uroot -pyuchao7777 -P3307 -h127.0.0.1 --master-data=2  kings --compact  > /data/3307/master-data.sql2
Warning: Using a password on the command line interface can be insecure.
[root@mysql-server56 3307]#
[root@mysql-server56 3307]#
[root@mysql-server56 3307]#
[root@mysql-server56 3307]#
[root@mysql-server56 3307]# ls /data/3307/master-data.sql*
/data/3307/master-data.sql  /data/3307/master-data.sql2
```

查看备份的结果文件

```
[root@mysql-server56 3307]# head -1  /data/3307/master-data.sql
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=120;


[root@mysql-server56 3307]# head -1  /data/3307/master-data.sql2
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=120;
```

### -x参数

既然是数据备份，比如说淘宝网的数据库要进行全量备份，但是例如在24:00整点，还有人在写入数据，那就无法保证数据的一致性。

因此得在备份时，将表锁住，防止数据写入，得到一个完整的数据备份，也就是24:00停止所有写入操作。

![image-20210420221017351](http://book.bikongge.com/sre/2024-linux/image-20210420221017351.png)

# 2.实战全流程

## 2.1数据创建

```
# 1.数据准备
[root@db-51 ~]#mysql -uroot -pwww.yuchaoit.cn
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 18
Server version: 5.7.28-log MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database yc_linux charset utf8mb4;
Query OK, 1 row affected (0.01 sec)

mysql> use yc_linux;
Database changed
mysql> create table t1(id int);
Query OK, 0 rows affected (0.00 sec)

mysql> insert into t1 values(1),(2),(3);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> commit ; 
Query OK, 0 rows affected (0.00 sec)

mysql> show master status;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000001 |     1901 |              |                  | ae068f82-06bc-11ed-839d-000c29b76f3a:1-8 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)


# 其实也会自动提交的
```

## 2.2 每周一全量备份

```
crontab -e
0 0 * * 1 备份脚本


# 备份命令
# --single-transaction，给所有数据库加锁，防止数据写入，导致备份错误
# --master-data 将binlog的信息以注释形式备份
# -R 导出mysql自定义函数
# -E 导出events事件 
# --triggers 导出所有数据表的触发器
mysqldump -uroot -pwww.yuchaoit.cn -A --master-data=2 --single-transaction --max_allowed_packet=64M -R -E  > /mysql_3306_backup/full_db_$(date +%F).sql
```

获取GTID事务信息，获取起点

```
[root@db-51 ~]#grep 'SET @@GLOBAL.GTID_PURGED=' /mysql_3306_backup/full_db_2022-02-24.sql 
SET @@GLOBAL.GTID_PURGED='ae068f82-06bc-11ed-839d-000c29b76f3a:1-8';
```

查看pos值，备份开始时，binlog位置点信息

```
 29 
  30 -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1901;
  31 
  32 --
```

## 2.3 模拟次日数据变化

```
查看当前gtid信息
[root@db-51 ~]#mysql -uroot -pwww.yuchaoit.cn
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 20
Server version: 5.7.28-log MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show master status;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000001 |     1901 |              |                  | ae068f82-06bc-11ed-839d-000c29b76f3a:1-8 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)

mysql> 

use yc_linux;
create table t2(id int);
insert into t2 values(66),(77),(88);
commit ;


# 模拟次日下午有人误删除了数据库
drop database yc_linux;
```

## 2.4 恢复全流程（重要）

### 日志截取

```
恢复周一全量备份的数据先
# 检查全备数据
[root@db-51 ~]#ls /mysql_3306_backup/full_db_2022-02-24.sql 
/mysql_3306_backup/full_db_2022-02-24.sql

# 查看gtid的分割点
# 数据是从这里开始变化的
[root@db-51 ~]#grep 'SET @@GLOBAL.GTID_PURGED=' /mysql_3306_backup/full_db_2022-02-24.sql 
SET @@GLOBAL.GTID_PURGED='ae068f82-06bc-11ed-839d-000c29b76f3a:1-8';


# 查看备份开始时，binlog的位置点
29 
30 -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1901;
31 
32 --


# 截取日志
# 起点
# 从全量开始的下一个gtid，或者基于1901的pos值都可以
mysql-bin.000001
ae068f82-06bc-11ed-839d-000c29b76f3a:1-9


# 终点，删库的操作
# 由于有了gtid的记录
# 直接拿到删库的gtid事务id
[root@db-51 ~]#mysql -uroot -pwww.yuchaoit.cn -e "show binlog events in 'mysql-bin.000001'" |grep -B 1 "drop database yc_linux"
mysql: [Warning] Using a password on the command line interface can be insecure.
mysql-bin.000001    2342    Gtid    51    2407    SET @@SESSION.GTID_NEXT= 'ae068f82-06bc-11ed-839d-000c29b76f3a:11'
mysql-bin.000001    2407    Query    51    2511    drop database yc_linux

# 日志截取（核心点了）
# 拿上一个记录即可，也就是删数据的前一个事务
# 恢复gtid的日志范围
# 也就是拿到了全量备份，次日的数据变化记录（截止到drop database 之前）
mysqlbinlog --skip-gtids --include-gtids='ae068f82-06bc-11ed-839d-000c29b76f3a:9-10'  /www.yuchaoit.cn/mysql_3306/logs/mysql-bin.000001 > /mysql_3306_backup/restore_shizhan.sql
```

### 数据恢复

```
# 1.关闭binlog，恢复无须记录
# 先恢复全量的数据
# 再恢复增量的binlog
# 结果就是查看t2会恢复吗，这是次日增量写入的数据

set sql_log_bin=0;
show tables from yc_linux;
source /mysql_3306_backup/full_db_2022-02-24.sql ;
source /mysql_3306_backup/restore_shizhan.sql ;
set sql_log_bin=1;
```

### 查看结果

> 牛啊牛！！！
>
> 为自己鼓掌！！啪啪啪

```sql
mysql> use yc_linux;
Database changed
mysql> show tables;
+--------------------+
| Tables_in_yc_linux |
+--------------------+
| t1                 |
| t2                 |
+--------------------+
2 rows in set (0.00 sec)

mysql> select * from t1;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

mysql> select * from t2;
+------+
| id   |
+------+
|   66 |
|   77 |
|   88 |
+------+
3 rows in set (0.01 sec)
```

> 至此，完成了，全量备份+增量备份，很大程度上，保证了数据恢复的安全性。