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

### 等式和关系运算符

* `==, !=, >, <, >=, <=` 没啥说的都基本操作。
* 跟 Ruby 一样 `==` 只比较两个对象的值相等。
* 如果需要比较对象是否指向同一个引用，需要用 `identical()` 函数。

### 类型测试运算符

| Operator | Meaning |
| --- | --- |
| as | Typecast |
| is | True if the object has the specified type |
| is! | False if the object has the specified type |

多数情况下推荐使用 `is`。`as` 仅作为一个简写的方法了解一下即可。

```dart
if (emp is Person) {
  emp.firstName = 'Bob';
}

(emp as Person).firstName = 'Bob'; // 如果 emp 为 null 或者不是 Person 的实例，则抛出异常
```

### 赋值运算符

* `=, -=, /=, %=, >>=, ^=, +=, *=, ~/=, <<=, &=, |=` 没啥说的都基本操作。
* `??=` 就像 Ruby 中的 `||=` 一样，只在对象为 null 的时候才赋值。

### 逻辑运算符

* `!expr, ||, &&` 基本操作

### 位操作运算符

* `&, |, ^, ~expr, <<, >>` 基本操作
* `~expr` 一元补位，0s become 1s; 1s become 0s

```dart
final value = 0x22;
final bitmask = 0x0f;

assert((value & bitmask) == 0x02); // AND
assert((value & ~bitmask) == 0x20); // AND NOT
assert((value | bitmask) == 0x2f); // OR
assert((value ^ bitmask) == 0x2d); // XOR
assert((value << 4) == 0x220); // Shift left
assert((value >> 4) == 0x02); // Shift right
```

### 条件表达式

Dart 中只有 false 才是 false

* 支持常见的三元表达式 `condition ? expr1 : expr2`
* `??` 表达式 `expr1 ?? expr2` 当 expr1 为 non-null 的时候返回 expr1，否则返回 expr2。不要和 Ruby 中的 `||` 搞混了，Ruby 中 nil 和 false 均被判断为 false，故 expr1 为 nil 和 false 的时候均返回 expr2，而 Dart 中 expr1 为 false 的时候返回 expr1。

```dart
var visibility = isPublic ? 'public' : 'private';
String playerName(String name) => name ?? 'Guest';
```

上面的表达式也可以写成下面的形式：

```dart
// Slightly longer version uses ?: operator.
String playerName(String name) => name != null ? name : 'Guest';

// Very long version uses if-else statement.
String playerName(String name) {
  if (name != null) {
    return name;
  } else {
    return 'Guest';
  }
}
```

### 级联运算符

严格来说它不是运算符，就是一种语法。`..` 级联操作允许对同一对象进行一系列操作。除了函数调用，还可以访问同一对象上的字段。这通常会省去创建临时变量的步骤，并编写出更容易出错的代码（就像我仍在 jQuery 时代）：

```dart
querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

进一步，还可以用嵌套级联写出更脆弱的代码：

```dart
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

有一种避免代码爆炸的方式：**不要在返回实际对象的函数上调用级联方法。**

### 其它运算符

`(), [], ., ?.` 将在后面的类章节介绍。

## 相关

* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
* [Dart 语言简易教程 - 四](https://www.jianshu.com/p/fdd046a6dc82)
