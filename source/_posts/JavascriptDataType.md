---
title: Javascript 数据类型
date: 2017-08-06 23:14:13
tags:
---

&emsp;&emsp;*本文摘录于Javascript高级程序设计（Professional JavaScript for Web Developers）并加上了自己的一些理解，如发现本文有什么错误之处，请麻烦通过以下方式联系我作修正，谢谢!*
&emsp;&emsp;*微信号: kuntang9080*
&emsp;&emsp;*邮箱：kuntang@163.com*
___

ECMAScript中有5种简单数据类型（也称基本数据类型）：Undefined、Null、Boolean、Number、String。还有一种复杂数据类型：Object。

<!-- more  -->

---

#####typeof 操作符

对一个值使用typeof操作符可能返回下列某个字符串

| 返回值 | 说明 |
|--------|--------|
| undefined |  这个值没有定义    |
| boolean   |  这个值是布尔值|
|string|这个值是字符串|
|number|这个值是数值|
|object|这个值是对象或者null|
|function|这个值是函数|

==注意：当typeof返回是object的时候对应着两个值，因此不能用typeof来区分object类型和null类型，此时应该用instanceof()函数==

---

#####Undefined类型

Undefined类型只有一个值，即undefined

任何使用var声明但未对其加以初始化的变量都会赋以undefined值。

```Javascript
var message = undefined;
alert(message == undefined);
// true
```

对未初始化的变量和未声明的变量使用typeof操作符都会返回undefined值。

```Javascript
// 只声明了message，没有声明age
var message;

alert(typeof message);
// undefined
alert(typeof age);
// undefined
```

==注意：因此我们应该保持对变量初始化时就赋值的好习惯，这样当我们做类型检测的时候就不会造成不必要的混乱==

---

#####Null 类型

Null类型只有一个值，即null。从逻辑角度上看，null值表示一个空对象指针，而这也正是使用typeof操作符检测null会返回"object"的原因

实际上undefined值是派生自null值的，因此

```Javascript
alert(null == undefined)
// true
```

==如果一个声明了一个将来才会使用到的变量，那么我们应该显式地将它赋值null而不是其它值 ==

---

#####Number类型

###### 1. 进制

&emsp;&emsp;八进制字面值第一位必须为0，十六进制前两位必须为0x
```Javascript
var octalNum = 070
// 八进制的56
var hexNum = 0xA
// 十六进制的10
```

######2. 其它进制转换为十进制

&emsp;&emsp;其它进制转换为十进制我们可以用Number()或者parseInt()函数。因为Number()函数分的情况比较混乱，所以在很多情况下我们会用parseInt()来做进制转换。

parseInt()函数提供第二个参数：转换时使用的基数（即多少进制）。例：
```Javascript
var hexTo = parseInt("0xAF", 16)
// 175
var num1 = parseInt("10", 2)
// 2
var num2 = parseInt("10", 8)
// 8
var num3 = parseInt("10", 16)
// 16
```

######3. 浮点数值

对于那些极大或者极小的数值，可以用e表示法（科学计数法）来表示。
```Javascript
var floatNum = 3.125e7
// 31250000
```

==注意：浮点数值的最高精度是17位小数，但在进行算术时其精度远远不如整数。比如0.1加0.2不等于0.3：==
```Javascript
alert(0.1 + 0.2)
// 0.30000000000000004
if (a + b == 0.3) {	// 不要做这样的判断
	alert("you got the 0.3")
}
```

如果要一定要做这样的判断，在此提供了一个解决方法
```Javascript
var temp = (a * 10 + b * 10) / 10 	//先将浮点数转化为整数作加法，然后再转为浮点数
if (temp == 0.3) {
	alert("you got the 0.3")
}
```

######4. NaN (Not a Number)

- 任何涉及NaN的操作或运算都会返回NaN
- NaN与任何值都不相等

```Javascript
alert(NaN / 10)
// NaN

alert(NaN == NaN)
// false
```

isNaN()函数用来判断这个参数是否为NaN，当isNaN()接收到一个参数之后，会尝试将这个值转换为数值，任何不能被转换为数值的值都会导致这个函数返回true。
```Javascript
alert(isNaN(NaN))
// true
alert(isNaN(10))
// false
alert(isNaN("10"))
// false（转为数字10）
alert(isNaN("blue"))
// true（不能转为数值）
alert(isNaN(true))
// false(true转为1)
```
在基于对象调用isNaN()函数时，会先调用对象的valueOf()方法，然后确定该方法返回的值是否可以转换为数值。如果不能，则基于这个**返回值**再调用toString()方法，再测试返回值。

######5. 数值转换

- Number()函数

- parseInt()函数

&emsp;&emsp;parseInt()函数在转换字符串时，会忽略字符串前面的空格，直至找到第一个非空格的字符。如果第一个字符就不是数字字符或者负号，就直接返回NaN。也就是说parseInt("a123")返回NaN，第一个字符"a"不是数字字符或者负号；parseInt("12.3")返回12，"."不是数字字符或者负号。

- parseFloat()函数

&emsp;&emsp;parseFloat()函数始终都会忽略前导的零。对十六进制格式的字符串始终返回0。

---

##### 6. String类型

数值、布尔值、对象和字符串都有toString()方法，但是null和undefined值没有这个方法。
在调用数值的toString()方法时，可以传递一个参数：输出的基数。通过这个基数可以使toString()方法输出以二进制、八进制、十六进制，乃至其他做生意有效进制格式表示的字符串值。

```Javascript
var num = 10;
alert(num.toString())	// 10
alert(num.toString(2))	// 1010
alert(num.toString(8))	// 12
alert(num.toString(10))	// 10
alert(num.toString(16))	// a
```

在不知道要转换的值是不是null或undefined的情况下，可以使用转型函数String()，这个函数能够将任何类型的值转换的为字符串。String()函数遵循下列规则：

- 如果有toString()方法，则调用该方法（没有参数）并返回相应的结果

- 如果是null，则返回"null"

- 如果是undefined，则返回"undefined"

---

##### 7. Object类型

Object的每个实例都具有下列属性和方法。

- constructor： 保存着用于创建当前对象的函数

- hasOwnProperty(propertyName): 用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在。其中，作为参数的属性名(propertyName)必须以字符串形式指定（例：o.hasOwnproperty("name")）

- isPrototypeOf(object)：用于检查传入的对象是否是当前对象的原型

- propertyIsEnumerable(propertyName)：用于检查给定的属性是否能够使用for-in语句来枚举

- toLocaleString()：返回对象的字符串表示，该字符串与执行环境的地区对应

- toString()：返回对象的字符串表示

- valueOf()：返回对象的字符串、数值或者布尔值表示。通常与toString()方法的返回值相同
