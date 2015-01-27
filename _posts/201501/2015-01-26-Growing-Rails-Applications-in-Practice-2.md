---
layout: post
title:  "Growing Rails Applications in Practice - 2"
date:   2015-01-26 20:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## Creating a system for growth - 上

### Dealing with fat models

在前面的章节，我们把代码从 controller 移动到 model 中。我们可以看到这样带来的很多好处：controller 变得很简单，更容易测试商业逻辑。但一段时间之后，model 开始产生如下问题：

* 你害怕保存数据会引起意外回调，比如给客户发送个电子邮件啥的。
* 过多的回调和验证让你很难创建一个简单的数据来进行单元测试。
* 不同的 UI 界面需要你的 model 提供不同的支持。比如：用户自己注册的时候，发送给他一个欢迎邮件。而管理员从后台注册用户就不触发这个动作。

All of these problems of so-called **fat models** can be mitigated. But before we look at solutions,
let’s understand why models grow fat in the first place.

#### Why models grow fat

model 体积变大的主要原因是因为它们要承担越来越多的任务。例如一个用户模型要支持如下的如多用例：

* A new user signs up through the registration form
* An existing user signs in with her email and password
* A user wants to reset her lost password
* A user logs in via Facebook (OAuth)
* A users edits her profile
* An admin edits a user from the backend interface
* Other models need to work with User records
* Background tasks need to batch-process users every night
* A developer retrieves and changes User records on the Rails console

所有这些用例都会影响模型中的其它用例。 You end up with a model that is very hard to use without undesired side effects getting in your way.

咱们看看，当这个项目你维护了1年之后，你的 User 模型都会包含些啥：

<table>
  <tr>
    <th>Use case</th>
    <th>Scar tissue in User model</th>
  </tr>
  <tr>
    <td>New user registration form</td>
    <td>
      Validation for password strength policy<br />
      Accessor and validation for password repetition<br />
      Accessor and validation for acceptance of terms<br />
      Accessor and callback to create newsletter subscription<br />
      Callback to send activation e-mail<br />
      Callback to set default attributes<br />
      Protection for sensitive attributes (e.g. admin flag)<br />
      Code to encrypt user passwords
    </td>
  </tr>
  <tr>
    <td>Login form</td>
    <td>
      Method to look up a user by either e-mail or screen name<br />
      Method to compare given password with encrypted password
    </td>
  </tr>
  <tr>
    <td>Password recovery</td>
    <td>
      Code to generate a recovery token<br />
      Code to validate a given recovery token<br />
      Callback to send recovery link
    </td>
  </tr>
  <tr>
    <td>Facebook login</td>
    <td>
      Code to authenticate a user from OAuth<br />
      Code to create a user from OAuth<br />
      Disable password requirement when authenticated via OAuth
    </td>
  </tr>
  <tr>
    <td>Users edits her profile</td>
    <td>
      Validation for password strength policy<br />
      Accessor and callback to enable password change<br />
      Callback to resend e-mail activation<br />
      Validations for social media handles<br />
      Protection for sensitive attributes (e.g. admin flag)
    </td>
  </tr>
  <tr>
    <td>Admin edits a user</td>
    <td>
      Methods and callbacks for authorization (access control)<br />
      Attribute to disable a user account<br />
      Attribute to set an admin flag
    </td>
  </tr>
  <tr>
    <td>Other models that works with users</td>
    <td>
      Default attribute values<br />
      Validations to enforce data integrity<br />
      Associations<br />
      Callbacks to clean up associations when destroyed<br />
      Scopes to retrieve commonly used lists of users
    </td>
  </tr>
  <tr>
    <td>Background tasks processes users</td>
    <td>
      Scopes to retrieve records in need of processing<br />
      Methods to perform required task
    </td>
  </tr>
</table>

你很难从这一大堆代码中理出头绪。即便代码的可读性没问题，但是这样的 model 是很难使用的：

* 你害怕更新记录，因为不知道会触发什么回调。比如，当用户更新了自己的个人资料，后台任务会同步该数据并发送一份邮件通知用户。
* 有时想要创建一个用户，却发现自己无法保存记录，因为有些回调禁止你这么做(在管理员后台这么做是有意义的)。
* 不同的用户表单需要不同类型的验证，无处不在的验证让你开始像猜谜一样，修改 model 的配置来启用或禁用验证行为。
* 单元测试几乎不可能，因为每一个与 User 的交互都会引发无数的副作用，你必须在它们中间开辟出一条路，才能确保测试无误。

#### The case of the missing classes

**Fat model** 往往意味着一些未被发现的抽象模型。上表中的功能，可以被抽象出如下模型：

