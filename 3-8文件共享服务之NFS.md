#  3-8文件共享服务之NFS

已知samba主要用于linux与windows之间共享文件夹

那用于Linux之间进行文件共享则是用NFS服务（Network FileSystem）

目的在于让不同的机器，不同的操作系统可以彼此分享各自的文件数据。

> NFS服务可以将远程Linux系统上的文件共享资源挂载到本地机器的目录上。

NFS很像Windows系统的网络共享、安全功能、网络驱动器映射，这也和Linux系统的samba服务类似。

一般情况下，Windows网络共享服务或samba服务用语办公局域网共享，而中小型网站集群架构后端通常用NFS数据共享，如果大型网络集群，还会用更复杂的文件系统，如GlusterFS、FastDFS等。

NFS系统已有30年发展历史，代表了一个稳定的网络文件系统，具备可扩展，高性能等特点。

由于网络速度的加快和延迟的减少，NFS系统一直是通过网络提供文件系统的不错的选择，特别是在中小型互联网企业用的广泛。

# NFS在企业的应用架构

在企业集群架构的工作场景中，NFS网络文件系统一般被用来存储共享视频、图片、静态文件，通常网站用户上传的文件也都会放在NFS共享里，例如BBS产品(论坛)产生的图片、附件、头像等，然后前端所有的节点访问静态资源时都会读取NFS存储上的资源。

阿里云等公有云平台的NAS就是云版的NFS服务应用。

像这个奔驰官网，需要展示大量的图片，动态图，html网页文件，这些都是存储在服务器上的。

![image-20220218111225811](/ajian/image-20220218111225811.png)

## 企业生产集群为什么需要共享存储

![img](/ajian/image-20200310095658885.png)

先看一下如果没有共享存储的问题

A用户上传图片到web01服务器，然后用户B访该图片，结果B的请求被负载均衡分发到了Web02，但是由于没有配置共享存储，web02没有该图片，导致用户B看不到该资源，用户心理很不爽呀。

那么如果配置了共享存储，无论A用户上传的图片是发给了web01还是其他，最终都会存储到共享存储上，用户B再访问该图片的时候，无论请求被负载均衡发给了web01、web02、web03最终都会去共享存储上寻找资源，这样也就能够访问到资源了。

这个共享存储对于中小企业，也就是使用服务器配置NFS网络文件共享系统实现。

![image-20200310100922965](/ajian/image-20200310100922965.png)

## NFS工作原理

问题来了，如何部署这个NFS网络文件系统呢？

我们来部署如下的操作

![image-20220218112818072](/ajian/image-20220218112818072.png)

# 任务需求

1.部署一台淘宝网服务器，网页服务器，提供静态网页的展示，该网站的html等静态资源远程存储在NFS服务器。

2.对NFS服务器上的静态资源实时备份（inotify+rsync）

# 任务分析

1.淘宝网使用的nginx技术（web服务器部署技术）

2.部署NFS服务器，创建共享文件夹（提供静态文件），发布该共享目录，提供给web服务器使用。

![image-20220218120526954](/ajian/image-20220218120526954.png)

# 学习目标

- NFS服务部署（掌握）
- mount命令挂载（了解）
- rsync+inotify实现数据实时备份（掌握）

# NFS理论储备

**NFS（Network File System）网络文件系统**

- 主要用于**linux系统**上实现文件共享的一种协议，其客户端**主要是Linux**
- **没有用户认证机制**，且数据在网络上传送的时候是**明文传送**，一般只能在**局域网**中使用
- 支持多节点同时挂载及并发写入

## NFS架构图

**该服务包括的组件：**

- RPC（Remote Procedure Call Protocol）： 远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，不需要了解底层网络技术的协议。
- **rpcbind** //负责NFS的数据传输，远程过程调用 ==tcp/udp协议 端口111==
- **nfs-utils** //控制共享哪些文件,权限管理

![image-20220218135245545](/ajian/image-20220218135245545.png)

如图是企业中NFS服务器和客户端的挂载结构

1.在NFS服务端设置一个共享的目录`/nfs-yuchao-nginx`，其他有权限访问NFS服务器的客户端都可以吧该目录`/nfs-yuchao-nginx`挂载到本地客户端的某个挂载点（挂载点就是一个目录，可以自由定义路径）

