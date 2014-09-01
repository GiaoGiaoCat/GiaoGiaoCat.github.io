---
layout: post
title:  "Building a robust JSON API client with Ruby"
date:   2014-09-01 12:10:00
categories: rails
tags: apis
author: "Victor"
---

如果你正在构建一个基于流行的第3方 API 服务器的 Ruby on Rails 的网站，可能你会选用一些现有的 Gem 然后只需简单的几行代码来查询并返回你所需要的数据。

但是这些 gem 有可能引入了你不知道的慢查询，或者需要使用一些超大的 lib，或者你根本不理解的 code。

你可以选择自己写 60 行代码，而不是引入一个 60k 的 Ruby 代码。本文就是讲解如何使用 HTTParty 来实现一个小型的 API 客户端。

## Testing the API

没有什么比浪费一上午时间调试 API 却发现你的 API 密钥是无效或失效更让人发疯。例如:

```
http://www.pixel-peeper.com/rest/?method=list_photos&api_key=[REDACTED]&camera=[CAMERAID]
```

如果是 **GET** 请求你可以使用浏览器测试，或者用 **curl** 来测试。

```json
{
  data: {
    results: [
      {
        camname: "EOS 5D Mark III",
        cammake: "Canon",
        camexifid: "CANON EOS 5D MARK III",
        lensname: "Canon EF 24-105mm f/4 L IS USM",
        author_name: null,
        author_url: null,
        id: "8175491230",
        iso: "1600",
        aperture: "4",
        exposure: "0.00625",
        focal_length: "105",
        small_url: "http://farm9.staticflickr.com/8478/8175491230_c94258586b_m.jpg",
        pixels: "22118400",
        lens_id: "920",
        flickr_user_id: "23722023@N06",
        camera_id: "1659",
        big_url: "https://www.flickr.com/photos/23722023@N06/8175491230/sizes/o/"
        },
      ...
    ]
  }
}
```

## Minimum viable integration

现在我们已经手动确认 API 是工作的了，下面需要进行和 API 交互的 Ruby 代码了。

Ruby 的标准库 [open-uri][open-uri] 提供 ```open(url)```。但是它不能很好的处理 timeouts 的情况。我发现有个比较好的解决方法是 require 'timeout' 然后使用 [Timeout::timeout(seconds)][timeout-block] 代码块来包含要执行的代码，虽然这看起来有点傻。

[Net::HTTP][net-http] 是一个非常棒的处理 HTTP 请求的库。但是我更推荐 [HTTParty][http-party] 它的内容也使用 Net::HTTP 。 HTTParty 提供了很多非常有用的样板进行 API 交互，且容易测试。如果你需要和一个复杂且参数挑剔的 API 进行交互，我也建议你试试 [faraday][faraday]。不过今天我们使用 HTTParty 已经足够了。

利用 HTTParty 我们自己实现了一个较小的库：

```ruby
require 'httparty'

class PixelPeeper
  include HTTParty
  base_uri 'www.pixel-peeper.com'

  def api_key
    ENV['PIXELPEEPER_API_KEY']
  end

  def base_path
    "/rest/?method=list_photos&api_key=#{ api_key }"
  end

  def examples_for_camera(camera_id, options = {})
    url = "#{ base_path }&camera=#{ camera_id }"
    self.class.get(url, options)['data']['results']
  end

  def examples_for_lens(lens_id, options = {})
    url = "#{ base_path }&lens=#{ lens_id }"
    self.class.get(url, options)['data']['results']
  end
end
```

不在源代码中布包含 API 密码是最佳实践。所以我们从环境变量中读取 key。如果你使用 Heroku，你需要通过 ```heroku config:set PIXELPEEPER_API_KEY=$key``` 来设置自己的 key。对于我们本地的开发环境，我们需要到处变量或者在命令行内设置这个 key。

由于我们在类中包含了 HTTParty，所以 API GET 请求会通过 ```self.class.get``` 方法调用 HTTParty 的方法。例如使用 ```base_uri``` 会自动解码 JSON 请求为 HASH。正因为 HTTParty 所以我们不必写太多的代码就实现了一个全功能的 API 客户端。

## Timeouts

目前为止它一个全功能的，但并不是一个强大的API客户端。如果我们的 Rails app 在一个请求内部调用了一个 API。而这个 API 响应要用 30 秒的时间。那么多次请求同一个缓慢的 API 会占用所有的 web servers 并且把网站搞挂了。所以对于这种场景最好的做法是硬超时(hard timeout)和优雅降级(gracefully degrade)。在我们的例子中，我们返回了一个 "fake" 空请求，所以模板会优雅降级。

HTTParty 会在无法连接服务器的时候，抛出 ```Net::OpenTimeout```，并在请求超时的时候抛出 ```Net::ReadTimeout```。这两者的区别是一个已经停止发送数据，一个仍在发送数据。所以我们想让我们的程序更健壮，只需要处理这两个异常返回空哈希，并且将超时时间设置为1秒。我使用 ```handle_timeouts``` 函数，它接受一个代码块。如果抛出一个异常，```handle_timeouts``` 将捕获该异常，并返回一个空哈希。

```ruby
require 'httparty'

class PixelPeeper
  include HTTParty
  base_uri 'www.pixel-peeper.com'
  default_timeout 1 # hard timeout after 1 second

  def api_key
    ENV['PIXELPEEPER_API_KEY']
  end

  def base_path
    "/rest/?method=list_photos&api_key=#{ api_key }"
  end

  def handle_timeouts
    begin
      yield
    rescue Net::OpenTimeout, Net::ReadTimeout
      {}
    end
  end

  def examples_for_camera(camera_id, options = {})
    handle_timeouts do
      url = "#{ base_path }&camera=#{ camera_id }"
      self.class.get(url, options)['data']['results']
    end
  end

  def examples_for_lens(lens_id, options = {})
    handle_timeouts do
      url = "#{ base_path }&lens=#{ lens_id }"
      self.class.get(url, options)['data']['results']
    end
  end
end
```

