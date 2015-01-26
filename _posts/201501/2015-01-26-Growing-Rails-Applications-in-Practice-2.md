---
layout: post
title:  "Growing Rails Applications in Practice - 2"
date:   2015-01-26 20:30:00
categories: rails
tags: learningnote
author: "Victor"
---

## Creating a system for growth - 上

### Dealing with fat models

在前面的章节，我们把代码从 controller 移动到 model 中。我们可以看到这样带来的很多好处：controller 变得很简单，更容易测试商业逻辑。但一段时间之后，model 开始产生如下问题：

* 你害怕保存数据会引起意外回调，比如给客户发送个电子邮件啥的。
* 过多的回调和验证让你很难创建一个简单的数据来进行单元测试。
* 不同的 UI 界面需要你的 model 提供不同的支持。比如：用户自己注册的时候，发送给他一个欢迎邮件。而管理员从后台注册用户就不处罚这个动作。

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

我们曾说过 **large applications are large**。当你需要实现恢复密码的工恩呢刚，但是不新建一个地方来放置这些代码，那么 **Code never goes away**。它必然会横跨几个类，并让这些类都变得难读难用。

**Code never goes away** 意思是你需要把代码组织到一个正确的地方，否则它会感染现有类。

想想，假如你要处理一批信件。这些信在你家里散落到各处，可能是办公桌，床上，餐桌，鞋盒子等等，因为它们没有一个固定地方，很难找到你要找的那封信，它们弄乱了你的办公桌。你不知道去哪找信来阅读，也不知道自己今天是否遗漏了某封重要信件。

当然，你可以可以创建一个收件箱来解决这个问题。只需要每天从收件箱取信然后放在办公桌前处理就好。

注意，我们并没有让你的信件消失，也不是让你不处理信件了。只是我们让这一切更有序，可以提高我们处理信件的效率。**越高的结构化，往往意味着我们的项目更长寿**。

#### Getting into a habit of organizing

做个收件箱不难，难的是要认清你身边有无数算乱的信件的能力。

类似的，当你要找个地方新添加代码的时候，不要马上去找已有的 ActiveRecord 类，试着去寻找一个新类来包含这些逻辑。

下面的章节我们将展示：

* 如何在你的代码中发现未被抽象的概念
* 如何将相互作用的代码和逻辑放在自己的类中，而让核心类苗条
* 如何识别出一段代码不该插入现有的 ActiveRecord 模型而是应该将其抽取到一个 service 类
* 如何借用 ActiveRecord 的功能方便的做到着一些

### A home for interaction-specific code

大多数成熟的 Rails 项目忍受巨大 model 的原因都是它们承担了太多功能。前面我们通过一个典型的 User 模型来描述为什么 model 会越来越巨大。

将一个巨大的 model 砍小的第一步是将互相之间有关系的代码抽取成一个单独的 model，并让核心 model 保持简洁。

你的核心 model 应该只包含如何内容：

* 最低程度的验证确保数据的完整
* 声明联合关系(belongs_to，has_many)
* 通用且重要的方法用来查找或操作数据

核心 model 不该包含特殊的与表单或用户界面相关的逻辑，例如：

* 只在特定表单才进行的验证，例如：only the sign up form requires confirmation of the user password
* 虚拟属性，以支持那些并非和数据库表对应的表单，例如：tags are entered into a single text field separated by comma, the model splits that string into individual tags before validation
* 仅在特定界面或用例才执行的回调，例如：只有用户注册页面了才发送邮件
* 关于授权验证的代码 [access control](http://bizarre-authorization.talks.makandra.com/)
* 用来渲染复杂试图的辅助方法

所有这些只与一个 UI(或一组相关的 UI) 有关系的逻辑和代码，最好被放在一个单独的 model。

如果你一直保持，将特定的业务逻辑抽取成 model 的做法，那么你的核心 model 也会保持简洁，并且大多没有啥回调。

#### A modest approach to form models

#### More convenience for form models

### Extracting service objects

#### Example

#### Did we just move code around?

