---
title: 通过 GitHub 搭建个人博客 Hexo
comments: true
toc: true
categories:
- Hexo
tags:
- Hexo
- GitHub
---

---

## README

> https://hexo.io/zh-cn/

安装之前确认好要安装的 Hexo 版本所对应的 Node 最低需求

例如 `Hexo 6.3.0` 最低支持 `node 14` 构建

<!--suppress HtmlUnknownTarget -->
<img src="/images/hexo-init-001.png" alt="image-20230817154901378" style="zoom:67%" />

> https://www.npmjs.com/package/hexo/v/6.3.0

---

## 安装 Hexo

```bash
npm install -g hexo-cli
```

---

## 建站

```bash
hexo init <folder>
cd <folder>
npm install
```

---

## 本地运行

```bash
hexo clean
hexo g
hexo s
```

访问 localhost:4000

---

## 部署

部署到 GitHub Pages

修改 `_config.yml` 替换添加如下

```
deploy:
  type: git
  repo: git@github.com:$username/$username.github.io.git
  branch: master
```

```bash
npm install hexo-deployer-git --save
```

最后执行三连。

```bash
hexo clean
hexo g
hexo d
```

访问 `$username.github.io`

仓库名称需要设置成 `$username.github.io` 的格式