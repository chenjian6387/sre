#  07-流程控制之while循环

所有练习题，建议

1. 自己拿到需求主动思考，先尝试着写
2. 然后再去借鉴看超哥的代码思路
3. 自己再试试换一个写法

三部曲，冲冲冲。

# 1.while使用场景

```
1. 当明确循环的限定次数，用for、不确定循环次数使用while
2. 如循环让用户输入的登录程序
3. 如循环操作的一些菜单程序，直到用户输入结束指令菜单
```

# 2.while语法

```
while 条件测试 # 条件成立为true后执行循环体
do
    循环体
done
```

# 3.案例

## 循环限定范围

```
#!/bin/bash
# author: www.yuchaoit.cn

# 循环输出1~10

count=1
while [ $count -le 10 ]
do
    echo "数字：$count"
    let count++
done
```

写法2

```
#!/bin/bash
# author: www.yuchaoit.cn

# 循环输出1~10

count=1
while [ $count -le 10 ]
do
    echo "数字：$count"
    count=$[$count+1]
done
```

## 按行循环读取文本

测试数据，且添加行号

```
我是云南的 云南怒江的

爱你孤身走暗巷
爱你不跪的模样
爱你对峙过绝望
不肯哭一场
```

写法1

```
#!/bin/bash
# author: www.yuchaoit.cn

# 利用exec读取文本数据
exec < t.log
line_num=1
while read txt
do
    echo "$line_num $txt"
    let line_num++
done
```

写方法2

```
#!/bin/bash
# author: www.yuchaoit.cn


line_num=1
while read txt
do
    echo "$line_num $txt"
    let line_num++
done < t.log
```

写法3

```
#!/bin/bash
# author: www.yuchaoit.cn


line_num=1
cat t.log|while read txt
do
    echo "$line_num $txt"
    let line_num++
done
```

# 4.练习题

## 循环累加1到100

```
#!/bin/bash
# author: www.yuchaoit.cn


sum=0
count=1

while [ $count -le 100 ]
do
    sum=$[$sum+$count]
    let count++
done

echo "sum总和：$sum"
```

## 循环累加1到100的偶数

```
#!/bin/bash
# author: www.yuchaoit.cn


sum=0
count=0

while [ $count -le 100 ]
do
    sum=$[$sum+$count]
    count=$[count+2]
done

echo "sum总和：$sum"
```

反之奇数如何求？试试写法

## 1~9的计算表达式

```
完成1 ~ 9 的计算表达式

打印如下内容

9 * 9 = 81
8 * 8 = 64
7 * 7 = 49
6 * 6 = 36
5 * 5 = 25
4 * 4 = 16
3 * 3 = 9
2 * 2 = 4
1 * 1 = 1
```

代码

```
#!/bin/bash
# author: www.yuchaoit.cn

num=9
while [ $num -ge 1 ]
do
    echo "$num * $num = $[ $num* $num ]"
    num=$[$num-1]
done
```

再打印如下

```
9*1 =9
8*2 =16
7*3 =21
6*4 =24
5*5 =25
4*6 =24
3*7 =21
2*8 =16
1*9 =9
```

代码

```
#!/bin/bash
# author: www.yuchaoit.cn

n1=9
n2=1
while [ $n1 -ge 1 -a $n2 -le 9 ]
do
    echo "$n1 * $n2 = $[ $n1 * $n2 ]"
    n1=$[$n1-1]
    n2=$[$n2+1]
done
```

## 测试while条件成立

```
死循环提示用户输入账号
猜对后结束程序
```

代码

```
#!/bin/bash
# author: www.yuchaoit.cn

while [ "$u" != "sunwukong"  ]
do
    read -p "请输入账号 sunwukong：" u
done
```

## 读取文件数据，创建系统用户

测试数据

```
[root@www.yuchaoit.cn ~]#cat u.log
yuchao01:1500:1234567:devops
bob01:1501:1234567:sre
jack01:1502:123456:dba


创建用户，指定
username
uid
password
groups

要求做好用户检测，不得反复检测
```

脚本

```
#!/bin/bash
# author: www.yuchaoit.cn

exec < u.log
while read line 
do
    groups=$(echo $line|awk -F':' '{print $4}')
    uid=$(echo $line|awk -F':' '{print $2}')
    password=$(echo $line|awk -F':' '{print $3}')
    username=$(echo $line|awk -F':' '{print $1}')

    # 组判断
    grep "$groups" /etc/group &>/dev/null
    if [ $? -ne 0  ];then
        echo "该$groups 不存在，创建中.."
        groupadd $groups
    fi
    echo '============================================================'
    # 用户创建
    grep "$username" /etc/passwd  &>/dev/null
  if [ $? -ne 0  ];then
      echo "该$username 不存在，创建中.."
      useradd $username -u $uid -G $groups 
      echo $password | passwd --stdin $username 
  fi

  # 用户检查
  echo "用户信息是：$(id $username)"

done
```

## 猜数字

```
需求

1. 随机生成1~100数字
2. 提示用户只能输入数字，做好程序判断
3. 友好提示，猜大了、猜小了
4. 死循环，只有猜对后结束程序
5. 统计猜了几次（只统计正确输入）
```

代码

```
#!/bin/bash
# author: www.yuchaoit.cn

# 取余，限制随机数范围在0~99
r_num=$(echo $[RANDOM%100+1])

guess_count=0

# 程序开始
while true
do
    read -p "====下定离手！！请输入数字===：" num

# 限制数字
    if [ -z $(echo $num |sed -rn '/^[0-9]+$/p') ];then
        echo "大哥，请输入纯数字，你会不会玩啊？"
        continue
    fi
    # 每次循环，统计+1
    let guess_count++

  if [ $num -gt $r_num ];then
      echo "你太大了。。"
  elif [ $num -lt $r_num ];then
      echo "你太小了"
  else
      echo "恭喜你，猜对了！！！随机数字是$r_num，一共猜了$guess_count次！！"
      exit
  fi
done
```

## 跳板机开发

需求，如果不正确输入，程序不会停止。

```
#!/bin/bash
# author: www.yuchaoit.cn

# 禁止用户通过快捷键信号，终止脚本
# trap "" HUP INT QUIT TSTP

while true
do

echo -e "
超哥的堡垒机v1版，欢迎你兄弟
===============
|    1. web7   |
|    2. web8   |
|    3. db51   |
|    4. db52   |
|    5. exit   |
==============="

read -p "请输入您的选择：" key
case $key in
    1)
    ssh 172.16.1.7
    ;;

    2)
    ssh 172.16.1.8
    ;;

    3)
    ssh 172.16.1.51
    ;;

    4)
    ssh 172.16.1.52
    ;;

    5)
    exit
    ;;

    *)
            echo "only 1 ~ 5"

    esac
done
```