title: merlin custom ddns with aliyun
date: 2016-06-30 04:47:51
tags:
---

首先要感谢opkg这个工具，让我们可以方便的在路由器上使用python和pip。

``` bash
$ opkg install python-pip
$ pip install aliyun-python-sdk-alidns

```

安装好后先写一个脚本找到自己域名的RecordID。
``` python
#!/opt/bin/python

from aliyunsdkcore import client
from aliyunsdkalidns.request.v20150109 import DescribeDomainRecordsRequest
from aliyunsdkalidns.request.v20150109 import UpdateDomainRecordRequest
import sys

if __name__ =='__main__':
        clt = client.AcsClient('Key........','Secret..................',
                                'cn-hangzhou')
        rquest=DescribeDomainRecordsRequest.DescribeDomainRecordsRequest()
        request.set_accept_format('json')
        request.set_DomainName('fire3.xyz')
        result = clt.do_action(request)
        print result
```


然后写一个小脚本，放在 /jffs/scripts/home.py:

``` python

#!/opt/bin/python

from aliyunsdkcore import client
from aliyunsdkalidns.request.v20150109 import DescribeDomainRecordsRequest
from aliyunsdkalidns.request.v20150109 import UpdateDomainRecordRequest
import sys

if __name__ =='__main__':
        clt = client.AcsClient('Key........','Secret..................',
                                'cn-hangzhou')
        request=UpdateDomainRecordRequest.UpdateDomainRecordRequest()
        request.set_RecordId(83035279)
        request.set_Value(sys.argv[1])
        request.set_RR('home')
        request.set_Type('A')
        request.set_TTL('600')
        result = clt.do_action(request)

```


再写一个小脚本/jffs/scripts/ddns-start调用上面的python:

```
#!/bin/sh

IP=`ip addr show ppp0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1`

if [ X"$IP" == X"" ]
then
/sbin/ddns_custom_updated 0
exit -1
fi

/jffs/scripts/home.py $IP

/sbin/ddns_custom_updated 1
```

大功告成。注意 /jffs/scripts/ddns-start是默认的ddns设置脚本，名字不可以变动。[见这里](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts)
