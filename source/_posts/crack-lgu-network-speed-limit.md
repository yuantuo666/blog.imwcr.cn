---
title: “解除” LGU 校园网限速
tags:
  - LGU
  - Linux
  - 多播
  - 技术
  - 虚拟机
  - 软路由
  - 限速
id: '142'
categories:
  - - 瞎鼓捣
date: 2022-10-04 15:29:00
cover: https://blog.imwcr.cn/wp-content/uploads/2022/10/frederik-lipfert-cWtsPbJtIvs-unsplash-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

## 声明

本文不涉及任何违法操作，原理为近似于使用多台电脑同时下载。

## 起因

某位同学需要下载几个数据集，其中之一就有19GB，但苦于校园网限速3MB/s，不知道要下到猴年马月，于是乎我就想办法把校园网限速这一问题解决。

看看人家北京邮电大学，人人分到至少100M的宽带，实际下载速度更是能达到15MB/s，快的网速，谁不爱呢

## 认证原理

想要解除限速，首先得要弄清楚校园网是这么认证的。

通过阅读ITSO的手册可以得知，有 **通过网页输入账号密码认证** 和 **通过802.1X身份验证** 两种方式。

好巧不巧，在询问 如何让小米台灯联网 时，工作人员给了我一个管理认证的网站，进入后就可以通过绑定 MAC 地址实现认证，而认证列表里正好有我的电脑的 MAC 地址。

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-17.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-17.png)

至此，LGU校园网的认证方法就很清楚了，即通过绑定 MAC 地址来进行设备验证。

## 限速原理

通过手机和电脑同时跑网速测试，发现两个设备都能达到 3MB/s 的速度，自然说明了学校限速是通过 MAC 地址 或 IP地址来进行限速，想要解除限速，自然就是多整几个 MAC 地址 和 IP地址 来叠加网速，以此达到解除限速的目的。

## 具体实现

实现多个虚拟网卡的最简单方式就是通过软路由。因为不清楚可行性，所以我采用了虚拟机的方式来安装软路由。

准备好以下文件：

1.  [下载 VMware Workstation Pro CN](https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html)
2.  [固件下载-爱快 iKuai-商业场景网络解决方案提供商 (ikuai8.com)](https://www.ikuai8.com/component/download) （32位 ISO）

然后我们就可以开始啦~

### 安装虚拟机

此处不过多赘述，安装激活后应该看到如下画面。

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image.png)

### 创建虚拟机

*   创建新的虚拟机
*   选择 典型（推荐）
*   安装程序光盘映像文件 选择 刚才下载的 ISO 文件

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-1.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-1.png)

*   操作系统 选择 Linux，版本选择 Linux 3.x版本

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-2.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-2.png)

*   一路下一步到这里，选择自定义硬件

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-3.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-3.png)

*   内存可以改为1GB，网络适配器一定要改为VMnet1

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-4.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-4.png)

*   接下来点击右下角添加一个网络适配器，因为软路由需要一个入网网卡，一个出网网卡

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-5.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-5.png)

*   网络适配器2选择桥接模式

![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-6.png)

*   关闭配置界面后，点击完成，完成虚拟机创建

### 系统安装

*   点击开启此虚拟机后等待加载
*   约1min后进入到安装界面，输入1并回车，输入y并回车，进行安装（使用数字键的话，记得打开Num Lock）

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-8.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-8.png)

*   3s后安装完成，系统自动重启

### 系统配置

*   进入后输入2回车，输入0回车，修改默认lan地址，我这里修改成了192.168.99.1，防止和路由器管理页面冲突

![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-9.png)

![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-10.png)

*   接下来需要修改本地网络适配器选项，在设置→网络和 Internet→高级网络设置→更多网络适配器选项 中，按下图操作（DNS根据具体需求填写，我这里设置的实际是我们学校的DNS服务器，因为学校的部分认证服务涉及到DNS）

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-11.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-11.png)

*   修改完成后，就可以通过 **http://192.168.99.1/** （取决于你设置的lan1地址）来访问管理面板，默认账号密码都是admin

*   接下来进入内外网设置，点击空闲网口绑定wan口

[![](https://www.ikuai8.com/attached/php/upload/image/20180201/1517457549540794.png)](https://www.ikuai8.com/attached/php/upload/image/20180201/1517457549540794.png)

*   接着进入wan1设置

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-12.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-12.png)

*   添加虚拟网卡并记下 MAC 地址

![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-13.png)

*   再按照下图开启负载均衡

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-15.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-15.png)

*   接着 添加MAC地址到认证服务器

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-14.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-14.png)

*   最后一步，将本地的正常上网的网卡关闭IPv4协议，如此，请求才会通过配置好的软路由进行转发

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-16.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-16.png)

## 享受千兆光纤

下午限速时测速

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-20.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-20.png)


夜间限速放宽时测速

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-19.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-19.png)

