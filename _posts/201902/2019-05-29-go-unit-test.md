---
layout: post
title:  "Go 的测试"
date:   2019-05-29 08:00:00

categories: go
tags: testing
author: "Victor"
---

## 好的测试的特点

单元测试的缺失不仅会意味着较低的工程质量，而且意味着重构的难以进行。

写好单元测试一定会有一些学习曲线和不适应，甚至会在短期内影响我们的开发效率，但是熟悉了这一套流程和接口之后，单元测试对我们的帮助会非常大，每一个单元测试都表示一个业务逻辑，每次提交时执行单元测试就能够帮助我们确定新的代码大概率上不会影响已有的业务逻辑，能够明显地降低重构的风险以及线上事故的数量

### 可测试

测试就是控制变量，在我们隔离了待测试方法中一些依赖之后，当函数的入参确定时，就应该得到期望的返回值。

为了减少每一个单元测试的复杂度，我们需要：

1. 尽可能减少目标方法的依赖，让目标方法只依赖必要的模块。
2. 依赖的模块也应该非常容易地进行 Mock。

单元测试的执行不应该依赖于任何的外部模块，无论是调用外部的 HTTP 请求还是数据库中的数据，我们都应该想尽办法模拟可能出现的情况，因为单元测试不是集成测试的，它的运行不应该依赖除项目代码外的其他任何系统。

### 接口

在 Go 语言中如果我们完全不使用接口，是写不出易于测试的代码的，作为静态语言的 Golang，只有我们使用接口才能脱离依赖具体实现的窘境，接口的使用能够为我们带来更清晰的抽象，帮助我们思考如何对代码进行设计，也能让我们更方便地对依赖进行 Mock。

接口的经典范式：

```go
ype Service interface { ... }

type service struct { ... }

func NewService(...) (Service, error) {
    return &service{...}, nil
}
```

如果你不知道应不应该使用接口对外提供服务，这时就应该无脑地使用上述模式对外暴露方法。

1. 使用大写的 Service 对外暴露方法
2. 使用小写的 service 实现接口中定义的方法
3. 通过 NewService 函数初始化 Service 接口

顺带给一下这个范式的例子。

```go
package post

type Service interface {
    ListPosts() ([]*Post, error)
}

type service struct {
    conn *grpc.ClientConn
}

func NewService(conn *grpc.ClientConn) Service {
    return &service{
        conn: conn,
    }
}

func (s *service) ListPosts() ([]*Post, error) {
    posts, err := s.conn.ListPosts(...)
    if err != nil {
        return []*Post{}, err
    }

    return posts, nil
}
```

```go
package main

import ...

func main() {
    conn, err = grpc.Dial(...）
    if err != nil {
        panic(err)
    }

    svc := post.NewService(conn)
    posts, err := svc.ListPosts()
    if err != nil {
        panic(err)
    }

    fmt.Println(posts)
}
```

### 函数简单

功能上的简单、单一，函数容易理解并且命名能够自解释。

## 组织方式

Golang 中的单元测试文件和代码都是与源代码放在同一个目录下按照 package 进行组织的，`server.go` 文件对应的测试代码应该放在同一目录下的 `server_test.go` 文件中。

### Test

单元测试的最常见以及默认组织方式就是写在以 `_test.go` 结尾的文件中，所有的测试方法也都是以 Test 开头并且只接受一个 `testing.T` 类型的参数：

```go
func TestAuthor(t *testing.T) {
    author := blog.Author()
    assert.Equal(t, "draveness", author)
}
```

如果我们要给函数名为 `Add` 的方法写单元测试，那么对应的测试方法一般会被写成 `TestAdd`，为了同时测试多个分支的内容，我们可以通过以下的方式组织 `Add` 函数相关的测试：

```go
func TestAdd(t *testing.T) {
    assert.Equal(t, 5, Add(2, 3))
}

func TestAddWithNegativeNumber(t *testing.T) {
    assert.Equal(t, -2, Add(-1, -1))
}
```

除了这种将一个函数相关的测试分散到多个 Test 方法之外，我们可以使用 for 循环来减少重复的测试代码，这在逻辑比较复杂的测试中会非常好用，能够减少大量的重复代码，不过也需要我们小心地进行设计：

```go
func TestAdd(t *testing.T) {
    tests := []struct{
        name     string
        first    int64
        second   int64
        expected int64
    } {
        {
            name:     "HappyPath":
            first:    2,
            second:   3,
            expected: 5,
        },
        {
            name:     "NegativeNumber":
            first:    -1,
            second:   -1,
            expected: -2,
        },
    }

    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            assert.Equal(t, test.expected, Add(test.first, test.second))
        })
    }
}
```

这种方式其实也能生成树形的测试结果，将 `Add` 相关的测试分成一组方便我们进行观察和理解，不过这种测试组织方法需要我们保证测试代码的通用性，当函数依赖的上下文较多时往往需要我们写很多的 `if/else` 条件判断语句影响我们对测试的快速理解。

