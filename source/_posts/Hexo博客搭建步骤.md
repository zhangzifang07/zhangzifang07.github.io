---
title: Hexo博客搭建步骤
excerpt: 记录本站从零搭建到部署的完整过程，包括环境准备、Hexo初始化、Fluid主题配置、文章编写、本地预览、GitHub Actions自动部署和换电脑恢复
date: 2026-07-16 10:00:00
permalink: /2026/07/16/hexo-blog-build-steps/
tags:
  - Hexo
  - Fluid
  - GitHub Pages
categories:
  - 博客搭建
---

这篇文章记录一下我这个博客是怎么搭建起来的。

文章尽量按照实际操作顺序写，技术人员照着改一下用户名、仓库地址、站点标题，就可以搭建一个属于自己的博客网站。

本站最终采用的方案是：

| 项目 | 方案 |
| --- | --- |
| 博客框架 | Hexo 8 |
| 博客主题 | Fluid |
| 文章格式 | Markdown |
| 源码管理 | Git |
| 源码分支 | hexo |
| 静态页面分支 | main |
| 自动部署 | GitHub Actions |
| 托管平台 | GitHub Pages |
| 访问地址 | `https://zhangzifang07.github.io` |

整体思路是：

```text
本地写 Markdown 文章
        ↓
推送源码到 GitHub 的 hexo 分支
        ↓
GitHub Actions 自动安装依赖并执行 hexo generate
        ↓
把生成后的 public/ 静态文件推送到 main 分支
        ↓
GitHub Pages 读取 main 分支并发布网站
```

也就是说，日常只需要维护 `hexo` 分支里的源码，`main` 分支里的网页文件交给自动部署流程生成。

---

## 一、准备工作

搭建前需要准备这些东西：

- Node.js
- Git
- GitHub 账号
- 一个 GitHub Pages 仓库
- 一个代码编辑器，例如 VS Code、Trae、WebStorm

### 1. 安装 Node.js

Hexo 是基于 Node.js 的静态博客框架，所以必须先安装 Node.js。

建议安装 LTS 版本。安装完成后打开命令行，执行：

```bash
node -v
npm -v
```

能正常输出版本号就说明安装成功。

例如：

```text
v22.x.x
10.x.x
```

### 2. 安装 Git

Git 用来管理博客源码，也用来把博客推送到 GitHub。

安装完成后执行：

```bash
git --version
```

能看到版本号就说明 Git 可用。

### 3. 配置 Git 用户信息

第一次使用 Git 时，建议先配置提交用户名和邮箱：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

查看配置：

```bash
git config --global --list
```

### 4. 配置 GitHub SSH Key

如果后面使用 SSH 地址推送代码，需要先配置 SSH Key。

生成密钥：

```bash
ssh-keygen -t ed25519 -C "你的邮箱"
```

一路回车即可。生成后查看公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

Windows PowerShell 可以使用：

```powershell
Get-Content ~/.ssh/id_ed25519.pub
```

复制输出内容，进入 GitHub：

```text
Settings -> SSH and GPG keys -> New SSH key
```

添加完成后测试连接：

```bash
ssh -T git@github.com
```

如果看到类似下面的提示，说明 SSH 配置成功：

```text
Hi 用户名! You've successfully authenticated
```

---

## 二、创建 GitHub Pages 仓库

GitHub Pages 个人站点仓库命名有固定规则：

```text
用户名.github.io
```

比如我的 GitHub 用户名是：

```text
zhangzifang07
```

所以仓库名就是：

```text
zhangzifang07.github.io
```

仓库创建完成后，访问地址就是：

```text
https://zhangzifang07.github.io
```

如果你搭建自己的博客，需要把文中的：

```text
zhangzifang07
```

替换成自己的 GitHub 用户名。

建议仓库先创建为空仓库，不要勾选初始化 README。后面本地初始化 Hexo 项目后再推送。

---

## 三、初始化 Hexo 项目

先安装 Hexo 命令行工具：

```bash
npm install -g hexo-cli
```

创建项目目录并初始化：

```bash
hexo init zhangzifang07
cd zhangzifang07
npm install
```

