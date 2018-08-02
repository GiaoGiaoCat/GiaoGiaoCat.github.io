---
layout: post
title:  "Dart 语言程序设计 1"
date:   2018-08-01 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

Dart 的目标是结构化的 Web 开发语言。既可以快速原型开发，又可以组织大型代码库。既可以用在桌面版和移动版的浏览器中，也可以在服务器端使用。

* 本质是动态类型语言，可选类型，在动态语言的基础上结合了静态语言的优点
* 类和接口是统一的，每个类都有一个隐士接口
* 工厂构造函数和命名构造函数、getter/setter 方法、语言级别的级联调用
* 面向对象和并发编程
* 语言规范、虚拟机、类库和工具

## 入门

安装 https://webdev.dartlang.org/tools/sdk#install

```
brew tap dart-lang/dart
brew install dart
```

安装 vscode 的 Dart 插件 https://marketplace.visualstudio.com/items?itemName=Dart-Code.dart-code

### A basic Dart program

下面是一些 Dart 的最基本特性：

```dart
// Define a function
printInteger(int aNumber) {
  print('The number is $aNumber'); // Print to console
}

// This is where the app starts executing
main() {
  var number = 42; // Declare and initialize a variable
  printInteger(number); // Call a function
}
```

* 注释风格与 JavaScript 相同 `// This is a comment`
* 可选类型，但这个例子中指定了变量类型为 `int`
* 用单引号或双引号来声明字符串
* 字符串插值的方式可用 `$variableName` 或 `${expression}`
* `main()` 类似 C 语言，程序开始的唯一入口
* `var` 一种声明变量而不指定其类型的方法

### 重要概念

当你学习Dart语言时，记住这些事实和概念:

* 变量中的所有东西都是一个对象，每个对象都是一个类的实例。`numbers, functions, null` 都是对象。所有对象继承自 Object 类。
* Dart 是强类型的，但类型可选。因为 Dart 可以做类型推断。当不想明确指定类型时，可以使用动态类型。
* 支持泛型，例如 `List<int>` 一组整型；`List<dynamic>` 一组任意类型的对象。
* Dart 支持顶级函数如 `main()`，以及绑定到类或对象的静态方法和实例方法。也可以在函数内部创建嵌套函数或局部函数。
* 同样也支持顶级变量，以及绑定到类和对象的静态变量和实例变量。实例变量有时称为字段或属性。
* 需要注意的是 Dart 没有所谓的 `public, protected, private` 关键词。如果一个标识符以 _ 开头，就表示它对其库是私有的。
* 标识符可以以字母下划线开头，后跟字符加数字任意组合
* 警告只是表明你的代码可能无法工作，但并不妨碍你的程序执行。错误可以是编译时错误，也可以是运行时错误。编译时错误根本阻止代码执行；运行时错误导致代码执行时引发异常。

泛型是指在强类型语言中编写代码时定义一些可变部分。泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。泛型方法，指在调用时可以接收不同类型的参数。

### 保留字

下表列出了 [Dart 语言的保留字](https://www.dartlang.org/guides/language/language-tour)。

### 变量

```dart
// 创建并初始化一个变量
var name = 'Bob';
dynamic name = 'Bob';
String name = 'Bob';
```

变量存储引用。上面的语句指，名为 name 的变量包含对值为 'Bob' 的字符串对象的引用。该变量被推断为字符串，我们也可以通过显示指定的方式来更改类型。如果对象不限于单个类型，按照设计指南的说法可以将指定为 Object 或 动态类型。

当然我们也可以显示声明对象的类型。

#### 默认值

```dart
int lineCount;
assert(lineCount == null); // assert() 在生产环境中会被忽略，在开发环境抛出异常。
```

在 Dart 中所有未赋值的初始化变量的值均为 `null`，不论该变量是什么类型。因为一切皆对象。

#### Final 和 const

`final` 或 `const` 来声明常量，并且可忽略 `var` 和类型。

* `final` 只能设置一次，可以用变量来初始化
* top-level `final` 或 类变量只在第一次使用时被初始化
* **实例变量可以是 final 但不能是 const**

```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';
final time = new DateTime.now(); // Ok

const bar = 1000000; // Unit of pressure (dynes/cm2)
const double atm = 1.01325 * bar; // Standard atmosphere
const time = new DateTime.now(); //Error，new DateTime.now()不是const常量

name = 'Alice'; // Error: a final variable can only be set once.
```

* `const` 是编译时常量，只能用编译时常量来初始化
* 如果 `const` 为类级别，则称为 `static const`
* `const` 变量等号右边的值可以是：number 或 string 的字面量，其它的 const variable，常数运算结果
* `const` 也可以用来定义常数值，该常数值可以赋值给其它变量，参考下面的例子

```dart
var foo = const [];
final bar = const [];
const baz = []; // Equivalent to `const []`
```

最后，只要这个变量不是 `final` 或 `const` 声明的，那么即便它曾经被赋值了一个常量，你也可以在需要的时候重新给这个变量赋值。如下：

```dart
var foo = const [];
foo = [1, 2, 3]; // Was const []
```


## 相关

* [SPDY](https://baike.baidu.com/item/SPDY/3399551?fr=aladdin)
* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
