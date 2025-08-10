# 03-mongodb基于角色的访问控制

```
https://www.mongodb.com/docs/v4.4/tutorial/enable-authentication/

https://www.mongodb.com/docs/manual/reference/configuration-options/#security-options
```

# 1.什么是认证

```
认证是验证客户端身份的过程。当启用访问控制（即授权）时，MongoDB要求所有客户端进 
行身份验证，以确定其访问。

认证方法：为了验证用户，MongoDB提供了该 db.auth()方法。 
对于mongoshell和MongoDB工具，可以通过从命令行传入用户身份验证信息来验证用户，认证机制
```

## mongodb不设密码的危险

![image-20221017112159858](/ajian/image-20221017112159858.png)

## 认证授权命令

```
db.auth() 将用户验证到数据库。
db.changeUserPassword() 更改现有用户的密码。
db.createUser() 创建一个新用户。
db.dropUser() 删除单个用户。
db.dropAllUsers() 删除与数据库关联的所有用户。
db.getUser() 返回有关指定用户的信息。
db.getUsers() 返回有关与数据库关联的所有用户的信息。
db.grantRolesToUser() 授予用户角色及其特权。
db.removeUser() 已过时。从数据库中删除用户。
db.revokeRolesFromUser() 从用户中删除角色。
db.updateUser() 更新用户数据。
db.createRole() 创建角色并指定其特权。
db.dropRole() 删除用户定义的角色。
db.dropAllRoles() 删除与数据库关联的所有用户定义的角色。
db.getRole() 返回指定角色的信息。
db.getRoles() 返回数据库中所有用户定义角色的信息。
db.grantPrivilegesToRole() 将权限分配给用户定义的角色。
db.revokePrivilegesFromRole() 从用户定义的角色中删除指定的权限
db.grantRolesToRole() 指定用户定义的角色从中继承权限的角色。
db.revokeRolesFromRole() 从角色中删除继承的角色。
db.updateRole() 更新用户定义的角色
```

## 创建超级用户、角色

```
1.账号、密码、认证数据库  

2.三要素，用户是在哪个库下的认证账号。
```

![image-20221017112924373](/ajian/image-20221017112924373.png)

```
注意顺序、先创建用户、再修改配置文件开启认证功能，反了就错了。

1.操作流程:
  1.先创建管理员用户
  2.以管理员用户身份创建普通用户及权限
  3.修改配置开启认证
  4.使用普通用户登录验证

https://www.mongodb.com/docs/v4.4/tutorial/enable-authentication/#enable-access-control


2.在未开启认证的情况下创建管理员
https://www.mongodb.com/docs/v4.4/tutorial/enable-authentication/#create-the-user-administrator


use admin

db.createUser(
  {
    user: "yuchao",
    pwd: "www.yuchaoit.cn",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)

# 文本密码，也可以加密填写   pwd: "www.yuchaoit.cn",


# userAdminAnyDatabase 所有数据库都有admin权限
# 指定库 admin
# 权限 readWriteAnyDatabase
```

## 查询admin库下的用户

```
> db.getUsers()
[
    {
        "_id" : "admin.yuchao",
        "userId" : UUID("b267ecea-8359-47ff-81a9-235a522a21d4"),
        "user" : "yuchao",
        "db" : "admin",
        "roles" : [
            {
                "role" : "userAdminAnyDatabase",
                "db" : "admin"
            },
            {
                "role" : "readWriteAnyDatabase",
                "db" : "admin"
            }
        ],
        "mechanisms" : [
            "SCRAM-SHA-1",
            "SCRAM-SHA-256"
        ]
    }
]
>
```

## 开启mongod服务的认证功能

```
[root chaoge-linux ~]#vim /opt/mongo_27017/conf/mongodb.conf

vim /opt/mongo_27017/conf/mongodb.conf
security:
    authorization: enabled
```

重启服务

```
[root chaoge-linux /data/mongo_27017]#systemctl restart mongod
[root chaoge-linux /data/mongo_27017]#ps -ef|grep mongo
mongo    125381      1  5 12:24 ?        00:00:00 /opt/mongodb/bin/mongod -f /opt/mongo_27017/conf/mongodb.conf
root     125423 124346  0 12:24 pts/0    00:00:00 grep --color=auto mongo
[root chaoge-linux /data/mongo_27017]#
```

## 账号密码登录

```
[root chaoge-linux /data/mongo_27017]#mongo -u yuchao -p www.yuchaoit.cn
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("3baec149-f04b-456f-bb96-c392789c7c26") }
MongoDB server version: 4.2.22
> 
> db.user_info.insertOne({"name":"chaoge"})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("634cda8767c5177eda761fdb")
}
> 
> db.user_info.find()
{ "_id" : ObjectId("634cda8767c5177eda761fdb"), "name" : "chaoge" }
>
```

### 查询库用户信息

```
> use admin;
switched to db admin


> db.getUser("yuchao")
{
    "_id" : "admin.yuchao",
    "userId" : UUID("0eb3ba85-0217-4229-8158-1cae17932430"),
    "user" : "yuchao",
    "db" : "admin",
    "roles" : [
        {
            "role" : "userAdminAnyDatabase",
            "db" : "admin"
        },
        {
            "role" : "readWriteAnyDatabase",
            "db" : "admin"
        }
    ],
    "mechanisms" : [
        "SCRAM-SHA-1",
        "SCRAM-SHA-256"
    ]
}
> 

> db.getUsers()
[
    {
        "_id" : "admin.yuchao",
        "userId" : UUID("0eb3ba85-0217-4229-8158-1cae17932430"),
        "user" : "yuchao",
        "db" : "admin",
        "roles" : [
            {
                "role" : "userAdminAnyDatabase",
                "db" : "admin"
            },
            {
                "role" : "readWriteAnyDatabase",
                "db" : "admin"
            }
        ],
        "mechanisms" : [
            "SCRAM-SHA-1",
            "SCRAM-SHA-256"
        ]
    }
]
> 


# userAdminAnyDatabase 内置的管理员权限
# 解释文档

https://www.mongodb.com/docs/v4.4/reference/built-in-roles/#mongodb-authrole-userAdminAnyDatabase
```

