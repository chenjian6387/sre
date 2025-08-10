# 06-ELK收集主流应用日志

# 1.收集nginx日志

学习背景：access.log，error.log目前日志混杂在一个es索引下。

![1676541430330](/ajian/1676541430330.png)

改进filebeat配置

https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html

加上日志输入文件的判断，采用不同的索引

![1676629181782](/ajian/1676629181782.png)

```yml
  1 filebeat.inputs:
  2 - type: log
  3   enabled: true
  4   paths:
  5     - /var/log/nginx/access.log
  6   json.keys_uner_root: true
  7   json.overwrite_keys: true
  8 
  9 
 10 - type: log
 11   enabled: true
 12   paths:
 13     - /var/log/nginx/error.log
 14 
 15 
 16 output.elasticsearch:
 17   hosts: ["http://localhost:9200"]
 18   indices:
 19     - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM.dd}"
 20       when.contains:
 21         log.file.path: "/var/log/nginx/access.log"
 22     - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM.dd}"
 23       when.contains:
 24         log.file.path: "/var/log/nginx/error.log"
 25 
 26 
 27 setup.ilm.enabled: false
 28 setup.template.enabled: false
 29 
 30 logging.level: info
 31 logging.to_files: true
 32 logging.files:
 33   path: /var/log/filebeat
 34   name: filebeat
 35   keepfiles: 7
 36   permissions: 0644
```

查看es数据

![1676629280380](/ajian/1676629280380.png)

## access.log

![1676629304101](/ajian/1676629304101.png)

## error.log

![1676629334116](/ajian/1676629334116.png)

## 完成nginx日志拆分index

```
curl 127.0.0.1/laoliu
curl 127.0.0.1/laoliu
```

![1676629500226](/ajian/1676629500226.png)

# 2.Kibana图形化定制

## 柱状图

选择可视化

![1676637530113](/ajian/1676637530113.png)

选择图形模式

![1676637566959](/ajian/1676637566959.png)

柱状图、x、y轴图形数据

![1676638183851](/ajian/1676638183851.png)

多个客户端，记得点击save，保存图形

![1676638321876](/ajian/1676638321876.png)

## data table

![1676639039439](/ajian/1676639039439.png)

## 可视化存档

![1676639374548](/ajian/1676639374548.png)

## 饼图

- 客户端类型
- 操作系统类型
- http状态码

![1676639993950](/ajian/1676639993950.png)

## 编辑dashboard

![1676640799248](/ajian/1676640799248.png)

# 3.使用ES预处理节点转换Nginx日志

> 背景
>
> 1.我们可以提提前修改nginx日志为json格式（常见做法，省事）
>
> 2.不修改源数据，使用es提供的grok转换器（太麻烦）
>
> 3.更高级用法是用filebeat模块

https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest.html

```
先恢复你nginx日志为基础格式
systemctl stop filebeat
> /var/log/nginx/access.log
vim /etc/nginx/nginx.conf
systemctl restart nginx
curl 127.0.0.1
cat /var/log/nginx/access.log
```

![1676778810704](/ajian/1676778810704.png)

```
nginx基础日志
127.0.0.1 - - [19/Feb/2023:11:53:59 +0800] "GET / HTTP/1.1" 200 9205 "-" "curl/7.29.0" "-"
127.0.0.1 - - [19/Feb/2023:11:53:59 +0800] "GET / HTTP/1.1" 200 9205 "-" "curl/7.29.0" "-"
```

## Grok

https://www.elastic.co/guide/en/elasticsearch/reference/master/grok-processor.html

GROK是一种采用组合多个预定义的正则表达式，用来匹配分割文本并映射到关键字的工具。通常用来对日志数据进行处理。本文档主要介绍GROK的模式说明以及常用语法。

GROK模式及说明如下表所示。

https://help.aliyun.com/document_detail/129387.html

https://developer.qiniu.com/insight/4759/grok-parser

![1676778949316](/ajian/1676778949316.png)

测试

![1676780294388](/ajian/1676780294388.png)

## grok转换语法

```
127.0.0.1                             ==> %{IP:clientip}
-                                     ==> -
-                                     ==> -
[08/Oct/2020:16:34:40 +0800]         ==> \\[%{HTTPDATE:nginx.access.time}\\]
"GET / HTTP/1.1"                     ==> "%{DATA:nginx.access.info}"
200                                 ==> %{NUMBER:http.response.status_code:long} 
5                                     ==> %{NUMBER:http.response.body.bytes:long}
"-"                                 ==> "(-|%{DATA:http.request.referrer})"
"curl/7.29.0"                         ==> "(-|%{DATA:user_agent.original})"
"-"                                    ==> "(-|%{IP:clientip})"
```

![image-20230219122958715](/ajian/image-20230219122958715.png)

### 字符对应grok模式

```go
%{IP:clientip} - - \[%{HTTPDATE:nginx.access.time}\] \"%{DATA:nginx.access.info}\" %{NUMBER:http.response.status_code:long} %{NUMBER:http.response.body.bytes:long} \"(-|%{DATA:http.request.referrer})\" \"(-|%{DATA:user_agent.original})\"
```

转换结果

![image-20230219123212958](/ajian/image-20230219123212958.png)

## 1.创建pipeline

![image-20230219123744346](/ajian/image-20230219123744346.png)

```go
GET _ingest/pipeline
PUT  _ingest/pipeline/pipeline-nginx-access
{
  "description" : "nginx access log",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{IP:clientip} - - \\[%{HTTPDATE:nginx.access.time}\\] \"%{DATA:nginx.access.info}\" %{NUMBER:http.response.status_code:long} %{NUMBER:http.response.body.bytes:long} \"(-|%{DATA:http.request.referrer})\" \"(-|%{DATA:user_agent.original})\""]
      }
    },{
      "remove": {
        "field": "message"
      }
    }
  ]
}
```

![image-20230219123838882](/ajian/image-20230219123838882.png)

## 2.修改filebeat

![image-20230219124337756](/ajian/image-20230219124337756.png)

```go
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

processors:
  - drop_fields:
      fields: ["ecs","log"]  //删除不需要的字段

output.elasticsearch:
  hosts: ["10.0.0.18:9200"]

  pipelines:
    - pipeline: "pipeline-nginx-access"
      when.contains:
        tags: "access"

  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"

setup.ilm.enabled: false
setup.template.enabled: false

logging.level: info
logging.to_files: true
```

例如删除如下字段

![image-20230219124500790](/ajian/image-20230219124500790.png)

## 3.测试pipeline转换的nginx日志

```
清空，删除所有es 索引，注意测试用法，生产别执行。
[root@es-node1 ~]#curl -X DELETE 'http://localhost:9200/_all'
{"acknowledged":true}
[root@es-node1 ~]#systemctl restart filebeat.service 
[root@es-node1 ~]#

[root@es-node1 ~]#systemctl restart filebeat.service 
[root@es-node1 ~]#

注意再去访问nginx，产生日志，此时filebeat会抓取新日志数据，写入es，生成索引

[root@es-node1 ~]#curl 127.0.0.1
[root@es-node1 ~]#curl 127.0.0.1
[root@es-node1 ~]#curl 127.0.0.1
```

![image-20230219125242245](/ajian/image-20230219125242245.png)

## 4.kibana可视化新格式es数据

![image-20230219131156727](/ajian/image-20230219131156727.png)

## 5.小结

只有专门维护ELK的高级工程师，只维护ELK，需要大量且复杂的维护日志系统，需要手工写grok转换规则。

![image-20230219131755490](/ajian/image-20230219131755490.png)
