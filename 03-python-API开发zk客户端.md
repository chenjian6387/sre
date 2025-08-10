# 03-python-API开发zk客户端

```
前面于超老师讲完了，zk运维的基本命令行玩法，更多的还是开发需要通过代码和zk结合处理。
大多数场景是java后端去操作。
这里我们以运维更友好的python来学习。
```

# 1.kazoo模块

zookeeper是一个用于维护配置信息、命名、提供分布式同步和提供组服务。它自身是高可用的，只要宕机节点不达到半数，zookeeper服务都不会离线。zookeeper为实现分布式锁，分布式栅栏，分布式队列，安全的配置存储交换，在线状态监控，选举提供了坚实的基础。

在hadoop环境中，zookeeper被广泛应用。hadoop 高可用是依赖zookeeper的实现的。Hbase，storm，kafka都强依赖zookeeper，没有zookeeper根本都运行不起来。阿里的微服务治理框架dubbo也是依赖zookeeper的。

python访问zookeeper使用的的模块是kazoo。

```
模块文档
https://kazoo.readthedocs.io/en/latest/api/client.html
```

完成功能

```
1. 会话连接、恢复
2. 节点增删改查
3. watch、acl操作
```

## kazoo获取zk服务端信息

```
1. 模块安装
pip3 install kazoo

# 于超老师这里的版本
kazoo==2.6.0
zookeeper==version--- (3, 5, 6)


2.链接zk

# -*- coding: UTF-8 -*-
'''
www.yuchaoit.cn with python ,zookeeper
'''
import sys
from kazoo.client import KazooClient, KazooState
import logging

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

# 开始心跳
zk.start()
# 获取子节点
znodes = zk.get_children('/')
print(znodes)


# 开始心跳
zk.start()
# 获取根节点数据和状态
data,stat=zk.get('/')
print('data---',data)
print('stat---',stat)

============
# 输出结果

data--- b''
stat--- ZnodeStat(czxid=0, mzxid=0, ctime=0, mtime=0, version=0, cversion=7, aversion=0, ephemeralOwner=0, dataLength=0, numChildren=3, pzxid=30064771073)



'''
这个是stat的输出：
ZnodeStat(czxid=0, mzxid=0, ctime=0, mtime=0, version=0, cversion=8448, aversion=0, ephemeralOwner=0, dataLength=0, numChildren=4, pzxid=30064771073)
ZnodeState的属性列表:
czxid ： 创建这个节点时的zxid
mzxid : 修改这个节点时的zxid
ctime ： 创建时间
mtime : 修改时间
version : 数据被修改的次数
cversion: 子节点被修改的次数
aversion: acl被改变的次数
ephemeralOwner:临时节点创建的用户，如果不是临时节点值为0
dataLength:节点数据长度
numChildren:子节点的数量
pzxid:子节点被修改的zxid
'''

#获取根节点的所有子节点，返回的是一个列表，只有子节点的名称
children = zk.get_children("/");
print(children)

#下面是根节点的返回值


#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()
```

![image-20221114151206987](/ajian/image-20221114151206987.png)

## 增删改查

### 增、查

```
# -*- coding: UTF-8 -*-
'''
www.yuchaoit.cn with python ,zookeeper
'''
import sys
from kazoo.client import KazooClient, KazooState
import logging

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

# 开始心跳
zk.start()

# 创建节点
# zk.create('/kazoo/yu1',b'good linux ,www.yuchaoit.cn',makepath=True) # 创建节点：makepath 设置为 True ，父节点不存在则创建，其他参数不填均为默认

znodes=zk.get_children('/kazoo')
print(znodes)
data=zk.get('/kazoo/yu1')
print(data)


#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()
```

### 修改节点

```
# -*- coding: UTF-8 -*-
'''
www.yuchaoit.cn with python ,zookeeper
'''
import sys
from kazoo.client import KazooClient, KazooState
import logging

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

# 开始心跳
zk.start()

# 创建节点
# zk.create('/kazoo/yu1',b'good linux ,www.yuchaoit.cn',makepath=True) # 创建节点：makepath 设置为 True ，父节点不存在则创建，其他参数不填均为默认

# znodes=zk.get_children('/kazoo')
# print(znodes)
# data=zk.get('/kazoo/yu1')
# print(data)

zk.set('/kazoo/yu1',b'not bad linux,www.yuchaoit.cn')
data,stat=zk.get('/kazoo/yu1')
print('节点数据提取：',data)

#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()
```

