title: compile aria2 for arm
date: 2016-11-15 08:01:22
tags: [arm, linux, aria2]
---

闲来无事试着给WD Mycloud编编软件试试。

<!-- more -->

准备编译环境
==================

目标处理器是Marvell的ARMADA 375，双核，具有FPU，所以采用带Hard float的工具链。

下载工具栏：

```
mkdir ~/toolchain
cd toolchain
wget https://releases.linaro.org/14.11/components/toolchain/binaries/arm-linux-gnueabi/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabi.tar.xz
wget https://releases.linaro.org/14.11/components/toolchain/binaries/arm-linux-gnueabi/runtime-linaro-gcc4.9-2014.11-arm-linux-gnueabi.tar.xz
wget https://releases.linaro.org/14.11/components/toolchain/binaries/arm-linux-gnueabi/sysroot-linaro-eglibc-gcc4.9-2014.11-arm-linux-gnueabi.tar.xz
tar xf gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabi.tar.xz
tar xf sysroot-linaro-eglibc-gcc4.9-2014.11-arm-linux-gnueabi.tar.xz


echo 'export PATH=$PATH:/home/fire3/toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin' >> ~/.bashrc
. .bashrc

```

设置sysroot目录
```
arm-linux-gnueabihf-gcc -print-sysroot
cd toolchain
ln -s `arm-linux-gnueabihf-gcc -print-sysroot` sysroot
```



Nettle编译
=============
```
wget https://ftp.gnu.org/gnu/nettle/nettle-3.1.tar.gz
tar xf nettle-3.1.tar.gz
cd nettle-3.1
PKG_CONFIG_PATH=/home/fire3/toolchain/sysroot/lib/pkgconfig ./configure --host=arm-linux-gnueabihf --prefix=/home/fire3/toolchain/sysroot  --with-sysroot=/home/fire3/toolchain/sysroot --enable-arm-neon  --enable-mini-gmp
make
make install
```

Gnutls编译
===============
```
wget http://www.ring.gr.jp/pub/net/gnupg/gnutls/v3.4/gnutls-3.4.0.tar.xz
tar xf gnutls-3.4.0.tar.xz
cd gnutls-3.4.0
PKG_CONFIG_PATH=/home/fire3/toolchain/sysroot/lib/pkgconfig ./configure --host=arm-linux-gnueabihf  --with-sysroot=/home/fire3/toolchain/sysroot --with-nettle-mini --with-included-libtasn1 --without-p11-kit --enable-static --prefix=/home/fire3/toolchain/sysroot
make
make install

```


GMP
===================

```
wget https://ftp.gnu.org/gnu/gmp/gmp-6.1.0.tar.bz2
tar xf gmp-6.1.0.tar.bz2 
cd gmp-6.1.0
PKG_CONFIG_PATH=/home/fire3/toolchain/sysroot/lib/pkgconfig ./configure --host=arm-linux-gnueabihf --with-sysroot=/home/fire3/toolchain/sysroot --prefix=/home/fire3/toolchain/sysroot
make install
```

Libz
=======
```
wget http://zlib.net/zlib-1.2.8.tar.gz
tar xf zlib-1.2.8.tar.gz 
cd zlib-1.2.8
CC=arm-linux-gnueabihf-gcc ./configure --prefix=/home/fire3/toolchain/sysroot
make install
```

LibXML2
==================
```
wget ftp://xmlsoft.org/libxml2/libxml2-git-snapshot.tar.gz
tar xf libxml2-git-snapshot.tar.gz 
cd libxml2-2.9.4/
PKG_CONFIG_PATH=/home/fire3/toolchain/sysroot/lib/pkgconfig ./configure --host=arm-linux-gnueabihf --with-sysroot=/home/fire3/toolchain/sysroot --prefix=/home/fire3/toolchain/sysroot  --without-python
make install

```

Aria2编译
=====================

```
PKG_CONFIG_PATH=/home/fire3/toolchain/sysroot/lib/pkgconfig ./configure --host=arm-linux-gnueabihf
make
```

静态版本因为pthread的原因无法编译成功。动态版本可以用如下命令查看其动态库:

```
$ arm-linux-gnueabihf-readelf -a ./aria2c | grep Shared
 0x00000001 (NEEDED)                     Shared library: [libxml2.so.2]
 0x00000001 (NEEDED)                     Shared library: [libdl.so.2]
 0x00000001 (NEEDED)                     Shared library: [libgnutls.so.30]
 0x00000001 (NEEDED)                     Shared library: [libz.so.1]
 0x00000001 (NEEDED)                     Shared library: [libhogweed.so.4]
 0x00000001 (NEEDED)                     Shared library: [libnettle.so.6]
 0x00000001 (NEEDED)                     Shared library: [libgmp.so.10]
 0x00000001 (NEEDED)                     Shared library: [libm.so.6]
 0x00000001 (NEEDED)                     Shared library: [libgcc_s.so.1]
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x00000001 (NEEDED)                     Shared library: [ld-linux-armhf.so.3]

```
