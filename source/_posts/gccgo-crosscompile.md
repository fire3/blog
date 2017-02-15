title: gccgo crosscompile
date: 2017-02-15 14:47:23
tags:
---

这篇文档写的有一些出入：

https://github.com/golang/go/wiki/GccgoCrossCompilation

最好的方法是利用gcc源代码编译gccgo时生成的go工具，比如在ubuntu上：

```
sudo apt install gccgo
```

会安装go-6这个程序，这个程序默认的是使用gccgo作为工具链的。

这样即可使用：

```
GCCGO=gccgo go-6 build xxx.go
```

如果gccgo设置成一个交叉编译的工具链中的gccgo，也是可以的。


可以用-n命令行参数查看一下：

```
fire3@ubuntu ~ $ GCCGO=gccgo ./go-6 build -n ./hello.go 

#
# command-line-arguments
#

mkdir -p $WORK/command-line-arguments/_obj/
mkdir -p $WORK/command-line-arguments/_obj/exe/
cd /home/fire3
/usr/bin/gccgo -I $WORK -c -g -m64 -fgo-relative-import-path=_/home/fire3 -o $WORK/command-line-arguments/_obj/_go_.o ./hello.go
ar rc $WORK/libcommand-line-arguments.a $WORK/command-line-arguments/_obj/_go_.o
cd .
/usr/bin/gccgo -o $WORK/command-line-arguments/_obj/exe/a.out $WORK/command-line-arguments/_obj/_go_.o -Wl,-( -m64 -Wl,--whole-archive -Wl,--no-whole-archive -Wl,- )
mv $WORK/command-line-arguments/_obj/exe/a.out hello

```

另外，如果我们的交叉编译gccgo参数不支持m64，可以写一个xgccgo脚本放在/usr/bin下,内容如下:

```
#!/bin/bash
args=$*
nargs=$(echo $args | sed "s/-m64//g")
mytarget-cros-linux-gnu-gccgo $nargs
```

然后使用:

```
GCCGO=xgccgo go-6 build hello.go
```
