---
layout: post
title: Local user can't be used to login in busybox dropbear
categories: [Geek]
tags: [linux]
---

## busybox环境下dropbear不能使用本地用户登录

简言之，读取系统用户信息的一系列函数，如getpwnam，getpwuid等，都依赖于libnss_xxx库，这些并不会静态链接进dropbear，从而在dropbear进行验证时，这些函数都会返回NULL，从而导致认证失败。

GNU Libc使用NSS来配置C库的行为，比如读取passwords和group。这是通过/etc/nsswitch.conf 和/lib/libnss_* 库来实现的。busybox避免使用libc的NSS，但是一些命令，如login和su还是使用libc的NSS。

busybox中默认启用CONFIG_USE_BB_PWD_GRP，这会使用内部的函数来访问/etc/passwd, /etc/group, and /etc/shadow 等文件，而不需要nss。而其他需要调用getpwnam等函数的软件，就会因为busybox环境的nss的缺失而无法使用，dropbear也是如此。

最简单的解决方法是重写这几个函数，可以参考fuzz.h和fuzz_common.h中的写法简单实现，使用固定的用户密码来启用dropbear。

![busybox-dropbear-1](/images/busybox-dropbear-1.png)

![busybox-dropbear-2](/images/busybox-dropbear-2.png)

---------------------------------------------

Ref:

[https://blog.csdn.net/force_eagle/article/details/4508978?locationNum=15](https://blog.csdn.net/force_eagle/article/details/4508978?locationNum=15)

[https://blog.csdn.net/hbcbgcx/article/details/94403845](https://blog.csdn.net/hbcbgcx/article/details/94403845)

[http://www.360doc.cn/mip/659312600.html](http://www.360doc.cn/mip/659312600.html)
