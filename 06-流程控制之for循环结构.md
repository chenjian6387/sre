#  06-流程控制之for循环结构

# 1.for循环使用场景

```
1. 需要反复、重复执行的任务
2. 如创建100个用户，打印一百遍 chaoge666、插入数据库一万条数据等。
```

# 2.for语法

```
for 变量名 in 取值列表
do
    每次循环要执行的命令
done


# for默认以空格分割独立的元素
```

# 3.for循环几个场景

## 3.1 循环多个字符串参数

场景1，多个单个参数

```
#!/bin/bash
for cai in "红烧肉" "酱牛肉" "烤羊肉串" "烤土豆片"
do
    echo "菜品：" $cai
done
```

场景2，不一样的写法，如下结果是什么？

```
#!/bin/bash
for cai in "红烧肉 五花肉 爆炒腰花" "酱牛肉 酱牛毽子" "烤羊肉串 烤羊宝" "烤土豆片 烤菜花" 
do
    echo "菜品：" $cai
done
```

## 3.2 从变量中循环取值

简单语法

```
#!/bin/bash

all_cal="红烧肉 五花肉 爆炒腰花 酱牛肉 酱牛毽子 烤羊肉串 烤羊宝 烤土豆片 烤菜花"

for cai in $all_cal
do 
    echo $cai
done
```

需求：提取现有机器上PATH变量的每一个路径。

```
#!/bin/bash
local_path=$(echo $PATH | sed 's/\:/ /g')
for p in $local_path
do
    echo $p
done
```

## 3.3 读取花括号序列

```
#!/bin/bash
for num in {0..15}
do
echo "循环的数字是：$num"
done
```

写法2，c语言风格，了解即可

```
#!/bin/bash

for (( num=0;num<15;num++ ))
do
    echo "循环的数字是：${num}"
done
```

## 3.4 从文件读取数据

> 注意，for循环是以遇见空格来进行换行，确认每一个元素，因此注意这个细节。

```
cat > shaokao.log <<'EOF'
烤羊头 烤羊肉串 烤羊宝
烤韭菜
烤土豆
EOF

# 循环读取
for cai in $(cat shaokao.log)
do
    echo "老板,来俩: $cai"
done
```

## 3.5 循环创建用户

```
#!/bin/bash
for num in {1..10}
do
    # 创建用户
    echo "$(useradd yu${num})"
    # 创建密码
    echo "$(echo pwd${num}|passwd --stdin yu${num})"
done
```

## 3.6 循环删除用户

```
#!/bin/bash
for user in yu{1..10}
do
    userdel ${user}
done
```

## 3.7 使用case开发用户创建脚本

```
提供菜单选项
1. useradd
2. userdel
以及输入用户名前缀，即可快速创建10个用户
```

代码

```
#!/bin/bash
echo -e "====用户快捷创建系统=====
1. 用户创建
2. 用户删除"

read -p "请选择你要执行的操作：" choice

case $choice in
1)
    read -p "请输入用户名前缀：" user
    read -p "请输入密码前缀：" pwd
    # 循环创建10个用户
    for num in {1..10}
    do
        useradd ${user}${num}
        echo ${pwd}${num}|passwd --stdin ${user}${num}
    done
    ;;

2)
    read -p "请输入要删除的用户名前缀：" del_user
    for num in {1..10}
    do
        userdel -r ${del_user}${num}
    done
    ;;
*)
    echo "只支持1~2选项！！"
esac
```

## 3.8 读取密码本，创建用户

密码本

```
cat >user_info.log<<'EOF'
user1:pass1
user2:pass2
user3:pass3
user4:pass4
user5:pass5
user6:pass6
user7:pass7
user8:pass8
user9:pass9
user10:pass10
EOF
```

开发脚本读取密码本，创建用户且设置密码

```
for i in $(cat user_info.log)
do
    # 提取用户名
    user=$(echo $i|awk -F':' '{print $1}')
    # 创建用户
    useradd ${user}
    # 提取密码
    pwd=$(echo $i|awk -F':' '{print $2}')
    # 创建密码
    echo ${pwd}|passwd --stdin ${user}
done
```

## 3.9 主机存活检测

这种写法是单进程检测

```
#!/bin/bash
for i in {1..254}
do
    ping -w 1 192.168.3.${i} > /dev/null 2>&1
    # 存活判断
    if [ $? == 0 ];then
        echo "172.16.1.${i} 运行中" > online_ip.log
    fi
done


#!/bin/bash
for i in {1..254}
do
    ping -w 1 192.168.3.${i} 
    # 存活判断
    if [ $? == 0 ];then
        echo "172.16.1.${i} 运行中" 
    fi
done
```

