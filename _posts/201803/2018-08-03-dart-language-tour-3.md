---
layout: post
title:  "Dart 语言程序设计 3 - 函数"
date:   2018-08-02 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 函数

* Dart 是真正的面向对象的语言，所以函数也是对象有其类型。
* 这意味着我们可以吧一个函数赋值给变量或当作参数传给另外的函数。
* 你也可以像调用一个类的实例对象一样来调用函数。后面的 Callable classes 部分会介绍这一特性。
*

下面介绍如何定义函数。

```dart
// 推荐的方式，建议明确函数的输入类型和返回类型，方便修改，也方便阅读
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

// 忽略类型也仍然有效
isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

// 如果函数只包含一行表达式，可以使用简写的方式，=> 是 { return expr; } 的简写形式
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

### 可选参数

* 函数的参数分为必选和可选参数。
* 定义函数时候需要先列出必选参数，然后列出可选参数。必选参数需要用 `@required` 标志。
* 可选参数可以是 positional 或 named，但不能同时是两者。

#### 可选的 named 参数

定义函数参数的时候，可以使用 `{param1, param2, …}` 的方式定义 nameed 参数。采用 `paramName: value` 的方式调用，例如：

```dart
/// Sets the [bold] and [hidden] flags ...
void enableFlags({bool bold, bool hidden}) {...}

enableFlags(bold: true, hidden: false);
```

使用 `@required` 来标明参数为必填，如果必填参数忘了填，那么分析器会帮我们检查出来。

```dart
const Scrollbar({Key key, @required Widget child})
```

`@required` 是定义在 [meta](https://pub.dartlang.org/packages/meta) 包中，所以在使用前需要先导入 package:meta/meta.dart 或者导入含有 meta 的其他包，比如 Flutter 的 package:flutter/material.dart。

#### 可选的 positional 参数

如果一个参数被 `[]` 包裹住，则这个参数就是可选参数。

```dart
String say(String from, String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```
下面是一些调用的例子：

```dart
assert(say('Bob', 'Howdy') == 'Bob says Howdy');
assert(say('Bob', 'Howdy', 'smoke signal') == 'Bob says Howdy with a smoke signal');
```

#### 默认参数值

函数可以用 = 为参数设置默认值，默认值必须是编译时常量，如果不设置默认值，那么参数的默认值是 null。

```dart
/// Sets the [bold] and [hidden] flags ...
void enableFlags({bool bold = false, bool hidden = false}) {...}

// bold will be true; hidden will be false.
enableFlags(bold: true);
```

```dart
String say(String from, String msg, [String device = 'carrier pigeon', String mood]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  if (mood != null) {
    result = '$result (in a $mood mood)';
  }
  return result;
}

assert(say('Bob', 'Howdy') == 'Bob says Howdy with a carrier pigeon');
```

```dart
void doStuff(
    {List<int> list = const [1, 2, 3],
    Map<String, String> gifts = const {
      'first': 'paper',
      'second': 'cotton',
      'third': 'leather'
    }}) {
  print('list:  $list');
  print('gifts: $gifts');
}
```

### main() 函数

每个应用程序都必须有一个顶级 `main()` 函数，作为应用程序的入口点。`main()` 函数返回void，并具有可选的 `List<String>` 作为参数。

**..操作符是 Dart 语言中的级联操作符，级联操作可以对一个对象执行多个操作。**

```dart
void main() {
  querySelector('#sample_text_id')
    ..text = 'Click me!'
    ..onClick.listen(reverseText);
}
```

下面是接受参数的命令行应用程序的 main() 函数的示例，也可以使用 [arg libray](https://pub.dartlang.org/packages/args) 来定义和解析命令行参数:

```dart
// Run the app like this: dart args.dart 1 test
void main(List<String> arguments) {
  print(arguments);

  assert(arguments.length == 2);
  assert(int.parse(arguments[0]) == 1);
  assert(arguments[1] == 'test');
}
```

### 函数优先原则

```dart
void main() {
  void printElement(int element) {
    print(element);
  }

  var list = [1, 2, 3];

  // Pass printElement as a parameter.
  list.forEach(printElement);
}
```

将函数赋值给变量：

```dart
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
assert(loudify('hello') != '!!! HELLO !!!', 'cool');
```

### 匿名函数

多数情况下函数都有个名字，像 main() 或 printElement() 这样。Dart 中也可以创建没名字的匿名函数，有时候也叫 lambda 或闭包。匿名函数可以赋值给变量。

匿名函数和普通函数一样，可以接收 0 或多个参数，参数中间以逗号分割。

```dart
([[Type] param1[, ...]]) {
  codeBlock;
};
```

下面的示例创建了一个匿名函数，其接收一个名为 item 的参数：

```dart
void main() {
  var list = ['apples', 'bananas', 'oranges'];
  list.forEach((item) {
    print('${list.indexOf(item)}: $item');
  });
}
```

如果函数只有一行表达式，使用箭头语法更好。

```dart
void main() {
  var list = ['apples', 'bananas', 'oranges'];
  list.forEach((item) => print('${list.indexOf(item)}: $item'));
}
```

### 静态作用域

Dart 是有静态作用域的语言，这意味着变量有自己的作用域。下面的例子，简单的解释了每个变量的作用域：

```dart
bool topLevel = true;
void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

### 闭包

闭包就是能够读取其他函数内部变量的函数。例如在 JavaScript 中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成 **定义在一个函数内部的函数**。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。

```dart
/// Returns a function that adds [addBy] to the
/// function's argument.
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}

void main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

### 测试函数的相等性

下面是一些测试 top-level functions, static methods, and instance methods 是否相等的例子。

```dart
void foo() {} // A top-level function

class A {
  static void bar() {} // A static method
  void baz() {} // An instance method
}

void main() {
  var x;

  // Comparing top-level functions.
  x = foo;
  assert(foo == x);

  // Comparing static methods.
  x = A.bar;
  assert(A.bar == x);

  // Comparing instance methods.
  var v = A(); // Instance #1 of A
  var w = A(); // Instance #2 of A
  var y = w;
  x = w.baz;

  // These closures refer to the same instance (#2),
  // so they're equal.
  assert(y.baz == x);

  // These closures refer to different instances,
  // so they're unequal.
  assert(v.baz != w.baz);
}
```

### 返回值

所有函数都有返回值，如果没有指定返回值则默认 `return null;`

```dart
foo() {}
assert(foo() == null);
```

## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
