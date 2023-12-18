---
title: PowerToys 改键冲突导致的 bug：Microsoft Teams 弹窗
tags:
  - Windows
  - 技术
categories:
  - - note
date: 2023-12-19 00:40:00
cover: /images/2023/12/surface-30f2UKzCMC0-unsplash.jpg
coverWidth: 1920
coverHeight: 1080
---

# TL;DR

因为在 Windows 11 的系统里同时使用了 [PowerToys](https://github.com/microsoft/PowerToys) 和 [utools](https://u.tools/)，导致了快捷键监听的 bug，会导致 Microsoft Teams 总是弹出。

解决方法是在 PowerToys 里直接把导致 bug 的快捷键修改掉。

顺便简单吐槽了下 Microsoft Teams 和 飞书 的使用体验。

# 问题

utools 是一个工具合集，可以通过设置快捷键来快速启动插件，例如快速启动应用、翻译、base64 编解码之类的，同时还可以支持自己编写脚本。

今年双十一下单了个 HHKB 的键盘，虽然它支持在 Windows 和 MacOS 之间切换，但是需要打开背面的盖子扳动 DIP 开关来切换，比较麻烦。

而且它的 Control 键的位置其实和 MacBook 会更加像，我也经常需要在两个系统上切换使用它，所以我决定把它设置为 MacOS 的模式，并且在 Windows 里使用 PowerToys 来改键达到一样的体验，map 的键位如下：

```
Remap keys:
Win -> Ctrl
Ctrl -> Win
```

现在就可以通过按下键盘上的 Command 键来等效于 Windows 的 Ctrl 键，完成一些快捷键操作，这个操作在 MacBook 上也是一样的键位，所以体验上还行。

但是在按下鼠标中键呼起 utools 时，我发现会导致 Microsoft Teams 的提示消息弹出，如下图所示。

![Microsoft Teams 弹框](/images/2023/12/Snipaste_2023-12-18_23-09-04.jpg)

因为我也经常需要使用 utools 这个中键功能，所以这个 bug 有点影响体验的。

# 解决

## 观察问题是什么

发现的问题是当选中文本并按下鼠标中键时，会导致 Microsoft Teams 的提示消息弹出。

我尝试修改了激活 utools 的快捷键，发现这个问题还是存在的，说明并不是鼠标中键的问题。同时，如果没有选中文本就按下鼠标中键，也不会出现这个问题。

接下来我围绕着 `Microsoft Teams` 和 `弹窗` 来进行排查。

## 在网上搜寻解决方法

用这上面提到的两个关键词进行搜索，[发现也有人在报告这个 bug](https://answers.microsoft.com/zh-hans/msteams/forum/all/win11%E4%B8%AD-microsoft/2963b826-e7c9-481b-8cb4-597437db67fa)，说明这个影响的不只是我一个人。

看到里面提到的是使用了 PowerToys 之后导致的，我就关闭了 PowerToys，发现这个 bug 确实就没有了。

我一个个的关闭了 PowerToys 里的设置，最后发现是在 `Keyboard Manager` 里的 `Remap keys` 设置导致的。

![Keyboard Manager](/images/2023/12/Snipaste_2023-12-18_23-21-39.jpg)

这里定位到了问题，但是我还是不理解为什么会出现这种情况，也没有很好的解决方法。

我也前往了 PowerToys 的 GitHub 仓库搜索相关的 issue，但是可能这个问题太细化了，所以并没有找到相关的信息。但至少确定了这个和改键是有关系的。

## 尝试在 Windows 设置里屏蔽这个弹窗

因为这个弹出的位置在左下角，我就检查了 Windows 的菜单栏设置，发现有一个隐藏 `聊天` 的设置，之前就被我关闭了，打开之后发现弹出的窗口其实就是对应这个 `聊天` 图标的。

![Win11 设置](/images/2023/12/Snipaste_2023-12-18_23-25-33.jpg)

所以这里就定位了这个弹窗是来自 Win11 系统的 `聊天`，这也可以解释为什么我尝试使用 火绒 的弹窗拦截不能捕获到这个窗口。

在继续搜索如何屏蔽 `聊天` 功能后，我尝试了修改策略组，但是发现并没有用。但是我发现了唤起这个窗口的快捷键是 `Win + C`，加上之前改键的问题，我就理解这个 bug 出现的原因了。

## bug 的原因

当选中文本并激活 utools 时，utools 会执行复制的操作，也就是 `Ctrl + C` 快捷键。但是这个快捷键被 `PowerToys` 拦截了，并且修改成了 `Win + C`，这个快捷键会唤起 Win11 的 `聊天` 窗口，所以就出现了这个 bug。

这也可以解释之前出现的情况：
- 尝试屏蔽 `聊天` 功能，但是并没有用，因为这个快捷键是系统级别的，所以无法屏蔽
- 关闭 `PowerToys` 之后，这个 bug 就没有了，因为这个快捷键是被 `PowerToys` 拦截修改了

所以这个 bug 是因为使用了 `PowerToys` 改键，并且和 `utools` 的执行的复制操作产生了冲突，导致了这个 bug。

## 解决方法

既然是因为被修改成了 `Win + C` 快捷键导致的，那么就直接在 `PowerToys` 里继续修改这个组合键为复制的快捷键 `Ctrl + C` 即可。

```
Remap shortcuts:
Win + C -> Ctrl + C
```

![修改快捷键](/images/2023/12/Snipaste_2023-12-18_23-37-13.jpg)


# 总结

重新复盘一下：
- 通过观察问题是什么，发现是选中文本并按下鼠标中键时，会导致 Microsoft Teams 的提示消息弹出
- 在网上搜寻解决方法，发现也有人在报告这个 bug，继续搜索发现和 `Win + C` 快捷键有关
- 尝试在 Windows 设置里屏蔽这个弹窗，但是并没有用
- bug 的原因是因为使用了 `PowerToys` 改键，并且和 `utools` 的执行的复制操作产生了冲突，导致了这个 bug
- 解决方法：在 `PowerToys` 里修改 `Win + C` 为 `Ctrl + C`

这个 bug 的排查过程有点点复杂，修改一些设置的时候见到了很多老版本 Windows 的窗口，不由的感叹向前兼容和屎山代码的威力，不知道微软员工修改代码的时候是不是也到处是坑……

## 顺便吐槽

这里顺便吐槽一句，微软做的这个 `聊天` 功能在国内真的有人在用吗……

学校因为用的是微软的一件套，邮箱使用的是 `Outlook` 体验还行，但在学校科研上的沟通使用的是 `Microsoft Teams (work or school)` 体验就很糟糕了。

对 `Teams` 我感觉唯一的好处就是：所有学生和教职员都在系统里，不需要加好友，就可以向学校内其他人发起聊天。

但是其他的功能被飞书吊打，飞书支持网页版，发送的文本支持富文本，还有非常好用的飞书文档（PS：实验室也是用的这个来做信息共享；之前[影视飓风的 TIM 也有提到过飞书相当不错](https://www.bilibili.com/video/BV1NR4y1G7TM/?t=350)，深有同感）。

反观 `Teams`，聊天框按 Ctrl + Enter 都是直接发送，没有富文本，还经常会出现图片加载不出来，消息/表情发不出去，服务器断连，会议无法投屏等问题。虽然它也有网页版，但是整体体验上和飞书差距还是比较大的。

# 参考资料

- https://www.theverge.com/22686302/windows-11-microsoft-teams-chat-how-to-uninstall
- https://www.jb51.net/os/win11/894869.html
- https://answers.microsoft.com/zh-hans/msteams/forum/all/win11%e4%b8%ad-microsoft/2963b826-e7c9-481b-8cb4-597437db67fa