2.客户端正确挂载完毕后，进入NFS客户端的挂载点，也就能够看到NFS服务端的`/nfs-yuchao-nginx`共享目录下的数据。在客户端查看时，NFS服务端`/nfs-yuchao-nginx`的目录数据就相当于本地一个目录而已，根本察觉不到任何区别。

## NFS与RPC的关系

我们已知NFS是通过网络来进行数据传输（网络文件系统），因此NFS会使用一些port来传输数据

> 但是NFS在传输数据的时，使用的端口是随机选择（可以重启NFS服务查看端口）。

既然NFS是随机端口选择（好比银行的取钱窗口总发生变化，你知道几号窗口是取钱业务吗？）那么NFS在传输数据的时候，怎么知道NFS服务器使用的端口是哪个呢？

答案就是NFS使用的RPC（Remote Procedure Call，就是远程过程调用）协议来实现的。

## 什么是RPC

RPC（Remote Procedure Call）**远程过程调用**，它是一种通过网络从**远程计算机程序上请求服务**，而不需要了解底层网络技术的协议。

下面举个简单的例子来理解 RPC。

```
远程过程调用`相对应的就是`本地过程调用。
```

同样是洗衣服，一个本地洗，一个远程洗。

- 打个比方，**你在家里**，要洗衣服，你直接把衣服放到洗衣机，开启洗衣机开关，这就是本地过程调用。
- 远程调用就是，**你不在家里**，你打个电话回家里，跟妈说帮忙洗个衣服，这就是远程调用。

![image-20220218140439259](/ajian/image-20220218140439259.png)

## NFS提供的RPC服务

nfs的数据传输是通过rpc协议的。

NFS服务就是使用RPC协议的帮忙，RPC服务实现的功能是记录每个NFS功能对应的端口号，并且在NFS客户端发出请求的时候，把该功能和对应的端口信息传递给发出请求的NFS客户端，保证客户端能够正确的连接到NFS的端口，达到数据传输的目的。

RPC就好比是一个中介，处在客户端、服务端之间。

![image-20220218140915590](/ajian/image-20220218140915590.png)

这就好比超哥要租房，此时超哥就是一个`NFS客户端`

中介来介绍房子信息，中介就好比`RPC服务`

房源拥有者也就相当于`NFS服务端`，提供数据的，中介手里必须得先储备好房东的信息，才能给房源信息转达给租客。

那么RPC服务又是如何知道每个NFS的端口呢？（中介如何知道房东的具体信息呢？）

*当NFS服务器启动时会随机采用若干端口，并且主动在RPC服务中注册相关端口以及功能信息。*

如此一来RPC服务就知道NFS服务对应的端口功能了，RPC服务默认使用固定111端口来监听NFS客户端提交的请求，并将正确的NFS端口信息回复给NFS客户端，这样，NFS客户端就可以和NFS服务器进行数据通信了。

![image-20220218141107810](/ajian/image-20220218141107810.png)

## RPCBIND服务

在启动NFS服务端之前，必须先启动RPC服务，在centos7服务器下为rpcbind服务，否则NFSserver无法向RPC注册信息了。

另外如果RPC服务重启，原来注册的NFS服务端信息也就失效了，也必须重启服务，再次注册信息给RPC服务。

特别要注意的是，修改NFS配置文件后不需要重启NFS，只需要执行`exportfs -rv` 命令即可或是`systemctl reload nfs`

![image-20220218141220317](/ajian/image-20220218141220317.png)

当访问程序通过NFS客户端向NFS服务器端存储文件时，其数据请求流程如下：

1.用户访问网站程序，由程序在NFS客户端上发出存取文件的请求，此时NFS客户端（执行程序的机器）的RPC服务（rpcbind）就会通过网络向NFS服务器的RPC服务的111端口发出NFS文件存取功能的请求。

2.NFS服务器RPC找到对应注册的NFS端口，通知NFS客户端RPC服务

3.此时NFS客户端获取到正确的端口，并与NFS daemon联机存取数据

4.NFS客户端把数据存取成功后返回给前端程序，告知用户存取结果，完成一次存取请求。

这也就证明，必须先启动RPC服务，再启动NFS服务的步骤。

