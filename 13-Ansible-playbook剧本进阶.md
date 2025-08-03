#  13-Ansible-playbook剧本进阶

# 剧本高级特性篇

# 循环

在写 playbook 的时候发现了很多 task 都要重复引用某个相同的模块，比如一次启动10个服务，或者一次拷贝10个文件，如果按照传统的写法最少要写10次，这样会显得 playbook 很臃肿。

如果使用循环的方式来编写 playbook ，这样可以减少重复编写 task 带来的臃肿。

```
https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html?highlight=loop
关于循环的标准用法

早期ansible教程中，关于循环关键字是with_item


ansible自2.5版本后，通过loop关键字提供循环功能

[root@master-61 /script]#ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```

## 创建多个系统用户

需求，在nfs机器组中创建5个用户test1~5，且均设置密码yuchao666

比较low的写法，看着都头疼，但是没办法，语法就是这样

（添加、或是删除用户，区别在于state=absent）

```
---
- name: create user test1~5
  hosts: nfs
  tasks:
    - name: create test1
      user: 
        name: test1
        state: absent
    - name: create test2
      user: 
        name: test2
        state: absent
    - name: create test3
      user: 
        name: test3
        state: absent
    - name: create tes4
      user: 
        name: test4
        state: absent
    - name: create test5
      user: 
        name: test5 
        state: absent
```

### 循环创建用户

```
--- 
- name: create test1~5
  hosts: nfs
  tasks:
    - name: create test1~5
      user:
        name: "{{ item }}"
        state: present
      loop:
        - test1
        - test2
        - test3
        - test4
        - test5
    - name: set password
      shell: echo 'yuchao666' |passwd --stdin "{{item}}"
      loop:
        - test1
        - test2
        - test3
        - test4
        - test5
```

### 循环删除用户

```
--- 
- name: create test1~5
  hosts: nfs
  tasks:
    - name: create test1~5
      user:
        name: "{{ item }}"
        state: absent
      loop:
        - test1
        - test2
        - test3
        - test4
        - test5
```

### 通过vars变量定义循环变量

上面会发现已然有重复的变量，还可以再简化

- 通过vars关键字定义用户列表，变量一般定义在任务开始之前
- 通过item关键字提取loop中每次循环的数据

循环创建用户且设置密码

```
---
- name: create user
  hosts: nfs
  vars:
     users_list:
      - test1
      - test2
      - test3
  tasks:
    - name: create user
      user:
        name: "{{ item }}"
        state: present
      loop: "{{ users_list }}"

    - name: set password
      shell: echo 'yuchao666' | passwd --stdin "{{ item }}"
      loop: "{{ users_list }}"
```

### 循环处理字典数据

创建用户以及用户id号

low办法超哥就不再写了，太墨迹

循环字典数据如下，字典就是key:value这样的数据

字典用法，主要是根据key、获取value

```
---
- name: create user
  hosts: nfs
  tasks: 
  - name: create user and uid
    user:
        name: "{{ item.user }}"
        uid: "{{ item.uid }}"


    loop:
    - {user: 'test1', uid: '2000'}
    - {user: 'test2', uid: '2001'}
    - {user: 'test3', uid: '2002'}
    - {user: 'test4', uid: '2003'}
```

### 写法2：通过vars定义字典数据

- vars定义字典数据
- loop提供循环功能，通过item变量提取循环数据

```yaml
---
- name: create user
  hosts: nfs
  vars:
    users_list:
    - {user: 'test1', uid: '2000'}
    - {user: 'test2', uid: '2001'}
    - {user: 'test3', uid: '2002'}
    - {user: 'test4', uid: '2003'}
  tasks:
  - name: create user and uid
    user:
      name: "{{ item.user}}"
      uid: "{{ item.uid }}"
    loop: "{{ users_list }}"
```

## 循环安装多个软件（yum基础环境安装）

