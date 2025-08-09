# 08-redis自动化安装

# 1.ruby脚本自动化安装

```
1.安装ruby开发环境
yum install rubygems -y

2.通过ruby包管理工具，安装操作redis的模块

gem sources --remove https://rubygems.org/
gem sources --remove http://mirrors.aliyun.com/rubygems/

gem sources -a https://mirrors.cloud.tencent.com/rubygems/

[root@db-51 ~]#gem sources -l
*** CURRENT SOURCES ***

https://mirrors.cloud.tencent.com/rubygems/

[root@db-51 ~]#gem install redis -v 3.3.3
Fetching: redis-3.3.3.gem (100%)
Successfully installed redis-3.3.3
Parsing documentation for redis-3.3.3
Installing ri documentation for redis-3.3.3
1 gem installed


2.清空redis环境
redis-cli -c -h 10.0.0.51 -p 6380 flushall
redis-cli -c -h 10.0.0.52 -p 6380 flushall
redis-cli -c -h 10.0.0.53 -p 6380 flushall

redis-cli  -h 10.0.0.51 -p 6380 cluster reset
redis-cli  -h 10.0.0.52 -p 6380 cluster reset
redis-cli  -h 10.0.0.53 -p 6380 cluster reset

redis-cli  -h 10.0.0.51 -p 6381 cluster reset
redis-cli  -h 10.0.0.52 -p 6381 cluster reset
redis-cli  -h 10.0.0.53 -p 6381 cluster reset

3.试试还能用吗redis-cluster
[root@db-51 ~]#redis-cli -c  -h 10.0.0.51 -p 6380 
10.0.0.51:6380> cluster info
cluster_state:fail

[root@db-51 ~]#redis-cli -c  -h 10.0.0.51 -p 6380 
10.0.0.51:6380> set name yuchao
(error) CLUSTERDOWN Hash slot not served




4.一键自动化部署redis集群
[root@db-51 ~]#
[root@db-51 ~]#/opt/redis/src/redis-trib.rb create --replicas 1 10.0.0.51:6380 10.0.0.52:6380 10.0.0.53:6380 10.0.0.51:6381 10.0.0.52:6381 10.0.0.53:6381
WARNING: redis-trib.rb is not longer available!
You should use redis-cli instead.

All commands and features belonging to redis-trib.rb have been moved
to redis-cli.
In order to use them you should call redis-cli with the --cluster
option followed by the subcommand name, arguments and options.

Use the following syntax:
redis-cli --cluster SUBCOMMAND [ARGUMENTS] [OPTIONS]

Example:
redis-cli --cluster create 10.0.0.51:6380 10.0.0.52:6380 10.0.0.53:6380 10.0.0.51:6381 10.0.0.52:6381 10.0.0.53:6381 --cluster-replicas 1

To get help about all subcommands, type:
redis-cli --cluster help


5.发现命令更新了，每一个节点自动分配一个slave
redis-cli --cluster create 10.0.0.51:6380 10.0.0.52:6380 10.0.0.53:6380 10.0.0.51:6381 10.0.0.52:6381 10.0.0.53:6381 --cluster-replicas 1
```

![image-20220812164503332](/ajian/image-20220812164503332.png)

## 使用集群测试

```
[root@db-51 ~]#redis-cli -c -h 10.0.0.51 -p 6381
10.0.0.51:6381> dbsize
(integer) 0
10.0.0.51:6381> set name www.yuchaoit.cn
-> Redirected to slot [5798] located at 10.0.0.52:6380
OK
10.0.0.52:6380> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:4
cluster_stats_messages_ping_sent:3340
cluster_stats_messages_pong_sent:3224
cluster_stats_messages_meet_sent:5
cluster_stats_messages_sent:6569
cluster_stats_messages_ping_received:3224
cluster_stats_messages_pong_received:3342
cluster_stats_messages_received:6566
10.0.0.52:6380>
```
