# linux命令入门

## 为什么要学Linux命令

- Linux从诞生就是黑屏界面，所有操作倚靠命令完成，如磁盘读写、文件操作、网络管理等
- 企业中，服务器的维护工作都是`ssh客户端`完成，没有图形界面
- 程序员想要管理linux服务器，必须学习常用命令

*Linux命令学习方法*

- 熟能生巧，多敲打，多练习即可
- 不可能一下子掌握所有命令用法，学会使用搜索引擎查阅命令资料

**当年超哥在一家美资企业，一位台湾老程序员送我的一本书。。。**

**可能是看我骨骼惊奇吧！！**

<img src="\ajian/253.png" alt="img" style="zoom:33%;" />

## 打开终端，输入命令

<img src="\ajian/image-20220104175259752.png" alt="image-20220104175259752" style="zoom:33%;" />

终端就是一个可以让你操作的地方，你打开终端，就可以输入指令，发给操作系统。

<img src="\ajian/image-20220104180116480.png" alt="image-20220104180116480" style="zoom:33%;" />

## 什么是linux命令

就是你在linux系统里，输入linux系统才能识别的一些指令，你输入的一些固定存在的单词字母，就是命令。

<img src=\ajian/image-20220104180341275.png" alt="image-20220104180341275" style="zoom:33%;" />

------

<img src="\ajian/243.png" alt="img" style="zoom:33%;" />

前面咱们已经成功安装了Linux系统--centos7，那么现在跟着超哥奔向Linux命令行的世界。

## Linux命令格式

<img src="\ajian/244.jpg" alt="img" style="zoom:33%;" />

1.一般情况下，【参数】是可选的，一些情况下【文件或路径】也是可选的

2.参数 ， 同一个命令，跟上不同的参数执行不同的功能

3.执行linux命令，添加参数的目的是让命令更加贴切实际工作的需要！

4.linux命令，参数之间，普遍应该用一个或多个空格分割！

```
1.完整的命令格式
2.没有参数，没有对象
3.没有参数，有对象
```

## Linux命令提示符

<img src="\ajian/245.png" alt="img" style="zoom: 50%;" />

```
命令提示符
[py@pylinux ~]$            普通用户py，登陆后

[root@pylinux ~]#        超级用户root，登录后

root代表当前登录的用户

@ 分隔符

pylinux 主机名

~  当前的登录的位置，此时是家目录

# 超级用户身份提示符
$ 普通用户身份提示符
```

### 美化命令提示符

```
[yuchao-linux-242 root /opt]#tail -2 /etc/profile
# 美化PS1显示
export PS1='\[\033[01;35m\][\[\033[01;32m\]\h\[\033[01;31m\] \u \w\[\033[31m\]\[\033[01;35m\]]\[\033[01;36m\]\$\[\033[00m\]'

[yuchao-linux-242 root /opt]#source /etc/profile
```

## Tab命令补全

<img src="\ajian/image-20220104200252888.png" alt="image-20220104200252888" style="zoom: 33%;" />

linux终端里，系统为了让你更省心，不用记忆那么多内容可以让你输入Tab键，可以快速列出你想要的内容，你只需要输入一个命令的开头即可，它一般可以补全linux命令，以及文件路径。

<img src="\ajian/image-20220104181435847.png" alt="image-20220104181435847" style="zoom:50%;" />

```
[yuchao@localhost Desktop]$ ls
ls        lsblk     lscpu     lshw      lsipc     lslogins  lsmd      lsmod     lsof      lsscsi    lsusb     
lsattr    lscgroup  lsdiff    lsinitrd  lslocks   lsmcli    lsmem     lsns      lspci     lssubsys  lsusb.py
```

## Linux命令初体验

### su命令

用于切换系统不同的用户。

如windows的用户切换，从老王用户，切换到小李用户。

<img src="\ajian/image-20220104181947377.png" alt="image-20220104181947377" style="zoom:50%;" />

linux用户的切换，并且我们直接切换到超级用户，切换到皇帝身份。

```bash
# 短横线 -   表示切换用户且加载该用户的环境变量，且进入该用户家目录
[yuchao@localhost Desktop]$ 
[yuchao@localhost Desktop]$ su - root
Password: 
[root@localhost ~]#
```

