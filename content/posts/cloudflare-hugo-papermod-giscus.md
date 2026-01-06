---
title: "Cloudflare Pages + Hugo + PaperMod + Giscus：一次完整的踩坑与收尾记录"
date: 2026-01-06
lastmod: 2026-01-06
draft: false
tags: ["Hugo", "Cloudflare Pages", "PaperMod", "Giscus"]
categories: ["博客"]
summary: "从 Hugo 版本冲突、主题不兼容、搜索 404，到 PaperMod 评论系统集成与 Git 仓库清洁，一次完整、真实的折腾记录。"
comments: true
---

## 前言

我原本只是想用 **Cloudflare Pages + Hugo** 搭一个简单、可长期维护的博客，结果一路从版本、主题、搜索、评论、Git 管理一路踩坑。

这篇文章是一次完整复盘，既给未来的自己，也给后来者。

---

## 一、Cloudflare Pages 的 Hugo 版本坑

### 1. Hugo 版本 404 的原因

Cloudflare Pages 使用的是 **asdf-hugo**，并不是 Hugo 官方发布的每一个版本都能直接下载。

表现为：

- `extended_latest` 404
- 指定版本如 `0.145.x` 下载失败

而 PaperMod 主题 **最低要求 Hugo ≥ 0.146.0**。

### 解决方案

在 Cloudflare Pages 构建环境中指定：

```
HUGO_VERSION=0.154.2
```

并在本地也使用同版本，保证一致性。

---

## 二、主题选择：Ananke → PaperMod

### Ananke 遇到的问题

- 与新 Hugo 版本不兼容
- 缺失模板文件
- 构建直接失败

### 选择 PaperMod 的原因

- 社区活跃、维护频繁
- 搜索 / SEO / 阅读体验成熟
- 支持 JSON 搜索输出
- 可高度定制

使用 submodule 安装：

```
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

---

## 三、搜索按钮 404 的真正原因

现象：

- `/search/` 页面存在
- `index.json` 可以访问
- 点击搜索按钮却 404

原因是 **home 输出未包含 JSON**。

必须配置：

```
[outputs]
  home = ["HTML", "RSS", "JSON"]
```

---

## 四、PaperMod 评论系统的隐藏设计

PaperMod 默认不会集成任何第三方评论系统。

主题中的：

```
layouts/partials/comments.html
```

默认是空实现，不会渲染任何内容。

---

## 五、正确集成 Giscus（重点）

### 错误做法

直接修改：

```
themes/PaperMod/layouts/partials/comments.html
```

问题：

- submodule 文件
- 升级主题会被覆盖
- Git 状态混乱

### 正确做法（官方推荐）

在项目中覆盖 partial：

```
layouts/partials/comments.html
```

内容如下：

```html
{{- if and .Site.Params.comments (not .Params.disableComments) -}}
<div class="comments">
<script src="https://giscus.app/client.js"
        data-repo="hunya-ops/bb.9ba.in"
        data-repo-id="R_kgDOQ0mg-A"
        data-category="General"
        data-category-id="DIC_kwDOQ0mg-M4C0o5b"
        data-mapping="pathname"
        data-reactions-enabled="1"
        data-emit-metadata="1"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
</div>
{{- end -}}
```

Hugo 会优先使用项目中的 partial，而不是主题里的。

---

## 六、Git 仓库清洁

### public 目录

不提交 `public/`，它是构建产物。

`.gitignore` 示例：

```
/public/
/resources/
.DS_Store
```

### 清理已提交的 .DS_Store

```
find . -name ".DS_Store" -print0 | xargs -0 git rm --cached --ignore-unmatch
git commit -m "chore: remove DS_Store"
git push
```

---

## 七、最终状态

- Cloudflare Pages 自动构建
- Hugo / 主题版本可控
- 搜索正常
- 评论系统稳定
- Git 仓库干净

现在，可以安心写博客了。
