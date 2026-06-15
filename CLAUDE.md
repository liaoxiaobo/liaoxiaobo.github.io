# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 [Hexo](https://hexo.io/) 7.3.0 和 [NexT](https://theme-next.js.org/) 主题（Gemini 方案）的个人博客。内容使用 Markdown 编写，并带有 YAML frontmatter，最终生成静态站点并部署到 GitHub Pages。

- 源码/工作分支：`blog`
- 部署目标：通过 `hexo-deployer-git` 推送到 `git@github.com:liaoxiaobo/liaoxiaobo.github.io.git` 的 `master` 分支
- 站点语言：`zh-CN`
- 生成输出目录：`public/`（已加入 `.gitignore`）
- 部署临时目录：`.deploy_git/`（已加入 `.gitignore`）

## 常用命令

命令通过 `package.json` 中的 scripts 和 Hexo CLI 管理。

```bash
# 安装依赖
npm install

# 启动本地开发服务器（默认 http://localhost:4000）
npm run server

# 生成静态站点
npm run build

# 清理生成的文件和缓存
npm run clean

# 生成并部署到 GitHub Pages
npm run deploy
```

也可以直接使用 Hexo CLI，例如 `npx hexo server --draft` 用于预览草稿。

## 内容结构

- `source/_posts/` —— Markdown 格式的博客文章。每篇文章使用 YAML frontmatter（`title`、`date`、`tags`、`categories`）。
- `source/about/index.md` —— 关于页面。
- `source/tags/index.md` 和 `source/categories/index.md` —— 标签和分类归档页面。
- `scaffolds/` —— `hexo new` 命令使用的模板文件（`post.md`、`draft.md`、`page.md`）。

`_config.yml` 中启用了 `post_asset_folder: true`，因此包含图片的文章应将资源放在与文章同名的文件夹中（例如 `source/_posts/my-post/image.png`），并使用相对路径引用。

## 配置说明

- `_config.yml` —— 站点级 Hexo 配置（主题、部署、永久链接、搜索等）。
- `themes/next/_config.yml` —— NexT 主题配置（方案、菜单、侧边栏、第三方服务）。主题默认使用 Gemini 方案，当前未使用备用主题配置文件（`source/_data/` 为空）。

## 架构说明

- 仓库采用源分支工作流：在 `blog` 分支上编辑并提交，然后运行 `npm run deploy`，由 Hexo 将生成的 `public/` 内容推送到 `master` 分支。
- 主题以普通目录形式放在 `themes/next/` 下，不是 Git 子模块。
- 除 `.github/dependabot.yml` 中定义的 Dependabot 更新外，没有测试、lint 配置或 CI 工作流。
