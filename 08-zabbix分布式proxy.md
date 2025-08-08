# 08-zabbix分布式proxy

# 1.为什么要学zabbix-proxy

```
https://www.zabbix.com/documentation/4.0/zh/manual/distributed_monitoring/proxies
```

![image-20220705190855504](/ajian/image-20220705190855504.png)

```
zabbix除了前面于超老师讲解的  zabbix-server  / zabbix-agent模式以外

还支持proxy分布式的功能

什么时候要用到？

zabbix proxy 使用场景:

监控远程区域设备
监控本地网络不稳定区域
当 zabbix 监控上千设备时,使用它来减轻 server 的压力
简化分布式监控的维护
```

# 2.zabbix-proxy工作流程

```
zabbix-proxy作用就是 临时存储数据，且转发，给zabbix-server，也就是采集的监控数据，中转站。

数据流走向就是

zabbix-agent  > zabbix-proxy > zabbix-server
```

![image-20220705191958671](/ajian/image-20220705191958671.png)

# 3.zabbix-proxy部署

> 准备一个新机器，部署zabbix-proxy

```bash
# 1.安装源，修改原
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm

sed -i 's#repo.zabbix.com#mirrors.tuna.tsinghua.edu.cn/zabbix#g' /etc/yum.repos.d/zabbix.repo

# 2.安装proxy和数据库
yum install zabbix-proxy-mysql mariadb-server -y

# 3.创建数据库账号
systemctl start mariadb.service
mysqladmin password www.yuchaoit.cn
mysql -uroot -pwww.yuchaoit.cn

create database zabbix_proxy character set utf8 collate utf8_bin;

grant all privileges on zabbix_proxy.* to zabbix_proxy@localhost identified by 'www.yuchaoit.cn';

flush privileges;

quit;

# 4. 导入zabbix-proxy的数据
zcat /usr/share/doc/zabbix-proxy-mysql-4.0.42/schema.sql.gz| mysql -uzabbix_proxy -pwww.yuchaoit.cn zabbix_proxy

# 5.导入zabbix-proxy配置文件

cat > /etc/zabbix/zabbix_proxy.conf <<'EOF'
ProxyMode=0 # 代理模式，0 主动， 1 被动
Server=10.0.0.61    # 填入zabbix-server地址
ServerPort=10051    # 填入zabbix-server端口
Hostname=zbx-proxy     # 填入主机名
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0 
PidFile=/var/run/zabbix/zabbix_proxy.pid 
SocketDir=/var/run/zabbix
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix_proxy
DBPassword=www.yuchaoit.cn
ConfigFrequency=60 # proxy多久和server同步配置信息
DataSenderFrequency=5 # proxy多久发送一次自己的数据给server
EOF

# 6.启动proxy
[root@zbx-proxy ~]#systemctl restart zabbix-proxy.service
```

# 4.zabbix-agent修改

```
此时的部署模式，agent要和proxy机器通信了，修改配置文件

1. 填入zabbix-proxy地址
2. 修改agent为主动模式，主动给proxy发消息，然后proxy再发给server

cat > /etc/zabbix/zabbix_agentd.conf<<'EOF'
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.62
ServerActive=10.0.0.62
HostMetadata=db
Include=/etc/zabbix/zabbix_agentd.d/*.conf
EOF

3.重启
[root@db-51 ~]#systemctl restart zabbix-agent.service
```

# 5.zabbix-UI设置代理服务器

```
到这里也就是配置zabbix-server了，设置zabbix-proxy
```

![image-20220705195841915](/ajian/image-20220705195841915.png)

具体代理配置

![image-20220705195958516](/ajian/image-20220705195958516.png)

# 6.打开zabbix-UI中的自动注册功能

![image-20220705200319657](/ajian/image-20220705200319657.png)

注意自动注册的条件。

![image-20220705200614355](/ajian/image-20220705200614355.png)

# 7.查看zabbix-server是否拿到db-51数据

通过proxy的模式

![image-20220705201825928](/ajian/image-20220705201825928.png)