### uname命令

在使用uname命令时，一般会固定搭配上-a参数来完整地查看当前系统的内核名称、主机名、内核发行版本、节点名、系统时间、硬件名称、硬件平台、处理器类型以及操作系统名称等信息。

```bash
1.作用：
    显示系统信息
2.语法：
    uname -参数

# 
[root@localhost opt]# uname -a
Linux localhost.localdomain 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

# 解释
Linux  系统名称

localhost.localdomain 主机名
 3.10.0-862.el7.x86_64 内核版本号
     3 是主版本号，重大变化，才会变动
     10 是次版本号，新加功能后版本数字变化
     0 表示修改次数
     862表示编译次数
     el7 表示是7代版本
     x86_64 表示系统是64位的

发布时间
GNU/Linux是操作系统名称，开源计划
```

### ls命令

功能：平铺显示目录下的文件列表

语法：

```
1.完整的命令格式
ls -l /

2.没有参数，没有对象，其实默认有一个点
ls

3.没有参数，有对象
ls /
```

> 关于路径

linux命令操作，关于要操作不同的文件，主要就分，绝对路径、相对路径

> 关于参数（选项）

```
ls -l  # 使用较长格式列出信息
ls -h  #-h, --human-readable          与-l 一起，以易于阅读的格式输出文件大小，kb、mb、gb
ls -a  # 显示隐藏文件 ，linux下以.开头的文件，表示是隐藏的
```

单位换算

```
# 1TB = 1024GB
# 1GB = 1024MB
# 1MB = 1024KB
# 1KB（千字节） = 1024B（字节）
```

### pwd命令

```
功能、语法
pwd(Print Working Directory) 显示当前目录
```

### cd命令

```
主要功能：cd全称change directory，切换目录（从一个目录跳转到另外一个目录）

基本语法：
cd 涉及文件操作了，那就存在绝对，相对路径了


# 练习
[yuchao@localhost ~]$ 
[yuchao@localhost ~]$ pwd
/home/yuchao
[yuchao@localhost ~]$ cd /opt
[yuchao@localhost opt]$ ls
rh
[yuchao@localhost opt]$ 
[yuchao@localhost opt]$ cd ../home/yuchao
[yuchao@localhost ~]$ 
[yuchao@localhost ~]$ 
[yuchao@localhost ~]$ pwd
/home/yuchao

[yuchao@localhost ~]$ cd -
/opt
[yuchao@localhost opt]$ 
[yuchao@localhost opt]$ 
[yuchao@localhost opt]$ cd ~
[yuchao@localhost ~]$ 
[yuchao@localhost ~]$
```

### 特殊目录

```
- 上一次目录
~ 当前用户家目录
. 当前位置
.. 上一层位置
```

### clear

```
clear 指令用来清除终端屏幕，在终端中通过快捷键 Ctrl+L 清除屏幕
```

### shutdown

```
shutdown 以一种安全的方式关闭系统。
-r
    重启。
-h
    停机。

# 练习
[root@linux ~]# shutdown –h now   #关机 

[root@linux ~]# shutdown –h 23:00   #晚上11点关机

[root@linux ~]# shutdown –r now   #重启

[root@linux ~]# shutdown –r +30 'reboot now'   #30分钟后重启，并且提示reboot now
```

### man命令

当你忘记了某个命令怎么用，以及参数有哪些，都是什么作用，该怎么查？

查于超老师的笔记？还是咋办？

```
man 命令，查看linux命令的帮助手册，以后慢慢看，且是英文的，知道这个方式即可，可以复制出来，翻译一下也行。
```

### history

```
history 命令可以用来显示曾执行过的命令，且显示条目有数量限制，默认是1000个，可修改，超哥以后再说

[yuchao@localhost ~]$ echo $HISTSIZE
1000


# 显示所有历史记录
[yuchao@localhost ~]$ history 

# 显示最后2条
[yuchao@localhost ~]$ history 2
    2  cd
    3  history 2

# 清空历史
[root@redhat ~]# history -c    #清除历史记录
```

### w命令

<img src="\ajian/image-20220308194534805.png" alt="image-20220308194534805" style="zoom:50%;" />

### hostnamectl命令

