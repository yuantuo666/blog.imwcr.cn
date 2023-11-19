---
title: 使用PHP搭建一个HTTP代理服务器
tags: []
id: '232'
categories:
  - - VPS
  - - 后端
  - - 教程
date: 2023-03-06 19:25:47
---

> _warning_ 警告  
> **本方法搭建的代理服务器传输内容均为明文，且有被篡改风险，仅供学习参考。**

## 代理选择

涉及到网络服务，很难离开代理，HTTP就是一种常见代理。

虽然可以使用网络爬虫去网上爬取公开的代理，但这些免费代理通常速度比较慢，而且筛选起来又比较麻烦；如果你决定使用付费，你又会发现这个市场鱼龙混杂，比如我购买的150元/月的付费代理，不是说完全不能用，而是发的请求经常无缘无故的失败（10%左右出错率）；而再往上价格也就更加昂贵了，那也没有必要了。

## 没设密码的Squid

再加上其实我的需求并不需要大量的IP，所以想要利用下闲置的服务器来跑个代理服务。

之前为了图方便，直接找了个Squid跑起来就没管了，结果之后再使用服务器时发现巨卡，top查看下发现是代理占用的，才想起来之前没有设置账号密码，估计是被别人扫到之后拿去盗用了，所以导致出现了这种状况。当时就把端口给关了，Squid也关了。

## PHP搭建

现在又需要使用，所以想着搭建一个，突然发现Github上有这样的项目，直接clone下来就可以用了。

（Workerman牛逼）

0\. 安装好 PHP 环境，偷懒的话直接装宝塔然后安装吧

```
https://www.bt.cn/new/download.html#BT之前出过一些事，如服务器被入侵、服务器使用者被请喝茶等...感兴趣可以自己了解
```

1\. 按照README.md的操作安装：

```
git clone https://github.com/walkor/php-http-proxy && cd php-http-proxy && composer install
```

如果没有安装git和composer的话，先自行安装。

2\. 再运行：

```
php start.php start
```

如果跑不起来，估计是PHP.ini中禁用了一些函数，取消全部的禁用就可以跑起来了，如果你还有别的网站也要使用PHP，最好复制一份 PHP.ini 到当前目录然后再修改，启动的时候带上修改之后的PHP.ini地址启动：

```
php -c php.ini start.php start
```

```
宝塔默认PHP.ini地址 /www/server/php/80/etc/php.ini  #其中80改为实际使用的版本
```

3\. 默认端口是8080，如果服务器提供商有防火墙需要放行，如果安装了宝塔，在安全页面也需要放行端口。

[![](/images/2023/03/image.png)](/images/2023/03/image.png)

宝塔面板放行

4\. 放行之后就可以尝试连接了，我这里使用的是SwitchyOmega这个插件

先下载安装

```
Google 商店: https://chrome.google.com/webstore/detail/padekgcemlokbadohgkifijomclgjgif
Github Releases: https://github.com/FelisCatus/SwitchyOmega/releases
```

[![](/images/2023/03/image-4.png)](/images/2023/03/image-4.png)

添加代理服务器

然后进入设置页面添加代理服务器，设置IP和端口后保存应用，最后切换到设置的情景模式，就可以使用了。

[![](/images/2023/03/image-10.png)](/images/2023/03/image-10.png)

设置情境模式

[![](/images/2023/03/image-7.png)](/images/2023/03/image-7.png)

选择代理服务器

## 添加白名单功能

吸取之前的教训，专门为PHP的代理服务器添加了白名单功能，因为BT的防火墙一次只能放一个IP，所以这个工作就交给workerman来完成吧

```
// ....前面的省略了....

// Emitted when data received from client.
$worker->onMessage = function($connection, $buffer)
{
    // get user ip
    $ip = $connection->getRemoteIp();
    $white_list = [
        '1.1.1.',
        '2.2.2.',
        '3.3.3.',
        '127.0.0.1',
        '123.123.123.123'
    ];
    // 填写IP的前缀

    $is_white = false;
    foreach($white_list as $white_ip)
    {
        if(strpos($ip, $white_ip) === 0)
        {
            $is_white = true;
            break;
        }
    }
    // refuse non-white ip
    if(!$is_white)
    {
        echo "Refuse $ip\ns";
        $connection->close();
        return;
    }

// ....后面的也省略了....
```

因为校园网有多个出口IP，所以使用了判断前缀的方式，whatever，能用就行。

## 进程守护

最后防止程序寄掉，上个supervisord进程守护，如果不用守护的话，装个screen然后开个窗口跑PHP就行了。

[![](/images/2023/03/image-8.png)](/images/2023/03/image-8.png)

进程守护

先安装，然后添加守护进程。

[![](/images/2023/03/image-9.png)](/images/2023/03/image-9.png)

添加守护

```
名称: proxyxyxyx
启动用户: root
运行目录: /root/php-http-proxy/
启动命令: php /root/php-http-proxy/start.php start
如果上面的不行可以试试这个，修改80为安装的PHP版本: /www/server/php/80/bin/php -c /www/server/php/80/etc/php.ini /root/php-http-proxy/start.php start
```

确定之后程序就跑起来了，可以在这里查看日志。

```
/www/server/panel/plugin/supervisor/log/proxyxyxyx.out.log
```

修改代码之后不会立即生效，需要在进程守护这里重启，或者手动关闭workerman（仅reload无效）。