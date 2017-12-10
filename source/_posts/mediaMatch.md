---
title: 如何通过JS通过媒体查询规则来做样式控制
date: 2017-12-10 20:33:32
tags: 记录
---

在移动开发中，我们可以通过CSS的@media媒体查询来控制在不同屏幕宽度上应用不同的样式，但是在某些情况下，我们也希望可以通过脚本来作更多的控制。下面就通过一个新特性`window.matchMedia`来作控制。

通过传入一个@media规则进window.matchMedia方法会返回一个新的MediaQueryList 对象，这个对象表示指定的媒体查询字符串解析后的结果。

`mql = window.matchMedia(mediaQueryString)`

其中mediaQueryString参数是一个字符串，表示即将返回一个新MediaQueryList对象的媒体查询。

> [更多详细查阅 MDN API](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/matchMedia)

MediaQueryList对象在一个document上维持着一系列的媒体查询，并负责处理当媒体查询在其document上发生变化时向监听器进行通知的发送。

如果你需要以编程方式来检测一个document上的媒体查询的值的变化，这个MediaQueryList对象使得通过观察其document而检测它的媒体查询的值的变化成为可能，而不是周期性地对这些媒体查询的值进行检查。

它有两个方法：

`void addListener(MediaQueryListListener listener);`
`void removeListener(MediaQueryListListener listener);`

**MediaQueryList对象是一个非标准的对象，使用时应注意其在浏览器上的可用性。**

> [关于MediaQueryList的介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaQueryList)

下面是一个兼容例子：

```
if (window.matchMedia) {
    var mql = window.matchMedia("(min-width: 480px");
    mql.addListener(mediaChangeHandler);
    mediaChangeHandler(mql);
} else {
    window.addEventListener('resize', function() {
        var innerWidth = window.innerWidth
                    || document.documentElement.clientWidth
                    || document.body.clientWidth;
        mediaChangeHandler(innerWidth >= 480 ? {matches: true} : {matches: false}, false);
    })    
}

function mediaChangeHandler(mql) {
    if (mql.matches) {
        // 匹配成功时
        
    } else {
        // 匹配失败时
    }
}
```