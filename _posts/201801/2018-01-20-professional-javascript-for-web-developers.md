---
layout: post
title:  "JavaScript 高级程序设计读书笔记 1"
date:   2018-01-20 14:30:00
categories: javascript
tags: learningnote
author: "Victor"
---

## 1. JavaScript 简史
JavaScript 是一种专为与网页交互而设计的脚本语言，由下列三个不同的部分组成:
1. ECMAScript，由 ECMA-262 定义，提供核心语言功能
2. 文档对象模型 `DOM`，提供访问和操作网页内容的方法和接口
3. 浏览器对象模型 `BOM`，提供与浏览器交互的方法和接口

## 2. 在 HTML 中使用 JavaScript
### `<script>` 元素
`defer` 和 `async` 都是异步加载脚本，都会在脚本加载完后执行。它们的区别如下：
* `defer` DOM 解析完成后，`document` 的 `DOMContentLoaded` 事件触发之前，按 script 顺序执行
* `async` 在 `window` 的 `load` 事件触发之前，不按 script 顺序执行

**在现实当中，延迟脚本并不一定会按照顺序执行，也不一定会在 DOMContentLoaded 事件触发 前执行，因此最好只包含一个延迟脚本。**

只要不存在 `defer` 和 `async` 属性，浏览器都会按照 `<script>` 元素在页面中出现的先后顺序对它们依次进行解析。另外，`type` 默认值是 `text/javascript` 不用再写了;`charset` 指定字符集，可忽略; `src` 外部文件; `language` 已废弃。

## 3. 基本概念
### 语法
1. ECMAScript 中的一切(变量、函数名和操作符)都区分大小写
2. 标识符，就是指变量、函数、属性的名字，或者函数的参数。第一个字符必须是一个字母、下划线 `_` 或一个美元符号 `$`，其他字符可以是字母、下划线、美元符号或数字。
3. ECMAScript 中的语句以一个分号结尾

