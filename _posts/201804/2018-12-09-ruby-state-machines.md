---
layout: post
title:  "Ruby 中的状态机"
date:   2018-11-27 12:00:00

categories: rails
tags: tip
author: "Victor"
---

假设我们的需求是实现信号灯，那么状态就是 红，黄，绿。每个状态都知道自己需要处理的事情：下一步要变成什么颜色的灯。

```ruby
if @light.state == "green"
  @light.play_green_sound
end
if @light.state == "green"
  @light.change_to_yellow
end
```

## 什么是 State Design Pattern

State Design Pattern 是实现状态机的一种形式。它包含 3 个组件：

* 一个 `Context` 类，它知道当前的状态
* 一个 `State` 类，它定义了各个状态需要实现的方法
* 每个状态一个类，这些类继承自 `State` 类

### 优势

每个状态都了解自己，所以也不需要检查当前状态。而较多的条件语句往往是代码复杂的根源之一。


## 使用状态机重构信号灯

```ruby
class TrafficLight
  def initialize
    @state = nil
  end

  def next_state(klass = Green)
    @state = klass.new(self)
    @state.beep
    @state.start_timer
  end
end
```

```ruby
class State
  def initialize(light)
    @light = light
  end

  def beep
  end

  def next_state
  end

  def start_timer
  end
end
```

注意 `State` 我们定义了三个空方法，在其它编程语言中，这种接口形式的代码很常见，但 Ruby 中的习惯可不是这样。在这里就是为了演示方便。

可以看到，我们需要在所有的状态之间共享 `initialize` 方法。因为它们都需要知道 `TrafficLight` 实例对象的上下文，以便发出状态变化的信号。

因为三个状态类的代码差不多，下面就不全写了。

```ruby
class Green < State
  def beep
    puts "Color is now green"
  end

  def next_state
    @light.next_state(Yellow)
  end

  def start_timer
    sleep 5; next_state
  end
end
```

现在所有的状态都知道如何切换到下一个状态了。

## AI Game Example

你可以用状态机解决依赖于当前状态的游戏，比如 [RubyWarrior](https://github.com/ryanb/ruby-warrior)。

在 RubyWarrior 中你拥有一个 player object 和一个 board。

你的目标是：

1. 击败 board 上的所有敌人
2. 到达出口，并且 HP 保持 0 以上

你一次可以做一个动作，这种游戏想要完成挑战需要做一个好的抉择。而查看当前状态有助于你作出选择，这也是我们要使用状态机的理由。

```ruby
class Attacking < State
  def play(warrior)
    warrior.attack!
    @player.set_state(Healing) unless enemy_found?(warrior)
  end
end
```

这是我们的战士可能处于的状态之一，当我们四周没有敌人时，进入愈合状态 Healing，从战斗中恢复过来。

## Using The AASM Gem

[AASM](https://github.com/aasm/aasm) 是 Ruby 世界中很有名的状态机 gem。它的理念是围绕着 events 来创建，所谓的 events 就像是按下电灯开关的事件一样。event 触发转换到其它状态。

```ruby
require 'aasm'
class Light
  include AASM

  aasm do
    state :on, :off

    event :switch do
      transitions :from => :on, :to => :off, :if => :on?
      transitions :from => :off, :to => :on, :if => :off?
    end
  end
end
```

```ruby
light = Light.new
p light.on? #=> true

light.switch
p light.on? #=> false
```

上文中的状态机表示，你只能在当前状态是 off 的时候转换到 on 状态。你可以在状态转换期间增加各种各样的回调，以便实现一些更复杂的逻辑：

* Sending an email
* Logging the state change
* Updating a live monitoring dashboard

AASM 也可以把当前状态通过 `ActiveRecord` 储存到数据库中。

## 原文链接

* [How to Use State Machines in Ruby](https://www.rubyguides.com/2018/12/state-machines/)
