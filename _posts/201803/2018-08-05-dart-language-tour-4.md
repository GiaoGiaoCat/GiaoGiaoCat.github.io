---
layout: post
title:  "Dart 语言程序设计 4 - 操作符"
date:   2018-08-05 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 操作符

这里有 [一个表格](https://www.dartlang.org/guides/language/language-tour#operators) 列出了 Dart 支持的操作符。

* 注意这个表格中各操作符所在的行数，也代表了其优先级，越靠上的优先级越高。但是和其它语言一样，尽量还是使用小括号来区分操作符的运算顺序以便阅读理解。
* 最左侧的操作数决定了使用哪个对象版本的运算符。5
* 后面的章节也介绍了重新其中部分操作符的方法。

```dart
```

```dart
a++
a + b
a = b
a == b
c ? a : b
a is T
```

```dart
// Parentheses improve readability.
if ((n % i == 0) && (d % i == 0)) ...
// Harder to read, but equivalent.
if (n % i == 0 && d % i == 0) ...
```

### 算术运算符

所有语言均有 `+, -, -expr, *, /, ~/, %` 但下面以 Ruby 为例举出不同，这里 dart 的答案更符合直觉。

```dart
assert(5 / 2 == 2.5) // in ruby: 5.0 / 2 == 2.5
assert(5 ~/ 2 == 2) // in ruby: 5 / 2 == 2
```

还有学 C 语言时候我一直搞不清的前缀和后缀表达式。

| Operator | Meaning |
| --- | --- |
| ++var | var = var + 1 (expression value is var + 1) |
| var++ | var = var + 1 (expression value is var) |
| --var | var = var - 1 (expression value is var - 1) |
| var-- | var = var - 1 (expression value is var) |

```dart
var a, b;

a = 0;
b = ++a; // Increment a before b gets its value.
assert(a == b); // 1 == 1

a = 0;
b = a++; // Increment a AFTER b gets its value.
assert(a != b); // 1 != 0

a = 0;
b = --a; // Decrement a before b gets its value.
assert(a == b); // -1 == -1

a = 0;
b = a--; // Decrement a AFTER b gets its value.
assert(a != b); // -1 != 0
```


## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
* [Dart 语言简易教程 - 四](https://www.jianshu.com/p/fdd046a6dc82)