如果想让本地文件夹名称和仓库名一致，也可以这样：

```bash
hexo init zhangzifang07.github.io
cd zhangzifang07.github.io
npm install
```

初始化完成后，目录结构大致如下：

```text
├── scaffolds/          # 文章、草稿、页面模板
├── source/             # 博客内容源文件
│   └── _posts/         # 正式文章目录
├── themes/             # 主题目录
├── _config.yml         # Hexo 主配置文件
├── package.json        # 项目依赖和脚本
└── package-lock.json   # 依赖锁定文件
```

可以先启动本地服务验证项目是否正常：

```bash
npx hexo s
```

访问：

```text
http://localhost:4000
```

如果能看到默认博客页面，说明 Hexo 初始化成功。

---

## 四、安装项目依赖

本站当前使用的主要依赖如下：

| 依赖 | 作用 |
| --- | --- |
| `hexo` | Hexo 核心框架 |
| `hexo-server` | 本地预览服务 |
| `hexo-theme-fluid` | Fluid 主题 |
| `hexo-deployer-git` | 手动部署到 Git 仓库 |
| `hexo-generator-index` | 首页生成 |
| `hexo-generator-archive` | 归档页生成 |
| `hexo-generator-category` | 分类页生成 |
| `hexo-generator-tag` | 标签页生成 |
| `hexo-renderer-marked` | Markdown 渲染 |
| `hexo-renderer-ejs` | EJS 模板渲染 |
| `hexo-renderer-stylus` | Stylus 样式渲染 |

安装 Fluid 主题和部署插件：

```bash
npm install hexo-theme-fluid --save
npm install hexo-deployer-git --save
```

为了方便日常使用，可以在 `package.json` 中配置脚本：

```json
{
  "scripts": {
    "build": "hexo generate",
    "clean": "hexo clean",
    "deploy": "hexo deploy",
    "server": "hexo server"
  }
}
```

这样后面可以使用：

```bash
npm run server
npm run build
npm run deploy
npm run clean
```

---

## 五、配置 Hexo 主配置

Hexo 的主配置文件是：

```text
_config.yml
```

这里主要配置站点信息、文章规则、主题和部署方式。

本站核心配置如下：

```yaml
title: Zhang的个人博客
subtitle: ''
description: ''
keywords:
author: zhangzifang
language: zh-CN
timezone: 'Asia/Shanghai'

url: https://zhangzifang07.github.io
permalink: :year/:month/:day/:title/

post_asset_folder: true
theme: fluid

deploy:
  type: git
  repository: git@github.com:zhangzifang07/zhangzifang07.github.io.git
  branch: main
```

如果你搭建自己的博客，重点修改这些字段：

| 配置项 | 说明 |
| --- | --- |
| `title` | 博客标题 |
| `author` | 作者名 |
| `language` | 中文站点建议使用 `zh-CN` |
| `timezone` | 国内一般使用 `Asia/Shanghai` |
| `url` | GitHub Pages 访问地址 |
| `post_asset_folder` | 是否开启文章资源文件夹 |
| `theme` | 当前使用主题 |
| `deploy.repository` | GitHub 仓库 SSH 地址 |
| `deploy.branch` | 静态页面部署分支 |

其中 `post_asset_folder: true` 很重要。开启后，新建文章时可以使用文章同名文件夹管理图片。

例如：

```text
source/_posts/E10开发环境搭建以及打包.md
source/_posts/E10开发环境搭建以及打包/image.png
```

文章中可以这样引用图片：

```markdown
{% asset_img "image.png" 图片说明 %}
```

这样比直接写相对路径更适合 Hexo 的文章资源管理。

---

## 六、安装并配置 Fluid 主题

安装主题：

```bash
npm install hexo-theme-fluid --save
```

在 `_config.yml` 中启用：

```yaml
theme: fluid
```

Fluid 主题配置文件是：

```text
_config.fluid.yml
```

如果项目根目录没有这个文件，可以从 Fluid 主题文档或主题包中复制默认配置，然后按需修改。

本站主要配置了下面这些内容：