```
记忆方式：hostname + control 主机名控制的意思
作用：它是用来修改主机名称

语法
[root@10 ~]# hostnamectl --help
hostnamectl [OPTIONS...] COMMAND ...

Query or change system hostname.

  -h --help              Show this help
     --version           Show package version
     --no-ask-password   Do not prompt for password
  -H --host=[USER@]HOST  Operate on remote host
  -M --machine=CONTAINER Operate on local container
     --transient         Only set transient hostname
     --static            Only set static hostname
     --pretty            Only set pretty hostname

Commands:
  status                 Show current hostname settings
  set-hostname NAME      Set system hostname
  set-icon-name NAME     Set icon name for host
  set-chassis NAME       Set chassis type for host
  set-deployment NAME    Set deployment environment for host
  set-location NAME      Set location for host
```

> centos7系统里有3类主机名

```
 --transient         Only set transient hostname
 --static            Only set static hostname
 --pretty            Only set pretty hostname
```

- 静态主机名，static，关机后，重启后，名字依然存在，因为信息写入了/etc/hostname文件，每次开机都会再读取该文件内容。
- 临时主机名，transient，关机，重启后，你设置的主机名失效。
- 优雅主机名，pretty，可以让主机名显示更好看，如一些特殊符号。

> 如何查看主机名，更改主机名
>
> （注意切换到root用户）

1.查看主机名

```
# 可以看到静态主机名
# 临时主机名，是系统默认读取的网络信息
# 还有系统平台信息，主机版本信息，内核版本


[root@10 ~]# 
[root@10 ~]# hostnamectl 
   Static hostname: localhost.localdomain
Transient hostname: 10.96.0.145
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 340cf8a8efa8496494a0e9d7388e5f1f
           Boot ID: 8bd702c0ae3b42248b789515e52def50
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10
```

2.设置静态主机名（基本用这个），永久生效

设置了永久主机名后，发现临时名字都没了

```
[root@10 ~]# hostnamectl set-hostname linux02.yuchaoit.cn
[root@10 ~]# 
[root@10 ~]# hostnamectl 
   Static hostname: linux02.yuchaoit.cn
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 340cf8a8efa8496494a0e9d7388e5f1f
           Boot ID: 0c2d37e952104524a0f1e58fdf33c2b5
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.el7.x86_64
      Architecture: x86-64
[root@10 ~]#
```

3.配置优雅主机名

用在什么情况下？

当你的主机名，有特殊字符的话，主机名会自动修改你的格式

```
[root@10 ~]# hostnamectl set-hostname linux02.yuchaoit.cn
[root@10 ~]# 
[root@10 ~]# hostnamectl 
   Static hostname: linux02.yuchaoit.cn
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 340cf8a8efa8496494a0e9d7388e5f1f
           Boot ID: 0c2d37e952104524a0f1e58fdf33c2b5
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.el7.x86_64
      Architecture: x86-64
[root@10 ~]# 
[root@10 ~]# hostnamectl --pretty set-hostname "yuchao's linux01 server"
[root@10 ~]# 
[root@10 ~]# hostnamectl 
   Static hostname: linux02.yuchaoit.cn
   Pretty hostname: yuchao's linux01 server
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 340cf8a8efa8496494a0e9d7388e5f1f
           Boot ID: 0c2d37e952104524a0f1e58fdf33c2b5
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.el7.x86_64
      Architecture: x86-64

[root@10 ~]# hostnamectl --pretty
yuchao's linux01 server
[root@10 ~]# 
[root@10 ~]# hostnamectl --static
linux02.yuchaoit.cn


# 如果你设置了不规则主机名，系统会去除特殊符号
[root@10 ~]# hostnamectl set-hostname "yuchao's linux01 server"
[root@10 ~]# 
[root@10 ~]# 
[root@10 ~]# hostnamectl 
   Static hostname: yuchaoslinux01server
   Pretty hostname: yuchao's linux01 server
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 340cf8a8efa8496494a0e9d7388e5f1f
           Boot ID: 0c2d37e952104524a0f1e58fdf33c2b5
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.el7.x86_64
      Architecture: x86-64
[root@10 ~]#
```

# 扩展：安装VMware Tools（可忽略）

VMware Tools 是一套可以提高虚拟机客户机操作系统性能并改善虚拟机管理的实用工具。

