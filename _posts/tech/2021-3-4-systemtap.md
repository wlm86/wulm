---
layout: post
title: systemtap
category: VIRTUAL
tags: VIRTUAL
description:  systemtap
---

### 1.**注意事项**

1. 代码注意事项 

   1) "#", // , /**/都可以

   2) 如果多个probe需要共用变量，需要通过global声明全局变量。
      例：global list[400] 声明了一个数组，可以存储400个元素

   4) 通过foreach遍历元素列表
      例1：foreach ([a,b] in foo+ limit 5) { }   升序列出top5
      例2：foreach ([a,b] in foo- limit 5) { }   降序列出top5
      例3：foreach ([a-,b] in foo) { }  根据第一个元素降序排列

   5) @min, @max, @count, @avg, @sum可以对单一的值进行聚合操作。
      此外可以用@hist_log 或者 @hist_linear来提取数据流的直方图。
      用这两额函数处理的结果目前只允许进行打印。

   6) probe timer.ms(10000) 用来计时

   7) a <<< 1 相当于 a += 1

   8) entry
    ```
    #!/usr/bin/stap
    
    global io_latency
    
    probe vfs.write.return {
       time = gettimeofday_ns() - @entry(gettimeofday_ns())
       io_latency <<< time
    }
    
    probe end {
       print (@hist_log(io_latency))                                                                                                                                       
    }
    ```

2. 目标代码内核变量 

   以美元符号$开头的标识符表示内核代码中的变量

   1) "$$vars"：  目的是打印probe函数处的每个变量
    ```
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$vars); exit(); }'
    iocb=0xffff8f603bbd3e18 iov=0xffff8f603bbd3e08 nr_segs=0x1 pos=0x0 filp=? retval=? seg=? count=0xffff8f603e229b00 ppos=?/n
    ```
   2) $$locals：$$vars子集，仅打印local变量
    ```
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$locals); exit(); }'
    filp=? retval=? seg=? count=0xffff8f5eedbaaa00 ppos=?/n
    ```
   3) $$parms： $$vars子集，仅包含函数参数
    ```
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$parms); exit(); }'
    iocb=0xffff8f603bbcbe18 iov=0xffff8f603bbcbe08 nr_segs=0x1 pos=0x0/n
    ```
   4) $$return：返回值，类似printf("return=%x",$return)，如果没有返回值，则是空串
    ```
    # stap -e 'probe kernel.function("generic_file_aio_read").return {printf("%ld\n", $return); exit(); }'
    57771
    下面的写法会报错
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%ld\n", $$return); exit(); }'
    semantic error: while processing probe kernel.function("generic_file_aio_read@mm/filemap.c:2040") from: kernel.function("generic_file_aio_read")
    semantic error: unable to find local '$return', [man error::dwarf] dieoffset 0x1c3503e in kernel, near pc 0xffffffff811b7ef0 in generic_file_aio_read mm/filemap.c (alternatives: $retval, $count, $seg, $iov, $pos, $filp, $iocb, $nr_segs, $ppos)): identifier '$$return' at <input>:1:65
            source: probe kernel.function("generic_file_aio_read") {printf("%ld\n", $$return); exit(); }
    
    ```
   5) $$parms$  可以展开结构体中的数据，如果想进一步展开可以用$$parms$$
    ```
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$parms); exit(); }'
    iocb=0xffff8f503d437e18 iov=0xffff8f503d437e08 nr_segs=0x1 pos=0x0
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$parms$); exit(); }'
    iocb={.ki_users={...}, .ki_filp=0xffff8f5ec3e9a200, .ki_ctx=0x0, .ki_cancel=0x0, .ki_dtor=0x0, .ki_obj={...}, .ki_user_data=0, .ki_pos=0, .private=0x0, .ki_opcode=0, .ki_nbytes=58283, .ki_buf=          (null), .ki_left=58283, .ki_inline_vec={...}, .ki_iovec=0x0, .ki_nr_segs=0, .ki_cur_seg=0, .ki_list={...}, .ki_eventfd=0x0} iov={.iov_base=0xc420936000, .iov_len=58283} nr_segs=1 pos=0
    # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$parms$$); exit(); }'
    iocb={.ki_users={.counter=1}, .ki_filp=0xffff8f503b766100, .ki_ctx=0x0, .ki_cancel=0x0, .ki_dtor=0x0, .ki_obj={.user=0xffff8f5038341040, .tsk=0xffff8f5038341040}, .ki_user_data=0, .ki_pos=0, .private=0x0, .ki_opcode=0, .ki_nbytes=58283, .ki_buf=          (null), .ki_left=58283, .ki_inline_vec={.iov_base=0x0, .iov_len=0}, .ki_iovec=0x0, .ki_nr_segs=0, .ki_cur_seg=0, .ki_list={.next=0x0, .prev=0x0}, .ki_eventfd=0x0} iov={.iov_base=0xc420936000, .iov_len=58283} nr_segs=1 pos=0
   ``` 

### 2.**使用**
1. 常用命令
    1) stap -e 直接加脚本语句 
        # stap -e 'probe kernel.function("generic_file_aio_read") {printf("%s\n", $$parms); exit(); }'
    2) 通过staprun直接运行内核模块，需要将stp脚本编译成内核模块
        staprun test.ko
    3) stap -L 查看内核目标函数的参数

         ```
         # stap -L 'kernel.function("vfs_read")'
         kernel.function("vfs_read@fs/read_write.c:450") $file:struct file* $buf:char* $count:size_t $pos:loff_t*
         # stap -L 'kernel.function("find_get_page")'
         kernel.function("find_get_page@mm/filemap.c:1029")

         ```
   

### 3.**ref**
  https://sourceware.org/systemtap/man/stapprobes.3stap.html
  https://www.xuebuyuan.com/3204236.html 
