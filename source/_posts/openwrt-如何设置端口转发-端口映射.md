---
title: OpenWrt 如何设置端口转发/端口映射
tags:
  - 技术
  - 服务器
  - 网路
id: '254'
categories:
  - - 瞎鼓捣
  - - 网络
date: 2023-03-20 21:46:10
cover: https://blog.imwcr.cn/wp-content/uploads/2023/03/lars-kienle-IlxX7xnbRF8-unsplash-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

## 小伙伴的问题

群里有个小伙伴问了这样一个问题

> 我们宿舍光猫改的桥接，用ax3路由器拨号，nas和r68s软路由插在ax3上，我的电脑和x3派插在r68s上，现在我在x3派上搭建了alist，[192.168.100.101:5244](http://192.168.100.101:5244)，怎么在手机连ax3的情况下访问

这个问题恰好我知道一点点，所以就回答了下，但他那边操作没有用，就顺便写个文章记录下。

先画个示意图，

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-11.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-11.png)

伪网络拓扑图

**因为r68s的限制，所以在ax3局域网的手机实际上是访问不到在r68s局域网的x3的，要解决这个问题也很简单，就是对r68s操作，修改它的配置，让它开放一个端口，并将这个端口映射到x3上。**

局域网可以想象成一个封闭的房间，一般来说，在同一个局域网的设备可以自由的通信，例如r68s和手机间。

但上图中的r68s阻隔了手机和x3的通信，因为r68s下的设备在另一个局域网，也就是在原来房间里的一个更小的房间里。

那电脑是怎么上网的呢，其实r68s也充当了一个门的角色，他允许局域网内的设备向外发起请求，但阻止了外面的设备向内发起请求，也可以说是一个防火墙。（其实真实网络环境中，存在这样大量套娃的“防火墙”，只能单向向外发起请求；其实猫的外面也有很多这种套娃，但这种设计最初的设计是为了缓解IPv4地址的枯竭，但也起到了防护内部设备的功能，具体来说涉及到了NAT技术，这里就不展开了）

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/d47c32afe1b5d859c7c33d63c502940.jpg)](https://blog.imwcr.cn/wp-content/uploads/2023/03/d47c32afe1b5d859c7c33d63c502940.jpg)

## r68s 设置

咋设置呢，先开放防火墙，再设置端口转发即可。

### 开放防火墙

1\. 访问 http://192.168.100.1/ 登录

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-12.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-12.png)


2\. 打开 网络 - 防火墙

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-13.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-13.png)


3\. 修改防火墙允许“入站数据”和“转发”，wan即为r68s对外的网口，修改为接受，则房子外的设备可以访问。

![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-14.png)

4\. 滑到页面底部 保存并应用

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-15.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-15.png)

这样，防火墙就开放啦。

### 设置端口转发/端口映射

1\. 进入 网络 - 防火墙 - 端口转发

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-16.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-16.png)


2\. 按图新建端口转发，注意内部IP地址记得**选对设备**，还要点击**添加按钮！！**

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-17.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-17.png)

3\. 保存并应用

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-18.png)](https://blog.imwcr.cn/wp-content/uploads/2023/03/image-18.png)

这样，就设置好啦~

### 体验使用

[![](https://blog.imwcr.cn/wp-content/uploads/2023/03/551456c51c523e738241eb314151e90.jpg)](https://blog.imwcr.cn/wp-content/uploads/2023/03/551456c51c523e738241eb314151e90.jpg)

现在访问 **http://\[r68s在手机局域网下的IP\]:5244/** 就可以访问到 x3 上部署的 alist 啦

（注意是**手机所在局域网**下的r68s的IP地址，不是在子局域网的IP地址）

当然，如果可以设置的话，你还可以尝试将猫（ax3）的端口也给映射，这样在校园网环境下也可以上网了。