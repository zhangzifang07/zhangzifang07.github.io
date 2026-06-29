# Hexo 个人博客

基于 [Hexo](https://hexo.io/) 8.1.2 搭建的静态博客站点，使用默认 Landscape 主题。

## 项目结构

```
├── .github/
│   └── dependabot.yml        # GitHub Dependabot 自动依赖更新配置
├── scaffolds/                # 文章模板
│   ├── draft.md              #   草稿模板
│   ├── page.md               #   页面模板
│   └── post.md               #   文章模板
├── source/                   # 博客内容源文件
│   └── _posts/               #   文章目录（Markdown 格式）
├── themes/                   # 主题目录
├── _config.yml               # Hexo 主配置文件
├── _config.landscape.yml     # Landscape 主题配置
└── package.json              # 项目依赖与脚本
```

## 快速开始

### 环境要求

- [Node.js](https://nodejs.org/) 18.0+ （推荐 LTS 版本）
- [Git](https://git-scm.com/)

### 安装依赖

```bash
npm install
```

### 本地预览

```bash
npx hexo server
# 或简写
npx hexo s
```

启动后访问 [http://localhost:4000](http://localhost:4000) 预览博客。

## 常用命令

| 命令 | 缩写 | 说明 |
|------|------|------|
| `hexo new <标题>` | — | 创建新文章 |
| `hexo new draft <标题>` | — | 创建草稿 |
| `hexo new page <标题>` | — | 创建新页面 |
| `hexo publish <标题>` | — | 将草稿转为正式文章 |
| `hexo server` | `hexo s` | 启动本地预览服务器 |
| `hexo generate` | `hexo g` | 生成静态文件到 public/ |
| `hexo deploy` | `hexo d` | 部署到远程仓库 |
| `hexo clean` | — | 清理缓存和生成的文件 |
| `hexo list <type>` | — | 列出内容（post/page/draft/tag/category） |
| `hexo version` | `hexo v` | 查看版本信息 |

### 常用组合

```bash
# 生成并部署
hexo g -d

# 本地预览（包含草稿）
hexo s --draft

# 清理缓存后重新生成（排查渲染问题）
hexo clean && hexo g
```

## 写作流程

### 发布文章

```bash
# 1. 创建文章
hexo new "我的文章"

# 2. 编辑 Markdown 文件
#    文件位于 source/_posts/我的文章.md

# 3. 本地预览
hexo s

# 4. 生成并部署
hexo g -d
```

### 草稿流程

```bash
# 1. 创建草稿（不会出现在博客中）
hexo new draft "想法"

# 2. 预览草稿
hexo s --draft

# 3. 草稿转正式文章
hexo publish "想法"

# 4. 部署
hexo g -d
```

## 部署到 GitHub Pages

### 1. 安装 Git 部署插件

```bash
npm install hexo-deployer-git --save
```

### 2. 配置 _config.yml

```yaml
deploy:
  type: git
  repo: https://github.com/你的用户名/你的用户名.github.io.git
  branch: main
```

### 3. 部署

```bash
hexo g -d
```

## 自定义配置

编辑 `_config.yml` 修改站点信息：

```yaml
# Site
title: 你的博客标题
subtitle: '你的博客副标题'
description: '你的博客描述'
keywords:
author: 你的名字
language: zh-CN        # 中文
timezone: 'Asia/Shanghai'  # 中国时区
```

## 更换主题

1. 在 [Hexo 主题商城](https://hexo.io/themes/) 选择主题
2. 安装主题，例如安装 Next 主题：

```bash
npm install hexo-theme-next --save
```

3. 修改 `_config.yml` 中的 theme 配置：

```yaml
theme: next
```

4. 重新生成预览：

```bash
hexo clean && hexo s
```

## 参考文档

- [Hexo 官方文档](https://hexo.io/docs/)
- [Hexo 主题商城](https://hexo.io/themes/)
- [Hexo 插件商城](https://hexo.io/plugins/)
- [Markdown 语法指南](https://markdown.com.cn/)
