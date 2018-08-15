---
layout: post
title:  "Dart 语言程序设计 7 -  类和对象"
date:   2018-08-14 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 类 Classes

* Dart 是一种面向对象的语言，有 classes 和基于 mixin 的继承方式
* 所有的对象都是类的实例，所有的类都属于同一个基类 Object
* 基于 mixin 的继承方式意味着，每个类只能有一个超类，但是超类可以被多个不同的子类继承

### Using class members

对象有 functions 和 data（也就是 methods 和 instance variables）。在一个对象上调用方法时，该方法可以访问此对象的其他 functions 和 data。

* 使用 `.` 来引用实例变量和方法
* 使用 `?` 代替 `.` 可以避免因左侧对象是 null 的时候引发异常

```dart
var p = Point(2, 2);
// Set the value of the instance variable y.
p.y = 3;
// Get the value of y.
assert(p.y == 3);
// Invoke distanceTo() on p.
num distance = p.distanceTo(Point(4, 4));
```

```dart
// If p is non-null, set its y value to 4.
p?.y = 4;
```

### 构造函数

可以利用构造函数来创建一个类的实例。构造函数的名称可以是 `ClassName` 或者 `ClassName.identifier`，例如下面的代码就分别用这两种方式创建了两个 Point 的实例：

```dart
var p1 = Point(2, 2);
var p2 = Point.fromJson({'x': 1, 'y': 2});
```

下面的代码也有相同的功能，不过用了我们熟悉的 `new` 关键字，在 Dart 2 中 `new` 是可选的：

```dart
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

一些类提供常量构造函数。要使用常量构造函数创建编译时常量，要将 `const` 关键字放在构造函数名称之前，但有注意创建两个相同的编译时常量的结果：

```dart
var p = const ImmutablePoint(2, 2);

var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);
assert(identical(a, b)); // They are the same instance!
```

在同一上下文中，可以省略构造函数和字面量前的 `const`，看如下代码例子：

```dart
// Lots of const keywords here.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

```dart
// Only one const, which establishes the constant context.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

### 获得对象的类型

想在运行时获得对象的类型，可以使用对象的 runtimeType 属性，它回返回一个 Type 对象。

```dart
print('The type of a is ${a.runtimeType}');
```

### 实例变量

先看如何声明实例变量。

```dart
class Point {
  num x; // Declare instance variable x, initially null.
  num y; // Declare y, initially null.
  num z = 0; // Declare z, initially 0.
}
```

* 所有未初始化的实例变量值均为 null
* 每个实例变量都同时有一个 getter 方法，

```dart
class Point {
  num x;
  num y;
}

void main() {
  var point = Point();
  point.x = 4; // Use the setter method for x.
  assert(point.x == 4); // Use the getter method for x.
  assert(point.y == null); // Values default to null.
}
```

## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
* [Dart 语言简易教程 - 五](https://www.jianshu.com/p/78d317b2ea79)