### 关键字和保留字
像其它语言一样，JavaScript 也有语言保留的关键字和保留字，不能用作标识符。列表参考 [保留字列表](https://www.alexleo.click/javascript-%E5%B0%8F%E7%AD%86%E8%A8%98-%E4%BF%9D%E7%95%99%E5%AD%97%E5%88%97%E8%A1%A8/)。

### 变量
1. ECMAScript 的变量是松散类型的，所谓松散类型就是可以用来保存任何类型的数据
2. 未经过初始化的变量，会保存一个特殊的值 `undefined`
3. 可以使用一条语句定义多个变量

```javascript
var message = "hi";
message = 100; // 有效，但不推荐

function test(){
message = "hi"; // 全局变量，不是我们推荐的做法。
}

var message = "hi", //使用一条语句定义多个变量
  found = false,
  age = 29;
```

### 数据类型
* 基本数据类型: `Undefined、Null、Boolean、Number、String`
* 复杂数据类型: `Object`

#### typeof 操作符
使用 `typeof` 检测给定变量的数据类型：`undefined` 这个值未定义；`boolean` 这个值是布尔值；`string` 这个值是字符串；`number` 这个值是数值；`object` 这个值是对象或 `null`；`function` 这个值是函数。

**`typeof(null)` 会返回 `object`，因为特殊值 `null` 被认为是一个空的对象引用。**

#### Undefined 类型
* `Undefined` 类型只有一个值，即特殊的 `undefined`。在使用 `var` 声明变量但未对其加以初始化时，这个变量的值就是 `undefined`。
* 对于尚未声明过的变量，只能执行一项操作，即使用 `typeof` 操作符检测其数据类型
* 论在什么情况下都没有必要把一个变量的值显式地设置为 `undefined`

**引入这个值是为了正式区分空对象指针与未经初始化的变量。**

#### Null 类型
* `Null` 类型只有一个值，即特殊的 `null`。`null` 值表示一个空对象指针
* 如果定义的变量准备在将来用于保存对象，那么最好将该变量初始化为 `null` 而不是其他值
* `undefined` 值是派生自 `null` 值

```javascript
if (car != null){
  // 只要直接检查 `null` 值就可以知道相应的变量是否已经保存了一个对象的引用
}

alert(null == undefined); //true
```

#### Boolean 类型
* 该类型只有两个字面值: `true` 和 `false`，注意区分大小写
* 要将一个值转换为其对应的 Boolean 值，可以调用转型函数 `Boolean()`
* 流控制语句自动执行相应的 Boolean 转换

| 数据类型 | 转换为 true 的值 | 转换为false的值 |
| === | === | === |
| Boolean | true | false |
| String | 任何非空字符串 | ""(空字符串) |
| Number | 任何非零数字值(包括无穷大) | 0和NaN |
| Object | 任何对象 | null |
| Undefined | n/a | undefined |

```javascript
var message = "Hello world!";
if (message){
    alert("Value is true");
}
```

#### Number 类型
* 可以表示整数和浮点数值
* 最基本的数值字面量格式是十进制整数，整数还可以通过八进制或十六进制来表示
* 八进制字面值的第一位必须是零(0)，然后是八进制数字序列(0~7)，如果字面值中的数值超出了范围，那么前导零将被忽略，后面的数值将被当作十进制数值解析
* 十六进制字面值的前两位必须是 0x，后跟任何十六进制数字(0~9 及 A~F)。字母 A~F 可以大写，也可以小写。
* 在进行算术计算时，所有以八进制和十六进制表示的数值最终都将被转换成十进制数值

```javascript
var octalNum1 = 070; // 八进制的 56
var octalNum2 = 079; // 无效的八进制数值——解析为 79
var octalNum3 = 08; // 无效的八进制数值——解析为 8
var hexNum1 = 0xA; // 十六进制的 10
var hexNum2 = 0x1f; // 十六进制的 31
```

##### 浮点数值
* 极大或极小的数值，可以用 `e` 表示法(即科学计数法)表示的浮点数值表示
* 浮点数值的最高精度是 17 位小数
* 关于浮点数值计算会产生舍入误差的问题，是使用基于 IEEE754 数值的浮点计算的通病，**因此，永远不 要测试某个特定的浮点数值。**

```javascript
var floatNum = 3.125e7; // 等于31250000
var floatNum = 3e-17; // 等于0.00000000000000003
```

##### 数值范围
* ECMAScript 能够表示的最小数值保 存在 `Number.MIN_VALUE` 中，在大多数浏览器中，这个值是 5e-324，相对应的还有 `Number.MAX_VALUE`
* 如果计算的结果得到了一个超出 JavaScript 数值范围的值，那么这个数值将被自动转换成特殊的 `Infinity` 或 `-Infinity`

##### NaN
* NaN，即非数值(Not a Number)是一个特殊的数值，这个数值用于表示一个本来要返回数值的操作数未返回数值的情况(这样就不会抛出错误了)
* 实际上只有 0 除以 0 才会返回 NaN，正数除以 0 返回 Infinity，负数除以 0 返回 -Infinity
* 任何涉及 NaN 的操作(例如 NaN/10)都会返回 NaN
* NaN 与任何值都不相等，包括 NaN 本身
* `isNaN()` 函数，在接收到一个值之后，会尝试 将这个值转换为数值，而任何不能被转换为数值的值都会导致这个函数返回 true。

##### 数值转换
* `Number()` 可以用于任何数据类型。`parseInt()` 和 `parseFloat()` 用于把字符串转换成数值
* 由于 `Number()` 函数在转换字符串时比较复杂而且不够合理，因此在处理整数的时候更常用的是 `parseInt()` 函数

#### String类型
* String 数据类型包含一些特殊的字符字面量，也叫转义序列，用于表示非打印字符，或者具有其他用途的字符。以 `\` 开头。
* 以十六进制代码 nn 表示的一个字符(其中n为0~F)。例如，`\x41` 表示"A"
* 以十六进制代码 nnnn 表示的一个Unicode字符(其中n为0~F)。例如，`\u03a3` 表示希腊字符Σ
* 数值、布尔值、对象和字符串值都有 `toString()`，但是 `null` 和 `undefined` 值没有这个方法
* 转型函数 `String()` 能够将任何类型的值转换为字符串

#### Object类型
* ECMAScript 中的对象其实就是一组数据和功能的集合。对象可以通过执行 new 操作符后跟要创建 的对象类型的名称来创建。
* 需要了解 `constructor`, `hasOwnProperty(propertyName)`, `isPrototypeOf(object)`, `propertyIsEnumerable(propertyName)` 等方法

### 操作符

* 一元操作符，主要用于基本算数运算和转换数据类型
* 位操作
* 布尔操作 `NOT`, `AND`, `OR`

## 不需要复习的知识点

1. 在使用 `<script>` 嵌入 JavaScript 代码时，记住不要在代码中的任何地方出现 `"</script>"` 字符串，因为浏览器会认为是结束标签，可以通过转移符 `/` 来解决这个问题。
2. 带有 `src` 属性的 `<script>` 元素不应该在其 `<script>` 和 `</script>` 标签之间再 包含额外的 JavaScript 代码，包含了也会被忽略。
3. `<noscript>` 在脚本无效的情况下向用户显示一条消息。
4. `typeof` 是关键字，它不能当作函数名。
5. 按照惯例，ECMAScript 标识符采用驼峰大小写格式，也就是第一个字母小写，剩下的每个单词的首字母大写。
6. 单行注释 `//`，多行注释 `/*` 和 `*/`
7. 严格模式 `"use strict";` 可以出现在文件最顶部或函数内部。
8. 省略了 `var` 操作符会生成全局变量
9. 给未经声明的变量赋值在严格模式 下会导致抛出 `ReferenceError` 错误。

```javascript
// 严格模式下，JavaScript 的执行结果会有很大不同
function doSomething(){
  "use strict";
  //函数体
}

// 把多条语句组合到一个代码块中
if (test)
    alert(test); // 有效但容易出错，不要使用

if (test) { // 推荐使用
    alert(test);
}
```

## 其它知识点
### 副效应
执行前置递增和递减操作时，变量的值都是在语句被求值以前改变的。

```javascript
var num1 = 2;
var num2 = 20;
var num3 = --num1 + num2;  // 等于 21，副效应，先执行 递增 操作，改变原始值，再执行计算
var num4 = num1 + num2; // 等于 21

var num1 = 2;
var num2 = 20;
var num3 = num1-- + num2; // 等于 22，先使用原始值计算，再执行 递减 操作
var num4 = num1 + num2; // 等于 21
```


## 参考

* [W3school](http://www.w3school.com.cn/tags/tag_script.asp)
* [MDN JavaScript pages](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Index)
