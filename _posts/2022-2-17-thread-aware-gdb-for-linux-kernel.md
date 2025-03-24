---
layout: post
title: Debugging the linux kernel using thread-aware of gdb
categories: [Geek]
tags: [linux, gdb]
---

## 使用gdb的thread-aware脚本来调试linux内核

### 背景

在开发版上bring up linux的工作，会遇到各种各样的问题，考验一个软件工程师的知识面和经验。这个时候，有多一种工具的辅助，就可能会简化这个工作。gdb可以通过jtag之类的调试接口与soc的调试模块相连，读写soc系统的各寄存器与内存地址上的数据。Thread-aware就是一个脚本，可以使用调试口来获取linux中的进程信息，这在console不可使用的时候会显得对调试很有帮助。

### Gdb的thread-aware脚本

Thread-aware需要gdb的python脚本支持，x86/arm架构在较早的gdb版本中已支持，riscv架构在Gdb 9.1开始支持。

要使用thread-ware，需要在linux源码的cfg中打开GDB_SCRIPTS，编译后得到的linux就可以执行以下命令加载osawareness脚本

![thread-aware-1](/images/thread-aware-1.png)

### 使用thread-aware命令

后续就可以使用lx命令来显示task相关内容

![thread-aware-2](/images/thread-aware-2.png)

例如：

![thread-aware-3](/images/thread-aware-3.png)

![thread-aware-4](/images/thread-aware-4.png)

![thread-aware-5](/images/thread-aware-5.png)

有些命令和函数在riscv下没有实现,

如 lx_current

![thread-aware-6](/images/thread-aware-6.png)

-----------------------------------------------

Ref:

[https://stackoverflow.com/questions/9561546/thread-aware-gdb-for-the-linux-kernel](https://stackoverflow.com/questions/9561546/thread-aware-gdb-for-the-linux-kernel)

[https://wiki.st.com/stm32mpu/wiki/Debugging_the_Linux_kernel_using_the_GDB](https://wiki.st.com/stm32mpu/wiki/Debugging_the_Linux_kernel_using_the_GDB)

[https://www.kernel.org/doc/html/v4.15/dev-tools/gdb-kernel-debugging.html](https://www.kernel.org/doc/html/v4.15/dev-tools/gdb-kernel-debugging.html)
