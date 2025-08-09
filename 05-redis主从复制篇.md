# 05-redis主从复制篇

我们知道要避免单点故障，即保证高可用，便需要冗余（副本）方式提供集群服务。

而Redis 提供了主从库模式，以保证数据副本的一致，主从库之间采用的是读写分离的方式。

# 主从复制概述

> 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。
>
> 前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

**主从复制的作用**主要包括：

- **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

- **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

- 负载均衡

  ：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；

  - 尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

- **高可用基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

主从库之间采用的是**读写分离**的方式。

- 读操作：主库、从库都可以接收；
- 写操作：首先到主库执行，然后，主库将写操作同步给从库。

![image-20220809175044394](/ajian/image-20220809175044394.png)

## 主从复制建立

```
准备好3个redis服务器

db-51
db-52
db-53
```

## 快速部署3个redis节点

```
ssh-keygen 
ssh-copy-id root@10.0.0.52
ssh-copy-id root@10.0.0.53

yum install rsync -y
rsync -avz /opt/redis* root@10.0.0.52:/opt/
rsync -avz /opt/redis* root@10.0.0.53:/opt/

# 修改配置文件
sed -i 's/51/52/g' /opt/redis_6379/conf/redis_6379.conf 
sed -i 's/51/53/g' /opt/redis_6379/conf/redis_6379.conf 

# 创建数据目录
mkdir -p /www.yuchaoit.cn/redis/data/

# 启动3个redis数据库
/opt/redis/src/redis-server /opt/redis_6379/conf/redis_6379.conf 

# 检查3个redis
/opt/redis/src/redis-cli ping
/opt/redis/src/redis-cli dbsize





# 主库写入测试数据
for i in {1..10000};do redis-cli set k_${i} v_${i} && echo "key --- ${i} is ok.";done


# 检查
[root@db-51 /www.yuchaoit.cn/redis/data]#redis-cli dbsize
(integer) 10000
```

## 配置主从关系

![image-20220809184004440](/ajian/image-20220809184004440.png)

```
建立主从关系后，master-slave的数据操作就是实时的了
从库执行命令
[root@db-52 /opt/redis_6379/conf]#/opt/redis/src/redis-cli 
127.0.0.1:6379> replicaof 10.0.0.51 6379
OK
127.0.0.1:6379> dbsize
(integer) 10000


[root@db-53 ~]#/opt/redis/src/redis-cli 
127.0.0.1:6379> replicaof 10.0.0.51 6379
OK
127.0.0.1:6379> dbsize
(integer) 10000
127.0.0.1:6379>
```

## 永久配置主从关系

写入配置文件

```
# replicaof <masterip> <masterport>
# 目前主流，新版redis 5 改为了这个参数。
```

## 检查复制关系

### slave

```
[root@db-53 ~]#/opt/redis/src/redis-cli 
127.0.0.1:6379> role
1) "slave"
2) "10.0.0.51"
3) (integer) 6379
4) "connected"
5) (integer) 210



127.0.0.1:6379> role
1) "slave"
2) "10.0.0.51"
3) (integer) 6379
4) "connected"
5) (integer) 280



127.0.0.1:6379> info replication
# Replication
role:slave
master_host:10.0.0.51
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:238
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:e6a27417c0d97271d1ccffae5923158737cef627
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:238
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:238
127.0.0.1:6379>
```

### master

```
127.0.0.1:6379> role
1) "master"
2) (integer) 266
3) 1) 1) "10.0.0.52"
      2) "6379"
      3) "266"
   2) 1) "10.0.0.53"
      2) "6379"
      3) "266"
127.0.0.1:6379> 


127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=10.0.0.52,port=6379,state=online,offset=322,lag=0
slave1:ip=10.0.0.53,port=6379,state=online,offset=322,lag=0
master_replid:e6a27417c0d97271d1ccffae5923158737cef627
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:322
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:322
127.0.0.1:6379>
```

## 试试主从关系

```
# master
# slave
```

![image-20220809184644282](/ajian/image-20220809184644282.png)

## 取消slave身份

```
127.0.0.1:6379> REPLICAOF no one
OK
127.0.0.1:6379> role
1) "master"
2) (integer) 586
3) (empty list or set)
127.0.0.1:6379> dbsize
(integer) 10001
```

![image-20220809184813544](/ajian/image-20220809184813544.png)

## redis主从细节问答

```
1. slave节点只读，不可写
2. slave不会故障转移
3. 主从故障需要人工介入的地方
- 主库的IP地址
- 从节点要重新 REPLICAOF 设置复制角色

4. 从库建立主从关系时，会清空自己的数据，慎重同步的对象。
5.无论是master，slave节点，在进行主从关键建立等大修改的操作时，务必先对持久化数据做好备份。slave也可以有RDB备份。
```

## 主库设置了密码

```
# master主库 redis_6379.conf
requirepass chaoge888


# slave从库必须设置密码，否则无法建立主从管理

以认证的方式连接到master。 如果master中使用了“密码保护”，slave必须交付正确的授权密码，才能连接成功。

“requirepass”配置项指定了当前server的密码。

此配置项中值需要和master机器的“requirepass”保持一致。

# slave配置文件

[root@db-53 ~]#tail -2 /opt/redis_6379/conf/redis_6379.conf 
masterauth chaoge888
replicaof 10.0.0.51 6379
```

![image-20220809190220699](/ajian/image-20220809190220699.png)