* PasswordRecovery
* AdminUserForm
* RegistrationForm
* ProfileForm
* FacebookConnect

我们曾说过 **large applications are large**。当你需要实现恢复密码的功能，但是又不新建一个地方来放置这些代码，那么 **Code never goes away**。它必然会横跨几个类，并让这些类都变得难读难用。

**Code never goes away** 意思是你需要把代码组织到一个正确的地方，否则它会感染现有类。

想想，假如你要处理一批信件。这些信在你家里散落到各处，可能是办公桌，床上，餐桌，鞋盒子等等，因为它们没有一个固定地方，很难找到你要找的那封信，它们弄乱了你的办公桌。你不知道去哪找信来阅读，也不知道自己今天是否遗漏了某封重要信件。

当然，你可以可以创建一个收件箱来解决这个问题。只需要每天从收件箱取信然后放在办公桌前处理就好。

注意，我们并没有让你的信件消失，也不是让你不处理信件了。只是我们让这一切更有序，可以提高我们处理信件的效率。**越高的结构化，往往意味着我们的项目更长寿**。

#### Getting into a habit of organizing

做个收件箱不难，难的是要认清你身边有无数算乱的信件的能力。

类似的，当你要找个地方新添加代码的时候，不要马上去找已有的 ActiveRecord 类，试着去寻找一个新类来包含这些逻辑。

下面的章节我们将展示：

* 如何在你的代码中发现未被抽象的概念
* 如何将相互作用的代码和逻辑放在单独的类中，而让核心类苗条
* 如何识别出一段代码不该插入现有的 ActiveRecord 模型而是应该将其抽取到一个 service 类
* 如何借用 ActiveRecord 的功能方便的做到这一切

### A home for interaction-specific code

大多数成熟的 Rails 项目忍受巨大 model 的原因都是它们承担了太多功能。前面我们通过一个典型的 User 模型来描述为什么 model 会越来越巨大。

将一个巨大的 model 砍小的第一步是将互相之间有关系的代码抽取成一个单独的 model，并让核心 model 保持简洁。

你的核心 model 应该只包含如下内容：

* 最低程度的验证确保数据的完整
* 声明联合关系(belongs_to，has_many)
* 用来查找或操作数据相关的方法，并且这些方法是通用且重要的

核心 model 不该包含特殊的与表单或用户界面相关的逻辑，例如：

