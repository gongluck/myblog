---
title: "CGO入门"
date: 2020-04-09T19:31:36+08:00
draft: false

featured_image: "img/cgo.jpg"
categories: go
tags: [go, cgo]

---

C/C++加上GO代表什么？

代表着既可以使用GO快速的开发项目,同时可以接纳C/C++的庞大历史遗产和极高性能！而且在GO中使用C/C++十分简单！

<!--more-->

### 代码仓库
https://github.com/gongluck/CGO-DEMO.git

### 准备
使用CGO,需要先安装gcc或者mingw。

### 启用CGO特性
    package main

    import "C"

    func main() {
	    println("hello, cgo")
    }
就是这么简单,一句 ***import "C"*** 就是告诉 ***go build*** 命令在编译和链接阶段启动gcc编译器。

### 使用C标准库的函数
    package main

    //#include <stdio.h>
    import "C"

    func main() {
	    C.puts(C.CString("hello, cgo"))
    }
***import "C"*** 上面紧接着的注释就是C代码,上面就是将标准库的头文件包含进我们的代码中。在GO中使用C的代码都是在伪包"C"中,所以使用C标准库中的 ***puts*** 函数就是 ***C.puts(...)*** 。由于C和GO中的字符(串)类型不是等价的,使用 ***C.CString(...)*** 将GO中的 ***string*** 转换成C的 ***char**** 。

### 使用自定义C函数
    package main

    /*
    #include <stdio.h>

    static void SayHello(const char* s)
    {
	    puts(s);
    }
    */
    import "C"

    func main() {
	    C.SayHello(C.CString("hello, cgo\n"))
    }
和前面使用C标准库的函数的例子差不多,只是将自定义函数的定义放到 ***import "C"*** 上面的注释中,接下来就可以在GO中通过"C"包使用了。另外,也可以将自定义C函数的定义放到C源码文件中,GO代码只在 ***import "C"*** 上面添加该函数的声明。
    
    package main

    //void SayHello(const char* s);
    import "C"

    func main() {
	    C.SayHello(C.CString("hello, cgo\n"))
    }
C源码文件的内容

    #include <stdio.h>

    void SayHello(const char* s)
    {
	    puts(s);
    }
如果两个源码文件都放在同一层目录下,可以直接使用 ***go build*** 编译。

其实,自定义的C函数也可以用C++实现
    
    #include <iostream>

    extern "C" {
	    #include "hello.h"
    }

    void SayHello(const char* s) {
	    std::cout << s;
    }
但是现在就必须通过一个C头文件给GO声明该函数
    
    void SayHello(const char* s);
GO代码改为
    
    package main

    //#include "hello.h"
    import "C"

    func main() {
	    C.SayHello(C.CString("hello, cgo\n"))
    } 
**函数的声明是C的,但是具体内部实现可以是C++的。而且能和GO结合使用的只能是C。**
    
### GO导出C接口
前面的例子都是GO调用C源码(接口),下面使用GO导出C接口。
首先在C头文件中声明函数,为了适配GO,注释了 ***const*** 修饰。

    void SayHelloGo(/*const*/ char* s);
在GO源码中实现该接口
    
    package main

    import "C"

    import "fmt"

    //export SayHelloGo
    func SayHelloGo(s *C.char) {
	    fmt.Print(C.GoString(s))
    }
通过CGO的 ***//export SayHello*** 指令将Go语言实现的函数导出为C语言函数。接下来就可以使用该实际是GO实现的C接口了
    
    package main

    //#include <hello.h>
    import "C"

    func main() {
	    C.SayHelloGo(C.CString("hello, cgo Go\n"))
    }

### 面向C接口的GO
    package main

    //void SayHello(_GoString_ s);
    import "C"

    import (
	    "fmt"
    )

    func main() {
	    C.SayHello("hello, cgo go")
    }

    //export SayHello
    func SayHello(s string) {
	    fmt.Print(s)
    }
***从Go1.10开始CGO新增加了一个_GoString_预定义的C语言类型，用来表示Go语言字符串。***
上面一个GO源码文件中,发生了这样的调用过程。GO的main函数中,调用了"C"包中的 ***SayHello*** 这个C函数,实际这个 ***SayHello*** C函数是GO实现的,所以转向调用GO中的 ***SayHello*** 。