```
在咱们期中综合架构开篇时，需要大家系统初始化，这个初始化步骤也是需要你做成ansible脚本的。
比如如下大量的基础软件，如何安装？

yum install -y tree wget bash-completion bash-completion-extras lrzsz net-tools sysstat iotop iftop htop unzip telnet ntpdate lsof
```

必然不能挨个的执行yum模块去安装，那得累死，循环写法如下

```
- name: yuchaoit.cn
  hosts: nfs
  remote_user: root
  tasks:
    - name: install basic packages
      yum:
        name: "{{ item }}"
        state: installed
      loop:
        - tree
        - wget 
        - bash-completion 
        - bash-completion-extras 
        - lrzsz 
        - net-tools 
        - sysstat 
        - iotop 
        - iftop
        - htop
        - unzip
        - telnet 
        - ntpdate
        - lsof
```

### 写法2：通过vars定义变量

```
- name: yuchaoit.cn
  hosts: nfs
  remote_user: root
  vars:
    basic_packages:
        - tree
        - wget 
        - bash-completion 
        - bash-completion-extras 
        - lrzsz 
        - net-tools 
        - sysstat 
        - iotop 
        - iftop
        - htop
        - unzip
        - telnet 
        - ntpdate
        - lsof
  tasks:
    - name: install basic packages
      yum:
        name: "{{ item }}"
        state: installed
      loop: "{{ basic_packages }}"
```

## rsync文件夹场景

```
比如部署nfs、rsync、nginx的综合剧本；
1.要安装多个软件
2.创建多个目录
3.复制多个目录
4.每个文件的权限都不一样
```

## 循环风格1：单行模式

比如rsync创建备份目录，有多个目录需要创建，普通的写法出现了诸多重复语句

```
- name: create data dir
  file: path=/data state=directory owner=www group=www

- name: create backup dir
  file: path=/backup state=directory owner=www group=www
```

## 循环风格2：缩进模式

上述创建备份目录的剧本语法，也可以用如下的缩进模式写，但是依然很多重复语句

```
- name: create data dir
  file:
    path: /data
    state: directory
    owner: www
    group: www

- name: create backup dir
  file:
    path: /backup
    state: directory
    owner: www
    group: www
```

改造为循环语句，使用yaml的缩进语法

```
- name: create data,backup dir
  file:
    path: "{{ item }}"
    state: directory
    owner: www
    group: www
  loop:
    - /data
    - /backup
```

## 循环风格3：混合语法

- 等号赋值语法
- 缩进语法

```
- name: create data backup 
  file: path="{{ item }}" state=directory owner=www group=www
  loop:
    - /data
    - /backup
```

## 循环风格4：循环结合字典取值

比如rsync服务部署，需要创建多个文件夹，以及对应的权限设置

```
- name: yuchaoit.cn
  hosts: backup
  tasks:
  - name: create_data
    file:
      path: "{{ item.file_path }}"
      state: directory
      owner: www
      group: www
      mode: "{{ item.mode }}"
    loop:
      - { file_path:'/data' ,mode:'755' }
      - { file_path:'/backup' ,mode:'755' }
```

# 变量定义

```
官网文档
https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html
```

在ansible中使用变量，能让我们的工作变得更加灵活，在ansible中，变量的使用方式有很多种，我们慢慢聊。

先说说怎样定义变量，变量名应该由字母、数字、下划线组成，变量名需要以字母开头，ansible内置的关键字不能作为变量名。

变量我们已经在循环知识中使用过了，主要通过vars关键字定义变量名、以及对应的变量值。

## 应用场景

```
1.通过自定义变量，可以在多个tasks中，通过变量名调用，取值
2.ansible在启动时，默认收集了客户端机器大量的静态属性（变量），可以提取该客户端的机器信息。
```

## vars自定义变量

定义多个文件夹变量，创建rsync的数据目录，配置文件

```
- hosts: backup
  vars:
    data_path: /data
    dest_path: /etc
    config_path: /etc/rsync.passwd
  tasks:
    - name: 01 mkdir data dir
      file:
        path: "{{ data_path }}" 
        state: directory

    - name: 02 copy  config file
      copy:
        src: "{{ config_path }}"
        dest: "{{ dest_path }}"
```

