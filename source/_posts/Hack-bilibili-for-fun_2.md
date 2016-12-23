title: "Hack bilibili for fun --- Episode 2"
date: 2016-03-11 16:48:07
tags: [android]
---

链接URL一目了然后，需要弄清除URL的含义细节，许多字段都可以顾名思义，比如如下例子：

```
http://api.bilibili.com/list?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=82&type=json&sign=9ba57bd5befcf86d9ec5917af0b7e368
```

可以看到 _hwid  类似机器代码，可能时IMEI识别码之类的。appkey这是这个应用统一的一个key，所有的这个安卓应用都是用这个key。order为排序方法，page和pagesize是页面编号和大小。platform所有链接均为android。tid不同，为B站的频道识别码。type均为json，最最关键的时sign这个字段，这个字段是这个URL的一个签名字段，api服务器需要检查这个字段才认可这个请求。我们需要搞清除sign的生成算法。在这里，不得不再感谢未混淆的代码以及一些开源软件事先的工作，比如 you-get。我们先打开源程序中程序员的调试插桩，看看原来程序员的调试打印：

修改这个文件： tv/danmaku/android/util/DebugLog.smali , 把这个里面的constructor里面的 const/4 v0, 0x0 全部改为 const/4 v0, 0x1 。就可以看到原来程序员的打印了。

综合BlaUriBuilder这个文件，看getSignedQuery实现，不难发现是用MD5算了一串字符串生成了sign。那么算的究竟时什么东西呢？原来程序员调试时也加了打印：

```
I/BLAUriBuilder(26691): signed  query _device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json
I/BLAUriBuilder(26691): escaped query _device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json&sign=3f5a589a226ffacea56e4ac96b7d3583
I/BLAUriBuilder(26691): escaped url http://api.bilibili.com/list?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json&sign=3f5a589a226ffacea56e4ac96b7d3583

```

可以看到，第一行中内容就是MD5计算的输入，但这个输入并不完整，仔细看还缺少了一个LibBili.getAppSecret获取的AppSecret，同样通过代码插桩，我们很方便的获取到这个Appsecret为ea85624dfcf12d7cc7b2b3a94fac1f2c，接下来就可以验证算法了，使用python：

```
>>> hashlib.md5(bytes('_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json')).hexdigest()
'81eae43caf2fc31516cd2640be9fa747'
>>> hashlib.md5(bytes('_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=jsonea85624dfcf12d7cc7b2b3a94fac1f2c')).hexdigest()
'3f5a589a226ffacea56e4ac96b7d3583'
```

可以看到跟上面打印中的sign计算出来一致。至此，sign的生成方法一目了然。 普通的获取获取一个频道的视频均是这个URL，另外就是搜索的URL生成：

```
I/BLAUriBuilder(26691): signed  query _device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&keyword=%E4%B9%8C%E9%BE%9F&order=default&page=1&pagesize=20&platform=android&type=json
I/BLAUriBuilder(26691): escaped query _device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&keyword=%E4%B9%8C%E9%BE%9F&order=default&page=1&pagesize=20&platform=android&type=json&sign=9d0886eb3d0731158c59e731b565aa25
I/BLAUriBuilder(26691): escaped url http://api.bilibili.com/search?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&keyword=%E4%B9%8C%E9%BE%9F&order=default&page=1&pagesize=20&platform=android&type=json&sign=9d0886eb3d0731158c59e731b565aa25

```

