# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概述

`lovely.vision` 是一个 Jekyll 静态博客，使用 `minima` 主题和 `jekyll-feed` 插件，通过 GitHub Pages 部署（`CNAME` 文件指向 `lovely.vision`）。站点内容为中文，主要是菜谱类博客。

## 常用命令

Ruby 依赖通过 Bundler 管理：

```bash
# 安装依赖（首次或 Gemfile 变更后）
bundle install

# 本地预览（默认 http://127.0.0.1:4000，修改文件自动重载，_config.yml 除外）
bundle exec jekyll serve

# 包含未发布的草稿
bundle exec jekyll serve --drafts

# 构建生产产物（输出到 _site/）
bundle exec jekyll build

# 清理生成产物
bundle exec jekyll clean
```

没有构建脚本、测试框架或 linter。修改 `_config.yml` 后必须重启 `jekyll serve`。

### 在 macOS 上用 Homebrew Ruby 时的注意点

`jekyll 4.4.1` 的运行时依赖链 `em-websocket -> eventmachine 1.2.7`。该 gem 自 2018 年起未再发布（rubygems 最新仍是 1.2.7），原生扩展在新版 Homebrew Ruby + CommandLineTools SDK 下找不到 C++ 标准头，无法编译。

解决：仅 macOS 用户需要做，**Windows / Linux 用户忽略**。

在仓库根创建 `.bundle/config`，内容如下：

```
---
BUNDLE_BUILD__EVENTMACHINE: "--with-cppflags='-stdlib=libc++ -I/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1' --with-ldflags='-stdlib=libc++'"
```

然后 `bundle install`。

> **不要用** `bundle config set --local build.eventachine "..."`：Bundler 4.x 会把 key 错写成 `BUNDLE_BUILD__EVENTACHINE`（少一个 M），导致 flag 失效。直接写文件最稳。

`.bundle/` 已加入 `.gitignore`，不会被提交。

## 代码架构

标准 Jekyll 4.x 站点，关键约定：

- **`_config.yml`** — `title: Lovely Vision`、`baseurl: ""`、`theme: minima`、启用 `jekyll-feed`、中文 `description`。修改后需重启服务。
- **`_layouts/recipe.html`** — 自定义布局，链入 `post` 布局并设置 `categories: recipe`。渲染时依赖页面的 `ingredients`、`directions`、`tips` 三个 front-matter 数组（`page.ingredients`、`page.directions`、`page.tips`），并在结尾插入一个带 SVG 图标的 "Tips" 卡片。整个布局自带 CSS（含 Raleway / Playfair Display 字体、棕色调色板 `#916953` / `#d69762`）。
- **`_layouts/`** — 只有 `recipe.html` 一份自定义布局；其他布局由 `minima` 主题提供。
- **`_posts/`** — 博客内容，必须使用 `YYYY-MM-DD-title.markdown` 命名规范。普通文章使用 `post` 布局，菜谱类必须使用 `recipe` 布局。
- **`index.markdown`** — 首页，`layout: home`。
- **`about.markdown`** — 关于页，`layout: page`，`permalink: /about/`。
- **`404.html`** — `layout: page`，`permalink: /404.html`。
- **`CNAME`** — GitHub Pages 自定义域名为 `lovely.vision`，不要删除或改动。
- **`.gitignore`** 忽略 `_site/`、`.jekyll-cache/`、`.jekyll-metadata/`、`vendor/`、`/.DS_Store`。

### 菜谱文章 front-matter 结构

每个菜谱 `.markdown` 文件以 YAML front-matter 开头，包含三个数组字段：

```yaml
---
layout: recipe
title:  菜名            # 中文标题，前置一个全角空格（保持现有风格）
date:   YYYY-MM-DD HH:MM:SS +0800
ingredients:
  - 食材名 <em>份量</em>   # 数量用 <em> 包裹以在 recipe.html 中通过浏览器默认样式加粗/斜体
directions:
  - 步骤一
  - 步骤二
tips:
  - 要点小贴士
---
```

正文中只需写前言或留空，主体内容由 `recipe.html` 自动渲染三个列表。修改现有菜谱时保持 front-matter 字段顺序与缩进风格一致。

### Disqus 评论

`_config.yml` 中配置了 `disqus.shortname: lovely-vision`。`minima` 主题默认在 `post` 布局下启用评论，新菜谱页面会自动启用，无需额外设置。

## 部署

仓库直接服务于 GitHub Pages，自定义域名为 `lovely.vision`。`main` / `master` 分支的新推送会自动构建并发布。`_site/` 是构建产物，不要手动编辑。
