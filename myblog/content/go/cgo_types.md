---
title: "CGO类型转换"
date: 2020-04-11T12:09:14+08:00
draft: true
description : ""
categories:
  - "go"

tags:
  - "go"
  - "cgo"

menu : side # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params

thumbnail: "img/cgo.jpg" # Thumbnail image
lead: "" # Lead text
comments: true # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---

由于CGO和C的数据类型不是完全等价匹配, 所有在使用CGO的过程中需要做类型转换。

<!--more-->

### 代码仓库
https://github.com/gongluck/CGO-DEMO.git

### _cgo_export.h
使用 ***go tool cgo .\hello.go*** 命令可以生成 ***cgo*** 的中间文件,其中 ***_cgo_export.h*** 声明了导出函数和C与GO直接类型的关系。

### C-CGO-GO类型映射
| C语言类型               | CGO类型     | Go语言类型 |
|------------------------|-------------|-----------|
| char                   | C.char      | byte      |
| singed char            | C.schar     | int8      |
| unsigned char          | C.uchar     | uint8     |
| short                  | C.short     | int16     |
| unsigned short         | C.ushort    | uint16    |
| int                    | C.int       | int32     |
| unsigned int           | C.uint      | uint32    |
| long                   | C.long      | int32     |
| unsigned long          | C.ulong     | uint32    |
| long long int          | C.longlong  | int64     |
| unsigned long long int | C.ulonglong | uint64    |
| float                  | C.float     | float32   |
| double                 | C.double    | float64   |
| size_t                 | C.size_t    | uint      |
| int8_t                 | C.int8_t    | int8      |
| uint8_t                | C.uint8_t   | uint8     |
| int16_t                | C.int16_t   | int16     |
| uint16_t               | C.uint16_t  | uint16    |
| int32_t                | C.int32_t   | int32     |
| uint32_t               | C.uint32_t  | uint32    |
| int64_t                | C.int64_t   | int64     |
| uint64_t               | C.uint64_t  | uint64    |

### GO字符串和切片
Go语言的字符串、切片、字典、接口和管道等特有的数据类型生成对应的C语言类型：
        
    typedef struct { const char *p; ptrdiff_t n; } GoString;
    typedef struct { void *data; GoInt len; GoInt cap; } GoSlice;
***GoString*** 类型会对 ***_cgo_export.h*** 头文件产生依赖,可以用 ***_GoString_*** 代替。以下两个函数用于获取字符串结构中的长度和指针信息:
    
    size_t _GoStringLen(_GoString_ s);
    const char *_GoStringPtr(_GoString_ s);

### GO访问C的结构体、联合、枚举类型
    package main

    /*
    #include <stdint.h>

    struct A{
	    int i;
	    float f;
	    int type;
	    int   size:10;	// The bit field cannot be accessed
	    float arr[];	// Zero-length arrays are also inaccessible
    };

    union B1 {
	    int i;
	    float f;
    };
    union B2 {
	    int8_t i8;
	    int64_t i64;
    };

    enum C {
    	ONE,
	    TWO,
    };
    */
    import "C"

    import (
	    "fmt"
	    "unsafe"
    )

    func main() {
    	var a C.struct_A
        fmt.Println(a.i)
	    fmt.Println(a.f)
	    fmt.Println(a._type)

	    var b1 C.union_B1
	    fmt.Printf("%T\n", b1) // [4]uint8
	    var b2 C.union_B2
	    fmt.Printf("%T\n", b2) // [8]uint8
	    fmt.Println("b1.i:", *(*C.int)(unsafe.Pointer(&b1)))
	    fmt.Println("b1.f:", *(*C.float)(unsafe.Pointer(&b1)))

	    var c C.enum_C = C.TWO
	    fmt.Println(c)
	    fmt.Println(C.ONE)
	    fmt.Println(C.TWO)
    }