# NFS配置知识储备

学习任何技术都是

1、先学技术点原理、架构流程

2、实践操作

3、部署过程如果出错，回顾原理，分析错误点。

## NFS软件列表

安装nfs服务，需要安装如下软件包

- nfs-utils：NFS服务的主程序，包括了rpc.nfsd、rpc.mountd这两个守护进程以及相关文档，命令
- rpcbind：是centos7/6环境下的RPC程序

```
[root@yuchao-server01 ~]# rpm -qa nfs-utils rpcbind
nfs-utils-1.3.0-0.54.el7.x86_64
rpcbind-0.2.0-44.el7.x86_64


2.如果没有，需要安装nfs和rpc软件。
[root@yuchao-server01 ~]# yum install nfs-utils rpcbind -y
```

## 环境配置

NFS也是C/S模式，准备一个NFS服务端，一个NFS客户端，两台linux机器

在Server机器上创建用于NFS文件共享的文件夹，且设置好权限

```
[root@yuchao-server01 ~]# mkdir /nfs-yuchao-nginx
[root@yuchao-server01 ~]# chmod -Rf 777 /nfs-yuchao-nginx/
[root@yuchao-server01 ~]#
```

## 修改NFS服务的配置文件

默认配置文件路径是`/etc/exports`

```
exports配置文件语法

NFS共享目录  NFS客户端地址(参数1、参数2...) 客户点地址2（参数1、参数2...）

例如
/        hostname1(rw)  hostname2(rw,no_root_squash)
/pub   *(rw)
/home/chao   123.206.16.61(ro)

参数解释
1.NFS共享目录：为NFS服务器要共享的实际目录，必须绝对路径，注意目录的本地权限，如果要读写共享，要让本地目录可以被NFS客户端的(nfsnobody)读写
2.NFS客户端地址，也就是NFS服务器端授权可以访问共享目录的客户端地址，详见下表
3.权限参数，对授权的NFS客户端访问权限设置，见下表
```

nfs客户端地址说明

| 客户端地址         | 具体地址         | 说明                                            |
| ------------------ | ---------------- | ----------------------------------------------- |
| 单一客户端         | 192.168.178.142  | 用的少                                          |
| 整个网段           | 192.168.178.0/24 | 24表示子网掩码255.255.255.0，指定网段，用的较多 |
| 授权域名客户端     | nfs.yuchaoit.cn  | 弃用                                            |
| 授权整个域名客户端 | *.yuchaoit.cn    | 弃用                                            |

## 配置参考

案例1

```
#修改nfs配置文件为如下示例
[root@chaogelinux nfsShare]# cat /etc/exports
/nfsShare *(insecure,rw,sync,root_squash)

#表示共享该文件夹，且提供给所有网段的机器可访问
配置规则是，可读写，数据同步写入到磁盘，把root管理员映射为本地的匿名用户，insecure是客户端从大于1024的端口发送链接
```

案例2

```
/home/chaoge/log   192.168.178.0/24(ro)
只读共享，例如一些生成服务器的日志目录，又不想给开发服务器的权限，可以用此办法，共享目录给他人只读查看
```

## nfs配置参数详解

```
ro 只读
rw 读写
root_squash 当nfs客户端以root访问时，它的权限映射为NFS服务端的匿名用户，它的用户ID/GID会变成nfsnobody
no_root_squash 同上，但映射客户端的root为服务器的root，不安全，避免使用
all_squash 所有nfs客户端用户映射为匿名用户，生产常用参数，降低用户权限，增大安全性。
sync 数据同步写入到内存与硬盘，优点数据安全，缺点性能较差
async 数据写入到内存，再写入硬盘，效率高，但可能内存数据会丢


/etc/exports  man 5 exports

共享目录        共享选项
/nfs/share      *(ro,sync)

共享主机：
*   ：代表所有主机
192.168.0.0/24：代表共享给某个网段
192.168.0.0/24(rw) 192.168.1.0/24(ro) :代表共享给不同网段
192.168.0.254：共享给某个IP
*.yuchaoit.com:代表共享给某个域下的所有主机

共享选项：
ro：只读
rw：读写
sync：实时同步，直接写入磁盘
async：异步，先缓存在内存再同步磁盘
anonuid：设置访问nfs服务的用户的uid，uid需要在/etc/passwd中存在
anongid：设置访问nfs服务的用户的gid
root_squash ：默认选项 root用户创建的文件的属主和属组都变成nfsnobody,其他人nfs-server端是它自己，client端是nobody。
no_root_squash：root用户创建的文件属主和属组还是root，其他人server端是它自己uid，client端是nobody。
all_squash： 不管是root还是其他普通用户创建的文件的属主和属组都是nfsnobody

说明：
anonuid和anongid参数和all_squash一起使用。
all_squash表示不管是root还是其他普通用户从客户端所创建的文件在服务器端的拥有者和所属组都是nfsnobody；服务端为了对文件做相应管理，可以设置anonuid和anongid进而指定文件的拥有者和所属组
```

