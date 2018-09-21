---
title: VPS搭建SS
date: 2018-09-08 22:49:00
tags:
    - 其他
categories: 其他
---

#   购买VPS服务器

声明：本教程仅供学习用

[购买地址][1]

推荐vultr的原因是，vultr支持支付宝、服务器较多，价格虽然不算便宜，但是网速稳定。
各个机房速度测试，直接ping 域名
地理位置 官方测试服务器ip 下载测试文件
```
Frankfurt, DE fra-de-ping.vultr.com 100M 1000M
Amsterdam, NL ams-nl-ping.vultr.com 100M 1000M
Paris, France par-fr-ping.vultr.com 100M 1000M
London, UK lon-gb-ping.vultr.com 100M 1000M
Singapore sgp-ping.vultr.com 100M 1000M
New York (NJ) nj-us-ping.vultr.com 100M 1000M
Tokyo, Japan hnd-jp-ping.vultr.com 100M 1000M
Chicago, Illinois il-us-ping.vultr.com 100M 1000M
Atlanta, Georgia ga-us-ping.vultr.com 100M 1000M
Miami, Florida fl-us-ping.vultr.com 100M 1000M
Seattle, Washington wa-us-ping.vultr.com 100M 1000M
Dallas, Texas tx-us-ping.vultr.com 100M 1000M
Silicon Valley, California sjo-ca-us-ping.vultr.com 100M 1000M
Los Angeles, California lax-ca-us-ping.vultr.com 100M 1000M
Sydney, Australia syd-au-ping.vultr.com 100M 1000M
   ```

经过测试，大陆地区，Tokyo的速度是最快的，延迟在50ms左右。

#   搭建ShadowSocksR服务
ssh连接购买的服务器
``ssh -p22 root@xx.xx.xx.xx``
搭建ssr服务
``wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh``
``chmod +x shadowsocksR.sh``
``./shadowsocksR.sh 2>&1 | tee shadowsocksR.log``
根据提示输入相关配置
相关指令：
卸载: ``./shadowsocksR.sh uninstall``
启动：``/etc/init.d/shadowsocks start``
停止：``/etc/init.d/shadowsocks stop``
重启：``/etc/init.d/shadowsocks restart``
状态：``/etc/init.d/shadowsocks status``
配置文件路径：``/etc/shadowsocks.json``
日志文件路径：``/var/log/shadowsocks.log``
代码安装目录：``/usr/local/shadowsocks``
如果要配置多个用户，多个端口，打开配置文件路径，修改如下：
```
{
    "server":"0.0.0.0",
    "server_ipv6":"[::]",
    "server_port":9001,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
        "9001":"123456",
        "9002":"123456",
        "9003":"123456"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "protocol":"origin",
    "obfs":"plain",
    "fast_open":false
}
```
设置相关端口后要在防火墙打开相关端口的通讯，Centos默认使用firewall命令，如下所示：
``sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent``
``sudo firewall-cmd --reload``
``firewall-cmd --list-all``

#   优化网络

1.系统层面
--------------------------------------------------

``vi /etc/sysctl.conf``

```
# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = htcp
# forward ipv4
net.ipv4.ip_forward = 1
```
保存生效
``sysctl -p``
其中最后的hybla是为高延迟网络（如美国，欧洲）准备的算法，需要内核支持，测试内核是否支持，在终端输入：
``sysctl net.ipv4.tcp_available_congestion_control``
如果结果中有hybla，则证明你的内核已开启hybla，如果没有hybla，可以用命令modprobe tcp_hybla开启。

对于低延迟的网络（如日本，香港等），可以使用htcp，可以非常显著的提高速度，首先使用modprobe tcp_htcp开启，再将net.ipv4.tcp_congestion_control = hybla改为net.ipv4.tcp_congestion_control = htcp，建议EC2日本用户使用这个算法。

2.TCP优化
--------------------------------------------------

1.修改文件句柄数限制
如果是``ubuntu/centos``均可修改``/etc/sysctl.conf``
找到``fs.file-max``这一行，修改其值为``1024000``，并保存退出。然后执行``sysctl -p``使其生效
修改``vi /etc/security/limits.conf``文件，加入

