---
layout: post
title: segfault探究
category: VIRTUAL
tags: VIRTUAL
description:  segfault探究
---
#   segfault探究

有时候会在/var/log/message 中看到 kernel qemu-kvm: segfault***。
所以来探究segfault。

### 1.**segfault at x ip xxx sp xxx error x**

1. error $x
   错误码由3个数字组成，从高到底分别为bit2 bit1和bit0,所以它的取值范围是0~7。
    1) bit2: 值为1表示是用户态程序内存访问越界，值为0表示是内核态程序内存访问越界 
    2) bit1: 值为1表示是写操作导致内存访问越界，值为0表示是读操作导致内存访问越界 
    3) bit0: 值为1表示没有足够的权限访问非法地址的内容，值为0表示访问的非法地址根本没有对应的页面，也就是无效地址。
    error_code:
    *      bit 0 == 0 means no page found, 1 means protection fault
    *      bit 1 == 0 means read, 1 means write
    *      bit 2 == 0 means kernel, 1 means user-mode
    *      bit 3 == 0 means data, 1 means instruction

   例如错误码是6,转换成二进制就是110，这3位对应的含义如下：
   用户态些内存越界，访问的是无效的地址。
   
### 2.**生成core文件**
    ```
    [root@host10573617 ~]# ulimit -c
    0
    [root@host10573617 ~]# ulimit -c unlimited
    [root@host10573617 ~]# ulimit -c
    unlimited
    ```
### 3.**gdb 调试core文件**

    # gdb -c core  core-file