## 启动NFS服务端

NFS服务都是基于RPC协议通信的默认端口是111，要确保系统运行了rpcbind服务

要注意的是`rpcbind服务`即使停止，111端口也不会挂掉，因为还有`rpcbind.socket`服务

> 意思是，启动rpc服务由2个结合运行
>
> rpcbind.service
>
> rpcbind.socket

```
[root@chaogelinux ~]# systemctl status rpcbind
● rpcbind.socket - RPCbind Server Activation Socket
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.socket; enabled; vendor preset: enabled)
   Active: active (running) since 二 2020-03-10 10:59:12 CST; 4h 35min ago
   Listen: /var/run/rpcbind.sock (Stream)
           0.0.0.0:111 (Stream)
           0.0.0.0:111 (Datagram)

3月 10 10:59:12 chaogelinux systemd[1]: Listening on RPCbind Server Activation Socket.


# 启动rpcbind服务
systemctl restart rpcbind

#启动
systemctl restart nfs-server
```

# 部署NFS服务端（重要）

创建共享目录，部署nfs服务端

```
1.安装部署nfs服务端
[root@yuchao-server01 ~]# yum install nfs-utils rpcbind -y

2.启动rpcbind服务
[root@yuchao-server01 ~]# systemctl start rpcbind

3.创建共享目录，以及创建共享的文件
[root@yuchao-server01 ~]# mkdir -p /nfs-yuchao-nginx/
[root@yuchao-server01 ~]# echo '<meta charset=utf8> 和于超老师学NFS没毛病' > /nfs-yuchao-nginx/index.html

4.修改目录权限
[root@yuchao-server01 ~]# chown -R nfsnobody.nfsnobody /nfs-yuchao-nginx/

5.查看该用户信息
[root@yuchao-server01 ~]# grep nfsnobody /etc/passwd
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin

6.修改nfs配置文件，设置共享文件的规则
insecure是客户端从大于1024的端口发送链接
all_squash： 不管是root还是其他普通用户创建的文件的属主和属组都是nfsnobody
rw 允许读写
sync 数据同步到磁盘
[root@yuchao-server01 ~]# cat /etc/exports
/nfs-yuchao-nginx 192.168.0.0/24(all_squash,insecure,rw,sync)

7.重新加载nfs服务，查看rpcbind进程，是否出现了111端口
[root@yuchao-server01 ~]# systemctl restart  nfs
[root@yuchao-server01 ~]# netstat -tnlp|grep rpc
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      21687/rpcbind
tcp        0      0 0.0.0.0:20048           0.0.0.0:*               LISTEN      22232/rpc.mountd
tcp        0      0 0.0.0.0:57594           0.0.0.0:*               LISTEN      22224/rpc.statd
tcp6       0      0 :::111                  :::*                    LISTEN      21687/rpcbind
tcp6       0      0 :::20048                :::*                    LISTEN      22232/rpc.mountd
tcp6       0      0 :::39764                :::*                    LISTEN      22224/rpc.statd

8.查看nfs服务端共享情况
[root@yuchao-server01 ~]# showmount -e
Export list for yuchao-server01:
/nfs-yuchao-nginx 192.168.0.0/24

9.查看nfs服务端远程共享的所有参数，是系统自动生成的，以及我们配置文件里定义的，都是默认的不需要了解太多
[root@yuchao-server01 ~]# cat /var/lib/nfs/etab
/nfs-yuchao-nginx    192.168.0.0/24(rw,sync,wdelay,hide,nocrossmnt,insecure,root_squash,all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,insecure,root_squash,all_squash)

10.把nfs服务端本地当做一个客户端，本地挂载该共享目录访问试试
挂载后，的确访问到了该共享目录的数据。
[root@yuchao-server01 ~]# mount -t nfs 192.168.0.110:/nfs-yuchao-nginx /mnt
[root@yuchao-server01 ~]#
[root@yuchao-server01 ~]# ls /mnt
index.html
[root@yuchao-server01 ~]# cat /mnt/index.html
<meta charset=utf8> 和于超老师学NFS没毛病

11.取消挂载，也就看不到数据了。
[root@yuchao-server01 ~]# umount /mnt
[root@yuchao-server01 ~]# cat /mnt/index.html
cat: /mnt/index.html: No such file or directory


至此超哥带你部署NFS服务端，完全没毛病了。
```

