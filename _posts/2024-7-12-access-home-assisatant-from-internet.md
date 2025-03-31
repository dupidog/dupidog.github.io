---
layout: post
title: Access home assistant from internet
categories: [Geek]
tags: [homeassistant]
---

# 让Home Assistant支持外网访问

---

## 1. 背景

`Home Assistant`是一个布署于家庭局域网的智能家居中枢，可以在家庭网中使用其管理页面`http://<ip>:<port>`来登录。

但`Home Assistant`的网页端和手机客户端默认是不支持外网访问的，就算配置了`NAT`端口转发，还是使用`CDN`方式（如`Cloudflare Tunnel`）做反向代理到公网，都是无法从公网连接。究其原因是`Home Assistant`会检查是否从内网地址访问，如果不是，那么会强制要求`https`方式连接并且会检查`TLS`证书是否有效。

## 2. 解决方法

### 2.1. 购买Home Assistant Cloud

在手机客户端或网页端连接页面处注册`Nabu Casa`帐号并订阅`Home Assistant Cloud`（第一个月免费），订阅后会得到了一个外网`URL`，用它填入连接设置中的外网`URL`，即可连接成功。

`Home Assistant Cloud`的本质是给你一个带`TLS`证书的`https`反向代理地址，零配置就可以使用，但收费。

### 2.2. 使用自己的域名和TLS证书

使用自己的域名指向`Home Assistant`的`NAT`外网地址或`CDN`地址，购买此域名的`TLS`证书，将证书路径放入`configuration.yaml`的`http`配置段中，如下：

```
http:
    server_port: 8123
    ssl_certificate: /ssl/fullchain.pem
    ssl_key: /ssl/privkey.pem

```

然后使用证书对应的域名访问即可。

此方式需要一定的配置工作，需要自己申请证书，比较适合自己的域名已经有证书的情况。

### 2.3. 使用X-Forwarded-For扩展头部
	
此方式最简单，适合使用`CDN`反向代理来加速网站的情况，先说如何做：

#### a) 配置configuration.yaml

在`configuration.yaml`中的`http`配置段加入`use_x_forwarded_for: true`以及`CDN`反向代理的连接地址范围

```
http:
	use_x_forwarded_for: true
	trusted_proxies:
	    - 10.0.3.0/24
```

#### b) 配置域名使用CDN加速

配置`Tunnel`指向`Home Assistant`的内网访问地址，域名配置使用`CDN`加速方式访问`Tunnel`。

#### c) 使用域名访问

网页端使用`https://<domain-name>:<port>`访问。

手机客户端在外部`URL`处填入`https://<domain-name>:<port>`，确认连接。

![](/images/home-assistant-internet.jpg)

## 总结

`X-Forwarded-For`是一个`HTTP`扩展头部。`HTTP/1.1`（`RFC 2616`）协议并没有对它的定义，它最开始是由`Squid`这个缓存代理软件引入，用来表示`HTTP`请求端真实`IP`。
由于`Cloudflare`等`CDN`的`Tunnel`可以开启客户端`https`证书，这时候就可以用它来通过`Home Assistant`的检查了。


