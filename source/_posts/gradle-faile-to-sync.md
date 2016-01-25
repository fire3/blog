title: "gradle faile to sync"
date: 2016-01-25 16:24:19
tags:
---


有一阵子没用一台Linux机器上的开发环境，gradle在同步时报"Broken Pipe".

还是stackoverflow上的答案靠谱：

```
iptables -t nat -F
echo 0 > /proc/sys/net/ipv4/ip_forward
```