通过 ***C.struct_xxx*** 来访问C语言中定义的 ***struct xxx*** 结构体类型。结构体的内存布局按照C语言的通用对齐规则,在32位Go语言环境C语言结构体也按照32位对齐规则,在64位Go语言环境按照64位的对齐规则。对于指定了特殊对齐规则的结构体,无法在CGO中访问。如果结构体的成员名字中碰巧是Go语言的关键字,可以通过在成员名开头添加下划线来访问。

C语言结构体中位字段对应的成员无法在Go语言中访问,如果需要操作位字段成员,需要通过在C语言中定义辅助函数来完成。对应零长数组的成员,无法在Go语言中直接访问数组的元素,但其中零长的数组成员所在位置的偏移量依然可以通过 ***unsafe.Offsetof(a.arr)*** 来访问。

在C语言中,我们无法直接访问Go语言定义的结构体类型。

通过 ***C.union_xxx*** 来访问C语言中定义的 ***union xxx*** 类型。但是Go语言中并不支持C语言联合类型,它们会被转为对应大小的字节数组。可以使用unsafe包强制转型为对应类型(这是性能最好的方式)访问联合类型成员;对于复杂的联合类型,推荐通过在C语言中定义辅助函数的方式处理。

可以通过 ***C.enum_xxx*** 来访问C语言中定义的 ***enum xxx*** 结构体类型。在C语言中,枚举类型底层对应int类型,支持负数类型的值。我们可以通过 ***C.ONE*** 、***C.TWO*** 等直接访问定义的枚举值。

### GO访问C的字符串、数组
根据在reflect包中有字符串和切片的定义:

    type StringHeader struct {
     Data uintptr
     Len  int
    }

    type SliceHeader struct {
        Data uintptr
        Len  int
        Cap  int
    }
不单独分配内存,可以在Go语言中直接访问C语言的内存空间。因为Go语言的字符串是只读的,用户需要自己保证Go字符串在使用期间,底层对应的C字符串内容不会发生变化、内存不会被提前释放掉。

    package main

    /*
    #include <string.h>
    char arr[10];
    char *s = "Hello";
    */
    import "C"
    import (
    	"fmt"
    	"reflect"
    	"unsafe"
    )

    func main() {
    	// C array -> Go slice
	    var arr0 []byte
	    var arr0Hdr = (*reflect.SliceHeader)(unsafe.Pointer(&arr0))
	    arr0Hdr.Data = uintptr(unsafe.Pointer(&C.arr[0]))
    	arr0Hdr.Len = 10
    	arr0Hdr.Cap = 10

	    arr1 := (*[31]byte)(unsafe.Pointer(&C.arr[0]))[:10:10]
	    fmt.Println(arr1)

	    // C string -> Go string
	    var s0 string
	    var s0Hdr = (*reflect.StringHeader)(unsafe.Pointer(&s0))
	    s0Hdr.Data = uintptr(unsafe.Pointer(C.s))
	    s0Hdr.Len = int(C.strlen(C.s))

	    sLen := int(C.strlen(C.s))
	    s1 := string((*[31]byte)(unsafe.Pointer(C.s))[:sLen:sLen])
	    fmt.Println(s1)
    }
在C语言中可以通过 ***GoString*** 和 ***GoSlice*** 来访问Go语言的字符串和切片。但由于GC的存在,可能会潜藏着危险。

### GO指针和数值转换
为了实现X类型指针到Y类型指针的转换,需要借助 ***unsafe.Pointer*** 作为中间桥接类型实现不同类型指针之间的转换。***unsafe.Pointer*** 指针类型类似C语言中的 ***void\**** 类型的指针。
任何类型的指针都可以通过强制转换为 ***unsafe.Pointer*** 指针类型去掉原有的类型信息,然后再重新赋予新的指针类型而达到指针间的转换的目的。

Go语言针对 ***unsafe.Pointr*** 指针类型特别定义了一个 ***uintptr*** 类型。可以 ***uintptr*** 为中介,实现数值类型到unsafe.Pointr指针类型到转换。