---
title: "在golang中使用C"
date: 2020-04-09T19:31:36+08:00
draft: true
description : "在golang中使用C"
categories:
  - "go"
tags:
  - "go"
menu : side # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
thumbnail: "img/first.jpg" # Thumbnail image
lead: "在golang中使用C" # Lead text
comments: true # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---

C/C++结合GO,可以说就是站在巨人的肩膀上做软件设计。
C/C++经过几十年的发展，已经积累了庞大的软件资产，它们很多久经考验而且性能已经足够优化。Go语言必须能够站在C/C++这个巨人的肩膀之上，有了海量的C/C++软件资产兜底之后，我们才可以放心愉快地用Go语言编程。C语言作为一个通用语言，很多库会选择提供一个C兼容的API，然后用其他不同的编程语言实现。Go语言通过自带的一个叫CGO的工具来支持C语言函数调用，同时我们可以用Go语言导出C动态库接口给其它语言使用。

<!--more-->

### 代码仓库
https://github.com/gongluck/CGO-DEMO.git

### 准备
使用CGO,需要先安装gcc或者mingw。

### 启用CGO特性
```GO
package main

import "C"

func main() {
	println("hello, cgo")
}
```
就是这么简单,一句
```GO
import "C"
```
就是告诉
```GO
go build
```
命令在编译和链接阶段启动gcc编译器。