# 31-jumpserver实战

上一节跟着超哥装好jumpserver后，后面我们就来验证，几大组件是否都能正常工作，以及如何使用。

# 虐一虐自己

```
这里，请你重启jumpserver服务器，让所有服务挂掉，你试试还能重新将官网跑起来么。

老司机是如何锻炼来的，就是反复的虐自己。。你才能进入无敌境界。
```

## 重启后无任何服务

```
[root@master-61 ~]#netstat -tunlp 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      939/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1173/master         
tcp6       0      0 :::22                   :::*                    LISTEN      939/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1173/master
```

## 启动所有组件

### 后端core

```
(py3) [root@master-61 ~]#/opt/jumpserver-v2.12.0/jms start all -d
```

### 前端lina

```
(py3) [root@master-61 ~]#cd /opt/lina-v2.12.0/
(py3) [root@master-61 /opt/lina-v2.12.0]#nohup yarn serve &
[1] 4139
(py3) [root@master-61 /opt/lina-v2.12.0]#nohup: ignoring input and appending output to ‘nohup.out’
```

### 前端luna

```
(py3) [root@master-61 /opt/lina-v2.12.0]#
(py3) [root@master-61 /opt/lina-v2.12.0]#cd /opt/luna-v2.12.0/
(py3) [root@master-61 /opt/luna-v2.12.0]#nohup ng serve &
[2] 4170
(py3) [root@master-61 /opt/luna-v2.12.0]#nohup: ignoring input and appending output to ‘nohup.out’
```

### 启动lion

```
cd /opt/lion-v2.12.0-linux-amd64/

(py3) [root@master-61 /opt/lion-v2.12.0-linux-amd64]#nohup ./lion -f config.yml &
[3] 4451
```

### 启动nginx

```
systemctl restart nginx
```

### 启动koko

```
(py3) [root@master-61 /opt/luna-v2.12.0]#/opt/koko-v2.12.0/koko -f /opt/koko-v2.12.0/config.yml -d
```

### 启动koko失败

