---
layout: post
title: Rescue a "dead" R6220 by nmrpflash.exe
---

## 网件R6220救砖

手贱在刷好openwrt的R6220的web升级页面刷了X-WRT，于是成砖。
正好手上没有TTL线又值清明小长假，想着只能回公司用TTL线看下是什么情况了吗？
想了一下还是不甘心，想用nmrpflash试一下，试了很多种方法才成功，此处感谢此帖的作者
https://blog.51cto.com/gaokui/2447612
同时也在此作一下整理与备份

# 一、出现情况：
1、 路由器开机后电源出现重复启动；
2、 按路由器上的恢复按钮，出现2灯、3灯交替闪烁，无法ping通192.168.1.1
3、 使用nmrpflash.exe这个软件刷机，一直提示“No response after 60 seconds. Bailing out”

# 二、解决办法：
1、将本地网卡IP修改为10.x.x.x，子网掩码为255.0.0.0；
2、下载nmrpflash.exe软件；
3、用管理员权限cmd进入nmrpflash.exe所在文件夹；
4、确认与R6220连接的网卡，如果不确认，用nmrpflash.exe -L这条命令看一下是哪一个设备
5、输入命令nmrpflash.exe -i <device> -f <flash_image_file>这条命令，如：
nmrpflash.exe -i net0 -f R6220_v1.1.0.50.img，出现
~~~
Advertising NMRP server on net0 ... /
~~~
6、此时给路由器上电，nmrpflash检测到路由器，会显示如下：
![netgear-detect](https://github.com/dupidog/dupidog.github.io/blob/master/images/netgear-nmrpflash-detect.jpg?raw=true)
7、然后一直出现Received keep-alive request.保持连接，直到提示Remote finished，表示成功刷写完毕。
Received keep-alive request.
Received keep-alive request.
Received keep-alive request.
Remote finished. Closing connection.
Reboot your device now.
~~~
7、重启，将本地网卡修改为自动获取。