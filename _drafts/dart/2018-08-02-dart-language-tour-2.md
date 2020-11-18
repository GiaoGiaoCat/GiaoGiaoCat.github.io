---
layout: post
title:  "Dart 语言程序设计 2 - 数据类型"
date:   2018-08-02 13:00:00

categories: dart
tags: learningnote
author: "Victor"
---

## 数据类型

Dart 支持的数据类型有：numbers, string, booleans, lists, maps, runes, symbols。由于 Dart 中的变量实际上是对象的引用或类的实例，所以可以使用构造函数初始化变量。一些内置类型有自己的构造函数，例如 `Map()`。

### Numbers
int 和 double 都是 num 类的子类。可以做一些基本的算术运算以及 `abs(), ceil(), floor()` 或诸如位移操作之类的。如果你需要的方法没找到，可以看看 dart:math 库。

#### int
Integer 值取决于平台类型，但不会超过 64 bits。Dart VM 的值范围是 -2 的 63 次方道 2 的 63 次方 - 1。并会编译成 JavaScript numbers 类型，其值为 -2 的 53 次方道 2 的 53 次方 - 1。

#### double
64位双精度浮点数，符合 IEEE 754 标准

```dart
// int 是没有小数点的
int x = 1;
int hex = 0xDEADBEEF;

// double 是有小数点的
double y = 1.1;
double exponents = 1.42e5;

// 数字和字符串之间的转换
// String -> int
var one = int.parse('1');
assert(one == 1);
// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);
// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');
// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');

// 位移运算 bitwise shift (<<, >>), AND (&), and OR (|) operators.
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 >> 1) == 1); // 0011 >> 1 == 0001
assert((3 | 4) == 7); // 0011 | 0100 == 0111
```

复习一下， 许多算术表达式也可以赋值给编译时常量，只要参与算术的都是常量即可。

```dart
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond;
```

### String

* Dart 中的字符串是 UTF-16 的编码单元。可使用双引号和单引号来定义。
* Dart 中的字符串内嵌表达式用的是 `${expression}` 方式。如果 expression 可被识别，也可以不写 `{}`。Dart 会在 expression 对象上调用 `toString()` 方法。
* 也可使用 `+` 将两个字符串连接到一起
* 可以使用三个单引号或双引号来创建多行的字符串
* 用 `r` 来创建原生字符串，后面会详细介绍如何用 Runes 来创建含有 Unicode 字符的字符串

```dart
var s = 'string interpolation';
assert('Dart has $s, which is very handy.' ==
    'Dart has string interpolation, ' +
        'which is very handy.');

var s1 = '''
You can create
multi-line strings like this one.
''';

var s = r'In a raw string, not even \n gets special treatment.';
```

只要插入表达式的计算结果为 null, numeric, string, boolean 都可以赋值给 const 编译时常量。

```dart
// These work in a const string.
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

// These do NOT work in a const string.
var aNum = 0;
var aBool = true;
var aString = 'a string';
const aConstList = [1, 2, 3];

const validConstString = '$aConstNum $aConstBool $aConstString';
// const invalidConstString = '$aNum $aBool $aString $aConstList';
```

### Booleans
Dart 的类型安全意味着不能使用 `if (nonbooleanValue) or assert (nonbooleanValue)` 这样的代码。如果要做检查，可以参考下面的用例：

```dart
// Check for an empty string.
var fullName = '';
assert(fullName.isEmpty);

// Check for zero.
var hitPoints = 0;
assert(hitPoints <= 0);

// Check for null.
var unicorn;
assert(unicorn == null);

// Check for NaN.
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

### Lists
在 Dart 中的数组是 List objects，所以这里不叫 Array 而叫 List。列表类型有许多很有用的方法，具体可以看 Generics 和 Collections 的文档。

```dart
// 这里分析器会推断 list 的类型为 List<int>，所以如果将来为其追加非 int 类型的对象，则分析器会引发运行时错误
var list = [1, 2, 3];

assert(list.length == 3);
assert(list[0] == 1);
list[1] = 5
assert(list[1] == 5);
```

需要创建一个列表类型的编译时常量，需要在列表前增加 const

```dart
var constantList = const [1, 2, 3];
// constantList[1] = 1; // Uncommenting this causes an error.
```

### Maps

分析器会推导 Map 的类型，如果传递与推导类型不符的键值对会出发分析器或运行时错误。Map 许多很有用的方法，具体可以看 Generics 和 Maps 的文档。

```dart
// 分析器会推导 gifts 的类型是 Map<String, String>
var gifts = {
  // Key:    Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings'
};

// 分析器会推导 gifts 的类型是 Map<int, String>
var nobleGases = {
  2: 'helium',
  10: 'neon',
  18: 'argon',
};
```

```dart
// new Map() 可以简写成 Map()
var gifts = Map();
gifts['first'] = 'partridge';

var nobleGases = Map();
nobleGases[2] = 'helium';

var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds'; // Add a key-value pair, just as you would in JavaScript

var gifts = {'first': 'partridge'};
assert(gifts['first'] == 'partridge');

var gifts = {'first': 'partridge'};
assert(gifts['fifth'] == null); // If you look for a key that isn’t in a map, you get a null in return

assert(gifts.length == 2);
```

创建一个 map 类型的编译时常量，也要在 map 字面量前面标注 const

```dart
final constantMap = const {
  2: 'helium',
  10: 'neno',
  18: 'argon',
};
// constantMap[2] = 'Helium'; // Uncommenting this causes an error.
```

### Runes

* Dart 字符串是 UTF-16 所以需要特殊语法在字符串中表示 UTF-32 的值，runes 用来表示 UTF-32 code point
* 当在 runes 中使用 list 的时候，一定要万分小心，不同的语言和字符集很容易把代码搞崩。
* 使用 `\uXXXX` 方式表示 UTF-16，4 hex digits 以上的用 `\u{XXXXX}` 表示 UTF-32。
* String 类有一些属性可以用来提取 rune 信息，也可以使用 codeUnitAt 和 codeUnit 获取 16-bit code unit

```dart
main() {
  var clapping = '\u{1f44f}';
  print(clapping);
  print(clapping.codeUnits);
  print(clapping.runes.toList());

  Runes input = new Runes(
      '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
  print(new String.fromCharCodes(input));
}
```

### Symbols
在 Dart 中符号用 `#` 开头来表示，入门阶段不需要了解这东西，可能永远也用不上。

## 相关

* [SPDY](https://baike.baidu.com/item/SPDY/3399551?fr=aladdin)
* [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)
* [Dart 学习笔记](http://www.cndartlang.com/dart/page/4)
