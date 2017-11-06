---
title: JavaScript 设计模式与开发实践
date: 2017-08-09 19:51:22
tags: 读书笔记
---

## 基础知识

### 一、this、call 和 apply

#### this的指向

除去不常用的with和eval的情况，具体到实际应用中，this的指向大致可以分为以下4种：

- 作为对象的方法调用。
- 作为普通函数调用。
- 构造器调用。
- Function.prototype.call或Function.prototype.apply调用。

<!-- more -->

1. 作为对象的方法调用

当函数作为对象的方法被调用时，this指向该对象：

```
var obj = {
	a: 1，
    getA: function() {
    	alert( this === obj );
        alert( this.a );
    }
}

obj.getA();
// output: true
// output: 1
```
2. 作为普通函数调用

当函数不作为对象的属性被调用时，也就是我们常说的普通函数方式，无论它在哪里调用，此时的this总是指向全局对象。在浏览器的JavaScript里，p个全局对象 是window对象。

```
window.name = 'golbalName';

var getName = function() {
	return this.name;
};

var anotherGetName = function() {
	console.log(getName())
}

console.log( getName()) ;
// output: globalName

anotherGetName();
// output: globalName
```

或者

```
window.name = 'globalName';

var myObject = {
	name: 'sven',
    getName: function(){
    	return this.name;
    };
};

var getName = myObject.getName;
cosnole.log( getName() );
// output: globalName

```

3. 有时候我们会遇到一些困扰，比如在div节点的事件函数内部，有一个局部的callback方法，callback被作为普通函数调用时，callback内部的this指向了window,但我们往往是想让它指向该div节点，见如下代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="div1">I am a div</div>

    <script>
        window.id = 'window'
        document.getElementById('div1').onclick =function() {
            alert( this.id ); // output: div1
            var callback =function() {
                alert( this.id ); // output: window
            }
            callback()
        }
    </script>
</body>
</html>
```

此时有一种简单的解决方案，可以用一个变量保存div节点的引用：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="div1">I am a div</div>

    <script>
        window.id = 'window'
        document.getElementById('div1').onclick =function() {
            var that = this
            var callback =function() {
                alert( that.id ); // output: div1
            }
            callback()
        }
    </script>
</body>
</html>
```

在ECMAScript5的strict模式下，这种情况下的this已经被规定为不会指向全局对象，而是undefined:


```
function func() {
	“use strict"
    alert( this ) // output: undefined
}
```

### 二、闭包和高阶函数

#### 闭包

## 设计模式

### 订阅模式

订阅模式的设计主要有两个结构，一个是存放订阅事件的数组，还有添加订阅事件的方法，广播执行订阅事件的方法。

例： 

```
var event = {
    // 存放订阅事件的数组队列
    clientList: [],
    // 添加订阅事件的方法
    listen: function( key, fn ) {
        // key 是订阅事件的代号标志，比如login表示登录订阅事件，
        // loadFail表示读取失败订阅事件
        // fn 是要订阅事件触发时执行的函数
        if (!this.clientList[ key ]) {
            this.clientList[ key ] = []
        }
        // 把订阅消息添加到缓存列表
        this.clientList[ key ].push( fn );
    },
    trigger: function() {
        var key = Array.prototype.shift.call( arguments )
            fns = this.clientList[ key ]
        
            if ( !fns || fns.length === 0 ) { // 如果没有绑定对应的消息
                return false
            }
        
            for ( var i = 0, fn; fn = fns[ i++ ];) {
                fn.apply( this, arguments )  // arguments 是 trigger 是带上的参数
            }
    }
}
```

我们来测试一下上面的代码：
首先定义一个installEvent函数让我们可以给所有对象都动态添加发布-订阅功能（但是这个并不是必须的，不过一般都会声明一个方便调用）

```
var installEvent = function( obj ) {
    for ( var i in event ) {
        obj[ i ] = event[ i ];
    }
}
```

假设一个场景，在课堂上老师个学生布置作业，然后等我学生到做作业的时间的时候，就开始做老师发布的作业。在这里老师是订阅者，学生是发布者。

