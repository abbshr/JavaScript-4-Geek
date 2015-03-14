汇总(2015 ES5)
===
> 汇集权威指南, 秘密花园, 设计模式, 网络资源以及个人总结.

> either the good parts or the confused parts, Node.js(Io.js) or pure JavaScript u want 2 know ;)

JavaScript是一门饱受争议的编程语言, 其存在固然有他优秀的一面. 当然, 在历史长河中得到的多数是批评, 这一点不可否认, 从它的某些"怪癖"确实能发现在语言设计上的缺陷和考虑的不周. 不过也恰恰因此使它看起来很神秘, Geek们喜欢通过Hack它来获取成就感, 比如`2 == [[[2]]]`返回`true`, `([]+[][-~[]/[]])[-~[]]+([]+{})[-~[]]+([]+-~[]/[])[(~-~-~[])*(~-~[])] `等于字符串`"not"`. 所以, 这里不管它是Good Parts还是Bad Parts, 先仔细玩味一番.

### 运算结合性与优先级

**注意运算符的优先级和结合性问题!**

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

#### 关系表达式中的类型转换

##### 恒等运算符 (`===`)

+ `NaN`与任何值都不相等, 包括其自身.
+ 如果两个字符串所显示的字符一样, 但具有不同编码的16位值, 那么他们被判为不等.

##### 相等运算符 (`==`)

+ 如果两个操作数类型相同, 那么和恒等比较一样.
+ 如果类型不同, 则遵循如下规则:
  - null和undefined相等
  - 如果一个值为数字, 一个为字符串, 就先将字符串转换为数字再做比较.
  - 如果一个值为布尔类型, 则转换为数字再进行比较.
  - 如果一个值为对象, 另一个为数字/字符串, 则将对象先通过`valueOf()`再通过`toString()`转化为原始值后进行比较, 但日期类型除外, 它直接使用`toString()`转化为原始值.
  - 其他比较均不相等.

##### 比较运算符 (`> < >= <=`)

+ 如果操作数为对象, 那么首先使用`valueOf()`, 如果不能转化为原始值, 那么使用`toString()`的结果进行比较操作.
+ 如果两个原始值都是字符串, 将按照字符code比较大小.
+ 如果两个原始值中至少一个不是字符串, 那么都转换为数字进行比较.
+ `NaN`与任何值的比较都将返回`false`.

#### 类型转换规则

value|to string|to number|to boolean|to object
:---:|:-------:|:-------:|:--------:|:-------:
`undefined`|`"undefined"`|`NaN`|`false`|`new TypeError()`
`null`|`"null"`|`0`|`false`|`new TypeError()`
`true`|`true`|`1`||`new Boolean(true)`
`false`|`false`|`0`||`new Boolean(false)`
`""`||`0`|`false`|`new String('')`
`"5"`(数字字符串)||`5`|`true`|`new String('5')`
`"ran"`(非数字字符串)||`NaN`|`true`|`new String('ran')`
`±0`|`"0"`||`false`|`new Number(0)`
`5`(非零)|`"5"`||`true`|`new Number(5)`
`NaN`|`NaN`||`false`|`new Number(NaN)`
`±Infinity`|`"±Infinity"`||`true`|`new Number(±Infinity)`
`[]`|`""`|`0`|`false`|
`[5]`(一个数字元素)|`"5"`|`5`|`true`|
`['ran']`(其他数组)|`join(',')`|`NaN`|`true`|
`function () {}`|函数源码字符串|`NaN`|`true`|
`{}`(任意对象)|如下节所示|如下节所示|`true`|

##### 对象向原始值的转化
任何对象{}都有两个原始的方法toString()和valueOf()，继承自原型链的顶端——Object.prototype对象。当对象做隐式转换时，首先会自动判断将要转化为什么类型，然后按规则调用上述两个方法。规则如下：

1. 如果对象要被转化为布尔型，最简单：所有引用类型或者说对象都转换成true。
2. 如果转化为字符串：若有toString()方法，则调用，当该方法返回原始值时被转化为字符串，最终返回这个字符串。若没有这个方法或者该方法返回的并不是原始值，则调用valueOf()方法，如果它返回原始值则被转化为字符串并最终返回该字符串，否则转化出错，抛出TypeError。
3. 如果转化为数字：和上面的情况相似，只不过先检查valueOf()后检查toString()，然后将原始值转化为数字而已，否则抛出TypeError。

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