例如，下面是在安装 VMware Tools 后才能使用的一部分功能：

- 显著提高了图形性能，并在支持 Windows Aero 的操作系统上提供 Aero
- Unity 功能；允许在主机桌面上显示虚拟机中的应用程序，就像任何其他应用程序窗口一样
- 在主机与客户机文件系统之间共享文件夹
- 在虚拟机和主机或客户端桌面之间拷贝和粘贴文本、图形和文件
- 提高了鼠标性能
- 将虚拟机中的时钟与主机或客户端桌面上的时钟进行同步
- 使用脚本帮助自动完成客户机操作系统操作
- 为虚拟机启用客户机自定。

安装步骤

1.点击，选择安装vmware tools

<img src="\ajian/image-20220105102838837.png" alt="image-20220105102838837" style="zoom: 67%;" />

2.打开命令行终端

<img src="\ajian/image-20220105105303421.png" alt="image-20220105105303421" style="zoom:67%;" />

1. 进入vmware tools安装路径，把文件复制出来

```
[root@10 ~]# cd /run/media/
[root@10 media]# ls
yuchao
[root@10 media]# cd yuchao/
[root@10 yuchao]# ls
VMware Tools
[root@10 yuchao]# cd VMware\ Tools/
[root@10 VMware Tools]# pwd
/run/media/yuchao/VMware Tools
[root@10 VMware Tools]# ls
manifest.txt  run_upgrader.sh  VMwareTools-10.3.10-13959562.tar.gz  vmware-tools-upgrader-32  vmware-tools-upgrader-64


# 复制这里的一个安装包
[root@10 VMware Tools]# cp VMwareTools-10.3.10-13959562.tar.gz /root
[root@10 VMware Tools]# 
[root@10 VMware Tools]# cd /root
[root@10 ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg  VMwareTools-10.3.10-13959562.tar.gz

# 解压缩安装
[root@10 ~]# 
[root@10 ~]# tar -xvf VMwareTools-10.3.10-13959562.tar.gz 

# 进入目录，安装程序，一路回车，全部默认即可
[root@10 ~]# cd vmware-tools-distrib/
[root@10 vmware-tools-distrib]# ./vmware-install.pl 


# 重启机器（注意，公司里的机器可别随便重启）
```

# Linux系统文件

> /etc初始化系统重要文件

- /etc/sysconfig/network-scripts/ifcfg-eth0:网卡配置文件
- /etc/resolv.conf:Linux系统DNS客户端配置文件
- /etc/hostname (CentOS7) /etc/sysconfig/network:(CentOS 6)主机名配置文件
- /etc/hosts:系统本地的DNS解析文件
- /etc/fstab:配置开机设备自动挂载的文件
- /etc/rc.local:存放开机自启动程序命令的文件
- /etc/inittab:系统启动设定运行级别等配置的文件
- /etc/profile及/etc/bashrc:配置系统的环境变量/别名等的文件
- /etc/profile.d:用户登录后执行的脚本所在的目录
- /etc/issue和/etc/issue.net:配置在用户登录终端前显示信息的文件
- /etc/init.d:软件启动程序所在的目录(centos 6)
- /usr/lib/systemd/system/ 软件启动程序所在的目录(centos 7)
- /etc/motd:配置用户登录系统之后显示提示内容的文件
- /etc/redhat-release:声明RedHat版本号和名称信息的文件
- /etc/sysctl.conf:Linux内核参数设置文件

> /proc重要路径

/proc/meminfo:系统内存信息

/proc/cpuinfo:关于处理器的信息，如类型，厂家，型号，性能等

<img src="\ajian/252.jpg" alt="img" style="zoom: 33%;" />

/proc/loadavg:系统负载信息，uptime 的结果

/proc/mounts:已加载的文件系统的列表

> /var目录下文件

/var/log:记录系统及软件运行信息文件所在的目录

/var/log/messages:系统级别日志文件

/var/log/secure:用户登录信息日志文件

/var/log/dmesg:记录硬件信息加载情况的日志文件

# 补充

# Linux系统基础概念

> 提示：可以放大linux图形化终端的字体

```
ctrl   shift   +   放大字体
ctrl  -    缩小字体
```

