---
title: 安装博客遇到的几个坑
tags: []
id: '13'
categories:
  - - 瞎鼓捣
date: 2021-02-18 16:40:02
cover: https://blog.imwcr.cn/wp-content/uploads/2021/02/Snipaste_2021-02-18_16-28-16.jpg
coverWidth: 1200
coverHeight: 600
---

啊啊啊~好大的坑

## 为什么要重新搭建博客

过两天就开学了，今天想着把博客搞一下。

原来的博客搭建在[旧服务器](http://149.129.83.207/)上，最开始没有套CDN，以为我这小网站，怎么可能被攻击呢。好家伙，结果真的就被攻击了，不过攻击的流量比较小，每次封了十个小时就解封了。

[![封禁短信通知](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-1024x686.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image.png)

封禁短信通知

[![三次攻击时间](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-1.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-1.png)

三次攻击时间

于是乎，我研究起了CDN，发现价格昂贵啊啊啊~（不是我一个高中生能够承担的价格。）这时，我看到了Cloudflare，但是呢，官网解析的服务器又太慢了，所以我选择了**自选ip**这个操作。这里就不详细赘述了，反正坑挺多的。

但好像有很多人都用这个来做fanqiang的加速，所以搞得我们这些小站长**嫖**起来不是很爽。不过这个速度对我们来说已经足够了。

但是呢我的**源站ip**已经暴露出来了，我总不可能套一个CDN，然后装作不知道吧。这不是掩耳盗铃吗？

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-5.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-5.png)

源站ip记录

如果别人还想攻击，可以去查一下我以前的ip，然后继续攻击，这个跟CDN就没有关系了。

所以我就忍痛又买了一个新的服务器。并且把网站全部搭建了一遍。不得不说，第二次大件没有第一次那么痛苦啦。

不得不说套上之后确实没有攻击了，不过如果是很大流量攻击，估计还是会出问题。

* * *

但是呢，**自选ip**就要隔一段时间自己去后台切换CDN的ip，所以我就想能不能自己弄个自动切换的。就在这时，一件事情发生了。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-6.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-6.png)

Cloudflare首页被GFW阻断

好家伙，看到这个我人都傻了。我都不敢相信。然后我就想是不是我应该换一个CDN？万一Cloudflare被搞了那不就没得搞手了。

这当正当我冥思苦想的时候，我又发现，他解封了。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-7.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-7.png)

Cloudflare首页封锁解除

这就很令人迷惑……

不过当我研究了实现原理之后，我还是决定放弃，毕竟太麻烦了。

实现原理大概就是通过lzc提供的工具[lzcCFNode](https://www.lzc256.com/go/lzcCFNode/)获取ip地址，然后通过阿里云后端提供的接口去更换ip地址。不过这个验证很麻烦。我就懒得搞。反正也就一周换一次，ip没多大问题。

## 几个坑

接下来说几个坑。刚才在创建博客的时候，发现突然就访问不了，并且弹出520错误。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-2-1024x698.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-2.png)

520错误

吓了我赶紧看一下我首页能不能正常加载。结果发现是没有问题的。然后我又打开后台去看日志。发现是444错误。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-3.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-3.png)

444错误

百度一下发现没有，这是自定义的错误代码。然后再仔细一看，发现居然是后台的防火墙拦截了。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-4.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-4.png)

防火墙提示

加入白名单后成功解决问题。

* * *

好家伙，我用wordpress写着写着文章，突然页面就崩溃了。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-8-1024x774.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-8.png)

页面崩溃

着实给我吓了一跳（不会吧，不会吧，我写了那么久的全没啦？？）然后一看下面那行代码我才心安。原来只是内存用光了，刷新下页面就行了，毕竟它有自动保存草稿的功能。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-9.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-9.png)

自动保存草稿

_就在我准备发布的是否，，又又崩溃了_

* * *

另外这个WordPress如果要修改固定链接，就一定要修改伪静态设置，要不然就加载不了文章页面。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-11-1024x489.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-11.png)

固定链接

[![](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-12.png)](https://blog.imwcr.cn/wp-content/uploads/2021/02/image-12.png)

宝塔伪静态设置

这里就放张宝塔伪静态的截图，自己搭建环境的dalao们肯定不需要了啦~

* * *

啊啊啊把post弄成page了 QAQ 只能手动修改

附修改方法：

1.  进入phpMyAdmin；
2.  进入WordPress对应的数据库；
3.  浏览wp\_posts数据表；
4.  找到相应的 页面Page 并编辑（找到相应的 文章Post 并编辑）；
5.  修改 post\_type 为 post（page）；
6.  修改 guid 将 ?p= 替换为 ?page\_id= （将 ?page\_id= 替换为 ?p=）；
7.  此时已经完成转换，编辑文章（页面）、更新，目的是刷新相关的设置选项；

* * *

其他好像没什么事要说的了

如果我不懒的话，还会再更新几篇文章的。