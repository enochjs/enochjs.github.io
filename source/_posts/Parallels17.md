---
title: Parallels17
date: 2022-05-03 16:15:49
tags: 软件
---

### 解决 Parallels Desktop 17 无法连接网络问题

- 修改一下两个配置文件
- /Library/Preferences/Parallels/dispatcher.desktop.xml
  - 修改 dispatcher.desktop.xml，找到 \<Usb>0\</Usb>，修改为 \<Usb>1\</Usb> 并保存。
- /Library/Preferences/Parallels/network.desktop.xml
  - 修改 network.desktop.xml，找到 \<UseKextless>1\</UseKextless> 或 \<UseKextless>-1\</UseKextless>，修改为 \<UseKextless>0\</UseKextless> 并保存

### Parallels Desktop 17.1.1 操作失败 执行该操作失败 解决方案

- 选择 操作 -> 配置 -> 高级 -> 虚拟机监控程序 -> Parallels，重启即可

### 相关链接

[无法连接网络问题](https://www.jianshu.com/p/4a5fbe7d698f)
[执行该操作失败](https://zhuanlan.zhihu.com/p/454602605)
