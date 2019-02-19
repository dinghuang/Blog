---
title: 基于Frp内网穿透反向代理的端口转发实现本地服务器
date: 2019-01-07 15:47:00
tags:
    - Linux
categories: Linux
---
      
# 基于frp内网穿透反向代理的端口转发实现本地服务器
## frp简介
[frp](https://github.com/fatedier/frp) 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。
- 利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。
- 对于 http, https 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个80端口。
- 利用处于内网或防火墙后的机器，对外网环境提供 tcp 和 udp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机。

## 使用

### 准备条件

公网服务器一台，内网服务器一台，公网服务器绑定域名1个。
### 开始搭建

#### 公网服务器

ssh连接到公网服务器上，新建目录

```
mkdir -p /usr/local/frp
```

根据对应的操作系统及架构，从 [Release](https://github.com/fatedier/frp/releases) 页面下载最新版本的程序。

```
wget https://github.com/fatedier/frp/releases/download/v0.22.0/frp_0.22.0_linux_arm64.tar.gz
```

解压

```
tar -zxvf  frp_0.22.0_linux_arm64.tar.gz
```

首先删掉``frpc``、``frpc.ini``两个文件，然后再进行配置
修改``frps.ini ``文件，这里使用了最简化的配置：

```
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 6081
max_pool_count = 20
allow_ports = 2000-3000,6081,4000-50000 #端口白名单
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
token = 123456 #客户端也要配置一样的token
authentication_timeout = 90000 #超时时间，如果客户端遇到服务启动认证失败，大概率是时区问题，服务器设置一下就好了
```

保存然后启动服务
```
./frps -c ./frps.ini
```
这是前台启动，后台启动命令为

```
nohup ./frps -c ./frps.ini &
```
可以通过访问``http://xx.xx.xx.xx:7500/static/#/proxies/tcp``访问``frp``服务的监控界面，账号密码与上面配置的一致。

#### 内网服务器

根据对应的操作系统及架构，从 [Release](https://github.com/fatedier/frp/releases) 页面下载最新版本的程序。

```
wget https://github.com/fatedier/frp/releases/download/v0.22.0/frp_0.22.0_linux_arm64.tar.gz
```

解压

```
tar -zxvf  frp_0.22.0_linux_arm64.tar.gz
```

首先删掉``frpc``、``frpc.ini``两个文件，然后再进行配置
修改 ``frpc.ini`` 文件。

```
# frpc.ini
[common]
server_addr = xx.xx.xx.xx #公网ip地址
server_port = 7000
token = 123456
 
#公网通过ssh访问内部服务器
[ssh]
type = tcp              #连接协议
local_ip = 127.0.0.1
local_port = 22         #ssh默认端口号
remote_port = 6000      #自定义的访问内部ssh端口号
 
#公网访问内部web服务器以http方式
[web]
type = http         #访问协议
local_port = 8081   #内网web服务的端口号
custom_domains = strongcat.top #所绑定的公网服务器域名，一级、二级域名都可以
```

保存然后执行启动

```
./frpc -c ./frpc.ini
```

这是前台启动，后台启动命令为

```
nohup ./frpc -c ./frpc.ini &
```

#### 认证超时解决办法
一般认证超时的原因是由于2个服务器之间时间不同，可以通过命令tzselect修改时区，按照步骤设置时区
```
$ tzselect
```

同步服务器时间
```
sudo yum install ntp
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp yes
```

查看时间确保同步``timedatectl``

## 开机自启动

我用的是centOS7的操作系统，为了防止因为网络或者重启问题Frp失效，所以写了一个开机启动服务，公网服务器配置：

```
[Unit]
Description=frp

[Service]
TimeoutStartSec=30
Type=simple
ExecStart=/root/frp/frp_0.22.0_linux_386/frps -c /root/frp/frp_0.22.0_linux_386/frps.ini
ExecStop=/bin/kill $MAINPID
Restart=on-failure
RestartSec=60s

[Install]
WantedBy=multi-user.target
```

内网服务器配置：
```
[Unit]
Description=frp
After=network.target
Wants=network.target

[Service]
TimeoutStartSec=30
Type=simple
ExecStart=/root/frp/frp_0.22.0_linux_386/frpc -c /root/frp/frp_0.22.0_linux_386/frpc.ini
ExecStop=/bin/kill $MAINPID
Restart=on-failure
RestartSec=60s

[Install]
WantedBy=multi-user.target
```

文件保存到``/etc/systemd/system/frp.service``中，并执行``systemctl daemon-reload``，``systemctl start frp``,开机启动``systemctl enable frp``

外网ssh访问内网服务器（直接使用配置里面数据演示）

```
ssh -oPort=6000 root@x.x.x.x
```
 
 将 ``www.strongcat.top`` 的域名 A 记录解析到 IP ``x.x.x.x``，如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名。
 
 通过浏览器访问 ``http://www.yourdomain.com:8080`` 即可访问到处于内网机器上的 ``web`` 服务。
 
 有些系统默认自带防火墙，需要开通端口
 
```
firewall-cmd --zone=public --add-port=6000/tcp --permanent  
systemctl stop firewalld.service  
systemctl start firewalld.service 
```

如果遇到``authorization timeout``错误的话，需要进行2个服务器之间的时间同步。2边服务器都执行下面的命令：
```
#下载ntpdate
yum install -y ntpdate
#调整时区为上海，也就是北京时间+8区
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
yes | cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#使用NTP来同步时间
ntpdate us.pool.ntp.org
#定时同步时间（每隔10分钟同步时钟）
crontab -l >/tmp/crontab.bak
echo "*/10 * * * * /usr/sbin/ntpdate us.pool.ntp.org | logger -t NTP" >> /tmp/crontab.bak
crontab /tmp/crontab.bak
```

如果是像我这种笔记本的话，可以设置系统关闭盖子的动作
```
vim /etc/systemd/logind.conf
```

```
HandlePowerKey         按下电源键后的行为，默认power off
HandleSleepKey          按下挂起键后的行为，默认suspend
HandleHibernateKey   按下休眠键后的行为，默认hibernate
HandleLidSwitch          合上笔记本盖后的行为，默认suspend

ignore 忽略，跳过
power off 关机
eboot 重启
halt 挂起
suspend shell内建指令，可暂停目前正在执行的shell。若要恢复，则必须使用SIGCONT信息。所有的进程都会暂停，但不是消失（halt是进程关闭）
hibernate 让笔记本进入休眠状态
hybrid-sleep 混合睡眠，主要是为台式机设计的，是睡眠和休眠的结合体，当你选择Hybird时，系统会像休眠一样把内存里的数据从头到尾复制到硬盘里 ，然后进入睡眠状态，即内存和CPU还是活动的，其他设置不活动，这样你想用电脑时就可以快速恢复到之前的状态了，笔记本一般不用这个功能。
lock 仅锁屏，计算机继续工作。
```

更多指令可以参考[这篇博客](http://www.jinbuguo.com/systemd/logind.conf.html)

最后重新加载服务使配置生效
```
systemctl restart systemd-logind
```

## 进阶（配合nginx实现域名转发）

### 购买域名
购买域名，国内需要备案，然后再阿里云中添加域名，创建域名解析，如图所示
![image](https://ws1.sinaimg.cn/large/9bc4cb9fgy1g0and9vxv5j225607ut9t.jpg)

### 安装nginx
```
docker pull nginx
```
新建文件``nginx.conf``
```

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

  # frp的接收http请求的反向代理
    server {
        listen 80;
        server_name *.strongsickcat.com strongsickcat.com;

        location / {
            # 7071端口即为frp监听的http端口
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host:80;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_connect_timeout 7d;
            proxy_send_timeout 7d;
            proxy_read_timeout 7d;

            }
        # 防止爬虫抓取
        if ($http_user_agent ~* "360Spider|JikeSpider|Spider|spider|bot|Bot|2345Explorer|curl|wget|webZIP|qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot|NSPlayer|bingbot")
            {
                return 403;
            }
    }
}
```
docker启动nginx
```
docker run -p 80:80 --name mynginx -v $PWD/www:/www -v $PWD/nginx.conf:/etc/nginx/nginx.conf -v $PWD/logs:/wwwlogs  -d nginx
```
附上frp客户端与服务端配置
```
#frps.ini
[common]
bind_port = 7000
max_pool_count = 20
allow_ports = 4000-50000
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = 742041978
token = 2524668868
authentication_timeout = 900
vhost_http_port = 8080
subdomain_host = strongsickcat.com
```
```
#客户端 frpc.ini
[common]
server_addr = 106.15.226.184
server_port = 7000
token = 2524668868
admin_addr = 127.0.0.1
admin_port = 7400

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[test_static_file]
type = tcp
remote_port = 16001
plugin = static_file
plugin_local_path = /root/file
plugin_strip_prefix = static
plugin_http_user = admin
plugin_http_passwd = 742041978

[kibana]
type = http
# local_port代表你想要暴露给外网的本地web服务端口
local_port = 5601
# subdomain 在全局范围内要确保唯一，每个代理服务的subdomain不能重名，否则会影响正常使用。
# 客户端的subdomain需和服务端的subdomain_host配合使用
subdomain = kibana

[elasticsearch]
type = http
local_port = 9200
subdomain = elasticsearch

[mysql]
type = tcp
local_port = 3306
remote_port = 13306

[prometheus]
type = http
local_port = 9090
subdomain = prometheus

[prometheus-linux]
type = tcp
local_port = 9100
remote_port = 9100

[prometheus-mysql]
type = tcp
local_port = 9104
remote_port = 9104

[grafana]
type = http
local_port = 3000
subdomain = grafana

[prometheusa]
type = tcp
local_port = 9090
remote_port = 9090
```
按照我的配置文件，可以直接通过子域名访问kibana服务
http://kibana.strongsickcat.com:8080
