---
layout: post
title:  "Growing Rails Applications in Practice - 1"
date:   2015-01-24 17:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## New rules for Rails

### Beautiful controllers

刚接触 MVC 的开发者，最头疼的问题大概是搞不懂 controller 的作用：

* 很难决定把一个新功能是写在 controller 还是写在 model 中。是由 model 来完成发送邮件通知的功能还是 controller 的责任呢？支撑 model 和用户界面差异的代码放在哪里呢？
*  model 和显示界面之间的自定义映射需要太多的 controller 代码。比如， controller 操作多个 model ；表单中含有 model 中不存在的附加字段；
*  controller 中的代码难以测试和试验，因为运行 controller 需要一个复杂的环境(request, params, sessions等等)。
* 由于没有一个明确的指导方案教大家如何设计 controller，以致于没有两个 controller 是完全一样的。这使得在现有的 UI 上工作一件苦差事，因为你必须要了解数据是如何在每个 controller 之间传递的。

我们不能干掉 controller 。但是遵循一些简单的原则，可以减少 controller 在整个项目中的重要性，并把 controller 的代码放到一个更好的地方。 **在 controller 中存在的商业逻辑越少越好。**

#### Normalizing user interactions

**所有的 controller 方法都应该对应成 CRUD，即便看起来不像 CRUD 的也可以对应的上。**

#### The case for consistent controller design

将 controller 的方法集中成 CRUD 的好处是：我们不再关注 controller ，只需要重点考虑 models 和 views 就行。

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

