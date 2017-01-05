title: compile rkt with kvm support
date: 2016-12-24 18:57:27
tags: [coreos,rkt,kvm]
---


尝试学习rkt，从尝试编译做起。

<!-- more -->

环境准备工作
==================

安装一些开发工具和开发库。编译环境ubuntu 16.04 x86_64 server。
```
sudo apt install git
git clone https://github.com/coreos/rkt.git
sudo apt install automake libacl1-dev libncurses5-dev libtspi-dev libsystemd-dev golang gcc bc libglib2.0-dev libpixman-1-dev libcap-dev libattr1-dev
```


编译
==============

```
cd rkt
./configure --with-stage1-flavors=kvm --with-stage1-kvm-hypervisors=lkvm,qemu
```

配置显示如下：
```
        rkt 1.21.0+git4e6437a

        stage1 setup

        type:                                   'flavor'
        default stage1 location:                ''
        default stage1 images directory:        '/usr/local/lib/rkt/stage1-images'

        built stage1 flavors:                   'kvm'
        default stage1 flavor:                  'kvm'
        implied default stage1 name:            'coreos.com/rkt/stage1-kvm'
        implied default stage1 version:         '1.21.0+git4e6437a'

        coreos/kvm flavor specific build parameters

        local CoreOS PXE image path:            ''
        local CoreOS PXE image systemd version: ''
        stage1 CoreOS board:                    'amd64-usr'

        kvm flavor specific build parameters

        hypervisors:                            'lkvm,qemu'

        other build parameters

        functional tests enabled:               'no'
        features:                               '+TPM +SDJOURNAL'
        ACI arch:                               'amd64'
        go version:                             '1.6.2'
        go arch:                                'amd64'
```

编译开始
```
make
```

如果遇到问题，可以查看具体的编译命令
```
make V=3
```
