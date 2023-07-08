---
title: MySQL 大小写敏感的坑
tags:
  - bug
  - MySQL
  - 技术
id: '204'
categories:
  - - Web
  - - 后端
date: 2023-01-15 23:15:58
cover: https://blog.imwcr.cn/wp-content/uploads/2023/01/caspar-camille-rubin-fPkvU7RDmCo-unsplash-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

## 遇到的坑

大小写的问题经常被忽略，但这次我在项目中遇到的问题就和大小写有关。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/01/image.png)](https://blog.imwcr.cn/wp-content/uploads/2023/01/image.png)


如上图所示，username的字符集被设置成了utf8\_general\_ci，即Unicode大小写不敏感。这也是MySQL默认的设置，但用户名作为区分用户的重要参数，大小写不敏感自然会出现很多意外的bug。

比如这次就是因为这个问题导致对该用户的操作未生效，用户注册时使用的是全大写，而登录时使用的是全小写，因为大小写不敏感，导致检验时通过。

但对用户的操作是通过Redis实现，而该操作是区分大小写的，因此发现了此bug。

如果此bug被利用，将导致对用户操作记录等失效，权限管理等失效。

## bug修复

将表的字符集（排序规则）修改为utf8-bin即可区分大小写。

但原先重名导致的错误数据却需要人工修复，这就取决于数据量的多少和用户的习惯了。

（理想中可以写个脚本修复，但项目中遇到的情况是部分数据丢失了，那也就只能手动修复了）

## 用户名一定要区分大小写吗？

（2023/02/02 更新）

这个问题其实没有标准答案，从两个方面来进行分析：

1.  如果不区分大小写，那么对于用户来说就更加方便，他们在登录时就无需考虑大小写问题（尤其是iPhone自带键盘输入首字母自动大写）
2.  如果区分大小写，在后端处理数据时，用户名这一个参数就可以用来区分唯一用户，出现的bug更少。

### 我们来看看 Google 是怎么做的

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image.png)

Google 登录：自动转换

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-1.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-1.png)


在输入正确的邮箱后，输入密码时，原来大写的邮箱会被自动转换成小写，当然，对于邮箱来说是这样，因为邮箱本身就不区分大小写。（如果有网站的数据库区分了邮箱的大小写，那理论上，通过修改大小写组合，一个邮箱可以注册很多次）

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-2.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-2.png)

_其实Gmail同时还会忽略.+等字符，也就是你的邮箱中间可以加点或者加号等，感兴趣的可以搜索Google别名邮箱_

### 那么 Github 呢

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-4.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-4.png)

可以看到，当尝试使用大写去注册时，会提示不可用。也就是说，当你注册一个名称之后，这个名称所有的大小写组合也被你占用了，这其实也很符合认知，因为正常的用户也不会刻意去区分用户名的大小写。

### 大小写问题总结

总而言之，对用户名大小写最好的处理就是不区分。在注册写入数据库时保留大小写数据，但在登录操作时不计较大小写的正确性，并且在验证通过之后，之后所有的处理都使用正确的大小写（即存储在数据库中的用户名）。

当然，让人头大的还有全角字符和各种奇奇怪怪的 Unicode 组合字符。（在邮件地址中不被允许）

eg. ｙｕａｎｔｕｏ６６６＠ｇｍａｉｌ．ｃｏｍ

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-6.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-6.png)

不一样的字符

[![](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-5.png)](https://blog.imwcr.cn/wp-content/uploads/2023/02/image-5.png)

全角字符

So, 最好注册时加上白名单机制，当然如果可以保证数据不出错的话，那奇奇怪怪的用户名也是可以接受的~