![image-20220523172536013](http://book.bikongge.com/sre/2024-linux/image-20220523172536013.png)

### 解决办法

![image-20220523173036987](http://book.bikongge.com/sre/2024-linux/image-20220523173036987.png)

------

再次注册终端

![image-20220523173128462](http://book.bikongge.com/sre/2024-linux/image-20220523173128462.png)

------

![image-20220523173144610](http://book.bikongge.com/sre/2024-linux/image-20220523173144610.png)

### 注意，本次要修改koko配置文件

```
修改配置文件，然后重新启动koko服务
# 项目名称, 会用来向Jumpserver注册, 识别而已, 不能重复
# NAME: 

# Jumpserver项目的url, api请求注册会使用
CORE_HOST: http://127.0.0.1:8080   # Core 的地址

# Bootstrap Token, 预共享秘钥, 用来注册coco使用的service account和terminal
# 请和jumpserver 配置文件中保持一致，注册完成后可以删除
#BOOTSTRAP_TOKEN: "$BOOTSTRAP_TOKEN"

再次运行koko
[root@master-61 /opt/koko-v2.12.0]#netstat -tunlp|grep koko
tcp6       0      0 :::5000                 :::*                    LISTEN      4604/./koko         
tcp6       0      0 :::2222                 :::*                    LISTEN      4604/./koko
```

## 启动情况(进程、端口)

```
[root@master-61 /opt/koko-v2.12.0]#netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:8070            0.0.0.0:*               LISTEN      3203/python3        
tcp        0      0 127.0.0.1:4200          0.0.0.0:*               LISTEN      4170/@angular/cli   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4482/nginx: master  
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      2284/python3        
tcp        0      0 0.0.0.0:5555            0.0.0.0:*               LISTEN      2787/python3        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      939/sshd            
tcp        0      0 0.0.0.0:9528            0.0.0.0:*               LISTEN      4150/node           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1173/master         
tcp6       0      0 :::5000                 :::*                    LISTEN      4604/./koko         
tcp6       0      0 :::2222                 :::*                    LISTEN      4604/./koko         
tcp6       0      0 :::8081                 :::*                    LISTEN      4451/./lion         
tcp6       0      0 :::5555                 :::*                    LISTEN      2787/python3        
tcp6       0      0 :::22                   :::*                    LISTEN      939/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1173/master
```

# 1.修改密码

![image-20220523173656072](http://book.bikongge.com/sre/2024-linux/image-20220523173656072.png)

# 2.设置管理员邮箱

![image-20220523184727826](http://book.bikongge.com/sre/2024-linux/image-20220523184727826.png)

![image-20220523210907176](http://book.bikongge.com/sre/2024-linux/image-20220523210907176.png)

确保能发出邮件，才是正常。

# 3.设置站点URL

![image-20220523211243771](http://book.bikongge.com/sre/2024-linux/image-20220523211243771.png)

# 4.jumpserver用户管理

![image-20220523220834712](http://book.bikongge.com/sre/2024-linux/image-20220523220834712.png)

## 4.1 jumpserver的平台用户

这里指的是并非服务器上的用户、而是这个平台上的用户。

![image-20220523215230308](http://book.bikongge.com/sre/2024-linux/image-20220523215230308.png)

```
管理员用户
就是jumpserver平台上最高权限用户，一般是领导管理该账号，或者运维，可以修改平台任何的信息，如创建用户、创建资产、创建授权规则等。

普通用户
权限较低的平台用户，如管理员创建该用户后，提供给开发、测试等人员登录使用，管理自己的资产。
```

#### 4.1.1 创建平台用户

```
用户管理
↓
用户列表
↓
创建用户
```

#### 4.1.2 平台用户也可以用于登录koko

![image-20220523215523362](http://book.bikongge.com/sre/2024-linux/image-20220523215523362.png)

登录后的结果，可以实现跳板机功能。

![image-20220523215559524](http://book.bikongge.com/sre/2024-linux/image-20220523215559524.png)

## 4.2 用户登录linux机器的用户（资产管理、系统用户）

![image-20220523220102256](http://book.bikongge.com/sre/2024-linux/image-20220523220102256.png)

### 4.2.1 特权用户（系统上的sudo用户）

我们最终得登录linux服务器吧，等有linux系统上本身的用户吧，也就是

```
/etc/passwd管理的用户
我们都学过了，有普通用户、超级用户
```

1，这个管理用户是存在于资产服务器上面的用户。就相当于你服务器上面的超级用户。

2，这个用户是给Jumpserver系统平台用的，jumpserver会用这个用户去服务器上面拿到服务器的一些信息。

比如：cpu，内存，硬盘，型号等等。所以说这个用户的权限要大。

3，jumpserver还会用这个用户去帮你在服务器上面创建系统用户。

4，这个用户你必须要先在你的服务器上面创建好。

这个用户你可以用root，但是root的权限太大了。

你可以在服务器上面创建一个普通用户，然后给这个普通用户sudo权限就行。 `NOPASSWD: ALL` sudo 权限的用户

![image-20220523220421891](http://book.bikongge.com/sre/2024-linux/image-20220523220421891.png)

```
特权用户 是资产已存在的, 并且拥有 高级权限 的系统用户， 如 root 或 拥有 `NOPASSWD: ALL` sudo 权限的用户。 JumpServer 使用该用户来 `推送系统用户`、`获取资产硬件信息` 等。
```

### 4.2.2 系统用户（普通用户）

1，这个用户就相当于你服务器上面的普通用户。

2，最终开发，测试...登录到服务器上面的用户就是这个用户。

3，这个用户你可以不用先在服务器上面创建好。管理用户会帮你去创建的。

![image-20220523221306580](http://book.bikongge.com/sre/2024-linux/image-20220523221306580.png)

## 4.3 jumpserver用户实践（重要）

```
公司来了个新开发，蔡徐坤，他需要有db-52机器的权限。
```

### 4.3.1 创建平台用户

此时超哥就是管理员，给蔡徐坤创建一个账号用于登录平台。

![image-20220523221717396](http://book.bikongge.com/sre/2024-linux/image-20220523221717396.png)

此时蔡徐坤可以登录这个平台了。

首次登录，必须要改密码了。

```
新账密


caixukun01
jinitaimei666
```

### 4.3.2 创建资产（被管理的服务器信息）

要管理服务器，你是不是至少得有一个能登录这个服务器的账号？

因此先创建系统用户（特权用户，且这个用户在目标服务器上得存在）。

![image-20220523222401765](http://book.bikongge.com/sre/2024-linux/image-20220523222401765.png)

------

![image-20220523222556174](http://book.bikongge.com/sre/2024-linux/image-20220523222556174.png)

接着就可以创建资产了（用这个yuyu01用户去登录这台机器）

![image-20220523222753053](http://book.bikongge.com/sre/2024-linux/image-20220523222753053.png)

注意，这个yuyu01用户，得在目标机器上存在，jumpserver默认是用ansible，使用这个账密去连接。

```
[root@db-52 ~]#echo 'www.yuchaoit.cn' | passwd --stdin yuyu01
Changing password for user yuyu01.
passwd: all authentication tokens updated successfully.
```

还得修改sudo配置

![image-20220523223122006](http://book.bikongge.com/sre/2024-linux/image-20220523223122006.png)

测试免密sudo

```
[yuyu01@db-52 ~]$sudo ls /root
anaconda-ks.cfg  change_network.sh  hello-100.log  hosts.ip  i_am_template-100.log  install_redis.sh
[yuyu01@db-52 ~]$
[yuyu01@db-52 ~]$
[yuyu01@db-52 ~]$sudo su -
Last login: Mon May 23 22:24:14 CST 2022 from 10.0.0.1 on pts/0
[root@db-52 ~]#
[root@db-52 ~]#
```

平台上测试，yuyu01该账号，是否可管理该资产（db-52）

![image-20220523223443403](http://book.bikongge.com/sre/2024-linux/image-20220523223443403.png)

查看资产列表

![image-20220523223645604](http://book.bikongge.com/sre/2024-linux/image-20220523223645604.png)

### 4.3.3 创建普通用户（caixukun01）

此时资产创建好之后，你得给这个caixukun01权限，去登录db-52机器。

先创建普通用户

```
caixukun01
jinitaimei
```

![image-20220523223831247](http://book.bikongge.com/sre/2024-linux/image-20220523223831247.png)

### 4.3.4 资产授权

![image-20220523224138286](http://book.bikongge.com/sre/2024-linux/image-20220523224138286.png)

查看资产授权情况

![image-20220523224215303](http://book.bikongge.com/sre/2024-linux/image-20220523224215303.png)

### 4.3.5 试试让这个开发去管理该资产

```
1.登录平台
2.查看资产
3.登录资产
```

![image-20220523232920229](http://book.bikongge.com/sre/2024-linux/image-20220523232920229.png)

### 4.3.6 继续添加资产

```
1.资产管理
2.添加资产
```

![image-20220523233726600](http://book.bikongge.com/sre/2024-linux/image-20220523233726600.png)

```
注意目标机器，是否创建了 系统用户，否则ansible没法用这个账户去连接web-8
```

![image-20220523234301072](http://book.bikongge.com/sre/2024-linux/image-20220523234301072.png)

还得给这个机器，添加sudo设置，你才能够远程管理它。

```
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
yuyu01 ALL=(ALL:ALL) NOPASSWD:ALL
```

![image-20220523234539001](http://book.bikongge.com/sre/2024-linux/image-20220523234539001.png)

### 4.3.7 继续给这个开发增加资产管理权限

![image-20220523234846423](http://book.bikongge.com/sre/2024-linux/image-20220523234846423.png)

### 4.3.8 让这个开发登录web-8机器

![image-20220523235057401](http://book.bikongge.com/sre/2024-linux/image-20220523235057401.png)

```
因为你给这个资产，添加了 特权用户 yuyu01，因此在web终端上，可以选择使用了。
```

远程批量查看服务器信息，批量执行命令

![image-20220523235228990](http://book.bikongge.com/sre/2024-linux/image-20220523235228990.png)

## 4.4 管理员的强大功能

### 4.4.1 会话管理

比如蔡徐坤正在基于web终端，操作服务器，我们可以在后台看到。

![image-20220523235539991](http://book.bikongge.com/sre/2024-linux/image-20220523235539991.png)

### 4.4.2 会话监控

![image-20220523235806594](http://book.bikongge.com/sre/2024-linux/image-20220523235806594.png)

### 4.4.3 强制下线

![image-20220523235854518](http://book.bikongge.com/sre/2024-linux/image-20220523235854518.png)

### 4.4.4 视频回放

![image-20220524000144841](http://book.bikongge.com/sre/2024-linux/image-20220524000144841.png)

jumpserver提供强大的历史会话功能，查看命令历史，操作视频回放，最大程度定位运维安全。

### 4.4.5 命令历史记录

![image-20220524000236434](http://book.bikongge.com/sre/2024-linux/image-20220524000236434.png)

## 4.5 koko命令行跳板机功能

我们上面都是基于网站平台使用的堡垒机功能，再来看看koko命令行的机器管理。

![image-20220524000631003](http://book.bikongge.com/sre/2024-linux/image-20220524000631003.png)

![image-20220524000859717](http://book.bikongge.com/sre/2024-linux/image-20220524000859717.png)

监控会话

![image-20220524001047733](http://book.bikongge.com/sre/2024-linux/image-20220524001047733.png)