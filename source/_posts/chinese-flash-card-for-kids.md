title: chinese flash card for kids
date: 2017-01-05 10:02:26
tags:
---

小朋友马上中班下学期了，阅读量越来越大，又很喜欢读绘本，准备开始多教一些汉字了。程序员老爹的思路和硬件工程师老妈的思路迥然不同，硬件工程师老妈准备从硬件入手，先裁剪小纸卡，然后一个一个做字卡。软件工程师老爹自然是从软件入手，准备自动化的生成可打印的字卡，提高可扩展性和生产效率。

<!-- more -->

软件老爹经过一番调研，准备祭出latex这个大杀器。做字卡，简简单单的大字当然不够帅，为了达到酷炫的效果，必须给大字加上自动化的米字格，这样也算为小学以后做字帖做“预研”了。经过一番google，发现了这个pstricks的技巧：

```
\documentclass[pstricks]{standalone}
\usepackage{CJKutf8}
\usepackage[overlap,CJK]{ruby}
\newpsstyle{gridstyle}{gridlabels=0pt,subgriddiv=1,gridcolor=lightgray}
\renewcommand\rubysep{-0.1ex}
\newsavebox\IBox
\newcommand\prepare[2][10]{\sbox\IBox{\raisebox{\depth}{\psscalebox{#1}{#2}}}}

\begin{document}
\begin{CJK*}{UTF8}{gkai}
\prepare[15]{犬}
\psset{xunit=\dimexpr\wd\IBox/2,yunit=\dimexpr\ht\IBox/2,linecolor=lightgray}
\begin{pspicture}[showgrid](2,2)
    \psline(2,0)(0,2)
    \psline(2,2)
    \rput[bl](0,0){\usebox\IBox}
\end{pspicture}
\end{CJK*}
\end{document}

```

这个tex文件经过latex->dvipdf编译后就是一个漂亮的带米字格的大字喽。

接下来的思路很简单，按照需要进行自动化生成和排版就可以喽。当然，这里的需要主要来自于硬件老妈的“客户需求”。

* 需求1：支持裁剪分割线。
* 需求2：大字在子卡里面的位置偏左，右边留下点空白可以写写画画。


这个需求不高嘛，程序员老爹分分钟搞定。不过牛皮吹出去了，但latex里面博大精深，虽说肯定能搞定，但是你让我手写一个像上面例子那样的代码我肯定不能在一个晚上搞定的。所以说，思路很重要，google的关键字很重要：）

通过google，我找到了一些控制table宽度和行高度的方法，也知道了table里面可以放pdf图片，有了这个思路，稍加实验就可以喽。大概样子是这样的：


```
\documentclass{article}
\usepackage[a4paper,lmargin=0mm,top=10mm]{geometry}
\usepackage{graphicx}
\usepackage{caption}
\usepackage{subcaption}
\usepackage{framed}
\usepackage[export]{adjustbox}
\usepackage{array}
\pagenumbering{gobble}
\graphicspath{{./diag/}}
\newcolumntype{L}[1]{>{\raggedright\let\newline\\\arraybackslash\hspace{10pt}}m{#1}}
\newcolumntype{C}[1]{>{\centering\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{R}[1]{>{\raggedleft\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{M}[1]{>{\centering\arraybackslash}m{#1}}
\newcolumntype{N}{@{}m{0pt}@{}}
\begin{document}
\begin{table}[ht]
    \begin{tabular}{L{101mm}|L{104mm}N}
        \hline
        \includegraphics[height=5cm]{tt} & \includegraphics[height=5cm]{tt} & \\[6.5cm]
        \hline
        \includegraphics[height=5cm]{tt} & \includegraphics[height=5cm]{tt} & \\[6.5cm]
        \hline
        \includegraphics[height=5cm]{tt} & \includegraphics[height=5cm]{tt} & \\[6.5cm]
        \hline
        \includegraphics[height=5cm]{tt} & \includegraphics[height=5cm]{tt} & \\[6.5cm]
        \hline
    \end{tabular}
\end{table}
\end{document}
```

pdf的实验效果令人满意，剩下就是自动化的事情了，随手写一个shell脚本就能搞定的事：

