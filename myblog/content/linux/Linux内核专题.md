---
title: "Linux内核专题"
date: 2020-12-28T22:22:43+08:00
draft: false

featured_image: "img/linux_wh.jpg"
categories: linux
tags: [linux]
---

## 十一、Linux内核专题

### 1.OpenResty

#### 1.1 简介

- **openresty**是一个基于**nginx**与**lua**的高性能**web**平台，其内部集成了大量精良的**lua**库、第三方模块以及大多数的依赖项。用于方便搭建能够处理超高并发、扩展性极高的动态**web**应用、**web**服务和动态网关。
- **openresty**通过汇聚各种设计精良的**nginx**模块，从而将**nginx**有效地变成一个强大的通用 **Web**应用平台。这样，**Web**开发人员和系统工程师可以使用**Lua**脚本语言调动**Nginx**支持的各种**C**以及**Lua**模块，快速构造出足以胜任**10K**乃至**1000K**以上单机并发连接的高性能**Web**应用系统。
- **openresty**的目标是让**Web**服务直接跑在**Nginx**服务内部，充分利用**Nginx**的非阻塞**I/O**模型，不仅仅对**HTTP**客户端请求，甚至于对远程后端诸如**MySQL、PostgreSQL、Memcached**以及**Redis**等都进行一致的高性能响应。

#### 1.2 lua-nginx-module

![lua-nginx-module](https://github.com/gongluck/CVIP/blob/master/images/lua-nginx-module.png?raw=true)

#### 1.3 cosocket

![cosocket](https://github.com/gongluck/CVIP/blob/master/images/cosocket.png?raw=true)

#### 1.4 环境搭建

```Shell
apt-get install libpcre3-dev libssl-dev perl make build-essential curl -y
wget https://openresty.org/download/openresty-1.19.3.1.tar.gz
tar -xzvf openresty-1.19.3.1.tar.gz
cd openresty-1.19.3.1/
./configure
make -j 8
sudo make install
```

### 2.Linux内核编程

#### 2.1 Linux内核编译升级

```Shell
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.tar.xz
tar -xvf linux-5.10.tar.xz
cd linux-5.10
cp /boot/config-xxx ./.config
make menuconfig
make -j 8
sudo su
make modules_install
make bzImage

cp arch/x86/boot/bzImage /boot/vmlinuz-4.4.16
cp .config /boot/config-4.4.16
cd /lib/modules/4.4.16/
update-initramfs –c –k 4.4.16
update-grub
```

#### 2.2 linux整体架构与子系统划分

![linux整体架构与子系统划分](https://github.com/gongluck/CVIP/blob/master/images/linux整体架构与子系统划分.png?raw=true)
