---
title: "CGO导入导出C静态库/动态库"
date: 2020-04-12T13:50:36+08:00
draft: false

featured_image: "img/cgo.jpg"
categories: go
tags: [go, cgo]

---

可以直接使用C源码到GO中当然是最好不过了,但如果只有三方库的静态库或者动态库,就要学会如何在GO中使用C的静态库或者动态库了。

<!--more-->

### 代码仓库
https://github.com/gongluck/CGO-DEMO.git

### GO使用C静态库
首先把下面的C代码编译成静态库
```
int number_add_mod(int a, int b, int mod);
```
```
#include "number.h"

int number_add_mod(int a, int b, int mod)
{
    return (a + b) % mod;
}
```
编译静态库使用命令
```
gcc -c -o number.o number.c
ar rcs libnumber.a number.o
```
接下来就可以在GO中使用了
```
package main

//#cgo CFLAGS: -I./number
//#cgo LDFLAGS: -L${SRCDIR}/number -lnumber
//#include "number.h"
import "C"
import "fmt"

func main() {
	fmt.Println(C.number_add_mod(10, 5, 12))
}
```
***#cgo*** 注释中通过 ***CFLAGS*** 和 ***LDFLAGS*** 指定头文件包含路径和静态库的路径。基本和使用 ***gcc*** 差别不大。

如果是VC环境,创建number.def文件说明导出函数的信息
```
LIBRARY number.lib

EXPORTS
number_add_mod
```
进入VC对应的命令行环境执行
```
cl /c number.c
link /LIB /OUT:number.lib number.obj /DEF:number.def
```
使用mingw工具执行
```
dlltool -dllname number.lib --def number.def --output-lib libnumber.a
```
得到了 ***libnumber.a*** 文件。GO使用lib需要这个 ***libnumber.a*** 作为代替。

### GO使用C动态库
gcc环境使用下面的命令编译动态库
```
gcc -shared -o libnumber.so number.c
```
使用前面例子的GO代码就可以使用动态库了。
如果是VC环境,创建number.def文件说明导出函数的信息
```
LIBRARY number.dll

EXPORTS
number_add_mod
```
进入VC对应的命令行环境执行
```
cl /c number.c
link /DLL /OUT:number.dll number.obj /DEF:number.def
```
使用mingw工具执行
```
dlltool -dllname number.dll --def number.def --output-lib libnumber.a
```
得到了 ***libnumber.a*** 文件。GO使用dll需要这个 ***libnumber.a*** 作为导入库。

**在运行时需要将动态库放到系统能够找到的位置。**

### GO导出C静态库/动态库
```
package main

import "C"

func main() {}

//export number_add_mod
func number_add_mod(a, b, mod C.int) C.int {
	return (a + b) % mod
}
```
此时,main函数仍需要保留,但是可以不执行任何代码。使用 ***//export*** 注释表明要导出的GO函数。编译命令要加上目标参数。
```
go build -buildmode=c-archive -o number.a
go build -buildmode=c-shared -o number.so
```
编译成功同时会生成对应的头文件。
