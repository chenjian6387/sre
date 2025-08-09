# 06-redis高可用哨兵篇

```
https://redis.io/docs/manual/sentinel/#sentinels-and-replicas-auto-discovery 
官网资料
```

在上文主从复制的基础上，如果注节点出现故障该怎么办呢？

在 Redis 主从集群中，哨兵机制是实现主从库自动切换的关键机制，它有效地解决了主从复制模式下故障转移的问题。

# 哨兵机制（Redis Sentinel）

> Redis Sentinel，即Redis哨兵，在Redis 2.8版本开始引入。
>
> 哨兵的核心功能是主节点的自动故障转移。

![image-20220810184327608](http://book.bikongge.com/sre/2024-linux/image-20220810184327608.png)

## 哨兵实现了什么

哨兵实现了什么功能呢？下面是Redis官方文档的描述：

- **监控（Monitoring）**：哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）**：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）**：客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。
- **通知（Notification）**：哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移；

而配置提供者和通知功能，则需要在与客户端的交互中才能体现。（查看客户端的配置文件更新）

## 哨兵集群的组建

> 上图中哨兵集群是如何组件的呢？哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制
>
> 也就是发布 / 订阅机制。

在主从集群中，主库上有一个名为`__sentinel__:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信的。

在下图中

- 哨兵 1 把自己的 IP和端口发布到`__sentinel__:hello`频道上
- 哨兵 2 和 3 订阅了该频道。
- 那么此时，哨兵 2 和 3 就可以从这个频道直接获取哨兵 1 的 IP 地址和端口号。
- 然后，哨兵 2、3 可以和哨兵 1 建立网络连接。

### 哨兵之间的发布订阅

![image-20220810190120135](http://book.bikongge.com/sre/2024-linux/image-20220810190120135.png)

```
这样一来，哨兵2与3，也都建立了连接，形成了集群，通过网络通信。
并且商量，master是否挂了，以及如何选举新的slave。
```

### 快速体验，发布、订阅玩法

```
# 订阅频道
127.0.0.1:6379> SUBSCRIBE channel_1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"  消息类型
2) "channel_1"  频道名
3) (integer) 1  成功



1) "message"
2) "channel_1"
3) "www.yuchaoit.cn_666666"
1) "message"
2) "channel_1"
3) "chaoge_linux666"


# 发布消息

127.0.0.1:6379> PUBLISH channel_1  www.yuchaoit.cn_666666
(integer) 0
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> PUBLISH channel_1 chaoge_linux666
(integer) 0
```

![image-20220810185508755](http://book.bikongge.com/sre/2024-linux/image-20220810185508755.png)

## 哨兵如何监控的redis

这是由哨兵向主库发送 INFO 命令来完成的。

就像下图所示，哨兵 2 给主库发送 INFO 命令，主库接受到这个命令后，就会把从库列表返回给哨兵。

接着，哨兵就可以根据从库列表中的连接信息，和每个从库建立连接，并在这个连接上持续地对从库进行监控。

哨兵 1 和 3 可以通过相同的方法和从库建立连接。

![image-20220810190444780](http://book.bikongge.com/sre/2024-linux/image-20220810190444780.png)

## 哨兵如何判定master下线

![image-20220810190831421](http://book.bikongge.com/sre/2024-linux/image-20220810190831421.png)

```
当一个哨兵确认master挂了，给其他哨兵发送 is-master-down-by-addr 命令，命令来询问对方是否认为给定的服务器已下线。

其他哨兵来投票，同意或拒绝。
```

## 怎么才能是新master

哨兵根据如下条件判断

- 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点
- 选择`salve-priority`从节点优先级最高（redis.conf）的
- 选择复制偏移量最大，只复制最完整的从节点.

![image-20220810191419829](http://book.bikongge.com/sre/2024-linux/image-20220810191419829.png)

## 故障如何转移原理

![image-20220810191855323](http://book.bikongge.com/sre/2024-linux/image-20220810191855323.png)

```
1. slave1 执行replicaof no one 
2. sentinel 通知slave2指向新的master
3. old_master成为slave角色，指向new_master
```

# 部署redis哨兵集群

## db-51

```
pkill redis


