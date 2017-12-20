---
layout: post
title:  "Growing Rails Applications in Practice - 3"
date:   2015-01-27 15:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## Creating a system for growth - 中

### Extracting service objects

学会权衡 ActiveModel 类之间的关联是很重要的。ActiveModel 能出色的处理面向用户的表单，它可以处理验证错误并返回相关的错误提示，这些都属于用户交互的核心功能。对于这种情况，你只需几行代码就能完成这些功能。

但是并非每一个应用或类都需要使用 ActiveRecord 或 ActiveModel。当你不需要使用 ActiveModel 的功能(比如验证，回调，脏数据跟踪等)的时候，你不应该强迫类接受 ActiveModel API，因为该类不需要 **一大堆跟回调有关的属性**。

审查整个 **fat model**，你通常会发现许多代码并不需要被捆绑到 ActiveRecord。这些代码通常由其他 model 调用，并且从不与 user 进行沟通，或者仅在少数有限的场景下进行简单的互动，比如：注册一个事件或查找一个值。

这样的代码应该被提取到一个纯 Ruby 类中，仅完成他们需要完成的一个任务。我们把这种方式的类叫做 **service object**，尽管它就是一个简单的 Ruby 类。

给 model 减肥的最好方法就是浏览你的代码，然后寻找相关的一组方法，并把它们放到自己的 service object 中。

#### Example