```

#!/bin/bash

. ./shflags
DEFINE_string file 'word' "input word file" i
FLAGS "$@" || exit $?
eval set -- "${FLAGS_ARGV}"

INPUT=`pwd`/${FLAGS_file}
CURRENT=`pwd`
DIR=`mktemp -d`
cd $DIR
#echo $DIR
function make_char()
{
mkdir diag
cd diag
words=`cat $INPUT`

c=0
for w in $words
do
    c=$(($c + 1))
    > $c.tex
    cat >$c.tex<<'EOF'
\documentclass[pstricks]{standalone}
\usepackage{CJKutf8} 
\usepackage[overlap,CJK]{ruby}   
\newpsstyle{gridstyle}{gridlabels=0pt,subgriddiv=1,gridcolor=lightgray}
\renewcommand\rubysep{-0.1ex}
\newsavebox\IBox
\newcommand\prepare[2][10]{\sbox\IBox{\raisebox{\depth}{\psscalebox{#1}{#2}}}}
                                                                                                                                          
\begin{document} 
\begin{CJK*}{UTF8}{gkai}
EOF
echo "\prepare[15]{$w}" >> $c.tex
cat >>$c.tex<<'EOF'
\psset{xunit=\dimexpr\wd\IBox/2,yunit=\dimexpr\ht\IBox/2,linecolor=lightgray}
\begin{pspicture}[showgrid](2,2) 
\psline(2,0)(0,2)  
\psline(2,2)     
\rput[bl](0,0){\usebox\IBox} 
\end{pspicture} 
\end{CJK*}      
\end{document}
EOF
latex $c.tex >/dev/null 2>&1
dvipdf $c.dvi
rm -rf $c.log $c.dvi $c.aux
done
cd - > /dev/null 2>&1
return $c
}

function make_book()
{

c=$1
cat >book.tex<<'EOF'
\documentclass{article}
\usepackage[a4paper,lmargin=0mm,top=10mm]{geometry}
\usepackage{graphicx}
\usepackage{caption}
\usepackage{subcaption}
\usepackage{framed}
\usepackage[export]{adjustbox}
\usepackage{array}
\pagenumbering{gobble}
\graphicspath{{./diag/}}
\newcolumntype{L}[1]{>{\raggedright\let\newline\\\arraybackslash\hspace{10pt}}m{#1}}
\newcolumntype{C}[1]{>{\centering\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{R}[1]{>{\raggedleft\let\newline\\\arraybackslash\hspace{0pt}}m{#1}}
\newcolumntype{M}[1]{>{\centering\arraybackslash}m{#1}}
\newcolumntype{N}{@{}m{0pt}@{}}
\begin{document}
EOF

table_count=$(($c/8))

char=1
for(( i=0; i<$table_count; i++))
do
    echo "\begin{table}[ht]" >>book.tex
    echo "\begin{tabular}{L{101mm}|L{104mm}N}" >>book.tex
    echo "\hline" >>book.tex
    echo "\\includegraphics[height=5cm]{$((char++))} & \\includegraphics[height=5cm]{$((char++))} & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\includegraphics[height=5cm]{$((char++))} & \includegraphics[height=5cm]{$((char++))} & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\includegraphics[height=5cm]{$((char++))} & \includegraphics[height=5cm]{$((char++))} & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\includegraphics[height=5cm]{$((char++))} & \includegraphics[height=5cm]{$((char++))} & \\\\[6.5cm]" >>book.tex
    echo "\hline" >>book.tex
    echo "\end{tabular}" >>book.tex
    echo "\end{table}" >>book.tex
done

echo "\end{document}" >> book.tex

pdflatex book.tex > /dev/null 2>&1
cp -rf book.pdf $CURRENT/
}

make_char
c=$?
make_book $c
rm -rf $DIR

echo "book.pdf generated."
```

这样我只需要准备一个'word'文件，如这样：

```
大 小 多 少 前 后 左 右
山 水 花 鸟 鱼 虫 狗 猪
```

就可以自动按需生成打印版的字卡啦：）


PS：脚本偷懒，一页8个字，如果word文件里面最后不足8个字，将不生成。
