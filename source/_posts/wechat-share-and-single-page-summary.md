layout: '[post]'
title: 单页面应用微信分享跳坑指南
tags:
  - 个人总结
categories:
  - 小程序
  - ''
date: 2018-08-01 19:18:00
---

## 前言

最近在开发的时候遇到了一个微信分享的bug，就是无论你在哪个路径下的页面，发送给朋友后点开都只会跳到项目的首页。本来微信分享这个只算是一个小功能，也很好解决，但由于项目的特殊性，使得在这个bug解决起来并没有那么顺手，所以记录一下备以后翻阅。

## 坑点

- Vue单页面应用，前端通过Hash控制路由——iOS在微信中不能正常地改变浏览器的hash值，分享出去的页面地址被莫名其妙地添加了参数。

- 微信的安全策略——由于存在js安全域名限制，使得在本地调试更难。

- jssdk配置签名。

## 跳坑方法

### 分享地址被奇怪的被带上了参数

在传统开发中，路由通常都是在后端完成的，但是在Vue单页面中，都是通过控制history interface来控制页面之间的跳转，在我们的项目中我们使用hash的方式，但是在分享给朋友后却发现分享地址被加上了一些参数，比如：

我分享出去的地址是：`market.lenkuntang.cn/#/home`，分享后会变成了`market.lenkuntang.cn/?from=singlemessage#/home`。这到底会不会影响到我们的分享操作呢？这就要了解vue-router的工作原理了，翻看了一下vue-router的源码，发现如下代码：

```
// this is delayed until the app mounts
  // to avoid the hashchange listener being fired too early
  setupListeners () {
    const router = this.router
    const expectScroll = router.options.scrollBehavior
    const supportsScroll = supportsPushState && expectScroll

    if (supportsScroll) {
      setupScroll()
    }

    window.addEventListener(supportsPushState ? 'popstate' : 'hashchange', () => {
      const current = this.current
      if (!ensureSlash()) {
        return
      }
      this.transitionTo(getHash(), route => {
        if (supportsScroll) {
          handleScroll(this.router, route, current, true)
        }
        if (!supportsPushState) {
          replaceHash(route.fullPath)
        }
      })
    })
  }
```