假设我们有一个 Note model。这个 model 有一个搜索方法使用 [MySQL LIKE query](http://dev.mysql.com/doc/refman/5.7/en/string-comparison-functions.html) 用来查询所有可用的 notes。所以，当调用 ``Note.search('growing rails')`` 的时候，产生的 SQL 查询是这样的：

```sql
SELECT * FROM notes WHERE
  title LIKE "%growing%" OR body LIKE "%growing%" OR
  title LIKE "%rails%" OR body LIKE "%rails%";
```

下面是 Note 的具体实现(private 的 class 方法是无效的)：

```ruby
class Note < ActiveRecord::Base

  validates_presence_of :title, :body

  def self.search(query)
    parts = []
    bindings = []
    split_query(query).each do |word|
      escaped_word = escape_for_like_query(word)
      parts << 'body LIKE ? OR title LIKE ?'
      bindings << escaped_word
      bindings << escaped_word
    end
    where(parts.join(' OR '), *bindings)
  end

  private

  def self.split_query(query)
    query.split(/\s+/)
  end

  def self.escape_for_like_query(phrase)
    phrase.gsub("%", "\\%").gsub("_", "\\_")
  end

end
```

想想看，``search, split_query, and escape_for_like_query`` 这些方法不需要在 model 类里面。最简单的重构方法是把这些相关的方法移动到一个新的类，它用来处理搜索相关功能：

```ruby
class Note::Search

  def initialize(query)
    @query = query
  end

  def matches
    parts = []
    bindings = []
    split_query.each do |word|
      escaped_word = escape_for_like_query(word)
      parts << 'body LIKE ? OR title LIKE ?'
      bindings << escaped_word
      bindings << escaped_word
    end
    Note.where(parts.join(' OR '), *bindings)
  end

  private

  def split_query
    @query.split(/\s+/)
  end

  def escape_for_like_query(phrase)
    phrase.gsub("%", "\\%").gsub("_", "\\_")
  end
end
```

``Note::Search`` 仅作一件事：进行查询并返回符合条件的结果。这是一个仅包含单一功能的简单类。

现在 Note 类简化成了：

```ruby
class Note < ActiveRecord::Base

  validates_presence_of :title, :body

end
```

我们在 Note 中删除了一些代码，又创建了一个新地方用来放置将来可能出现的和搜索相关的功能。如果出现新需求，比如要把数据引擎从 Mysql 切换到 Lucene，``Note::Search`` 是一个很好的放置这些代码的地方。

沿着这样的路线，你可以把代码独立成外部概念，像：``note``，``Search`` 或 ``Excel export``。这一策略从长远来看，可保持应用在不断增加功能的同时又易于维护。

我们应该积极寻找机会将代码抽取成 ``service objects``。很多时候，这不难识别，很容易将这些代码抽取出来。下面的例子提供了一些典型的场景：

Fat model style | Service object style
---|---
``Note.dump_to_excel(path)`` | Note::ExcelExport.save_to(path)
``User.authenticate(username, password)`` | Login.authenticate(username, password)
``Project#changes`` | Project::ChangeReport.new(project).changes
``Invoice#to_pdf`` | Invoice::PdfRenderer.render(invoice)

#### Did we just move code around?

让我们看看重构之后的 Note 类，你可能注意到我们没删除一行代码。事实上我们为了声明类，反而增加了两行。然后，原本交织在一起的逻辑被分散成了多个。功能组件之间达成了松耦合的目标。

通过这种剥离。我们的核心 model 仅剩下基础的数据完整性验证，关联声明和通用的方法，我们的类现在变得容易使用：

* 我们限制了可能互相影响的副作用。例如：你不必再因为保存对象会不会引起发送邮件或什么其它动作而担惊受怕。
* 你的核心 model 更易于测试，因为它只包含有限的验证和回调，很容易创建测试数据。
* 相比一个混合了许多不同功能的大类来说，重构之后的类更容易浏览。我们只需要专注某一个单一功能，例如：``Note::Search``，``Note::Export`` 等

就像收件箱让我们的公寓更整洁，且帮我们提高处理信件的效率一样，把这些相关的业务逻辑放在单独的模型或 service objects 中，让我们的应用程序组织结构更好，并且能处理更多的商业逻辑。

### Organizing large codebases with namespaces

随着 Rails 项目的不断增长，``app/models`` 文件夹也会随着一起变得巨大。我们都看过在一个项目中存在上百个 models 的情况。``app/models`` 文件夹过大，变得很难在文件之间导航。通过查看 models 文件夹几乎不可能理解应用的功能，每一个重要的 model 旁边都跟随了一些支撑它的类。

把 models 利用命名空间分配到不同的子文件夹，可以避免我们被 .rb 文件淹死。这种方法不会减少实际的文件数目，但是能突出显示重要的部分，方便我们浏览代码。

给 model 增加命名空间很简单。比如我们有一个 ``Invoice`` 类，并且可以有多个 invoice items:

```ruby
class Invoice < ActiveRecord::Base
  has_many :items
end

class Item < ActiveRecord::Base
  belongs_to :invoice
end
```

很明显 Invoice 包含许多 Items，但是一个 Item 不能在 Invoice 之外单独存在。其它的类可能只和 Invoice 交互，而不管 Item。既然如此，那么可以把 Item 移动到 Invoice 命名空间下。我们需要把类重命名为 ``Invoice::Item`` 并将其移动到 ``app/models/invoice/item.rb``

```ruby
# app/models/invoice/item.rb
class Invoice::Item < ActiveRecord::Base
  belongs_to :invoice
end
```

这个看起来微不足道的重构，可能在接下来的几周不能带来什么明显的好处。这也是一个 Rails 团队的坏习惯，它推荐大家避免产生过多的类，似乎增加一个文件是一个多糟糕的事情。但是，事实上是一个巨大的 models 文件夹才有问题。

但是，因为我们有了 ``models/invoice`` 文件夹，当你的同事需要添加一个和 invoice 相关的 model 时，那么他知道文件该放在哪了。

File | Class
---|---
app/models/invoice.rb | Invoice
app/models/invoice/item.rb | Invoice::Item
app/models/invoice/reminder.rb | Invoice::Reminder
app/models/invoice/export.rb | Invoice::Export

请注意命名空间如何鼓励使用 Service Objects 来代替 fat models 实现更多的功能。

#### Real-world example

为了形象的演示命名空间的效果，我们要重构一个非常旧的项目，它没有使用命名空间，下面这个例子来自真实的产品。

下面是重构之前的文件夹：

* activity.rb
* amortization_cost.rb
* api_exchange.rb
* api_schema.rb
* budget_calculator.rb
* budget_rate_budget.rb
* budget.rb
* budget_template_group.rb
* budget_template.rb
* business_plan_item.rb
* business_plan.rb
* company.rb
* contact.rb
* event.rb
* fixed_cost.rb
* friction_report.rb
* internal_working_cost.rb
* invoice_approval_mailer.rb
* invoice_approval.rb
* invoice_item.rb
* invoice.rb
* invoice_settings.rb
* invoice_template.rb
* invoice_template_period.rb
* listed_activity_coworkers_summary.rb

看看这文件列表，你能告诉我这个应用是干啥的吗？几乎肯定不会。事实上，它是一个管理项目和发票的应用。

让我们看看重构之后的版本：

* /activity
* /api
* /contact
* /invoice
* /planner
* /report
* /project
* activity.rb
* contact.rb
* planner.rb
* invoice.rb
* project.rb
* user.rb

注意，现在 ``app/models`` 文件夹能让你对项目的核心功能一览无遗。每一个文件都仍然存在，但整齐的组织成了一个清晰的目录结构。如果我们让一个新同事去修改 invoice 的功能，通过在这种方式他可以很快找到该去哪里修改代码。

#### Use the same structure everywhere

在一个典型的 Rails 应用中，有许多地方的结构和 models 文件夹一样。例如，你经常看到按照 models 命名的 helper 和 unit test。

当你开始使用命名空间，确保这个命名方式在所有地方都按照 model 的组织方式来重新组织。这样你的项目会得到更好的组织方式，使程序解耦。

比方说我们有一个命名空间叫 ``Project::Report``，那么我们的 helpers 和 controllers 和 views 应该看起来如下：

File | Class
---|---
app/models/project/report.rb | Project::Report
app/helpers/project/report_helper.rb | Project::ReportHelper
app/controllers/projects/reports_controller.rb | Projects::ReportsController
app/views/projects/reports/show.html.erb | View template

While this might feel strange at first, it allows for natural nesting of folders in in app/views:

注意，我们把 controller 放在 ``Projects`` （复数）命名空间。首先你可能觉得奇怪，这样的好处是它允许文件夹的自然嵌套，我们的 ``app/views`` 如下：

```
app/
  views/
    projects/
      index.html.erb
      show.html.erb
      reports/
        show.html.erb
```

如果把 controller 放在单数的命名空间 ``Project``，Rails 会把视图模板的结构组织成如下：

```
app/
  views/
    project/
      reports/
        show.html.erb
    projects/
      index.html.erb
      show.html.erb
```

注意当使用单数 project 和复数 projects 时在文件夹内部的区别。我们认为 views 中的视图组织相比 controller 的命名空间保持单数来说，更重要。

#### Organizing test files

When we have tests we nest the test cases and support code like we nest our models. For instance, when you use RSpec and Cucumber, your test files should be organized like this:

File | Description
---|---
spec/models/project/report_spec.rb | Model test
spec/controllers/projects/reports_controller_spec.rb | Controller test
features/project/reports.feature | Cucumber untegration test
features/step_definitions/project/report_steps.rb | Step definitions

#### Other ways of organizing files

Of course models/controllers/tests don’t always map 1:1:1, but often they do. We think it is at the very least a good default with little ambiguity. When you look for a file in a project structured like this, you always know where to look first.

If another way to split up your files feels better, just go ahead and do it. Do not feel forced to be overly consistent, but always have a good default.
