---
layout: post
title:  "Growing Rails Applications in Practice - 1"
date:   2015-01-24 17:30:00
categories: rails
tags: learningnote
author: "Victor"
---

## New rules for Rails

### Beautiful controllers

* 很难决定把一个新功能是写在控制器还是写在模型中。是由模型来完成发送邮件通知的功能还是由控制器处理呢？处理模型和用户界面差异的代码放在哪里呢？
* 模型和屏幕之间的自定义映射需要太多的控制器代码。比如，控制器操作多个模型；表单中含有模型中不存在的附加字段；
* 控制器中的代码难以测试和试验，因为运行控制器需要一个复杂的环境(request, params, sessions等等)。
* 由于没有一个缺乏明确的指导方针教大家如何设计的控制器，以致于没有两个控制器是完全一样的。这使得在现有的 UI 上工作一件苦差事，因为你必须要了解数据是如何在每个控制器之间传递的。

我们不能干掉控制器。但是遵循一些简单的原则，可以减少控制器在整个项目中的重要性，并把控制代码放到一个更好的地方。在控制器中存在的商业逻辑越少越好。

#### Normalizing user interactions

所有的控制器方法都应该对应成 CRUD，即便看起来不像 CRUD 的也可以对应的上。

#### The case for consistent controller design

将控制器的方法集中成 CRUD 的好处是：我们不再关注控制器，只需要重点考虑 models 和 views 就行。

#### A better controller implementation

```ruby
class NotesController < ApplicationController

  def index
    load_notes
  end

  def show
    load_note
  end

  def new
    build_note
  end

  def create
    build_note
    save_note or render 'new'
  end

  def edit
    load_note
    build_note
  end

  def update
    load_note
    build_note
    save_note or render 'edit'
  end

  def destroy
    load_note
    @note.destroy
    redirect_to notes_path
  end

  private

  def load_notes
    @notes ||= note_scope.to_a
  end

  def load_note
    @note ||= note_scope.find(params[:id])
  end

  def build_note
    @note ||= note_scope.build @note.attributes = note_params
  end

  def save_note
    if @note.save
      redirect_to @note
    end
  end

  def note_params
    note_params = params[:note]
    note_params ? note_params.permit(:title, :text, :published) : {}
  end

  def note_scope
    Note.scoped
  end
end
```

#### Why have controllers at all?

* Security (authentication, authorization)
* Parsing and white-listing parameters
* Loading or instantiating the model
* Deciding which view to render

#### A note on controller abstractions

