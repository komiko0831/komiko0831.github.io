---
title: 你好，Komiko！
description: 博客上线测试文章，验证 Hugo + GitHub Pages 部署链路是否正常。
slug: hello-komiko
date: 2026-08-06 00:00:00+0000
categories:
    - 随笔
tags:
    - 测试
    - 博客
---

欢迎来到 **Komiko 的博客**！这是第一篇测试文章，用来验证整个部署链路是否正常：

Hugo（本地写作）→ git push → GitHub Actions 自动构建 → GitHub Pages 发布

## 这个博客是怎么搭起来的？

- 静态站点生成器：[Hugo](https://gohugo.io/)（Extended 版）
- 主题：[hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack) v4
- 托管：[GitHub Pages](https://docs.github.com/en/pages)，由 GitHub Actions 自动部署

## 发布一篇新文章

只需要三步：

```bash
hugo new content/post/我的新文章/index.md
# 用 Markdown 写好内容
git add .
git commit -m "发布新文章"
git push
```

推送后等一两分钟，站点会自动更新，不需要手动构建。

## 顺便测试一下 Markdown 渲染

> 引用块：保持热爱，奔赴山海。

- 无序列表项
- 还有**加粗**、`行内代码`、[链接](https://github.com/komiko0831)……

如果你能看到这篇文章，说明一切正常！以后我会在这里记录技术笔记和生活随想。

—— Komiko
