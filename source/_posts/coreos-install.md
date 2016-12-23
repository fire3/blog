title: coreos install
date: 2016-12-23 15:34:49
tags: [coreos]
---

熟悉CoreOS，先从安装做起。

<!-- more -->

ISO如何获得自不必说，建好虚拟机，挂载ISO，启动后进入系统，按照网页说明进行安装：

https://coreos.com/os/docs/latest/installing-to-disk.html

需要注意的是，必须采用cloud-config的方式进行安装，否则装完了也无法进入系统。

虚拟机里面的图形非常不友好，我们可以先简单的给core用户配置一个密码，然后用scrt，putty之类的工具，登陆进去操作。

还需要参考文档：

https://github.com/coreos/coreos-cloudinit/blob/master/Documentation/cloud-config.md

另外，要了解一些yaml文件的格式，比如不可用tab代替空格。

主要是先配置users相关，比如用户名密码之类的，否则安装到硬盘也是无法进入系统的。
