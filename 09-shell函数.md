#  09-shell函数

# 1.函数作用

函数是一个非常实用的技能，用于封装代码块，复用代码，省去同一段代码，重复写，导致代码像一块烂抹布；

封装函数后，代码立刻化身为高级绸缎！

```
shell代码，自上而下

先定义、后调用
```

# 2.函数定义与调用

方法1，完整写法

```
function hello(){

    echo "hello chaoge linux."
}


function ngx_status(){
    systemctl status nginx
    ps -ef|grep nginx
}



# 执行函数
hello
ngx_status
```

方法2，懒人写法

```
hello(){
    echo "welcome my linux~~ www.yuchaoit.cn"
}


start(){
    systemctl start nginx
    ps -ef|grep nginx
    netstat -tunlp|grep nginx

}


# 先后调用函数
start
hello
```

# 3.函数传参

```
1. 这里的函数传参，是指单独给函数传递执行参数，和给脚本传入参数是两码事
2. 函数传参是指，函数在执行的时候，可以传入位置参数，这样函数连带参数一起执行。
```

## 3.1 函数传参语法（细节）

```
#!/bin/bash 

function hello(){
    echo "函数开始执行"
    # 注意这里的参数，
    echo "函数体中接收的参数1 ：" $1
    echo "函数体中接收的参数2 ：" $2
}

# 这里传入的是函数参数
hello laoliu laoba

echo "函数外，可以正常的接收位置参数1：" $1
echo "函数外，可以正常的接收位置参数2：" $2
```

![image-20220626204947157](/ajian/image-20220626204947157.png)

## 3.2 计算器函数（注意参数语法）

```
# 写一个函数，接收函数参数

#!/binbash

calc(){
        case $2 in
        +)
            echo "$1 + $3 = $[$1+$3]" ;;

        -)
            echo "$1 - $3 = $[$1-$3]" ;;

        ×)
            echo "$1 * $3 = $[$1*$3]" ;;            
        /)
            echo "$1 / $3 = $[$1/$3]" ;;
        *)
            # $0 依然是指文件名
            echo "usage: bash $0 num1 {+|-|×|/} num2"
        esac
}

# 调用函数，传参、且传递脚本的位置参数
calc $1 $2 $3
```

![image-20220626210815691](/ajian/image-20220626210815691.png)

# 4.函数实战练习

## 4.1 nginx管理脚本

```
#!/bin/bash

Usage(){
    echo "Usage： bash $0 {start|stop|restart}"
}


start_nginx(){
    echo "nginx启动中"
}

stop_nginx(){

    echo "nginx已关闭"
}

# 接收用户输入指令
case $1 in
start)
    start_nginx ;;
stop)
    stop_nginx ;;
restart)
    stop_nginx
    start_nginx
    ;;
*)
    Usage
esac
```

![image-20220626213037705](/ajian/image-20220626213037705.png)

## 4.2 多级菜单版

核心功能就在于条件判断的嵌套

```
包括while循环、函数、break、continue的总和用法
```

代码

```bash
#!/bin/bash
# author: www.yuchaoit.cn


# 菜单封装函数，便于调用

menu1(){

echo -e "
=============欢迎来到于超老师的linux课程========
兄弟，请你按照如下规则，输入选项
1. Install Nginx
2. Install Mysql
3. Install Redis
4. bye
============================================
"
}


nginx_menu(){
echo -e "
===============请选择对于软件版本==============
1. Install Nginx1.15
2. Install Nginx1.16
3. Install Nginx1.17
4. 返回上一层
=============================================
"
}


mysql_menu(){
    echo "mysql菜单还在开发中....."
}




# 程序打印菜单1
menu1


while true
do
    # 用户选择菜单1
    read -p "您请输入对应的序号：" num1
    # 一级条件判断
    case $num1 in

    1)
        # 进入菜单2
        nginx_menu
        while true
        do
            read -p "请选择对应的nginx版本：" num2
        case $num2 in
            1)
              echo "成功安装nginx 1.15版本！！" 
              ;;
            2)
              echo "成功安装nginx 1.16版本！！" 
              ;;
            3)
              echo "成功安装nginx 1.7版本！！"
              ;;
            4)
              # 清屏，返回上一层
              clear
              menu1
              # 中断二级菜单的循环
              break
              ;;
             *)
              echo "请按规则填写 1 ~ 4 序号...."
         esac
            done
            ;;
    2)
        mysql_menu
        ;;
    3)
        echo "redis菜单于超老师努力开发中。。。"
        ;;
    4)
        echo "bye bye 。"
        exit
        ;;
    *)
        echo "请按规则输入菜单序号1 ~ 4 ！！"
        continue
    esac
done
```
