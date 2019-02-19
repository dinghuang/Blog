---
title: CentOS上安装Shadowsocks客户端
date: 2019-01-11 15:47:00
tags:
    - Linux
categories: Linux
---

# CentOS上安装Shadowsocks客户端
## Shadowsocks简介
Shadowsocks，是一种加密的传输方式（一种基于 Socks5 代理方式的网络数据加密传输包）；SS 是目前主流的科学上网方式，是目前最稳定最好用的科学上网工具之一。


## 安装

### 安装pip

pip是Python的包管理工具，我们接下来是使用pip安装的Shadowsocks。

1. 通过yum管理工具安装：
```
yum install -y pip
```
2. 镜像库没有这个包，那么可以手动安装:
```
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py

```

### 安装Shadowsocks客户端

```
pip install --upgrade pip
pip install shadowsocks
```
新建配置文件``vi /etc/shadowsocks.json``：
```
{
"server":"x.x.x.x",
"server_port":25247,
"local_address": "127.0.0.1",
"local_port":25252,
"password":"123456",
"timeout":1000,
"method":"aes-256-cfb",
"workers": 10
}
```
编写启动服务``vi /etc/systemd/system/shadowsocks.service``:
```
[Unit]
Description=shadowsocks

[Service]
TimeoutStartSec=30
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```
启动服务``systemctl start shadowsocks``，运行 ``curl --socks5 127.0.0.1:25252 http://httpbin.org/ip`` ， 返回你ss服务器ip，则说明Shadowsocks客户端启动成功。


## 使用Privoxy把shadowsocks转换为Http代理
### Privoxy简介
Privoxy是一个代理辅助工具，这里用Privoxy把Shadowsocks socks5代理转换为http代理。可以作为kubernetes的docker容器需要访问google的服务，也同时可以作为命令行的代理，本实例用作命令行代理。

### 安装
使用yum安装：
```
yum install privoxy -y
```
修改配置文件``vi /etc/privoxy/config``，加入一行代码
```
forward-socks5 / 127.0.0.1:25252 .
listen-address 127.0.0.1:8118  #这里的ip也可以是k8s的ip
```
启动服务``systemctl start privoxy``，执行命令``curl -x localhost:8118 google.com``，返回数据则表示服务启动成功

### 全局设置
编辑文件``vi /etc/profile``：
```
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
```
使配置生效
```
source /etc/profile
```
测试代码``curl www.google.com``，返回数据则成功设置全局命令行代理












