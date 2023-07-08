---
title: 使用 CDN 时，如何保护源站 IP
tags: []
id: '172'
categories:
  - - uncategorized
date: 2023-03-02 21:19:44
---

> _warning_ 2023/04/01 + 05/04更新  
> **勘误 Host 部分 + 增加其他泄露方式。Cloudflare 可以修改回源端口。**

很多人可能会想，等我套上了 CDN，那我的 IP 就不会被泄露，站就打不死了。

事实确实如此，因为之前就有报道说 Cloudflare 为免费套餐的用户抵御了有史以来最大的 DDOS 攻击\[1\]，但当你自己配置时，可能发现你的源站还是会被打死。

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-21.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-21.png)

阿里云封禁通知

这也是一些攻击者用来“击穿CDN”的方法，我们来看一看原因是什么。

[![](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-22.png)](https://blog.imwcr.cn/wp-content/uploads/2022/10/image-22.png)

一些击穿 CDN 的攻击者

## 未使用虚拟主机

如果说你没有使用虚拟主机，如 NGINX 的配置中 server 没有指定或者是通配符，那么直接用一些工具（如网络测绘工具）搜索网站特征信息（类似标题，meta标签等），就可以直接找到源站 IP。

## HTTP 协议中 Host 漏洞

如果你使用过 宝塔 面板，就会知道虚拟主机这个概念，可以让一台服务器运行多个网站。而识别网站具体是哪一个，则仅仅是靠 URL 中的地址，即 NGINX 在检查这个地址之后就会返回网站源码。

但是，在请求头中有一个 Host 参数，这个参数是服务器端用来识别当前访问的网站具体是什么网站。（虚拟主机实现的原理）

```
Host: <host>:<port>
```

**而这里也就是漏洞出现的地方。**

比如我自己的网站 imwcr.cn 配置了 CDN，但是如果你伪造一个域名，Host 写成 imwcr.cn，url 写源站 IP，发送请求后， NGINX 就会傻乎乎的返回网站源码。

通过这个漏洞，我们就可以找到源站 IP 了，方法很简单，只要向全球 IP 发送带有 目标站点 域名 的 GET 请求（或 HEAD 请求），等一个状态码为 200 OK 就找到了。

![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image.png)

这种方法十分简单，通常几分钟就可以扫描完成。同时如果知道源站服务器的一些信息，还可以缩小 IP 范围，加快扫描速度。

## 其他可能的泄露方式

在添加 CDN 之前，源站 IP 就被设置成 DNS 解析过。

又如 HTTPS 证书泄露，即直接访问一个 IP 的时候，如果没有正确配置 NGINX 的话，可能 HTTPS 证书会被错误的发送给浏览器。

## 保护源站 IP

### 可设置回源 Host 的 CDN

对于一些可以设置回源 Host 头的 CDN，只需要进 CDN 后台修改为非真正域名，例如改成 www.youtube.com，然后：

*   在 宝塔 面板中增加绑定 www.youtube.com 这个域名；
*   或者在 NGINX 配置中的 server\_name 改成 www.youtube.com 即可。

如果想要更加安全，还可以将默认回源端口进行修改。因为一般攻击者扫描的端口都是 80， 也就是 HTTP 协议的默认端口；有时候可能会扫描 443，即 HTTPS 协议默认端口，但进行TLS握手还要额外的消耗，所以主要还是 80 端口。

### Cloudflare

因为免费版 Cloudflare 不可以自定义 回源 Host，回源端口也不支持修改，所以只能在服务端想办法了。

（5.4 发现 Cloudflare 支持修改回源端口，在 Rules 中的 Origin Rules 可以找到相关设置）

而 NGINX 有 allow/deny 的模式，可以通过 IP 来区分请求是否来自 Cloudflare （可能还是有风险，不清楚 CF 的 Worker 和 Warp 是否使用这些 IP）

Cloudflare 的 IP 可以在官网找到：[https://www.cloudflare.com/ips](https://www.cloudflare.com/ips)

在 本地 新建一个 cloudflare.conf 配置文件：

```
allow 173.245.48.0/20;
allow 103.21.244.0/22;
allow 103.22.200.0/22;
allow 103.31.4.0/22;
allow 141.101.64.0/18;
allow 108.162.192.0/18;
allow 190.93.240.0/20;
allow 188.114.96.0/20;
allow 197.234.240.0/22;
allow 198.41.128.0/17;
allow 162.158.0.0/15;
allow 104.16.0.0/13;
allow 104.24.0.0/14;
allow 172.64.0.0/13;
allow 131.0.72.0/22;

allow 2400:cb00::/32;
allow 2606:4700::/32;
allow 2803:f800::/32;
allow 2405:b500::/32;
allow 2405:8100::/32;
allow 2a06:98c0::/29;
allow 2c0f:f248::/32;

deny all;
```

然后在 需要限制的网站配置文件 的 server block 中 引用此文件（宝塔面板的网站配置在 **/www/server/panel/vhost/nginx**）

```
include /www/server/panel/vhost/nginx/cloudflare.conf;
```

### 防止 HTTPS 证书泄露

可以开启 NGINX 的 ssl\_reject\_handshake 拒绝 SNI 字段为空的 TCP 请求，或者自签一个假的 HTTPS 证书配置到 NGINX 中。（似乎宝塔就是第二种方法，来防止 HTTPS 窜站）

\[1\] 不清楚为何，找不到 Cloudflare 官方对此事的报道。