注意：

- `/etc/exports文件的语法不要写错，细心`
- 修改/etc/exports文件后，注意要重启`systemctl reload nfs`或是`exportfs -r`重新加载配置，无需重启

# 部署NFS客户端（重要）

```
1.准备好另外一台linux机器，安装nfs工具包
[root@yuchao-client01 ~]# yum install nfs-utils rpcbind -y

2.确保rpc服务正常
[root@yuchao-client01 ~]# systemctl status rpcbind
● rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-02-18 15:17:46 CST; 54s ago
  Process: 71676 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 71677 (rpcbind)
    Tasks: 1
   CGroup: /system.slice/rpcbind.service
           └─71677 /sbin/rpcbind -w

Feb 18 15:17:46 yuchao-client01 systemd[1]: Starting RPC bind service...
Feb 18 15:17:46 yuchao-client01 systemd[1]: Started RPC bind service.
Hint: Some lines were ellipsized, use -l to show in full.
[root@yuchao-client01 ~]#

3.远程查看共享文件信息
[root@yuchao-client01 ~]# showmount -e 192.168.0.110
Export list for 192.168.0.110:
/nfs-yuchao-nginx 192.168.0.0/24

4.进行挂载查看共享文件信息
[root@yuchao-client01 ~]# mount -t nfs 192.168.0.110:/nfs-yuchao-nginx /mnt
验证挂载
[root@yuchao-client01 ~]# mount -l |grep mnt
192.168.0.110:/nfs-yuchao-nginx on /mnt type nfs4 (rw,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.123,local_lock=none,addr=192.168.0.110)

5.进入共享文件夹，读写操作
[root@yuchao-client01 ~]# cd /mnt/
[root@yuchao-client01 mnt]# cat index.html
<meta charset=utf8> 和于超老师学NFS没毛病，哎对对对
```

![image-20220218152259655](/ajian/image-20220218152259655.png)

## 部署nginx网站服务

```
1.安装nginx软件，必须配置好epel源
[root@yuchao-client01 mnt]# yum install nginx -y

2.查看nginx网页目录的默认文件
[root@yuchao-client01 mnt]# ls /usr/share/nginx/html/
404.html  50x.html  en-US  icons  img  index.html  nginx-logo.png  poweredby.png


3.挂载nfs共享文件夹到nginx的网页目录，让nginx可以读取到nfs的共享数据
执行mount挂载后，文件夹的内容会被隐藏，显示nfs共享的数据。
[root@yuchao-client01 mnt]# ls /usr/share/nginx/html/
404.html  50x.html  en-US  icons  img  index.html  nginx-logo.png  poweredby.png
[root@yuchao-client01 mnt]#
[root@yuchao-client01 mnt]#
[root@yuchao-client01 mnt]# mount -t nfs 192.168.0.110:/nfs-yuchao-nginx /usr/share/nginx/html/
[root@yuchao-client01 mnt]#
[root@yuchao-client01 mnt]# ls /usr/share/nginx/html/
index.html  我是超哥创建的客户端文件.txt  超哥牛啊
[root@yuchao-client01 mnt]#


4.启动nginx，查看网页
[root@yuchao-client01 mnt]# systemctl start nginx

5.查看nginx端口，进程
[root@yuchao-client01 mnt]# ps -ef|grep nginx
root      71998      1  0 15:29 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx     71999  71998  0 15:29 ?        00:00:00 nginx: worker process
nginx     72000  71998  0 15:29 ?        00:00:00 nginx: worker process
nginx     72001  71998  0 15:29 ?        00:00:00 nginx: worker process
nginx     72002  71998  0 15:29 ?        00:00:00 nginx: worker process
root      72004  71365  0 15:29 pts/1    00:00:00 grep --color=auto nginx
[root@yuchao-client01 mnt]#
[root@yuchao-client01 mnt]#
[root@yuchao-client01 mnt]# netstat -tnlp|grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      71998/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      71998/nginx: master

6.用浏览器访问该机器的ip:80端口

http://192.168.0.123/


7.或者再linux中访问，需要安装elinks工具，是一个linux下文本型浏览器
[root@yuchao-client01 mnt]# yum install elinks -y
```

