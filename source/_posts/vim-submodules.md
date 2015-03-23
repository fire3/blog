title: "vim submodules"
date: 2015-03-23 16:33:54
tags:[vim,git]
---

最近开始用submodule管理vim插件，但是有一个烦人的事情：

```
fire3@fire3-Lenovo:~/.vim$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

   modified:   bundle/CuteErrorMarker (untracked content)
   modified:   bundle/OmniCppComplete (untracked content)
   modified:   bundle/VisIncr (untracked content)
   modified:   bundle/autoclose (untracked content)
   modified:   bundle/javacomplete (untracked content)
   modified:   bundle/matchit (untracked content)
   modified:   bundle/snipmate-snippets (modified content)
   modified:   bundle/vim-multiple-cursors (untracked content)
   modified:   bundle/xmledit (untracked content)
   modified:   bundle/yankring (untracked content)

no changes added to commit (use "git add" and/or "git commit -a")

```

出现了很多`modified`状态。仔细一看都是生成了`doc/tags`文件。解决方法：

```
$ vim ~/.gitignore
doc/tags

$ git config --global core.excludesfile ~/.gitignore
```

当然，也可以选择无视，呵呵。
