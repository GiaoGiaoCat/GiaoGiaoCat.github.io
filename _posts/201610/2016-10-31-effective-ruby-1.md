---
layout: post
title:  "Effective Ruby - 1"
date:   2016-10-30 12:30:00
categories: ruby
tags: learningnote
author: "Victor"
---

## 让自己熟悉 Ruby
### 理解 Ruby 中的 `True`
* 除了 `false` 和 `nil` 外所有的值都表示真值
* 如果你需要区分 `false` 和 `nil`，可以使用 `nil?` 方法或 `==` 操作符并将 `false` 作为左操作对象

比如配置文件，`false` 表示应该被禁用，而 `nil` 表示没有显示定义，因此应该使用默认值。故我们需要区分 `false` 和 `nil`。而将 `false` 放在左边意味着 Ruby 会将表达式解析为 `FlaseClass#==` 方法的调用（该方法继承自 `Object` 类），如果 `false` 放在右边是有风险的，因为其它类可能覆盖 `Object#==` 方法。

### 所有对象的值都可能为 `nil`
* 假设任何对象都可以为 `nil`，包括方法参数和调用方法的返回值
* 将变量显示转换为期望的类型比时刻担心其为 `nil` 要容易的多

在产品环境我们经常遇到 `undefined method 'fubar' for nil:NilClass (NoMethodError)` 的错误。我们可以使用上面提到的方法来避免这个情况。

```ruby
# 常见的防守写法
person.save if person

# 转换为期待的类型 nil.to_s #=> ''
def fix_title(title)
  title.to_s.capitalize
end

# 过滤掉 nil
name = [first, middle, last].compact.join(' ')
```

### 避免使用 Ruby 中古怪的 Perl 风格语法
* 推荐使用 `String#match` 替代 `String#=~`
* 使用更长、更表意的全局变量的别名，而非短的、古怪的名字。比如用 `$LOAD_PATH` 代替 `$:`，但是要注意这需要载入 `English` 库，Rails 已经载入这个库。
* 避免使用隐式读写全局变量 `$_` 的方法，比如 `Kernel#print` 等

### 留神，常量是可变的
* 总是将常量冻结
* 如果常量引用了一个集合对象，那么冻结这个集合及其所有元素
* 要防止常量被重新赋值，可以冻结定义它的那个模块

```ruby
# 即便冻结常量，仍然能被修改，只是会有警告信息，所以我们要冻结模块
module Default
  TIMEOUT = "a".freeze
end

Default::TIMEOUT = "b"
#=> warning: already initialized constant Default::TIMEOUT
#=> warning: previous definition of TIMEOUT was here
#=> "b"

module Default
  TIMEOUT = "a"
end
Default.freeze

Default::TIMEOUT = 'b'
#=> RuntimeError: can't modify frozen Module
```

### 留意运行时警告
* 如果你写的是 Ruby 脚本，那么使用 `-w` 命令选项以启用编译和运行时的警告
* 如果是 Rails 程序，可以在运行 Rake 测试的时候启用警告，后面几章会讲