```
// 先给老师添加发布订阅功能
var teacher = {}
installEvent( teacher )

// 老师备课时先定义好将要布置的作业
function doMath() {
	console.log( 'do Math homework' )
}

function doMathTest() {
	console.log( 'do Math test homework' )
}

function doEnglish() {
	console.log( 'do Englisth homework' )
}

// 在课堂上老师发布作业
teacher.listen( 'Math', function() {
    doMath();
})

teacher.listen( 'Math', function() {
	doMathTest()
})

teacher.listen( 'English', function() {
    do English()
})

// 做作业的时间到了，学生做老师发布的作业
// 用if来假设条件成立，可以去掉
if ( new Date().now === 8888 ) {
	teacher.trigger( 'Math' )
	teacher.trigger( 'English' )
}

```

发布订阅模式还很适合协作开发，比如上面的例子，老师只需要负责这天需要做哪科作业（Math, English)，做哪一题(doMath, doMathTest, doEnglish)。而学生只需要负责到点就去完成作业。

### 中介者模式

中介者就是把许多相关联的对象进行解耦。有时候对象与对象之间的操作会互相影响，有些对象要在其他对象改变的时候做出相应的响应，这个时候就要用到中介者模式。

中介者模式一般有两种实现方式：

- 利用发布-订阅模式。
- 中介者开放一些接口给其它对象调用，而具体实现的逻辑在中介者中实现，然后中介者把执行后的结果发送给其它对象。

考虑一个现实的一个例子：
这手机购买的过程中，可以选择手机的颜色和数量，同时在页面会有相应手机库存的显示，然后页面底下的购习按钮会根据库存等情况作出不同的展示。

假设有这几种规格的手机：

```
var goods = {
	"red": 3,
	"blue: 6
}
```

粗略地把上面分成以下几种情况：

1. 选择红色手机，买4个，显示库存不足，购买按钮不可点
2. 选择蓝色手机，买5个，显示库存充足，购买按钮可点
3. 不有选择颜色或者数量的时候，购买按钮不可点

以下进行编码：

页面HTML代码：

```
选择颜色： 
<select name="" id="colorSelect">
    <option value="">请选择</option>
    <option value="red">红色</option>
    <option value="blue">蓝色</option>
</select>
输入购买数量：
<input type="text" id="numberInput">
您选择了颜色：
<div id="colorInfo"></div><br/>
您输入了数量：
<div id="nubmerInfo"></div>
您输入了容量：
<div id="momeryInfo"></div>
```

先定义商品规格：

```
var goods = {
    "red|32G": 3,
    "red|64G": 0,
    "blue|32G": 1,
    "blue|16G": 6
}
```

定义中介者来作处理其中判断的逻辑，返回一个事件让其它关联对象调用。

```
var mediator = (function(){
    var colorSelect = document.getElementById('colorSelect')
    var memorySelect = document.getElementById('memorySelect')
    var numberInput = document.getElementById('numberInput')
    var colorInfo = document.getElementById('colorInfo')
    var memoryInfo = document.getElementById('memoryInfo')
    var numberInfo = document.getElementById('numberInfo')
    var nextBtn = document.getElementById('nextBtn')

    return {
        changed: function(obj){
            var color = colorSelect.value,  // 颜色
                memory = memorySelect.value, // 内存
                number = numberSelect.value, // 数量
                stock = goods[ color + '|' + memory ];  // 颜色和内存对应的手机库存数量
            
            if ( obj === colorSelect ) {
                colorInfo.innerHTML = color
            } else if ( obj === memorySelect ) {
                memoryInfo.innerHTML = memory
            } else if ( obj === numberInput ) {
                numberInfo.innerHTML = number
            }

            if ( !memory ) {
                nextBtn.disabled = true
                nextBtn.innerHTML = '请选择手机颜色'
                return
            }

            if ( !memory ) {
                nextBtn.disabled = true
                nextBtn.innerHTML = '请选择内存大小'
                return
            }

            if ( Number.isInteger ( number - 0 ) && number > 0 ) {
                nextBtn.disabled = true
                nextBtn.innerHTML = '请输入正确的购买数量'
                return
            }

            nextBtn.disabled = false
            nextBtn.innerHTML = '放入购物车'
        }
    }
})()
```

当相对应的对象改变的时候，中介者就能通过响应作出正确的处理，而不用把所有的判断逻辑放在各个事件监听所触发的函数里。

```
colorSelect.onchange = function() {
    mediator.changed( this )
}

memorySelect.onchange = function() {
    mediator.changed( this )
}

numberInput.onchange = function() {
    mediator.changed( this )
}
```

