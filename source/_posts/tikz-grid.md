title: Latex 中文米字格实现分析
date: 2017-01-06 16:30:38
tags:
---

上一回制作了中文米字格，初次接触Tikz，发现有必要深入了解一下，万一后面还有需要呢，比如，制作拼音格，制作描红格等等。

<!--more-->

网上的代码示例如下：

```
% \grid for a single character
\newcommand\grid[1]{%
\fontsize{160}{160}
\begin{tikzpicture}[baseline=(char.base)]
  \path[use as bounding box]
    (0,0) rectangle (1em,1em);
  \draw[help lines,step=0.5em]
    (0,0) grid (1em,1em);
  \draw[help lines,dashed]
    (0,0) -- (1em,1em)  (0,1em) -- (1em,0);
  \node[inner sep=0pt,anchor=base west]
    (char) at (0em,\gridraiseamount) {#1};
\end{tikzpicture}}                                                                                                                        
 
% \gridraiseamount is a font-specific value
\newcommand\gridraiseamount{0.12em}
```

我们先就这个片段搞搞清楚。

首先是：

```
\begin{tikzpicture}[baseline=(char.base)]

```
baseline是tikzpicture的一个option， 决定了这幅图在垂直方向上的基准位置。
char则是后面代码中node的名称，node是tikzpicture中的一个子图片。char.base是一个坐标位置，指示先绘制完node以后，计算出char.base这个锚点的位置，然后将baseline设置到这个点的纵坐标位置上。可以理解为baseline与这个点在垂直方向上一致。

```
\path[use as bounding box] (0,0) rectangle (1em,1em);

```

这里面 \path[use as bounding box]决定了这个tikzpicture的外围大小，这里设置成从1em大小的正方形。

em是一个相对长度单位，相当于大写字母M的宽度，实际比M要更款一些，比如当前字体设置为100pt，那么em就是100pt，而M宽度略小于100pt。

\path[use as bounding box] 可以简写为 \useasboundingbox ，效果相同。

接下来是：

```
  \draw[help lines,step=0.5em]
    (0,0) grid (1em,1em);
```

help lines是一个style，可以修改，比如：

```
\tikzstyle help lines=[color=blue!50,very thin]
\tikzstyle help lines=[color=red!50,thin]
```

\draw 则是绘制控制了，这里绘制了一个step=0.5em的grid，grid大小从（0,0）到(1em ,1em)，这一步画出了田字格。

```
  \draw[help lines,dashed]
    (0,0) -- (1em,1em)  (0,1em) -- (1em,0);
```

这一步则是绘制出了两个斜45度交叉的虚线，并且style同样是help lines。

最后绘制我们的字符，利用了一个node,如下：

```
  \node[inner sep=0pt,anchor=base west]
    (char) at (0em,\gridraiseamount) {#1};

```

这里需要掌握node，anchor的概念。node是一个sub picture，可以利用{}添加文字。anchor用于定位，首先确定选用哪一个anchor，然后用at确定该ahchor的位置。

比如：

![](/images/tikz-node-anchor.png)

这里第二个node，anchor在west，也就是Circle一词的最左边，把这个anchor放置在c.east，c是第一个node的名称，就是放在第一个node的“东边”，即右边。

回头看我们这个例子，anchor选择了base west，定位到(0em,0.12em)的位置，其中0.12em是稍微调整了一下，使得汉字在米字格里面显得更加居中。用如下的代码也可以实现类似的效果：

```
  \node[inner sep=0pt,anchor=south west]
    (char) at (0em,0.05em) {#1};
```

选用south west的anchor，并适当调整位置即可。

最终效果如下：

![](/images/tikz-2.png)

接下来，我们准备再改造一下这个grid宏，让它可以加上拼音：

```
\newcommand\pygrid[1]{%
\fontsize{100}{100}
\begin{tikzpicture}[baseline=(char.base)]
  \tikzstyle help lines=[color=red!50,thin]
  \path[use as bounding box]
    (0,0) rectangle (1em,1.6em);
  \draw[help lines,step=0.5em]
    (0,0) grid (1em,1em);
  \draw[help lines,dashed]
    (0,0) -- (1em,1em)  (0,1em) -- (1em,0);
    \draw[help lines] (0em,1em) -- (0em,1.6em)
    (0em,1.6em) -- (1em,1.6em) (1em,1.6em) -- (1em,1em);
    \draw[help lines,dashed] (0em,1.2em) -- (1em,1.2em);
    \draw[help lines,dashed] (0em,1.4em) -- (1em,1.4em);
  \node[inner sep=0pt,anchor=south west]
    (char) at (0em,0.06em) {#1};
\end{tikzpicture}}
```

这样就可以实现这样的效果了：

![](/images/tikz-3.png)

