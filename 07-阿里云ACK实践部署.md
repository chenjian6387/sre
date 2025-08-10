# 07-阿里云ACK实践部署

# ACK创建无状态nginx应用

> 在default下创建nginx
>
> 基于yaml创建

![image-20230226144228132]/(ajian/image-20230226144228132.png)

## Deployment模板

![image-20230226144156446]/(ajian/image-20230226144156446.png)

点击创建

控制器创建中

![image-20230226144310216]/(ajian/image-20230226144310216.png)

## 故障诊断

![image-20230226144605334]/(ajian/image-20230226144605334.png)

## 查看事件

![image-20230226144628929]/(ajian/image-20230226144628929.png)

## 发起阿里云工单

遇见有不会的问题，可以在阿里云提工单，有工程师解决。

## 错误解决

> ECS无法走公网，所以无法下载docker.io的镜像

![image-20230226145708015]/(ajian/image-20230226145708015.png)

查看ECS

![image-20230226145728309]/(ajian/image-20230226145728309.png)

> 核心错误，在于，购买ACK时，没设置SNAT网关，没有给ECS配置EIP。

![image-20230226150121271]/(ajian/image-20230226150121271.png)

## 解决过程

> 问题1：ACK创建nginx应用报错，镜像拉取错误。
>
> 解决：ACK创建的2个ECS节点无公网IP，要么添加公网IP，要么走内网获取docker镜像。

## 容器镜像服务实践

> https://cr.console.aliyun.com/cn-beijing/instances
>
> 理解为阿里云给你提供的本地docker、harbor

![image-20230226150424871]/(ajian/image-20230226150424871.png)

### 设置个人仓库密码

```
Yuchao123
```

![image-20230226150555949]/(ajian/image-20230226150555949.png)

### 创建命名空间

![image-20230226150848084]/(ajian/image-20230226150848084.png)

### 创建镜像仓库（坑）

> 这就是是你阿里云上的，镜像名字，和你需要的镜像，一个名字即可，它不是个仓库。

![image-20230226151929834]/(ajian/image-20230226151929834.png)

### 推送你本地镜像到阿里云仓库

如下是正确玩法

```bash
# 输入你刚才设置的，个人仓库密码
[yuc-tx-2 root ~]#docker login --username=tb411045_2013 registry.cn-beijing.aliyuncs.com
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[yuc-tx-2 root ~]#


# 本地拉取镜像
[yuc-tx-2 root ~]#docker pull nginx:latest
latest: Pulling from library/nginx
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Image is up to date for nginx:latest
docker.io/library/nginx:latest
[yuc-tx-2 root ~]#

#修改tag，上传到你的阿里云镜像仓库
[yuc-tx-2 root ~]#docker tag 605c77e624dd registry.cn-beijing.aliyuncs.com/devops01-k8s/nginx:latest

[yuc-tx-2 root ~]#docker push  registry.cn-beijing.aliyuncs.com/devops01-k8s/nginx:latest
```

### 创建secret

你的阿里云镜像仓库，是有认证密码的，因此你想使用，得加上`配置管理，保密字典`

![image-20230226153750236]/(ajian/image-20230226153750236.png)

### 大坑，是否有公网

> ECS是否有公网，是否能访问到阿里云镜像仓库的公网？

![image-20230226154539566]/(ajian/image-20230226154539566.png)

### 再次创建你的deployment-nginx

![image-20230226153020079]/(ajian/image-20230226153020079.png)

选择私有镜像、登录私有镜像的认证密码。

![image-20230226153917997]/(ajian/image-20230226153917997.png)

### 创建成功

![image-20230226153957472]/(ajian/image-20230226153957472.png)

### 进入pod内

![image-20230226154133449]/(ajian/image-20230226154133449.png)

### 本地kubectl管理nginx-pod

![image-20230226154249162]/(ajian/image-20230226154249162.png)

## 小结

- 阿里云ACK本质是你购买了ECS、SLB、EIP的组合，并且给你安装部署好k8s集群。
- ECS能否上公网，决定了你容器镜像下载的方式（内网/公网）
- 可以给ACK集群，打开SNAT功能
- 阿里云不懂的操作，可以提交工单，提供截图，文字描述，有阿里云工程师回答。
- 于超老师友情提醒，阿里云ACK很烧钱，做实验，尽量越短时间做完，否则几百块大洋就飞了。

# ACK创建网络服务

## 自建的K8S集群如何访问pod

![image-20230227110948963]/(ajian/image-20230227110948963.png)

## 阿里云ACK创建SVC

![image-20230227132940874]/(ajian/image-20230227132940874.png)

### service文档

> https://help.aliyun.com/document_detail/181518.html

### 创建Service

![image-20230227115628329]/(ajian/image-20230227115628329.png)

> 修改yaml，修改网络类型

![image-20230227131324620]/(ajian/image-20230227131324620.png)

> yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service1 #TODO: to specify your service name
  labels:
    app: nginx-svc
spec:
  selector:
    app: nginx-deploy #TODO: change label selector to match your backend pod
  ports:
  - protocol: TCP
    name: http
    port: 80 #TODO: choose an unique port on each node to avoid port conflict
    targetPort: 80
  type: LoadBalancer
#  type: LoadBalancer
```

> 注意svc匹配的pod名字

![image-20230227131203455]/(ajian/image-20230227131203455.png)

### 创建SVC结果

由于是loadbalaner类型，阿里云会创建一个SLB作为访问入口，可以访问到svc代理的pod

![image-20230227131416844]/(ajian/image-20230227131416844.png)

### 查看SVC创建的SLB

![image-20230227131707132]/(ajian/image-20230227131707132.png)

> slb代理的ECS地址

![image-20230227131739823]/(ajian/image-20230227131739823.png)

> 通信架构

![image-20230227131902227]/(ajian/image-20230227131902227.png)

### kubectl查看pod日志

> 阿里云的SLB，是轮训模式

```
[yuc-tx-2 root ~]#kubectl logs -f nginx-deploy-fb5f6f54b-
nginx-deploy-fb5f6f54b-85kn5  nginx-deploy-fb5f6f54b-sg4tj
```

![image-20230227132526841]/(ajian/image-20230227132526841.png)

### 访问架构图

![image-20230227132630693]/(ajian/image-20230227132630693.png)

## 阿里云ACK创建ingress

基于域名的负载均衡，访问你的pod应用

![image-20230227133128942]/(ajian/image-20230227133128942.png)

### ingress转发svc

![image-20230227133435976]/(ajian/image-20230227133435976.png)

### 你可以修改svc的类型

![image-20230227134002491]/(ajian/image-20230227134002491.png)

> 更新后

![image-20230227134028123]/(ajian/image-20230227134028123.png)

> 负载均衡的IP也自动被删除了

## 域名访问结果

> 但是可能会被工信部拦截，域名没备案的话

![image-20230227133711749]/(ajian/image-20230227133711749.png)

## 小结

- 走svc的loadbalancer直接访问
- 走ingress的域名访问（得做好域名解析）
