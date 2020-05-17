---
title: "CMake入门"
date: 2020-05-17T14:17:09+08:00
draft: true

categories:
  - "cmake"
tags:
  - "cmake"
  - "cpp"

---

CMake是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。只是 CMake 的组态档取名为 CMakeLists.txt。Cmake 并不直接建构出最终的软件，而是产生标准的建构档（如 Unix 的 Makefile 或 Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是 CMake 和 SCons 等其他类似系统的区别之处。

![](../../img/cmake-start-index.jpg)

<!--more-->

### 代码仓库
https://github.com/gongluck/CMake-Demo.git

### 简单项目
创建目录**mydemo**，再在**mydemo**目录中创建**CMakeLists.txt**和**main.c**两个文件。
```C++
//main.c
#include <stdio.h>

int main()
{
    printf("hello, cmake!\n");
    return 0;
}
```
```CMake
#CMakelists.txt
# CMake最低版本要求
cmake_minimum_required(VERSION 3.0)

# 项目名称
project(mydemo)

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. SRCS)

# 指定生成目标
add_executable(mydemo ${SRCS})
```
之后执行命令：
```command
cmake -S . -B ./build
cmake --build ./build
```
第一条命令是生成目标平台的项目，将生成使用系统上默认的编译套件的工程。***-S***后面的参数是**CMake**工程**CMakeLists.txt**文件所在目录，***-B***后面的参数是将要生成的目标平台项目文件存放的目录。
第二条命令是构建项目，当然也可以通过其他工具打开项目构建或者执行***make***命令等。***--build***是**CMake**程序构建的命令，后面的参数是需要构建的项目的路径。
![cmake_build.jpg](../../img/cmake_build.jpg)