* 只在特定表单才进行的验证，例如：only the sign up form requires confirmation of the user password
* 虚拟属性，以支持那些并非和数据库表对应的表单，例如：tags are entered into a single text field separated by comma, the model splits that string into individual tags before validation
* 仅在特定界面或用例才执行的回调，例如：只有用户注册页面了才发送邮件
* 关于授权验证的代码 [access control](http://bizarre-authorization.talks.makandra.com/)
* 用来渲染复杂视图的辅助方法

所有这些只与一个 UI(或一组相关的 UI) 有关系的逻辑和代码，最好被放在一个单独的 model。

如果你一直保持这种将特定的业务逻辑抽取成 model 的做法，那么你的核心 model 也会保持简洁，并且不会被回调干扰。

#### A modest approach to form models

关于模型的重构不是什么新鲜事。你会从 Github 上发现一大堆和 **presenters**，**exhibits**，**form models** 相关的 gems。不幸的是，其中很大部分看起来都不像从实践中出发。下面是一些我们在实践中遇到的问题：

* 无法和 Rails 的表单方法工作，比如 ``form_for``
* 无法使用 ActiveRecord 宏风格的代码，比如 ``validates_presence_of``
* 无法使用嵌套资源型表单
* 只针对单个对象工作，面对一组对象就无效了，比如 index 动作
* 创建新记录和编辑已经存在的记录需要不同的主持类, 虽然用户界面是相同的
* 你需要从核心类中复制粘贴验证和逻辑代码
* 频繁使用委托(过多的委托，会引起自身的混乱)
* 增加许多文件，加大理解难度

这些 gems 让我们一次又一次失望，我们想知道：是否有可能既享受 ActiveRecord 的便利，又能把不同的逻辑放在各自的类中。

我们发现这是可行的，甚至不需要引入什么特殊的 gem 就能做到这一点。我们需要的就是利用普通的继承把和页面交互相关的代码从 model 中抽取到另外一个地方。

让我们来看一个例子。思考下面这个已经有点巨大的 User model 如何才能变得好起来：

```ruby
class User < ActiveRecord::Base

  validates :email, presence: true, uniqueness: true
  validates :password, presence: true, confirmation: true
  validates :terms, acceptance: true

  after_create :send_welcome_email

  private

  def send_welcome_email
    Mailer.welcome(user).deliver
  end

end
```

很明显这些代码涉及注册表单，应该从 User 核心类移除。当某些情况下想要跳过验证（管理员在后台操作）或者某种情况下创建用户不发送欢迎邮件的时候（控制台创建用户），当前这个 model 让你开始遇到麻烦。

下面我们减小 model 的体积，仅保留基本的逻辑：

```ruby
class User < ActiveRecord::Base
  validates :email, presence: true, uniqueness: true
end
```

所有跟注册有关的代码都移动到 ``User::AsSignUp`` 类中，该类继承 ``User`` 类：

```ruby
class User::AsSignUp < User

  validates :password, presence: true, confirmation: true
  validates :terms, acceptance: true

  after_create :send_welcome_email

  private

  def send_welcome_email
    Mailer.welcome(user).deliver
  end
end
```

需要注意的是我们不必在新类中重复验证 email 属性。由于这个新 model 继承自核心 model，它自动继承了核心 model 的全部行为，并在此基础上进行自己的定义。

当我们使用 ``User::AsSignUp`` 来代替原来的 ``User`` 类之后，用来处理注册表单的 controller 也非常简单：

```ruby
class UsersController < ActionController

  def new
    build_user
  end

  def create
    build_user
    if @user.save
      redirect_to dashboard_path
    else
      render 'new'
    end
  end

  private

  def build_user
    @user ||= User::AsSignUp.new(user_params)
  end

  def user_params
    user_params = params[:user]
    user_params.permit(
      :email,
      :password,
      :password_confirmation,
      :terms
    )
  end
end
```

需要注意的是，我们使用 ``User::AsSignUp`` 作为类名仅仅是因为大家都喜欢用这样的命名约定而已，并没有强迫你使用 **As** 作为前缀或命名空间。但是，我们推荐使用这种目录结构来帮助你更好的组织模型。

File | Class definition
---|---
user.rb | class User < ActiveRecord::Base
user/as_sign_up.rb | class User::AsSignUp < User
user/as_profile.rb | class User::AsProfile < User
user/as_admin_form.rb | class User::AsAdminForm < User
user/as_facebook_login.rb | class User::AsFacebookLogin < User

后面的章节我们会介绍如何用命名空间来组织代码。

#### More convenience for form models

在不断地实践中，我们发现可以通过适当增加一些小的调整，能让我们更方便的和 models 交互。最后，我们把这些小的功能打包成了前面的章节提到过的 ``ActiveType`` 这个 gem。即使你并不需要使用 ``ActiveType`` 从 model 带来的便利，这里有几个例子说明为什么 ``ActiveType`` 能让你的开发工作更轻松：

* 像 ``url_for``，``form_for``，``link_to`` 这样的辅助方法会根据类名判断出路由地址。如果使用 ``User::AsSignUp`` 这样的名字，这些方法就无法正确工作。而 ``ActiveType`` 内部已经帮忙处理这个问题，它会把 ``link_to(@user)`` 方法路由到 ``user_path(@user)``，即便 ``@user`` 是一个 ``User::AsSignUp`` 的实例。
* 当使用[单表继承](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#class-ActiveRecord::Base-label-Single+table+inheritance)时，你很可能不想在 ``type`` 字段存一个叫做 ``User::AsSignUp`` 的值，``ActiveType`` 也解决了这个问题，在这种情况下，我们不存 ``User::AsSignUp``，仍然存 ``User``。
* 对于表单模型，你需要控制一些虚拟属性(某些属性自动转换为整数或日期类型)。尤其当你的模型不适合数据库映射的时候。``ActiveType`` 附带了一些简单的语法糖方便我们在类中定义这些属性。

切换到 ``ActiveType`` 很简单，你只需要从

```ruby
class User::AsSignUp < User
```

改成

```ruby
class User::AsSignUp < ActiveType::Record[User]
```

一切都和原来一样，但是你得到了上述我们介绍的优点。

需要注意的时，``ActiveType`` 除了能让表单更方便还有其它优势，你仍然可以像原 Ruby 类一样使用前面章节我们讨论过的 ``ActiveModel`` API。

下一章我们将讨论如何进一步减少核心 model 的体积。

### Can’t I just split my model into modules?

有很多技术可以把 **fat models** 分割成多个 modules，比如 DDH 推荐的 [Put chubby models on a diet with concerns](https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns)。

虽然这种技术可以有效地在多个模型中共享行为，但你必须了解这并非是文件组织的唯一形式。由于所有这些 modules 都会在运行时加载进入你的 model，实际上它并没有降低 model 的代码量。你的回调和方法不会因为被划分在多个模块中而消失。仅仅只是换了个地方而已。
