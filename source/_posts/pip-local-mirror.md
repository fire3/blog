title: "pip local mirror"
date: 2016-03-28 18:37:37
tags:
---

pip安装软件方便，但是在一个没有互联网的环境里就抓瞎了。发现了一个这个好工具：

https://github.com/wolever/pip2pi 可以方便的把pip软件依赖的各种py包都爬下来，利用 pip2tgz 命令就可以了：

```
$ pip2tgz packages/ mkdocs
```