```
*               soft    nofile           512000
*               hard    nofile          1024000
```
针对centos,还需要修改``vi /etc/pam.d/common-session``文件，加入
``session required pam_limits.so``

2.修改``vi /etc/profile``文件，加入
``ulimit -SHn 1024000``
然后重启服务器执行``ulimit -n``，查询返回``1024000``即可。

``sysctl.conf``报错解决方法
修复``modprobe``的：
```
rm -f /sbin/modprobe 
ln -s /bin/true /sbin/modprobe
```
修复``sysctl``的：
```
rm -f /sbin/sysctl 
ln -s /bin/true /sbin/sysctl
```

3.软件辅助优化
--------------------------------------------------

软件辅助优化都得参考系统内核，查看是否适用，如果不适用，可以修改系统内核。

2.1 锐速

锐速是TCP底层加速软件,官方已停止推出永久免费版本,但网上有破解版可以继续使用。需要购买的话先到锐速官网注册帐号,并确认[内核版本][2]是否支持锐速的版本。

一键安装速锐破解版

``wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh``
一键卸载

``chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f``
设置

```
Enter your accelerated interface(s) [eth0]: eth0
Enter your outbound bandwidth [1000000 kbps]: 1000000
Enter your inbound bandwidth [1000000 kbps]: 1000000
Configure shortRtt-bypass [0 ms]: 0
Auto load ServerSpeeder on linux start-up? [n]:y
```
**是否开机自启**
``Run ServerSpeeder now? [y]:y #是否现在启动``
执行lsmod，看到有appex0模块即说明锐速已正常安装并启动。

至此，安装就结束了，但还有后续配置。
修改``vi /serverspeeder/etc/config``文件的几个参数以使锐速更好的工作

```
accppp="1" #加速PPTP、L2TP V-P-N；设为1表示开启，设为0表示关闭
advinacc="1" #高级入向加速开关；设为 1 表示开启，设为 0 表示关闭；开启此功能可以得到更好的流入方向流量加速效果；
maxmode="1" #最大传输模式；设为 1 表示开启；设为 0 表示关闭；开启后会进一步提高加速效果，但是可能会降低有效数据率。
rsc="1" #网卡接收端合并开关；设为 1 表示开启，设为 0 表示关闭；在有些较新的网卡驱动中，带有 RSC 算法的，需要打开该功能。
l2wQLimit="512 4096" #从 LAN 到 WAN 加速引擎在缓冲池充满和空闲时分别能够缓存的数据包队列的长度的上限；该值设置的高会获得更好的加速效果，但是会消耗更多的内存
w2lQLimit="512 4096" #从 WAN 到 LAN 加速引擎在缓冲池充满和空闲时分别能够缓存的数据包队列的长度的上限；该值设置的高会获得更好的加速效果，但是会消耗更多的内存
```
重读配置以使配置生效``/serverspeeder/bin/serverSpeeder.sh reload``

查看锐速当前状态``/serverspeeder/bin/serverSpeeder.sh stats``

查看所有命令``/serverspeeder/bin/serverSpeeder.sh help``

停止``/serverspeeder/bin/serverSpeeder.sh stop``

启动``/serverspeeder/bin/serverSpeeder.sh start``

重启锐速``/serverspeeder/bin/serverSpeeder.sh restart``

2.1 安装Google BBR

要求内核版本``4.13.5-1.el7.elrepo.x86_64`` 以上
```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y
```
检查内核是否更新
``rpm -qa | grep kernel``
启动
``grub2-set-default 1``
重启
``shutdown -r now``
查看是否生效
``uname -r``
安装Google BBR
```
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sysctl -p
```
检查是否安装成功
``sysctl net.ipv4.tcp_available_congestion_control``
执行命令后，看是否是提示
``“net.ipv4.tcp_available_congestion_control = bbr cubic reno”``
执行命令，是否提示bbr
``lsmod | grep bbr``


  [1]: https://my.vultr.com/
  [2]: http://dl.serverspeeder.com/ls.do?m=availables