### 删除

```
# -*- coding: UTF-8 -*-
'''
www.yuchaoit.cn with python ,zookeeper
'''
import sys
from kazoo.client import KazooClient, KazooState
import logging

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

# 开始心跳
zk.start()

# 创建节点
# zk.create('/kazoo/yu1',b'good linux ,www.yuchaoit.cn',makepath=True) # 创建节点：makepath 设置为 True ，父节点不存在则创建，其他参数不填均为默认

# znodes=zk.get_children('/kazoo')
# print(znodes)
# data=zk.get('/kazoo/yu1')
# print(data)

# zk.set('/kazoo/yu1',b'not bad linux,www.yuchaoit.cn')
# data,stat=zk.get('/kazoo/yu1')
# print('节点数据提取：',data)


# 删除节点
# 不删除父节点

zk.delete('/kazoo/yu1',recursive=False)

#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()
```

### watchers事件

```
# -*- coding: UTF-8 -*-
'''
www.yuchaoit.cn with python ,zookeeper
'''
import sys
from kazoo.client import KazooClient, KazooState
import logging

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

# 开始心跳
zk.start()

# 创建节点
# zk.create('/kazoo/yu1',b'good linux ,www.yuchaoit.cn',makepath=True) # 创建节点：makepath 设置为 True ，父节点不存在则创建，其他参数不填均为默认

# znodes=zk.get_children('/kazoo')
# print(znodes)
# data=zk.get('/kazoo/yu1')
# print(data)

# zk.set('/kazoo/yu1',b'not bad linux,www.yuchaoit.cn')
# data,stat=zk.get('/kazoo/yu1')
# print('节点数据提取：',data)


# # 删除节点
# # 不删除父节点
# #  参数 recursive：若为 False，当需要删除的节点存在子节点，会抛异常 NotEmptyError 。若为True，则删除 此节点 以及 删除该节点的所有子节点
# zk.delete('/kazoo/yu1',recursive=False)


# 监控器
def t1(event):
    print('以触发')

zk.get('/kazoo',watch=t1)
print('第一次获取value')
zk.set('/kazoo',b'www.yuchaoit.cn')
zk.get('/kazoo',watch=t1)
print('第二次获取value')

#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()

'''
输出
第一次获取value
以触发
第二次获取value
'''
```

## 遍历所有子节点

```
import sys
from kazoo.client import KazooClient, KazooState
import logging

from importlib import reload

reload(sys)

logging.basicConfig(
    level=logging.DEBUG
    , stream=sys.stdout
    , format='query ok---%(asctime)s %(pathname)s %(funcName)s%(lineno)d %(levelname)s: %(message)s')

# 创建一个客户端，可以指定多台zookeeper，
zk = KazooClient(
    # hosts='10.0.0.18:2181'
     hosts='10.0.0.18:2181,10.0.0.19:2181,10.0.0.20:2181'
    , timeout=10.0  # 连接超时时间
    ,logger=logging  # 传一个日志对象进行，方便 输出debug日志

)

#递归遍历所有节点的子节点函数,_zk是KazooClient的对象，node是节点名称字符串，func是回调函数
def zk_walk(_zk, node, func):
    data, stat = _zk.get(node)
    children = _zk.get_children(node)
    func(node, data, stat, children)
    if len(children) > 0:
        for sub in children:
            sub_node = ''
            if node != '/':
                sub_node = node + '/' + sub
            else:
                sub_node = '/' + sub
            zk_walk(_zk, sub_node, func)

#测试zk_walk的打印回调函数，只是把所有数据都打印出来
def printZNode(node,  data, stat, children):
    print("node  : " + node)
    print("data  : " + str(data))
    print("stat  : " + str(stat))
    print("child : " + str(children))
    print('-'*50)

#开始心跳
zk.start()

#遍历谋个节点的所有子节点
zk_walk(zk, '/', printZNode)

#执行stop后所有的临时节点都将失效
zk.stop()
zk.close()
```

python操作zk基本玩法就到这，更多的是如从事python大数据的开发工程师，需要继续看kazoo模块的官网文档，研究其他用法。
