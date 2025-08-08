# 07-2-zabbix自动发现

# 1.什么是zabbix自动发现

```
当需要监控的主机数量太多，你没办法一个个去web页面添加‘
因此zabbix可以基于网络环境，自动发现，添加主机且监控。

自动发现可以完成
1.自动发现、添加主机
2.添加主机且指定关联的模板


zabbix的自动发现可以基于
1. ip地址、或者ip网段
2. 来自于zabbix-agent的请求
3. 基于SNMP客户端的请求


自动发现有坑，如网络等待太久
以及自动发现无须修改zabbix_agent配置文件，官网也没有修改操作。
```

# 2.配置zabbix自动发现

![image-20220705163218917](/ajian/image-20220705163218917.png)

添加新的自动发现规则

![image-20220705163953846](/ajian/image-20220705163953846.png)

```
这里的更新时间，基于测试的环境，可以短一点，方便调试
生产环境不宜太短，否则频繁检测，必然有额外的系统资源消耗。
```

## 2.1 自动发现之后的动作

![image-20220705164206093](/ajian/image-20220705164206093.png)

------

创建动作，注意选择，【自动发现】

![image-20220705165115672](/ajian/image-20220705165115672.png)

------

创建动作的条件

![image-20220705165614246](/ajian/image-20220705165614246.png)

------

![image-20220705165744405](/ajian/image-20220705165744405.png)

## 2.2 检查自动发现规则

![image-20220705165907373](/ajian/image-20220705165907373.png)

## 2.3 新加几个机器试试

db-51 db-52

部署脚本

```
# 1.安装zabbix-agent，修改配置文件，填入zabbix-server即可

# 1.目标机器安装zabbix-agent 
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-agent-4.0.11-1.el7.x86_64.rpm

# 友情提醒，先做好时间同步！！
ntpdate -u ntp.aliyun.com



# 2.修改zabbix-agent配置文件
# 修改配置如下，保证和我一样即可，无须添加其他参数。
cat >/etc/zabbix/zabbix_agentd.conf <<'EOF'
PidFile=/var/run/zabbix/zabbix_agentd.pid 
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.61
Include=/etc/zabbix/zabbix_agentd.d/*.conf
EOF

# 3.启动agent
systemctl restart zabbix-agent 
systemctl enable zabbix-agent


netstat -tunlp|grep zabbix
```

## 2.4 确认自动添加成功

一般要等待几分钟即可自动发现。

![image-20220705173841856](/ajian/image-20220705173841856.png)

查看zabbix-server日志

![image-20220705173821954](/ajian/image-20220705173821954.png)

## 2.5 坑记录,zabbix-server报警

![image-20220705174126960](/ajian/image-20220705174126960.png)

```
[root@m-61 ~/p3-shell]#grep -E '^[a-Z]' /etc/zabbix/zabbix_server.conf 
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
SocketDir=/var/run/zabbix
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=www.yuchaoit.cn
StartDiscoverers=50
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
UnavailableDelay=360
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=3000
```

## 2.6 再加一个cicd-99机器

测测自动发现

![image-20220705181856450](/ajian/image-20220705181856450.png)
