---
layout: post
title: Suspend disks for QNAP nas
categories: [Geek]
tags: [nas]
---

## 威联通（QNAP）硬盘不休眠（待机）的解决方法

此处感谢此帖
[威联通（QNAP）硬盘不休眠（待机）的解决方法](https://blog.csdn.net/hanziyuan08/article/details/104933718)
的作者，同时也在此作一下整理与备份。

### 背景

好奇观察了下 453Bmini 硬盘的休眠（待机）情况，发现在已有 SSD 作为系统和软件安装区域后，HDD 仍不进入休眠状态，或进入休眠短时间被唤醒。
我的 1 号硬盘位是SSD，所以以 2 号硬盘位的 HDD 举例：查看硬盘是否休眠可通过命令 hdparm -C /dev/sdb，如果输出 drive state is: active/idle 则表示硬盘是唤醒状态，如果输出 drive state is: standby 则表示硬盘已休眠（低功耗模式）。

### 调查

首先排查 NAS 系统中是否开启了硬盘待机，在“控制台”>“系统”>“硬件”中，我已经开启；
排查了[官方文档中描述的其他场景](https://www.qnap.com/zh-cn/how-to/faq/article/%E4%B8%BA%E4%BD%95%E6%88%91%E7%9A%84-nas-%E7%A1%AC%E7%9B%98%E4%B8%8D%E8%BF%9B%E5%85%A5%E5%BE%85%E6%9C%BA%E6%A8%A1%E5%BC%8F)，
我用了 Container Station，但是全部都是在 SSD 上，应当也不会阻止 HDD 休眠。
这就比较奇怪了，只能网上搜索一番，直到找到下面这两篇博客给我解了疑：

1. [在威联通NAS上完美实现硬盘单独休眠](http://www.nasyun.com/thread-67376-1-1.html)

2. [QNAP 威联通磁盘分区探索与数据导出](https://post.smzdm.com/p/301806/)

我将上面的博客内容总结一下，除系统盘之外的硬盘不能休眠，是因为 QNAP 选择将每一块硬盘都分出来两个区块，然后将这部分区块组成了 RAID1（下文称为“系统 RAID1”，路径为 /dev/md9 和 /dev/md13），也就是说只要系统有读写，那所有硬盘都将不能休眠。

那如何解决呢？很简单，将非系统盘的分区移出“系统 RAID1”即可，这里以我的第二块硬盘举例：

```shell
mdadm /dev/md9 --fail /dev/sdb1
mdadm /dev/md13 --fail /dev/sdb4
```

### 添加定时任务

这里我提供两个脚本，一个用于断开“系统 RAID1”，一个用于恢复“系统 RAID1”：

```shell
#!/bin/sh
# fail_raid1.sh（断开系统 RAID1）
mdadm /dev/md9 --fail /dev/sdb1
mdadm /dev/md13 --fail /dev/sdb4
```

```shell
#!/bin/sh
# readd_raid1.sh（恢复系统 RAID1）
mdadm /dev/md9 --re-add /dev/sdb1
mdadm /dev/md13 --re-add /dev/sdb4
```

需要注意的是，QTS（QNAP 的系统）不能通过 crontab -e 这个方式添加定时任务，因为系统内部有一套逻辑会覆盖通过这种方法增加的定时任务，详情可参考官方文档：

[Add_items_to_crontab](https://wiki.qnap.com/wiki/Add_items_to_crontab)

```shell
# 每天 0 点恢复系统 RAID1
echo "0 0 * * * /bin/bash /share/CACHEDEV1_DATA/my_cron/readd_raid1.sh" >> /etc/config/crontab
# 每天 0 点 15 分恢复系统 RAID1
echo "15 0 * * * /bin/bash /share/CACHEDEV1_DATA/my_cron/fail_raid1.sh" >> /etc/config/crontab
# 每 10 分钟检测 HDD 是否休眠，结果保存在 raid1_monitor.log 中
echo "*/10 * * * * /bin/bash /share/CACHEDEV1_DATA/my_cron/raid1_monitor.sh" >> /etc/config/crontab
# 重启 crontab
crontab /etc/config/crontab && /etc/init.d/crond.sh restart
```