优化写法，全部放入后台执行，并发执行。

可以通过jobs查后台进程列表。

```
#!/bin/bash
> ip.txt

for ip in {1..254}
do  
   ping -w 1 192.168.3.${ip} >> ip.txt &
done

echo "存活主机如下:"
# 寻找有数据包返回的行就行了
awk -F"[: ]" '/icmp/{print $4}' ip.txt
```

## 3.10 提取于超老师博客园博文链接

```
# 思路

# 步骤1.下载超哥博客园首页html，里面有相关文章的url，提取即可
curl -s https://www.cnblogs.com/pyyu -o www.yuchaoit.cn.html


# 思路1，提取出关于具体文章的url，请用多种命令提取。
awk -F'\"' '/postTitle2/{print $4}' www.yuchaoit.cn.html > all_url.log
grep 'postTitle2' www.yuchaoit.cn.html |grep -o 'ht.*l'
grep 'postTitle2' www.yuchaoit.cn.html |sed -r 's/(^.*)="(.*)">/\2/g'
grep 'postTitle2' www.yuchaoit.cn.html |sed -r 's/.*="(.*)">/\1/g'



# 思路2，提取出文章的标题
sed -rn '/postTitle2/,/<\/span>/p' www.yuchaoit.cn.html |grep -Ev '[<>]' | sed 's/ //g' > all_title.log

# 思路3，拼接url和标题（paste命令）
[root@www.yuchaoit.cn ~]#paste -d " " all_title.log all_url.log  > chaoge_blog.log
[root@www.yuchaoit.cn ~]#
[root@www.yuchaoit.cn ~]#cat chaoge_blog.log |column -t

# 思路4，shell拼接url和标题（遍历数组写法）
all_url=($(cat all_url.log))
all_title=($(cat all_title.log))
url_num=$(cat all_url.log|wc -l)
for (( i=0;i<$url_num;i++ ))
do
    echo -e "【于超老师博客】${all_url[i]} \t  ${all_title[i]}"
done



# 思路5
title_num=$(cat all_title.log|wc -l)
# 循环用sed打印每一行，拼接即可
for line_num in $(seq 1 ${title_num})
do
    each_title=$(sed -n "${line_num}"p all_title.log)
    each_url=$(sed -n "${line_num}"p all_url.log)
    echo -e "${each_title} \t ${each_url}" >> chaoge_blog.log
done

# 查看结果
cat chaoge_blog.log |column -t
```

## 3.11 mysql数据库分库分表备份

mysql备份方案1，全库、全表备份

```
mysqldump -uroot -pwww.yuchaoit.cn -A -B --single-transaction > /opt/all-db.sql
一般备份时都会进行压缩处理，以节省磁盘空间，如下

mysqldump -uroot -pwww.yuchaoit.cn -A -B --single-transaction |  gzip >/opt/all-db.sql
```

还有一种备份方案，可以按不同的库，以及对于库中的表，进行备份，更为独立。

```
现有的数据库
[root@db-51 ~]#mysql -uroot -pwww.yuchaoit.cn 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)
```

库和表的结构是

```
如wordpress数据库的数据表如下

MariaDB [(none)]> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [wordpress]> show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
12 rows in set (0.00 sec)
```

### 需求

```
1. 将wordpress数据库单独备份到 /backup_mysql/wordpress/
2. 以及对应的数据表
/backup_mysql/wordpress/wp_posts.sql
```

### 思路

```
1. 查看库下的表
[root@db-51 ~]#mysql -uroot -pwww.yuchaoit.cn -e "show tables from wordpress;"

2. 单独导出库中的表数据
mysqldump -uroot -pwww.yuchaoit.cn  wordpress wp_posts > /backup/wordpress/wp_posts.sql
```

脚本开发

```bash
#!/bin/bash
now=$(date +%F-%T)
table_list=$(mysql -uroot -pwww.yuchaoit.cn -e "show tables from wordpress;" |grep -v 'Tables')

if [ ! -d /backup/wordpress/ ];then
    mkdir -p /backup/wordpress/
fi

for table in ${table_list}
do
    mysqldump -uroot -pwww.yuchaoit.cn  wordpress ${table} > /backup/wordpress/${table}.sql.${now}
done
```