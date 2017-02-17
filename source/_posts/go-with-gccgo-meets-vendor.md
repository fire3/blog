title: go vendor 包管理和gccgo不兼容
date: 2017-02-17 15:48:40
tags:
---

vendor固然好，可惜gccgo配合go工具并不能用。

<!-- more -->

在编译coreos的mayday时，这个项目使用了glide管理的vendor目录，
实验如下：

```
go get github.com/coreos/mayday
go build -compiler gccgo
```

各种报错，找不到依赖关系，我们可以这样操作：

```
cp -a src/github.com/coreos/mayday/vendor/* src/
```

把依赖的各种包都放到$GOPATH/src 下面后，再用gccgo编译：

```
go build -compiler gccgo github.com/coreos/mayday
```

这样就可以了！

另外，gccgo工具包里面安装的go-6之类的工具，默认就采用gccgo编译

```
go-6 build github.com/coreos/mayday
```

这样也是可以的。
