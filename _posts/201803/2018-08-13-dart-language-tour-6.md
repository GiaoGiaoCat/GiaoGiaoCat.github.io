---
layout: post
title:  "Dart 语言程序设计 6 - 异常"
date:   2018-08-12 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 异常 Exceptions

* Dart 可以抛出和捕捉异常。异常表示发生了意外的事情。如果不捕获异常，一旦异常发生，则程序会中断执行并停止运行。
* Dart 代码不会在代码中指定代码将会丢出什么异常，所以调用系统代码时，不会强制要求处理异常。这点与 Java 的处理方式是不同的。
* Dart 提供 Exception 和 Error 和一些预定义的子类型来处理异常。自己也可以定义属于自己的异常类型。
* Dart 可以抛出任何非空对象，而不仅仅是 Exception 或 Error 对象作为异常。

### Throw

**生产环境通常抛出 Error 或 Exception 类型，而不是抛出其它非空对象。**

先一个抛出异常的例子：

```dart
throw FormatException('Expected at least 1 section');
```

也可以抛出任意对象：

```dart
throw 'Out of llamas!';
```

因为抛出异常也属于表达式，所以可以将 throw 语句放在 `=>` 语句中：

```dart
void distanceTo(Point other) => throw UnimplementedError();
```

### Catch

捕获异常我们才能有机会处理这些异常。

* 可以抛出多个 Exception 但是由第一个 catch 到的分局来处理
* 可以通过 on 语句来指定需要捕获的异常类型，由 catch 来处理异常抛出的对象
* 如果 catch 分局没有自定类型，则可以处理任何类型的抛出对象

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}
```

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```

你可以给 catch 传递 1 或 2 个对象。第一个是抛出的异常，第二个是堆栈跟踪 StackTrace 对象。

```dart
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

### rethrow

有时候我们在处理完异常之后，希望它继续传播。

```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // Runtime error
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // Allow callers to see the exception.
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

### Finally

* 无论是否发生异常，都想要一些代码得以运行，就可以使用 finally 语句。
* 如果没有 catch 到异常，那 finally 语句执行后，传播异常。
* 就算 catch 到异常了，也会在最后执行 finally 部分的代码。

```dart
try {
  breedMoreLlamas();
} finally {
  // Always clean up, even if an exception is thrown.
  cleanLlamaStalls();
}
```

```dart
try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e'); // Handle the exception first.
} finally {
  cleanLlamaStalls(); // Then clean up.
}
```

## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
* [Dart 语言简易教程 - 五](https://www.jianshu.com/p/83adc77839b6)
