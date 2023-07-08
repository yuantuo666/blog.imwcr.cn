---
title: 修复 Blackboard 愚蠢的滚动条
tags:
  - 技术
id: '273'
categories:
  - - JavaScript
  - - Web
date: 2023-04-22 15:05:49
cover: https://blog.imwcr.cn/wp-content/uploads/2023/04/Snipaste_2023-04-22_15-04-59.png
coverWidth: 1200
coverHeight: 600
---

## Updates 页面的问题

学校用的是 Blackboard 平台分享教学资料，然后这个系统有个 Updates 功能，可以显示最近的新消息。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-1.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-1.png)

这个功能是挺好的，但是进入这个页面后，页面的滚动条就令人非常难受。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-2.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-2.png)

乍一看感觉没啥大问题，但你如果用鼠标滚轮滚动一下，就会发现这个滚动条是跳跃的。就是说滚动一下会直接到很下面。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-3.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-3.png)

就是说，滚动一下，你就看到的完全就是新的内容。

## 问题修复

这里可以用 Tampermonkey 油猴脚本 来实现。

先放上完成的脚本链接。

安装地址： [h](https://github.com/yuantuo666/Stupid_BB/raw/main/main.user.js)[ttps://github.com/yuantuo666/Stupid\_BB/raw/main/main.user.js](https://github.com/yuantuo666/Stupid_BB/raw/main/main.user.js)

（如果你们学校的 Blackboard 也是长这样，可能我们用的是一个系统，你可以试着修改脚本中的匹配网址 @match 后面的成你们学校的Blackboard 理论上就可以运行了）

脚本干的事就两件，第一件干掉滚动监听事件，第二件增加滚动条。

### 干掉滚动监听事件

Edge浏览器右键检查就可以看到当前这个 iframe 框架中页面的监听事件，这个 mousewheel 就是要干掉的目标。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-4.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-4.png)

手动点击删除后发现滚动确实失效了，说明就是这个事件处理的。

为了干掉这个事件，开始尝试了直接调用 EventTarget.removeEventListener() ，然后发现必须要传入绑定的函数才可以删除，而绑定的函数在加载的 JavaScript 里，自然就获取不到，这个方法就行不通了。

之后又发现了有 getEventListeners() 这个方法可以获取到对应的事件绑定的函数，测试确实可以删除，但是这个方法只在 Chromium 内核的浏览器控制台中有用，在油猴脚本/加载到页面中的 JavaScript脚本 中是不生效的，想要获取到对应函数再删除的方法也不行了。

![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-5.png)

之后发现可以通过在页面执行之前修改原型链劫持 addEventListener() 即阻止 Blackboard 的 JavaScript 脚本添加事件监听，这种方法确实可行，但 油猴脚本的加载时机 导致 劫持代码 不一定会在 添加监听 前执行，也就是说劫持不一定成功，这种方式最后也没有被使用。

```
// 劫持方法
let oldAdd = EventTarget.prototype.addEventListener
EventTarget.prototype.addEventListener=function(...args){
    if (args[0] === 'mousewheel') {
        console.log(args);
        return;
    }
    oldAdd.call(this,...args);
}
```

最终也就是脚本中的方法，将第三个参数设置为 true，也就是 useCapture 设置为 true （false 表示不使用 Capture，而使用 Bubbling 模式）

```
window.addEventListener('mousewheel', function(e){e.stopPropagation()}, true)
```

当鼠标滚动事件发生时，浏览器是从外到内处理绑定的事件，此时如果启用 Capture 将直接执行；接下来，再从内到外处理绑定的事件，此时如果是 Bubbling 模式将被执行。

调用 event.stopPropagation(); 即阻止捕获和冒泡阶段中当前事件的进一步传播。因为页面默认的 useCapture 设置为 false，这样操作相当于阻止了这个滚动事件。

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-6.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-6.png)

### 添加滚动条

这一步就简单许多了，只需要找到对应的元素，然后将其 style 中的 overflow 属性改为 scroll 就行了

[![](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-7.png)](https://blog.imwcr.cn/wp-content/uploads/2023/04/image-7.png)

转换成油猴脚本就是：

```
let alerts = document.getElementById('left_stream_alerts');
if (alerts) alerts.style.overflow = 'scroll';
```

顺便把原来的滚动条删除下：

```
let scrollbar = document.getElementsByClassName('scrollbar_track');
if (scrollbar.length === 3) {
    scrollbar[2].style.display = 'none';
}
```

### 最后的倔强

因为 Blackboard 页面还有窗口尺寸的监听，如果窗口尺寸修改了，还会将设置的 overflow 修改回去，所以最后的油猴代码中还加了循环检查，保证页面尺寸修改了，滚动条还在。