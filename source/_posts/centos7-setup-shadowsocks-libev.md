title: "centos7 setup shadowsocks-libev"
date: 2016-06-09 04:01:10
tags:
---

安装Shadowssocks服务端
=======================


``` bash
	wget https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
	mv librehat-shadowsocks-epel-7.repo /etc/yum.repos.d/
	yum install shadowsocks-libev
```

接下来修改配置文件： /etc/shadowsocks-libev/config.json

类似如下配置：

``` json
{
    "server":"0.0.0.0",
    "server_port":9000,
    "local_port":1080,
    "password":"password",
    "timeout":60,
    "method":"aes-256-cfb",
}
```

配置防火墙:

``` bash
firewall-cmd --permanent --add-port=9000/tcp
firewall-cmd --permanent --add-port=9000/udp
firewall-cmd --reload
```

启动shadowsocks-libev

``` bash
service shadowsocks-libev start
```

开机启动 

```bash
systemctl enable shadowsocks-libev.service
```

安装kcptun服务端
=======================

```bash
cd /root
wget https://github.com/xtaci/kcptun/releases/download/v20161009/kcptun-linux-amd64-20161009.tar.gz
tar xvf kcptun-linux-amd64-20161009.tar.gz
```


Install supervisord:

```bash
yum -y install python-pip
pip install supervisor
echo_supervisord_conf > /etc/supervisord.conf
```

Add tcptun service to supervisord:
```
cat <<EOF >> /etc/supervisord.conf
[program:tcptun]
command = /root/server_linux_amd64 -l :9002 -t 127.0.0.1:9000 --crypt none --mtu 1200 --nocomp --mode normal --dscp 46
user = root
autostart = true
autoresart = true
stderr_logfile = /var/log/supervisor/tcptun.stderr.log
stdout_logfile = /var/log/supervisor/tcptun.stdout.log
EOF
```

为kcp服务端配置防火墙:
```bash
firewall-cmd --permanent --add-port=9002/tcp
firewall-cmd --permanent --add-port=9002/udp
firewall-cmd --reload
```

启动supervisord:
```
mkdir -p /var/log/supervisor/
supervisord -c /etc/supervisord.conf
```

开机启动supervisord：
```
echo "supervisord -c /etc/supervisord.conf" >> /etc/rc.local
```


优化服务器内核配置
=======================

修改配置文件:
```bash
cat <<EOF >> /etc/sysctl.conf
#TCP OPT
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
#TCP OPT END
EOF
```

设置生效：
```bash
sysctl -p
```
