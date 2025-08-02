#  YUM与开源项目（Web运维）

# 学习目标

1、了解Linux软件的安装方式

2、掌握更新yum源

3、掌握YUM软件安装方式

4、了解LAMP环境以及AMP的关系

5、了解阿里云ECS的创建过程

6、能够yum方式搭建lamp环境

7、能够实现Discuz!论坛部署

8、能够购买域名与解析域名

# 前言

## 1、项目背景

运维小于，目前刚入职了一家电子商务公司。

主要负责大型商城系统维护，公司主营母婴用品，如奶瓶、奶嘴、童装等等，最近，很多客服发现一个问题：很多宝妈会在评论区互相咨询产品相关信息。

于是公司决定针对这一需求，要求运维为公司迅速上线一款论坛系统，方便宝妈交流产品、育儿心得等等。

## 2、项目需求

满足功能，并且省钱（开源项目），暂时没时间招程序员了，招人，写代码，上线，三个月就过去了！！

市面上会有很多开源组织，开源技术团队，产品开发完毕后，秉着开源精神，公开源代码让网民免费下载、使用，这样也会获得更大的曝光量，更多的技术人加入组织。

也给大量用户，带来方便，开箱即用。

Discuz！ = Apache + PHP + MySQL

![image-20220118112047256](http://book.bikongge.com/sre/2024-linux/image-20220118112047256.png)

# 一、YUM概述

## 1、Linux软件的安装方式

在CentOS系统中，软件管理方式通常有三种方式：`rpm安装`、`yum安装`以及`编译安装`。

> 编译安装，从过程上来讲比较麻烦，包需要用户自行下载，下载的是源码包，需要进行编译操作，编译好了才能进行安装，这个过程对于刚接触Linux的人来说比较麻烦，而且还容易出错。
>
> 好处在于是源码包，对于有需要自定义模块的用户来说非常方便。
>
> 专业linux运维肯定是要掌握编译软件的方法。

## 2、什么是yum

Yum（全称为 `Yellow dog Updater, Modified`）是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。

基于rpm包管理，能够从**指定的服务器**(yum源）自动下载RPM包并且安装，可以==自动处理依赖性关系==，并且==一次安装所有依赖的软件包==，无须繁琐地一次次下载、安装。

> 先回忆下，rpm包管理

在 RPM(红帽软件包管理器)公布之前，要想在 Linux 系统中安装软件只能采取源码包 的方式安装。

早期在 Linux 系统中安装程序是一件非常困难、耗费耐心的事情，而且大多数 的服务程序仅仅提供源代码，需要运维人员自行编译代码并解决许多的软件依赖关系，==因此 要安装好一个服务程序，运维人员需要具备丰富知识、高超的技能，甚至良好的耐心。==

==而且在 安装、升级、卸载服务程序时还要考虑到其他程序、库的依赖关系，所以在进行校验、安装、 卸载、查询、升级等管理软件操作时难度都非常大。==

### 软件包依赖关系

在早期系统运维中，安装软件是一件非常费事费力的事情。系统管理员不得不下载软件源代码编译软件，并且为了系统做各种调整。

==尽管源代码编译形式的软件增强了用户定制的自由度，但是在小软件上耗费精力是缺乏效率的，于是软件包应运而生。==

软件包管理可以将管理员从无休止的兼容问题中释放。yum工具就可以自动搜索依赖关系，并执行安装。

rpm软件包在安装的时候，由作者定义依赖关系

必须解决依赖关系，软件才能正常工作

### 如何检查软件依赖

> 通过rpm命令，可以检查某软件的依赖关系。
>
> 注意，这种方法只适用于**已安装**的包。如果你需要检查一个**未安装**包的依赖关系，你首先需要把这个包先下载到本地来（不需要安装）。
>
> 只能查询已安装的应用程序，依赖哪些其他软件。

![image-20220118113420619](http://book.bikongge.com/sre/2024-linux/image-20220118113420619.png)

> 其他检查rpm包依赖关系的方法，待会学完yum工具，再用

### yum配置文件

```
/etc/yum.conf
```

![image-20220118134001356](http://book.bikongge.com/sre/2024-linux/image-20220118134001356.png)

系统默认的yum仓库文件

```
[root@yuchao-linux01 opt]# ls /etc/yum.repos.d/ -l
total 32
-rw-r--r--. 1 root root 1664 Apr 29  2018 CentOS-Base.repo       网络yum源配置文件
-rw-r--r--. 1 root root 1309 Apr 29  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Apr 29  2018 CentOS-Debuginfo.repo   内核更新相关软件包
-rw-r--r--. 1 root root  314 Apr 29  2018 CentOS-fasttrack.repo   快速下载通道
-rw-r--r--. 1 root root  630 Apr 29  2018 CentOS-Media.repo        本地光盘yum配置文件
-rw-r--r--. 1 root root 1331 Apr 29  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 4768 Apr 29  2018 CentOS-Vault.repo
```

### 软件包管理神器

为了能让用户更方便、省心的管理软件，进行安装、卸载、升级，系统都会有一些方便的工具。

比如windows的360软件管家

![image-20220118135716473](http://book.bikongge.com/sre/2024-linux/image-20220118135716473.png)

而Linux的软件管家是什么？就是yum ![image-20220118140849677](http://book.bikongge.com/sre/2024-linux/image-20220118140849677.png)

## 3、配置阿里yum源

使用阿里yum源代替系统默认的yum源

备份默认的yum源

![image-20220118141234559](http://book.bikongge.com/sre/2024-linux/image-20220118141234559.png)

下载新的阿里云yum网络源，当我们yum install 就能够自动去阿里云的yum仓库寻找rpm包，而不是centos官网了。

教程https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11nCvpuy

```
1. 做好备份
[root@yuchao-linux01 yum.repos.d]# pwd
/etc/yum.repos.d
[root@yuchao-linux01 yum.repos.d]# ls
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo
[root@yuchao-linux01 yum.repos.d]# 
[root@yuchao-linux01 yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak


# 2.获取阿里云yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 3.查看新的yum源
[root@yuchao-linux01 yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
--2022-01-18 14:13:57--  https://mirrors.aliyun.com/repo/Centos-7.repo
Resolving mirrors.aliyun.com (mirrors.aliyun.com)... 124.165.127.206, 125.39.76.202, 125.39.76.204, ...
Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|124.165.127.206|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2523 (2.5K) [application/octet-stream]
Saving to: ‘/etc/yum.repos.d/CentOS-Base.repo’

100%[======================================================================>] 2,523       --.-K/s   in 0s      

2022-01-18 14:13:57 (490 MB/s) - ‘/etc/yum.repos.d/CentOS-Base.repo’ saved [2523/2523]

[root@yuchao-linux01 yum.repos.d]# ll
total 36
-rw--w--w-  1 root root 2523 Dec 26  2020 CentOS-Base.repo       这是新下载的
-rw-r--r--. 1 root root 1664 Apr 29  2018 CentOS-Base.repo.bak
-rw-r--r--. 1 root root 1309 Apr 29  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Apr 29  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Apr 29  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Apr 29  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Apr 29  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 4768 Apr 29  2018 CentOS-Vault.repo

# 4.清空yum缓存
yum clean all


# 5.生成新缓存，便于yum install 加速下载，生成cache
yum makecache
```

此时的网络yum源配置文件，已经是来自于阿里云的了。

![image-20220118141459634](http://book.bikongge.com/sre/2024-linux/image-20220118141459634.png)

生成缓存 ![image-20220118141820358](http://book.bikongge.com/sre/2024-linux/image-20220118141820358.png)

## 4、yum命令

### ① 查询操作

语法：# yum search 关键词

linux下的软件搜索，你想装东西，就用yum

```
[root@yuchao-linux01 yum.repos.d]# yum search firefox
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
========================================================= N/S matched: firefox =========================================================
firefox.x86_64 : Mozilla Firefox Web browser
firefox.i686 : Mozilla Firefox Web browser

  Name and summary matches only, use "search all" for everything.
[root@yuchao-linux01 yum.repos.d]#
```

### ② 安装操作

语法：# yum [-y] install 关键词

```
[root@yuchao-linux01 yum.repos.d]# yum install -y firefox
```

![image-20220118142544780](http://book.bikongge.com/sre/2024-linux/image-20220118142544780.png)

确保安装完毕

![image-20220118142647681](http://book.bikongge.com/sre/2024-linux/image-20220118142647681.png)

确保浏览器可用了

![image-20220118142719465](http://book.bikongge.com/sre/2024-linux/image-20220118142719465.png)

### ③ 卸载操作

语法：# yum [-y] remove 关键词

如何删除火狐浏览器？

```
[root@yuchao-linux01 yum.repos.d]# yum remove -y firefox
```

![image-20220118142544780](http://book.bikongge.com/sre/2024-linux/image-20220118142544780.png)

![image-20220118143037948](http://book.bikongge.com/sre/2024-linux/image-20220118143037948.png)

### ④ 更新操作

语法：#yum [-y] update [包的关键词]

==特别注意：包的关键词如果不写，则表示更新整个系统（全局更新，也包含内核）==

==千万别直接执行yum update -y，升级是一个重大的事==

升级，代表着所有内容都会更新，牵一发而动全身，你很多软件可能会全面崩溃。

```
# 升级vim
[root@yuchao-linux01 yum.repos.d]# yum update -y vim
```

![image-20220118143534621](http://book.bikongge.com/sre/2024-linux/image-20220118143534621.png)

> 若是升级一个不存在的软件，则提示找不到

```
[root@yuchao-linux01 yum.repos.d]# yum update -y firefox
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Package(s) firefox available, but not installed.
No packages marked for update
```

### ⑤yum获取软件依赖

yum命令本身就可以用来下载一个RPM包，标准的yum命令提供了--downloadonly(只下载)的选项来达到这个目的。

==该功能主要用于，离线安装，提前获取好rpm，这是一个非常省事的办法==

```
[root@yuchao-linux01 opt]# yum install --downloadonly --downloaddir=. python3
[root@yuchao-linux01 opt]# ll
total 9488
-rw--w--w- 1 root root   71844 Nov 18  2020 python3-3.6.8-18.el7.x86_64.rpm
-rw--w--w- 1 root root 7286976 Nov 18  2020 python3-libs-3.6.8-18.el7.x86_64.rpm
-rw--w--w- 1 root root 1702324 Oct 15  2020 python3-pip-9.0.3-8.el7.noarch.rpm
-rw--w--w- 1 root root  644052 Aug 23  2019 python3-setuptools-39.2.0-10.el7.noarch.rpm
[root@yuchao-linux01 opt]# 
[root@yuchao-linux01 opt]# 

此时你就可以拷贝走这些rpm包，再进行安装即可
```

### ⑥扩展rpmdep工具

> 这个只做了解，用于练习linux命令操作，以及yum操作。

还有一个办法是使用rpmdep工具，rpmdep是一个命令行工具，可以显示已安装包的完整包依赖关系图。

该工具会分析RPM包的依赖性，从完整的排完序的拓扑图中摘取部分包的信息，形成列表展示给用户。

该工具的输出结果可以直接使用到Dotty（可视化展示工具）中去。

```
1.获取工具
[root@yuchao-linux01 yum.repos.d]# wget http://downloads.sourceforge.net/project/rpmorphan/rpmorphan/1.14/rpmorphan-1.14-1.noarch.rpm

2.安装工具
[root@yuchao-linux01 yum.repos.d]# rpm -ivh rpmorphan-1.14-1.noarch.rpm 

3.安装绘图工具graphviz
[root@yuchao-linux01 yum.repos.d]#  yum install graphviz -y

4.生成软件依赖关系图片
[root@yuchao-linux01 opt]# rpmdep.pl -dot gzip.dot gzip
gzip depends upon basesystem,bash,ca-certificates,centos-release,chkconfig,coreutils,filesystem,gawk,glibc,glibc-common,gmp,grep,info,keyutils-libs,krb5-libs,libacl,libattr,libcap,libcom_err,libffi,libgcc,libselinux,libsepol,libstdc++,libtasn1,libverto,ncurses,ncurses-base,ncurses-libs,nspr,nss-softokn-freebl,nss-util,openssl-libs,p11-kit,p11-kit-trust,pcre,popt,sed,setup,tzdata,zlib
[root@yuchao-linux01 opt]# 
[root@yuchao-linux01 opt]# 
[root@yuchao-linux01 opt]# dot
dot      dot2gxl  dotty    
[root@yuchao-linux01 opt]# dot -Tpng -o output.png gzip.dot
[root@yuchao-linux01 opt]# ls
gzip.dot  output.png  tcpdump-4.9.2-4.el7_7.1.x86_64.rpm

5.在图形化下查看png图片
```

![image-20220118145652366](http://book.bikongge.com/sre/2024-linux/image-20220118145652366.png)

> 同理，也可以查看firefox浏览器的安装，底层牵扯了哪些依赖，如果没有yum都得你自己去处理

```
# firefox
[root@yuchao-linux01 opt]# rpmdep.pl -dot firefox.dot firefox
[root@yuchao-linux01 opt]# dot -Tpng -ofirefox.png firefox.dot

# python
[root@yuchao-linux01 opt]# rpmdep.pl -dot python.dot python
```

# 二、LAMP概述

## 1、什么是LAMP

LAMP是公认的最常见、最古老的黄金Web技术栈、

```
其实就是

Linux 操作系统
Apache/Nginx    web服务器 
Mysql/Mariadb
Perl/Php/Python
```

- LAMP：==L==inux + ==A==pache + ==M==ySQL + ==P==HP LAMP 架构（组合）
- LNMP：Linux + Nginx + MySQL + php-fpm LNMP 架构（组合）

lamp

![image-20200117165542095](http://book.bikongge.com/sre/2024-linux/image-20200117165542095.png)

lnmp

![image-20200212203206493](http://book.bikongge.com/sre/2024-linux/image-20200212203206493.png)

### Linux

Linux到底好在哪？用Linus本人的话说就是，普通老百姓用户，压根别说你是在`使用`操作系统，你需要的只是应用程序，而不是操作系统。

操作系统主要是提供给程序员API，用于构建和运行应用的一个平台。

如果来说，你常用的应用在Linux下运行的更好，更方便，那没问题。

但是如果你平时用的软件，都和Linux没什么关系，那你没必要选择Linux。

那当然作为运维人员，你可以一手使用windows、一手使用Linux，毕竟你的服务器运维工作，几乎都是Linux环境了。

Linux系统主要是以开发者为中心，Windows主要以消费者为中心这是本质的区别。

Linux的特点是几乎所有的开发任务相关工具，都有很完善的支持，从底层的编译器，make编译工具，到bash脚本，git代码管理，vim编辑器，依赖管理工具等等都很齐全。

然而Windows/Mac的操作系统很少能完善这些开发工具的，Linux则是默认预装的开发环境。

WIndows几乎都是图形化接口，而Linux几乎都是现有命令行，再由图形化操作接口，更容易实现自动化。

![image-20200118163840299](http://book.bikongge.com/sre/2024-linux/image-20200118163840299.png)

### apache

![image-20200118161748267](http://book.luffycity.com/linux-book/%E4%BA%92%E8%81%94%E7%BD%91%E6%9C%8D%E5%8A%A1%E5%9F%BA%E7%A1%80/pic/image-20200118161748267.png)

Apache Web Server虽然称之为`web服务器`，但是不是意味着他是一个`物理服务器`，它只是电脑软件中的一个软件而已，Web服务器的作用是将HTTP请求从前端转发到后端应用上。

### php

PHP是一门服务端脚本编程语言，主要用于web开发，常用PHP脚本嵌入HTML源码中执行。

PHP是全球知名的编程语言之一，程序员可以免费试用，PHP支持多种操作系统，开发效率高，支持多种数据库操作。

国内众多网站，百度、雅虎、新浪都在大量使用PHP语言进行开发，知名的论坛软件Discuz也是由PHP开发且占据了80%的论坛软件市场。

==世界上最好的语言（梗）==

![image-20220118152656853](http://book.bikongge.com/sre/2024-linux/image-20220118152656853.png)

### MySQL

Mysql是一款数据库管理系统，也就是一个存储数据的工具，用户可以自行对数据库进行增加、删除、修改、查询等操作。

MySQL是数据库管理系统中的一款软件，被业界广泛使用，例如新浪、QQ、淘宝、都在大量使用MySQL数据库。

腾讯QQ使用Linux与MySQL数据库，存储注册用户2.8亿的信息，活跃人数9000万，凭借万台服务器搭建的数据库集群，腾讯QQ同时在线人数也达到了千万，这证明了MySQL数据库的大容量、快速响应特点。

MySQL是一款关系型数据库，尤其适合Web应用，特别是电商领域，MySQL遍布各种行业、移动、爱立信、惠普、银行、思科、摩托萝拉、等等。

![image-20200118162000347](http://book.luffycity.com/linux-book/%E4%BA%92%E8%81%94%E7%BD%91%E6%9C%8D%E5%8A%A1%E5%9F%BA%E7%A1%80/pic/image-20200118162000347.png)

### LAMP图解

LAMP（Linux-Apache-MySQL-PHP）网站架构是目前国际流行的Web框架，该框架包括：Linux操作系统，Apache网络服务器，MySQL数据库，Perl、PHP或者Python编程语言

所有组成产品均是开源软件，是国际上成熟的架构框架，很多流行的商业应用都是采取这个架构

LAMP具有通用、跨平台、高性能、低价格的优势，因此LAMP无论是性能、质量还是价格都是企业搭建网站的首选平台。

![image-20200203165504534](http://book.bikongge.com/sre/2024-linux/image-20200203165504534.png)

我们是怎么访问网站的。

![image-20220118153236113](http://book.bikongge.com/sre/2024-linux/image-20220118153236113.png)

# 三、LAMP环境准备（阿里云）

> 这一节，我们就要进行搭建一个论坛，并且放入到互联网中，体验下，一个网站，从零到可以在浏览器访问到，是什么过程。
>
> 需要你掌握超哥前面讲的知识
>
> 1.linux基础命令，文件操作，ssh登录，解压缩等
>
> 2.linux软件安装，管理
>
> 3.理解一整套，运维部署的流程，做好清晰的部署笔记。
>
> 4.理解阿里云服务器的购买、使用流程。

要想部署一个互联网上可以访问到的环境，必须先具备以下内容 ：

服务器（IP、帐号密码、终端）、相应的软件、域名（备案、解析）、代码等。

## 1、注册阿里云账号

阿里云官网：https://www.aliyun.com/

立即注册

![image-20220118154229854](http://book.bikongge.com/sre/2024-linux/image-20220118154229854.png)

账户密码注册

![image-20220118155309260](http://book.bikongge.com/sre/2024-linux/image-20220118155309260.png)

填入验证码注册

![image-20220118155334409](http://book.bikongge.com/sre/2024-linux/image-20220118155334409.png)

注册成功

![image-20220118155436191](http://book.bikongge.com/sre/2024-linux/image-20220118155436191.png)

## 2、实名认证

购买服务器要进行实名认证，用于后面的域名购买，域名备案。

![image-20220118160112256](http://book.bikongge.com/sre/2024-linux/image-20220118160112256.png)

进行个人实名认证

![image-20220118160215380](http://book.bikongge.com/sre/2024-linux/image-20220118160215380.png)

快捷使用支付宝实名认证

![image-20220118160328676](http://book.bikongge.com/sre/2024-linux/image-20220118160328676.png)

后续填写信息后，完整实名认证。

![image-20220118160549935](http://book.bikongge.com/sre/2024-linux/image-20220118160549935.png)

## 3、进入管理控制台

![image-20220118160906292](http://book.bikongge.com/sre/2024-linux/image-20220118160906292.png)

找到阿里云服务器ECS功能

![image-20220118160953559](http://book.bikongge.com/sre/2024-linux/image-20220118160953559.png)

体验阿里云服务器

![image-20220118161012052](http://book.bikongge.com/sre/2024-linux/image-20220118161012052.png)

了解一下阿里云是什么ECS

![image-20220118161106280](http://book.bikongge.com/sre/2024-linux/image-20220118161106280.png)

进入ECS

阿里云会提供一些教程，帮助小白，来部署不同的应用。

比如你是想

- 搭建网站
- 搭建小程序
- 部署个人博客
- 公司网站上线

![image-20220118161341579](http://book.bikongge.com/sre/2024-linux/image-20220118161341579.png)

## 4、购买ECS

![image-20220118161614444](http://book.bikongge.com/sre/2024-linux/image-20220118161614444.png)

目前有活动、实名认证后，可以免费试用一个月。

### 机器配置选择

现在就等同于你在逛淘宝，选择机器的配置，内存，磁盘，CPU，以及既然是云服务器，要选择网络带宽（家里电脑要上网，要去装个电信网。）

![image-20220118162021231](http://book.bikongge.com/sre/2024-linux/image-20220118162021231.png)

准备创建

![image-20220118162040648](http://book.bikongge.com/sre/2024-linux/image-20220118162040648.png)

> 选择centos7.9版本
>
> 配置是1核、2G内存、40G云盘、带宽是1M

创建成功

![image-20220118165958397](http://book.bikongge.com/sre/2024-linux/image-20220118165958397.png)

### 查看机器信息

找到你的公网IP地址

![image-20220118170136010](http://book.bikongge.com/sre/2024-linux/image-20220118170136010.png)

### 设置服务器连接密码

账户root

密码设置的难一点，保护你的公网服务器。

![image-20220118170624058](http://book.bikongge.com/sre/2024-linux/image-20220118170624058.png)

> 重启中，修改密码后，需要重启服务器生效。

![image-20220118170829341](http://book.bikongge.com/sre/2024-linux/image-20220118170829341.png)

# 四、部署LAMP环境

## 1、登录阿里云服务器

![image-20220118171645124](http://book.bikongge.com/sre/2024-linux/image-20220118171645124.png)

创建连接

![image-20220118171808783](http://book.bikongge.com/sre/2024-linux/image-20220118171808783.png)

连接后，查看服务器基本信息 ![image-20220118172026009](http://book.bikongge.com/sre/2024-linux/image-20220118172026009.png)

修改主机名

```
[root@iZ2zegj6wtqlyu37r4ngr6Z ~]# hostnamectl set-hostname yuchao-aliyun
```

## 2、关闭内置防火墙

阿里云有提供公网防火墙（安全组）

![image-20220118172228144](http://book.bikongge.com/sre/2024-linux/image-20220118172228144.png)

> 在实验阶段，我们先临时关闭防火墙，我们会在后面网络安全篇，着重讲解防火墙规则
>
> 添加规则，是对服务器流量，以及各种应用程序进行把控，只有在你学习了各种linux程序部署、搭建、使用后。
>
> 才有了知识铺垫，然后进行安全流量控制。

![image-20220118172835890](http://book.bikongge.com/sre/2024-linux/image-20220118172835890.png)

## 3、查看是否安装apache

```
[root@yuchao-aliyun ~]# 
[root@yuchao-aliyun ~]# rpm -qa httpd
[root@yuchao-aliyun ~]#
```

没有结果，表示未安装httpd服务，也就是没装apache这个web服务器。

## 4、是否安装MySQL

```
[root@yuchao-aliyun ~]# rpm -qa mysql
```

## 5、是否安装php

```
[root@yuchao-aliyun ~]# rpm -qa php
```

> 为什么检查，因为如果机器安装过这些软件，或者安装后，卸载了，但是没有卸载干净，导致一些依赖软件的残留。
>
> 我们再进行安装的时候，就会碰到依赖冲突的错误。
>
> 建议新手用新机器操作。

## 6、LAMP环境之Apache安装

① 使用yum命令安装httpd软件包

apache这个软件，在linux中软件包的名字，是叫做httpd，因此得通过yum安装这个httpd

> 由于是阿里云服务器，默认用的也是阿里云yum源了。

```
[root@yuchao-aliyun ~]# yum install httpd -y
```

② 配置/etc/httpd/conf/httpd.conf文件

linux中安装、使用软件，流程就是

1.下载安装

2.修改配置文件

3.启动、使用

```
[root@yuchao-aliyun ~]# vim /etc/httpd/conf/httpd.conf

修改本行配置
一般填入网站的域名，如果没有可以写入IP地址
```

![image-20220118175955692](http://book.bikongge.com/sre/2024-linux/image-20220118175955692.png)

③ 使用systemctl命令重启httpd服务,使用netstat -ntlp命令，查看是否有80端口监听

```
[root@yuchao-aliyun ~]# systemctl restart httpd
[root@yuchao-aliyun ~]# 
[root@yuchao-aliyun ~]# 
[root@yuchao-aliyun ~]# netstat -tnlp|grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      1334/httpd
```

有80端口存在，并且该httpd服务，网络连接状态已经是LISTEN，监听中了。

好比银行的一个窗口，开始营业，对外服务了，你可以去窗口办理业务，获取数据了！

![image-20220118180333764](http://book.bikongge.com/sre/2024-linux/image-20220118180333764.png)

④ 设置httpd服务开机启动

![image-20220118180415913](http://book.bikongge.com/sre/2024-linux/image-20220118180415913.png)

⑤ 查看本机的IP地址,阿里云服务器从控制台可以看到

![image-20220118180516885](http://book.bikongge.com/sre/2024-linux/image-20220118180516885.png)

阿里云可以看到公网IP地址

![image-20220118180538382](http://book.bikongge.com/sre/2024-linux/image-20220118180538382.png)

⑥在浏览器中，输入本机IP地址，如下图所示：

```
123.57.24.213
```

## 7、打开阿里云安全组（图解）

![image-20220118181219778](http://book.bikongge.com/sre/2024-linux/image-20220118181219778.png)

阿里云安全组

![image-20220118181310620](http://book.bikongge.com/sre/2024-linux/image-20220118181310620.png)

添加规则

![image-20220118181419766](http://book.bikongge.com/sre/2024-linux/image-20220118181419766.png)

## 8、成功访问apache

```
http://123.57.24.213/
```

并且只要你有网络，其他人就可以访问这个网站。

![image-20220118181447597](http://book.bikongge.com/sre/2024-linux/image-20220118181447597.png)

## 9、LAMP之MYSQL

阿里云yum源默认是没有mysql的软件的，因此你直接用装不了。

比如

```
yum -y install mysql-community-server
```

![image-20220118181802313](http://book.bikongge.com/sre/2024-linux/image-20220118181802313.png)

配置mysql的软件rpm源

![image-20220118183535112](http://book.bikongge.com/sre/2024-linux/image-20220118183535112.png)

这个教程去mysql官网即可

==超哥的笔记认第二、谁认第一？==

```
https://dev.mysql.com/

http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

# 1.下载mysql的yum源
[root@yuchao-aliyun local]# cd /usr/local/
[root@yuchao-aliyun local]# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

# 2.安装，查看mysql的yum源

[root@yuchao-aliyun local]# rpm -ivh mysql-community-release-el7-5.noarch.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-community-release-el7-5    ################################# [100%]

[root@yuchao-aliyun local]# 
[root@yuchao-aliyun local]# ls -l  /etc/yum.repos.d/
total 16
-rw-r--r-- 1 root root  675 Jan 18 17:00 CentOS-Base.repo
-rw-r--r-- 1 root root  230 Jan 18 17:00 epel.repo
-rw-r--r-- 1 root root 1209 Jan 29  2014 mysql-community.repo
-rw-r--r-- 1 root root 1060 Jan 29  2014 mysql-community-source.repo

# 3.此时可以安装mysql
yum -y install mysql-community-server

# 4.安装完毕后，启动mysql
完成后，系统自动生成mysql服务管理脚本，systemctl可以去调用
也是我们通过systemctl 去管理的服务的名字

[root@yuchao-aliyun local]# systemctl start mysqld 

# 5.查看mysql运行端口，进程
[root@yuchao-aliyun local]# netstat -tnlp|grep mysql
tcp6       0      0 :::3306                 :::*                    LISTEN      1754/mysqld  

[root@yuchao-aliyun local]# ps -ef|grep mysql
mysql     1587     1  0 18:43 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql     1754  1587  0 18:43 ?        00:00:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock
root     12022  1202  0 18:54 pts/0    00:00:00 grep --color=auto mysql

# 6.确保mysql启动后，初始化数据，进行使用
默认的mysql没有密码，没数据，得初始化使用
[root@yuchao-aliyun local]# mysql_secure_installation 

# 7.连接mysql服务端
[root@yuchao-aliyun local]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 5.6.51 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

mysql> 
mysql> exit
Bye
```

这个repo文件，就是指定了一个rpm包的下载地址

![image-20220118184049205](http://book.bikongge.com/sre/2024-linux/image-20220118184049205.png)

安装完成

![image-20220118184329677](http://book.bikongge.com/sre/2024-linux/image-20220118184329677.png)

查看mysql服务的名字，已经启动mysql

![image-20220118185011913](http://book.bikongge.com/sre/2024-linux/image-20220118185011913.png)

查看进程

![image-20220118185532624](http://book.bikongge.com/sre/2024-linux/image-20220118185532624.png)

初始化数据库

![image-20220118200937804](http://book.bikongge.com/sre/2024-linux/image-20220118200937804.png)

连接mysql客户端

![image-20220118201119721](http://book.bikongge.com/sre/2024-linux/image-20220118201119721.png)

c/s模式

![image-20220118201350679](http://book.bikongge.com/sre/2024-linux/image-20220118201350679.png)

查看数据库

![image-20220118201700369](http://book.bikongge.com/sre/2024-linux/image-20220118201700369.png)

## 10、LAMP安装PHP

1、使用yum安装php即可

```
[root@yuchao-aliyun local]# yum install php -y
```

![image-20220118201847866](http://book.bikongge.com/sre/2024-linux/image-20220118201847866.png)

2、重启httpd服务

![image-20220118202058584](http://book.bikongge.com/sre/2024-linux/image-20220118202058584.png)

apache是需要和php结合起来工作的，我们这里主要练习yum工具，安装，部署网站，其中原理，超哥会在网站架构篇，详细，通透的讲解其中原理。

```
[root@yuchao-aliyun local]# systemctl restart httpd

1.在安装php之后，重启httpd
2.php能够自动和apache结合工作了。
```

### 测试LAMP

```
1.进入httpd，apache的网站根目录，也就是这个网页存放的地方。
[root@yuchao-aliyun local]# cd /var/www/html/
[root@yuchao-aliyun html]# 
[root@yuchao-aliyun html]# 
[root@yuchao-aliyun html]# vim index.php 
[root@yuchao-aliyun html]# 
[root@yuchao-aliyun html]# pwd
/var/www/html
[root@yuchao-aliyun html]# 
[root@yuchao-aliyun html]# cat index.php 
<?php
    echo '超哥带你学LAMP、学习Linux云计算';
?>
[root@yuchao-aliyun html]# 


2.这里的意思是，我们访问apache，然后看到php脚本，脚本内的代码是打印一句话。
```

![image-20220118202531456](http://book.bikongge.com/sre/2024-linux/image-20220118202531456.png)

![image-20220118202923506](http://book.bikongge.com/sre/2024-linux/image-20220118202923506.png)

> 此时我们已经能够正确访问到
>
> 一个支持linux+apache+mysql+php体系的系统平台
>
> 也已经看到了网站显示的内容
>
> 此时你已经准备好了一个LAMP环境，部署论坛网站，换一套代码就好了。

# 五、部署Discuz论坛

![image-20220118203421260](http://book.bikongge.com/sre/2024-linux/image-20220118203421260.png)

## 点击下载

![image-20220118203654404](http://book.bikongge.com/sre/2024-linux/image-20220118203654404.png)

## 码云下载Discuz下载

![image-20220118203723417](http://book.bikongge.com/sre/2024-linux/image-20220118203723417.png)

## 上传到linux

1.linux里安装lrzsz软件，用于上传下载、或者用FTP。

```
[root@yuchao-aliyun html]# yum install lrzsz -y
```

2.用命令上传文件

```
# 输入rz命令，xshell自动弹出文件接收功能
# 后面传输大量文件，还是使用FTP工具，一般如XFTP
[root@yuchao-aliyun html]# rz

# 上传到apache的网页根目录，这个目录下，只要存放了HTML文件，php文件，就能访问到
[root@yuchao-aliyun html]# pwd
/var/www/html
[root@yuchao-aliyun html]# ls
DiscuzX-master.zip  index.php

# 安装unzip
[root@yuchao-aliyun html]# yum install -y unzip

# 解压缩Discuz代码
[root@yuchao-aliyun html]# unzip DiscuzX-master.zip 

# 这个论坛的源代码，就在这里了。
[root@yuchao-aliyun DiscuzX-master]# pwd
/var/www/html/DiscuzX-master
[root@yuchao-aliyun DiscuzX-master]# ls
LICENSE  readme  README.md  upload  utility


# 最后异步，需要把/var/www/html/DiscuzX-master/upload下代码，全部移动到 /var/www/html 这个位置，且必须在这个位置
```

![image-20220119092337444](http://book.bikongge.com/sre/2024-linux/image-20220119092337444.png)

注意看，最终，Discuz论坛的代码，要放在哪里

![image-20220119093239048](http://book.bikongge.com/sre/2024-linux/image-20220119093239048.png)

# 六、访问Discuz论坛

![image-20220119093745895](http://book.bikongge.com/sre/2024-linux/image-20220119093745895.png)

此时再访问这个apache，也就是阿里云服务器的地址，就可以自动访问到discuz论坛安装界面了。

> 安装环境监察

![image-20220119094158104](http://book.bikongge.com/sre/2024-linux/image-20220119094158104.png)

------

![image-20220119094219755](http://book.bikongge.com/sre/2024-linux/image-20220119094219755.png)

发现少了一个关于mysql的连接驱动

![image-20220119094716283](http://book.bikongge.com/sre/2024-linux/image-20220119094716283.png)

## 安装mysql连接驱动

linux运维的日常就是，根据手册，部署，遇见问题，1、看懂 2、琢磨怎么解决 3、yum可以安装大部分软件，解决大部分问题。

> 你可以借助搜索引擎，搜索报错信息，找到网络上大部分经验相同的人，如何解决
>
> 你可以问超哥 哈哈，有时候，向老师傅请教，能更快的先解决问题，然后自己再吸收这个解决的经验。

上述问题，可以直接yum安装

```
[root@yuchao-aliyun html]# yum install php-mysqli -y
```

![image-20220119095042052](http://book.bikongge.com/sre/2024-linux/image-20220119095042052.png)

一般有安装更新，软件都会重启，让其生效，我这里访问的是apache，因此重启httpd服务，让这个新驱动生效。

```
[root@yuchao-aliyun html]# systemctl restart httpd
```

再次访问Discuz安装界面，刷新即可。

![image-20220119095326093](http://book.bikongge.com/sre/2024-linux/image-20220119095326093.png)

## 解决目录权限问题

![image-20220119095508222](http://book.bikongge.com/sre/2024-linux/image-20220119095508222.png)

1.确认我们的httpd目录，Discuz代码存放的目录

```
# 页面提示的错误信息，就是这里的目录权限不够
[root@yuchao-aliyun html]# ll
total 12452
-rw-r--r--  1 root root     2848 Jan 17 10:56 admin.php
drwxr-xr-x  9 root root     4096 Jan 17 10:56 api
-rw-r--r--  1 root root      727 Jan 17 10:56 api.php
drwxr-xr-x  2 root root     4096 Jan 17 10:56 archiver
drwxr-xr-x  2 root root     4096 Jan 17 10:56 config
-rw-r--r--  1 root root     1040 Jan 17 10:56 connect.php
-rw-r--r--  1 root root      106 Jan 17 10:56 crossdomain.xml
drwxr-xr-x 12 root root     4096 Jan 17 10:56 data
drwxr-xr-x  6 root root     4096 Jan 17 10:56 DiscuzX-master
-rw-r--r--  1 root root 12630971 Jan 18 20:37 DiscuzX-master.zip
-rw-r--r--  1 root root     5558 Jan 17 10:56 favicon.ico
-rw-r--r--  1 root root     2245 Jan 17 10:56 forum.php
-rw-r--r--  1 root root      821 Jan 17 10:56 group.php
-rw-r--r--  1 root root     1280 Jan 17 10:56 home.php
-rw-r--r--  1 root root     7044 Jan 17 10:56 index.php
drwxr-xr-x  5 root root     4096 Jan 17 10:56 install
drwxr-xr-x  2 root root     4096 Jan 17 10:56 m
-rw-r--r--  1 root root      998 Jan 17 10:56 member.php
-rw-r--r--  1 root root     2371 Jan 17 10:56 misc.php
-rw-r--r--  1 root root     1788 Jan 17 10:56 plugin.php
-rw-r--r--  1 root root      977 Jan 17 10:56 portal.php
-rw-r--r--  1 root root      615 Jan 17 10:56 robots.txt
-rw-r--r--  1 root root     1274 Jan 17 10:56 search.php
drwxr-xr-x 10 root root     4096 Jan 17 10:56 source
drwxr-xr-x  7 root root     4096 Jan 17 10:56 static
drwxr-xr-x  3 root root     4096 Jan 17 10:56 template
drwxr-xr-x  7 root root     4096 Jan 17 10:56 uc_client
drwxr-xr-x 14 root root     4096 Jan 17 10:56 uc_server
```

2.添加权限

```
[root@yuchao-aliyun html]# 
[root@yuchao-aliyun html]# chmod -R 777 /var/www/html/*
```

3.再次查看安装

![image-20220119100013992](http://book.bikongge.com/sre/2024-linux/image-20220119100013992.png)

## 开始安装

![image-20220119100103528](http://book.bikongge.com/sre/2024-linux/image-20220119100103528.png)

------

![image-20220119100322669](http://book.bikongge.com/sre/2024-linux/image-20220119100322669.png)

如果信息填入正常，php正确读取到mysql数据库，即刻自动安装，数据也会写入到数据库里。

![image-20220119100417514](http://book.bikongge.com/sre/2024-linux/image-20220119100417514.png)

登录admin管理员账户

![image-20220119100503908](http://book.bikongge.com/sre/2024-linux/image-20220119100503908.png)

## 使用discuz发帖子

![image-20220119101530287](http://book.bikongge.com/sre/2024-linux/image-20220119101530287.png)

------

![image-20220119101636264](http://book.bikongge.com/sre/2024-linux/image-20220119101636264.png)

> 至此，基于LAMP环境部署的DIscuz论坛，已经部署完毕了，你学废了吗？

最终，我们还差一个网站的域名解析，就更完美了，但是域名涉及备案流程，较为复杂，我们在学习阿里云计算篇，继续学习。