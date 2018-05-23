---
title: git中闹不明白的autocrlf
date: 2018-05-23 15:10:52
categories:
- git
tags:
- git
---
git管理代码仓库的时候，会经常遇到这样的情况：
```shell
warning: CRLF will be replaced by LF in Test.java.
The file will have its original line endings in your working directory.
```
遇到上面的警告， 基本上都是将 autocrlf 设置为 false　解决，或者手动改变文件的CRLF属性。（IDEA具备这个功能，其他编辑器不知道）

## 什么是CRLF?
CRLF是Carriage-Return Line-Feed的缩写，意思是回车换行，就是回车(CR, ASCII 13, \r) 换行(LF, ASCII 10, \n)。
## 有什么不同？
* CRLF->Windows-style
* LF->Unix Style
* CR->Mac Style

CRLF表示句尾使用回车换行两个字符(即我们常在Windows编程时使用"\r\n"换行)
LF表示表示句尾，只使用换行.
CR表示只使用回车.
## git的三种模式
core.autocrlf有三种模式，分别对应三种转换情况。x代码换行符号，第一个是pull的转换情况，第二个是commit的时候情况。
1) true:             x -> LF -> CRLF
2) input:            x -> LF -> LF
3) false:            x -> x -> x
### 换行符=LF,autocrlf=true
如果你的源文件中是换行符是LF，而autocrlf=true, 此时git add就会遇到 fatal: LF would be replaced by CRLF 的错误。有两个解决办法：
1. 将源文件中的LF转为CRLF即可【推荐】
2. 将autocrlf 设置为 false

### 换行符=CRLF,autocrlf=input
如果你的源文件中是换行符是CRLF，而autocrlf=input,  此时git add也会遇到 fatal: CRLF would be replaced by LF 的错误。有两个解决办法：
1. 将你源文件中的CRLF转为LF【推荐】
2. 将autocrlf 设置为true 或者 false

我的建议：在Unix/Linux上设置 autocrlf = input, 在Windows上设置autocrlf = true（默认值）。
----------------------------------------------------------------------------------------------------------------------------------
这样的话，
Windows：（true）
提交时，将CRLF 转成 LF再提交；
切出时，自动将LF 转为 CRLF;

Unix/Linux: (input)
提交时,   将CRLF 转成 LF再提交；
切出时，保持LF即可

这样即可保证仓库中永远都是LF. 而且在Windows工作空间都是CRLF, Unix/Linux工作空间都是LF.
----------------------------------------------------------------------------------------------------------------------------------
core.autocrlf
假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。

Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。用core.autocrlf来打开此项功能，如果是在Windows系统上，把它设置成true，这样当签出代码时，LF会被转换成CRLF：

```shell
$ git config --global core.autocrlf true  
```

Linux或Mac系统使用LF作为行结束符，因此你不想 Git 在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把core.autocrlf设置成input来告诉 Git 在提交时把CRLF转换成LF，签出时不转换：

```shell
$ git config --global core.autocrlf input  
```
这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。
如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置false取消此功能，把回车符记录在库中：
```shell
$ git config --global core.autocrlf false  
```