有一种实践原则是通常会在测试代码比较简单时使用第一种组织方式，而在依赖较多、函数功能较为复杂时使用第二种方式。

### Suite

第二种比较常见的方式是按照簇进行组织，其实就是对 Go 语言默认的测试方式进行简单的封装，我们可以使用 [stretchr/testify](https://github.com/stretchr/testify) 中的 suite 包对测试进行组织：

```go
import (
    "testing"
    "github.com/stretchr/testify/suite"
)

type ExampleTestSuite struct {
    suite.Suite
    VariableThatShouldStartAtFive int
}

func (suite *ExampleTestSuite) SetupTest() {
    suite.VariableThatShouldStartAtFive = 5
}

func (suite *ExampleTestSuite) TestExample() {
    suite.Equal(suite.VariableThatShouldStartAtFive, 5)
}

func TestExampleTestSuite(t *testing.T) {
    suite.Run(t, new(ExampleTestSuite))
}
```

可以使用 suite 包，以结构体的方式对测试簇进行组织，suite 提供的 `SetupTest/SetupSuite` 和 `TearDownTest/TearDownSuite` 是执行测试前后以及执行测试簇前后的钩子方法，我们能在其中完成一些共享资源的初始化，减少测试中的初始化代码。

### BDD

最后一种组织代码的方式就是使用 BDD 的风格对单元测试进行组织，ginkgo 就是 Golang 社区最常见的 BDD 框架。

```go
var _ = Describe("Book", func() {
    var (
        book Book
        err error
    )

    BeforeEach(func() {
        book, err = NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)
    })

    Describe("loading from JSON", func() {
        Context("when the JSON fails to parse", func() {
            BeforeEach(func() {
                book, err = NewBookFromJSON(`{
                    "title":"Les Miserables",
                    "author":"Victor Hugo",
                    "pages":1488oops
                }`)
            })

            It("should return the zero-value for the book", func() {
                Expect(book).To(BeZero())
            })

            It("should error", func() {
                Expect(err).To(HaveOccurred())
            })
        })
    })
})
```

BDD 框架中一般都包含 `Describe、Context` 以及 `It` 等代码块，其中 `Describe` 的作用是描述代码的独立行为、`Context` 是在一个独立行为中的多个不同上下文，最后的 `It` 用于描述期望的行为，这些代码块最终都构成了类似 **『描述……，当……时，它应该……』** 的句式帮助我们快速地理解测试代码。

## Mock 方法

* 项目中的单元测试应该是稳定的并且不依赖任何的外部项目，它只是对项目中函数和方法的测试，所以我们需要在单元测试中对所有的第三方的不稳定依赖进行 Mock，也就是模拟这些第三方服务的接口
* 除此之外，为了简化一次单元测试的上下文，在同一个项目中我们也会对其他模块进行 Mock，模拟这些依赖模块的返回值。

单元测试的核心就是隔离依赖并验证输入和输出的正确性，Go 语言作为一个静态语言提供了比较少的运行时特性，这也让我们在 Go 语言中 Mock 依赖变得非常困难。

Mock 的主要作用就是保证待测试方法依赖的上下文固定，在这时无论我们对当前方法运行多少次单元测试，如果业务逻辑不改变，它都应该返回完全相同的结果，在具体介绍 Mock 的不同方法之前，我们首先要清楚一些常见的依赖，一个函数或者方法的常见依赖可以有以下几种：

1. 接口
2. 数据库
3. HTTP 请求
4. Redis、缓存以及其他依赖

### 接口

首先要介绍的其实就是 Go 语言中最常见也是最通用的 Mock 方法，也就是能够对接口进行 Mock 的 [golang/mock](https://github.com/golang/mock) 框架，它能够根据接口生成 Mock 实现，假设我们有以下代码：

```go
package blog

type Post struct {}

type Blog interface {
	ListPosts() []Post
}

type jekyll struct {}

func (b *jekyll) ListPosts() []Post {
 	return []Post{}
}

type wordpress struct{}

func (b *wordpress) ListPosts() []Post {
	return []Post{}
}
```

假设我们有个博客，那就需要一个 `ListsPosts` 方法用于返回全部的文章列表，在这时我们就需要定义一个 `Post` 接口，接口要求遵循 `Blog` 的结构体必须实现 `ListPosts` 方法。

当我们定义好了 `Blog` 接口之后，上层 Service 就不再需要依赖某个具体的博客引擎实现了，只需要依赖 `Blog` 接口就可以完成对文章的批量获取功能：

```go
package service

type Service interface {
	ListPosts() ([]Post, error)
}

type service struct {
    blog blog.Blog
}

func NewService(b blog.Blog) *Service {
    return &service{
        blog: b,
    }
}

func (s *service) ListPosts() ([]Post, error) {
    return s.blog.ListPosts(), nil
}
```

如果我们想要对 `Service` 进行测试，我们就可以使用 gomock 提供的 `mockgen` 工具命令生成 `MockBlog` 结构体，使用如下所示的命令：

```go
$ mockgen -package=mblog -source=pkg/blog/blog.go > test/mocks/blog/blog.go

$ cat test/mocks/blog/blog.go
// Code generated by MockGen. DO NOT EDIT.
// Source: blog.go

// Package mblog is a generated GoMock package.
...
// NewMockBlog creates a new mock instance
func NewMockBlog(ctrl *gomock.Controller) *MockBlog {
	mock := &MockBlog{ctrl: ctrl}
	mock.recorder = &MockBlogMockRecorder{mock}
	return mock
}

// EXPECT returns an object that allows the caller to indicate expected use
func (m *MockBlog) EXPECT() *MockBlogMockRecorder {
	return m.recorder
}

// ListPosts mocks base method
func (m *MockBlog) ListPosts() []Post {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "ListPosts")
	ret0, _ := ret[0].([]Post)
	return ret0
}

// ListPosts indicates an expected call of ListPosts
func (mr *MockBlogMockRecorder) ListPosts() *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "ListPosts", reflect.TypeOf((*MockBlog)(nil).ListPosts))
}
```

这段 mockgen 生成的代码非常长的，所以我们只展示了其中的一部分，它的功能就是帮助我们验证任意接口的输入参数并且模拟接口的返回值。

而在生成 Mock 实现的过程中，这里有一些可以分享的经验：

* 在 `test/mocks` 目录中放置所有的 Mock 实现，子目录与接口所在文件的二级目录相同，在这里源文件的位置在 `pkg/blog/blog.go`，它的二级目录就是 `blog/`，所以对应的 Mock 实现会被生成到 `test/mocks/blog/` 目录中
* 指定 package 为 `mxxx`，默认的 `mock_xxx` 看起来非常冗余，上述 blog 包对应的 Mock 包也就是 `mblog`
* mockgen 命令放置到 Makefile 中的 mock 下统一管理，减少祖传命令的出现

Mock 这块还是需要将来在实际工作中再慢慢补全。

### SQL

在遇到数据库的依赖时，我们一般都会使用 sqlmock 来模拟数据库的连接，当我们使用 sqlmock 时会写出如下所示的单元测试：

```go
func (s *suiteServerTester) TestRemovePost() {
	entry := pb.Post{
		Id: 1,
	}

	rows := sqlmock.NewRows([]string{"id", "author"}).AddRow(1, "draveness")

	s.Mock.ExpectQuery(`SELECT (.+) FROM "posts"`).WillReturnRows(rows)
	s.Mock.ExpectExec(`DELETE FROM "posts"`).
		WithArgs(1).
		WillReturnResult(sqlmock.NewResult(1, 1))

	response, err := s.server.RemovePost(context.Background(), &entry)

	s.NoError(err)
	s.EqualValues(response, &entry)
	s.NoError(s.Mock.ExpectationsWereMet())
}
```

最常用的几个方法就是 `ExpectQuery` 和 `ExpectExec`，前者主要用于模拟 SQL 的查询语句，后者用于模拟 SQL 的增删，从上面的实例中我们可以看到这个这两种方法的使用方式，建议各位先阅读相关的文档再尝试使用。

### HTTP

httpmock 就是一个用于 Mock 所有 HTTP 依赖的包，它使用模式匹配的方式匹配 HTTP 请求的 URL，在匹配到特定的请求时就会返回预先设置好的响应。

```go
func TestFetchArticles(t *testing.T) {
	httpmock.Activate()
	defer httpmock.DeactivateAndReset()

	httpmock.RegisterResponder("GET", "https://api.mybiz.com/articles",
		httpmock.NewStringResponder(200, `[{"id": 1, "name": "My Great Article"}]`))

	httpmock.RegisterResponder("GET", `=~^https://api\.mybiz\.com/articles/id/\d+\z`,
		httpmock.NewStringResponder(200, `{"id": 1, "name": "My Great Article"}`))

	...
}
```

如果遇到 HTTP 请求的依赖时，就可以使用上述 httpmock 包模拟依赖的 HTTP 请求。

### 猴子补丁

略

### 断言

在最后，我们简单介绍一下辅助单元测试的 assert 包，它提供了非常多的断言方法帮助我们快速对期望的返回值进行测试，减少我们的工作量：

```go
func TestSomething(t *testing.T) {
  assert.Equal(t, 123, 123, "they should be equal")

  assert.NotEqual(t, 123, 456, "they should not be equal")

  assert.Nil(t, object)

  if assert.NotNil(t, object) {
    assert.Equal(t, "Something", object.Value)
  }
}
```

## 相关链接

* [如何写出优雅的 Golang 代码](https://draveness.me/golang-101)
* [stretchr/testify](https://github.com/stretchr/testify)
* [ginkgo](https://github.com/onsi/ginkgo)
* [golang/mock](https://github.com/golang/mock)
* [sqlmock](https://github.com/DATA-DOG/go-sqlmock)
* [httpmock](https://github.com/jarcoal/httpmock)
