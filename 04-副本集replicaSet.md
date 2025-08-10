# 04-副本集replicaSet

> mongodb高可用架构

https://www.mongodb.com/docs/manual/tutorial/deploy-replica-set/

复制是跨多个服务器同步数据的过程。

复制提供冗余，并通过不同数据库服务器上的多个数据副本增加数据可用性。

复制保护数据库免受单个服务器的丢失。

复制还允许从硬件故障和服务中断中恢复。

使用其他数据副本，可以将其专用于灾难恢复，报告或备份。

# 为什么复制？

- 保持数据安全
- 数据的高可用性(24 * 7)
- 灾难恢复
- 维护无停机(如备份，索引重建，压缩)
- 读取缩放(额外的副本可读)
- 副本集对应用程序是透明的

# 副本集角色

```
主节点、读写
副本节点、同步主节点数据
仲裁节点、非必须、不存储数据，不竞主，只投票
```

# 什么是副本集

MongoDB通过使用副本集来实现复制。

副本集是托管相同数据集的一组 mongod 实例。

在一个副本中，一个节点是接收所有写操作的主节点。

所有其他实例(例如辅助节点)都应用主节点的操作，以便它们具有相同的数据集。

副本集可以只有一个主节点。

- 副本集是一组两个或多个节点(通常最少需要`3`个节点)。
- 在副本集中，一个节点是主节点，其余节点是次要节点。
- 所有数据从主节点复制到辅助节点。
- 在自动故障切换或维护时，选择为主节点建立，并选择新的主节点。
- 恢复故障节点后，它再次加入副本集，并作为辅助节点。

显示了MongoDB复制的典型图，客户端应用程序始终与主节点进行交互，然后主节点将数据复制到辅助节点。

