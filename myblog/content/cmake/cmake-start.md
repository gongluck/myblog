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

**CMake**是一个跨平台的安装(编译)工具，可以用简单的语句来描述所有平台的安装(编译过程)。能够输出各种各样的**makefile**或者**project**文件，能测试编译器所支持的**C++**特性,类似**UNIX**下的**automake**。只是 **CMake**的组态档取名为**CMakeLists.txt**。**Cmake**并不直接建构出最终的软件，而是产生标准的建构档(如 **Unix**的**Makefile**或**Windows Visual C++**的**projects/workspaces**)，然后再依一般的建构方式使用。这使得熟悉某个集成开发环境(IDE)的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是**CMake**和**SCons**等其他类似系统的区别之处。

至于出现一份代码，到处编译，但是编译不过就有可能是代码的问题了。

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

### 增加子模块
创建目录**myfun**，再在**myfun**目录中创建**CMakeLists.txt**、**myfun.h**和**myfun.c**三个文件。
```C++
//myfun.h
#ifndef __MYFUN_H__
#define __MYFUN_H__

int myfun(int a, int b);

#endif//__MYFUN_H__
```
```C++
//myfun.c
#include "myfun.h"

int myfun(int a, int b)
{
    return a+b;
}
```
```CMake
#CMakelists.txt
# CMake最低版本要求
cmake_minimum_required(VERSION 3.0)

# 获取当前文件夹名
STRING(REGEX REPLACE ".*/(.*)" "\\1" CURRENT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR})

# 项目名称
project(${CURRENT_FOLDER})

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. SRCS)

# 生成链接库
add_library(${PROJECT_NAME} ${SRCS})
```
子模块中的
```CMake
# 生成链接库
add_library(${PROJECT_NAME} SHARED ${SRCS})
```
***SHARED***可以省略，省略后相当于***STATIC***。

**mydemo**下的**CMakeLists.txt**和**main.c**也要做修改。然后，**重新构建生成**。
```C++
//main.c
#include <stdio.h>
#include "myfun/myfun.h"

int main()
{
    printf("hello, cmake!\n");
    printf("1+1=%d\n", myfun(1, 1));
    return 0;
}
```
```CMake
#CMakelists.txt
# CMake最低版本要求
cmake_minimum_required(VERSION 3.0)

# 获取当前文件夹名
STRING(REGEX REPLACE ".*/(.*)" "\\1" CURRENT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR})

# 项目名称
project(${CURRENT_FOLDER})

# 设置变量
set(SUBMODULE myfun)

# 添加子目录
# 必须放在aux_source_directory前,否则同名变量SRCS会冲突
add_subdirectory(${SUBMODULE})

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. SRCS)

# 指定生成目标
add_executable(${PROJECT_NAME} ${SRCS})

# 添加链接库
target_link_libraries(${PROJECT_NAME} ${SUBMODULE})
```
主模块中主要改动在
```CMake
# 添加子目录
# 必须放在aux_source_directory前,否则同名变量SRCS会冲突
add_subdirectory(${SUBMODULE})

# 添加链接库
target_link_libraries(${PROJECT_NAME} ${SUBMODULE})
```
***add_subdirectory***命令添加子模块目录，**CMake**会自动执行子模块中的**CMakelists.txt**。
***target_link_libraries***命令是将子模块链接到主模块。
![cmake_build2.PNG](../../img/cmake_build2.PNG)

### 自定义编译选项
主模块的**CMakelists.txt**修改成：
```CMake
#CMakeLists.txt
# CMake最低版本要求
cmake_minimum_required(VERSION 3.0)

# 获取当前文件夹名
STRING(REGEX REPLACE ".*/(.*)" "\\1" CURRENT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR})

# 项目名称
project(${CURRENT_FOLDER})

# 加入一个配置头文件用于处理 CMake 对源码的设置
configure_file(
  ${PROJECT_SOURCE_DIR}/config.h.in
  ${PROJECT_BINARY_DIR}/config.h
  )

# 自定义编译选项
option(USESUBMODULE "use submodule" ON)
if (USESUBMODULE)
    # 设置变量
    set(SUBMODULE myfun)
    
    # 添加包含路径
    include_directories(${SUBMODULE})
 
    # 添加子目录
    # 必须放在aux_source_directory前,否则同名变量SRCS会冲突
    add_subdirectory(${SUBMODULE})

    # 设置附加库变量
    set(EXTRA_LIBS ${EXTRA_LIBS} ${SUBMODULE})
endif (USESUBMODULE)

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. SRCS)

# 指定生成目标
add_executable(${PROJECT_NAME} ${SRCS})

# 添加链接库
target_link_libraries(${PROJECT_NAME} ${EXTRA_LIBS})
```
创建**config.h.in**文件并输入：
```CMake
#cmakedefine USEMYPRINT
```
***option***命令可以设置自定义编译选项，***configure_file***命令用于设置配置文件。生成目标工程时，目标目录会生成一个**config.h**文件。**config.h**中，如果***USESUBMODULE***选项被打开就为：
```C++
#define USESUBMODULE
```
否则是：
```C++
/* #undef USESUBMODULE */
```
**main.c**也需要调整：
```C++
//main.c
#include <stdio.h>

#ifdef USESUBMODULE
#include "myfun.h"
#endif//USESUBMODULE

int main()
{
    printf("hello, cmake!\n");
#ifdef USESUBMODULE
    printf("1+1=%d\n", myfun(1, 1));
#endif//USESUBMODULE
    return 0;
}
```
**CMake**打开选项的命令：
```command
cmake -D[宏名]=[宏值]
```
所以，开关***USESUBMODULE***选项使用：
```command
cmake -S . -B ./build -DUSESUBMODULE=ON
cmake -S . -B ./build -DUSESUBMODULE=OFF
```
![cmake_build3.PNG](../../img/cmake_build3.PNG)

### 安装
**CMakeLists.txt**的***install***命令：
```CMake
# 安装目标文件
install(TARGETS [名称] DESTINATION [目录])
# 安装普通文件
install(FILES [名称] DESTINATION [目录])
```
主模块和子模块的**CMakeLists.txt**分别添加对应的***install***命令：
```CMake
# 指定安装路径
install(TARGETS ${PROJECT_NAME} DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/config.h" DESTINATION include)
```
```CMake
# 指定库的安装路径
install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(FILES ${PROJECT_NAME}.h DESTINATION include)
```
最后执行：
```command
cmake -S ./ -B ./build
cmake --build ./build --config Release
cmake --install ./build --config Release --prefix ./install
```
***cmake --install ./build***命令就是执行 **./build**目录中项目的安装，***--prefix ./install***参数表示安装到 **./install**目录。
![cmake_build4.PNG](../../img/cmake_build4.PNG)