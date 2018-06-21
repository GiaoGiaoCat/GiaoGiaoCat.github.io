---
layout: post
title:  "Installing a specific version of a homebrew package"
date:   2015-11-25 22:00:00
categories: tool
tags: terminal
author: "Victor"
---

## Installing from a tap

```bash
brew tap homebrew/versions
brew search elasticsearch
brew install homebrew/versions/elasticsearch11
```
## Installing from a specific commit

```bash
brew tap homebrew/boneyard # Then you can run brew versions.
brew versions elasticsearch
brew versions elasticsearch | grep 1.2.1
```

```bash
cd /usr/local/Library
git checkout 3bbd4f1 /usr/local/Library/Formula/elasticsearch.rb
brew install elasticsearch
```

```bash
brew unlink elasticsearch
brew install elasticsearch
```

## 相关链接

* [Homebrew-versions](https://github.com/Homebrew/homebrew-versions)
* [Installing a specific version of a homebrew package](http://effectif.com/mac-os-x/installing-specific-version-of-homebrew-formula)
* [Switching Package Versions with Brew](http://thejacklawson.com/2012/09/switching-package-versions-with-brew/)
* [Homebrew install specific version of formula?](http://stackoverflow.com/questions/3987683/homebrew-install-specific-version-of-formula)
