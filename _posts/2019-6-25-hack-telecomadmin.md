---
layout: post
title: Telecomadmin hack for telecom fiber modem E8-C
categories: [Geek]
tags: [hack, network]
---

## 电信光猫破解telecomadmin密码

是个老技术帖，此处做一个备份。后面`E8-C`升级后，`http`方式不能下载到`backpresettings.conf`，需要用`ftp`方式来下载，方法如下：

-------------------------

1. 在浏览器上输入`http://192.168.1.1/logoffaccount.html`，设置隐藏用户改为启用，这样就可以用工程账号登陆了；

2. 登录工程帐号（用户名：`fiberhomehg2x0` 密码：`hg2x0`），登录网址`http://192.168.1.1/` （如果这个你走不了，请联系电信处理吧，别乱弄的网都上不了。）

3. 工程模式中，开启`FTP`功能（自己设置一个用户名和密码，默认都是`admin`）；

4. 出厂设置-升级`image`-【更新 -- 预配置生成】，点击[生成预配置]；

5. 登录`FTP`，地址`ftp://192.168.1.1/fhconf/` ，用户名和密码是3中你设置的，复制其中的`backpresettings.conf`到本地；

6. 使用记事本打开，后中字符串极为密码，需使用`base64`解密，在这里解密`http://tool.chinaz.com/Tools/Base64.aspx`，即为密码，用户名是`telecomadmin`；

7. 使用`telecomadmin`和密码登录`http://192.168.1.1/`，即可管理猫大部分功能。
