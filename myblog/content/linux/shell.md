---
title: "Shell基础"
date: 2020-10-17T16:33:15+08:00
draft: false

featured_image: "img/linux_wh.jpg"
categories: linux
tags: [linux]
---

### 使用Shell

**test.sh需要执行权限**

```shell
/bin/bash ./test.sh
#或者
./test.sh
```

### 简单Shell文件

```shell
#!/bin/bash
#第一句指明使用哪一个解析程序

#输出字符串
echo "hello, shell!"

#定义变量，=号之间不能有空格
tmp="test"
#输出变量，$紧跟变量名取变量值
echo $tmp

#循环输出文件名
for file in $(ls $pwd); do
    echo $file
done

#循环加法
sum=0
for i in {1..100}; do
    let sum+=i  #let 指明后面的是数字运算
done
echo $sum

#网络探寻
for i in {1..254}; do
    ping -c 2 -i 0.1 192.168.3.$i &> /dev/null
    if [ $? -eq 0 ]; then   # $?可以获取上一个指令的执行结果
        echo 192.168.3.$i is up
    else
        echo 192.168.3.$i is down
    fi
done
```