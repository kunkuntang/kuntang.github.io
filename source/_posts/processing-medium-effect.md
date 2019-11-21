layout: '[post]'
title: 渐进式图片加载效果
tags:
  - 每周一练
  - 最佳实践
  - ''
categories: []
date: 2017-09-11 09:01:00
---
前些日子在逛知乎，发现有个效果挺不错的，就是当一开始页面上的图片是模糊的，过一会图片变清晰。突然感觉这种效果比传统的占位图效果要好的多，于是在好奇心的驱使下百度了一下效果，得出的实现常用可以分为图床和图片地址替换。下面简单叙述一下两个方案的实现思路：

### 图床

就是在图片原来的地方放一个标签用来放模糊的照片，盖在原来清晰大图片的上方，当大图加载成功后，原来盖在上面的图片隐藏掉。图床可以是一个Div或者是Canvas等。

<!-- more -->

### 地址替换

思路和上面差不多，一开始的时候先加载一张低清的图，把高清图片的地址存在某个属性里，等页面加载完成后，用JS取到高请图片的地址，然后用image对象加载，等加载完成后再把图片地址替换成高清的地址。

本文介绍的是方法二用地址替换的方式来实现：

### 案例效果

![案例效果](/images/processing_image.gif)

### 案例代码

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Processing Image Effect Demo</title>
    <link rel="stylesheet" href="./css/index.css">
</head>
<body>
    <div class="img-con" id="imgCon">
        <img src="./img/1_small.jpg" data-src="./img/1_big.jpeg" alt="图片1" id="img">
        <img src="./img/2_small.jpg" data-src="./img/2_big.jpg" alt="图片2" id="img">
        <img src="./img/3_small.jpg" data-src="./img/3_big.jpg" alt="图片3" id="img">
        <img src="./img/4_small.jpg" data-src="./img/4_big.jpg" alt="图片4" id="img">
        <img src="./img/5_small.jpg" data-src="./img/5_big.jpg" alt="图片5" id="img">
    </div>
    <script>
        window.onload = function() {
            var imgCon = document.getElementById('imgCon')
            var imgs = imgCon.getElementsByTagName('img')
            for (var i = 0; i < imgs.length; i++) {
                
                (function(curImg) {
                    var tempImg = null;
                    tempImg = document.createElement('img')
                    console.log(curImg)
                    tempImg.src = curImg.dataset.src;
                    tempImg.onload = function(e) {
                        curImg.src = tempImg.src
                        curImg.style.filter = 'blur(0px)'
                    }
                })(imgs[i])
            }
        }
    </script>
</body>
</html>
```

```CSS
.img-con img {
    width: 100%;
    height: 400px;
    -webkit-transition: filter .3s ease-out 0s;
    -moz-transition: filter .3s ease-out 0s;
    transition: filter .3s ease-out 0s;
    filter: blur(10px);
    -ms-filter: blur(10px);
    -webkit-filter: blur(10px);
}
```

### 关键知识点

- JS Image对象
- CSS3 filter blur 属性

*注意： Image对象读取图片的过程是异步的，需要弄清楚代码执行的时序问题*