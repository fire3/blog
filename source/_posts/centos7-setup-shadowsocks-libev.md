title: "centos7 setup shadowsocks-libev"
date: 2016-06-09 04:01:10
tags:
---


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
