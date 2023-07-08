---
title: VScode Remote-SSH 在MacOS上启用代理
tags:
  - Linux
  - 服务器
  - 网路
id: '329'
categories:
  - - VPS
  - - Web
  - - 网络
date: 2023-06-17 17:05:31
---

Remote-SSH是VScode的一个插件，可以方便的连接服务器，并实现远程代码编写。

但是当服务器不在同一个网络环境下时，连接可能经常不稳定或者各种原因连接不上，导致写着写着就断开连接要求重新刷新窗口。

Step 1: 安装转发工具

[https://nmap.org/ncat/](https://nmap.org/ncat/)

Windows直接下载Portal版本就可以来，下文的ncat替换成ncat.exe的绝对路径。

Step 2: 设置配置

```
Host Test
    HostName 8.8.8.8
    Port 22
    User root
    ProxyCommand ncat --proxy-type http --proxy 127.0.0.1:7890 %h %p
```

Step 3: 启动你的小猫和Vscode~

![](https://blog.imwcr.cn/wp-content/uploads/2023/06/截屏2023-06-17-16.54.18.png)