- 浏览器图标
- 导航栏标题
- 导航菜单
- 首页标语
- 代码块复制按钮
- 代码高亮
- 深色模式
- 文章目录
- 关于页头像、昵称、简介、联系方式
- 页脚版权信息
- 自定义 FontAwesome 图标库

### 1. 导航栏配置

```yaml
navbar:
  blog_title: "Zhang的个人博客"
  menu:
    - { key: "home", link: "/", icon: "iconfont icon-home-fill" }
    - { key: "archive", link: "/archives/", icon: "iconfont icon-archive-fill", name: "文章" }
    - { key: "category", link: "/categories/", icon: "iconfont icon-category-fill" }
    - { key: "tag", link: "/tags/", icon: "iconfont icon-tags-fill" }
    - { key: "about", link: "/about/", icon: "iconfont icon-user-fill" }
```

### 2. 首页标语配置

```yaml
index:
  slogan:
    enable: true
    text: "STAY HUNGRY, STAY FOOLISH! —— 求知若饥，虚心若愚"
```

### 3. 代码块配置

```yaml
code:
  copy_btn: true
  language:
    enable: true
    default: "TEXT"
  highlight:
    enable: true
    line_number: true
    lib: "highlightjs"
```

这样文章中的代码块会显示语言、行号，并且带复制按钮。

### 4. 深色模式配置

```yaml
dark_mode:
  enable: true
  default: auto
```

开启后，页面右上角会出现明暗模式切换按钮，也可以根据系统偏好自动切换。

### 5. 关于页配置

```yaml
about:
  enable: true
  avatar: /img/avatar.png
  name: "zhangzifang"
  intro: "热爱技术，热爱生活"
  icons:
    - { class: "iconfont icon-github-fill", link: "https://github.com/zhangzifang07", tip: "GitHub" }
    - { class: "fa-solid fa-envelope", qrcode: "/img/qq-qrcode.png", tip: "QQ邮箱" }
    - { class: "iconfont icon-wechat-fill", qrcode: "/img/wechat-qrcode.png", tip: "微信" }
```

这里的二维码图片放在：

```text
source/img/qq-qrcode.png
source/img/wechat-qrcode.png
```

由于邮箱图标使用了 FontAwesome，需要在 `_config.fluid.yml` 中引入 CSS：

```yaml
custom_css:
  - https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css
```

### 6. 页脚配置

```yaml
footer:
  content: '
    <a href="https://zhangzifang07.github.io" target="_blank" rel="nofollow noopener"><span>© 2026zhangzifang</span></a>
    <i class="iconfont icon-love"></i>
    <a href="https://github.com/fluid-dev/hexo-theme-fluid" target="_blank" rel="nofollow noopener"><span>Fluid</span></a>
  '
```

---

## 七、配置文章模板

Hexo 新建文章时，会读取：

```text
scaffolds/post.md
```

默认模板比较简单，可以改成下面这样：

```markdown
---
title: {{ title }}
date: {{ date }}
permalink:
tags:
  -
categories:
  -
excerpt:
---
```

这样每次执行：

```bash
npx hexo new "文章标题"
```

都会自动生成完整的文章头部。

建议每篇文章都填写：

- `permalink`
- `tags`
- `categories`
- `excerpt`

例如：

```markdown
---
title: Hexo博客搭建步骤
date: 2026-07-16 10:00:00
permalink: /2026/07/16/hexo-blog-build-steps/
tags:
  - Hexo
  - Fluid
  - GitHub Pages
categories:
  - 博客搭建
excerpt: 记录本站从零搭建到部署的完整过程
---
```

其中 `permalink` 建议使用英文路径，避免中文路径在浏览器、搜索引擎或分享时出现编码问题。

---

## 八、创建关于页面

创建关于页面：

```bash
npx hexo new page about
```

生成文件：

```text
source/about/index.md
```

内容示例：

```markdown
---
title: 关于我
layout: about
---

## 关于我

热爱技术，热爱生活。

在这里记录我的技术学习笔记和生活感想，欢迎交流！

## 联系我

- GitHub: [https://github.com/zhangzifang07](https://github.com/zhangzifang07)
```

