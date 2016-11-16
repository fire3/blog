title: compile dpdk
date: 2016-11-03 11:08:47
tags:
---

编译dpdk
===============


在x86_64 linux ubuntu 16.04环境下：

```
make install T=x86_64-native-linuxapp-gcc
```

如果需要查看详细输出，可以运行
```
make install T=x86_64-native-linuxapp-gcc V=1
```


