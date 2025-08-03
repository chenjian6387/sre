#  09-SSH安全登录

![image-20220421180411182](http://book.bikongge.com/sre/2024-linux/image-20220421180411182.png)

# 机器准备

![image-20220422190745439](http://book.bikongge.com/sre/2024-linux/image-20220422190745439.png)

# 什么是SSH

SSH 或 Secure Shell 协议是一种远程管理协议，允许用户通过 Internet 访问、控制和修改其远程服务器。

SSH 服务是作为未加密 Telnet 的安全替代品而创建的，它使用加密技术来确保进出远程服务器的所有通信都以加密方式进行。

> 它提供了一种用于验证远程用户、将输入从客户端传输到主机以及将输出转发回客户端的机制。

SSH是每一台Linux电脑的标准配置。

随着Linux设备从电脑逐渐扩展到手机、外设和家用电器，SSH的使用范围也越来越广。

不仅程序员离不开它，很多普通用户也每天使用。

SSH具备多种功能，可以用于很多场合。有些事情，没有它就是办不成。

## SSH是一种网络协议

简单说，SSH是一种网络协议，用于计算机之间的加密登录。

如果一个用户从本地计算机，使用SSH协议登录另一台远程计算机，我们就可以认为，这种登录是安全的，即使被中途截获，密码也不会泄露。

最早的时候，互联网通信都是明文通信，一旦被截获，内容就暴露无疑。1995年，芬兰学者Tatu Ylonen设计了SSH协议，将登录信息全部加密，成为互联网安全的一个基本解决方案，迅速在全世界获得推广，目前已经成为Linux系统的标准配置。

需要指出的是，SSH只是一种协议，存在多种实现，既有商业实现，也有开源实现。

本文针对的实现是[OpenSSH](https://www.openssh.com/)，它是自由软件，应用非常广泛。

## SSH客户端

- MobaXterm
- Xshell
- Termius
- PuTTY
- Terminal
- Finalshell
- linux提供的ssh命令

```
# ssh的man帮助
SSH(1)                                     BSD General Commands Manual                                     SSH(1)

NAME
     ssh — OpenSSH SSH client (remote login program)


[root@nfs-31 ~]#rpm -qf /usr/bin/ssh
openssh-clients-7.4p1-16.el7.x86_64
```

# 为什么需要SSH

如果一个用户从本地计算机，使用`SSH协议登录另一台远程计算机`，我们就可以认为，这种登录是安全的，即使被中途截获，密码也不会泄露。

Secure Shell是Linux系统首选的登录方式，以前使用FTP或telnet登录服务器，**都是以明文的形式在网络中发送账号密码**，很容易被黑客截取到数据，篡改后威胁服务器数据安全。

因此如何对数据加密，安全传输是重中之重，主要方式有两种：

- 对称加密（秘钥加密）
- 非对称加密（公钥加密）

## 连接服务器的方式

```
1. 去机房，拿上键盘、显示器直接连接
2. 使用telnet命令连接（明文数据，密码未加密）
3. 使用ssh安全可靠的远程登录（密码被加密）
```

## 不安全登录telnet（实践）

我们一切操作，都会产生各种数据包，且都是隐藏在计算机背后的网络数据交互，看不见，摸不着，但是可以通过抓包工具，专门的提取出这些数据包，进行分析。

这一功能，可以让运维同学理解服务启停、部署、运行过程中的网络通信原理，以及故障分析。可以让开发同学，进行接口调试，网页数据提取，比如黑客，是熟练玩转抓包工具的。

简单说，你打开QQ、邮箱，输入账号密码、点击登录，其实计算机背后产生了N多个数据交互，请求与响应，通过抓包工具，甚至可以抓取出账号密码等重要信息。(这只是一个应用场景，还是要遵纪守法)

> 我们以telnet命令，登录服务器为例，查看数据包
>
> telnet是早期用来登录服务器、交换机的一个指令，但是登录账密是明文的不够安全，后来采用ssh登录linux了。

```
1.部署telnet服务端，用于telnet登录该机器
[root@nfs-31 ~]#yum install telnet-server telnet -y

2.启动telnet服务
systemctl start telnet.socket
systemctl status telnet.socket

3.检查telnet服务，默认23端口
[root@nfs-31 ~]#netstat -tunlp|grep 23
tcp6       0      0 :::23                   :::*                    LISTEN      1/systemd 

4.注意关闭防火墙
systemctl stop firewalld
```

## 打开windows的telnet功能

![image-20220125143009439](http://book.bikongge.com/sre/2024-linux/image-20220125143009439.png)

------

![image-20220125143039494](http://book.bikongge.com/sre/2024-linux/image-20220125143039494.png)

## 修改linux认证

linux默认为了安全性，已经禁用了telnet登录，可以临时修改这个，可以测验telnet登录服务器。

```
[root@nfs-31 ~]#vim /etc/pam.d/remote 
# 注释这一行
#auth       required     pam_securetty.so
```

或者使用普通用户，telnet即可登录了

```
useradd yuchao01
```

## windows中telnet登录

```
1.打开cmd
C:\Windows\system32>telnet 10.0.0.31

2.是可以登录的
Kernel 3.10.0-862.el7.x86_64 on an x86_64                                                                               nfs-31 login: root                                                                                                      Password:                                                                                                               Last failed login: Thu Apr 21 19:11:48 CST 2022 from ::ffff:10.0.0.1 on pts/0                                           There was 1 failed login attempt since the last successful login.                                                       Last login: Thu Apr 21 19:11:32 from ::ffff:10.0.0.1                                                                    [root@nfs-31 ~]#
```

## 安装wireshark抓包工具

### 抓取telnet的明文密码

抓取telnet登录时发出的网络数据包，也就是输入的账号密码信息

![image-20220421192500878](http://book.bikongge.com/sre/2024-linux/image-20220421192500878.png)

开始捕获后，开始你的telnet登录操作

![image-20220421192903496](http://book.bikongge.com/sre/2024-linux/image-20220421192903496.png)

### 试试抓取ssh的密码

![image-20220421200047572](http://book.bikongge.com/sre/2024-linux/image-20220421200047572.png)

### 小结

```
1.ssh是足够安全的，但是你不能说服务器就没危险了
2.危险的在于你是否能保管好linux的用户密码

3.银行很安全吧
4.你要是银行卡、密码被人钓鱼网站骗取了，还安全吗？
5.所以，做好安全防护，有安全意识最重要，最不安全的其实是人。
```

# 学习SSH的任务背景

```
这里讲解堡垒机、或是跳板机，并非和ssh知识点有直接关联、而是ssh应用的一个明显场景；

ssh的作用依然只是进行服务器的安全连接
1.安全登录服务器，进行账号密码验证；
2.设置密码以外认证方式，使用秘钥登录。

堡垒机是利用该ssh特性，加大服务器安全而设计的一种服务器集群形式，以后超哥还会再针对堡垒机业务讲解。
```

![image-20220422155303302](http://book.bikongge.com/sre/2024-linux/image-20220422155303302.png)

为了最大程度的保护公司内网服务器的安全，公司内部有一台服务器做跳板机（JumpServer）。

运维人员在维护过程中首先要统一登录到这台服务器，然后再登录到目标设备进行维护和操作。

由于开发人员有时候需要通过跳板机登录到线上生产环境查看一些业务日志，所以现在需要运维人员针对不同的人员和需求对==账号密码进行统一==管理，并且遵循**权限最小化**原则。

```
禁用root登录
可以用sudo提权
```

# 部署ssh要求

![image-20220210144727523](http://book.bikongge.com/sre/2024-linux/image-20220210144727523.png)

1. 跳板机上为开发人员创建用户，及公共目录，供开发人员使用，并做好权限控制
2. 所有线上生产服务器搭建sshd服务。
3. 对于ssh服务根据需求进行配置
   - 禁止root用户远程登录
   - 更改默认端口（22=>10086）
4. 线上生产服务器创建devyu用户，并安装工具来生成随机密码

## 3.涉及知识点

- 权限管理，文件权限，用户权限
- ssh服务配置
- 生成随机密码工具

# 服务部署基础知识

## 1、什么是服务（程序）

- 运行在操作系统后台的一个或者多个程序，为系统或者用户提供特定的`服务`
- 可靠的，并发的，连续的不间断的运行，随时接受请求
- 通过交互式提供服务

## 2、服务架构模型

### 1、B/S架构

- B/S(browser/server) 浏览器/服务器

概念：这种结构用户界面是完全通过浏览器来实现，使用http协议、https协议

优势：节约开发成本

![image-20220210145103400](http://book.bikongge.com/sre/2024-linux/image-20220210145103400.png)

### 2、C/S架构

- C/S（client/server）客户端/服务器

概念：指的是客户端和服务端之间的通信方式，客户端提供用户请求接口，服务端响应请求进行对应的处理，并返回给客户端

优势：安全性较高，一般面向具体的应用

![image-20220210145346035](http://book.bikongge.com/sre/2024-linux/image-20220210145346035.png)

### 3、两者区别

**B/S：** 1、广域网，只需要有浏览器即可 2、一般面向整个互联网用户，安全性低 3、维护升级简单

**C/S：** 1、需要具体安装对应的软件 2、一般面向固定用户，安全性较高

> 我们是怎么访问的淘宝网？
>
> 1.www.taobao.com，浏览器访问网站，IP+port
>
> 2.淘宝APP

```
https://www.yuchaoit.cn:443

比如这样一个完整的URL，包括了协议，主机名，端口号
```

## 3、端口号设定

**说明:端口号只有整数，范围是从0 到65535**

- 1～255：一**般是知名端口号**，如:ftp 21号、web 80、ssh 22、telnet 23号等
- 256～1023：通常都是由Unix系统占用来提供特定的服务
- 1024~5000：客户端的临时端口，随机产生
- 大于5000：为互联网上的其他服务预留，工作里一般建议直接用大于5000的端口，并且要使用netstat命令检查下。

## 3.1 查看系统默认服务端口

```
[root@yuchao-linux01 ~]# cat /etc/services  |wc -l
11176
```

![image-20220210150134620](http://book.bikongge.com/sre/2024-linux/image-20220210150134620.png)

你看，这linux默认有11176个默认端口，表示每一个程序，默认启动后，会打开这个端口，提供访问。

那么如果黑客根据这个服务表，大规模，批量扫描这些端口，以及尝试暴力用密码登录，那你的服务器就很危险了。

## 4、常见的网络服务

- 文件共享服务：==FTP、SMB、NFS==
- 域名管理服务：==DNS==
- 网站服务：==Apache(httpd)==、Nginx、Lighttpd、IIS
- 邮件服务: Mail
- 远程连接服务：==SSH==、telnet
- 动态地址管理服务:DHCP

# (实战1)SSH密码登录原理

熟悉Linux的人那肯定都对SSH不陌生。

ssh是一种用于安全访问远程服务器的协议，远程管理工具。

它之所以集万千宠爱为一身，就是因为它的安全性。那么它到底是怎么样来保证安全的呢？到底是如何工作的呢？

**首先，在讲SSH是如何保证安全的之前，我们先来了解以下几个密码学相关概念：**

### 1、加密算法（了解）

#### **①对称加密算法(DES)**

```
https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%8A%A0%E5%AF%86%E6%A8%99%E6%BA%96

数据加密标准（英语：Data Encryption Standard，缩写为 DES）是一种对称密钥加密块密码算法

https://baike.baidu.com/item/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95/211953

对称加密算法
```

- 优点、该算法加密强度大，几乎无法破解
- 缺点，密钥不能丢失，拿到加密方式，自然源数据就被破解，暴露给别人了。

![image-20220210151308351](http://book.bikongge.com/sre/2024-linux/image-20220210151308351.png)

1.于超想和一个美女，杰西卡打招呼，但是又怕被女朋友发现，因此于超用了一个加密算法，比如通过一个`密钥A`来给打招呼的信息加密，得到一个密文数据，其他人是看不懂的，再发给这位外国美女杰西卡。

2.杰西卡收到消息后，必须通过同样的`密钥A`解密，才能看懂这句话，"交个朋友吧"

![image-20220210151741056](http://book.bikongge.com/sre/2024-linux/image-20220210151741056.png)

3.加密算法是指通过程序对明文计算处理后，得到一个无法直观看懂的数据。

> 总结
>
> 1.发送方，使用密钥、对明文数据加密，然后再发出去
>
> 2.接收方，必须用同一个密钥、对密文解密，才能转为明文。
>
> 3.如果明文数据不加密直接发送，是非常危险的，很容易就被其他人捕获，比如于超给美女打招呼，立马被女朋友发现，当场腿打断。
>
> 4.对称加密强度很高，难以破解，问题是当机器数量较多的时候，大量的发送密钥，难以保证密钥安全性，一旦某一个Client被窃取密钥，其他机器的安全性也就崩塌了，**因此，非对称加密应运而生**

#### **②非对称加密算法(RSA)**

```
非对称加密分为：公钥（Public Key）与私钥（Private Key）

使用公钥加密后的密文，只能使用对应的私钥才能解开，破解的可能性很低。
```

![image-20220210154234611](http://book.bikongge.com/sre/2024-linux/image-20220210154234611.png)

1.杰西卡生成一对公私钥、其中公钥是可以直接发给任何人的，但是私钥必须杰西卡自己保护好，不得泄露；

2.当于超给杰西卡打招呼时，杰西卡的公钥会给发于超；

3.于超拿着杰西卡的公钥对明文加密，得到密文，此时可以公开发给杰西卡了；

4.杰西卡收到密文后，通过自己本地的私钥，将这个密文，解析为明文 "交个朋友吧"；

> 总结：
>
> 1.发送方（于超）使用接收方（杰西卡）发来的`公钥`将`明文数据加密`为密文，然后再发出;
>
> 2.接收方（杰西卡）收到密文消息后，用自己本地保存的`私钥`解密这个密文，最终得到明文数据；

### 2、对称、非对称加密算法区别是？

- 对称加密
  1. 使用同一个密钥进行加密和解密，密钥容易泄露
  2. 加密速度快，效率高
  3. 数据传输，速度快
  4. 安全性较低；
- 非对称加密
  1. 使用不同的密钥（公钥和私钥）进行加密和解密
  2. 加密速度远远慢于对称加密
  3. 数据传输速度慢
  4. 安全性较高

> 由于机器配置足够高，网速足够快，这些快慢几乎区别不大，只有在超大大规模机器数量下，才能看出区别。
>
> 因为如今的网络传输速率、机器IO速率太快了；

既然知道了关于对数据要加密后再传输，否则就是不安全的。

这就指的是，当我们用ssh命令登录时，输入的账户密码，也是被加密后发送给linux服务器的，要么多不安全。

### 3、SSH认证方式

我们登录linux服务器，使用ssh登录的话有两种认证方式

- 账户密码

```
root
yuchao666
```

- 密钥认证

```
[root@web-7 ~]#ls /root/.ssh/id_rsa
id_rsa      id_rsa.pub
```

### 4、（重点）SSH基于用户名密码认证原理

```
密码登录，使用的是目标机器的，公私钥
[root@rsync-41 ~]#ls /etc/ssh/ssh_host_ecdsa_key*
/etc/ssh/ssh_host_ecdsa_key  /etc/ssh/ssh_host_ecdsa_key.pub
```

![image-20220210165927280](http://book.bikongge.com/sre/2024-linux/image-20220210165927280.png)

1.SSH客户端向SSH服务端发起登录请求

2.SSH服务端将自己的公钥发给SSH客户端

注意，如果是首次建立连接，会有如下指纹信息确认，让用户确认自己连接的机器信息正确。

```
[root@web-7 ~]#ssh root@10.0.0.41
The authenticity of host '10.0.0.41 (10.0.0.41)' can't be established.
ECDSA key fingerprint is SHA256:Csqwr63+SZRFFOug/IGoFTgRe8hDSI/QalSMBcC6IaU.
ECDSA key fingerprint is MD5:4c:9a:37:e2:5b:b5:de:a8:bf:90:b5:28:d8:5b:ac:60.
Are you sure you want to continue connecting (yes/no)?
```

3.在首次登录时，只要你输入了yes，就表示你信任了该机器的公钥，该机器的公钥信息会写入到客户端的`~/.ssh/known_hosts`

```
你可以在客户端检查该文件
[root@web-7 ~]#cat ~/.ssh/known_hosts 
10.0.0.41 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL/Sx3bAaNcKqo7pC4FTYk3gyZ6hd1D/DKUWVfOd4gZb/8XwlAxWauceHe/BAsW5Z8pEmG6AjSyHM8ckOs94c7Y=

如果你删除了该公钥，下次ssh连接，会再次让你确认机器的指纹信息
```

4.下一步就是等待你输入密码，只要你输入了密码，SSH客户端就会使用服务端发来的公钥，将超哥输入的密码加密为密文后，再发给SSH服务端；

```
[root@web-7 ~]#ssh root@10.0.0.41
The authenticity of host '10.0.0.41 (10.0.0.41)' can't be established.
ECDSA key fingerprint is SHA256:Csqwr63+SZRFFOug/IGoFTgRe8hDSI/QalSMBcC6IaU.
ECDSA key fingerprint is MD5:4c:9a:37:e2:5b:b5:de:a8:bf:90:b5:28:d8:5b:ac:60.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.41' (ECDSA) to the list of known hosts.
root@10.0.0.41's password:
```

5.SSH服务端收到密文密码后，再用自己本地的私钥解密，看到超哥输入的密码；

```
root@10.0.0.41's password:   **********
你输入的密码，会被公钥加密，发送到目标机器上验证，ssh真是太靠谱了
```

6.SSH服务端将解密后的明文，和linux上的用户密码文件做对比，`/etc/shadow`，正确则登录成功

7.ssh认证成功后，返回登录成功，并且返回一个随机会话口令给客户端，这个随机口令用于后续两台机器之间的数据通信加密。

### 4.1 密码登录ssh小结

```
1.ssh登录时，为了最大程度保证账户、密码安全，使用非对称加密；

2.登录后，客户端、服务端之间的数据通信，采用随机口令，再对随机口令进行对称加密，因为速度快，开始后续的ssh命令执行，都是加密的数据操作。
```

### 5.如何确认目标机器的正确性（了解）

```
ecdsa算法
椭圆曲线数字签名算法 (Elliptic Curve Digital Signature Algorithm)
https://zh.wikipedia.org/zh-hans/%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E7%AE%97%E6%B3%95
```

#### 机器指纹信息

```
只要是两台新机器，首次ssh连接，就会出现如下的指纹确认
[root@web-7 ~]#
[root@web-7 ~]#ssh root@10.0.0.41
The authenticity of host '10.0.0.41 (10.0.0.41)' can't be established.
ECDSA key fingerprint is SHA256:Csqwr63+SZRFFOug/IGoFTgRe8hDSI/QalSMBcC6IaU.
ECDSA key fingerprint is MD5:4c:9a:37:e2:5b:b5:de:a8:bf:90:b5:28:d8:5b:ac:60.
Are you sure you want to continue connecting (yes/no)? 

这一段信息意思是，无法确认10.0.0.41这台机器的真实性，但是发现了机器的指纹
SHA256:Csqwr63+SZRFFOug/IGoFTgRe8hDSI/QalSMBcC6IaU.
你自己确认下是否是你要连接的机器
```

#### ssh-keygen扫描公钥

ssh-keyscan命令，可以获取目标机器的公钥信息，通过web-7扫描 rsync-41机器的公钥

```
[root@web-7 ~]#ssh-keyscan 10.0.0.41 
# 10.0.0.41:22 SSH-2.0-OpenSSH_7.4
10.0.0.41 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC9lqWZdqpEGg6hV4PbSf/n0NI33oaedK/KFP5YWxF6A
# 10.0.0.41:22 SSH-2.0-OpenSSH_7.4
10.0.0.41 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC/a19vEmUMzhzEM8Hbqy2Cd2KL838ojWDO/TDiMOpaKZ6okJ7kBsy8eK0ukYqMj/xBgco8rlc8nz171rul3HN1uUgu7+c8TNbAyggyiWNmaZzYMbWYwC5ZhhlPA8RU+39Znw4BpwKqEC76h2dAj/OOwEADIJK4GuwEnsTmCy2CSh4hqL7M1MWlsV7Kkc0ZBA4tYmz/qAYakl8PFF+CY7YWMJFWDvqLxGdEcqBkvIJ1hEuB21UDhXCOimYgp94frzcyeFoGv4WKcKw7Dq3lPh9gT038v6bw/EA5PC4ubjmly3XJUKZoRVcg773Oa/TcBOeZfTW2zPK83X0hQNk9T1KJ
# 10.0.0.41:22 SSH-2.0-OpenSSH_7.4
10.0.0.41 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL/Sx3bAaNcKqo7pC4FTYk3gyZ6hd1D/DKUWVfOd4gZb/8XwlAxWauceHe/BAsW5Z8pEmG6AjSyHM8ckOs94c7Y=
```

#### 查看机器公钥

直接去rsync-41机器上检查公钥信息

```
[root@rsync-41 ~]#
[root@rsync-41 ~]#cat /etc/ssh/ssh_host_ecdsa_key.pub 
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL/Sx3bAaNcKqo7pC4FTYk3gyZ6hd1D/DKUWVfOd4gZb/8XwlAxWauceHe/BAsW5Z8pEmG6AjSyHM8ckOs94c7Y=
```

#### 指纹是公钥加密而来(sha256算法)

计算rsync-41机器的指纹，再和我们第一次ssh连接的命令比较，是否一致

```
[root@rsync-41 ~]#ssh-keygen -E SHA256 -lf /etc/ssh/ssh_host_ecdsa_key.pub
256 SHA256:Csqwr63+SZRFFOug/IGoFTgRe8hDSI/QalSMBcC6IaU no comment (ECDSA)
```

#### 对比机器指纹

![image-20220422164206176](http://book.bikongge.com/sre/2024-linux/image-20220422164206176.png)

## 6、ssh密码认证小结

- SSH是Linux下远程管理的工具，相比Telnet安全，运维人员必备的神器！
- SSH的全称Secure Shell，安全的shell，是Client/Server架构，默认端口号为22，TCP协议
- 必须搞懂SSH通信加密的原理、过程

# (实战2-服务器安全)sshd服务部署

### 1、搭建所有服务的套路

- 关闭防火墙和selinux(实验环境都先关闭掉)
- 配置yum源(公网源或者本地源)
- 软件安装和检查
- 了解并修改配置文件
- 启动服务检查运行状态并设置开机自启动

### 2、搭建SSH服务

这部分内容可以参考于超老师前面讲解的系统初始化篇操作即可

#### （一）关闭防火墙和selinux

```
# 关闭firewalld防火墙
# 临时关闭
systemctl stop firewalld

# 关闭开机自启动
systemctl disable firewalld

# 关闭selinux
# 临时关闭
setenforce 0

# 修改配置文件  永久关闭
vim /etc/selinux/config
SELINUX=disabled
```

#### （二）配置yum源

> 注意：一般情况下使用网络源即可。
>
> 如果**没有网络**的情况下，才需要配置本地源

```
# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# yum clean all
# yum makecache

这个yum源配置，
```

#### （三）软件安装

##### ①确认是否安装openssh

由于每一台机器都是默认安装了sshd服务的，这里你可以采用期中综合架构里的任何一台机器测试。

```
[root@web-7 ~]#rpm -qa |grep openssh
openssh-server-7.4p1-16.el7.x86_64
openssh-clients-7.4p1-16.el7.x86_64
openssh-7.4p1-16.el7.x86_64
```

##### ②查看openssh-server软件包的文件列表

```
[root@yuchao-linux01 ~]# rpm -ql openssh-server
# 配置文件
/etc/ssh/sshd_config
/etc/sysconfig/sshd

# 服务管理脚本
/usr/lib/systemd/system/sshd.service        =>  systemctl start sshd

# 文件共享服务 提供文件上传下载的服务
/usr/libexec/openssh/sftp-server

# 二进制文件程序文件
/usr/sbin/sshd

# 公钥生成工具
/usr/sbin/sshd-keygen

# man手册
/usr/share/man/man5/sshd_config.5.gz
/usr/share/man/man8/sftp-server.8.gz
/usr/share/man/man8/sshd.8.gz
```

##### ③查看openssh-clients软件包的文件列表

```
rpm -ql openssh-clients

# 客户端配置文件
/etc/ssh/ssh_config

# 远程copy命令 服务器间进行文件传输
/usr/bin/scp

# sftp客户端  上传下载文件操作
/usr/bin/sftp
/usr/bin/slogin
/usr/bin/ssh
/usr/bin/ssh-add
/usr/bin/ssh-agent
/usr/bin/ssh-copy-id
/usr/bin/ssh-keyscan

# 客户端man手册
/usr/share/man/man1/scp.1.gz
/usr/share/man/man1/sftp.1.gz
/usr/share/man/man1/slogin.1.gz
/usr/share/man/man1/ssh-add.1.gz
/usr/share/man/man1/ssh-agent.1.gz
/usr/share/man/man1/ssh-copy-id.1.gz
/usr/share/man/man1/ssh-keyscan.1.gz
/usr/share/man/man1/ssh.1.gz
/usr/share/man/man5/ssh_config.5.gz
/usr/share/man/man8/ssh-pkcs11-helper.8.gz
```

#### （四）ssh基本安全配置

提升服务器ssh安全，就是来修改该配置文件了，先来看简单的配置；

往下继续看，于超老师还准备了ssh优化篇；

```
man 5 sshd_config
[root@web-7 ~]#cat /etc/ssh/sshd_config 

1.禁用root用户登录、降低权限
2.修改端口

配置文件如下
[root@web-7 ~]#grep -E '^(Permit|Port)' /etc/ssh/sshd_config 
Port 22422
PermitRootLogin no
```

##### 4.1 pwgen随机密码生成工具

别忘记创建一个用于登录的普通用户，密码随机生成

```
1.安装
yum install -y pwgen

2.命令参数，生成不同的密码规则

pwgen支持的选项。
  -c或-大写字母
    在密码中至少包含一个大写字母
  -A或--不大写
    不在密码中包含大写字母
  -n 或 --数字
    在密码中至少包含一个数字
  -0 或 --no-numerals
    不在密码中包含数字
  -y或--符号
    在密码中至少包括一个特殊符号
  -r <chars> 或 --remove-chars=<chars>（删除字符
    从生成密码的字符集中删除字符
  -s 或 --secure
    生成完全随机的密码
  -B 或--模棱两可
    不要在密码中包含模棱两可的字符
  -h 或 --help
    打印一个帮助信息
  -H 或 --sha1=path/to/file[#seed] 。
    使用指定文件的sha1哈希值作为（不那么）随机生成器
  -C
    以列形式打印生成的密码
  -1
    不在列中打印生成的密码
  -v或--不使用元音
    不要使用任何元音，以避免意外的讨厌的字。

3.生成 完全随机、携带数字的密码
[root@web-7 ~]#pwgen -sn |head -1
T0AR1i2c

4.设置密码
[root@web-7 ~]#echo T0AR1i2c | passwd --stdin yuchao01
```

随机生成指定风格的密码

打印，包含大写字母、数字、不包含特殊歧义、完全随机、且一行一个密码、密码长度为8、生成5个密码。

```
[root@web-7 ~]#pwgen -cnBs1 8 5
KaKWvp9F
4HHvsWWR
NpKWn3pN
LuqzuAh9
bR7Xg9Hs
```

改了配置文件就得重启

#### （五）服务管理

```
# 重启服务
systemctl restart sshd

# 查看状态
systemctl status sshd
# 进程查看方式
ps aux |grep sshd
# 端口查看方式
netstat -lntp|grep sshd

# 开启自启动
systemctl enable sshd
```

#### （六）登录测试

```
无法登录
[C:\~]$ ssh root@10.0.0.7 22422


可以登录
[C:\~]$ 
[C:\~]$ ssh yuchao01@10.0.0.7 22422


Connecting to 10.0.0.7:22422...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
[yuchao01@web-7 ~]$
[yuchao01@web-7 ~]$echo "超哥带你学Linux"
超哥带你学Linux
```

练习结束，为了方便，可以再改回来。

- 允许root登录
- 默认22端口

## 3、 sshd服务部署小结

- 掌握ssh认证方式
  - ssh通信加密方式原理、流程。
  - 密码认证模式
- 禁止root登录服务器，增强服务器安全性
- 更改ssh服务默认端口，增强服务器安全性
- 密码生成工具，生成随机密码，增强安全性。
- 熟练使用ssh客户端工具，xshell、ssh命令、secureCRT等。

```
密码登录涉及的配置文件

1.客户端的~/.ssh/known_hosts  记录了目标机器，基于ecdsa算法的公钥


2.记录了linux机器的ssh相关文件目录
[root@rsync-41 ~]#ll /etc/ssh/  |awk '{print $NF}'
moduli
ssh_config
sshd_config
ssh_host_ecdsa_key      # 提供ssh密码认证的私钥
ssh_host_ecdsa_key.pub  # 用于生成指纹的，提供ssh密码认证的公钥
ssh_host_ed25519_key
ssh_host_ed25519_key.pub
ssh_host_rsa_key
ssh_host_rsa_key.pub
```