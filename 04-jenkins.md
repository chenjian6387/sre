# 04-jenkins

# 1.代码上线发展史

代码发布上线是每一个 IT 企业必须要面临的，而且不管是对开发或者是运维来说，代 码上线本身就是一个件非常痛苦的事情，很多时候每一次发布都是一次考验。

为了提高上线 的效率，代码上线的方式，方法，工具也不断的发展，基本上可以分为以下几个阶段。

## 没有jenkins

软件在开发者的机器上通过脚本或者手动构建，源代码保存在代码服务器中，但是开发者经常要修改本地代码，因此每次发布，都需要手动合并代码，再进行手动构建，这个过程费时费力。

## 晚上进行构建

在这个阶段，团队有构建服务器，自动化的构建在晚上进行。

构建过程只是简单的编译代码，没有可靠的和可重复的单元测试。

然而，开发人员每天提交代码。如果某个开发人员 提的代码和其他人的代码冲突的话，构建服务器会在第二天通过邮件通知团队。

所以又一段 时间构建是处于失败状态的。

## 晚上进行构建并进行自动化测试

团队对 CI 和自动化测试越来越重视。

无论什么时候版本管理系统中的代码改变了都会 触发编译构建过程，团队成员可以看到是代码中的什么改变触发了这个构建。

并且，构建脚本会编译应用并且会执行一系列的单元测试或集成测试。

除了邮件，构建服务器还可以通过 其他方式通知团队成员，如微信，钉钉通知。

## 代码质量度量

自动化的代码质量和测试覆盖率的度量手段有助于评价代码质量和测试的有效性。

## 更加认真的对待测试

CI和测试紧密相关，如今测试驱动开发被广泛的使用，是的对自动化的构建更加有信心。

应用不仅仅是简单的编译和测试，而是如果测试成功会被自动的部署到一个应用服务器，以此进行更多的测试工作与性能测试。

## 验收测试与自动化部署

验收测试驱动的开发被使用，使得我们能够了解项目的状态。

这些自动化的测试使用行 为驱动的开发和测试驱动的开发工具来作为交流和文档工具，发布非开发人员也能读懂的测 试结果报告。由于测试在开发的早起就已经被自动化的执行了，所以我们能更加清楚地了解 到什么已经做了，什么还没有做。

每当代码改变或晚上，应用被自动化地部署到测试环境中， 以供 QA 团队测试。

当测试通过之后，软件的一个版本将被手工部署到生产环境中，团队也可以在出现问题的时候回滚之前的发布记录。

# 2.jenkins来了

 Jenkins是一个可扩展的持续集成引擎，是一个开源软件项目，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

**构建伟大，无所不能**

Jenkins是开源CI&CD软件领导者， 提供超过1000个插件来支持构建、部署、自动化， 满足任何项目的需要。

