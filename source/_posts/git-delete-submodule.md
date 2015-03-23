title: "git delete submodule"
date: 2015-03-23 16:28:26
tags: [git]
---

git的submodule，添加容易，删除困难。

<!-- more -->

Git并没有内置删除submodule的命令。

* 第一步： 删除.gitmodules文件中的相关信息。
* 第二步：删除.git/config 文件中的相关信息。
* 第三步：删除相关目录，比如利用命令`git rm --cached submodules/xxx`

以上三步，缺一不可。