There are gems like [Inherited Resources](https://github.com/josevalim/inherited_resources) or [Resource Controller](https://github.com/makandra/resource_controller) that generate a uniform controller implementation for you. E.g. the following code would give you a fully implemented UsersController with the [seven RESTful default actions](http://guides.rubyonrails.org/routing.html#resource-routing-the-rails-default) with a single line of code:

```ruby
class UsersController < ResourceController::Base
end
```

When Ruby loads the ``UsersController`` class, Resource Controller would dynamically generate a [default implementation](https://makandracards.com/makandra/637-default-implementation-of-resource_controller-actions) for your controller actions.

使用 gem 来优化 controller 很好，但是当团队来了新队员的时候，他需要花更多的时间弄懂这个 gem，请自己权衡利弊。

## Relearning ActiveRecord

### Understanding the ActiveRecord lifecycle

关于 ActiveRecord 有一个隐藏的陷阱，它让你以为你可以按照自己的想法写出包含一些方法的，看起来很简单的类。但是实际上，你必须严格按照 ActiveRecord 约定的风格，你的代码才能有效。

Say you have an ``Invite`` model. Invites are created without an associated User, and can be accepted later. When an invite is accepted, the current user is assigned and a Membership is created. So we provide an ``accept!`` method that does all of that:

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

这意味着 ``Invite`` 类很容易被其他错误的方法调用，从而破坏了数据完整性。记住，面向对象的基本原则之一就是 **让你的 model 难以被滥用(Make it hard to misuse your model)**。

一种解决方案是 [get rid of all the “bad” methods that ActiveRecord generates for you](https://github.com/objects-on-rails/fig-leaf)，也就是留下你可以控制的白名单方法。但是我们推荐另一种解决方案：利用 ActiveRecord 提供的 API，使用 ``setters``， ``create!``， ``update_attributes!`` 更合适。

不是跟 ActiveRecord 对着干，而是为 ``Invite`` 类实现一种替代方案：

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

上面的代码通过验证和回调来保证数据完整性。这样的模式，开发者可以随意使用任何 ActiveRecord 方法来操作它的状态。

**在你提供的自定义 model 方法中绝不要依赖其他代码，坚持在回调和验证中使用 ActiveRecord 提供的 API。**

你可能想知道，是否有不执行回调的方法，比如上面的 ``accept!``。放心，你可以在 model 中随意添加这些便利的方法，提高我们读取和修改记录时的安全性。再次重申在这些方法中不要依赖其它代码，总是强迫使用回调和验证来确保数据的完整性。

### Aren’t callbacks evil?

如果你是一位有经验的 Rails 程序员，你肯定遇到过回调过多的坑。如果你想告诉别人在项目中要尽量少的使用回调，别着急，后面的章节会讨论这一点。

### The true API of ActiveRecord models

当你开始构造 model 的验证和回调行为时，你会发现 model 的 API 看起来如下：

* 用任何能满足你需求的 API 来实例化一条记录。
* 用任何能满足你需求的 API 来操作一条记录。
* 操作结果不会自动保存到数据库，相反它进入一种 **dirty** 状态。你可以检查脏数据中的错误或者预览数据修改结果来决定是否保存修改到数据库。
* 当操作记录通过验证，所有的修改通过一次操作保存进入数据库。

一旦你接受只使用回调来表达 model 限制的做法，你可以收获许多好处：

* 借助 ActiveRecord 的错误追踪，你的视图会大大简化。不需要在 controller 中读写变量来显示错误状态。无效的表单字段会由 Rails 表单辅助方法自动高亮。
* 开发人员不需要再了解每一个 model 的 API。可以很容易的理解每个 model 是如何验证各个字段并产生什么行为的。
* 不会再因意外误用一个 model ，然后产生一条处于意外状态的记录。
* 你的 controller 也因此变瘦小。
* 你会发现有许多有用的类库可以和标准的 ActiveRecord API 协同工作。比如：[state_machine](https://github.com/pluginaweek/state_machine) 提供状态机功能。 [paper_trail](https://github.com/airblade/paper_trail) 在 ActiveRecord 的生命周期中增加了钩子用来监控数据改变。

注意，ActiveRecord model 并不是唯一的。我们下面来看看无需持久化的类如何工作。

## User interactions without a database

在前面的章节中，我们重点展示了 ActiveRecord 的错误信息处理，使得它能有效的和用户交互。

但是 model 不一定和数据库表对应。有许多 UI 交互不会导致数据库中的数据改变。例如：

* A sign in form
* A search form
* A payment form like Stripe’s where the user enters a credit card, but the credit card data is
stored on another server (using an API call)

我们都喜欢 ActiveRecord 风格的 model ，毕竟 ActiveRecord 有很多方便的特点：

* Validations and form roundtrips, where invalid fields are highlighted and show their error message.
* Attribute setters that cast strings to integers, dates, etc. (everything is a string in the params).
* Language translations for model and attribute names.
* Transactional-style form submissions, where an action is only triggered once all validations
pass.

这一章展示如何配置 ActiveModel，能让我们享用全部 ActiveRecord 的优势，而又不用关心数据表。

### Writing a better sign in form

每个人都写过用户登录的表单，它看起来和注册用户差不多。当输入错误数据的时候，显示错误信息。两者的区别是：表单提交后，当数据验证通过，登录操作并没有和数据库交互，而是修改了 session。

如果我们要实现这个功能，通常是创建一个新的 model，用来包含这些逻辑。下面创建了一个 ``SignIn`` 类包含 ``email`` 和 ``password`` 字段。并且我们要验证 ``email`` 必须是一个存在的账号地址，并且 ``password`` 正确。

```ruby
class SignIn < PlainModel

  attr_accessor :email
  attr_accessor :password

  validate :validate_user_exists
  validate :validate_password_correct

  def user
    User.find_by_email(email) if email.present?
  end

  private

  def validate_user_exists
    if user.blank?
      errors.add(:user_id, 'User not found')
    end
  end

  def validate_password_correct
    if user && !user.has_password?(password)
      errors.add(:password, 'Incorrect password')
    end
  end
end
```

``PlainModel`` 类大概有 20 行代码用来封装 ActiveModel 相关的功能。如果你不熟悉 ActiveModel，你可以认为它是类似于 ActiveRecord:: Base 那样的一个基础类。下面的章节将提供 ``PlainModel`` 的完整代码。本例中，你只需要知道 ``PlainModel`` 是一个基类，它能让你自己编写的 Ruby 类更像是 ActiveRecord。

下面是 ``SignIn`` 类用到的 controller 和两个动作。``new`` 显示登录表单，``create`` 用来验证用户输入信息，并设置一个 session cookie。

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController

  def new
    build_sign_in
  end

  def create
    build_sign_in
    if @sign_in.save
      # remember user in cookie here
    else
      render 'new'
    end
  end

  private

  def build_sign_in
    @sign_in = SignIn.new(sign_in_params)
  end

  def sign_in_params
    sign_in_params = params[:sign_in]
    sign_in_params.permit(:email, :password) if sign_in_params
  end

end
```

我们的 controller 与拥有数据库支持的 ActiveRecord  model 的 controller 看起来没啥不同。``@sign_in`` 拥有和 ActiveRecord 对象一样的生命周期：

* Attributes are set from the params
* The object is asked to validate its attributes
* If the object can be “saved”, some action is performed

事实上 SessionsController 的基本上是从前文的 default controller implementation 复制过来的。

最后看试图部分：

```erb
# app/views/sessions/new.html.erb
<h1>Signin</h1>

<%= form_for(@sign_in, url: sessions_path) do |form| %>

  <%= form.label :email %>
  <%= form.text_field :email %>

  <%= form.label :password %>
  <%= form.password_field :password %>

  <%= form.submit %>

<% end %>
```

现在，试图和使用 ActiveRecord  model 的试图也没啥区别。我们可以使用 ``form_for`` 和 ``text_field`` 方法。自动高亮出错的字段，并且在表单中显示出错的脏数据。

### Building PlainModel

```ruby
class PlainModel

  include ActiveModel::Model
  include ActiveSupport::Callbacks
  include ActiveModel::Validations::Callbacks

  define_callbacks :save

  def save
    if valid?
      run_callbacks :save do
        true
      end
    else
      false
    end
  end
end
```

随着时间的推移，我们会把越来越多的功能从 ActiveRecord 引入 PlainModel。例如，我们增加了字段类型验证，指定某个字段必须是数值类型或日期类型等，或者 ``belongs_to`` 功能。

最后，我们把这些功能包装在了一个 gem 中，叫做 [ActiveType](https://github.com/makandra/active_type)。

使用 ActiveType，我们的 ``SignIn`` 看起来如下：

```ruby
class SignIn < ActiveType::Object

  attribute :email, :string
  attribute :password, :string

  validate :validate_user_exists
  validate :validate_password_correct

  def user
    User.find_by_email(email) if email.present?
  end

  private

  def validate_user_exists
    if user.blank?
      errors.add(:user_id, 'User not found')
    end
  end

  def validate_password_correct
    if user && !user.has_password?(password)
      errors.add(:password, 'Incorrect password')
    end
  end

end
```

不同于之前的例子，这回我们继承自 ``ActiveType::Object``，然后使用 ``attribute`` 宏来定义属性，而不是使用 ``attr_accessor``。

后面的章节我们会提到 extracting interaction-specific code。

### Refactoring controllers from hell

使用基于 ActiveModel 模型的类是重构 controller 的好方法。当我们说一个 controller 很差的时候，通常指代码太长，包含验证以及大量的业务逻辑。

现在，我们展示一个这样的 controller ，并演示如何使用 ActiveType 来重构它。这是生产环境中的真实代码，它支持 UI 交互，让用户选择两个账户合并。

一旦用户执行合并动作，将执行如下操作：

* All credits from the first account are transferred to the second account.
* All subscriptions from the first account are re-assigned to the second account.
* The first account is deleted.
* The owner of the first account is notified by e-mail that her account no longer exists.

下面是重构前的代码：

```ruby
class AccountsController < ApplicationController

  def merge_form
    @source_missing = false
    @target_missing = false
  end


  def do_merge
    source_id = params[:source_id]
    target_id = params[:target_id]
    if source_id.blank?
      @source_missing = true
    end
    if target_id.blank?
      @target_missing = true
    end

    if source_id.present? && target_id.present?
      source = Account.find(source_id)
      target = Account.find(target_id)
      Account.transaction do
        target.update_attributes!(:credits => target.credits + source.credits)
        source.subscriptions.each do |subscription|
          subscription.update_attributes!(:account => target)
        end
        Mailer.account_merge_notification(source, target).deliver
        source.destroy
      end
      redirect_to target
    else
      render 'merge_form'
    end
  end

end
```

由于没有使用 ActiveRecord 对象，所以下面的试图不能使用 ``form_for`` 方法，并且要手动控制一切。下面展示了，如何自己实现表单显示错误信息的功能：

```erb
<h1>Mergetwoaccounts</h1>

<%= form_tag do_merge_accounts_path do %>

  <%= label_tag :source_id, 'Source account' %>

  <% if @source_missing %>
    <div class="error">
      You must select a source account!
    </div>
  <% end %>

  <% selectable_accounts = Account.all.collect { |a| [a.name, a.id] } %>
  <%= select_tag :source_id, options_for_select(selectable_accounts) %>
  <%= label_tag :target_id, 'Target account' %>

  <% if @target_missing %>
    <div class="error">
      You must select a target account!
    </div>
  <% end %>

  <%= select_tag :target_id, options_for_select(selectable_accounts) %>

  <%= submit_tag %>

<% end %>
```

当我们把 controller 中的对象继承于 ActiveType 之后，我们的试图重构如下：

```erb
<h1>Mergetwoaccounts</h1>

<%= form_for @merge do |form| %>

  <%= form.error_messages_on :source_id %>
  <%= form.label :source_id %>
  <%= form.collection_select :source_id, Account.all, :id, :name %>

  <%= form.error_messages_on :target_id %>
  <%= form.label :target_id %>
  <%= form.collection_select :target_id, Account.all, :id, :name %>

  <%= form.submit %>

<% end %>
```

试图变得更短和容易理解。因为 ``@merge`` 对象支持 ActiveModel API，所以我们可以使用 ``form_for`` 和其它内建的辅助方法。比如给错误字段加个高亮之类的。

下面我们开始重构 controller ：

```ruby
class AccountMergesController < ApplicationController

  def new
    build_merge
  end

  def create
    build_merge
    if @merge.save
      redirect_to @merge.target
    else
      render 'new'
    end
  end

  private

  def build_merge
    @merge ||= AccountMerge.new(params[:merge])
  end

end
```

现在 controller 的风格也基本和 default controller implementation 一样了。

我们把原来实现验证和进行合并动作的代码重构之后放入 AccountMerge 类：

```ruby
class AccountMerge < ActiveType::Object

  attribute :source_id, :integer
  attribute :target_id, :integer

  validates_presence_of :source_id, :target_id

  belongs_to :source, class_name: 'Account'
  belongs_to :target, class_name: 'Account'

  after_save :transfer_credits
  after_save :transfer_subscriptions
  after_save :destroy_source
  after_save :send_notification

  private

  def transfer_credits
    target.update_attributes!(:credits => target.credits + source.credits)
  end

  def transfer_subscriptions
    source.subscriptions.each do |subscription|
      subscription.update_attributes!(:account => target)
    end
  end

  def destroy_source
    source.destroy
  end

  def send_notification
    Mailer.account_merge_notification(source, target).deliver
  end
end
```

可以看到，我们把合并操作移到 ``private`` 方法中，并在验证成功后调用它。

```ruby
after_save:transfer_credits
after_save:transfer_subscriptions
after_save:destroy_source
after_save:send_notification
```

你可能奇怪，我们做的看起来无非就是新建个文件然后四处移动原来的代码而已。没错！我们没有改变任何逻辑。注意 ``AccountMerge`` 类是如何定义自己的验证和合并动作并让整个逻辑显得更清晰的。这种修改让我们的 controller 从又臭又长变得非常漂亮。

另外，把代码从 controller 移动到 model 中也能让我们方便的进行单元测试。测试 controller 需要很多上下文环境(request, params, sessions)等。但是单元测试只需要调用该方法，并观察返回值。

由于能够使用 ``form_for`` 和许多内置的辅助方法，我们的试图也更短更易读。

现在，你的 controller 和程序中的其它 controller 一样：

* Instantiate an object
* Assign attributes from the params
* Try to save the object
* Render a view or redirect

这种一致性让其他的开发者很容易的理解和阅读你的代码。他知道你的 controller 如何工作，所以他只需要专注在 ``AccountMerge``  model 上即可。