如果使用 Fluid 的 `layout: about`，页面上的头像、昵称、图标等主要来自 `_config.fluid.yml` 的 `about` 配置。

---

## 九、写文章和管理图片

新建文章：

```bash
npx hexo new "我的第一篇文章"
```

生成文件：

```text
source/_posts/我的第一篇文章.md
```

如果开启了：

```yaml
post_asset_folder: true
```

还会生成同名资源文件夹：

```text
source/_posts/我的第一篇文章/
```

文章图片就放在这个文件夹里：

```text
source/_posts/我的第一篇文章/image.png
```

文章中引用：

```markdown
{% asset_img "image.png" 图片说明 %}
```

不建议在这种场景下使用 Obsidian 的：

```markdown
![[image.png]]
```

也不建议直接依赖复杂相对路径。Hexo 的文章资源文件夹配合 `{% asset_img %}` 更稳定。

---

## 十、本地预览和生成静态文件

本地预览：

```bash
npx hexo s
```

访问：

```text
http://localhost:4000
```

如果使用了 `package.json` 脚本，也可以执行：

```bash
npm run server
```

生成静态文件：

```bash
npx hexo g
```

或：

```bash
npm run build
```

生成结果会放到：

```text
public/
```

如果遇到页面缓存、主题切换后不生效、生成内容异常，可以清理后重新生成：

```bash
npx hexo clean
npx hexo g
```

或：

```bash
npm run clean
npm run build
```

---

## 十一、配置 Git 忽略文件

博客源码需要提交到 GitHub，但不是所有文件都应该提交。

`.gitignore` 可以这样配置：

```gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
```

说明：

| 文件或目录 | 是否提交 | 原因 |
| --- | --- | --- |
| `node_modules/` | 不提交 | 依赖体积大，可通过 `npm install` 恢复 |
| `public/` | 不提交到源码分支 | 这是生成后的静态页面 |
| `.deploy_git/` | 不提交 | Hexo 部署缓存目录 |
| `db.json` | 不提交 | Hexo 缓存文件 |
| `.log` | 不提交 | 日志文件 |

源码分支只需要保留文章、配置、模板、图片和依赖声明文件。

---

## 十二、建立源码分支 hexo

本站使用双分支结构：

```text
GitHub 仓库：zhangzifang07.github.io

main 分支
  用途：存放生成后的 HTML、CSS、JS 静态文件
  来源：GitHub Actions 自动生成并强制推送

hexo 分支
  用途：存放博客源码
  内容：Markdown 文章、配置文件、主题配置、图片、package.json
  日常：所有写作和配置修改都在这个分支进行
```

初始化 Git，并创建 `hexo` 分支：

```bash
git init
git checkout -b hexo
```

关联远程仓库：

```bash
git remote add origin git@github.com:zhangzifang07/zhangzifang07.github.io.git
```

提交源码：

```bash
git add -A
git commit -m "初始化 Hexo 博客源码"
git push -u origin hexo
```

以后日常写文章，只需要在 `hexo` 分支提交和推送源码。

---

## 十三、配置 GitHub Actions 自动部署

为了避免每次手动执行 `hexo g -d`，可以使用 GitHub Actions 自动部署。

在项目中创建文件：

```text
.github/workflows/deploy.yml
```

内容如下：

```yaml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - hexo

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4
        with:
          ref: hexo

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm install

      - name: Generate static files
        run: npx hexo generate

      - name: Verify build output
        run: |
          if [ ! -d "public" ] || [ -z "$(ls -A public)" ]; then
            echo "Error: public/ directory is empty, hexo generate may have failed"
            exit 1
          fi
          echo "Build output verified: $(ls public | wc -l) files"

      - name: Deploy to main branch
        run: |
          cd public
          git init
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "site: ${{ github.event.head_commit.message }}" || echo "Nothing to commit"
          git branch -M main
          git push -f "https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git" main
```

这个流程做了几件事：

