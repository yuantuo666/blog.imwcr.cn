---
title: 如何抓包获取百度网盘网页版完整 Cookie
tags:
  - 技术
  - 服务器
  - 百度网盘
id: '181'
categories:
  - - 教程
date: 2022-11-24 13:43:21
cover: /images/2022/11/image-7.png
coverWidth: 1200
coverHeight: 600
---

1\. 使用 Edge 浏览器打开已经登录的百度网盘网页版

[![](/images/2022/11/image.png)](/images/2022/11/image.png)

2\. 按下 F12 打开开发者工具

或者 对 **百度网盘logo** 右键，选择 **检查**

[![](/images/2022/11/image-1.png)](/images/2022/11/image-1.png)

3\. 选择 网络 选项卡

[![](/images/2022/11/image-2.png)](/images/2022/11/image-2.png)

4\. 刷新页面

5\. 找到一个 请求 URL 为 https://pan.baidu.com 开头的请求，一般第一个就可以

[![](/images/2022/11/image-3.png)](/images/2022/11/image-3.png)

6\. 在请求的详细信息页面往下滑找到 Cookie 参数

[![](/images/2022/11/image-4.png)](/images/2022/11/image-4.png)

7\. 对 Cookie 右键复制值

[![](/images/2022/11/image-5.png)](/images/2022/11/image-5.png)

8\. 粘贴到安装页面

[![](/images/2022/11/image-6.png)](/images/2022/11/image-6.png)