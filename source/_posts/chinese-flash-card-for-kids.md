title: 中文字卡快速制作
date: 2017-01-05 10:02:26
tags:
---

小朋友马上中班下学期了，阅读量越来越大，又很喜欢读绘本，准备开始多教一些汉字了。程序员老爹的思路和硬件工程师老妈的思路迥然不同，硬件工程师老妈准备从硬件入手，先裁剪小纸卡，然后一个一个做字卡。软件工程师老爹自然是从软件入手，准备自动化的生成可打印的字卡，提高可扩展性和生产效率。

<!-- more -->

先上图，最终效果如下：

![](/images/book.png)

硬件工程师老妈的需求如下：

* 最好有米字格。
* 有字卡的裁剪线。
* 每个字卡的字偏左，右边留点空间写写画画

这个需求如果用word来干不要折腾死软件工程师老爹我么，还好我懂latex，还会google，会点shell脚本。

Google一番以后，形成了主要思路。

* 利用网上大神写好的tikz宏给字加上米字格
* 放在表格里面排版，方便控制位置
* 控制表格的宽度和高度，加上表格边框以便裁剪
* 自动化脚本生成，方便后期添加新字

附脚本如下：
```
#!/bin/bash

INPUT=`pwd`/word
CURRENT=`pwd`
DIR=`mktemp -d`
cd $DIR

function make_book()
{

word=`cat $INPUT`

unset chars
for c in $word
do
    chars[${#chars[@]}]=$c
done
count=${#chars[@]}

table_count=$(($count/8))

cat >book.tex <<'EOF'
\documentclass{article}
\usepackage{xeCJK}
\setCJKmainfont{AR PL KaitiM GB}
\usepackage{tikz}
\usepackage[a4paper,lmargin=0mm,top=10mm]{geometry}
\usepackage[export]{adjustbox}
\usepackage{array}
\pagenumbering{gobble}

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

\newcolumntype{L}[1]{>{\raggedright\let\newline\\\arraybackslash\hspace{10pt}}m{#1}}
\newcolumntype{C}[1]{>{\centering\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{R}[1]{>{\raggedleft\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{M}[1]{>{\centering\arraybackslash}m{#1}}
\newcolumntype{N}{@{}m{0pt}@{}}

% \Grid for a CJK string
\makeatletter
\newcommand\Grid[1]{%
  \@tfor\z:=#1\do{\grid{\z}}}
\makeatother

\begin{document}

\xeCJKsetup{PunctStyle=plain}

EOF
char=0
for(( i=0; i<$table_count; i++))
do
    
    echo "\begin{table}[ht]" >>book.tex
    echo "\begin{tabular}{L{101mm}|L{104mm}N}" >>book.tex
    echo "\hline" >>book.tex
    echo "\\grid{${chars[$((char++))]}} & \\grid{${chars[$((char++))]}}  & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\\grid{${chars[$((char++))]}} & \\grid{${chars[$((char++))]}}  & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\\grid{${chars[$((char++))]}} & \\grid{${chars[$((char++))]}}  & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\\grid{${chars[$((char++))]}} & \\grid{${chars[$((char++))]}}  & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\end{tabular}" >>book.tex
    echo "\end{table}" >>book.tex
done

echo '\end{document}' >>book.tex
xelatex book.tex > /dev/null 2>&1
cp -rf book.pdf $CURRENT/
cp -rf book.tex $CURRENT/
rm -rf $DIR
}

make_book

echo "book.pdf generated."
```