| 步骤 | 作用 |
| --- | --- |
| 监听 `hexo` 分支 push | 只有源码分支更新时才部署 |
| 拉取源码 | 获取 Markdown、配置和主题配置 |
| 安装 Node.js | 准备构建环境 |
| 执行 `npm install` | 安装 Hexo 和主题依赖 |
| 执行 `npx hexo generate` | 生成 `public/` 静态页面 |
| 检查 `public/` | 防止空目录被部署 |
| 推送到 `main` 分支 | 发布 GitHub Pages 网站 |

注意这里使用的是 GitHub 自动提供的：

```text
secrets.GITHUB_TOKEN
```

不需要自己额外配置 Token。

---

## 十四、配置 GitHub Pages 发布分支

第一次自动部署成功后，GitHub 仓库会出现 `main` 分支。

进入 GitHub 仓库设置：

```text
Settings -> Pages
```

配置：

```text
Source: Deploy from a branch
Branch: main
Folder: /root
```

保存后等待一会儿，GitHub Pages 会发布网站。

访问：

```text
https://你的用户名.github.io
```

如果能看到博客首页，说明部署成功。

---

## 十五、手动部署方式

如果暂时不配置 GitHub Actions，也可以使用 Hexo 的手动部署方式。

先安装部署插件：

```bash
npm install hexo-deployer-git --save
```

在 `_config.yml` 中配置：

```yaml
deploy:
  type: git
  repository: git@github.com:zhangzifang07/zhangzifang07.github.io.git
  branch: main
```

手动生成并部署：

```bash
npx hexo g -d
```

或者：

```bash
npx hexo generate
npx hexo deploy
```

手动部署时要理解一点：

```text
hexo g -d
```

只负责把 `public/` 生成结果推送到 `main` 分支，不会帮你备份源码。

所以如果采用手动部署，仍然要单独执行：

```bash
git add -A
git commit -m "更新博客源码"
git push
```

把源码推送到 `hexo` 分支。

---

## 十六、日常写作和发布流程

配置好 GitHub Actions 后，日常流程非常简单。

### 1. 新建文章

```bash
npx hexo new "文章标题"
```

### 2. 编辑文章

文章位置：

```text
source/_posts/文章标题.md
```

图片位置：

```text
source/_posts/文章标题/
```

图片引用：

```markdown
{% asset_img "图片名.png" 图片说明 %}
```

### 3. 本地预览

```bash
npx hexo s
```

浏览器访问：

```text
http://localhost:4000
```

### 4. 推送源码并自动部署

```bash
git add -A
git commit -m "新增文章：文章标题"
git push
```

推送后，GitHub Actions 会自动构建并部署到 `main` 分支。

一般等待 1 到 2 分钟，再访问博客地址即可看到最新内容。

---

## 十七、换电脑后如何恢复博客

如果换了一台电脑，不要去克隆 `main` 分支，因为 `main` 分支只有生成后的网页文件，没有源码。

正确做法是克隆 `hexo` 分支：

```bash
git clone -b hexo git@github.com:zhangzifang07/zhangzifang07.github.io.git
```

进入项目：

```bash
cd zhangzifang07.github.io
```

安装依赖：

```bash
npm install
```

本地预览：

```bash
npx hexo s
```

确认没问题后，就可以继续写文章：

```bash
npx hexo new "新文章"
git add -A
git commit -m "新增文章：新文章"
git push
```

如果新电脑还没有配置 GitHub SSH Key，需要先完成前面提到的 SSH Key 配置。

---

## 十八、项目结构说明

本站当前结构大致如下：

```text
F:\zhangzifang07\
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions 自动部署
├── scaffolds/                  # 文章模板
│   ├── draft.md                # 草稿模板
│   ├── page.md                 # 页面模板
│   └── post.md                 # 文章模板
├── source/                     # 博客内容源文件
│   ├── _posts/                 # 文章目录
│   ├── about/                  # 关于页面
│   └── img/                    # 全局图片资源
├── themes/                     # 主题目录，本项目主要使用 npm 安装的 Fluid
├── _config.yml                 # Hexo 主配置文件
├── _config.fluid.yml           # Fluid 主题配置文件
├── package.json                # 项目依赖与脚本
├── package-lock.json           # 依赖版本锁定
└── README.md                   # 项目说明
```

