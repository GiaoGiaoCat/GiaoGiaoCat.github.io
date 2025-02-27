---
layout: post
title:  "Using private key in a .env file"
date:   2020-12-10 18:10:00

categories: tool
tags: tip
author: "Victor"
---

## 问题

需要把一个 private_secret_key 放到 .env 文件中。

## 解决方案

执行 `echo "PRIVATE_KEY=\"`sed -E 's/$/\\\n/g' my_rsa_2048_priv.pem`\"" >> .env`

获得如下结果：

```bash
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\n
dasdasdadasdasdasdasdasdasdasdadasdasdadasa\n
huehuauhhuauhahuauhauahuauhehuehuauheuhahue\n
-----END RSA PRIVATE KEY-----\n"
```

把它们合成一行。

```bash
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\ndasdasdadasdasdasdasdasdasdasdadasdasdadasa\nhuehuauhhuauhahuauhauahuauhehuehuauheuhahue\n-----END RSA PRIVATE KEY-----\n"
```

## 参考

* https://stackoverflow.com/questions/55459528/using-private-key-in-a-env-file