There are gems like [Inherited Resources](https://github.com/josevalim/inherited_resources) or [Resource Controller](https://github.com/makandra/resource_controller) that generate a uniform con- troller implementation for you. E.g. the following code would give you a fully implemented UsersController with the [seven RESTful default actions](http://guides.rubyonrails.org/routing.html#resource-routing-the-rails-default) with a single line of code:

When Ruby loads the UsersController class, Resource Controller would dynamically generate a [default implementation](https://makandracards.com/makandra/637-default-implementation-of-resource_controller-actions) for your controller actions.

使用 gem 来优化 controller 很好，但是当团队来了新队员的时候，他需要花更多的时间弄懂这个 gem，请自己权衡利弊。

## Relearning ActiveRecord

### Understanding the ActiveRecord lifecycle

关于 ActiveRecord 有一个隐藏的陷阱，它让你以为你可以按照自己的想法写出包含一些方法的，看起来很简单的类。但是实际上，你必须严格按照 ActiveRecord 约定的风格，你的代码才能有效。

Say you have an Invite model. Invites are created without an associated User, and can be accepted later. When an invite is accepted, the current user is assigned and a Membership is created. So we provide an accept! method that does all of that:

```ruby
class Invite < ActiveRecord::Base

  belongs_to :user

  def accept!(user)
    self.user = user
    self.accepted = true
    Membership.create!(:user => user)
    save!
  end
end
```

The problem with the code above is that there are a dozen ways to circumvent the ``accept!`` method by setting the ``user`` or ``accepted`` attribute with one of ActiveRecord’s many auto-generated methods. For instance, you cannot prevent other code from simply changing the ``accepted`` flag without going through ``accept!``:

```ruby
invite = Invite.find(...)
invite.accepted = true
invite.save!
```

很不幸，现在 ``Invite`` 有了一个奇怪的状态：它已经 ``accepted`` 了，但是没有用户和它关联，也缺少 ``membership``。我们很容易构造出这样的数据：

```ruby
invite.update_attributes!(accepted: true)
invite[:accepted] = true; invite.save
Invite.new(accepted: true).save
Invite.create!(accepted: true)
```

在 [Objects on Rails](http://objectsonrails.com/) 一书中，作者称其为 **infinite protocol**。

这意味着 Invite 类很容易被其他错误的方法调用破坏了数据完整性。记住，面向对象的基本原则之一就是 **让你的模型难以被滥用(Make it hard to misuse your model)**。

consider an alternative implementation of the Invite class:

有一种解决方案 [get rid of all the “bad” methods that ActiveRecord generates for you](https://github.com/objects-on-rails/fig-leaf)。留下你可以控制的白名单方法，但是我们推荐另一种解决方案：利用 ActiveRecord 提供的 API PI，使用 ``setters``， ``create!``, ``update_attributes!`` 更合适。

不是跟 ActiveRecord 对着干，而是为 Invite 类实现一种替代方案：

```ruby
class Invite < ActiveRecord::Base

  belongs_to :user

  validates_presence_of :user_id, :if => :accepted?

  after_save :create_membership_on_accept

  private

  def create_membership_on_accept
    if accepted? && accepted_changed?
      Membership.create!(:user => user)
    end
  end
end
```

上面的代码通过验证和回调来保证数据完整性。这样的模式，开发者可以自由使用任何 ActiveRecord 方法来操作它的状态。

**在你提供的自定义模型方法中绝不要依赖其他代码，坚持在回调和验证中使用 ActiveRecord 提供的 API。**

你可能想知道，对于看起来不执行回调的方法怎么办，比如上面的 ``accept!``。没问题， 你可以在模型中随意添加这些方法，便于读取和修改记录。再次重申在这些方法中不要依赖其它代码，总是强迫使用回调和验证来确保数据的完整性。

### Aren’t callbacks evil?

如果你是一位有经验的 Rails 程序员，你肯定遇到过回调过多的坑。如果你尝试过这种通过，并想说明要尽量少的使用回调，别着急，后面的章节会讨论这一点。

### The true API of ActiveRecord models

当你开始构造模型的验证和回调行为时，你会发现模型的 API 看起来如下：

* 用任何能满足你需求的 API 来实例化一条记录。
* 用任何能满足你需求的 API 来操作一条记录。
* 操作结果不会自动保存到数据库，相反它进入一种 **dirty** 状态。你可以检查脏数据中的错误或者预览数据修改结果来决定是否保存修改到数据库。
* 当操作记录通过验证，所有的修改通过一次操作保存进入数据库。

一旦你接受只使用回调来表达模型限制的做法，你可以收获许多好处：

* 借助 ActiveRecord 的错误追踪，你的视图会大大简化。不需要在控制器中读写变量来显示错误状态。无效的表单字段会由 Rails 表单辅助方法自动高亮。
* 开发人员不需要再了解每一个模型的API。可以很容易的理解每个模型是如何验证各个字段并产生什么行为的。
* 不会再因意外误用一个模型，然后产生一条处于意外状态的记录。
* 你的控制器也因此变瘦小。
* 你会发现有许多有用的类库可以和标准的 ActiveRecord API 协同工作。比如：[state_machine](https://github.com/pluginaweek/state_machine) 提供状态机功能。 [paper_trail](https://github.com/airblade/paper_trail) 在 ActiveRecord 的生命周期中增加了钩子用来监控数据改变。

注意，ActiveRecord 模型并不是唯一的。我们下面来看看非持续存在的类如何工作。

## User interactions without a database
