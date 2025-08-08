# 05-jenkins结合远程仓库

![image-20220711135045265](http://book.bikongge.com/sre/2024-linux/image-20220711135045265.png)

既然是持续集成，对代码进行构建，我们得获取代码仓库的内容，这里选择我们搭建的gitlab服务器

# 1.开发工程师的机器

```
1. 在window上生成ssh-key

$ ssh-keygen.exe -t rsa -C 'www.yuchaoit.cn'


2.添加到代码仓库github/gitlab都玩一玩，公司用这俩居多
```

![image-20220711135631351](http://book.bikongge.com/sre/2024-linux/image-20220711135631351.png)

gitlab上添加该机器的ssh-key允许上传代码，咱这里就不区分多个账户，多个权限了，都先基于root账户，实现部署流程，理解jenkins是怎么工作的。

![image-20220711193730723](http://book.bikongge.com/sre/2024-linux/image-20220711193730723.png)

# 2.gitlab新建项目

![image-20220711194015936](http://book.bikongge.com/sre/2024-linux/image-20220711194015936.png)

# 3.开发提交代码

1.准备好代码

```
这里利用python程序，flask代码做实验

# coding:utf-8

from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello 超哥带你学linux www.yuchaoit.cn</h1>'

if __name__ == '__main__':
    app.run()
```

2.克隆gitlab代码仓库到本地

```bash
1.先设置git身份
Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop
$ git config --global user.name "laoliu"

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop
$ git config --global user.email "yc_uuu@163.com"





2.克隆代码

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop
$ git clone git@10.0.0.99:root/my_flask.git
Cloning into 'my_flask'...
The authenticity of host '10.0.0.99 (10.0.0.99)' can't be established.
ED25519 key fingerprint is SHA256:fsFB+VUXvu9atyktELhNhs0zzRdli9XbqehOn2We9yo.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.99' (ED25519) to the list of known hosts.
warning: You appear to have cloned an empty repository.

3.开发代码，进行提交，推送到代码仓库

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ ls
my_app.py

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ cat my_app.py

# coding:utf-8

from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello 超哥带你学linux www.yuchaoit.cn</h1>'

if __name__ == '__main__':
    app.run(host="0.0.0.0")


4.版本提交，推送gitlab
$ git add .

$ git commit -m 'my_app.py 首次开发'
[master (root-commit) 3e973ff] my_app.py 首次开发
 1 file changed, 11 insertions(+)
 create mode 100644 my_app.py


Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ git push -u origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 20 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 402 bytes | 402.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To 10.0.0.99:root/my_flask.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

# 4.检查代gitlab码仓库

![image-20220711195023076](http://book.bikongge.com/sre/2024-linux/image-20220711195023076.png)

# 5.配置jenkins的job获取代码仓库

![image-20220711195924339](http://book.bikongge.com/sre/2024-linux/image-20220711195924339.png)

```
给jenkins服务器安装git
[root@jenkins-100 ~]#yum install git -y
```

![image-20220711200023661](http://book.bikongge.com/sre/2024-linux/image-20220711200023661.png)

```
添加jenkins服务器的ssh-key到gitlab服务器
[root@jenkins-100 ~]#ssh-keygen -t rsa -C "www.yuchaoit.cn"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:WRnTFIqkwh7IXB2gWyFkrkE1BLwx9bYHtD9YRYFdbgk www.yuchaoit.cn
The key's randomart image is:
+---[RSA 2048]----+
|.+O=++..=E=+o.   |
|.O *+.o+.oo=o    |
|. X ==... ++     |
| + +.o*  o.      |
|. . .o +S        |
|      . .        |
|                 |
|                 |
|                 |
+----[SHA256]-----+


[root@jenkins-100 ~]#cat ~/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDA9C597NnGpdyRYkDtF4zQmTa+bRxXqll3XX7LJDjLfsgfUZbfolj0KwkmdIvpQjecDrKff33bOIhGQQ64okmQlKPyp+iISO6sRCH1p2VhZNFEWOeBRtzA+TFrLX4WeVFJFg2IuOE1cFuKGESBC7pqZZf4H12QaNCunLwWLTrqoUGvfW0+rXOBGaXPW1yNpTMevnPkN81ZKiqhONtUE+suYwwYi8zgi54CXZZBNEcyXhZH2gLLser/hy+16vqYZ65enGBcfPYBNSHt35DcNs/Qs6nLpT/UBxblQwFI5ktq7C6cm6igYVAuVpomDNdD+LCjvRhijQBCbxlvHwXcO9Tl www.yuchaoit.cn
```

![image-20220711200204258](http://book.bikongge.com/sre/2024-linux/image-20220711200204258.png)

添加jenkins的认证凭证，使用自己的私钥即可

![image-20220711200424898](http://book.bikongge.com/sre/2024-linux/image-20220711200424898.png)

------

![image-20220711200505101](http://book.bikongge.com/sre/2024-linux/image-20220711200505101.png)

至此，jenkins就可以下载gitlab的代码了。

试试看能下载什么，点击构建

![image-20220711201921165](http://book.bikongge.com/sre/2024-linux/image-20220711201921165.png)

# 6.开发脚本实现项目部署

注意是，jenkins > 目标机器(web-7测试)

```
# 注意免密登录了
ssh-copy-id root@10.0.0.7
```

部署代码

若是做更多考虑，可以先成脚本，然后远程去调用。

```
#!/bin/bash
# author: www.yuchaoit.cn

# 发送代码到目标机器
cd /var/lib/jenkins/workspace/my_flask  && scp my_app.py root@10.0.0.7:/opt/

# 给远程机器部署python3环境，代码运行环境
ssh root@10.0.0.7 "yum install python3 python3-devel python3-pip"
ssh root@10.0.0.7 "pip3 install flask"


# 远程启动进程，后台运行
# 重启进程
ssh root@10.0.0.7 "pkill python3"

ssh root@10.0.0.7 "nohup /usr/bin/python3 /opt/my_app.py >/dev/null 2>&1 &"
```

![image-20220711203603889](http://book.bikongge.com/sre/2024-linux/image-20220711203603889.png)

# 7.测试访问web7的flask项目

![image-20220711203652300](http://book.bikongge.com/sre/2024-linux/image-20220711203652300.png)

# 8.完成项目更新（鼠标一点，自动更新，操心啥啊）

让开发去写代码就好了

```
Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ vim my_app.py

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ git add .
warning: in the working copy of 'my_app.py', LF will be replaced by CRLF the next time Git touches it

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ git commit -m '更新代码'
[master 5ff3d74] 更新代码
 1 file changed, 1 insertion(+), 1 deletion(-)


推送代码

Sylar@DESKTOP-G6C412R MINGW64 ~/Desktop/my_flask (master)
$ git push -u origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 20 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 301 bytes | 301.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To 10.0.0.99:root/my_flask.git
   3604269..5ff3d74  master -> master
branch 'master' set up to track 'origin/master'.
```

# 9.鼠标一点，下班下班

![image-20220711204054658](http://book.bikongge.com/sre/2024-linux/image-20220711204054658.png)