# 05-mongodb备份恢复

# 1.mongo状态查看

```
监控及时获得应用的运行状态信息，在问题出现时及时发现。 
监控哪些


CPU、内存、磁盘I/O、应用程序（MongoDB）、进程监控（ps -aux）、错误日志监控
```

## mongo内部状态

```
db.serverStatus()
查看实例运行状态（内存使用、锁、用户连接等信息）　通过比对前后快照进行性能分析
"connections"     # 当前连接到本机处于活动状态的连接数
"activeClients"   # 连接到当前实例处于活动状态的客户端数量
"locks"           # 锁相关参数
"opcounters"      # 启动之后的参数
"opcountersRepl"  # 复制想关
"storageEngine"   # 查看数据库的存储引擎
"mem"             # 内存相关


db.stats()
显示信息说明：
"db" : "test" ,表示当前是针对"test"这个数据库的描述。想要查看其他数据库，可以先运行$ use databasename(e.g  $use admiin).

"collections" : 3,表示当前数据库有多少个collections.可以通过运行show collections查看当前数据库具体有哪些collection.

"objects" : 5，表示当前数据库所有collection总共有多少行数据。显示的数据是一个估计值，并不是非常精确。

"avgObjSize" : 36,表示每行数据是大小，也是估计值，单位是bytes
"dataSize" : 468,表示当前数据库所有数据的总大小，不是指占有磁盘大小。单位是bytes
"storageSize" : 13312,表示当前数据库占有磁盘大小，单位是bytes,因为mongodb有预分配空间机制，为了防止当有大量数据插入时对磁盘的压力,因此会事先多分配磁盘空间。
"numExtents" : 3,似乎没有什么真实意义。我弄明白之后再详细补充说明。
"indexes" : 1 ,表示system.indexes表数据行数。
"indexSize" : 8192,表示索引占有磁盘大小。单位是bytes
 "fileSize" : 201326592，表示当前数据库预分配的文件大小，例如test.0,test.1，不包括test.ns。
```

## mongostat工具

mongostat在功能上类似于UNIX / Linux文件系统实用程序vmstat

```
参数          参数说明
insert      每秒插入量
query       每秒查询量
update      每秒更新量
delete      每秒删除量
conn        当前连接数
qr|qw       客户端查询排队长度（读|写）最好为0，如果有堆积，数据库处理慢。
ar|aw       活跃客户端数量（读|写）
time        当前时间
```

命令

```
[root chaoge-linux ~]#
[root chaoge-linux ~]#mongostat --port 28018 -o vsize,res --humanReadable=false --noheaders -n 1
1849688064 92274688
[root chaoge-linux ~]#mongostat --port 28018 -o vsize,res 
vsize   res
1.72G 88.0M
1.72G 88.0M
1.72G 88.0M
1.72G 88.0M
^C2022-10-18T11:02:35.936+0800    signal 'interrupt' received; forcefully terminating
[root chaoge-linux ~]#

-o vsize,res 只显示这两列
--humanReadable=false 将GB转为kb
--noheaders 不输出首行字段标题
-n 1 输出一次


# 直接执行
# 显示crud的状态，以及内存使用率等
[root chaoge-linux ~/.ssh]#mongostat 
insert query update delete getmore command dirty used flushes vsize   res qrw arw net_in net_out conn                time
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.48G 75.0M 0|0 1|0   166b   38.6k    2 Oct 18 11:08:44.444
    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.48G 75.0M 0|0 1|0   111b   38.4k    2 Oct 18 11:08:45.446
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.48G 75.0M 0|0 1|0   112b   38.5k    2 Oct 18 11:08:46.443
    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.48G 75.0M 0|0 1|0   111b   38.4k    2 Oct 18 11:08:47.444
^C2022-10-18T11:08:47.924+0800    signal 'interrupt' received; forcefully terminating
[root chaoge-linux ~/.ssh]#
```

## 监控测试

```
循环插入数据，在mongo shell里执行
mongo

for (i=0;i<100000;i++){db.users.insert({"id":i,"name":"www.yuchaoit.cn","date":new Date()});}
```

![image-20221018113358942](/ajian/image-20221018113358942.png)

## zabbix自带mongo的监控模板

...

## 集群监控

```
rs.status()

查看health字段
```

# 2.备份恢复

```
工具命令

mongodump/mongorestore 

mongoexport/mongoimport

应用场景


定时备份，全量备份， mongodump/mongorestore  备份格式是bson  压缩格式gzip

分析数据，迁移数据，  mongoexport  mongoimport  备份数据格式是json  或csv
```

## mongodump命令

```
参数                 参数说明
-h                  指明数据库宿主机的IP
-u                  指明数据库的用户名
-p                  指明数据库的密码
-d                  指明数据库的名字
-c                  指明collection的名字
-o                  指明到要导出的文件名
-q                  指明导出数据的过滤条件
--authenticationDatabase  验证数据的名称
--gzip              备份时压缩
```

### 全库备份

