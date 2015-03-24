title: "update git submodule"
date: 2015-03-24 22:52:59
tags: [git]
---

Git 的submodule如果想跟upstream同步一下应该怎么办呢？ 

<!-- more -->

不是 git submodule update ,而是应该如下操作：

``` bash
# get the submodule initially
git submodule add ssh://bla submodule_dir
git submodule init

# time passes, submodule upstream is updated
# and you now want to update

# change to the submodule directory
cd submodule_dir

# checkout desired branch
git checkout master

# update
git pull

# get back to your project root
cd ..

# now the submodules are in the state you want, so
git commit -am "Pulled down update to submodule_dir"

```

参加： http://stackoverflow.com/questions/5828324/update-git-submodule-to-latest-commit-on-origin?answertab=votes#tab-top