cat >/opt/redis_6379/conf/redis_6379.conf <<'EOF'
daemonize yes
bind 127.0.0.1 10.0.0.51
port 6379
pidfile /opt/redis_6379/pid/redis_6379.pid
logfile /opt/redis_6379/logs/redis_6379.log
save 900 1
save 300 10
save 60 10000
dbfilename www.yuchaoit.cn_redis_dump.rdb
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
dir /www.yuchaoit.cn/redis/data/
appendfsync everysec
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
aof-use-rdb-preamble yes 
EOF

# 启动
mkdir -p /opt/redis_6379/pid/
redis-server /opt/redis_6379/conf/redis_6379.conf 

# 测试
redis-cli info
```

## db-52和db-53

```
cat >/opt/redis_6379/conf/redis_6379.conf <<EOF
daemonize yes
bind 127.0.0.1 $(ifconfig ens33|awk 'NR==2{print $2}')
port 6379
pidfile /opt/redis_6379/pid/redis_6379.pid
logfile /opt/redis_6379/logs/redis_6379.log
save 900 1
save 300 10
save 60 10000
dbfilename www.yuchaoit.cn_redis_dump.rdb
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
dir /www.yuchaoit.cn/redis/data/
appendfsync everysec
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
aof-use-rdb-preamble yes 
EOF

# 启动
mkdir -p /opt/redis_6379/pid/
redis-server /opt/redis_6379/conf/redis_6379.conf 

# 测试
redis-cli info
```

## 配置主从关系

```
[root@db-51 ~]#redis-cli -h 10.0.0.52 replicaof 10.0.0.51 6379
OK
[root@db-51 ~]#redis-cli -h 10.0.0.53 replicaof 10.0.0.51 6379
OK
[root@db-51 ~]#
[root@db-51 ~]#
[root@db-51 ~]#
[root@db-51 ~]#redis-cli -h 10.0.0.51 info replication 
# Replication
role:master
connected_slaves:2
slave0:ip=10.0.0.53,port=6379,state=online,offset=0,lag=1
slave1:ip=10.0.0.52,port=6379,state=online,offset=0,lag=0
master_replid:a15f89ab55de2c2c2f560dae53ad736a5ee31eda
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:0
```

## 三台机器（部署哨兵）

```
mkdir -p /data/redis_26379
mkdir -p /opt/redis_26379/{conf,pid,logs}

cat > /opt/redis_26379/conf/redis_26379.conf <<EOF
bind $(ifconfig ens33|awk 'NR==2{print $2}')
port 26379
daemonize yes
logfile /opt/redis_26379/logs/redis_26379.log
dir /data/redis_26379
# 监控主节点，地址信息，以及至少需要2个哨兵同意下线
sentinel monitor www.yuchaoit.cn_redis 10.0.0.51 6379 2
# 超过30s没回复认定master下线
sentinel down-after-milliseconds www.yuchaoit.cn_redis 30000
# 当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，
原来的从节点会向新的主节点发起复制操作，限制每次向新的主节点发起复制操作的从节点个数为1
sentinel parallel-syncs www.yuchaoit.cn_redis 1
# 故障转移超时时间为180000毫秒
sentinel failover-timeout www.yuchaoit.cn_redis 180000
EOF

# 授权
useradd redis -M -s /sbin/login
chown -R redis.redis /data/redis*
chown -R redis.redis /opt/redis*


# systemctl服务脚本
cat >/usr/lib/systemd/system/redis-sentinel.service <<EOF
[Unit]
Description=Redis service by www.yuchaoit.cn
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/redis-sentinel /opt/redis_26379/conf/redis_26379.conf --supervised systemd
ExecStop=/usr/local/bin/redis-cli -h $(ifconfig ens33|awk 'NR==2{print $2}') -p 26379 shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
EOF

# 重启
systemctl daemon-reload
systemctl restart redis-sentinel

# 检查
netstat -tunlp|grep 26379

redis-cli -h $(ifconfig ens33|awk 'NR==2{print $2}') -p 26379

