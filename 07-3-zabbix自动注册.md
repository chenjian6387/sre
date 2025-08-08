# 07-3-zabbix自动注册

# 1.什么是自动注册

```
前面于超老师带你学习了自动发现，也就是配置好一个网络环境后，zabbix-server主动去网络环境中扫描，然后发现目标机器然后监控，此时的agent是被动等待的。
那如果需要扫描多种网段，且机器数量很大的话，你的zabbix-server服务器可就很难受了。。。



因此自动注册，就是由zabbix-agent主动发起注册请求，你来监控我把！！！
极大的降低了zabbix-server的服务器压力。
```

# 2.配置自动注册

![image-20220705182505297](/ajian/image-20220705182505297.png)

填写详细信息，自动注册的规则

![image-20220705183320715](/ajian/image-20220705183320715.png)

## 2.1 自动注册动作

逻辑就是，先注册上这台机器，注册好之后，要干啥？写动作。

![image-20220705183459660](/ajian/image-20220705183459660.png)

# 3.修改zabbix-agent配置文件

删除zabbix里的cicd-99机器，测试下，能否自动注册，或者你重新装一个机器。

```
改为自动注册的配置文件，添加
1. ServerActive=填写zabbix-server地址
2. HostMetadata，填写元数据的关键词，也就是自动注册的条件


[root@cicd-99 ~]#cat  /etc/zabbix/zabbix_agentd.conf 
PidFile=/var/run/zabbix/zabbix_agentd.pid 
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.61
ServerActive=10.0.0.61
HostMetadata=Linux
Include=/etc/zabbix/zabbix_agentd.d/*.conf


3.重启agent
systemctl restart zabbix-agent
```

![image-20220705184100308](/ajian/image-20220705184100308.png)
