---
layout: post
title:  "同一台电脑配置多个 git 账号"
date:   2019-08-08 17:10:00

categories: tool
tags: terminal server
author: "Victor"
---

离职了，不用学 Java 了。开心，专心写 Go。

## 背景

为了假装我们团队研发力量强大，需要一个人伪装 10 个不同的身份在同不同的仓库中提交代码。嗯，你信吗？

## 步骤

首先按照 Github 的说明先 [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。

修改 `~/.ssh/config`

```
Host *
  ServerAliveInterval 30
  AddKeysToAgent yes
  UseKeychain yes

# Default
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa

# Other
Host puppy.github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_2
```

执行 ssh-agent 让 ssh 识别新的私钥，因为默认只读取 id_rsa，为了让 SSH 识别新的私钥，需将其添加到 SSH agent 中：

```
ssh-add ~/.ssh/id_rsa_2
```

如果你有多台电脑，你的 `id_rsa_2, id_rsa_2.pub` 是从别的电脑同步过来的。有可能遇到 `WARNING: UNPROTECTED PRIVATE KEY FILE!` 错误。

```
sudo chmod 600 ~/.ssh/id_rsa_2
sudo chmod 600 ~/.ssh/id_rsa_2.pub
```

这时候，我们在 pull 和 push 代码的时候，身份识别会有问题，所以需要取消全局设置。

```
git config --global --unset user.name
git config --global --unset user.email
```

然后测试一波，上面的操作是否生效。

```
ssh -T git@puppy.github.com
```

接下来 clone 项目了。为了使用新身份，需要修改路径。比如原来是 `git@github.com:wjp2013/wjp2013.github.io.git`，要改成 `git@puppy.github.com:wjp2013/wjp2013.github.io.git`。

进入项目下，重新设置身份。

```
git config user.email "xxx@gmail.com"
git config user.name "Jack"
```

## 参考

* [同一台电脑配置多个 git 账号](https://github.com/jawil/notes/issues/2)
* [SSH Keys: Fixing the warning ‘UNPROTECTED PRIVATE KEY FILE!’](https://medium.com/@maximbilan/ssh-keys-fixing-the-warning-unprotected-private-key-file-17fabdca7d3b)