现在，这个类已经足够满足我们生产环境的要求了。现在最坏的情况无非就是因为 API 服务器的问题，造成我们浪费了1秒钟时间但是却返回了空内容而已。它不会造成我们的 Rails 服务因此会挂掉。

## Caching API responses locally

通过上一步我们保护了我们的网站，但是我们仍然能看到每次都会发送很多请求和响应。通过 Redis 的本地缓存，我们可以无需请求外部 API，并在几毫秒内返回缓存数据。此外，当 API 服务出现故障的时候，我们也不出出现没有数据展示的问题。并且缓存可以帮我们避免重复请求。当进行下面的步骤之前，请小心，有一些 API 服务的协议和授权声明禁止缓存它们的数据。但是，除非敏感数据，否则大部分 API 服务器希望你能缓存数据，这样可以较少 API 服务器的负载。

继 ```handle_timeouts``` 之后，我们用 ```handle_caching``` 方法来缓存数据。该方法接受一个代码块和缓存参数。camera 和 lense 方法会折叠进 ```examples(options)``` 方法，这里 options 是一个哈希，由 ```camera_id``` 或 ```lens_id``` 的键值组成的集合。

```ruby
require 'httparty'

class PixelPeeper
  include HTTParty
  base_uri 'www.pixel-peeper.com'
  default_timeout 1 # hard timeout after 1 second

  def api_key
    ENV['PIXELPEEPER_API_KEY']
  end

  def base_path
    "/rest/?method=list_photos&api_key=#{ api_key }"
  end

  def handle_timeouts
    begin
      yield
    rescue Net::OpenTimeout, Net::ReadTimeout
      {}
    end
  end

  def cache_key(options)
    if options[:camera_id]
      "pixelpeeper:camera:#{ options[:camera_id] }"
    elsif options[:lens_id]
      "pixelpeeper:lens:#{ options[:lens_id] }"
    end
  end

  def handle_caching(options)
    if cached = REDIS.get(cache_key(options))
      JSON[cached]
    else
      yield.tap do |results|
        REDIS.set(cache_key(options), results.to_json)
      end
    end
  end

  def build_url_from_options(options)
    if options[:camera_id]
      "#{ base_path }&camera=#{ options[:camera_id] }"
    elsif options[:lens_id]
      "#{ base_path }&lens=#{ options[:lens_id] }"
    else
      raise ArgumentError, "options must specify camera_id or lens_id"
    end
  end

  def examples(options)
    handle_timeouts do
      handle_caching(options) do
        self.class.get(build_url_from_options(options))['data']['results']
      end
    end
  end
end
```

所有缓存的逻辑都在 ```handle_caching(options)``` 和 ```cache_key(options)```，方法中。后者创建唯一的 key 用来给每一条基于 ID 查询的请求，按照其响应类型储存其缓存结果。Redis 鼓励可读性好的键名，但是当你的键名越来越长且变的笨拙或者难以预测的时候，使用例如 ```SHA-1(options.to_json)``` 这样的唯一性键名是完全合理的。

```handle_caching(options)``` 检查 key 是否存在，如果存在就返回有效的值，否则就产生传入的代码块，并储存到 Redis。```Object#tap``` 是一个小技巧，它总是返回对象本身。但传递给该对象一个代码块，并执行。这是一个很好的模式，当你完成计算并返回值，但仍需要引用它的时候。比如：储存到缓存，打印到日志，发送邮件等等。本文最后附上 ```tap``` 的示例代码。

## Consuming the API

使用 API​​ 的客户端非常简单：一行代码来创建一个 API 的客户端实例，一行代码由 ```camera_id``` 或 ```lens_id``` 查询。

```ruby
def example_pictures_for(gear)
  pp = PixelPeeper.new
  if gear.pp_lens_id.present?
    pp.examples(lens_id: gear.pp_lens_id)
  elsif gear.pp_camera_id.present?
    pp.examples(camera_id: gear.pp_camera_id)
  else
    []
  end.take(8)
end
```

## Summary and final thoughts

正如我们所展示的，HTTParty 可以用来快速创建一个强大和高性能的 JSON API 的客户端。使用该辅助方法并结合代码块允许我们创建组合的辅助方法，并根据各自职责的保持代码分离，而不是混合在一起。

下次想要自己实现一个小型 API 客户端的时候，再也不用担心了。

### 原文链接

* [Building a robust JSON API client with Ruby](http://www.binpress.com/tutorial/ruby-tutorial-building-a-robust-json-api-client/140)

### Object.tap method

```tap{|x|...} → obj```

Yields x to the block, and then returns x. The primary purpose of this method is to “tap into” a method chain, in order to perform operations on intermediate results within the chain.

```ruby
(1..10)                .tap {|x| puts "original: #{x.inspect}"}
  .to_a                .tap {|x| puts "array: #{x.inspect}"}
  .select {|x| x%2==0} .tap {|x| puts "evens: #{x.inspect}"}
  .map { |x| x*x }     .tap {|x| puts "squares: #{x.inspect}"}
```

[open-uri]: http://www.ruby-doc.org/stdlib-2.1.2/libdoc/open-uri/rdoc/OpenURI.html
[timeout-block]: https://www.ruby-forum.com/topic/77694
[net-http]: http://www.ruby-doc.org/stdlib-2.1.2/libdoc/net/http/rdoc/Net/HTTP.html
[http-party]: https://github.com/jnunemaker/HTTParty
[faraday]: https://github.com/lostisland/faraday
