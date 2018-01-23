---
layout: post
title:  "Yarn 基础"
date:   2017-12-25 10:30:00
categories: javascript
tags: rails5 javascript
author: "Victor"
---

## 开始使用

Yarn 是一个依赖管理工具。它能够管理你的代码，也可以使用其他开发者开发的代码，让你更容易的开发软件。代码是通过包（有时也被称为模块）进行共享的。在每一个包中包含了所有需要共享的代码，另外还定义了一个 `package.json` 文件，用来描述这个包。

### 安装

通过 [Homebrew](https://brew.sh/) 包管理工具安装 Yarn。

```bash
brew update
brew install yarn
```

你需要设置你终端的 `PATH` 环境变量，以便全局访问 Yarn 的可执行文件。添加 `export PATH="$PATH:`yarn global bin`"` 到你的 profile 文件（可能是 `.profile、.bashrc、.zshrc` 等文件）。

测试一下 Yarn 是否能够正确运行：`yarn --version`。

### 用法

```bash
# 初始化一个新的项目
yarn init

# 添加一个依赖包
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]

# 更新一个依赖包
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]

# 删除一个依赖包
yarn remove [package]

# 安装所有的依赖包
yarn
yarn install
# Forcing a re-download of all packages
yarn install --force
# Installing only production dependencies
yarn install --production
```

### 必要的文件

为了能够让任何人继续开发或使用你的包，下列文件必须确保被提交到源码管理系统中：

* `package.json` 该文件记录了此包的所有依赖。
* `yarn.lock` 该文件保存了此包的每个依赖的确切版本号。


### 持续集成

Yarn 能够非常容易地用于不同持续集成系统中。Yarn 用于加速构建过程，通过保存 Yarn 的缓存目录可以在不同构建过程中重复使用。参看[此链接](https://yarn.bootcss.com/docs/install-ci.html#semaphore-tab)。

### 其它命令

* `yarn outdated` Lists version information for all package dependencies.
* `yarn list` List installed packages.
* `yarn run` Runs a defined package script.

You may define scripts in your package.json file.

```json
{
  "name": "my-package",
  "scripts": {
    "build": "babel src -d lib",
    "test": "jest"
  }
}
```

```bash
yarn run test
yarn run test -- -o --watch
```
