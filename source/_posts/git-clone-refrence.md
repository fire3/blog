title: "git clone reference"
date: 2015-03-18 20:19:13
tags:
---

git clone linux是一个漫长的过程，所以建议采用reference的方式clone，如下：

```
git clone  --reference ./linux-stable.git/.git/  https://github.com/penberg/linux-kvm.git
```

reference可以是之前clone好的一个本地目录。
