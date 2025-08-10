# 08-dashboard

# 1.dashboard是什么

https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/

Dashboard 是基于网页的 Kubernetes 用户界面。

Kubernetes Dashboard是一个旨在为Kubernetes世界带来通用监控和操作Web界面的项目，集合了命令行可以操作的所有命令。

使用Kubernetes Dashboard，您可以：

- 向Kubernetes集群部署容器化应用
- 诊断容器化应用的问题
- 管理集群的资源
- 查看集群上所运行的应用程序
- 创建、修改Kubernetes上的资源（例如Deployment、Job、DaemonSet等）
- 展示集群上发生的错误

例如：您可以伸缩一个Deployment、执行滚动更新、重启一个Pod或部署一个新的应用程序。

```
开源社区地址：https://github.com/kubernetes/dashboard
```

# 2.下载配置文件，修改地址

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml

# 修改svc
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort # 修改dashboard的svc，给宿主机映射一个端口

---
```

# 3.应用资源

```
[root@k8s-master-10 /all-k8s-yml/class]#kubectl create -f dashboard.yml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

# 4.查看资源

```
[root@k8s-master-10 /all-k8s-yml/class]#kubectl -n kubernetes-dashboard get po -owide
NAME                                         READY   STATUS    RESTARTS   AGE   IP          NODE          NOMINATED NODE   READINESS GATES
dashboard-metrics-scraper-7b59f7d4df-4zmj9   1/1     Running   0          51s   10.2.2.99   k8s-node-12   <none>           <none>
kubernetes-dashboard-665f4c5ff-x8gpl         1/1     Running   0          51s   10.2.1.79   k8s-node-11   <none>           <none>
[root@k8s-master-10 /all-k8s-yml/class]#
```

# 5.创建管理员账户

这里涉及k8s的认证、鉴权RBAC知识点，以后k8s进阶阶段再去理解。

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
# 多个yaml配置
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

创建管理员账户

```
[root@k8s-master-10 /all-k8s-yml/class]#kubectl create -f dashboard-admin.yml 
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

# 6.查看dashboard资源

```
[root@k8s-master-10 /all-k8s-yml/class]#kubectl create -f dashboard-admin.yml 
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
[root@k8s-master-10 /all-k8s-yml/class]#
[root@k8s-master-10 /all-k8s-yml/class]#kubectl -n kubernetes-dashboard get po -owide
NAME                                         READY   STATUS    RESTARTS   AGE   IP          NODE          NOMINATED NODE   READINESS GATES
dashboard-metrics-scraper-7b59f7d4df-4zmj9   1/1     Running   0          15m   10.2.2.99   k8s-node-12   <none>           <none>
kubernetes-dashboard-665f4c5ff-x8gpl         1/1     Running   0          15m   10.2.1.79   k8s-node-11   <none>           <none>
[root@k8s-master-10 /all-k8s-yml/class]#
[root@k8s-master-10 /all-k8s-yml/class]#kubectl -n kubernetes-dashboard get svc -owide
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE   SELECTOR
dashboard-metrics-scraper   ClusterIP   10.1.157.236   <none>        8000/TCP        15m   k8s-app=dashboard-metrics-scraper
kubernetes-dashboard        NodePort    10.1.138.197   <none>        443:30649/TCP   15m   k8s-app=kubernetes-dashboard
[root@k8s-master-10 /all-k8s-yml/class]#
[root@k8s-master-10 /all-k8s-yml/class]#
[root@k8s-master-10 /all-k8s-yml/class]#kubectl -n kubernetes-dashboard get secrets -owide
NAME                               TYPE                                  DATA   AGE
admin-user-token-lqnsd             kubernetes.io/service-account-token   3      90s
default-token-cwdsg                kubernetes.io/service-account-token   3      15m
kubernetes-dashboard-certs         Opaque                                0      15m
kubernetes-dashboard-csrf          Opaque                                1      15m
kubernetes-dashboard-key-holder    Opaque                                2      15m
kubernetes-dashboard-token-b5ljb   kubernetes.io/service-account-token   3      15m
```

获取token用于登录

```
[root@k8s-master-10 /all-k8s-yml/class]#kubectl -n kubernetes-dashboard describe secrets admin-user-token-lqnsd 
Name:         admin-user-token-lqnsd
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: ff197be0-af9e-4d1f-94a9-3b6d72b500b0

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlVkbVBNMno1UXQ5NXE2YTQxYWljeGRkT0JoZTdZQkFUTXNaS3JIZUgyMzAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWxxbnNkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmZjE5N2JlMC1hZjllLTRkMWYtOTRhOS0zYjZkNzJiNTAwYjAiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.k-jXJShfzfWbZ7nOcosyalMxtNUtbYwlwvtHrgjSucq0BDjuhJCZt52O9MSBSsibmjbbz0MUehyo6KLm2FeRxZbP0W474y0tFzMXxu1OTl29bfrLiK_KtWny-wlQD0IoP6Eh8OysfhBnMKEKZlhE8ls21vxLJQKOesAFB8PzBAflCKuweIqVPe3puVDsCpV1CmDRAKk2VMOtlzjsvm3pBs5N83Vx53TvgKSod6pTB4tO-WZTE3FBPIP0v4t5bsGjDZ7-PSXKVJsHpbFDpO8D0MJ2xQeBvt2s8s_MrBxChOoIQMJtIjZcu85XKHN2CqxWPG6RLto2syTYpNiuiWVInw
```

# 7.登录dashboard

![image-20220917145639183](http://book.bikongge.com/sre/2024-linux/image-20220917145639183.png)

## 页面化管理集群资源

![image-20220917145920423](http://book.bikongge.com/sre/2024-linux/image-20220917145920423.png)