### 查看内置所有角色

```
https://www.mongodb.com/docs/v4.4/reference/built-in-roles/
```

## 未认证的登录，无法操作

```
[root chaoge-linux /data/mongo_27017]#mongo
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("909f1db0-b677-407e-8b37-ad6fb55f2789") }
MongoDB server version: 4.2.22
> db.user_info.insertOne({"name":"chaoge"})
2022-10-17T12:28:19.179+0800 E  QUERY    [js] uncaught exception: WriteCommandError({
    "ok" : 0,
    "errmsg" : "command insert requires authentication",
    "code" : 13,
    "codeName" : "Unauthorized"
}) :
WriteCommandError({
    "ok" : 0,
    "errmsg" : "command insert requires authentication",
    "code" : 13,
    "codeName" : "Unauthorized"
})
WriteCommandError@src/mongo/shell/bulk_api.js:417:48
executeBatch@src/mongo/shell/bulk_api.js:915:23
Bulk/this.execute@src/mongo/shell/bulk_api.js:1163:21
DBCollection.prototype.insertOne@src/mongo/shell/crud_api.js:264:9
@(shell):1:1
>
```

## 创建普通库用户

```
要在MongoDB部署中创建用户，请连接到部署，然后使用db.createUser()方法或createUser命令添加用户。

MongoDB是一个nosql数据库服务器。

默认安装提供使用mongo命令通过命令行访问数据库而不进行身份验证。下面我们来学习如何在具有适当身份验证的Mongodb服务器中创建用户。



use crm;
> db.createUser(
  {
    user: "chaoge",
   pwd:  "www.yuchaoit.cn",
    roles: ["readWrite"]
  }
 )
Successfully added user: { "user" : "chaoge", "roles" : [ "readWrite" ] }
> 
> 


> db.getUsers()
[
    {
        "_id" : "crm.chaoge",
        "userId" : UUID("c1e62274-4144-4c0d-b055-d7389bf2f674"),
        "user" : "chaoge",
        "db" : "crm",
        "roles" : [
            {
                "role" : "readWrite",
                "db" : "crm"
            }
        ],
        "mechanisms" : [
            "SCRAM-SHA-1",
            "SCRAM-SHA-256"
        ]
    }
]



# 验证数据库用户
> db.auth("chaoge","www.yuchaoit.cn")
1

验证失败
> db.auth("chaoge","www.yuchaoit.cnc")
Error: Authentication failed.
0



# 删除用户
> db.dropUser("chaoge")
true
```

### 普通用户读写数据

```
# 重新用普通用户登录，管理数据库
[root chaoge-linux /data/mongo_27017]#mongo -u chaoge -p www.yuchaoit.cn --authenticationDatabase crm
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?authSource=crm&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("20605a27-3798-4146-8bc3-e31bebb18cf9") }
MongoDB server version: 4.2.22
> 


# 可以删除库下的集合，库也就没了。

# 测试写入数据
[root chaoge-linux /data/mongo_27017]#mongo -u chaoge -p www.yuchaoit.cn --authenticationDatabase crm
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?authSource=crm&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("30d82bf0-5f5c-45e9-b780-e28160cee0a0") }
MongoDB server version: 4.2.22
> 
> show dbs
> 
> use crm
switched to db crm
> 


> show collections
ops
> db.ops.find()
{ "_id" : ObjectId("634cf4396446be98e247fb4e"), "name" : "chaoge" }
{ "_id" : ObjectId("634cf43e6446be98e247fb4f"), "name" : "sanpang" }
>
```

## 创建只读普通用户

```
[root chaoge-linux /data/mongo_27017]#mongo -u yuchao -p www.yuchaoit.cn
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("544fb8fa-96a1-47f4-9d16-1b79d2ca1b1a") }
MongoDB server version: 4.2.22
> use blog;
> db.createUser(
 {"user":"ergou","pwd":"123456",roles:[{role:"read",db:"blog"}]}
)
Successfully added user: {
    "user" : "ergou",
    "roles" : [
        {
            "role" : "read",
            "db" : "blog"
        }
    ]
}
> 
> 

> db.blog.insertOne({"title":"超哥带你学linux www.yuchaoit.cn"})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("634cf5a00173482abfaf36ed")
}
> exit
bye




# 普通用户测试


[root chaoge-linux /data/mongo_27017]#mongo -u ergou -p 123456 --authenticationDatabase blog
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?authSource=blog&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c9c60742-e12c-438b-8dfd-61c669a11d63") }
MongoDB server version: 4.2.22
> db
test
> show dbs
blog  0.000GB
> use blog
switched to db blog
> show collections
blog
> db.blog.find()
{ "_id" : ObjectId("634cfc5a3850780b4798e2e5"), "title" : "超哥带你学linux www.yuchaoit.cn" }
{ "_id" : ObjectId("634cfc693850780b4798e2e6"), "title" : "超哥带你学linux www.yuchaoit.cn" }


无法写入数据，权限不足。
```

普通用户只能看到自己有权限的数据库

```
[root chaoge-linux /data/mongo_27017]#mongo 127.0.0.1:27017  -u ergou -p 123456 --authenticationDatabase blog
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/test?authSource=blog&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7a737e3f-ac15-4163-b898-b1becbc0182f") }
MongoDB server version: 4.2.22
> show dbs
blog  0.000GB
>
```