几个重点目录：

| 路径 | 说明 |
| --- | --- |
| `source/_posts/` | 所有正式文章 |
| `source/about/` | 关于页面 |
| `source/img/` | 全局图片，例如头像、二维码、站点图片 |
| `scaffolds/post.md` | 新文章模板 |
| `_config.yml` | Hexo 主配置 |
| `_config.fluid.yml` | Fluid 主题配置 |
| `.github/workflows/deploy.yml` | 自动部署配置 |

---

## 十九、常用命令速查

| 命令 | 作用 |
| --- | --- |
| `npx hexo new "标题"` | 新建文章 |
| `npx hexo new page about` | 新建页面 |
| `npx hexo s` | 本地预览 |
| `npx hexo g` | 生成静态页面 |
| `npx hexo d` | 手动部署 |
| `npx hexo g -d` | 生成并手动部署 |
| `npx hexo clean` | 清理缓存和生成文件 |
| `npm install` | 安装项目依赖 |
| `npm run server` | 使用脚本启动本地服务 |
| `npm run build` | 使用脚本生成静态页面 |
| `git status` | 查看 Git 状态 |
| `git add -A` | 暂存所有修改 |
| `git commit -m "说明"` | 提交源码 |
| `git push` | 推送源码并触发自动部署 |

---

## 二十、常见问题

### 1. 页面没有更新

先确认本地是否生成成功：

```bash
npx hexo clean
npx hexo g
```

如果本地正常，再确认是否已经推送：

```bash
git status
git push
```

然后到 GitHub 仓库的 Actions 页面查看部署任务是否成功。

### 2. GitHub Pages 打开是 404

检查几个地方：

- 仓库名是否是 `用户名.github.io`
- GitHub Pages 是否设置为 `main` 分支 `/root`
- Actions 是否成功推送了 `main` 分支
- `_config.yml` 中的 `url` 是否正确

### 3. 图片不显示

如果是文章同名文件夹里的图片，确认：

```yaml
post_asset_folder: true
```

并使用：

```markdown
{% asset_img "图片名.png" 图片说明 %}
```

同时检查图片是否真的放在文章同名目录下。

### 4. 中文链接访问异常

建议每篇文章都配置英文 `permalink`：

```yaml
permalink: /2026/07/16/hexo-blog-build-steps/
```

不要依赖中文文件名直接生成访问链接。

### 5. 修改主题配置不生效

修改 `_config.fluid.yml` 后，建议重启本地服务：

```bash
npx hexo clean
npx hexo s
```

### 6. 不小心提交了 public 或 node_modules

先确认 `.gitignore` 是否包含：

```gitignore
node_modules/
public/
.deploy*/
db.json
```

如果已经被 Git 跟踪，需要从暂存跟踪中移除，但保留本地文件：

```bash
git rm --cached -r node_modules public .deploy_git db.json
git add .gitignore
git commit -m "修正忽略文件"
```

---

## 二十一、完整流程总结

从零搭建一个同类型博客，可以按这个顺序执行：

```text
1. 安装 Node.js
2. 安装 Git
3. 配置 GitHub SSH Key
4. 创建 用户名.github.io 仓库
5. 安装 hexo-cli
6. 使用 hexo init 初始化项目
7. 安装项目依赖
8. 安装 Fluid 主题
9. 修改 _config.yml
10. 修改 _config.fluid.yml
11. 配置 scaffolds/post.md 文章模板
12. 创建 about 页面
13. 配置 .gitignore
14. 创建 hexo 源码分支并推送
15. 配置 GitHub Actions
16. 设置 GitHub Pages 使用 main 分支
17. 写文章并本地预览
18. git push 后自动发布
```

搭建完成后，日常只需要记住这个流程：

```text
写文章
  ↓
本地预览
  ↓
提交到 hexo 分支
  ↓
GitHub Actions 自动部署 main 分支
  ↓
GitHub Pages 发布网站
```

这样博客源码和线上网页是分开的：`hexo` 分支负责维护内容和配置，`main` 分支负责承载生成后的静态网站。后续迁移电脑、回滚文章、修改主题都会比较清晰。