<img src="\ajian/image-20220107135032207.png" alt="image-20220107135032207" style="zoom:67%;" />

## 1.严格区分大小写

windows：不区分大小写，比如你创建文件夹，输入大写，小写，windows都认为是同一个。

<img src="\ajian/image-20220104112314933.png" alt="image-20220104112314933" style="zoom:50%;" />

linux严格区分大小写，文件夹的名字，大小写不同，就是不同的文件夹，文件也一样，因此我们后续对文件，文本操作，要注意大小写的区分处理。

<img src="\ajian/image-20220107135032207.png" alt="image-20220107135032207" style="zoom:67%;" />

可以看到，仅有一个字母的大小写不同，也是3个文件夹。

## 2.文件扩展名

### windows

Windows是依赖扩展名，区分不同的文件类型。比如

<img src="\ajian/image-20220104112923872.png" alt="image-20220104112923872" style="zoom:50%;" />

------

![image-20220104113039579](\ajian/image-20220104113039579.png)

```
.txt 是普通文本，一般是记事本这样的编辑器打开
.exe 是window下可以直接执行的文件，比如qq.exe 双击可以运行出qq登录器
.doc 是word文档
```

> windows下修改文件名，是会修改文件类型的，会修改文件属性。

<img src="\ajian/image-20220104113652453.png" alt="image-20220104113652453" style="zoom:50%;" />

------

<img src="\ajian/image-20220104113705995.png" alt="image-20220104113705995" style="zoom:50%;" />

### linux

而linux是大有不同，通过`权限位标识`来确定文件类型，我们在学习文件属性篇，重点再讲解，先记住如下概念。

- linux不通过文件扩展名来识别文件类型，文件扩展名，仅仅就是让运维人员能够肉眼一眼，就知道它是什么类型，便于管理
  - 但其实是该文件类型在创建时已经定义好，文件名只是用于显示，不像windows下有实际意义。
  - 即使你修改linux的文件名，也不会修改文件的类型。

<img src="\ajian/image-20220104113943464.png" alt="image-20220104113943464" style="zoom:50%;" />

------

<img src="\ajian/image-20220104114021021.png" alt="image-20220104114021021" style="zoom: 33%;" />

### 扩展名小结

为了区分出文件类型，我们还是会给linux文件，添加上阅读性更好的文件扩展名字。

常见的有

- 压缩文件
  - Linux 下常见的压缩文件名有 *.gz、*.bz2、*.zip、*.tar.gz、*.tar.bz2、*.tgz 等。
  - 为什么压缩包一定要写扩展名呢？很简单，如果不写清楚扩展名，那么管理员不容易判断压缩包的格式，虽然有命令可以帮助判断，但是直观一点更加方便。
  - 就算没写扩展名，在 Linux 中一样可以解压缩，不影响使用。
- 软件安装包
  - 如windows下的exe文件一样作用，linux也需要安装软件，也有软件包的格式。后面学习软件管理时重点讲解。
  - 如redhat系列的RPM包，所有的RPM包都是.rpm后缀格式。
- 脚本文件
  - 如shell脚本，.sh
  - 如python脚本，.py
  - 如java的 .java
- 网页相关的文件
  - .html
  - .jpg
  - .js
  - .css

> 这张图下展示了大多的linux后缀格式

<img src="\ajian/image-20220104114901241.png" alt="image-20220104114901241" style="zoom:50%;" />

## 3.Linux一切皆文件

<img src="\ajian/image-20220104135614494.png" alt="image-20220104135614494" style="zoom: 33%;" />

> 使用linux，记住一句话，linux一切皆文件，linux上所有的内容，都以文件的形式保存。
>
> 比如我们可以通过访问某个路径下的文件内容，读取如网卡的信息，读取如U盘的信息。
>
> 以查看进程为例，进程指的是系统上运行的一个程序，如windows下的进程，这是一段程序，通过任务管理器，才能找到它的信息。

<img src="\ajian/image-20220104135923206.png" alt="image-20220104135923206" style="zoom: 33%;" />

> linux读取进程信息，可以直接找到进程的文件

<img src="\ajian/image-20220104161003471.png" alt="image-20220104161003471" style="zoom: 25%;" />

### linux普通文件/文件夹

