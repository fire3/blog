title: "tar exclude vcs"
date: 2015-03-17 23:55:12
tags:
---

今天学会一招，tar命令的参数`--exclude-vcs`，这个很好用，可以方便的排除常见的版本库目录！

比如:

```
tar -cf source.tar --exclude-vcs ./source
```