[文档](https://www.jenkins.io/zh/doc)[下载](https://www.jenkins.io/zh/download)

官网：https://jenkins.io/

![image-20220710175227830](http://book.bikongge.com/sre/2024-linux/image-20220710175227830.png)

```
##### 持续集成和持续交付
作为一个可扩展的自动化服务器，Jenkins 可以用作简单的 CI 服务器，或者变成任何项目的持续交付中心。

##### 简易安装
Jenkins 是一个基于 Java 的独立程序，可以立即运行，包含 Windows、Mac OS X 和其他类 Unix 操作系统。

##### 配置简单
Jenkins 可以通过其网页界面轻松设置和配置，其中包括即时错误检查和内置帮助。

##### 插件
通过更新中心中的 1000 多个插件，Jenkins 集成了持续集成和持续交付工具链中几乎所有的工具。

##### 扩展
Jenkins 可以通过其插件架构进行扩展，从而为 Jenkins 可以做的事提供几乎无限的可能性。

##### 分布式
Jenkins 可以轻松地在多台机器上分配工作，帮助更快速地跨多个平台推动构建、测试和部署。
```

## 2.1 不用jenkins怎么部署

```
* 研发人员上传开发好的代码到github代码仓库

* 需要将代码下载到linux服务器部署
  * 运维人员手动下载再部署
  * 运维人员使用脚本下载再部署
```

![image-20220710175907307](http://book.bikongge.com/sre/2024-linux/image-20220710175907307.png)

```
也正是上一节于超老师让大家部署在gitlab中的halo代码

现在只需要通过编写shell脚本，实现
1. 拉取代码
2. 部署环境
3. 运行程序
完成此基本逻辑即可
```

## 2.2 使用jenkins轻松部署

![image-20220710181416658](http://book.bikongge.com/sre/2024-linux/image-20220710181416658.png)

使用jenkins会是如何

![image-20220710181733759](http://book.bikongge.com/sre/2024-linux/image-20220710181733759.png)

## 2.3 安装jenkins

准备好jenkins服务器，建议配置至少4G、2C

```
jenkins默认是8080端口
注意别和其他服务冲突，重新创建一个机器
jenkins-100服务器
1.去清华源下载
https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/

2.下载好如下系列软件包即可

[root@jenkins-100 /opt]#ls
jdk-8u181-linux-x64.rpm  jenkins-2.190.1-1.1.noarch.rpm  jenkins_plugins.tar.gz



3.软件需求：由于jenkins是使用java语言开发，因此运行需要安装java运行时环境（jdk）。
可以通过清华镜像站下载jenkins安装包，以及jdk环境可以通过yum直接安装，也可以下载好rpm包安装

[root@jenkins-100 /opt]#rpm -ivh jdk-8u181-linux-x64.rpm 
warning: jdk-8u181-linux-x64.rpm: Header V3 RSA/SHA256 Signature, key ID ec551f03: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:jdk1.8-2000:1.8.0_181-fcs        ################################# [100%]
Unpacking JAR files...
    tools.jar...
    plugin.jar...
    javaws.jar...
    deploy.jar...
    rt.jar...
    jsse.jar...
    charsets.jar...
    localedata.jar...
[root@jenkins-100 /opt]#



4.安装jenkins
[root@jenkins-100 /opt]#rpm -ivh jenkins-2.190.1-1.1.noarch.rpm 
warning: jenkins-2.190.1-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:jenkins-2.190.1-1.1              ################################# [100%]




检查jenkins版本
[root@jenkins-100 /opt]#rpm -qa jenkins
jenkins-2.190.1-1.1.noarch
```

## 2.4 启动，检查jenkins

```
[root@jenkins-100 /opt]#netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      999/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1226/master         
tcp6       0      0 :::22                   :::*                    LISTEN      999/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1226/master         
[root@jenkins-100 /opt]#
[root@jenkins-100 /opt]#
[root@jenkins-100 /opt]#systemctl start jenkins
[root@jenkins-100 /opt]#
[root@jenkins-100 /opt]#netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      999/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1226/master         
tcp6       0      0 :::8080                 :::*                    LISTEN      11590/java          
tcp6       0      0 :::22                   :::*                    LISTEN      999/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1226/master
```

## 2.5 jenkins配置文件

学习jenkins首先要明白一点，那就是jenkins一切皆文件，jenkins没有数据库，所有的数据都是以文件的形式存在，所以我们要了解jenkins的主要目录和文件。

```
[root@m01 ci_cd_rpm]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins


jenkins加载插件很多，非常吃内存，且所有操作都是io操作，需要系统IO速度较快，需要机器配置较高

jenkins配置文件：/etc/sysconfig/jenkins
jenkins核心目录：/var/lib/jenkins
/usr/lib/jenkins/jenkins.war WAR包
/etc/sysconfig/jenkins 配置文件
/var/lib/jenkins/ 默认的JENKINS_HOME目录
/var/log/jenkins/jenkins.log Jenkins日志文件
/var/lib/jenkins/secrets/initialAdminPassword 存放初始密码
/var/lib/jenkins/plugins    插件目录
```

查看jenkins默认的密码

```
[root@jenkins-100 /opt]#cat /var/lib/jenkins/secrets/initialAdminPassword 
0b71ec8275a34bbd99ba450135397471
```

## 2.6 登录jenkins

![image-20220710204619385](http://book.bikongge.com/sre/2024-linux/image-20220710204619385.png)

## 2.7 自定义jenkins插件

别用自带的jenkins社区插件，下载的太累，一般会用以装好的机器，导出的插件，离线导入更高效。

![image-20220710204805136](http://book.bikongge.com/sre/2024-linux/image-20220710204805136.png)

![image-20220710205214321](http://book.bikongge.com/sre/2024-linux/image-20220710205214321.png)

## 2.8 jenkins首页

![image-20220710205306461](http://book.bikongge.com/sre/2024-linux/image-20220710205306461.png)

## 2.9 修改admin密码

![image-20220710205345547](http://book.bikongge.com/sre/2024-linux/image-20220710205345547.png)

------

修改密码

![image-20220710205420529](http://book.bikongge.com/sre/2024-linux/image-20220710205420529.png)

```
随后会让你重新登录
账号admin，密码www.yuchaoit.cn
```

## 3.0 忘记密码/密码配置文件

```
如果忘记密码，可以改配置文件，强制改密

[root@jenkins-100 /opt]#cp /var/lib/jenkins/config.xml{,.bak}


删除config.xml 的如下配置

  8   <useSecurity>true</useSecurity>
  9   <authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy">
 10     <denyAnonymousReadAccess>true</denyAnonymousReadAccess>
 11   </authorizationStrategy>
 12   <securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
 13     <disableSignup>true</disableSignup>
 14     <enableCaptcha>false</enableCaptcha>
 15   </securityRealm>


然后重启服务即可
systemctl restart jenkins

2，重启Jenkins服务

3，进入首页>“系统管理”>“Configure Global Security”；(全局安全配置)

4，勾选“启用安全”；

5，点选“Jenkins专有用户数据库”，并点击“保存”；

6，重新点击首页>“系统管理”,发现此时出现“管理用户”；

7，点击进入展示“用户列表”；

8，点击右侧进入修改密码页面，修改后即可重新登录。
```

# 3.jenkins插件安装

```
因为下载插件的官方在国外,网络可能会不稳定。

如果在安装插件那一步出现offline或者找不到XXX插件的报错,可以换个网络试试。

或者休息一下,换个时间再试。

如果实在是无法下载插件,可以将别人下载好的插件打包给你，解压到/var/lib/jenkins/plugins/目录

需要重启jenkins服务,才能在web界面读取到解压的插件
```

![image-20220710210527733](http://book.bikongge.com/sre/2024-linux/image-20220710210527733.png)

------

![image-20220710220508791](http://book.bikongge.com/sre/2024-linux/image-20220710220508791.png)

------

![image-20220710220528218](http://book.bikongge.com/sre/2024-linux/image-20220710220528218.png)

## 3.1 在清华源站下载插件

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/

下载.hpi文件，然后在jenkins里面导入上传即可，自动安装，重启生效
```

![image-20220710220752136](http://book.bikongge.com/sre/2024-linux/image-20220710220752136.png)

```
可以更换插件源为清华的
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
替换
https://updates.jenkins.io/update-center.json
```

## 3.2 导入本地插件(汉化插件)

```
可以把插件备份的机器，所有的插件目录/var/lib/jenkins/plugins 资料直接打包

解压缩到新插件目录即可，jenkins一切皆文件，可以直接迁移

jenkins没有数据库，一切皆文件，都是写在xml文件里

插件以及准备好了，如果没有可以向超哥要

[root@jenkins-100 /opt]#tar -zxf jenkins_plugins.tar.gz 
[root@jenkins-100 /opt]#ls
jdk-8u181-linux-x64.rpm  jenkins-2.190.1-1.1.noarch.rpm  jenkins_plugins.tar.gz  plugins

[root@jenkins-100 /opt]#cd plugins/

[root@jenkins-100 /opt/plugins]#ls |wc -l
263

移动插件到jenkins的插件目录即可
[root@jenkins-100 /opt/plugins]#mv * /var/lib/jenkins/plugins/

重启jenkins，查看插件列表
systemctl restart jenkins
```

![image-20220710222410044](http://book.bikongge.com/sre/2024-linux/image-20220710222410044.png)

------

以改为中文了，nice

![image-20220710222706436](http://book.bikongge.com/sre/2024-linux/image-20220710222706436.png)

# 4.jenkins配置相关

## 4.1安装插件后，首页多了些内容

![image-20220710222824353](http://book.bikongge.com/sre/2024-linux/image-20220710222824353.png)

------

![image-20220710222853549](http://book.bikongge.com/sre/2024-linux/image-20220710222853549.png)

## 4.2 jenkins配置文件目录

```
[root@m01 plugins]# rpm -ql jenkins
/etc/init.d/jenkins                    # 启动目录
/etc/logrotate.d/jenkins        # 日志切割
/etc/sysconfig/jenkins          # jenkins主配置文件
/usr/lib/jenkins                        #jenkins程序文件 jenkins.war，如 直接替换新版jenkins.war包，重启即升级
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins                      # jenkins主目录，如果要备份jenkins所有数据，直接拷贝这个目录即可
/var/log/jenkins
```

## 4.3 jenkins用户信息

```
/var/lib/jenkins/users  # 该目录下存放用户信息
如 admin的信息/var/lib/jenkins/users/admin/config.xml ，改密码等等



[root@jenkins-100 /opt/plugins]#cat /var/lib/jenkins/users/users.xml 
<?xml version='1.1' encoding='UTF-8'?>
<hudson.model.UserIdMapper>
  <version>1</version>
  <idToDirectoryNameMap class="concurrent-hash-map">
    <entry>
      <string>admin</string>
      <string>admin_1712638716449695657</string>
    </entry>
  </idToDirectoryNameMap>
</hudson.model.UserIdMapper>[root@jenkins-100 /opt/plugins]#
```

## 4.4 jenkins主配置文件

```
主配置文件解读
jenkins默认的用户为jenkins，生产环境建议使用jenkins用户，然后使用sudo进行授权，在学习过程中若是想要避免权限问题，可以直接改为root用户。

友情提醒，jenkins的配置文件，特别是用户，别乱改，否则jenkins里的job也无法用了



默认配置参数，只改一个root即可，因为需要用jenkins去执行很多命令，脚本等。
[root@m01 plugins]# grep -Ev "^$|^#" /etc/sysconfig/jenkins
JENKINS_HOME="/var/lib/jenkins"                #jenkins主数据目录，数据备份，也只需要打包该文件夹即可
JENKINS_JAVA_CMD=""
JENKINS_USER="root"                                # 启动用户
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
JENKINS_PORT="8080"                                         # 启动端口，可以改
JENKINS_LISTEN_ADDRESS=""                          # 监听地址
JENKINS_HTTPS_PORT=""
JENKINS_HTTPS_KEYSTORE=""
JENKINS_HTTPS_KEYSTORE_PASSWORD=""
JENKINS_HTTPS_LISTEN_ADDRESS=""
JENKINS_DEBUG_LEVEL="5"
JENKINS_ENABLE_ACCESS_LOG="no"
JENKINS_HANDLER_MAX="100"
JENKINS_HANDLER_IDLE="20"
JENKINS_ARGS=""

重启
systemctl restart jenkins

检查主进程，是否是root运行了
[root@jenkins-100 /opt/plugins]#ps -ef|grep jenkins
root      17763      1 99 06:34 ?        00:00:12 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
root      17847   1488  0 06:34 pts/0    00:00:00 grep --color=auto jenkins
```

## 4.5 修改时区

![image-20220710223612767](http://book.bikongge.com/sre/2024-linux/image-20220710223612767.png)

# 5.jenkins初体验

## 5.1创建jenkins任务job

构建作业是一个持续集成服务器的基本职能，构建的形式多种多样，可以是编译和单元测试，也可能是打包及部署，或者是其他类似的作业。

在 Jenkins 中，构建作业很容 易建立，而且根据你的需要你可以安装各种插件，来创建多种形式的构建作业，下面我们先来学习创建自由式构建作业。

自由式的构建作业是最灵活和可配置的选项，并且可以用于任何类型的项目，它的配置 相对简单，其中很多配置在的选项也会用在其他构建作业中。

在 Jenkins 主页面，点击左侧菜单栏的“新建”或者“New job”

## 5.2 freestyle-job介绍

- 自由风格项目，比较灵活，可以构建任何形式的项目
- 在页面添加模块配置与参数
- 每个job仅能实现一个开发功能
- 无法将配置代码化，不利于Job配置迁移和版本控制
- 学习相对简单，用于执行简单任务

## 5.3 Jenkins创建freestyle项目

![image-20220710223852801](http://book.bikongge.com/sre/2024-linux/image-20220710223852801.png)

```
1.输入任务名字，job名字，定义一个有意义的名字，需要根据业务命名，如slb-nginx

注意，一旦确定job名字，就不要轻易修改了，jenkins一切皆文件

job名字已经被写入各种配置了，生成一系列的文件，文件夹

来看看刚才创建的freestyle的项目目录
[root@jenkins-100 /opt/plugins]#ls /var/lib/jenkins/jobs/
yuchao-freestyle-job

[root@jenkins-100 /opt/plugins]#ls /var/lib/jenkins/jobs/yuchao-freestyle-job/ -l
total 4
drwxr-xr-x 2 root root  23 Jul 11 06:38 builds
-rw-r--r-- 1 root root 468 Jul 11 06:38 config.xml


如果你修改了文件夹，这里会生成一个新的job文件夹配置，数据目录就会紊乱出错
```

勾选“丢弃旧的构建”，这是我们必须提前考虑的重要方面，就是我们如何处理构建历史，构建作业会消耗大理的磁盘空间

尤其是你存储的构建产物(比如执行 java 构建时会 生成的 JAR、WAR 等)

还有就是最大构建的任务个数，这里是5个

![image-20220710224300582](http://book.bikongge.com/sre/2024-linux/image-20220710224300582.png)

```
丢弃旧的构建提示

如果项目特别多，文件大，会造成jenkins数据目录过大，特别是编译类型的项目
    保持构建5天
    保留构建最大个数5个 

构建触发器
    手动构建
    定义触发器


构建环境

构建
    执行一系列的操作
    Execute shell
        执行linux命令
```

## 5.4 定义构建步骤

![image-20220710224556444](http://book.bikongge.com/sre/2024-linux/image-20220710224556444.png)

## 5.5 执行job，立即构建

![image-20220710224648316](http://book.bikongge.com/sre/2024-linux/image-20220710224648316.png)

第一次执行的日志记录

![image-20220710224803121](http://book.bikongge.com/sre/2024-linux/image-20220710224803121.png)

查看日志细节

![image-20220710224837837](http://book.bikongge.com/sre/2024-linux/image-20220710224837837.png)

## 5.6 查看jenkins的工作目录与构建结果

```
在上面的控制台输出内容中，我们可以看job执行时的当前工作目录是jenkins主目录+workspaces+job名称定义的目录。

目前的一个自由风格job工作区，job执行时，如果命令产生的数据，都会在这里，除非指定了其他的目录
/var/lib/jenkins/workspace/yuchao-freestyle-job


去命令行里看看
[root@jenkins-100 /opt/plugins]#ls /var/lib/jenkins/workspace/yuchao-freestyle-job/

空的，啥也没有，我们也没创建什么

job执行的命令也都的确执行了
[root@jenkins-100 /opt/plugins]#cat /tmp/linux.log 
超哥带你学linux，网址 www.yuchaoit.cn
```

## 5.7 第二次构建（修改构建命令）

![image-20220710225258562](http://book.bikongge.com/sre/2024-linux/image-20220710225258562.png)

多次执行试试，然后查看日志

![image-20220710225557362](http://book.bikongge.com/sre/2024-linux/image-20220710225557362.png)

## 5.8 小结gitlab的构建任务

```
例如运维可以使用jenkins的job功能，给其他不懂linux命令的同事，通过一键点击，就能够执行linux命令

例如服务重启等

可以有效的防止不懂linux的技术人员，在服务器上做奇怪的操作。
```