```
# 备份实践

mongodump --port 27017 -o /mongoback/mongo_backup

[root chaoge-linux /data/mongo_backup]#mongodump --port 27017 -o /data/mongo_backup
2022-10-18T22:54:18.374+0800    writing admin.system.version to /data/mongo_backup/admin/system.version.bson
2022-10-18T22:54:18.375+0800    done dumping admin.system.version (1 document)
2022-10-18T22:54:18.375+0800    writing test.user to /data/mongo_backup/test/user.bson
2022-10-18T22:54:18.376+0800    done dumping test.user (1 document)
[root chaoge-linux /data/mongo_backup]#
[root chaoge-linux /data/mongo_backup]#tree
.
├── admin
│   ├── system.version.bson
│   └── system.version.metadata.json
└── test
    ├── user.bson
    └── user.metadata.json

2 directories, 4 files
[root chaoge-linux /data/mongo_backup]#

# 数据都导出为bson文件了




# 完整需要账号密码的备份命令

mongodump -h 10.0.0.17:27017 -u yuchao -pwww.yuchaoit.cn --authenticationDatabase admin -o /backup/full_${date}_mongo


# 副本集备份命令
mongodump --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019" -o /backup/full_${date}_mongo
```

### 备份单个库

```
# 如果有账号密码的话，得如此备份
mongodump -h 10.0.0.17:27017 -u yuchao -pwww.yuchaoit.cn --authenticationDatabase admin  -d test -o /data/backup/mongo


mongodump --port 27017 -d test  -o /data/mongo_test_backup

[root chaoge-linux /data]#cd mongo_test_backup/
[root chaoge-linux /data/mongo_test_backup]#tree
.
└── test
    ├── user.bson
    └── user.metadata.json

1 directory, 2 files
[root chaoge-linux /data/mongo_test_backup]#
```

### 备份单个库下的集合

```
mongodump -h 10.0.0.17:27017 -u yuchao -pwww.yuchaoit.cn --authenticationDatabase admin  -d test  -c user  -o /data/backup/mongo
```

### 备份数据且压缩gzip

```
mongodump --port 27017 -o /data/mongo_backup --gzip

[root chaoge-linux /data/mongo_backup]#tree
.
├── admin
│   ├── system.version.bson
│   ├── system.version.bson.gz
│   ├── system.version.metadata.json
│   └── system.version.metadata.json.gz
└── test
    ├── user.bson
    ├── user.bson.gz
    ├── user.metadata.json
    └── user.metadata.json.gz

2 directories, 8 files
[root chaoge-linux /data/mongo_backup]#
```

### bsondump转换json

```
解析bson文件

[root chaoge-linux /data/mongo_backup/test]#bsondump --outFile=user.json user.bson
2022-10-18T23:49:24.050+0800    1 objects found

[root chaoge-linux /data/mongo_backup/test]#cat user.json 
{"_id":{"$oid":"634ebdf4cb5add8e8e9738aa"},"name":"www.yuchaoit.cn"}
```

## mongorestore恢复

### 恢复bson文件

```
[root chaoge-linux ~]#mongorestore --port 27017 /mongoback/mongo_backup/
2022-10-19T00:05:53.824+0800    preparing collections to restore from
2022-10-19T00:05:53.825+0800    reading metadata for test.user from /mongoback/mongo_backup/test/user.metadata.json
2022-10-19T00:05:53.840+0800    restoring test.user from /mongoback/mongo_backup/test/user.bson
2022-10-19T00:05:53.848+0800    no indexes to restore
2022-10-19T00:05:53.848+0800    finished restoring test.user (1 document, 0 failures)
2022-10-19T00:05:53.848+0800    1 document(s) restored successfully. 0 document(s) failed to restore.
[root chaoge-linux ~]#


# 删除的数据又回来了
> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
> use test
switched to db test
> db.user.find()
{ "_id" : ObjectId("634ece2ec58db92fbe044a9c"), "addr" : "北京", "name" : "yuchao" }
>
```

### 恢复gzip格式数据

```
mongorestore --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019"  --gzip
```

### 模拟恢复动作

```
mongorestore --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019"  --gzip --drop --dryRUn

# --drop 遇见重复的数据删除后再导入
```

### 恢复到指定库

```
mongorestore --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019"  --dir=/mongoback/mongo_backup/ -d test --drop --gzip
```

### 恢复到指定集合

```
恢复到集合，必须要求备份的数据是bson格式

mongorestore --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019"  --dir=/mongoback/mongo_backup/ -d test  -c user --drop
```

## mongoexport导出数据

### json格式

```
mongoexport  和mongodump用起来很相似

# 必须指定备份哪个集合的数据，导出为json
mongoexport --port 27017 -d test -c user -o /data/mongo_backup.json

[root chaoge-linux /data/mongo_backup]#cat /data/mongo_backup.json
{"_id":{"$oid":"634ebdf4cb5add8e8e9738aa"},"name":"www.yuchaoit.cn"}
```

### csv格式

```
mongoexport --port 27017 -d test -c user --type=csv --fields=name  -o /data/mongo_backup.csv

[root chaoge-linux /data/mongo_backup]#cat /data/mongo_backup.csv 
name
www.yuchaoit.cn
```

# 3.其他数据导入mongodb

## 1.mysql导出csv数据

```
select * from world.city into outfile '/var/lib/mysql/city.csv' fields terminated by ',';
```

## 2.手动修改csv格式头部

```
ID,Name,CountryCode,District,Population
```

## 3.mongo导入csv

```
mongoimport --host="dba/10.0.0.17:28017,10.0.0.17:28018,10.0.0.17:28019"  --type=csv --headerline -d world -c city /var/lib/mysql/city.csv --drop
```

## 4.csv导入es

```
csv > mongo > json > es
```