普通文件，类似windows里的txt文件概念，可以直接写入内容，查看文件内容，这就是普通文件。

> windows普通文件特点，一般可以这么操作的就称之为普通文件，包括了很多类型，如.txt .doc

- 编辑器直接打开
- 写入，修改，保存，查看内容

<img src="\ajian/image-20220104161540042.png" alt="image-20220104161540042" style="zoom:33%;" />

> linux普通文件，如何区分，我们知道没法通过后缀名直接判断
>
> 有办法

<img src="\ajian/image-20220104161937197.png" alt="image-20220104161937197" style="zoom:33%;" />

### linux可执行文件

> 可执行文件，指的是，该文件，可以双击运行，产生一些执行任务，比如QQ安装包，批量处理脚本文件。

windows，肉眼可以通过不同的后缀，一般可以得知，是什么文件类型，如下是常见的可执行类型。

<img src="\ajian/image-20220104162300660.png" alt="image-20220104162300660" style="zoom: 67%;" />

linux，是否可执行，就不是后缀决定的，而依然是通过文件属性查看的。（这个属性，在你创建该文件时就决定了，以不同的linux命令决定）

<img src="\ajian/image-20220104170830243.png" alt="image-20220104170830243" style="zoom: 33%;" />

### linux文件夹

可以通过肉眼观察（前提是安装了图形化linux界面），蓝色的是文件夹，以及文件属性开头是d，表示目录。

文件夹用于管理一堆文件，以及子文件夹。

<img src="\ajian/image-20220104171007469.png" alt="image-20220104171007469" style="zoom: 33%;" />

windows的文件夹就不多说了。

## 4.Linux的目录必须挂载后使用

挂载，mount，指的是给存储设备分配盘符，让我们能找到，使用存储设备（U盘，移动硬盘）

> windows下的挂载，超哥的电脑里有好几块硬盘，C盘是固态硬盘，D，E两个盘是机械硬盘。
>
> windows操作系统给硬盘加上了标记，我们通过C盘，盘符，即可找到C盘里的数据。

<img src="\ajian/image-20220104172559463.png" alt="image-20220104172559463" style="zoom:50%;" />

> linux里没有提供这样的字母，盘符。
>
> 而是
>
> 1.创建一个空文件夹，该文件夹有个名字，叫做挂载点（理解为windows下的盘符概念）
>
> 2.把设备和这个空文件夹做一个连接（这就叫做挂载），挂载是通过linux命令实现，后面讲。

### 图解linux挂载

<img src="\ajian/image-20220104183304532.png" alt="image-20220104183304532" style="zoom: 33%;" />

windows

- 买硬盘
- 分区
- 格式化

linux

- 买硬盘
- 分区
- 格式化
- 挂载

挂载命令

```
mount  /dev/sda1 /mnt/chaoge_file
```

## 5.一切皆文件细节

linux/unix下的哲学核心思想是‘一切皆文件’。

```
“一切皆文件”，指的是，对所有文件（目录、字符设备、块设备、套接字、打印机、进程、线程、管道等）操作，读写都可用fopen()/fclose()/fwrite()/fread()等函数进行处理。屏蔽了硬件的区别，所有设备都抽象成文件，提供统一的接口给用户。虽然类型各不相同，但是对其提供的却是同一套操作界面。更进一步，对文件的操作也可以跨文件系统执行。
```

那么如何操作一个已经打开的文件？

```
文件描述符（file descriptor），简称fd。这里就使用到了“文件描述符”，它是一个对应某个已经打开的文件的索引（非负整数）。
```

文件类型

