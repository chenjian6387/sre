# 08-阿里云ACK部署mysql

# ACK迁移tomcat实践

将你原本虚拟机的tomcat，迁移到ACK全流程。

## 运行tomcat

当你整体的k8s基础环境以ACK运行时，不影响你之前的操作，yaml也都一样使用，确保镜像可用就行。

- 一是要额外的学习成本，支付ACK的运行成本
- 二也是享受ACK带来的高可用，以及降低集群本身的维护成本

## mysql-deploy.yml

> 注意镜像的坑，你的ECS是否支持公网，不行就还是自己先做好容器镜像仓库。

```bash
# 用一个你自己的机器，推送镜像到阿里云
docker pull mysql:5.7

[yuc-tx-2 root ~]#docker tag c20 registry.cn-beijing.aliyuncs.com/pyyu666/mysql:5.7
[yuc-tx-2 root ~]#docker push  registry.cn-beijing.aliyuncs.com/pyyu666/mysql:5.7


docker pull kubeguide/tomcat-app:v1
[yuc-tx-2 root ~]#docker tag a29 registry.cn-beijing.aliyuncs.com/pyyu666/tomcat-app:v1
[yuc-tx-2 root ~]#docker push registry.cn-beijing.aliyuncs.com/pyyu666/tomcat-app:v1
```

### 创建secret

创建mysql控制器，更严谨的事`mysql statefulSet`，练习时可以继续用deployment

![image-20230227145531544](/ajian/image-20230227145531544.png)

### k8s引入私有仓库镜像(yml)

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/pull-image-private-registry/ 文档

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: mysql-dp
  namespace: default
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels: 
      app: mysql-dp
  template:
    metadata:
      name: mysql-dp
      labels:
        app: mysql-dp
    spec:
      imagePullSecrets:
      - name: aliyun-images   # 在这里引入你阿里云创建的secret即可，若是本地化k8s，就手工创建secret即可
      containers:
      - name: mysql-dp
        image: registry-vpc.cn-beijing.aliyuncs.com/pyyu666/mysql:5.7
        imagePullPolicy: IfNotPresent
        ports:
        - name: mysql-port
          containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "www.yuchaoit.cn"
        args:
        - --character-set-server=utf8
        - --collation-server=utf8_bin
```

### 创建mysql-deploy成功

> 这里稍有难度的就是，k8s使用自建私有仓库的镜像下载。

![image-20230227145659011](/ajian/image-20230227145659011.png)

## Mysql-svc.yml

代理mysql控制器的pod

```yml
apiVersion: v1
kind: Service
metadata: 
  name: mysql-svc
  namespace: default
spec:
  selector:
    app: mysql-dp
  ports:
  - name: mysql-port
    port: 3306
    protocol: TCP
    targetPort: 3306
```

![image-20230227145847871](/ajian/image-20230227145847871.png)

> 去ECS上访问SVC，登录mysql-pod
>
> 走master快捷登录，mysql-pod

```bash
[yuc-tx-2 root ~]#kubectl exec -it mysql-dp-55fbd69987-fc27g bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@mysql-dp-55fbd69987-fc27g:/# mysql -uroot -pwww.yuchaoit.cn -h 172.16.215.235
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## Tomcat-deploy.yml

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: tomcat-dp
  namespace: default
  labels:
    app: tomcat
spec:
  replicas: 1
  selector:
    matchLabels: 
      app: tomcat-dp
  template:
    metadata:
      name: tomcat-dp
      labels:
        app: tomcat-dp
    spec:
      imagePullSecrets:
      - name: aliyun-images   # 在这里引入你阿里云创建的secret即可，若是本地化k8s，就手工创建secret即可
      containers:
      - name: tomcat-dp
        image: registry-vpc.cn-beijing.aliyuncs.com/pyyu666/tomcat-app:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: tomcat-port
          containerPort: 8080 
        env:
        - name: MYSQL_SERVICE_HOST 
          value: mysql-svc
```

![image-20230227151427919](/ajian/image-20230227151427919.png)

## tomcat-svc.yml

```yml
apiVersion: v1
kind: Service
metadata: 
  name: tomcat-svc
  namespace: default
spec:
  selector:
    app: tomcat-dp
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
    nodePort: 30002
  type: LoadBalancer
```

![image-20230227151703988](/ajian/image-20230227151703988.png)

## 测试tomcat

![image-20230227153341838](/ajian/image-20230227153341838.png)

## ACK小结

至此，咱们就先暂时的，以学习环境所需的

- 购买，创建阿里云k8s环境，理解背后创建的ack对应的ecs、slb、eip等
- 在ACK云上进行应用部署，nginx部署，包括私有镜像仓库的设置
- ACK云上进行本地tomcat部署练习，创建deployment,svc资源
- 更多ACK玩法，可知只需要继续学习k8s自身功能，以及在公司资金支撑下，可以更方便的玩转ACK更多功能
