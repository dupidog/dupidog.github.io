---
layout: post
title: Solution to local login busybox dropbear
categories: [Geek]
tags: [linux]
---

## 解决busybox环境下dropbear不能使用本地用户登录的问题

简言之，读取系统用户信息的一系列函数，如`getpwnam`，`getpwuid`等，都依赖于`libnss_xxx`库，这些并不会静态链接进`dropbear`，从而在`dropbear`进行验证时，这些函数都会返回`NULL`，从而导致认证失败。

GNU Libc使用NSS来配置C库的行为，比如读取passwords和group。这是通过`/etc/nsswitch.conf`和`/lib/libnss_*`库来实现的。`busybox`避免使用libc的NSS，但是一些命令，如`login`和`su`还是使用libc的NSS。

`busybox`中默认启用`CONFIG_USE_BB_PWD_GRP`，这会使用内部的函数来访问`/etc/passwd`, `/etc/group`, 和`/etc/shadow`等文件，而不需要NSS。而其他需要调用`getpwnam`等函数的软件，就会因为`busybox`环境的NSS的缺失而无法使用，`dropbear`也是如此。

最简单的解决方法是重写这几个函数，可以参考`fuzz.h`和`fuzz_common.h`中的写法简单实现，使用固定的用户密码来启用`dropbear`。

![busybox-dropbear-1](/images/busybox-dropbear-1.png)

![busybox-dropbear-2](/images/busybox-dropbear-2.png)

---------------------------------------------

Ref:

[https://blog.csdn.net/force_eagle/article/details/4508978?locationNum=15](https://blog.csdn.net/force_eagle/article/details/4508978?locationNum=15)

[https://blog.csdn.net/hbcbgcx/article/details/94403845](https://blog.csdn.net/hbcbgcx/article/details/94403845)

[http://www.360doc.cn/mip/659312600.html](http://www.360doc.cn/mip/659312600.html)