## 获取主机静态属性（ip地址）

```
内置获取ip的变量
ansible_all_ipv4_addresses #适用于多ip
ansible_default_ipv4.address #适用单ip
```

剧本

```
[root@master-61 ~]#cat get_ip.yaml 
- hosts: web
  tasks:
  - name: 01 get ip address
    debug: msg="该web组机器，ip是  {{ ansible_all_ipv4_addresses }}" 
  - name: 02 get hostname
    debug: msg="该web组，主机名是 {{ ansible_hostname }}"
  - name: 03 单ip
    debug: msg="{{ansible_default_ipv4.address }}"
  - name: 04 eth0 ip地址是
    debug: msg="{{ansible_facts.eth0.ipv4.address}}"
  - name: 05 eth1 ip地址是
    debug: msg="{{ansible_facts.eth1.ipv4.address}}"
```

## 完整的ansible内置变量手册

```
https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html
```

通过setup模块可以看到所有的内置变量，提取变量的值需要遵循python的数据类型语法，如列表还是字典取值

```
[root@master-61 ~]#ansible web -m ansible.builtin.setup |wc -l
```

## 主机清单文件中也用到了变量

1.主机清单文件中定义变量

```
[root@master-61 ~]#tail -20  /etc/ansible/hosts 

[all:vars]
ansible_port=22999
#ansible_user=root
#ansible_password=123123


[web:vars]
nginx_version='1.19'

[web]
172.16.1.7 port=22999 
172.16.1.8 port=22999
172.16.1.9 port=22999

[nfs]
172.16.1.31

[backup]
172.16.1.41
```

2.只要操作该主机组，即可使用该变量

```
- hosts: web
  tasks:
  - name: 01 get nginx port
    debug: msg="nginx port is {{port}}"
  - name: 02 get nginx version
    debug: msg="nginx version is {{nginx_version}}"
```

执行

```
[root@master-61 ~]#ansible-playbook get_nginx.yaml
```

## loop循环中引用变量

```
- hosts: backup
  tasks:
  vars: rsyncd_config=/script/rsyncd.conf rsyncd_pwd=/script/rsync.passwd
  tasks:
  - name: 01 copy config
    copy:
      src: "{{ item.src }}"
      dest: "{{ item.dest }}"
      mode: "{{ item.mode  }}"
    loop:
      - {src:"{{ rsyncd_config }}",dest:"/etc/",mode:'0644'}
      - {src:"{{ rsyncd_pwd }}",dest:"/etc/",mode:'0600'}
```

# 注册变量

ansible的模块在运行之后，其实都会返回一些"返回值"，只是默认情况下，这些"返回值"并不会显示而已

我们可以把这些返回值写入到某个变量中，这样我们就能够通过引用对应的变量从而获取到这些返回值了

这种将模块的返回值写入到变量中的方法被称为"注册变量"

那么怎样将返回值注册到变量中呢？我们来看一个playbook示例。

## 使用场景

```
调试，回显命令的执行结果
把状态保存为变量，其他task再继续调用
```

## 用内置变量获取IP地址写入文件，并且显示文件内容

```
- hosts: nfs
  tasks:
  - name: echo ip address
    shell: "echo {{ ansible_default_ipv4.address }} >> /tmp/ip.log"

  - name: cat ip.log
    shell: "cat /tmp/ip.log"
    register: about_ip_log

  - name: debug about_ip_log
    debug: 
      msg: "{{ about_ip_log.stdout_lines }}"
```

## 注册多个变量

同时记录且显示客户端的ip信息、主机名信息

结合循环知识，打印多个命令的结果