[hash.js](https://user-gold-cdn.xitu.io/2018/8/1/164f36e016b35048)

原来在vue-router初始化的时候，会监听`window`对象的`hashchange`属性，如想发现浏览器的`hash`值发生变化了，就会调用`History.transitionTo`方法，关键就在这个方法会传入一个`getHash`方法为作参数，如果在这种地址`market.lenkuntang.cn/?from=singlemessage#/home`也能正确地拿到正确的`hash`的话，那我们就可以断定这种意外对我们的分享是没有影响的。当我们继续去看`getHash`方法，在`hash.js`往下翻点会找到这个方法的实现：

```
export function getHash (): string {
  // We can't use window.location.hash here because it's not
  // consistent across browsers - Firefox will pre-decode it!
  const href = window.location.href
  const index = href.indexOf('#')
  return index === -1 ? '' : href.slice(index + 1)
}
```

我们可以清楚地知道，当这条地址`market.lenkuntang.cn/?from=singlemessage#/home`经过`getHash`之后会直接返回`#`号后面的字符串，也就是
`/home`，所以可以得出是不会对我们分享的功能有影响的。

### iOS在微信环境中浏览器地址不变

在Vue-router实现前端控制路由都是通过HTML5 新增的History Interface接口来控制页面之间的跳转的，在跳转的同时通过修改`window`中`loaction`的`hash`属性反映回浏览器的地址，但是当遇到iOS时却意外地发现这个`hash`属性一直没有被改变，导致每次分享出去的地址都是首页，在网上一查发现这原来是个通病，解决的方法就是引入微信的JsSDK来手动控制分享的地址。

### 引入JsSDK所带来的问题

在引入了JsSDK后，首先要对它进行配置，相关配置项如下：

```
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名
    jsApiList: [] // 必填，需要使用的JS接口列表
});
```

说明一下这里的参数分别从哪里来，appId是从微信公众号里获取的，`timestamp`和`nonceStr`还有`signature`是从服务器中返回的。jsApiList可以在[所有JS接口列表](https://mp.weixin.qq.com/wiki?action=doc&id=mp1421141115&t=0.11471355121805527#63)中找到。

> 注：`timestamp`和`nonceStr`其实是可以在前端生成然后传给服务器再参与签名的计算的，但一般在考虑到安全原因，`timestamp`, `nonceStr`这些参数应该从服务器返回回来（因为它参与了签名的计算）。

> 注意：这里的传入的随机字符串字段`nonceStr`是**驼峰命名！！！**

然后就是引入JsSDK中遇到最大的问题——签名问题，要正确地实现使用JsSDK，在服务器端首先要集齐这四种元素：
- noncestr（随机字符串）
- jsapi_ticket（通过微信接口获得的ticket）
- timestamp（时间戳）
- url（当前网页的URL，不包含#及其后面部分）

然后把这些元素按字典序（ASCII 码从小到大排序）排后使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串，再对字符串进行sha1加密，字段名和字段值都采用原始值，不进行URL 转义，即可得到所谓的签名。

> 注意：这里的传入的随机字符串字段`noncestr`是**全小写！！！**

最后附上签名检验工具的地址：[http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign](http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign)

还有示例代码：[http://demo.open.weixin.qq.com/jssdk/sample.zip](http://demo.open.weixin.qq.com/jssdk/sample.zip)


得到签名后再把`timestamp`，`nonceStr`和`signature`传回给前端进行JsSDK的初始化配置。

### 再说计算签名的URL

这里再说说参与签名的url，因为这里传过去的是当前见面的URL且不包括#及其后面部分，这对于使用Hash模式的单页面应用来说是个好消息，这样就代表我们只需要在页面加载时初始化一次后便可以在所有页面上使用（对于传统的路径导航，因为URL变了所以要重新初始化，也就是说要在使用到的JsSDK功能的页面中都要重新请求后台接口拿签名再初始化！！）。所以，一般来说我们通常会在`App.vue`这个文件中作JsSDK的初始化操作，当初始化正确后便可在其它页面上直接使用JsSDK接口的功能。

次外，由于微信存在对JsSDK的使用限定在微信公众号里所设置的JS接口安全域名范围里，所以对于本地调度用的`localhost`域名来说是不可行的，直接提示`invalid url domain`，在这里有两种方式可以解决这个问题，一种是通过修改`host`的方法来实现本地调试，方法如下：

#### window系统：

进入系统盘目录（通常是C盘）： `C:\Windows\System32\drivers\etc`，找到`hosts`文件，打开后文件末尾添加一条记录`127.0.0.1 market.lenkuntang.cn`,这条记录的意思是当你访问`market.lenkuntang.cn`这个地址的时候会重定向到`127.0.0.1`这个ip地址，从而实现本地调试的目的。

##### mac系统

打开一个finder，然后按快捷键command+shift+G，输入`private/etc/hosts`回车后就能找到对应的hosts文件，由于是权限问题，是无法直接在那个目录中修改hosts文件的，所以要把文件复制到桌面或者其它有修改权限的目录，然后打开后也是类似window一样在文件末尾添加一条记录`127.0.0.1 market.lenkuntang.cn`,保存后拖回原目录确定覆盖。

另一种是使用腾讯云的开发者实验室的在线Web IDE来登录到测试服务器，然后直接在服务器上进行修改，线上验证。但是由于这个Web IDE目前不支持SSH密钥方式登录，只能用账号和密码的方式登录。所以也是有一定的局限性的。

附上Web IDE工具地址：[https://cloud.tencent.com/developer/labs/gallery](https://cloud.tencent.com/developer/labs/gallery)

点击其中一个教程，然后选择开始上机下方的*使用已有*云主机标签，在弹出的登录界面中正确填写你服务器的IP地址和账号密码便可直接登入服务器内进行相关操作。


![登录界面](https://user-gold-cdn.xitu.io/2018/8/1/164f52d678022dc4?w=2550&h=1270&f=png&s=173339)

* 登录界面

![登录成功后的界面](https://user-gold-cdn.xitu.io/2018/8/1/164f52e11cd4a132?w=2556&h=1262&f=png&s=214551)

* 登录成功后的界面

### 使用微信开发者工具来本地调试

当我们配置好了所有东西后，打开浏览器我们可以在控制台的输出中看到JsSDK的相关信息，但是我们却不知道是否可以正确分享，难道我们每次都要使用手机来访问本地服务来验证吗？而且在使用手机来访问本地服务的时候，使用的是本地电脑的ip地址，这样去拿到签名肯定是不对，会报`invalid url domain`错误，当然也可以改手机的`hosts`，但是这就不是那么容易改了，安卓的话要root，苹果的话...算了算了。还是换种方法，这个时候我们应该使用微信开发者工具来进行调试，微信开发者工具可以模拟微信环境，可以进行微信想着的操作，所以使用这个工具我们就可以愉快地在本地进行调试啦。

而且，在遇到需要微信登录的页面时，如何是用普通的浏览器来打开就会跳到微信的授权登录页，而用开发者工具来打开则会像手机一样弹出授权页：

![普通浏览器打开](https://user-gold-cdn.xitu.io/2018/8/1/164f519bbd68322e?w=334&h=583&f=png&s=11804)

* 普通浏览器打开

![微信开发者工具](https://user-gold-cdn.xitu.io/2018/8/1/164f51ebf2b165e6?w=380&h=698&f=png&s=30325)

* 微信开发者工具

### 总结

通过这几天对微信分享的研究，总体对微信的JsSDK的使用有了大概的认识和了解，虽然其中也遇到不少的坑和麻烦的地方，但是既然问题出现就只能尽量地去简化问题再解决它。