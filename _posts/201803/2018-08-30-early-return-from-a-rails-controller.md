---
layout: post
title:  "在 controller 中使用 return 的方法"
date:   2018-08-30 13:00:00

categories: rails
tags: tip
author: "Victor"
---

在重构 controller 时，会遇到在 redirecting 或 rendering 之后紧跟 return 的代码，若想把这些代码抽取到单独的地方，如没有掌握正确的写法，反而会带来一些麻烦。

先看下面的例子。

## 经典的 redirect_to and return

```ruby
class Controller
  def show
    unless @order.awaiting_payment? || @order.failed?
      redirect_to edit_order_path(@order) and return
    end

    if invalid_order?
      redirect_to tickets_path(@order) and return
    end

    # even more code over there ...
  end
end
```

不考虑上面的代码是否应该放到 model 或 service 中，仅就本文讨论的重点来看，如何在 controller 内部重构这些代码。

### extracted_method and return

```ruby
class Controller
  def show
    verify_order and return
    # even more code over there ...
  end

  private

  def verify_order
    unless @order.awaiting_payment? || @order.failed?
      redirect_to edit_order_path(@order) and return true
    end

    if invalid_order?
      redirect_to tickets_path(@order) and return true
    end
  end
end
```

这里的问题是，在抽取 `verify_order` 方法之后，需要修改所有 `return` 的返回值，一定是返回 `true`，否则还会引入另外一个 bug。

另外一个违反直觉的地方是 `verify_order and return` 这里，为什么验证通过之后，我还有从方法中返回？难道不应该继续执行下去吗？

### extracted_method or return

```ruby
class Controller
  def show
    verify_order or return
    # even more code over there ...
  end

  private

  def verify_order
    unless @order.awaiting_payment? || @order.failed?
      redirect_to edit_order_path(@order) and return
    end

    if invalid_order?
      redirect_to tickets_path(@order) and return
    end

    return true
  end
end
```

使用 `verify_order or return` 后语义上看起来通顺多了，要么订单被验证，要么立刻从方法中返回（不再继续执行下去）。但是你需要注意的是，用这种方法必须在 `verify_order` 最后一行手动返回 `true`。

### extracted_method{ return }

```ruby
class Controller
  def show
    verify_order{ return }
    # even more code over there ...
  end

  private

  def verify_order
    unless @order.awaiting_payment? || @order.failed?
      redirect_to edit_order_path(@order) and yield
    end

    if invalid_order?
      redirect_to tickets_path(@order) and yield
    end
  end
end
```

这方法的缺点显而易见，首先并不是所有的 Ruby 程序员都很好的掌握了 block 的正确用法。其次，单独看 `verify_order` 方法，并不能知道它会停止代码的执行流程。

**如果一个方法需要根据外部参数才能理解其内部行为，这违背了我们解耦程序的初衷。**

### extracted_method; return if performed?

```ruby
class Controller
  def show
    verify_order; return if performed?
    # even more code over there ...
  end

  private

  def verify_order
    unless @order.awaiting_payment? || @order.failed?
      redirect_to edit_order_path(@order) and return
    end

    if invalid_order?
      redirect_to tickets_path(@order) and return
    end
  end
end
```

使用 `ActionController::Metal#performed?` 可以测试是否有一个 render 或 redirect 发生。这似乎是一个很好的解决方案，**适用于在 render 或 redirect 后将代码提取到只负责中断流的方法中的情况。**我们根本不需要调整提取的方法。代码可以保持原样，也不用在乎该方法的返回值。

## 原文

* [4 ways to early return from a rails controller](https://blog.arkency.com/2014/07/4-ways-to-early-return-from-a-rails-controller/)
