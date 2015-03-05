title: "Letv安卓客户端打开调试选项"
date: 2015-03-05 08:20:25
tags: letv
---

跟踪Letv安卓客户端Http链接最简单的方法，不是搭建什么代理服务器，不是什么抓包分析，而是直接利用Android客户端开发人员留下的调试接口！

<!-- more -->

当然，调试接口不会轻易打开的，控制的开关在这里： com/letv/http/LetvHttpConstant.smali


修改一下这个文件

```
.class public final Lcom/letv/http/LetvHttpConstant;
.super Ljava/lang/Object;
.source "LetvHttpConstant.java"


# static fields
.field public static final CONNECT_TIMEOUT:I = 0x4e20

.field public static final LOG:Ljava/lang/String; = "LetvHttp"

.field public static final READ_TIMEOUT:I = 0x4e20

.field public static isDebug:Z


# direct methods
.method static constructor <clinit>()V
    .locals 1

    .prologue
    .line 23
    const/4 v0, 0x1

    sput-boolean v0, Lcom/letv/http/LetvHttpConstant;->isDebug:Z

    return-void
.end method

.method public constructor <init>()V
    .locals 0

    .prologue
    .line 3
    invoke-direct {p0}, Ljava/lang/Object;-><init>()V

    return-void
.end method

.method public static setDebug(Z)V
    .locals 1
    .param p0, "isDebug"    # Z

    const/4 v0, 0x1
    .prologue
    .line 26
    sput-boolean v0, Lcom/letv/http/LetvHttpConstant;->isDebug:Z

    .line 27
    return-void
.end method

```

我们把setDebug中 sput-boolean强行设置成0x1,另外初始化时也把isDebug设置成1即可。

然后重新打包运行吧：

``` bash
mkdir build
mkdir signed
adb shell pm uninstall com.letv.android.client
apktool build leshishipin_78 -o build/leshishipin_78.apk
d2j-apk-sign.sh -f build/leshishipin_78.apk -o signed/leshishipin_78.apk
adb install signed/leshishipin_78.apk
```

查看TAG是"LetvHttp"的打印就可以看到很多信息喽！
