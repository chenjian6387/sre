# 03-zabbix如何监控服务器

# 1.zabbix架构图

![image-20220629141732050](/ajian/image-20220629141732050.png)

## zabbix核心概念

先记住如下zabbix中的核心几个概念

```
主机 ( HOST ) : 就是具体的一个监控对象，某一个被监控的实例，可以是一个数据库，也可以是一个操作系统。

模板 ( Template )：定义了具体一类监控对象的抽象，比如 Windows 模板，就是用来专门在监控Windows的时候，直接选择这个模板就可以实现开箱即用的数据采集。

监控项 ( ITEM )：监控项定义了具体的某一项采集指标，比如 CPU使用率，设备温度等，采集方式可以是多种支持的采集协议。

触发器 ( Trigger )：触发器是基于监控项存在的，通过WEB页面定义触发器表达式创建。比如 CPU温度大于90度，可以定义为：last(/zabbix_server/cpu_temp) > 90

动作 ( Action )：动作是基于触发器存在的，创建动作时可以选择具体的某个触发器，当触发器表达式满足要求时，会触发对应的Action执行，具体的动作定义可以执行 JavaScript，Shell脚本 等。
```

## zabbix-server

```
Zabbix Server 是 C 语言开发的 Zabbix 服务端，有着 强悍的采集和计算性能，而且资源使用率很低。主要的功能如下：

定时读取 Zabbix 数据库，同步 Zabbix UI 配置的信息到缓存，下发到 Zabbix Agent 或者 Zabbix Proxy。

关于这俩不同的采集进程，可以通过(ps -ef|grep zabbix 查看进程列表)

对于被动采集（主动和被动是从设备侧角度来看的）, Zabbix Server 会有专门的 Poller 线程去采集数据，可以定义特定的时间区间或者特定的频率。

对于主动采集，就是Agent或者设备主动上报数据，Zabbix Server 也会有专门的 Trapper 线程来接收数据，时间间隔或者频率取决于设备侧或者Agent上报配置。

接收到的历史数据（来自于 Agent、Proxy、设备侧），Zabbix Server 会缓存下来，进行告警表达式计算，进行动作触发，最终会同步到数据库的历史记录表，history开头的表。
```

## zabbix-proxy

```
Zabbix Proxy 其实就是一个简化版的Zabbix Server，具备除了 Zabbix Server 有的告警相关的功能，其他的都是 Proxy 具备的

Zabbix Proxy 会定时把采集到的数据上报到 Zabbix Server，Zabbix Server 具备全量的数据，才可以做告警计算，复杂的告警计算可以跨设备。

针对大型监控采集环境，可以 通过 Proxy 来实现分布式采集 ，可以非常有效的减轻 Zabbix Server 的采集压力。

针对 弱网环境 ，就是网络质量不高的现场环境，Proxy 可以在本地采集，缓存本地，不断重试给Server发送数据，直到发送成功为止，可以大大降低现场的采集失败率。
```

Zabbix 官方文档给出的描述

![../_images/zabbix_proxy.jpg](/ajian/zabbix_proxy.jpg)

```
更多更详细的可以看
https://www.zabbix.com/documentation/4.0/zh/manual/distributed_monitoring/proxies
```

# 2.zabbix快速监控主机（agent）

```
# 1.目标机器安装zabbix-agent 
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-agent-4.0.11-1.el7.x86_64.rpm

# 友情提醒，先做好时间同步！！
ntpdate -u ntp.aliyun.com



# 2.修改zabbix-agent配置文件

官网资料，关于配置文件的解释
https://www.zabbix.com/documentation/4.0/zh/manual/appendix/config/zabbix_agentd


修改配置如下，保证和我一样先
[root@web-7 ~]#grep -E '^[a-Z]' /etc/zabbix/zabbix_agentd.conf 
PidFile=/var/run/zabbix/zabbix_agentd.pid 
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0                # 日志切割，不切割
Server=10.0.0.61      #    填写zabbix-server地址
Include=/etc/zabbix/zabbix_agentd.d/*.conf


3.启动agent
[root@web-7 ~]#systemctl start zabbix-agent 
[root@web-7 ~]#systemctl enable zabbix-agent
Created symlink from /etc/systemd/system/multi-user.target.wants/zabbix-agent.service to /usr/lib/systemd/system/zabbix-agent.service.
[root@web-7 ~]#


4.检查,agent的端口是10050
[root@web-7 ~]#netstat -tunlp|grep zabbix
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      1644/zabbix_agentd  
tcp6       0      0 :::10050                :::*                    LISTEN      1644/zabbix_agentd
```

