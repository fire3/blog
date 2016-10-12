title: "centos7 setup shadowsocks-libev"
date: 2016-06-09 04:01:10
tags:
---

* 安装Shadowssocks服务端
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
sudo systemctl enable shadowsocks-libev.service
```

* 安装kcptun服务端

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
supervisord -c /etc/supervisord.conf
```

开机启动supervisord：
```
echo "supervisord -c /etc/supervisord.conf" >> /etc/rc.local
```

