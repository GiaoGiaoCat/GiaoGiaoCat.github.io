---
layout: post
title:  "Form Object"
date:   2017-06-11 11:30:00
categories: rails
tags: refactoring
author: "Victor"
---

### 什么是 Form Object

封装了一个表单对象的 Ruby 类。

### 何时使用 Form Object

* 需要在一个表单中更新多个 ActiveRecord 模型，这比使用 `accepts_nested_attributes_for` 要好的多。
* 提交的数据不需要持久化。
* 创建一个没有数据库模型支持的表单。

比如修改密码，找回密码，登录，注册之类的表单。而根据复杂度不同，可能这些 From Object 又会去调用 Service Object。

### Form Object 和 Service Object 的区别

代码和功能上没啥区别，只有在抽象哲学层面有点不同。Form Object 是抽象出来提供给 HTML 表单用的。

### 使用 `accepts_nested_attributes_for` 怎么了？

你可能看过这篇文章 [accepts_nested_attributes_for with Has-Many-Through Relations](https://robots.thoughtbot.com/accepts-nested-attributes-for-with-has-many-through), 并且觉得 Rails Way 很棒啊，但是这会带来如下问题：

* 你的 model 需要关心 view 是如何收集数据的
* 强迫 view 的结构必须和 model 的结构和 relationships 对应起来
* ActiveRecord 对象增加更多复杂的 API
* 当同一个模型中存在两个 `accepts_nested_attribtutes_for` 或者一个 `accepts_nested_attribtutes_for` 关联的 model 存在其它 relationships 的时候，很容易使人迷惑

### 例子

下面是一个正常的例子。

```ruby
class Signup
  include Virtus

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attr_reader :user
  attr_reader :company

  attribute :name, String
  attribute :company_name, String
  attribute :email, String

  validates :email, presence: true
  # … more validations …

  # Forms are never themselves persisted
  def persisted?
    false
  end

  def save
    if valid?
      persist!
      true
    else
      false
    end
  end

private

  def persist!
    @company = Company.create!(name: company_name)
    @user = @company.users.create!(name: name, email: email)
  end
end
```

```ruby
class SignupsController < ApplicationController
  def create
    @signup = Signup.new(params[:signup])

    if @signup.save
      redirect_to dashboard_path
    else
      render "new"
    end
  end
end
```

如果表单中的 `persist!` 逻辑太复杂，您可以将此方法与 `Service Object` 组合。由于 validation 的逻辑通常是关联上下文的，因此可以在 `Form Object` 或 `Service Object` 中进行定义，而不需要在 ActiveRecord model 中定义 validate。

但是这并不意味着，model 里面一个验证都没有。应该把核心的验证逻辑放在 model 中。比如 `User` 模型中应该保留 `email` 唯一性的验证。

## Form Object with active_type

* Let your form inherit from ApplicationForm
* Put your forms in `app/forms/`
* 文件以 form 结尾并能描述出功能。例如：`app/forms/admins/change_password_form.rb`
* initialize always requires a model that the form represents.
* `#sync` writes form data back to the model and nested models. This will only use setter methods on the model(s).

### 例子

```ruby
# app/forms/application_form.rb
class ApplicationForm < ActiveType::Object
  self.abstract_class = true

  attribute :form_object

  validates :form_object, presence: true
  validate :verify_form_object_class_correct

  after_save :sync

  private

  def sync
    raise NotImplementedError, 'Must be implemented by subtypes.'
  end

  # NOTE: can implemented by subtypes.
  def form_object_class
    self.class.to_s[/\A([^:]+)/, 1].singularize.constantize
  rescue
    raise "Could not determine form class from #{self.class}."
  end

  def verify_form_object_class_correct
    errors.add :base, :form_object_class_incorrect unless form_object.is_a? form_object_class
  end
end
```

```ruby
# app/forms/admins/password_form.rb
class Admins::PasswordForm < ApplicationForm
  attribute :password
  attribute :password_confirmation

  validates :password, confirmation: true

  private

  def sync
    form_object.update(password: password)
  end
end
```

```ruby
class Admins::PasswordsController < ApplicationController
  before_action :load_admin, :build_password_form

  def update
    if @password_form.save
      redirect_to admins_path, notice: 'Admin password was successfully updated.'
    else
      render :edit
    end
  end

  private

  def load_admin
    @admin = Admin.find(params[:admin_id])
  end

  def build_password_form
    @password_form = Admins::PasswordForm.new(form_object: @admin)
    @password_form.attributes = form_params
  end

  def form_params
    form_params = params[:admins_password_form]
    form_params ? form_params.permit(:password, :password_confirmation) : {}
  end
end
```

如果你的 From Object 要同时保存两个对象，可以参考下面的代码。

```ruby
class Registration
  # Make this model play nice with `form_for`. Gives us validations, initialize, etc
  include ActiveModel::Model

  # Accessors for the fields we are exposing in the form
  attr_accessor :email, :password, :zip

  # This is an implementation detail of our authentication (Clearance). It's ultimately going to
  # sign this object in and that process expects a user object that has these fields available
  delegate :remember_token, :id, to: user

  # For this object to be valid, we want the child objects (user and profile) to be valid. This is
  # the validation method that does this.
  validate :validate_children

  # We tell registration it's a user. This will keep it pointing to Users controller and allow it to use
  # user translations
  def self.model_name
    User.model_name
  end

  # This is what our controller calls to save the user.
  def save
    if valid?
      # Create a transaction. If any of the database stuff in here fails, they will raise an exception
      # (because they are bang methods) and rollback the transaction.
      ActiveRecord::Base.transaction do
        user.save!
        profile.save!
      end
    end
  end

  private

  # Initialize the user object with the arguments that apply to user
  def user
    @user ||= User.new(email: email, password: password)
  end

  # initialize the profile object with the arguments that apply to profile
  def profile
    @profile ||= user.build_profile(zip: zip)
  end

  # the implementation of our validation method. We just delegate to our
  # two child objects which define their validations (presence on all fields in this case)
  def validate_children
    if user.invalid?
      promote_errors(user.errors)
    end

    if profile.invalid?
      promote_errors(profile.errors)
    end
  end

  # Errors on `user` or `profile` aren't helpful to us because `user.email` isn't on our form. `email` is however.
  # Take all errors from our child objects and promote them to the same field on the base object. This will make
  # them render properly on the form.
  def promote_errors(child_errors)
    child_errors.each do |attribute, message|
      errors.add(attribute, message)
    end
  end
end
```

## 参考

* [#416 Form Objects pro](http://railscasts.com/episodes/416-form-objects)
* [Form Objects](https://forum.upcase.com/t/form-objects/2267)
* [Rails Form Objects With dry-rb](http://cucumbersome.net/2016/09/06/rails-form-objects-with-dry-rb.html)
* [Extensible Rails 4 Form Object Design](http://stratus3d.com/blog/2015/04/04/extensible-rails-4-form-object-design/)
* [Reform gem](https://github.com/trailblazer/reform)
* [formeze gem](https://github.com/timcraft/formeze)