| 类型               | 简称             | 描述                                                         |
| :----------------- | :--------------- | :----------------------------------------------------------- |
| 普通文件           | -,Normal File    | 如mp4、pdf、html log； 用户可以根据访问权限对普通文件进行查看、更改和删除，包括 纯文本文件(ASCII)；二进制文件(binary)；数据格式的文件(data);各种压缩文件.第一个属性为 [-] |
| 目录文件           | d,directory file | /usr/ /home/ 目录文件包含了各自目录下的文件名和指向这些文件的指针，打开目录事实上就是打开目录文件，只要有访问权限，就可以随意访问这些目录下的文件。能用#cd命令进入的。第一个属性为[d]，例如 [drwxrwxrwx] |
| 硬链接             | -,hard links     | 若一个inode号对应多个文件名，则称这些文件为硬链接。硬链接就是同一个文件使用了多个别名删除时,只会删除链接, 不会删除文件; 硬链接的局限性:1.不能引用自身文件系统以外的文件,即不能引用其他分区的文件;2.无法引用目录; |
| 符号链接（软链接） | l,symbolic link  | 若文件用户数据块中存放的内容是另一文件的路径名的指向，则该文件就是软连接，克服硬链接的局限性, 类似于快捷方式,使用与硬链接相同。 |
| 字符设备文件       | c,char           | 文件一般隐藏在/dev目录下，在进行设备读取和外设交互时会被使用到 即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]。 #/dev/tty的属性是 crw-rw-rw-，注意前面第一个字 c，这表示字符设备文件 |
| 块设备文件         | b,block          | 存储数据以供系统存取的接口设备，简单而言就是硬盘。 # /dev/hda1 的属性是 brw-r—– ，注意前面的第一个字符是b，这表示块设备，比如硬盘，光驱等设备 系统中的所有设备要么是块设备文件，要么是字符设备文件，无一例外 |
| FIFO管道文件       | p,pipe           | 管道文件主要用于进程间通讯。FIFO解决多个程序同时存取一个文件所造成的错误。比如使用mkfifo命令可以创建一个FIFO文件，启用一个进程A从FIFO文件里读数据，启动进程B往FIFO里写数据，先进先出，随写随读。 # pipe |
| 套接字             | s,socket         | 以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。用于进程间的网络通信，也可以用于本机之间的非网络通信，第一个属性为 [s]，这些文件一般隐藏在/var/run目录下，证明着相关进程的存在 |

## 6.图解linux与Windows目录

*Linux与windows区别*

- windows特点:E:\学习视频\高清视频\
- Linux目录特点:/etc/hosts /root/data/yuchao.txt

<img src="\ajian/248.jpg" alt="img" style="zoom:33%;" />

**Linux** 系统目录结构基本特点：

1.Linux下一切从`根`开始，根里面的第一层目录，叫做一级目录，然后依次二级目录。

2.Linux下面的目录是一个有层次的目录结构

3.在linux中每个目录可以挂载到不同的设备(磁盘)上

4.Linux 下设备不挂载不能使用，不挂载的设备相当于没门没窗户的监狱(进不去出不来)，挂载相当于给设备创造了一个入口(挂载点，一般为目录)

### 操作系统目录分隔符

*windows平台命令行目录分隔符*

<img src="\ajian/246.png" alt="img" style="zoom:33%;" />

*Linux平台命令行目录分隔符*

<img src="\ajian/247.jpg" alt="img" style="zoom:33%;" />

### Linux与Windows的目录结构比较

Linux首先是建立一个根"/"文件系统，所有的目录也都是由根目录衍生出来。

登录系统后，在当前命令窗口输入命令:

```
ls /
```

查看结果如下图：

<img src="\ajian/250.jpg" alt="img" style="zoom:50%;" />

在Linux底下，所有的文件与目录都是由根目录开始，是目录与文件的源头，然后一个个的分支下来，如同树枝状，因此称为这种目录配置为：**目录树**。

目录树的特点是什么呢？

- 目录树的起始点是根目录(/,root);
- 每一个目录不止能使用本地的文件系统，也可以使用网络上的文件系统，可以利用NFS服务器挂载特定目录。
- 每一个文件在此目录树中的文件名，包含完整路径都是独一无二的。

### Linux目录挂载

**挂载**通常是将一个`存储设备`挂接到一个已经存在的`目录`上，访问这个`目录`就是访问该存储设备的内容。

对于Linux系统来说，一切接文件，所有文件都放在以`根目录`为起点的树形目录结构中，任何硬件设备也都是文件形式

<img src="\ajian/321.jpg" alt="img" style="zoom:33%;" />

如图所示，是U盘存储设备和Linux系统自己的文件系统结构，此时Linux想要使用U盘的硬件设备，必须将Linux`本身的目录`和硬件设备的文件目录合二为一，此过程就称之为`挂载`。

