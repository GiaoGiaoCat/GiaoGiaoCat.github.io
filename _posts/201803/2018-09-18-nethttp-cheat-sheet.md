---
layout: post
title:  "Net::HTTP 小抄"
date:   2018-09-18 14:00:00

categories: ruby
tags: tip
author: "Victor"
---

## 小抄合集

可能你更喜欢使用 `HTTParty, Typhoeus, rest-client` 之类封装更好的库，但我还是认为 `Net::HTTP` 是最无痛的方式，因为它是标准库啊。

### Standard HTTP Request

```ruby
require "net/http"
require "uri"

uri = URI.parse("http://google.com/")

# Shortcut
response = Net::HTTP.get_response(uri)

# Will print response.body
Net::HTTP.get_print(uri)

# Full
http = Net::HTTP.new(uri.host, uri.port)
response = http.request(Net::HTTP::Get.new(uri.request_uri))
```

### Basic Auth

```ruby
require "net/http"
require "uri"

uri = URI.parse("http://google.com/")

http = Net::HTTP.new(uri.host, uri.port)
request = Net::HTTP::Get.new(uri.request_uri)
request.basic_auth("username", "password")
response = http.request(request)
```

### Dealing with response objects

```ruby
require "net/http"
require "uri"

uri = URI.parse("http://google.com/")

http = Net::HTTP.new(uri.host, uri.port)
request = Net::HTTP::Get.new(uri.request_uri)

response = http.request(request)

response.code             # => 301
response.body             # => The body (HTML, XML, blob, whatever)
# Headers are lowercased
response["cache-control"] # => public, max-age=2592000
```

### POST form request

```ruby
require "net/http"
require "uri"

uri = URI.parse("http://example.com/search")

# Shortcut
response = Net::HTTP.post_form(uri, {"q" => "My query", "per_page" => "50"})

# Full control
http = Net::HTTP.new(uri.host, uri.port)

request = Net::HTTP::Post.new(uri.request_uri)
request.set_form_data({"q" => "My query", "per_page" => "50"})

response = http.request(request)
```

### File upload - input type="file" style

```ruby
require "net/http"
require "uri"

# Token used to terminate the file in the post body. Make sure it is not
# present in the file you're uploading.
BOUNDARY = "AaB03x"

uri = URI.parse("http://something.com/uploads")
file = "/path/to/your/testfile.txt"

post_body = []
post_body < < "--#{BOUNDARY}rn"
post_body < < "Content-Disposition: form-data; name="datafile"; filename="#{File.basename(file)}"rn"
post_body < < "Content-Type: text/plainrn"
post_body < < "rn"
post_body < < File.read(file)
post_body < < "rn--#{BOUNDARY}--rn"

http = Net::HTTP.new(uri.host, uri.port)
request = Net::HTTP::Post.new(uri.request_uri)
request.body = post_body.join
request["Content-Type"] = "multipart/form-data, boundary=#{BOUNDARY}"

http.request(request)
```

### SSL/HTTPS request

```ruby
require "net/https"
require "uri"

uri = URI.parse("https://secure.com/")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true
http.verify_mode = OpenSSL::SSL::VERIFY_NONE

request = Net::HTTP::Get.new(uri.request_uri)

response = http.request(request)
response.body
response.status
response["header-here"] # All headers are lowercase
```

### SSL/HTTPS request with PEM certificate

```ruby
require "net/https"
require "uri"

uri = URI.parse("https://secure.com/")
pem = File.read("/path/to/my.pem")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true
http.cert = OpenSSL::X509::Certificate.new(pem)
http.key = OpenSSL::PKey::RSA.new(pem)
http.verify_mode = OpenSSL::SSL::VERIFY_PEER

request = Net::HTTP::Get.new(uri.request_uri)
```

### REST methods

```ruby
# Basic REST.
# Most REST APIs will set semantic values in response.body and response.code.
require "net/http"

http = Net::HTTP.new("api.restsite.com")

request = Net::HTTP::Post.new("/users")
request.set_form_data({"users[login]" => "quentin"})
response = http.request(request)
# Use nokogiri, hpricot, etc to parse response.body.

request = Net::HTTP::Get.new("/users/1")
response = http.request(request)
# As with POST, the data is in response.body.

request = Net::HTTP::Put.new("/users/1")
request.set_form_data({"users[login]" => "changed"})
response = http.request(request)

request = Net::HTTP::Delete.new("/users/1")
response = http.request(request)
```

## 原文

* [Net::HTTP Cheat Sheet](http://www.rubyinside.com/nethttp-cheat-sheet-2940.html)
* [A collection of Ruby Net::HTTP examples.](https://github.com/augustl/net-http-cheat-sheet)
