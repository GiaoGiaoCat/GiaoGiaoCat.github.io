---
layout: post
title:  "Handling HTTP Request Errors"
date:   2019-04-31 13:00:00

categories: go
tags: tip
author: "Victor"
---

写个简单的小例子，自己做个端点，返回状态码 500。

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/500", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(500)
		w.Write([]byte("NOT-OK"))
	})
	go http.ListenAndServe(":8080", nil)

	_, err := http.Get("http://localhost:8080/500")
	if err != nil {
		log.Fatal(err)
	}
}
```

我们期望抛错的地方并没有执行，原因是因为。

> An error is returned if the Client’s CheckRedirect function fails or if there was an HTTP protocol error. A non-2xx response doesn’t cause an error.

所以 GET 请求应该按照下面的写法，自己手动处理非 200 的状态码。

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	resp, err := http.Get("https://httpbin.org/get")
	// 域名无法访问的时候才会抛错
	if err != nil {
		// Print the message and then calls os.Exit to terminate the program.
		log.Fatalln(err)
	}

	// An error is returned if there were too many redirects or if there was an HTTP protocol error.
	// A non-2xx response doesn’t cause an error.
	// 需要我们自己处理状态码非 200 的情况
	if resp.StatusCode != 200 {
		b, _ := ioutil.ReadAll(resp.Body)
		fmt.Println("second err")
		log.Fatal(string(b))
	}
	// Forgetting to close the response body can cause resource leaks in a long running programs.
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("second err")
		log.Fatalln(err)
	}

	log.Println(string(body))
}
```

## 原文

* [Handling HTTP Request Errors in GO](http://www.metabates.com/2015/10/15/handling-http-request-errors-in-go/)
* [HANDLE HTTP REQUEST ERRORS IN GO](https://pliutau.com/handle-http-request-errors-in-go/)
