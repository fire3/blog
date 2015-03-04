title: "折腾Hexo第一天"
date: 2015-03-04 15:57:36
tags: "hexo"
---

## 安装

务必按照官方网站说明来：
``` bash
$ sudo npm install -g hexo-cli
```

如果运行了如下命令:
``` bash
npm install -g hexo
```
在deploy时我遇到了报错。

解决方法：
``` bash
sudo rm -f /usr/local/bin/hexo
sudo npm install -g hexo-cli
```

<!-- more -->

## 配置

设置了github 的deploy配置：

```
deploy:
  type: git
  repo: git@github.com:fire3/fire3.github.io.git
  branch: master
```

但是CNAME却总是在deploy时删除。发现CNAME必须放在source目录下。


##  主题

采用这个漂亮的主题： https://github.com/iissnan/hexo-theme-next

``` bash
$ git submodule add https://github.com/iissnan/hexo-theme-next themes/next
```

然后修改 _config.yml

```
theme: next
```