![image-20220218153013271](/ajian/image-20220218153013271.png)

elinks浏览器，按q退出，按esc显示菜单。

![image-20220218153523693](/ajian/image-20220218153523693.png)

## 更新网页内容

1.我们客户端的nginx页面，是来自于nfs服务端的

2.更新nfs共享数据，客户端的nginx页面级会变化。

```
1.修改nfs服务端文件
[root@yuchao-server01 ~]# cat  /nfs-yuchao-nginx/index.html
<meta charset=utf8> 和于超老师学NFS没毛病，哎对对对
<h1>超哥带你学linux，冲冲冲！！！</h1>
[root@yuchao-server01 ~]#

2.再次访问客户端的nginx网页
```

![image-20220218153809846](/ajian/image-20220218153809846.png)

# 任务2，实时备份

当网页文件发生了变化，我们要对数据做好备份，因此想起来超哥教的Inotify+rsync。

```
1.nfs服务端机器上，安装inotify工具
[root@yuchao-server01 ~]# yum install inotify-tools -y

2.安装rsync程序
[root@yuchao-server01 ~]# yum install inotify-tools rsync -y

3.编写脚本，完成数据同步
检测共享文件夹，只要有了数据变化，立即触发rsync备份
这里大家可以自行再部署一个rsync备份服务器，看前面超哥的文档即可。
这里超哥就直接本地备份演示了，创建一个备份目录
mkdir /nginx-html-bak

脚本内容
[root@yuchao-server01 rsync_sh]# cat rsync_nginx.sh
#!/bin/bash
/usr/bin/inotifywait -mrq -e modify,delete,create,attrib,move  /nfs-yuchao-nginx/  | while read line
do
    rsync -a --delete  /nfs-yuchao-nginx/  /nginx-html-bak
    echo "`date +%F\ %T`出现事件$line" >> /var/log/rsync.log 2>&1
done





4.执行脚本，放入后台运行
[root@yuchao-server01 rsync_sh]# bash rsync_nginx.sh &
[1] 23533
[root@yuchao-server01 rsync_sh]# jobs
[1]+  Running                 bash rsync_nginx.sh &

5.检测日志
[root@yuchao-server01 rsync_sh]# tail -f /var/log/rsyncd.log
```

客户端操作

```
1.修改nfs共享文件的html
[root@yuchao-client01 mnt]# cat index.html
<meta charset=utf8> 和于超老师学NFS没毛病，哎对对对
<h1>超哥带你学linux，冲冲冲！！！</h1>
<h2>超哥带你学nfs+inotify+rsync</h2>
```

![image-20220218155836587](/ajian/image-20220218155836587.png)

# 超哥提示

如果过程里出现了失败，大部分原因是

1.防火墙未关

2.nfs配置权限不对

3.网络不通

其他就是看报错信息，对症下药。

# 开机自动挂载nfs

如果机器重启，nfs这个mount挂载动作会失效，导致看不到网页，我们来看看效果

```
1.客户端机器，取消挂载nfs
[root@yuchao-client01 mnt]# umount /usr/share/nginx/html/
[root@yuchao-client01 mnt]#
```

![image-20220218160616019](/ajian/image-20220218160616019.png)