## 安装zabbix-get检查连接情况

```
1. 去服务端安装
[root@m-61 ~]#yum install zabbix-get -y


2. 试试和agent连得通吗？通就可以去管理了
# 友情提醒，要和zabbix_agent中填写的网段是同一个

[root@m-61 ~]#zabbix_get -s 10.0.0.7  -k agent.ping 
1
```

# 3.进入zabbix-ui添加web7机器

```
1.启动客户端机器的agent之后，以及指定了zabbix-server的地址，它俩就可以通信了

2.去zabbix-ui页面，创建主机


3.如果zabbix-server想监控自己，也要启动zabbix-agent
[root@m-61 ~]#systemctl start zabbix-agent
[root@m-61 ~]#systemctl enable zabbix-agent
Created symlink from /etc/systemd/system/multi-user.target.wants/zabbix-agent.service to /usr/lib/systemd/system/zabbix-agent.service.
[root@m-61 ~]#
```

![image-20220629161814868](/ajian/image-20220629161814868.png)

## 添加主机

```
可以填写主机名，以及可见名称

只要监控agent的接口  10.0.0.7:10050 填对了就好
```

![image-20220629162058691](/ajian/image-20220629162058691.png)

## 添加一个模板

![image-20220629162140966](/ajian/image-20220629162140966.png)

## 最终结果

![image-20220629162801826](/ajian/image-20220629162801826.png)

# 4.更细节的添加操作

```
在安装完毕zabbix-agent之后，我们想对某台机器进行监控，采集各种数据

还得进去zabbix-UI 进行主机添加，流程是

创建主机群组
创建主机
添加监控项
配置触发器
创建图形
告警配置
```

## 创建主机群组

![image-20220629173241250](/ajian/image-20220629173241250.png)

```
添加一个web组，用于管理一组都属于web组的机器，如web7 web8 web9
接下来就是添加主机了
```

## 添加主机

![image-20220629173334031](/ajian/image-20220629173334031.png)

------

![image-20220629173702551](/ajian/image-20220629173702551.png)

## 关联模板

![image-20220629173805887](/ajian/image-20220629173805887.png)

```
主机添加好之后，你就得定义要监考哪些内容了，是监控内存？CPU、磁盘、服务、还是其他？

问题是，你一台机器要添加这些、给你一百台机器，都手动反复的添加？

因此zabbix给了你模板的功能，这一堆需要监控的内容，被制作成了统一的模板，拿来即用，效率很高。

你只需要将主机和模板关联即可。

具体流程

1. 选择主机，进入主机详细
2. 进入模板选项，配置模板
3. 自己选择一个模板添加，一般直接加 template os linux即可
4. 注意要点击添加按钮。
```

![image-20220629174636209](/ajian/image-20220629174636209.png)

------

![image-20220629174723670](/ajian/image-20220629174723670.png)

## 小试牛刀，看看美观的图形

```
1.经过上述添加之后，web7机器的信息就已经被监控中了
执行如下命令，给cpu来点压力
[root@web-7 ~]#while true;do echo "www.yuchaoit.cn 超哥带你学linux";done


2.通过图形功能，进行cpu的监控图示
```

### cpu图示

![image-20220629175726522](/ajian/image-20220629175726522.png)

### 内存图示

![image-20220629180201351](/ajian/image-20220629180201351.png)
