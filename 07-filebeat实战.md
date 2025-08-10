# 07-filebeat实战

# 1.打开filebeat支持nginx模块

```bash
[root@es-node1 /etc/filebeat]#ls
fields.yml  filebeat.reference.yml  filebeat.yml  filebeat.yml.bak  modules.d
[root@es-node1 /etc/filebeat]#ls modules.d/

[root@es-node1 /etc/filebeat]#filebeat version
filebeat version 7.9.1 (amd64), libbeat 7.9.1 [ad823eca4cc74439d1a44351c596c12ab51054f5 built 2020-09-01 19:58:51 +0000 UTC]
[root@es-node1 /etc/filebeat]#


查看开启了哪些模块
[root@es-node1 /etc/filebeat]#filebeat modules list
Error in modules manager: modules management requires 'filebeat.config.modules.path' setting


修改配置文件
[root@es-node1 /etc/filebeat]#tail -5 /etc/filebeat/filebeat.yml
logging.to_files: true

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true 


默认未激活的模块
[root@es-node1 /etc/filebeat]#filebeat modules list|wc -l
60


激活filebeat对nginx支持的模块，本质就是去修改modules.d/下的yml文件，去掉了disable后缀

[root@es-node1 /etc/filebeat]#filebeat modules enable nginx
Enabled nginx


# 打开nginx模块文件，添加nginx日志输入源

cat > /etc/filebeat/modules.d/nginx.yml << 'EOF'
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log"]
  ingress_controller:
    enabled: false
EOF


# 最后修改filebeat.yml，不需要input了
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true 

output.elasticsearch:
  hosts: ["10.0.0.51:9200"]
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/access.log"

    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        log.file.path: "/var/log/nginx/error.log"

setup.ilm.enabled: false
setup.template.enabled: false

logging.level: info
logging.to_files: true


# 清理es、filebeat、nginx环境、测试使用

# 重启filebeat
```

![image-20230219135514917](/ajian/image-20230219135514917.png)

## 查看kibana（pipeline）

![image-20230219135631053](/ajian/image-20230219135631053.png)

## kibana展示es

![image-20230219140042517](/ajian/image-20230219140042517.png)

# 2.filebeat收集tomcat日志

```bash
# jdk
[root@tomcat-10 ~/tomcat-all]#tar -xf jdk-8u221-linux-x64.tar.gz -C /opt
[root@tomcat-10 ~/tomcat-all]#ls /opt
jdk1.8.0_221

配置软连接，修改PATH
[root@tomcat-10 ~/tomcat-all]#ln -s /opt/jdk1.8.0_221/ /opt/jdk8
# sed命令，快速修改 /etc/profile 全局用户环境变量文件

sed -i.ori '$a export JAVA_HOME=/opt/jdk8\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar' /etc/profile

# tomcat
[root@es-node1 /opt]#tar -xf apache-tomcat-8.0.27.tar.gz 
[root@es-node1 /opt]#cd /opt/apache-tomcat-8.0.27/
[root@es-node1 /opt/apache-tomcat-8.0.27]#


# 修改tomcat日志格式
135         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
136                prefix="localhost_access_log" suffix=".txt"
137                pattern="{&quot;clientip&quot;:&quot;%h&quot;,&quot;ClientUser&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&quot;,&quot;AccessTime&quot;:&quot;%t&quot;,&quot;m    ethod&quot;:&quot;%r&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;SendBytes&quot;:&quot;%b&quot;,&quot;Query?string&quot;:&quot;%q&quot;,&quot;partner&quot;:&quot;%{Referer}i&quot;,&qu    ot;AgentVersion&quot;:&quot;%{User-Agent}i&quot;}"/>


[root@es-node1 /opt/jdk8]#/opt/apache-tomcat-8.0.27/bin/shutdown.sh 
Using CATALINA_BASE:   /opt/apache-tomcat-8.0.27
Using CATALINA_HOME:   /opt/apache-tomcat-8.0.27
Using CATALINA_TMPDIR: /opt/apache-tomcat-8.0.27/temp
Using JRE_HOME:        /opt/jdk8
Using CLASSPATH:       /opt/apache-tomcat-8.0.27/bin/bootstrap.jar:/opt/apache-tomcat-8.0.27/bin/tomcat-juli.jar
[root@es-node1 /opt/jdk8]#/opt/apache-tomcat-8.0.27/bin/startup.sh 
Using CATALINA_BASE:   /opt/apache-tomcat-8.0.27
Using CATALINA_HOME:   /opt/apache-tomcat-8.0.27
Using CATALINA_TMPDIR: /opt/apache-tomcat-8.0.27/temp
Using JRE_HOME:        /opt/jdk8
Using CLASSPATH:       /opt/apache-tomcat-8.0.27/bin/bootstrap.jar:/opt/apache-tomcat-8.0.27/bin/tomcat-juli.jar
Tomcat started.
[root@es-node1 /opt/jdk8]#

# 查看tomcat日志。是否是json
[root@es-node1 /opt/jdk8]#cat /opt/apache-tomcat-8.0.27/logs/localhost_access_log.2023-02-19.txt 
{"clientip":"127.0.0.1","ClientUser":"-","authenticated":"-","AccessTime":"[19/Feb/2023:20:55:36 +0800]","method":"GET / HTTP/1.1","status":"200","SendBytes":"11250","Query?string":"","partner":"-","AgentVersion":"curl/7.29.0"}
[root@es-node1 /opt/jdk8]#

#配置filebeat文件
```

Filebeat.yml

```yml
  1 filebeat.inputs:
  2 - type: log
  3   enabled: true
  4   paths:
  5     - /opt/apache-tomcat-8.0.27/logs/localhost_access_log.*.txt
  6   json.keys_under_root: true
  7   json.overwrite_keys: true
  8   tags: ["tomcat"]
  9 
 10 filebeat.config.modules:
 11   path: ${path.config}/modules.d/*.yml
 12   reload.enabled: true
 13 
 14 output.elasticsearch:
 15   hosts: ["10.0.0.18:9200"]
 16   indices:
 17     - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM}"
 18       when.contains:
 19         log.file.path: "/var/log/nginx/access.log"
 20 
 21     - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM}"
 22       when.contains:
 23         log.file.path: "/var/log/nginx/error.log"
 24     - index: "tomcat-access-%{[agent.version]}-%{+yyyy.MM}"
 25       when.contains:
 26         tags: "tomcat"  
 27 
 28 setup.ilm.enabled: false
 29 setup.template.enabled: false
 30 
 31 logging.level: info
 32 logging.to_files: true
```

重启filebeat

```bash
[root@es-node1 /opt/jdk8]#vim /etc/filebeat/filebeat.yml
[root@es-node1 /opt/jdk8]#systemctl restart filebeat.service
```

![image-20230219210009872](/ajian/image-20230219210009872.png)

# 3.小结

ElasticSearch + Filebeat + kibana玩法先到这，基本流程

```
1. filebeat + 目标软件日志（json）
2. 发给es
3. kibana管理es索引，进行数据展示，过滤，可视化管理
```