```
2.因此要做好开机自动就挂载，虽然生产环境下一般不会重启，但是这事你得做。
想清楚，是在哪台机器上做好挂载东西？客户端机器！
注意语法
[root@yuchao-client01 mnt]# tail -1 /etc/fstab
192.168.0.110:/nfs-yuchao-nginx /usr/share/nginx/html/ nfs defaults 0 0


3.取消挂载
[root@yuchao-client01 mnt]# umount /usr/share/nginx/html/
[root@yuchao-client01 ~]# mount -l |grep nginx

4.重启机器，查看自动挂载
reboot

5.重启查看挂载情况，已经自动挂载了
[root@yuchao-client01 ~]# mount -l |grep nginx
192.168.0.110:/nfs-yuchao-nginx on /usr/share/nginx/html type nfs4 (rw,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.123,local_lock=none,addr=192.168.0.110)

6.再次启动nginx
[root@yuchao-client01 ~]# systemctl start nginx

7.再次访问nginx网页
http://192.168.0.123/
```

![image-20220218162720864](/ajian/image-20220218162720864.png)

# Autofs服务（扩展知识）

## 【为什么要用autofs】

Linux的mount命令用于挂载文件系统

- 对于`本地固定`的设备，例如硬盘分区可以使用mount进行挂载；
- 对于网络文件系统，如nfs可以使用mount进行挂载；

以及超哥也讲了`/etc/fstab`文件，可以实现开机自动挂载。

但是

在`/etc/fstab`文件中，如果定义了过多的自动挂载配置，无疑都会随着服务器开机而进行挂载，但是这些挂载后的资源，我们都一定会使用吗？如果不用则会给服务器造成硬件资源压力，以及网络带宽压力。

> 因此一些具有`动态特性`的文件系统，如`光盘、软盘、U盘、甚至NFS、SMB`等文件系统，特点是当需要，且使用的时候才有必要挂载。

当光盘或是U盘需要使用的时候，我们即可插入服务器，进行相应的挂载即可，但是NFS或SMB这样的远程共享，我们就不一定知道何时使用，进行挂载，何时不用，也不造成资源浪费。

## 【autofs特点】

Autofs和mount的不同点在于，Autofs是一种守护进程。它在后台检测用户是否要访问一个还没有挂载的文件系统，autofs会自动检查该文件系统是否存在，存在则自动挂载。

且autofs检测到已经挂载的文件系统有一段时间没用，则会自动将其卸载，省去了人力维护挂载设备的成本，以及不会造成服务器资源浪费。

## 【autofs的缺点】

autofs特点是只有用户请求时才执行挂载，所以当高并发访问时，开始请求的瞬间需要执行挂载，性能较差，因此在高并发业务场景下，宁愿保持挂载也不使用autofs自动挂载。

### 安装autofs

我们应该是在`需要挂载`的那台机器执行

```
[root@yuchao-client01 ~]#  yum install autofs -y
```

【修改autofs配置文件】

autofs配置文件以`挂载点 子配置文件`的格式填写

```
autofs默认配置文件是
[root@yuchao-client01 ~]# grep -v '^#' /etc/auto.master
# 这一行的意思是
# 定义了一个挂载点是/misc ，需要在auto.misc文件中定义挂载动作
语法是

挂载点   配置文件路径
/misc    /etc/auto.misc
/net    -hosts
+dir:/etc/auto.master.d
+auto.master
```

修改配置文件，添加我们想要自动挂载的目录信息

如何写配置文件，参考`/etc/auto.misc`即可。

```
[root@yuchao-client01 ~]# vim /etc/auto.master
写入如下配置
配置表示 不在这里写入挂载点，在后面配置文件里指定
# by yuchao ,about nfs
/- /etc/auto.nfs


创建auto.nfs配置文件
[root@yuchao-client01 ~]# cat  /etc/auto.nfs
/usr/share/nginx/html/ -ro,soft,intr 192.168.0.110:/nfs-yuchao-nginx


挂载点   参数  远程nfs共享目录

参数解释 -ro 只读
soft 使用软挂载的方式挂载系统，若Client的请求得不到回应，则重新请求并传回错误信息。
intr 允许NFS中断文件操作和向调用它的程序返回值，默认不允许文件操作被中断。
```

检查当前的挂载情况（取消掉nfs的挂载，用于测试）