# 查看哨兵信息
info sentinel
```

### 哨兵部署结果

![image-20220810200243145](http://book.bikongge.com/sre/2024-linux/image-20220810200243145.png)

### 哨兵注意点

```
1.哨兵发起故障转移的条件是master挂掉，slave挂掉无动作。
2. 哨兵会主动维护redis配置文件，无须手动
3.如果主动关系发生变化，哨兵自动同步配置文件
4.有哨兵后，不再需要手工维护主从关系。
```

### 哨兵常用命令

```
# 查询redis的master节点状态
10.0.0.51:26379> sentinel get-master-addr-by-name www.yuchaoit.cn_redis
1) "10.0.0.51"
2) "6379"

# 列出所有被监视的主服务器，以及这些主服务器的当前状态。
10.0.0.51:26379> sentinel master www.yuchaoit.cn_redis


# 列出给定主服务器的所有从服务器，以及这些从服务器的当前状态。
10.0.0.51:26379> sentinel slaves  www.yuchaoit.cn_redis

# sentinel ckquorum
检测当前可达的Sentinel节点总数是否达到的个数。
例如当quorum=3，而当前可达的Sentinel节点个数为2个，那么将无法进行故障转移。

10.0.0.51:26379> sentinel ckquorum   www.yuchaoit.cn_redis
OK 3 usable Sentinels. Quorum and failover authorization can be reached


# 只能敲打关于sentinel的命令
```

# 测试主从故障自动化转移

```
1. 关闭master节点
2. 查看slave是否选举
3.查看哨兵配置文件
4.查看redis配置文件
5.查看新主节点是否可写入
6.查看新slave是否可以读取，同步
```

## 正常m-s关系

![image-20220810201819388](http://book.bikongge.com/sre/2024-linux/image-20220810201819388.png)

## 故障转移

![image-20220810202017969](http://book.bikongge.com/sre/2024-linux/image-20220810202017969.png)

## 新主从关系

![image-20220810202141842](http://book.bikongge.com/sre/2024-linux/image-20220810202141842.png)

## 查看配置文件

### redis

![image-20220810202322191](http://book.bikongge.com/sre/2024-linux/image-20220810202322191.png)

### sentinel

![image-20220810202707902](http://book.bikongge.com/sre/2024-linux/image-20220810202707902.png)

## 修复old_master

```
[root@db-51 ~]#cat /opt/redis_6379/conf/redis_6379.conf 
daemonize yes
bind 127.0.0.1 10.0.0.51
port 6379
pidfile /opt/redis_6379/pid/redis_6379.pid
logfile /opt/redis_6379/logs/redis_6379.log
save 900 1
save 300 10
save 60 10000
dbfilename www.yuchaoit.cn_redis_dump.rdb
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
dir /www.yuchaoit.cn/redis/data/
appendfsync everysec
appendonly yes
appendfilename "www.yuchaoit.cn_appendonly.aof"
aof-use-rdb-preamble yes
```

### 自动转为slave角色

因为开始的m-s复制关系，sentinle记录三个节点的信息。

一旦启动，自动加入slave角色，无须人为介入了。

![image-20220810203155426](http://book.bikongge.com/sre/2024-linux/image-20220810203155426.png)

# 提升篇

## 程序是如何读取的redis哨兵集群

```python
# 于超老师以python为例，展示 后端如何读写取redis集群

# python代码
[root@db-51 ~]#yum install python3 python3-pip -y
[root@db-51 ~]#pip3 install redis


# 代码测试

from redis.sentinel import Sentinel

# 指定sentinel的集群列表
sentinel_conn=Sentinel([('10.0.0.51',26379),('10.0.0.52',26379),('10.0.0.53',26379)], socket_timeout=0.1)
# 获取redis集群信息
#sentinel.discover_master('www.yuchaoit.cn_redis')
#sentinel.discover_slaves('www.yuchaoit.cn_redis')

# 设置读写分离节点

# 写入节点
master=sentinel_conn.master_for('www.yuchaoit.cn_redis', socket_timeout=0.1)  
# 获取读取节点
slave=sentinel_conn.slave_for('www.yuchaoit.cn_redis', socket_timeout=0.1)

# 写入
print(master.set('new_name','chaoge666'))
print(slave.get('new_name'))
```

![image-20220810204350491](http://book.bikongge.com/sre/2024-linux/image-20220810204350491.png)