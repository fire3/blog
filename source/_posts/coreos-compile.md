title: coreos compile
date: 2016-07-01 08:47:16
tags: coreos
---

紧跟云计算潮流，从底层了解一下CoreOS系统，先从编译CoreOS系统做起!

<!-- more -->


# 编译准备

CoreOS继承了ChromeOS的编译系统，十分精致，隐藏了诸多细节。我们先按照 [官方指南](https://coreos.com/os/docs/latest/sdk-modifying-coreos.html)上面来初始化环境。

```
$ mkdir coreos; cd coreos
$ repo init -u https://github.com/coreos/manifest.git
$ repo sync
```

环境初始化好后，开始探寻编译的一些细节。编译开始要执行：

```
$ ./chromite/bin/cros_sdk

```

这个命令里面大有玄机。这个命令会下载一个SDK包，然后展开到chroot目录里面，进行一系列配置，然后在chroot到该目录中。可以打开Log查看到一些蛛丝马迹。

比如修改：./chromite/lib/cros_build_lib.py，在Runcommand里面加上一些日志记录。

``` patch
t a/lib/cros_build_lib.py b/lib/cros_build_lib.py
index 055013b..8e8ee94 100644
--- a/lib/cros_build_lib.py
+++ b/lib/cros_build_lib.py
@@ -384,6 +384,14 @@ def RunCommand(cmd, print_cmd=True, error_ok=False, error_message=None,
     if var not in env and var in os.environ:
       env[var] = os.environ[var]
 
+  if cwd:
+      logger.log(debug_level, 'RunCommand: %s in %s',
+                 ' '.join(map(repr, cmd)), cwd)
+  else:
+      logger.log(debug_level, 'RunCommand: %r', ' '.join(map(repr, cmd)))
+
+
+
   # Print out the command before running.
   if print_cmd or log_output:
     # Note we reformat the argument into a form that can be directly

```

经过追踪后发现，这个脚本是关键: src/scripts/sdk_lib/make_chroot.sh。

然后把这个脚本执行时用 'bash -x' 参数，可以看到更多的细节。


``` patch
diff --git a/sdk_lib/make_chroot.sh b/sdk_lib/make_chroot.sh
index 90db639..1040fcc 100755
--- a/sdk_lib/make_chroot.sh
+++ b/sdk_lib/make_chroot.sh
@@ -1,4 +1,4 @@
-#!/usr/bin/env bash
+#!/bin/bash -x

```

这个脚本执行完后，就会执行src/scripts/sdk_lib/enter_chroot.sh	,进入chroot环境了。