```
[root@yuchao-client01 ~]# mount -l |grep nginx
192.168.0.110:/nfs-yuchao-nginx on /usr/share/nginx/html type nfs4 (rw,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.123,local_lock=none,addr=192.168.0.110)
[root@yuchao-client01 ~]#
[root@yuchao-client01 ~]#
[root@yuchao-client01 ~]# umount /usr/share/nginx/html/
```

### 启动autofs

```
[root@yuchao-client01 ~]# systemctl restart autofs
[root@yuchao-client01 ~]#
[root@yuchao-client01 ~]# systemctl enable autofs
Created symlink from /etc/systemd/system/multi-user.target.wants/autofs.service to /usr/lib/systemd/system/autofs.service.

[root@yuchao-client01 ~]# systemctl status autofs
● autofs.service - Automounts filesystems on demand
   Loaded: loaded (/usr/lib/systemd/system/autofs.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-02-18 17:17:03 CST; 1min 42s ago
 Main PID: 3360 (automount)
   CGroup: /system.slice/autofs.service
           └─3360 /usr/sbin/automount --systemd-service --dont-check-daemon

Feb 18 17:17:03 yuchao-client01 systemd[1]: Starting Automounts filesystems on demand...
Feb 18 17:17:03 yuchao-client01 automount[3360]: setautomntent: lookup(sss): setautomntent: No such file...tory
Feb 18 17:17:03 yuchao-client01 systemd[1]: Started Automounts filesystems on demand.
Hint: Some lines were ellipsized, use -l to show in full.
```

### 取消开机自动挂载配置

```
修改 /etc/fstab 即可
```

## 查看自动挂载效果

1.确保当前没有挂载

```
1.关闭autofs服务
[root@yuchao-client01 ~]# systemctl stop autofs

[root@yuchao-client01 ~]#
[root@yuchao-client01 ~]# mount -l |grep nginx
[root@yuchao-client01 ~]#


2.修改网站内容，查看挂载生效效果（注意，是在哪改html文件，能搞得清吗？）
[root@yuchao-server01 ~]# cat  /nfs-yuchao-nginx/index.html
<meta charset=utf8> 和于超老师学NFS没毛病，哎对对对<h1>超哥带你学linux，冲冲冲！！！</h1><h2>超哥带你学nfs+inotify+rsync</h2><h1>autofs我也学会了，可把我牛逼坏了</h1>
```

![image-20220218172007949](/ajian/image-20220218172007949.png)

2.当你开启了autofs，只要进入该自动挂载的目录（只要你一访问nginx网站，会自动访问该目录），会立即自动挂载

```
[root@yuchao-client01 ~]# systemctl restart autofs
```

3.查看网页效果

![image-20220218172913835](/ajian/image-20220218172913835.png)

最后，可以设置自动取消挂载的时间。

```
[root@yuchao-client01 ~]# cat /etc/autofs.conf |grep -i '^timeout ='
timeout = 10
[root@yuchao-client01 ~]#
[root@yuchao-client01 ~]# systemctl restart autofs

10秒之后，自动取消挂载了
[root@yuchao-client01 ~]# mount -l |grep nginx
/etc/auto.nfs on /usr/share/nginx/html type autofs (rw,relatime,fd=12,pgrp=4142,timeout=10,minproto=5,maxproto=5,direct,pipe_ino=57248)

当你访问了该目录，自动挂载


[root@yuchao-client01 ~]# ls /usr/share/nginx/html/
index.html  我是超哥创建的客户端文件.txt  超哥牛啊
[root@yuchao-client01 ~]#

发现自动挂载了

[root@yuchao-client01 ~]# mount -l |grep nginx
/etc/auto.nfs on /usr/share/nginx/html type autofs (rw,relatime,fd=12,pgrp=4142,timeout=10,minproto=5,maxproto=5,direct,pipe_ino=57248)
192.168.0.110:/nfs-yuchao-nginx on /usr/share/nginx/html type nfs4 (ro,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,soft,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.123,local_lock=none,addr=192.168.0.110)
[root@yuchao-client01 ~]#
```

# 作业

1.掌握NFS与RPC通信流程

2.掌握NFS部署c/s模式

3.掌握NFS+nginx部署网页

4.完成inotify+rsync+NFS数据实时备份

5.完成autofs自动挂载NFS。
