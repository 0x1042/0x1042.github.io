
- [GDB 使用](#gdb-使用)
  - [背景](#背景)
  - [GDB dump线程栈](#gdb-dump线程栈)
  - [找到负载高的线程](#找到负载高的线程)
  - [寻找对应的行号](#寻找对应的行号)


## 背景

> 线上流量没变，但是出现瞬时负载呈直线上涨，1～2分钟整个服务不可用。从火焰图看，top sample 集中在一个函数之内，但是无法精确到函数的行号。

## GDB dump线程栈

```shell

# gdb.cmd 
set logging file thread.txt
set logging on
thread apply all bt
quit
```

然后执行,会把所有的线程栈打印到文件中  

```shell
gdb --command=gdb.cmd --pid 939
```


## 找到负载高的线程

```shell
# 按照cpu排序，找到对应的线程id  比如959
top -H -p 939 
```

在thread stack文件中找到对应的线程id的栈

```shell
572 Thread 2 (Thread 0x7ff946b91640 (LWP 959) "ib_io_ibuf"):
573 #0  syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38
574 #1  0x00007ff957218279 in io_getevents () from /lib/x86_64-linux-gnu/libaio.so.1
575 #2  0x00005606d5644ccb in LinuxAIOHandler::collect() ()
576 #3  0x00005606d5645526 in LinuxAIOHandler::poll(fil_node_t**, void**, IORequest*) ()
577 #4  0x00005606d5648f68 in os_aio_handler(unsigned long, fil_node_t**, void**, IORequest*) ()
578 #5  0x00005606d57fd32d in fil_aio_wait(unsigned long) ()
579 #6  0x00005606d564238a in ?? ()
580 #7  0x00005606d5642479 in ?? ()
581 #8  0x00007ff9570b8253 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
582 #9  0x00007ff956d41ac3 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:442
583 #10 0x00007ff956dd3a40 in clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:81
```

## 寻找对应的行号

```shell
addr2line 0x00005606d4b388a0 -e exec_binary
```

**对于静态链接程序有效** 