---
layout: post
title: Rescue a "dead" R6220 by nmrpflash
---

## 网件R6220救砖

手贱在刷好openwrt的R6220的web升级页面刷了X-WRT，于是成砖。正好手上没有TTL线又值清明小长假，想着只能回公司用TTL线看下是什么情况了吗？
想了一下还是不甘心，想用nmrpflash试一下，试了很多种方法才成功。

此处感谢此帖
[网件R6220救砖](https://blog.51cto.com/gaokui/2447612)
的作者，同时也在此作一下整理与备份。

### 一、出现情况：

1.  路由器上电后，只有电源灯和交换机(有线网口)灯亮，其余灯都灭。

2.  按路由器上的恢复按钮上电，出现2灯、3灯交替闪烁，但无法ping通192.168.1.1。

3.  使用nmrpflash.exe这个软件刷机，一直提示

```
No response after 60 seconds. Bailing out
```

### 二、分析原因及救砖前提：

按恢复按键上电能出现2灯3灯闪烁，说明原厂的uboot应该没损坏。至于为什么没正常启动X-WRT，没有TTL线也不好分析，可能是R6220版本与固件不匹配的原因导致启动失败。

救砖前提是uboot没有损坏，也必须是原厂的。如果刷过breed等不死uboot，那自然也是有办法刷回正常固件的，也没必要使用本方法。

### 三、解决办法：

1. 将本地网卡IP修改为`10.x.x.x`，子网掩码为`255.0.0.0`。

2. 下载`nmrpflash.exe`和适用于你当前windows版本的winpcap软件，先安装好winpcap。

3. 用管理员权限`cmd`进入`nmrpflash.exe`所在文件夹。

4. 确认与R6220连接的网卡，如果不确认，用`nmrpflash.exe -L`这条命令看一下是哪一个设备
```
D:\>nmrpflash.exe -L
net0   10.183.10.23     54:42:49:73:5d:c8  (以太网)
net1   0.0.0.0          2a:dd:08:fc:4d:f4  (本地连接* 3)
net2   192.168.1.42     78:dd:08:fc:4d:f4  (WLAN)
net3   0.0.0.0          1a:dd:08:fc:4d:f4  (本地连接* 2)
```

5. 输入命令
```
nmrpflash.exe -i <device> -f <flash_image_file>
```
`<device>`为与R6220连接的网卡设备，`<flash_image_file>`为factory刷机包，测试可以为官方升级包或openwrt的factory.img。例如：
```
nmrpflash.exe -i net0 -f R6220_v1.1.0.50.img
```
当出现如下打印时，说明nmrpflash正在等待路由器响应：
```
Advertising NMRP server on net0 ... /
```

6. 此时给路由器上电，nmrpflash检测到路由器，打印如下：
```
Received configuration request from A0:63:91:11:00:98
Sending configuration: 10.164.183.252 netmask 255.0.0.0
Received upload request with empty filename
Uploading iR6220_v1.1.0.50.img ... OK
Waiting for remote to respond.
```

7. 然后一直出现Received keep-alive request.保持连接，直到提示Remote finished，表示成功刷写完毕。
```
Received keep-alive request.
Received keep-alive request.
Received keep-alive request.
Remote finished. Closing connection.
Reboot your device now.
```

8. 重启，将本地网卡修改为自动获取，至此成功救砖。
