# 06-1-nginx监控

# 1.监控nginx链接数状态status

```
# 1.开启status页面功能

cat > /etc/nginx/conf.d/status.conf <<'EOF'
server{

    listen 80;
    server_name localhost;
    location /nginx_status {
        stub_status on;
        access_log off;
    }
}
EOF


# 2.访问测试
[root@web-7 ~]#systemctl restart nginx
[root@web-7 ~]#
[root@web-7 ~]#
[root@web-7 ~]#curl 10.0.0.7/nginx_status
Active connections: 1 
server accepts handled requests
 1 1 1 
Reading: 0 Writing: 1 Waiting: 0
```

# 2.开发nginx监控状态脚本

```
# 自定义监控内容，也就是自定义key的操作
# 脚本核心思路就是，提取status页面的数值，交给zabbix

# 1.开发监控nginx脚本

#!/bin/bash
# Author: www.yuchaoit.cn

NGINX_COMMAND=$1
CACHEFILE="/tmp/nginx_status.log"
CMD="/usr/bin/curl http://127.0.0.1/nginx_status"

# 判断是否有status日志文件
if [ ! -f $CACHEFILE ];then
    $CMD >$CACHEFILE 2>/dev/null
fi


# 检查status日志有效期，限定状态文件在60秒内
# 记录最后一次status日志的生成时间（秒）
STATUS_TIME=$(stat -c %Y $CACHEFILE)

# 以unix时间计算，seconds since 1970-01-01 00:00:00 UTC
# 当前系统时间减去日志时间，推算，是否超过60秒，超过就立即重新生成
TIMENOW=$(date +%s)

if [  $[ $TIMENOW - $STATUS_TIME ]  -gt 60 ];then
    rm -f $CACHEFILE
fi

if [ ! -f $CACHEFILE ];then
    $CMD > $CACHEFILE 2>/dev/null 
fi


nginx_active(){
    grep 'Active' $CACHEFILE |awk '{print $NF}'
    exit 0;
}

nginx_reading(){
    grep 'Reading' $CACHEFILE |awk '{print $2}'
    exit 0;
}

nginx_writing(){
    grep 'Writing' $CACHEFILE |awk '{print $4}'
    exit 0;
}

nginx_waiting(){
    grep 'Waiting' $CACHEFILE |awk '{print $6}'
    exit 0;
}

nginx_accepts(){
    awk NR==3 $CACHEFILE|awk '{print $2}'
    exit 0;
}

nginx_handled(){
    awk NR==3 $CACHEFILE|awk '{print $2}'
    exit 0;
}

nginx_requests(){
    awk NR==3 $CACHEFILE|awk '{print $3}'
    exit 0;
}


# 对脚本传入参数判断，需要获取什么值
case $NGINX_COMMAND in 
    active)
        nginx_active ;;
    reading)
        nginx_reading;;
    writing)
        nginx_writing;;
    waiting)
        nginx_waiting;;
    accepts)
        nginx_accepts;;
    handled)
        nginx_handled;;
    requests)
        nginx_requests;;
    *)
        echo "Invalid arguments" 
        exit 2
        ;;
esac
```

# 3.编写agent自定义key配置文件

```bash
[root@web-7 /etc/zabbix/zabbix_agentd.d]#cat nginx_status.conf 
UserParameter=nginx_status[*],/bin/bash /etc/zabbix/zabbix_agentd.d/nginx_status.sh $1



# 授权
[root@web-7 /etc/zabbix/zabbix_agentd.d]#chmod +x nginx_status.sh 
[root@web-7 /etc/zabbix/zabbix_agentd.d]#chown -R zabbix.zabbix ./*
[root@web-7 /etc/zabbix/zabbix_agentd.d]#ll
total 16
-rw-r--r-- 1 zabbix zabbix   87 Jul  4 13:40 nginx_status.conf
-rwxr-xr-x 1 zabbix zabbix 1697 Jul  4 13:40 nginx_status.sh
-rw-r--r-- 1 zabbix zabbix   52 Jun 29 19:06 tcp_status.conf
-rw-r--r-- 1 zabbix zabbix 1531 Jul 29  2019 userparameter_mysql.conf
[root@web-7 /etc/zabbix/zabbix_agentd.d]#


# 重启
[root@web-7 /etc/zabbix/zabbix_agentd.d]#systemctl restart zabbix-agent
```

# 4.测试zabbix_get

这里务必测试所有参数都是有结果的

```
[root@m-61 ~/p3-shell]#
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[active]
1
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[reading]
0
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[writing]
1
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[waiting]
0
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[accepts]
10074
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[handled]
10075
[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[requests]
10007


[root@m-61 ~/p3-shell]#zabbix_get -s 10.0.0.7 -k nginx_status[requestsxxx]
Invalid arguments
[root@m-61 ~/p3-shell]#
```

# 5.zabbix-UI界面创建模板

按照如图规则，创建7个监控项，触发器即可。

快速克隆即可。

![image-20220704140758558](http://book.bikongge.com/sre/2024-linux/image-20220704140758558.png)

触发器

```
测试加一个，当requests请求数到达5万时报警
```

![image-20220704141102945](http://book.bikongge.com/sre/2024-linux/image-20220704141102945.png)

# 6.主机关联模板

```
分别给web7 和web8 关联模板，使用该监控项
```

![image-20220704141151602](http://book.bikongge.com/sre/2024-linux/image-20220704141151602.png)

# 7.查看最新数据

![image-20220704141339465](http://book.bikongge.com/sre/2024-linux/image-20220704141339465.png)

# 8.查看nginx_status图形数据

![image-20220704141549564](http://book.bikongge.com/sre/2024-linux/image-20220704141549564.png)