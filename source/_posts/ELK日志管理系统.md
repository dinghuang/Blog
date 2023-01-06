---
title: ELK日志管理系统
date: 2020-07-06 18:33:00
tags:
    - JAVA
categories: JAVA
---

# 简介
官方文档:https://www.elastic.co/guide/en/elastic-stack/current/overview.html

# 搭建
本教程基于Elasticsearch版本7.7.0


要在docker中搭建，要起3个套件

其中elasticearch会遇到docker内存问题，可以看这个解决
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

或者
macOS with Docker for Mac

The vm.max_map_count setting must be set within the xhyve virtual machine:

From the command line, run:

```
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

Press enter and use`sysctl` to configure vm.max_map_count:
```
sysctl -w vm.max_map_count=262144
```
To exit the screen session, type Ctrl a d.

```
docker network create elasticsearch
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.7.0
docker run -d -p 9200:9200 -p 9300:9300 --network elasticsearch -e "discovery.type=single-node" --name elasticsearch docker.elastic.co/elasticsearch/elasticsearch:7.7.0
docker pull docker.elastic.co/kibana/kibana:7.7.0
docker run -d --link elasticsearch:elasticsearch --name kibana -p 5601:5601 --network elasticsearch docker.elastic.co/kibana/kibana:7.7.0
docker pull docker.elastic.co/logstash/logstash:7.7.0
docker run -d -p 5044:5044  --network elasticsearch --link elasticsearch:elasticsearch --name logstash docker.elastic.co/logstash/logstash:7.7.0
```


遇到错误
```
Security must be explicitly enabled when using a [basic] license. Enable security by setting [xpack.security.enabled] to [true] in the elasticsearch.yml file and restart the node.
```
编辑elasticsearch.yml，加入``xpack.security.enabled:true``,然后重启节点
重启时遇到错误:
```
ERROR: [1] bootstrap checks failed
 [1]: Transport SSL must be enabled if security is enabled on a [basic] license. Please set [xpack.security.transport.ssl.enabled] to [true] or disable security by setting [xpack.security.enabled] to [false]
 ```
编辑elasticsearch.yml，加入``xpack.security.transport.ssl.enabled:true``,然后重启节点

执行设置用户名和密码的命令,这里需要为4个用户分别设置密码，``elastic, kibana, logstash_system,beats_system``

```
bin/elasticsearch-setup-passwords interactive
```

# 配置文件修改
进入logstash容器里面修改配置文件
```
docker exec -it 54b504186a47 /bin/bash # 这里 54b504186a47是容器id
vi /usr/share/logstash/config/logstash.yml
```

logstash.yml配置文件如下
```
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
path.config: /usr/share/logstash/config/*.conf
path.logs: /var/log/logstash
```
修改logstash-sample.conf
```
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    codec => json_lines
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "springboot-logstash-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
}
```
这样logstash的读取就是通过一个tcp服务读取

# springboot结合

引入包
```
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.4</version>
</dependency>
```
添加配置文件``logback-spring.xml``
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />

    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>127.0.0.1:5044</destination>
        <!-- 日志输出编码 -->
        <encoder charset="UTF-8"
                 class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "logLevel": "%level",
                        "serviceName": "${springAppName:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
        <appender-ref ref="CONSOLE" />
    </root>

</configuration>
```
启动应用，配置kibana的索引，如图所示
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/WechatIMG469.png)

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/WechatIMG470.png)

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/WechatIMG471.png)

如果添加了索引，页面没有显示索引，还要继续添加的话，这个是Kibana的问题，重启一下Kibana容器就好了。