```
挂载操作会隐藏原本Linux目录中的文件，因此选择Linux本身的目录，最好是新建空目录用于挂载
挂载之后，这个目录被称为挂载点
```

<img src="\ajian/322.jpg" alt="img" style="zoom:33%;" />

此时U盘文件系统已经是Linux文件系统的一部分，访问/sdb-u文件夹，即是访问访问U盘系统中的文件夹。

### linx的环境变量

同学们应该都会配置windows下的环境变量（PATH），都知道系统会按照PATH的设定，去每个PATH定义的目录下搜索可执行文件。

那么如何查看Linux下的PATH环境变量呢？

```
执行命令：
echo $PATH
echo命令是有打印的意思
$符号后面跟上PATH,表示输出PATH的变量
```

PATH(一定是大写的)这个变量是由一堆目录组成，分隔符是":"号，而不同于windows的";"号。

<img src="\ajian/278.jpg" alt="img" style="zoom: 50%;" />

### 绝对路径与相对路径

> Linux中非常重要的概念--路径，路径用来定位如何找到某个文件。

Linux下特别注意文件名/路径的写法，可以将所谓的路径(path)定义为绝对路径(absolute)和相对路径(relative)。这两种文件名/路径的写法依据是这样的：

- 绝对路径：由根目录(/)为开始写起的文件名或者目录名称，如/home/yuchao/test.py
- 相对路径：相对于目前路径的文件名写法。例如./home/yuchao/exam.py或../../home/yuchao/exam.py，简单来说只要开头不是/，就是属于相对路径，

> 因此你必须了解，相对路径是：**以你当前所在路径的相对路径来表示的。**

![img](\ajian/279.png)

例如你现在在/home 这个目录下，如要进入/var/log这个路径，如何写呢？

1. cd /var/log (绝对路径)
2. cd ../var/log(相对路径)

结果如图：

因为你在/home底下，因此你要回到上一层(../)之后，才能继续前往/var，特别注意：

- . :代表当前的目录，也可以用./ 来表示，注意结尾的斜线，目录分隔符。
- .. :代表上一层的目录，也可以用../来表示

<img src="\ajian/280.png" alt="img" style="zoom:50%;" />

### 图解绝对/相对路径

<img src="\ajian/image-20220104182559992.png" alt="image-20220104182559992" style="zoom:33%;" />

### Linux常见目录作用

<img src="\ajian/251.png" alt="img" style="zoom:50%;" />

- **/bin**：bin是Binary的缩写, 这个目录存放着最经常使用的命令。

- **/boot：**这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。

- **/dev ：**dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。

- **/etc：**这个目录用来存放所有的系统管理所需要的配置文件和子目录。

- **/home**：用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。

- **/lib**：这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。

- **/lost+found**：这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。

- **/media**：linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。

- **/mnt**：系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。

- **/opt**： 这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

- **/proc**：这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。 这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器：

  ```
  echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
  ```

- **/root**：该目录为系统管理员，也称作超级权限者的用户主目录。

- **/sbin**：s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。

- **/selinux**： 这个目录是Redhat/CentOS所特有的目录，Selinux是一个安全机制，类似于windows的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。

- **/srv**： 该目录存放一些服务启动之后需要提取的数据。

- **/sys**：这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。

  sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。

- **/tmp**：这个目录是用来存放一些临时文件的。

- **/usr**：这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。

- **/usr/bin：**系统用户使用的应用程序。

- **/usr/sbin：**超级用户使用的比较高级的管理程序和系统守护程序。

- **/usr/src：**内核源代码默认的放置目录。

- **/var**：这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。

在linux系统中，有几个目录是比较重要的，平时需要注意不要误删除或者随意更改内部文件。

**/etc： 上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。**

**/bin, /sbin, /usr/bin, /usr/sbin: 这是系统预设的执行文件的放置目录，比如 ls 就是在/bin/ls 目录下的。**

**值得提出的是，/bin, /usr/bin 是给系统用户使用的指令（除root外的通用户），而/sbin, /usr/sbin 则是给root使用的指令。**

**/var： 这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在/var/log 目录下，另外mail的预设放置也是在这里。**

### Linux软件安装到哪

<img src=\ajian\image-20220118183630133.png" style="zoom:33%;" />
