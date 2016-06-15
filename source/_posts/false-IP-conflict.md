title: IP地址冲突，静态IP变169
date: 2016-06-15 09:04:28
tags:
---

办公环境里面的一台虚拟机，有阵子没有重启过了，重启了一次，发现IP失灵了。
明明是个静态IP，非要给我设置成169的微软私有IP，反复禁用网卡，再启用网卡，偶尔还会报IP冲突的错误。
经过一番折腾，最终还是在google找到了 [答案](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1028373)

解决方法如下，打开注册表进入到这个目录下::

        HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters

新建一个DWORD 32位值: ArpRetryCount，值为0.

重启即可。原因还是交换机那边设置的问题。
