title: sphinx and latex
date: 2016-12-17 16:28:24
tags:
---

新版的sphinx支持latex需要修改两个文件了：

```
/usr/local/lib/python2.7/dist-packages/sphinx/templates/latex/content.tex_t 
/usr/local/lib/python2.7/dist-packages/sphinx/writers/latex.py
```

第一个模板文件中找到位置加上2行CJK相关的配置。

```
\usepackage{multirow}
\usepackage{eqparbox}
\usepackage{CJKutf8}

<!-- snip -->
\end{CJK}
\end{document} 
```

第二个latex.py中需要改一下，在begin{document}后面增加一行。

```
BEGIN_DOC = r'''
\begin{document}
\begin{CJK}{UTF8}{gbsn}
```
