title: bash memo
date: 2016-12-18 09:06:43
tags: [linux]
---

好记性不如烂笔头。Bash博大精深，里面门道多了。

<!-- more -->

一些变量操作模式
===========================

```
${var:-default}  : 如果var为unset或者empty，则设置为default值。
${var-default}  : 如果var为unset，则设置为default值。
${#var}: Length of Variable’s Contents
${var%PATTERN}: Remove the Shortest Match from the End
${var%%PATTERN}: Remove the Longest Match from the End
${var#PATTERN}: Remove the Shortest Match from the Beginning
${var##PATTERN}: Remove the Longest Match from the Beginning
${var//PATTERN/STRING}: Replace All Instances of PATTERN with STRING
${var:OFFSET:LENGTH}: Return a Substring of $var
${var^PATTERN}: Convert to Uppercase
${var,PATTERN}: Convert to Lowercase
```

数组
===========================

Integer-Indexed Arrays
--------------------------


显示数组：
```
$ printf "%s\n" "${BASH_VERSINFO[0]}"
4

$ printf "%s\n" "${BASH_VERSINFO[1]}"
0

$ printf "%s\n" "${BASH_VERSINFO[*]}"
4 0 10 1 release i686-pc-linux-gnuoldld

$ printf "%s\n" "${BASH_VERSINFO[@]}"
4
0
10
1
release
i686-pc-linux-gnuoldld

$ printf "%s\n" "${BASH_VERSINFO[@]:1:2}" ## minor version number and patch level
0
10

```
变量个数测试：
```
$ printf "%s\n" "${#BASH_VERSINFO[*]}"
6
```

具体某个变量的长度：
```
$ printf "%s\n" "${#BASH_VERSINFO[2]}" "${#BASH_VERSINFO[5]}"
2
22
```

设置数组元素：
```
$ unset a
$ a[${#a[@]}]="1 $RANDOM" ## ${#a[@]} is 0
$ a[${#a[@]}]="2 $RANDOM" ## ${#a[@]} is 1
$ a[${#a[@]}]="3 $RANDOM" ## ${#a[@]} is 2
$ a[${#a[@]}]="4 $RANDOM" ## ${#a[@]} is 3
$ printf "%s\n" "${a[@]}"
1 6007
2 3784
3 32330
4 25914
```

一次性声明数组：
```
$ province=( Quebec Ontario Manitoba  )
$ printf "%s\n" "${province[@]}"
Quebec
Ontario
Manitoba
```

增加数组元素：
```
$ province+=( Saskatchewan  )
$ province+=( Alberta "British Columbia" "Nova Scotia"  )
$ printf "%-25s %-25s %s\n" "${province[@]}"
```

Associative Arrays
-----------------------

相当于key-value数组，需要用declare -A声明。

```
$ declare -A array
$ for subscript in a b c d e
> do
> array[$subscript]="$subscript $RANDOM"
> done
$ printf ":%s:\n" "${array["c"]}" ## print one element
:c 1574:
$ printf ":%s:\n" "${array[@]}" ## print the entire array
:a 13856:
:b 6235:
:c 1574:
:d 14020:
:e 9165:
```
