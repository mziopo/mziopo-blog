---
title: 配置Git SSH
comments: true
categories:
- SSH
tags:
- Git
- SSH
---

## README

本篇主要记录Windows环境，生成及配置非默认、具有别名的SSH keys。

> id_ed25519.pub 这种以默认的名称命名的公钥文件，不需要额外配置即可自动识别。
>
> github_olive.pub 这种自定义key的公钥文件，需要在ssh-agent中添加才可被识别。

## Generate SSH

```shell
root@Virtual:/# cd C:\Users\olive\.ssh
root@Virtual:/# ssh-keygen -t ed25519 -C "<your github email>"
Generating public/private ed25519 key pair.
# 自定义key
Enter file in which to save the key (C:\Users\olive/.ssh/id_ed25519): github_olive
# 其他默认
...
root@Virtual:/# dir
2023/08/16  15:43               411 github_olive
2023/08/16  15:43                98 github_olive.pub
```

## Github 配置 SSH keys

Setting -> SSH and GPG keys -> New SSH key

## 本地测试是否配置成功

```shell
root@Virtual:/# ssh -T git@github.com
git@github.com: Permission denied (publickey).
```

具体原因看README。

## 配置 Config 文件

ssh-agent & ssh-add 反正我配置的这俩只在当前窗口生效，所以直接选择配置Config文件一劳永逸。
> $ eval "$(ssh-agent -s)"
> Agent pid 555
> ssh-add ~/.ssh/github_olive

```shell
root@Virtual:/# cd C:\Users\olive\.ssh
root@Virtual:/# dir
2023/08/16  15:43                 1 config
2023/08/16  15:43               411 github_olive
2023/08/16  15:43                98 github_olive.pub
root@Virtual:/# vim config
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_olive
```

再次测试是否授权。

```shell
root@Virtual:/# ssh -T git@github.com
Hi olive! You've successfully authenticated, but GitHub does not provide shell access.
```

