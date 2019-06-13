---
layout: post
title: Use ssh protocol to connect git remote repository via proxy
---

## Git使用ssh跨越代理连接远程仓库

由于上网环境，必须使用代理来连接外网。
传统的设置代理的方式往支持http和https协议，全局代理的设置方法如下：

```
git config --global http.proxy=[prot://]<server>:<port>
git config --global https.proxy=[prot://]<server>:<port>
```

或者在仓库中使用如下命令配置只适用于本仓库的代理：

```
git config http.proxy=[prot://]<server>:<port>
git config https.proxy=[prot://]<server>:<port>
```

但每次同步都需要输入密码，比较麻烦，还是希望能用ssh的方式，可以使用证书来认证，免去输密码的麻烦。
由于git使用ssh协议时调用了系统的ssh命令，所以需要配置ssh_config文件，可以指定需要使用代理的网站、用户、端口，以及使用的私钥，以下是使用代理服务器192.168.0.100:1080访问github的例子。

```
Host github.com
    HostName github.com
    User git
    Port 22
    ProxyCommand nc -X 5 -x 192.168.0.100:1080 %h %p
    IdentityFile ~/.ssh/id_rsa
```

加入ssh_config后就可以直接访问目标服务器了。
