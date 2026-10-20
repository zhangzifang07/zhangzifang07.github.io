# Hexo 个人博客

基于 [Hexo](https://hexo.io/) 8.1.2 搭建的静态博客站点，使用 Fluid 主题。

## 项目结构

```
├── .github/
│   └── dependabot.yml        # GitHub Dependabot 自动依赖更新配置
├── scaffolds/                # 文章模板
│   ├── draft.md              #   草稿模板
│   ├── page.md               #   页面模板
│   └── post.md               #   文章模板
├── source/                   # 博客内容源文件
│   ├── _posts/               #   文章目录（Markdown 格式）
│   └── css/custom.css        #   自定义阅读样式
├── themes/                   # 主题目录
├── _config.yml               # Hexo 主配置文件
├── _config.fluid.yml         # Fluid 主题配置
└── package.json              # 项目依赖与脚本
```

## 快速开始

### 环境要求

- [Node.js](https://nodejs.org/) 22（与当前 GitHub Actions 配置保持一致）
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

`npx` 会使用项目已安装的 Hexo，无需全局安装 `hexo-cli`。下文中的 `hexo` 命令同样可以加上 `npx` 前缀。

### 阅读与外观调整

- `_config.fluid.yml`：首页头图高度为 50%，文章页为 40%，分类、标签与归档页为 30%；标题直接显示，隐藏预计阅读时长。
- `source/css/custom.css`：正文行距、窄屏字号、导航背景、面板阴影和更新时间样式。颜色使用主题变量，兼容明暗模式。
- 桌面文章正文的左右内边距为 6%，文章容器最大宽度为 960px；右侧目录标题自然换行，子目录展开后不再被两行截断，长目录可滚动浏览。
- 各页面头图使用 `source/img/morning-lake.webp`（晨光湖山摄影风），蒙版不透明度为 0.2，背景位置为 `center 50%`，展示山峦与湖面的层次并保留原有标语与文章标题。
- 文章目录使用 Fluid 原有的右侧目录。
- 自定义样式通过主题配置中的 `custom_css` 加载，修改配置后请重启预览服务。

修改后可先清理缓存，再预览：

```bash
npx hexo clean
npx hexo server
```

本地构建检查使用 `npx hexo generate`。确认后提交并推送到 `hexo` 分支，触发现有 GitHub Actions 发布流程。

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