```
- name: yuchaoit.cn
  hosts: nfs
  tasks:
  - name: 01 get ip
    shell: "echo {{ ansible_default_ipv4.address }} > /tmp/ip.log"

  - name: 02 get hostname
    shell: "echo {{ ansible_hostname }} > /tmp/hostname.log"

  - name: 03 echo hostname
    shell: "cat /tmp/hostname.log"
    register: hostname_log

  - name: 04 echo ip
    shell: "cat /tmp/ip.log"
    register: ip_log

  - name: 05 show mount info
    shell: "showmount -e 172.16.1.31"
    register: showmount_log

  - debug:
      msg: "{{item}}"

    loop:
      - "{{ showmount_log.stdout_lines}}"
      - "{{ ip_log.stdout_lines}}"
      - "{{ hostname_log.stdout_lines}}"
```

## 判断当配置文件变化后，就重启服务

我们重启配置服务的标准是，修改了配置文件，否则无须重启

例如，判断rsyncd.conf文件状态发生变化后，就重启服务。

```
- name: yuchaoit.cn
  hosts: backup
  tasks:
  - name: 01 copy rsyncd.conf
    copy: src=/script/rsyncd.conf dest=/etc/
    register: conf_status

  - name: 02 start rsyncd.service
    systemd: name=rsyncd state=started enabled=yes


  - name: 03 restart rsyncd.service
    systemd:
      name: rsyncd
      state: restarted
    when: conf_status.changed
```

查看rsync进程端口是否更新，即为rsyncd服务是否重启

```
[root@master-61 ~]#ansible backup -m shell -a "netstat -tunlp|grep rsync"
172.16.1.41 | CHANGED | rc=0 >>
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN      635/rsync           
tcp6       0      0 :::873                  :::*                    LISTEN      635/rsync
```

修改rsyncd.conf，再次执行剧本

```
[root@master-61 ~]#ansible backup -m shell -a "netstat -tunlp|grep rsync"
172.16.1.41 | CHANGED | rc=0 >>
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN      8274/rsync          
tcp6       0      0 :::873                  :::*                    LISTEN      8274/rsync
```

# when条件判断语句

```
文档
https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html
```

使用场景

```
判断nfs配置文件是否存在

1.存在，则显示其文件内容
2.不存在，则输出 /etc/exports is not exists。
```

答案

```
- name: yuchaoit.cn
  hosts: nfs
  vars:
    nfs_file: /etc/exports
  tasks:
  - name: 01 check nfs config
    shell: "cat {{nfs_file}}"
    register: nfs_result
    ignore_errors: true

  - name: 02 debug nfs config
    debug:
      msg: "{{ansible_hostname}} has {{nfs_file}},file content is : {{nfs_result.stdout}}"
    when: nfs_result is success

  - name: 03 debug nfs not exists
    debug: msg="{{nfs_file}} is not exists."
    when: nfs_result is failed
```

# 高级特性handler

```
官网文档
https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html


Handlers: running operations on change
处理程序：对更改运行操作
```

handler这个机制一般用于服务状态管理，如

- 你在tasks中修改了nginx的配置文件，你就必须得重启nginx服务，才能生效

```
handler解决什么问题
1.现状、配置文件修改了，程序不可能自己重启，得手动restart


利用handler添加在剧本中，实现效果
1.配置文件没变化，就不执行restart重启
2.配置文件发生了变化，就执行restart重启
```

## low办法实现

```
[root@master-61 ~]#cat restart_rsync.yaml 
- name: yuchaoit.cn
  hosts: backup
  remote_user: root
  tasks:
  - name: copy rsyncd.conf
    copy:
      src=/script/rsyncd.conf
      dest=/etc/
  - name: restart rsyncd.service
    systemd:
      name=rsyncd
      state=restarted
```

你会发现，这个用法，无论你配置文化改没改，都必然会执行重启的task，这个写法就不太合理。

## 改造为handler

```
1.handlers中的任务会被tasks调用
2.只有tasks的确执行了，发生了change状态，handler才会执行
3.在tasks任务列表中，定义notify属性，用于触发handler的执行
```

剧本

