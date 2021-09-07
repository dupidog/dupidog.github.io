---
layout: post
title: The solution to L2TP VPN connection failure in Win10
categories: [Geek]
tags: [vpn]
---

## Win10 中 L2TP-VPN 不能连接的解决方法

### 错误现象

使用 `Win10` 内置的 `VPN`  客户端连接 `L2TP` 类型的 `VPN` 时连接超时，报如下错误：

>无法建立计算机与VPN服务器之间的网络连接，因为远程服务器未响应。
这可能是因为未将计算机与远程服务器之间的某种网络设备（如防火墙、NAT、路由器等）配置为允许VPN连接。
请与管理员或服务提供商联系以确定哪种设备可能产生此问题。

### 解决方法

网上搜索了一下，很多帖子讲这个问题，需要改两处注册表（其实是新增两个键值）。
推测是 `Win10` 默认的配置禁止了 `VPN` 拨号器进行 `L2TP` 拨号时需要使用的 `IPSec`。
修改方法如下：

1. `Win+R` 键打开运行框或在搜索框中输入 `regedit` 打开注册表编辑器；

2. 注册表编辑器中找到 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\Parameters` 将 `ProhibitIPSec` 的值改为 `0` (如没有则新增此键，类型为 `DWORD`）；

3. 注册表编辑器中找到 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent` 将 `AssumeUDPEncapsulationContextOnSendRule` 的值改为 `2` (如没有则新增此键，类型为 `DWORD`）；

4. 重启计算机。

这样就能使用 `L2TP` 类型的 `VPN` 了。