[blog 关于原型链的解释](https://github.com/abbshr/abbshr.github.io/issues/33)

几种继承方式:
```js
// 1
cobj = Object.create(pobj);
// 2
cobj.__proto__ = pobj;
// 3
var Construct = function () { this.id = 'R' };
Construct.prototype = { pro: 'fun' };
Construct.call(cobj);
// 4
var Con2 = function () { this.id = 'S' };
Con2.prototype = new Construct

Con2.prototype = Construct.prototype

Con2.prototype.__proto__ = Construct.prototype
```

### 闭包

[blog 闭包, 作用域链, 内存泄露](https://github.com/abbshr/abbshr.github.io/issues/31)

### other Tricks

#### eval
eval之所以不被建议使用，是因为他会‘破坏作用域’。

除了`eval()`，`Function()`或`new Function()`都能达到解析并执行代码的目的。只不过`Function()`和`new Function()`解析的代码只能在局部作用域执行，因此代码中的var声明的变量不会成为全局变量。而eval()却可以访问、修改它的外部作用域的变量，可能影响到全局变量。

eval如何破坏作用域？

```js
var foo = 1;
function test() {
    var foo = 2;
    var bar = eval;  //这里将bar设为eval的引用
    bar(' foo = 3; ');
    return foo;
}
test();   //返回2
console.log(foo);  //返回3
```

正常思路来讲，test会返回foo=3，控制台打印的值为1才对吧，这就是eval破会作用域的结果。为什么eval会这么做？
eval只有在被直接调用并且调用函数就是eval 本身时，解析的代码才会在当前作用域执行，否则就会像上面那样。

#### this

1. 全局环境下，this指向全局对象。
2. 函数直接调用时，内部this指向全局对象。
3. 调用对象的方法时，函数内部this指向调用该方法的对象。
4. 以构造函数的方式调用（即在函数名前加new），this指向新创建的对象.
5. 显式设置this，例如：使用Function.prototype上的call来显式设置调用该方法的对象时，函数内this指向该对象。
6. 直接调用在函数内部定义的闭包时，闭包内this指向全局对象。（同情况2）
    ```js
    var bar.getAge = function () {
     function getBirth() {
         return this.birthday;
     }
     getBirth();  //this指向全局
    };
    //注意：闭包中的this不会因为getBirth对象没有birthday属性而按照变量解析顺序去向上一作用域寻找birthday，因为this是关键字，而非变量！
    //要想获取bar对象，应该在getAge内部声明一个变量来存储this，然后将这个变量以参数形式传入getBirth：
    var bar.getAge = function () {
     var that = this;
     function getBirth(that) {
         return that.birthday;
     }
     getBirth(that);  //正常返回
    };
    ```
7. 将方法赋值给一个变量，则函数内部this指向全局对象（此时相当于情况2）。
    ```js
    var tes = bar.getAge;
    bar.getAge();//this指向bar
    tes();//this指向全局对象
    ```
8. 将函数/方法以参数形式传给另一个函数时，参数this指向全局对象，这有点像“将函数/方法赋值给形参”（同情况7）。
    ```js
    function foo(callback) {
     callback();
    }
    foo(bar.done);//done的this不会再指向bar，而是全局对象
    ```
9. 对象字面量声明中，this不能用来指向对象本身，而是当前有效作用域的全局对象：
    ```js
    var obj = {
     name: 'mmm',
     do: this.geName, //这里就不能这么写了，因为this为全局对象，正确的写法是像下面的方法这样以获取obj对象
     getName: function {
         return this.name;
     }
    };
    ```

```js
var reverse = [].reverse;
reverse();
// => window or TypeError
// 因为reverse返回this
```

#### new

当函数以new调用，则会表现的与普通调用有所不同。表面上看就是产生了一个标准对象，其实这个调用过程中函数内部是这样的:

1. 首先创建一个空对象，该对象自动继承了构造函数的原型。
2. 构造函数中定义的属性/方法也被添加到刚刚创建的对象里。 
3. 新创建的对象被构造函数内的this引用。
4. 最后，如果函数没有显示设置return的对象，则 隐式返回这个新创建的对象。

** 注意构造函数的有效return返回的永远是一个对象，而不可能是其他类型 
所以，类似return 2、return 'hey'的返回值都不会被真正返回，这时仍然返回内部创建的对象.**

#### Function

##### 变量解析顺序
比如当访问函数内变量“ran”时，会按如下顺序寻找ran：

1. 当前作用域是否有“ran”？是，则访问成功；否，则跳到2.  
2. 函数形参是否使用“ran”为名？是，则访问成功；否，则跳到3.  
3. 函数自身的标识（函数名）是否叫“ran”？是，则访问成功；否，则跳到4.  
4. 回溯到上一层作用域，并同时跳到1.，如此循环直到当前作用域为顶层作用域为止，若仍未找到“ran”，则返回undefined。

##### 表达式与声明

```js
var f = function fn() {};
f();
fn();
// => ReferenceError

(function func() {})();
func();
// => ReferenceError
// 两处ReferenceError, 是因为"立即函数"调用相当于立刻调用函数表达式, 所以函数的name并不在当前作用域中.
```

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
函数内部无论在那里声明变量，效果都等同于在函数顶部进行声明。

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

```js
var arr = new Array(3);
var ary = [,,,];
// 两个创建方式是等效的, 都会产生一个稀疏数组, 并且元素尚未存在. 和下面是不一样的:
var ar = [undefined, undefined, undefined];

0 in ary;
// => false
0 in ar;
// => true
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
### 启动流程 & 模块系统
[阅读源码理解node.js的启动, require和moudle那些事儿](https://github.com/abbshr/abbshr.github.io/issues/24)

### 流机制
[node stream: the secret part](https://github.com/abbshr/abbshr.github.io/issues/37)

### 事件机制
[libuv源码初探](https://github.com/abbshr/abbshr.github.io/issues/26)
[libuv源码初探2](https://github.com/abbshr/abbshr.github.io/issues/39)

[node笔记](https://github.com/abbshr/abbshr.github.io/issues/1)
[node笔记2](https://github.com/abbshr/abbshr.github.io/issues/2)

### 底层I/O
[从Node.js再谈Linux I/O](https://github.com/abbshr/abbshr.github.io/issues/20)