```
- name: yuchaoit.cn
  hosts: backup
  remote_user: root
  tasks:
  - name: 01 copy rsyncd.conf
    copy:
      src=/script/rsyncd.conf
      dest=/etc/
    notify:
      - restart rsyncd.service

  handlers:
  - name: restart rsyncd.service
    systemd:
      name: rsyncd
      state: restarted
```

细节注意

```
1.handlers必须写在结尾
2.handlers定义的任务名字，必须和notify一致
```

# 给task打上tag标签

你写了一个很长的playbook，其中有很多的任务，这并没有什么问题，不过在实际使用这个剧本时，你可能只是想要执行其中的一部分任务而已

或者，你只想要执行其中一类任务而已，而并非想要执行整个剧本中的全部任务

这个时候我们该怎么办呢？我们可以借助tags实现这个需求。

见名知义，tags可以帮助我们对任务进行'打标签'

当任务存在标签以后，我们就可以在执行playbook时，借助标签，指定执行哪些任务，或者指定不执行哪些任务了。

```
tag作用
调试，选择性的执行某个task
```

## 1.部署nfs-server剧本参考

```
- name: yuchaoit.cn
  hosts: nfs
  tasks:
  - name: 01 安装nfs-utils 服务
    yum: name=nfs-utils state=installed
    tags: 01_install_nfs_service

  - name: 02 安装rpcbind 服务
    yum: name=rpcbind state=installed
    tags: 02_install_rpcbind_service

  - name: 03 创建组
    group: name=www gid=666
    tags: 03_add_group

  - name: 04 创建用户
    user: name=www uid=666 group=www create_home=no shell=/sbin/nologin
    tags: 04_add_user

  - name: 05 创建共享目录
    file: path=/data owner=www group=www state=directory
    tags: 05_create_data_dir

  - name: 06 拷贝配置文件
    copy: src=/script/exports dest=/etc/exports
    tags: 06_copy_nfs_exports

  - name: 07 创建关于rsync密码文件
    copy: content='yuchao666' dest=/etc/rsync.passwd mode=600
    tags: 07_create_rsync_passwd

  - name: 08 启动rpcbind
    service: name=rpcbind state=started enabled=yes
    tags: 08_start_rpcbind

  - name:  09 启动nfs
    systemd: name=nfs state=started enabled=yes
    tags: 09_start_nfs
```

## 2.打印剧本中可用的标签

也就是你可以直接选择执行哪些任务

```
[root@master-61 ~]#ansible-playbook --list-tags tag_nfs.yaml 

playbook: tag_nfs.yaml

  play #1 (nfs): yuchaoit.cn    TAGS: []
      TASK TAGS: [01_install_nfs_service, 02_install_rpcbind_service, 03_add_group, 04_add_user, 05_create_data_dir, 06_copy_nfs_exports, 07_create_rsync_passwd, 08_start_rpcbind, 09_start_nfs]
```

## 3.指定运行某个标签

```
[root@master-61 ~]#ansible-playbook -t 01_install_nfs_service tag_nfs.yaml 
[root@master-61 ~]#ansible-playbook -t 04_add_user tag_nfs.yaml
```

## 4.指定运行多个标签

```
[root@master-61 ~]#ansible-playbook -t 01_install_nfs_service,05_create_data_dir,04_add_user tag_nfs.yaml
```

## 5.指定不运行一个/多个标签

```
[root@master-61 ~]#ansible-playbook --skip-tags  01_install_nfs_service,05_create_data_dir,04_add_user tag_nfs.yaml
```

# 选择tasks执行

## 使用场景

```
1.调试剧本时，task数量太多，不想从头执行，可以指定执行位置
```

## 查看task列表

