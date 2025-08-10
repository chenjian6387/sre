# 04-阿里云NAS

欢迎使用NAS文件系统

阿里云文件存储NAS是一个可共享访问，弹性扩展，高可靠，高性能的分布式文件系统。兼容POSIX 文件接口，可支持数千台计算节点共享访问，可以挂载到弹性计算ECS、神龙裸金属、容器服务ACK、弹性容器ECI、批量计算BCS、高性能计算EHPC，AI训练PAI等计算业务上提供高性能的共享存储，用户无需修改应用程序，即可无缝迁移业务系统上云。

[立即开通](https://common-buy.aliyun.com/?commodityCode=naspost)[产品文档](https://help.aliyun.com/document_detail/27518.html)技术支持直达

https://nasnext.console.aliyun.com/introduction

![image-20230225105146001](/ajian/image-20230225105146001.png)

# 如何选项购买

![image-20230225105238792](/ajian/image-20230225105238792.png)

## 购买通用型NAS

https://nasnext.console.aliyun.com/overview

https://nasnext.console.aliyun.com/introduction

![image-20230225105925462](/ajian/image-20230225105925462.png)

## 性能型/容量型

![image-20230225110119109](/ajian/image-20230225110119109.png)

## NAS详细

![image-20230225110153222](/ajian/image-20230225110153222.png)

## ECS挂载NAS

![image-20230225110207499](/ajian/image-20230225110207499.png)

```bash
sudo mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 178774ade0-vbc65.cn-beijing.nas.aliyuncs.com:/ /mnt

推荐使用以上命令通过 NFSv3 协议挂载，获得最佳性能。如果您的应用依赖文件锁，即需要使用多台 ECS 同时编辑一个文件，请使用 NFSv4 协议挂载。
```

实践

```bash
[root@devops-web02 ~]# yum install nfs-utils -y


[root@devops-web02 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        868M     0  868M   0% /dev
tmpfs           879M     0  879M   0% /dev/shm
tmpfs           879M  484K  878M   1% /run
tmpfs           879M     0  879M   0% /sys/fs/cgroup
/dev/vda1        20G  2.8G   16G  16% /
tmpfs           176M     0  176M   0% /run/user/0
[root@devops-web02 ~]# sudo mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 178774ade0-vbc65.cn-beijing.nas.aliyuncs.com:/ /mnt
[root@devops-web02 ~]#
[root@devops-web02 ~]#
[root@devops-web02 ~]# df -h
Filesystem                                      Size  Used Avail Use% Mounted on
devtmpfs                                        868M     0  868M   0% /dev
tmpfs                                           879M     0  879M   0% /dev/shm
tmpfs                                           879M  488K  878M   1% /run
tmpfs                                           879M     0  879M   0% /sys/fs/cgroup
/dev/vda1                                        20G  2.8G   16G  16% /
tmpfs                                           176M     0  176M   0% /run/user/0
178774ade0-vbc65.cn-beijing.nas.aliyuncs.com:/  1.0P     0  1.0P   0% /mnt
[root@devops-web02 ~]#
```

## 上云NFS架构

![image-20230225110732589](/ajian/image-20230225110732589.png)

## 多台ECS共享NAS数据

Web01、web02共享

```
sudo mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 178774ade0-vbc65.cn-beijing.nas.aliyuncs.com:/ /test-nas
```

![image-20230225111100836](/ajian/image-20230225111100836.png)

## NAS运维

![image-20230225111136627](/ajian/image-20230225111136627.png)

## NAS备份服务

![image-20230225111326994](/ajian/image-20230225111326994.png)

## NAS备份、恢复实践

### 创建备份

![image-20230225111518200](/ajian/image-20230225111518200.png)

### 立即执行备份任务

![image-20230225111558390](/ajian/image-20230225111558390.png)

### 模拟删除、恢复

删除

```
[root@devops-web02 ~]# cd /mnt
[root@devops-web02 mnt]# ls
于超老师带你学linux.txt  哎哟不错哦.txt
[root@devops-web02 mnt]# rm -f *
[root@devops-web02 mnt]# ls
[root@devops-web02 mnt]#
```

恢复

![image-20230225111718816](/ajian/image-20230225111718816.png)

### 确认恢复

![image-20230225111831991](/ajian/image-20230225111831991.png)
