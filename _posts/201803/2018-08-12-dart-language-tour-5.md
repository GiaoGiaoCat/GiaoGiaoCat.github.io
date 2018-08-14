---
layout: post
title:  "Dart 语言程序设计 5 - 流程控制"
date:   2018-08-12 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 流程控制

可以使用以下任一项来控制 Dart 代码的流程：`if, else, for, while, do-while, break, continue, switch, case, asset` 以及后面会介绍的 `try-catch, throw`。

### if else

```dart
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

### for 循环

```dart
var message = StringBuffer('Dart is fun');
for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

* dart 可正确捕获 for 循环中的闭包的值，避免了一个 JavaScript 常见的陷阱
* 如果对象是可迭代的，并且不太关心当前迭代的计数器，那么使用 `forEach()` 方法更好
* 像 List 或 Set 这样可迭代的类，均支持 `for-in` 迭代形式

```dart
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}
callbacks.forEach((c) => c());
// dart output 0 then 1, javascript print 2 then 2.
```

```dart
candidates.forEach((candidate) => candidate.interview());
```

```dart
var collection = [0, 1, 2];
for (var x in collection) {
  print(x); // 0 1 2
}
```

### while 和 do-while

* while 先判断条件，再执行循环
* do-while 先执行一次循环，再判断条件是否满足再次循环

```dart
while (!isDone()) {
  doSomething();
}

do {
  printLine();
} while (!atEndOfPage());
```

### break 和 continue

* break 中断循环
* continue 跳过本次执行，进入下次迭代

```dart
while (true) {
  if (shutDownRequestd()) break;
  processIncomingRequests();
}

for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

在 List 和 Set 上使用迭代方法来重新上面的例子，看起来会舒服多了：

```dart
candidates
  .where((c) => c.yearsExperience >= 5)
  .forEach((c) => c.interview());
```

### switch 和 case

* Dart 中的 switch/case 语句使用 `==` 操作符来比较整数，字符串和编译时常量
* 比较的对象必须是同一个类的实例（不能是该类的子类实例），并且该类不能重写 `==` 方法
* 枚举类型配合 switch 服用效果很好
* 每个非空的 case 子句需要以 `break` 结尾，其它有效结束非空 case 子句的方式还有 `continue, throw, return`
* 当没有 case 子句匹时，执行 default 子句的代码

```dart
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
    break;
  case 'PENDING':
    executePending();
    break;
  case 'APPROVED':
    executeApproved();
    break;
  case 'DENIED':
    executeDenied();
    break;
  case 'OPEN':
    executeOpen();
    break;
  default:
    executeUnknown();
}
```

Go 里面 switch 默认相当于每个 case 最后带有 break，匹配成功后不会自动向下执行其他 case，而是跳出整个 switch, 但是可以使用 fallthrough 强制执行后面的 case 代码。

* 空的 case 语句会 fall-through，这个跟 go 语言的类似
* 配合 continue 语句也可以实现类似 goto 的功能
* case 子句可以有局部变量，该变量仅在该子句的范围内可见

```dart
var command = 'CLOSED';
switch (command) {
  case 'CLOSED': // Empty case falls through.
  case 'NOW_CLOSED':
    // Runs for both CLOSED and NOW_CLOSED, just print 1.
    print('1');
    break;
  case 'OPEN':
    print('2');
    break;
}
```

```dart
var command = 'CLOSED';
switch (command) {
  case 'CLOSED':
    executeClosed();
    continue nowClosed;
  // Continues executing at the nowClosed label.

  nowClosed:
  case 'NOW_CLOSED':
    // Runs for both CLOSED and NOW_CLOSED.
    executeNowClosed();
    break;
}
```

### assert

如果布尔条件为 `false`，`assert` 语句中断正常执行流程。**仅在开发模式下有效**

```dart
// Make sure the variable has a non-null value.
assert(text != null);

// Make sure the value is less than 100.
assert(number < 100);

// Make sure this is an https URL.
assert(urlString.startsWith('https'));

assert(urlString.startsWith('https'),
    'URL ($urlString) should start with "https".');
```



## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
* [Dart 语言简易教程 - 五](https://www.jianshu.com/p/83adc77839b6)
