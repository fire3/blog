title: "Hack bilibili for fun --- Episode 1"
date: 2016-03-11 16:48:07
tags:
---

B站的视频虽说本人甚少参观，但是本着研究以及造福大众的学术态度，还是光顾并折腾了一番。

先想办法窥测一下B站的API吧。老方法，通过反编译B站的安卓客户端进行，这是最原汁原味的API。

B站的客户端 ：　http://app.bilibili.com/images/android.html

这里选取怀旧版进行ｈａｃｋ，主要原因是上面一个版本apktool居然无法解包。下面一个“怀旧版”也遇到了一些小的技术问题，
按照默认方式解包时无法正常打包，资源文件故障。资源文件故障的解决方法是不解包资源文件。我们用下面命令来解包。

```
        apktool d -r ./BiliPlayer_White.apk  
```

我们用这个小脚本调试：

```
        rm -rf build signed
        mkdir build
        mkdir signed
        adb shell pm uninstall tv.danmaku.bilixl
        apktool build BiliPlayer_White -o build/Biliplayer.apk
        d2j-apk-sign.sh -f build/Biliplayer.apk -o signed/Biliplayer.apk
        adb install signed/Biliplayer.apk
```

我们先想办法找到B站的Http请求URL。

我先从反编译的一堆smali里面逛了逛，另外配合d2j-dex2jar工具转换的jar包和jd-gui工具看了看，评估一下这个工作量。不看不知道，一看吓一跳，
B站的程序员实在是良心大大的好程序员，思路清晰，函数封装合理，最最最重要是，代码没有加入混淆。都知道，安卓代码不加混淆反编译就是明文啊，这
可阅读性提高太多了。你看，人家目录名都放的多么符合程序员的编程规范： smali/tv/danmaku/bili/api ， 这还用说么，这不是api这是啥。先不忙着激动，
让我们来找找HttpGet这个关键字。一搜一大把啊，太符合逻辑了。顺便说一句，HttpGet这个是apache http接口里面的，就算混淆了代码，用了这个也还是能追踪到。
继续筛选我们要的东西，buildHttpGet被发现，太有爱了，要我我也喜欢这个命名。我得写了统一的接口来生成http请求啊，我得封装一组操作来执行Get操作啊，于是乎就
有了buildHttpGet，接下来就简单了，开始加Log：

```
.method public buildHttpGet()Lorg/apache/http/client/methods/HttpGet;
    #.locals 2

    # add a local register by fire3
    .locals 3

    .prologue
    .line 105
    new-instance v0, Lorg/apache/http/client/methods/HttpGet;

    invoke-virtual {p0}, Ltv/danmaku/bili/api2/utility/BLAUriBuilder;->build()Landroid/net/Uri;

    move-result-object v1

    invoke-virtual {v1}, Landroid/net/Uri;->toString()Ljava/lang/String;

    move-result-object v1

#add by fire3
const-string v2, "fire3-bili-Get"
invoke-static {v2, v1}, Landroid/util/Log;->i(Ljava/lang/String;Ljava/lang/String;)I
#add by fire3 end


    invoke-direct {v0, v1}, Lorg/apache/http/client/methods/HttpGet;-><init>(Ljava/lang/String;)V

    .line 106
    .local v0, "httpRequest":Lorg/apache/http/client/methods/HttpGet;
    invoke-direct {p0, v0}, Ltv/danmaku/bili/api2/utility/BLAUriBuilder;->setupRequest(Lorg/apache/http/HttpRequest;)V

    .line 108
    return-object v0
.end method
```

找到合适的地方加个Log太Easy了，看上面代码就知道。然后pack程序，装上执行，点击点击看看，就可以在adb shell里面用logcat搜了：

```
30|root@generic:/ # logcat | grep fire3                                       

I/fire3-bili-Get( 1714): http://api.bilibili.com/bangumi?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&btype=2&platform=android&type=json&sign=1d1ddc1de2d3741a5d20206034852284
I/fire3-bili-Get( 1714): http://api.bilibili.com/index?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&platform=android&type=json&v=2&sign=ebe01fa7223eb329c177792f5ace70da
I/fire3-bili-Get( 1714): http://api.bilibili.com/list?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json&sign=3f5a589a226ffacea56e4ac96b7d3583
I/fire3-bili-Get( 1714): http://api.bilibili.com/list?_device=android&_hwid=ccbb856c97ccb8d2&appkey=c1b107428d337928&order=hot&page=1&pagesize=8&platform=android&tid=145&type=json&sign=3f5a589a226ffacea56e4ac96b7d3583
```

这个是货真价实的一手资料！下一步就开始分析接口了。B站的工作正式启动，未完待续。