```
[root@master-61 ~]#ansible-playbook --list-tasks tag_nfs.yaml 

playbook: tag_nfs.yaml

  play #1 (nfs): yuchaoit.cn    TAGS: []
    tasks:
      01 安装nfs-utils 服务    TAGS: [01_install_nfs_service]
      02 安装rpcbind 服务    TAGS: [02_install_rpcbind_service]
      03 创建组    TAGS: [03_add_group]
      04 创建用户    TAGS: [04_add_user]
      05 创建共享目录    TAGS: [05_create_data_dir]
      06 拷贝配置文件    TAGS: [06_copy_nfs_exports]
      07 创建关于rsync密码文件    TAGS: [07_create_rsync_passwd]
      08 启动rpcbind    TAGS: [08_start_rpcbind]
      09 启动nfs    TAGS: [09_start_nfs]
```

## 选择执行的task位置

从第五步骤开始

```
[root@master-61 ~]#ansible-playbook --start-at-task '05 创建共享目录' tag_nfs.yaml
```

# 总结（playbook规范流程）

> 通过这个流程，去编写、检验、阅读所有的playbook都是一个靠谱的办法
>
> 无论是你自己写好剧本，进行校验
>
> 还是工作后，先看公司里现成的剧本，进行维护，都可以基于这个思路

## 1.检查剧本语法

如果语法不对，ansible会具体告诉你错误的位置

```
[root@master-61 ~]#ansible-playbook --syntax-check tag_nfs.yaml 

playbook: tag_nfs.yaml
```

## 2.检查该剧本操作的主机有哪些

搞清楚这个剧本会影响到哪些主机，关乎于这些机器的作用

```
[root@master-61 ~]#ansible-playbook --list-hosts get_ip.yaml 

playbook: get_ip.yaml

  play #1 (web): web    TAGS: []
    pattern: [u'web']
    hosts (3):
      172.16.1.7
      172.16.1.8
      172.16.1.9
```

## 3.查看剧本有哪些任务

轻松的搞清楚这个剧本有什么作用，整体的工作流程

```
[root@master-61 ~]#ansible-playbook --list-tasks tag_nfs.yaml 

playbook: tag_nfs.yaml

  play #1 (nfs): yuchaoit.cn    TAGS: []
    tasks:
      01 安装nfs-utils 服务    TAGS: [01_install_nfs_service]
      02 安装rpcbind 服务    TAGS: [02_install_rpcbind_service]
      03 创建组    TAGS: [03_add_group]
      04 创建用户    TAGS: [04_add_user]
      05 创建共享目录    TAGS: [05_create_data_dir]
      06 拷贝配置文件    TAGS: [06_copy_nfs_exports]
      07 创建关于rsync密码文件    TAGS: [07_create_rsync_passwd]
      08 启动rpcbind    TAGS: [08_start_rpcbind]
      09 启动nfs    TAGS: [09_start_nfs]
```

## 4.模拟剧本执行

模拟执行，查看执行流程是否存在错误，以及执行的状态

```
[root@master-61 ~]#ansible-playbook -C tag_nfs.yaml
```

## 5.真正执行剧本

对目标机器发生实质性的改变、修改操作

```
[root@master-61 ~]#ansible-playbook  tag_nfs.yaml
```

# ansible剧本大作业（吐血提醒）

> 等你坐在访客室、开始1v1和面试官交流面试
>
> 你就会回想起，当初于超老师让我好好学ansible，结合备份同步访问写剧本，以及让我锻炼口述能力，叙述架构流程的重要性，悔不当初啊！！
>
> 以及面试官问你
>
> ansible熟悉吗？写下你用过的哪些模块？
>
> 用过ansible高级特性吗？说一说有哪些
>
> 你心中一万个羊驼。。。超哥当时怎么教来着，我为什么没好好做作业，害！

## 使用学过的模块

```
如
shell
copy
file
group
user
script
yum
systemd
等
```

## 尽量结合剧本高级特性

使用高级特性，是为了简化脚本写法，提升可维护性，同时也会提升阅读难度，需要额外学习。

```
循环 loop
变量 vars
注册变量 register
条件语句 when
触发任务 handler
标签tag
```

## 综合练习

```
继续改造 ，完全改造为ansible-playbook 一键部署

rsync
nfs
lsyncd
nginx
```