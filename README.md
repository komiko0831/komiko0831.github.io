# Komiko 的博客

个人博客，基于 [Hugo](https://gohugo.io/) + [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack) + GitHub Pages。

- 站点地址：https://komiko0831.github.io/
- 源码仓库：https://github.com/komiko0831/komiko0831.github.io
- 部署方式：push 到 `main` 后由 GitHub Actions 自动构建并发布（工作流见 `.github/workflows/deploy.yml`）

## 发布新文章

```bash
hugo new content/post/文章名/index.md   # 创建文章（Markdown）
hugo server                            # 本地预览 http://localhost:1313
git add . && git commit -m "..." && git push
```

## 本地开发

```bash
brew install hugo            # 需要 Extended 版
hugo mod tidy                # 拉取主题模块
hugo server
```

## 目录结构

```
├── config/_default/   # 站点配置（主题参数、菜单等）
├── content/
│   ├── _index.md      # 首页
│   ├── page/          # 独立页面（归档/链接/搜索）
│   └── post/          # 博客文章（每篇一个目录，index.md）
├── assets/            # 自定义资源（头像、favicon、scss）
└── .github/workflows/ # 自动部署工作流
```
