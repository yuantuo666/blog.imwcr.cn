---
title: Win10下端口被占用的解决方法
tags: []
id: '138'
categories:
  - - 教程
date: 2022-06-29 20:31:53
---

## 起因

今天在调试 aria2c 的时候，突然就用不了了

我就感觉很奇怪，也没有干什么啊，为什么之前能用，现在突然就用不了了

用终端执行报错如下

```
06/29 19:56:17 [ERROR] IPv4 RPC: failed to bind TCP port 6800
Exception: [SocketCore.cc:312] errorCode=1 Failed to bind a socket, cause: An attempt was made to access a socket in a way forbidden by its access permissions.

06/29 19:56:17 [ERROR] IPv6 RPC: failed to bind TCP port 6800
Exception: [SocketCore.cc:312] errorCode=1 Failed to bind a socket, cause: An attempt was made to access a socket in a way forbidden by its access permissions.

06/29 19:56:17 [ERROR] Exception caught
Exception: [DownloadEngineFactory.cc:219] errorCode=1 Failed to setup RPC server.
```

（这里的提示的原因是 “An attempt was made to access a socket in a way forbidden by its access permissions.” 而并非 “Address already in use.” 这一点我也没注意到）

## 寻找解决方法

基本上网上提供的解决方法就是查看占用ip的进程，然后结束进程

找的各种方法都不管用

1.  netstat -ano 查看所有端口，结果没发现有占用 6800 的
2.  关闭杀软，关闭防火墙，依旧没用
3.  重启电脑，依旧没用

然而，很奇怪的是 同样使用aria2的 Motrix 却可以正常使用（端口为16800）

* * *

经过很长一段时间的摸索，在无意间看到的 **Hyper-V** 在我脑海中留下了印象

打开“更改适配器选项”时，映入眼前的一幕让我傻了眼

[![](/images/2022/06/Snipaste_2022-06-29_20-10-05.jpg)](/images/2022/06/Snipaste_2022-06-29_20-10-05.jpg)

更改适配器选项

## 最终解决方法

然后我用关键词在必应上搜了一下，终于找到了解决方案

先说方法

1.  netsh int ipv4 show dynamicport tcp 查看端口范围是否异常，如果启动端口是1000+，说明很有可能是端口被 Hyper-V 随机保留
2.  用管理员身份打开 cmd 或 终端
3.  执行下面的命令更改起始端口
    *   netsh int ipv4 setdynamic tcp start=49152 num=16384
    *   netsh int ipv6 setdynamic tcp start=49152 num=16384
4.  重启电脑
5.  netsh int ipv4 show dynamicport tcp 再次查看端口范围是否异常

原因就是 Windows10 的 **Hyper-V** 在开机时随机保留端口的时候，保留了你想要使用的，导致很多软件不能使用

因为这个问题导致受影响的软件包括但不限于以下：

*   aria2
*   aria2c
*   Motrix
*   Minecraft
*   V2Ray
*   Clash

有时候能用可能仅仅是因为恰好这个端口没有被保留

[更多参考信息](https://zhaoji.wang/solve-the-problem-of-windows-10-ports-being-randomly-reserved-occupied-by-hyper-v/)