![image-20221017163522651](http://book.bikongge.com/sre/2024-linux/image-20221017163522651.png)

# 部署副本集

```
mkdir -p /opt/mongo_2801{7,8,9}/{conf,log,pid}  
mkdir -p /data/mongo_2801{7,8,9}

cat >/opt/mongo_28017/conf/mongodb.conf <<EOF
systemLog:
 destination: file   
 logAppend: true  
 path: /opt/mongo_28017/log/mongodb.log

storage:
 journal:
   enabled: true
 dbPath: /data/mongo_28017
 directoryPerDB: true
 wiredTiger:
    engineConfig:
       cacheSizeGB: 0.5 
       directoryForIndexes: true
    collectionConfig:
       blockCompressor: zlib
    indexConfig:
       prefixCompression: true

processManagement:
 fork: true
 pidFilePath: /opt/mongo_28017/pid/mongod.pid

net:
 port: 28017
 bindIp: 127.0.0.1,10.0.0.17

replication: # 开启主从复制
   oplogSizeMB: 1024  # 类似binlog日志的大小单位，1GB
   replSetName: dba   # 副本集名
EOF

# 复制3个实例，区分端口号
cp /opt/mongo_28017/conf/mongodb.conf /opt/mongo_28018/conf/
cp /opt/mongo_28017/conf/mongodb.conf /opt/mongo_28019/conf/

# 启动3个mongodb实例
sed -i 's#28017#28018#g' /opt/mongo_28018/conf/mongodb.conf
sed -i 's#28017#28019#g' /opt/mongo_28019/conf/mongodb.conf
mongod -f /opt/mongo_28017/conf/mongodb.conf
mongod -f /opt/mongo_28018/conf/mongodb.conf
mongod -f /opt/mongo_28019/conf/mongodb.conf


# 检查3个实例
ps -ef|grep mongo
netstat -lntup|grep mongo
```

# 登录mongo副本集

```
1. 修改配置文件，以副本集模式启动多实例

2. 初始化副本集 28017 作为主节点
https://www.mongodb.com/docs/manual/tutorial/deploy-replica-set/#initiate-the-replica-set


mongo --port 28017

# 只需要在一个节点执行，其他节点自动加入

rs.initiate(
   {
      _id: "dba",
      version: 1,
      members: [
         { _id: 0, host : "10.0.0.17:28017" },
         { _id: 1, host : "10.0.0.17:28018" },
         { _id: 2, host : "10.0.0.17:28019" }
      ]
   }
)
```

![image-20221017165420685](http://book.bikongge.com/sre/2024-linux/image-20221017165420685.png)

```
该方法是官网推荐的模式，存在选举时间
```

查看副本集状态

```
rs.status()
# rs是管理集群的命令
```

![image-20221017165648697](http://book.bikongge.com/sre/2024-linux/image-20221017165648697.png)

## 从库开启slve

```
不开启会提示报错，读的权限都没用，slave状态不正确

dba:SECONDARY> show dbs;
2022-10-17T17:11:47.201+0800 E  QUERY    [js] uncaught exception: Error: listDatabases failed:{
    "operationTime" : Timestamp(1665997900, 1),
    "ok" : 0,
    "errmsg" : "not master and slaveOk=false",
    "code" : 13435,
    "codeName" : "NotPrimaryNoSecondaryOk",
    "$clusterTime" : {
        "clusterTime" : Timestamp(1665997900, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs/<@src/mongo/shell/mongo.js:147:19
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:99:12
shellHelper.show@src/mongo/shell/utils.js:906:13
shellHelper@src/mongo/shell/utils.js:790:15
@(shellhelp2):1:1
dba:SECONDARY>
```

开启方式

```
echo "rs.slaveOk()" > ~/.mongorc.js
```

## 测试副本集

主库写入数据

```
dba:PRIMARY> db.myblog.insertOne({"website":"www.yuchaoit.cn"})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("634d1c065705b0c66b18384a")
}
dba:PRIMARY> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
dba:PRIMARY> db.myblog.find()
{ "_id" : ObjectId("634d1c065705b0c66b18384a"), "website" : "www.yuchaoit.cn" }
dba:PRIMARY>
```

从库只读权限

![image-20221017171522450](http://book.bikongge.com/sre/2024-linux/image-20221017171522450.png)

## 副本集常用命令

```
rs.config()
rs.status()
rs.isMaster()
dba:PRIMARY> rs.printReplicationInfo()
configured oplog size:   1024MB
log length start to end: 1430secs (0.4hrs)
oplog first event time:  Mon Oct 17 2022 16:53:20 GMT+0800 (CST)
oplog last event time:   Mon Oct 17 2022 17:17:10 GMT+0800 (CST)
now:                     Mon Oct 17 2022 17:17:13 GMT+0800 (CST)
dba:PRIMARY> 

查看从库信息
dba:PRIMARY> rs.printSlaveReplicationInfo()
WARNING: printSlaveReplicationInfo is deprecated and may be removed in the next major release. Please use printSecondaryReplicationInfo instead.
source: 10.0.0.17:28018
    syncedTo: Mon Oct 17 2022 17:17:20 GMT+0800 (CST)
    0 secs (0 hrs) behind the primary 
source: 10.0.0.17:28019
    syncedTo: Mon Oct 17 2022 17:17:20 GMT+0800 (CST)
    0 secs (0 hrs) behind the primary 
dba:PRIMARY> 

# 最新命令

dba:PRIMARY> rs.printSecondaryReplicationInfo()
source: 10.0.0.17:28018
    syncedTo: Mon Oct 17 2022 17:17:37 GMT+0800 (CST)
    0 secs (0 hrs) behind the primary 
source: 10.0.0.17:28019
    syncedTo: Mon Oct 17 2022 17:17:37 GMT+0800 (CST)
    0 secs (0 hrs) behind the primary 
dba:PRIMARY>
```

# 副本集故障转移

```
1. 关闭主库
[root chaoge-linux /data/mongo_27017]#mongod -f /opt/mongo_28017/conf/mongodb.conf --shutdown
killing process with pid: 9492

## 2.查看副本集信息
rs.status()
rs.config()

3.查看新的复制关系
```

![image-20221017172240152](http://book.bikongge.com/sre/2024-linux/image-20221017172240152.png)

新的主从关系

![image-20221017180117142](http://book.bikongge.com/sre/2024-linux/image-20221017180117142.png)

# 从库权重调整

了解即可

```
myconfig=rs.conf()
myconfig.members[0].priority=100  // 选择第几个成员，设置优先级
rs.reconfig(myconfig) // 重新加载配置

# 主动降级优先级，从主库，降到从库
rs.stepDown()

# 恢复配置
myconfig=rs.conf()
myconfig.members[0].priority=1
rs.reconfig(myconfig)
```

# 

# 增加节点

```
增加节点和删除节点
1.创建新节点
mkdir -p /opt/mongo_28010/{conf,log,pid}  
mkdir -p /data/mongo_28010
cp /opt/mongo_28017/conf/mongodb.conf /opt/mongo_28010/conf/
sed -i 's#28017#28010#g' /opt/mongo_28010/conf/mongodb.conf 
mongod -f /opt/mongo_28010/conf/mongodb.conf
mongo --port 28010

2.集群加入新节点，只能在主库加
rs.add("10.0.0.17:28010")

3.删除节点
rs.remove("10.0.0.17:28010")
```

![image-20221017181922124](http://book.bikongge.com/sre/2024-linux/image-20221017181922124.png)