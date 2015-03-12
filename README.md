汇总(2015 ES5)
===

> either the good parts or the confused parts, Node.js(Io.js) or pure JavaScript u want 2 know ;)

JavaScript是一门饱受争议的编程语言, 其存在固然有他优秀的一面. 当然, 在历史长河中得到的多数是批评, 这一点不可否认, 从它的某些"怪癖"确实能发现在语言设计上的缺陷和考虑的不周. 不过也恰恰因此使它看起来很神秘, Geek们喜欢通过Hack它来获取成就感, 比如`2 == [[[2]]]`返回`true`, `([]+[][-~[]/[]])[-~[]]+([]+{})[-~[]]+([]+-~[]/[])[(~-~-~[])*(~-~[])] `等于字符串`"not"`. 所以, 这里不管它是Good Parts还是Bad Parts, 先仔细玩味一番.

### 运算结合性与优先级

```js
var val = 'smtg';
console.log('Value is ' + (val === 'smtg') ? 'Something' : 'Nothing');
// => Something
// 因为+相比?有更高的结合优先级
```

```js
1 + - + + + - + 1
// => 2
// 记住运算的右结合规律
// 注意加号+之间有空格, 否则会被当做左值处理
```

### 类型转换

```js
var a = [1, 2, 3],
    b = [1, 2, 3],
    c = [1, 2, 4]
a ==  b	// false
a === b	// false
a >   c	// false
a <   c	// true

// 解释是: 在大小比较上, 对操作数调用了toString()方法, 而字符串间是通过CharCode比较大小的.
```

```js
function showCase(value) {
    switch(value) {
    case 'A':
        console.log('Case A');
        break;
    case undefined:
        console.log('undefined');
        break;
    default:
        console.log('Do not know!');
    }
}
showCase(new String('A'));

// => Do not know!
// 解释: switch 内部使用了全等运算 ===
```

再看下一个:
```js
function showCase2(value) {
    switch(value) {
    case 'A':
        console.log('Case A');
        break;
    case 'B':
        console.log('Case B');
        break;
    case undefined:
        console.log('undefined');
        break;
    default:
        console.log('Do not know!');
    }
}
showCase2(String('A'));

// => case A
// 不带new的String为类型转换函数, 并不创建对象.
```

### 原型继承

### 闭包

### other Tricks

#### eval

#### Timer

#### this

```js
var reverse = [].reverse;
reverse();
// => window or TypeError
// 因为reverse返回this
```

#### new

#### Function

##### name

##### 表达式与声明

##### arguments
```js
function sidEffecting(ary) {
  ary[0] = ary[2];
}
function bar(a,b,c) {
  c = 10
  sidEffecting(arguments);
  return a + b + c;
}
bar(1,1,1);

// => 21
// 记得arguments与函数形参之间有一个双向绑定(互相映射), 就算他们不在同一个作用域.
// 上面这个例子很好的说明了这点.
// notice: ES5移除了这一特性。
// 并且如果使用了"use stricts"语句（严格模式），arguments会变成保留字，也就是说不能将arguments当做变量名使用，不能给其赋值。
```

##### 提升


#### Array

```js
[].reduce(function () {})
// throw TypeError
// 对一个空数组进行无始值reduce调用会抛出TypeError
```

```js
var ary = [0,1,2];
ary[10] = 10;
ary.filter(function(x) { return x === undefined;});
// => []
// 因为filter只作用在非空元素上. (除了filter还有map也是这样)
```

#### Number

```js
Infinity % ANY_NUMBER
// => NaN
```

```js
var min = Math.min(), max = Math.max()
min < max;
// => false
// 如果不传入参数, min()返回Infinity, max()返回-Infinity
```

```js
Number.MIN_VALUE > 0
// => true
// MIN_VALUE是一个大于0的最小值:)
```

这段代码不会打印101, 而是进入一个死循环. 就算在在JavaScript世界里也很令人费解, 究竟发生了什么? 
```js
var END = Math.pow(2, 53);
var START = END - 100;
var count = 0;
for (var i = START; i <= END; i++) {
    count++;
}
console.log(count);
```

这还有个蛋疼的例子
```js
var a = 111111111111111110000,
    b = 1111;
a + b;
// => 111111111111111110000
```

别急, 还有更奇葩的数学计算
```js
var two   = 0.2
var one   = 0.1
var eight = 0.8
var six   = 0.6
[two - one == one, eight - six == two]
// => [true, false]
// 实际上eight - six == 0.20000000000000007
```

这是因为JavaScript中的浮点计算导致的. 了解一下背景知识:

> JavaScript采用IEEE 754标准定义的64位浮点格式表示数字. 这是一种二进制表示法, 可以精确表示如`1/2`,`1/8`,`1/1024`等分数, 但十进制分数如`1/10`,`1/100`等则不能**精确**表示.
> JavaScript中能表示的整数范围是`[2^-53, 2^53]`, 如果使用超出此范围的整数, 将无法保证低位数字的精度!
> 然而实际的操作(如数组索引,位操作符)则是基于32位整数的.

#### RegExp

```js
"1 2 3".replace(/\d/g, parseInt);
// => 1 NaN 3
```

要理解这个结果首先要知道`replace`的用法. 当`replace`第二个参数为回调时, 会传递三个参数: **第一个匹配项**, **匹配项索引**, **母串**. 而parseInt(2, 2)是非法运算, 返回为NaN.

```js
function captureOne(re, str) {
  var match = re.exec(str);
  return match && match[1];
}
var numRe  = /num=(\d+)/ig,
    wordRe = /word=(\w+)/i,
    a1 = captureOne(numRe,  "num=1"),
    a2 = captureOne(wordRe, "word=1"),
    a3 = captureOne(numRe,  "NUM=2"),
    a4 = captureOne(wordRe,  "WORD=2");
[a1 === a2, a3 === a4];

// => [true, false]
```

这是由于正则表达式中`g`引起的. `g`代表全局匹配, 启用后正则对象会记录lastIndex, 下次从这个索引继续匹配(不管是否为同一个字符串). 一旦匹配成功, 就会更新lastIndex, 否则lastIndex为0.

#### Date

```js
var a = new Date("epoch");
// => Invaild Date
// 不会抛出错误. o_O
```

```js
var a = Date(0);
var b = new Date(0);
var c = new Date();
[a === b, b === c, a === c];

// => [false, false, false]
// 以普通函数调用Date()只会返回一个字符串
```

```js
var a = new Date("2014-03-19"),
    b = new Date(2014, 03, 19);
[a.getDay() === b.getDay(), a.getMonth() === b.getMonth()]

// => [false, false]
```

Date构造函数可以传入多种类型的参数, 但如果是传入**多个**参数, 代表月份的第二个参数是从0开始的, 也就是3代表4月. 详见[MDN对Date的解释](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date#Syntax).

## Node.js (Io.js)
### 启动流程

### 模块系统

### 流机制

### 事件机制

### 底层I/O