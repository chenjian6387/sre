# 06-阿里云ACK

https://help.aliyun.com/product/85222.html

![image-20230225183806366](http://book.bikongge.com/sre/2024-linux/image-20230225183806366.png)

# 为什么花钱买ACK？

## 自建Kubernetes的劣势

- 搭建集群繁琐。

  您需要手动配置Kubernetes相关的各种组件、配置文件、证书、密钥、相关插件和工具，整个集群搭建工作需要花费专业人员数天到数周的时间。

- 在公共云上，需要投入大量的成本实现和云产品的集成。

  与阿里云上其他产品的集成，需要您自己投入成本来实现，如日志服务、监控服务和存储管理等。

- 容器是一个系统性工程，涉及网络、存储、操作系统、编排等各种技术，需要专门的人员投入。

- 容器技术一直在不断发展，版本迭代快，需要不断地试错、升级、测试。

## ACK优势

https://help.aliyun.com/document_detail/86739.html

## ACK用在哪些生产环境

https://help.aliyun.com/document_detail/86740.html

### DevOps与ACK

https://help.aliyun.com/document_detail/151985.htm?spm=a2c4g.11186623.0.0.2c2548d1FvVIPQ#task-2404827

- 微服务应用的CI/CD（持续集成和持续交付）。
- 线上应用蓝绿部署升级。
- 自动化测试与部署。

DevOps的目的是构建一种文化和环境，使构建、测试、发布软件更加快捷、频繁和可靠 。

而到了容器时代，需要部署的机器不但量更大，变化更剧烈，有的甚至需要根据条件自动升缩，为了满足企业敏捷的需求，持续部署也成了必需。

本方案使用云效完成容器应用（小程序后端服务）的自动化构建和持续部署。

![image-20230225184245061](http://book.bikongge.com/sre/2024-linux/image-20230225184245061.png)

## ACK Pro版集群计费说明

https://help.aliyun.com/document_detail/462280.html

# 创建ACK集群

## 多种ACK套餐

![image-20230225184750059](http://book.bikongge.com/sre/2024-linux/image-20230225184750059.png)

> 托管版

```
只需创建 Worker 节点，Master 节点由容器服务创建并托管。
具备简单、低成本、高可用、无需运维管理 Kubernetes 集群 Master 节点的特点，您可以更多关注业务本身。

只需要创建node工作节点，去跑你的业务就得了，不需要操心master，也无法操作。
小公司需要部署k8s测试，买这个就行了，不用担心master的宕机，阿里做好了高可用。
```

> 专有版

```
需要创建 3 个 Master（高可用）节点及若干 Worker 节点，可对集群基础设施进行更细粒度的控制，需要自行规划、维护、升级服务器集群。

业务较为复杂，需要细粒度的控制k8s所有环境，有更多调整。
等于购买了阿里的机器部署自主可维护的k8s高可用集群。
```

> ASK集群

```
无需创建和管理 Master 节点及 Worker 节点，即可通过控制台或者命令配置容器实例的资源、指定应用容器镜像以及对外服务的方式，直接启动应用程序。

最简单，直接创建pod运行你的应用即可，不需要维护master，node的k8s组件环境
```

ACK边缘托管版

```
提供一个支持边缘计算的托管 Kubernetes 集群，您可以将边缘节点接入到边缘集群中进行托管，为您节省运维成本，并提供类似边缘自治、网络自治等适配边缘计算场景能力
```

注册集群

```
注册集群功能可以为分布在各处的外部 Kubernetes 集群提供统一的使用和管理方式
```

## 图解

![image-20230225190554868](http://book.bikongge.com/sre/2024-linux/image-20230225190554868.png)

## 购买ACK托管版(master)

![image-20230225190811623](http://book.bikongge.com/sre/2024-linux/image-20230225190811623.png)

> pro版和标准版
>
> 区别在于可以对master有更多的设置，不需要你创建，但是可以额外的设置，主要在于安全性监控。
>
> 标准版用于学习测试
>
> Pro版本用于公司商业运行

![image-20230225190955388](http://book.bikongge.com/sre/2024-linux/image-20230225190955388.png)

### 创建ACK标准模板

![image-20230225213122846](http://book.bikongge.com/sre/2024-linux/image-20230225213122846.png)

### ACK标准版配置

### k8s-1.22.15版本

### docker容器运行时

![image-20230225213914759](http://book.bikongge.com/sre/2024-linux/image-20230225213914759.png)

### 暴露ACK到公网（EIP）

> 暴露api-server地址到公网，即可远程管理ACK集群。

![image-20230226113107805](http://book.bikongge.com/sre/2024-linux/image-20230226113107805.png)

### ACK连接RDS

- 高可用数据库，不是你自己pod起mysql
- 使用RDS-mysql作为数据库

### ACK安全组

![image-20230226123602418](http://book.bikongge.com/sre/2024-linux/image-20230226123602418.png)

只要你的ACK，ECS，RDS使用阿里云创建的VPC，默认都可以互通。

### 高级选项

![image-20230226123727251](http://book.bikongge.com/sre/2024-linux/image-20230226123727251.png)

## Worker节点配置购买

### 费用计算

k8s集群有最低运行要求，并且费用计算还是（ECS+ACK+EIP+SLB）

### 配置要求

```
Yuchao123 密码
```

![image-20230226124615089](http://book.bikongge.com/sre/2024-linux/image-20230226124615089.png)

## K8S其他组件配置

### ingress-nginx

![image-20230226125225658](http://book.bikongge.com/sre/2024-linux/image-20230226125225658.png)

## 购买订单检查

![image-20230226125350394](http://book.bikongge.com/sre/2024-linux/image-20230226125350394.png)

### 修改后

![image-20230226130052213](http://book.bikongge.com/sre/2024-linux/image-20230226130052213.png)

## ACK创建中

![image-20230226130156315](http://book.bikongge.com/sre/2024-linux/image-20230226130156315.png)

## ACK集群运行中

![image-20230226131144384](http://book.bikongge.com/sre/2024-linux/image-20230226131144384.png)

### 类似dashboard

![image-20230226135726395](http://book.bikongge.com/sre/2024-linux/image-20230226135726395.png)

### 通过kubectl操作ACK(本地>阿里云)

![image-20230226135800448](http://book.bikongge.com/sre/2024-linux/image-20230226135800448.png)

> 可以通过你本地机器，走api-server提供的公网地址，进行远程管理
>
> 只要有证书就可以通过api-server的认证

```bash
# 配置源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装kubectl
yum install     kubectl-1.19.3 ipvsadm -y

# 查看文件
[yuc-tx-2 root /etc/yum.repos.d]#cat ~/.kube/config |head -4
apiVersion: v1
clusters:
- cluster:
    server: https://123.56.15.123:6443


# 查看k8s worker节点
[yuc-tx-2 root /etc/yum.repos.d]#kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
cn-beijing.192.168.0.31   Ready    <none>   60m   v1.22.15-aliyun.1
cn-beijing.192.168.0.32   Ready    <none>   60m   v1.22.15-aliyun.1
```

![image-20230226141020199](http://book.bikongge.com/sre/2024-linux/image-20230226141020199.png)

> 非常好用

### 如何接管k8s集群

- 连接到api-server
- 有证书权限

## k8s命令补全

```bash
yum install bash-completion -y
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

![image-20230226141944864](http://book.bikongge.com/sre/2024-linux/image-20230226141944864.png)

## 更安全的ECS内网连接ACK

同一个VPC下的ECS，都可以连接ACK

![image-20230226142211603](http://book.bikongge.com/sre/2024-linux/image-20230226142211603.png)

# ACK节点管理

![image-20230226142248268](http://book.bikongge.com/sre/2024-linux/image-20230226142248268.png)

## 节点详细

![image-20230226142410996](http://book.bikongge.com/sre/2024-linux/image-20230226142410996.png)

## 工作负载（namespace）

![image-20230226142449114](http://book.bikongge.com/sre/2024-linux/image-20230226142449114.png)

## Service管理

![image-20230226142610272](http://book.bikongge.com/sre/2024-linux/image-20230226142610272.png)

## ACK提供可视化k8s管理

![image-20230226142644964](http://book.bikongge.com/sre/2024-linux/image-20230226142644964.png)

# ACK监控

> 你需要用k8s、维护、监控k8s，所有东西，不需要自己去搭建了，并且个人搭建导致的毫无规章制度，运维难得问题。
>
> 统一ACK解决了所有问题，只需要招聘懂kubernetes的运维，即可维护ACK上的公司技术业务。

![image-20230226142849179](http://book.bikongge.com/sre/2024-linux/image-20230226142849179.png)