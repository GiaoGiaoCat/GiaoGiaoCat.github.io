---
layout: post
title:  "使用 interface 进行缓存"
date:   2019-11-01 17:10:00

categories: golang
tags: tip
author: "Victor"
---

缓存很难实现的很好，并且很难测试。

## 问题

假设我们要缓存 Github API 的调用。假设我们需要缓存我们的 repo 列表。

```go
package client

type Repository struct {
  Name string `json:"name"`
}

func GetRepositories() ([]Repository, error) {
  // TODO: do a call to https://api.github.com/users/caarlos0/repos
  return []Repository{}, nil
}
```

因为我们是从另外一个网站发起调用，所以很容易被限制调用次数。因为这个数据变化较少，用户不会在意看到的是 5 分钟前的数据。

我们可以很容易的使用 [go-cache](https://github.com/patrickmn/go-cache) 做到这一点。

```go
var cache = cache.New(5*time.Minute, 5*time.Minute)

func GetRepositories() ([]Repository, error) {
  cached, found := cache.Get("my-repos")
  if found {
    return cached.([]Repository), nil
  }
  // TODO: do a call to https://api.github.com/users/caarlos0/repos
  // result := blah
  c.cache.Set(repo, result, cache.DefaultExpiration)
  return []Repository{}, nil
}
```

这种实现方案会有如下问题：

* 缓存是全局的
* 客户端现在对缓存有严格的依赖性
* 无法单独测试缓存逻辑

## 使用 interfaces 解决这个问题

解决的方法是创建一个接口，并用另一个仅处理缓存的实现来处理客户端实现。

例如，我们可以创建如下的接口：

```go
// client.go
package client

type Repository struct {
  Name string `json:"name"`
}

type Client interface {
  GetRepositories() ([]Repository, error)
}
```

下面是它的实现：

```go
// github.go
package client

func NewGithubClient() Client {
  return ghClient{}
}

type ghClient struct {}

func (ghClient) GetRepositories() ([]Repository, error) {
  // TODO: do a call to https://api.github.com/users/caarlos0/repos
  return []Repository{}, nil
}
```

最后，用一个包含其它 client 的实现来处理缓存。

```go
// cache.go
package client

func NewCachedClient(client Client, cache *cache.Cache) Client {
  return cachedClient{
    client: client,
    cache:  cache,
  }
}

type cachedClient struct {
  client Client
  cache *cache.Cache
}

func (c cachedClient) GetRepositories() ([]Repository, error) {
  cached, found := c.cache.Get("my-repos")
  if found {
    return cached.([]Repository), nil
  }
  // call the underlying client
  live, err := c.client.GetRepositories()
  c.cache.Set(repo, result, cache.DefaultExpiration)
  return live, err
}
```

现在，我们可以对缓存进行单独测试了。

## 测试

在我们的示例中，我们可以轻松测试缓存实现，我们只需要创建一个伪造的客户端实现并将其包装在 `cachedClient` 中，然后为其编写一些测试。

一个简单的实现示例：

```go
// cache_test.go
package client

type cacheTestClient struct {
  result *[]Repository
}

func (f cacheTestClient) GetRepositories() ([]Release, error) {
  return *f.result, nil
}

func TestCachedClient(t *testing.T) {
  var cache = cache.New(1*time.Minute, 1*time.Minute)
  var expected = []Repository{
    { Name: "caarlos0/version_exporter" },
    { Name: "caarlos0/dotfiles" },
    { Name: "caarlos0/carlosbecker.com" },
  }

  var cli = NewCachedClient(cacheTestClient{result: &expected}, cache)

  // test getting from out fake client
  t.Run("get fresh", func(t *testing.T) {
    res, err := cli.GetRepositories()
    require.NoError(t, err)
    require.Equal(t, expected, res)
  })


  // here we change the inner fake client result, but the result
  // should be the cached one
  t.Run("get from cache", func(t *testing.T) {
    var oldExpected = expected
    expected = append(rel, Repository{Name: "caarlos0/env"})
    res, err := cli.GetRepositories()
    require.NoError(t, err)
    require.Equal(t, oldExpected, res)
  })


  // here we flush the cache and verify that the result is the one
  // from the fake client
  t.Run("flush cache", func(t *testing.T) {
    c.Flush()
    res, err := cli.GetRepositories()
    require.NoError(t, err)
    require.Equal(t, expected, res)
  })
}
```

尽管这是一个简单的示例，但它能用。

您可以编写一个更聪明的假客户端（比如上面的例子就没处理错误），并且还可以将此策略用于 Redis 支持的缓存，其它类似的 API 调用或 SQL 数据库。

这种 **接口 + 装饰器模式** 的组合可以用于很多场合，例如断路器之类的场景。

## 原文链接

[Golang: cache things using interfaces](https://carlosbecker.com/posts/golang-cache-interface/)
