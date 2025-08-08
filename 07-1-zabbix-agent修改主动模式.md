# 07-1-zabbix-agent修改主动模式

```
一个经典的面试问题

说说zabbix的主动模式、被动模式你了解多少。
```

# 1.zabbix-agent工作模式

```
zabbix-agent进程，有两种工作模式，主动模式，被动视频
```

## 1.1 被动模式

```
被动模式是指
zabbix-server 将需要请求的数据，发给zabbix-agent，然后agent接收到请求后才进行对客户端机器数据采集，采
集完毕后发给zabbix-server，交给zabbix-UI展示。

但是这个过程是一次一次完成的，也就是，有多少个监控项，就需要发起多少次请求，执行多少次脚本。

zabbix-server发起请求后，也同时在等待agent返回数据；
zabbix-agent接收请求的时候，也在响应、发送数据。

因此这种效率，机器数量一旦多了，被动模式的效率就彻底废了。
```

![image-20220705152104941](http://book.bikongge.com/sre/2024-linux/image-20220705152104941.png)

## 1.2 主动模式

主动模式下，由zabbix-agent主动给zabbix-server发请求，索要监控项的列表，然后agent将对应的数据一次性全部采集后发给zabbix-server。

明显agent主动发数据，效率要高得多，很适合监控项非常多，以及机器数量规模较大的场景。

![image-20220705153118815](http://book.bikongge.com/sre/2024-linux/image-20220705153118815.png)

# 2.zabbix-agent主动模式更改

只需要zabbix-agent的配置文件即可

```
[root@web-7 ~]#grep '^[a-Z]' /etc/zabbix/zabbix_agentd.conf 
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.61
ServerActive=10.0.0.61  # 必须加这行，填入zabbix-server地址
Hostname=web-7                    # 必须加这行，填入当前机器主机名
Include=/etc/zabbix/zabbix_agentd.d/*.conf


重启agent服务
systemctl restart zabbix-agent
```

注意agent配置文件的主机名，要和zabbix-UI中的主机名对上。

![image-20220705155807395](http://book.bikongge.com/sre/2024-linux/image-20220705155807395.png)

# 3.zabbix-UI里修改监控项为主动模式

```
这里以更通用的内置 template os linux模板的监控项为案例
```

![image-20220705154606653](http://book.bikongge.com/sre/2024-linux/image-20220705154606653.png)

点击批量更新

![image-20220705154756918](http://book.bikongge.com/sre/2024-linux/image-20220705154756918.png)

## 3.1查看最新的监控项数据

![image-20220705162307321](http://book.bikongge.com/sre/2024-linux/image-20220705162307321.png)