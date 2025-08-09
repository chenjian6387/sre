# 04-redis安全篇

redis被攻击，作为突破口，服务器惨遭毒手的事太常见了。

大多数云服务器被攻击，都是redis，mongodb等数据库被入侵。

因此修改端口，密码，以及注意bind运行地址，是必须。

思考是否要暴露redis到公网。

# 1.设置密码、端口

配置

```
[root@db-51 ~]#cat /opt/redis_6379/conf/redis_6379.conf 
daemonize yes
bind 127.0.0.1 10.0.0.51
port 26379
pidfile /opt/redis_6379/pid/redis_6379.pid
logfile /opt/redis_6379/logs/redis_6379.log

dir /www.yuchaoit.cn/redis/data/

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
requirepass chaoge666
```

启动

```
[root@db-51 ~]#redis-server /opt/redis_6379/conf/redis_6379.conf 
[root@db-51 ~]#netstat -tunlp|grep redis
tcp        0      0 10.0.0.51:26379         0.0.0.0:*               LISTEN      2857/redis-server 1 
tcp        0      0 127.0.0.1:26379         0.0.0.0:*               LISTEN      2857/redis-server 1 
[root@db-51 ~]#
```

## 密码验证

玩法1

```
[root@db-51 ~]#redis-cli -h 127.0.0.1 -p 26379
127.0.0.1:26379> ping
(error) NOAUTH Authentication required.
127.0.0.1:26379> auth chaoge666
OK
127.0.0.1:26379> 
127.0.0.1:26379> ping
PONG
127.0.0.1:26379>
```

玩法2

```
[root@db-51 ~]#redis-cli -h 127.0.0.1 -p 26379 -a 'chaoge666' dbsize
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(integer) 3

[root@db-51 ~]#redis-cli -h 10.0.0.51 -p 26379 -a 'chaoge666' dbsize
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
(integer) 3
[root@db-51 ~]#
```

# 2.redis远程连接（python）

```
1.windows安装python3
C:\Users\Sylar>where python
C:\Users\Sylar\AppData\Local\Programs\Python\Python39\python.exe
C:\Users\Sylar\AppData\Local\Microsoft\WindowsApps\python.exe

C:\Users\Sylar>where pip
C:\Users\Sylar\AppData\Local\Programs\Python\Python39\Scripts\pip.exe




2.测试代码

# 安装python操作redis的模块
pip install redis


# 测试代码 cat test_redis.py
import redis
conn=redis.StricRedis(host='10.0.0.51',port=26379,password='chaoge666',db=0)
conn.set('name','www.yuchaoit.cn')
print(conn.get('name'))

# 执行结果

$ python test_redis.py
b'www.yuchaoit.cn'

# 至此确认远程连接，读写数据成功。
[root@db-51 /www.yuchaoit.cn/redis/data]#redis-cli -h 127.0.0.1 -p 26379 -a 'chaoge666' get name 
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
"www.yuchaoit.cn"
```

# 3.禁用危险命令

## 背景

最近安全事故濒发啊，前几天发生了“ 顺丰高级运维工程师的删库事件”，今天又看到了PHP工程师在线执行了Redis危险命令导致某公司损失400万..

什么样的Redis命令会有如此威力，造成如此大的损失？

具体消息如下：

> 据云头条报道，某公司技术部发生2起本年度PO级特大事故，造成公司资金损失400万，原因如下：
>
> 由于PHP工程师直接操作上线redis，执行键 *wxdb（此处省略）cf8* 这样的命令，导致redis锁住，导致CPU飙升，引起所有支付链路卡住，等十几秒结束后，所有的请求流量全部挤压到了rds数据库中，使数据库产生了雪崩效应，发生了数据库宕机事件。
>
> 该公司表示，如再犯类似事故，将直接开除，并表示之后会逐步收回运维部各项权限。

看完这个消息后，我心又一惊，为什么这么低级的问题还在犯？为什么线上的危险命令没有被禁用？这事件报道出来真是觉得很低级......

且不说是哪家公司，发生这样的事故，不管是大公司还是小公司，我觉得都不应该，相关负责人应该引咎辞职！

对Redis稍微有点使用经验的人都知道线上是不能执行`keys *`相关命令的，虽然其模糊匹配功能使用非常方便也很强大，在小数据量情况下使用没什么问题，数据量大会导致Redis锁住及CPU飙升，在生产环境建议禁用或者重命名！

## 还有哪些危险命令？

Redis的危险命令主要有以下几个：

- keys

客户端可查询出所有存在的键。

- flushdb

> 删除当前所选数据库的所有键。此命令永远不会失败。

删除Redis中当前所在数据库中的所有记录，并且此命令从不会执行失败。

- flushall

> 删除所有现有数据库的所有键，而不仅仅是当前选定的数据库。此命令永远不会失败。

删除Redis中所有数据库中的所有记录，不只是当前所在数据库，并且此命令从不会执行失败。

- config

客户端可修改Redis配置。

## 怎么禁用或重命名危险命令？

看下`redis.conf`默认配置文件，找到`SECURITY`区域，如以下所示。

```
1）禁用命令

rename-command KEYS     ""

rename-command FLUSHALL ""

rename-command FLUSHDB  ""

rename-command CONFIG   ""


2）重命名命令

rename-command KEYS     "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command FLUSHALL "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command FLUSHDB  "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

rename-command CONFIG   "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

上面的XX可以定义新命令名称，或者用随机字符代替。

经过以上的设置之后，危险命令就不会被客户端执行了。
```

### 实践

```
修改配置文件，加入限制参数
rename-command KEYS     "www.yuchaoit.cn"
rename-command FLUSHALL "FFTRLZL7fmI="
rename-command FLUSHDB  "XA4z7FKXag0="
rename-command CONFIG   "CIxG1pYpVis="
```