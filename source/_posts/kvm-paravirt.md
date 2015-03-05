title: "kvm内核调试小记"
date: 2015-03-05 15:03:26
tags: [kvm,kernel]
---

换了一台Ubuntu 12.04的机器，很久以前一份实验用内核在新的机器kvm环境下无法启动了。内核没有任何打印输出，也没有panic，就是无信息，有些无所适从。
用bochs试了试，可以进入内核运行啊，很是奇怪。突然灵光一闪，会不会打开我这份内核的paravirt相关配置就好了呢？

<!-- more -->

果然，修改配置如下:

```
CONFIG_PARAVIRT_GUEST=y
# CONFIG_PARAVIRT_TIME_ACCOUNTING is not set
# CONFIG_XEN is not set
# CONFIG_XEN_PRIVILEGED_GUEST is not set
CONFIG_KVM_GUEST=y
CONFIG_PARAVIRT=y
CONFIG_PARAVIRT_CLOCK=y
# CONFIG_PARAVIRT_DEBUG is not set

```

运行kvm测试：

``` bash
sudo kvm -serial stdio  -kernel ../linux-stable/arch/x86/boot/bzImage -append "console=ttyS0,9600"
```

打印出现，终于可以恢复实验环境了。

附上一份kvm启动脚本：

``` bash
sudo kvm -serial stdio  -hda busybox.img -boot c -smp 4 -m 3000 -net nic,model=e1000 -net tap,script=./scripts/qemu-ifup.sh  -localtime  -display none
```

qemu-ifup.sh如下:

``` bash
#!/bin/sh
set -x
switch=virbr1
if [ -n "$1" ];then
    /usr/bin/sudo tunctl -u `whoami` -t $1
    /usr/bin/sudo ip link set $1 up
    /usr/bin/sudo brctl addif $switch $1
    exit 0
else
    echo "Error: no interface specified"
    exit 1
fi
```
