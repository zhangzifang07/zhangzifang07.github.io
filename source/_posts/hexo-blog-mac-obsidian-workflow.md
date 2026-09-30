---
title: 把 Hexo 博客迁到 Mac，并用 Obsidian 打造一站式写作流
date: 2026-09-30 15:10:00
permalink: /2026/09/30/hexo-blog-mac-obsidian-workflow/
tags:
  - Hexo
  - Obsidian
  - macOS
categories:
  - 博客搭建
excerpt: 一次完整的博客迁移实战：从 Windows 迁到 Mac，再用软链接 + Templater + Shell Commands 把 Obsidian 变成写作、发布一条龙的博客工作台
---

## 背景

我的博客是 Hexo + Fluid 主题 + GitHub Pages 的经典组合，之前一直在 Windows 机器上维护。最近换了 Mac，于是花了一个下午完成了两件事：

1. 把博客从 Windows 迁移到 Mac
2. 把 Obsidian 和博客打通，实现「写作 → 发布」全程不离开 Obsidian

顺便把整个过程记录下来，给有同样需求的朋友一个参考。

## 一、先搞清楚仓库结构

我的博客仓库 `zhangzifang07.github.io` 采用**双分支设计**：

| 分支 | 内容 | 谁在提交 |
|------|------|----------|
| `hexo` | Hexo 源码（配置、文章、主题） | 我自己 |
| `main` | 构建后的静态页面 | GitHub Actions 自动部署 |

也就是说：**日常只需要维护 `hexo` 分支**，push 之后 CI 会自动构建发布到 Pages，完全不需要在本地装部署工具。想明白这一点，迁移就非常简单了——源码都在远程仓库里，新机器拉下来就能用。

## 二、迁移步骤

### 1. 配置 GitHub SSH Key

新电脑想 push 代码，先配钥匙：

```bash
# 生成密钥
ssh-keygen -t ed25519 -C "你的邮箱"

# 查看公钥，复制后去 GitHub → Settings → SSH keys 添加
cat ~/.ssh/id_ed25519.pub

# 验证
ssh -T git@github.com
# 出现 "Hi xxx! You've successfully authenticated" 即成功
```

### 2. 拉取源码

正常情况 `git clone -b hexo <仓库地址>` 即可。但如果目标目录里已经有文件（比如放了别的东西），clone 会报错，可以用等价的手动方式：

```bash
cd ~/code/blog_jxy
git init
git remote add origin git@github.com:xxx/xxx.github.io.git
git fetch origin
git checkout hexo
```

这四条命令其实就是 `git clone` 内部干的事：建仓库 → 关联远程 → 下载代码 → 检出分支。

### 3. 安装依赖

```bash
npm install
```

因为仓库里有 `package-lock.json`，npm 会按锁文件**精确还原**所有依赖版本，新机器装出来的环境和原来完全一致，不存在「我这能跑你那不能跑」的问题。

### 4. 验证

```bash
npx hexo s
```

浏览器打开 `http://localhost:4000`，看到自己的博客就说明迁移成功。

**注意**：不需要全局安装 Hexo，它就是项目 `node_modules` 里的一个本地包，用 `npx` 调用即可。

## 三、打通 Obsidian：软链接方案

我的日常笔记都在 Obsidian 里管理，如果写博客还要切换到另一个目录、另一个编辑器，体验太割裂。好在有个零成本的方案——**软链接（symbolic link）**。

### 原理

```bash
ln -s ~/code/blog_jxy/source/_posts "/path/to/MyObsidian/Blog"
```

这条命令在 Obsidian 库里创建了一个叫 `Blog` 的「传送门」，指向博客的文章目录 `source/_posts`。**文件本体只有一份**，躺在博客仓库里由 git 管理；通过 Obsidian 打开它，编辑的就是同一份文件。不存在「同步」概念，因为它压根就是同一个东西。

效果：

- Obsidian 侧边栏多出一个 `Blog` 文件夹，里面就是全部博客文章
- 在这里新建/修改笔记 = 直接修改博客仓库，写完 push 就发布
- 库里其他私人笔记永远不会被发布（Hexo 只认 `source/_posts` 里的文件）
- iCloud 不会把这个软链接文件夹同步到手机——这反而是好事，文章由 git 管理，不会在云端重复占空间

### 唯一的纪律：注意 Markdown 方言

Obsidian 的 `[[双链]]`、`![[嵌入]]`、正文 `#标签` 是私有语法，Hexo 不认识。博客文章里要用标准写法：

| Obsidian 特有写法 | 博客里应该用 |
|---|---|
| `[[另一篇笔记]]` | `[另一篇笔记](文章链接)` |
| `![[图片.png]]` | `![](图片.png)` |
| 正文里 `#标签` | 写进 frontmatter 的 `tags` |

## 四、Templater：让 frontmatter 自动生成

Hexo 文章开头需要一段 frontmatter（title/date/permalink/tags 等），每次手写太麻烦。用社区插件 **Templater** 配一个模板：

```yaml
---
title: hexo-blog-mac-obsidian-workflow
date: 2026-09-30 15:08:42
permalink: /2026/09/30/hexo-blog-mac-obsidian-workflow/
tags:
  - 
categories:
  - 
excerpt: 
---
```

### 踩过的坑：自动触发时机

一开始我配置了「新建文件时自动触发模板」，结果发现：右键新建笔记时文件还叫「未命名」，模板立刻执行，把 `title` 和 `permalink` 都固化成了「未命名」，之后再改名模板也不会重跑。

**正确姿势是手动插入模式：**

```
① 右键 Blog 文件夹 → 新建笔记
② 第一时间给文件命名（建议英文短横线风格，permalink 更干净）
③ Cmd+P → "Templater: 插入模板"
④ title、date、permalink 全部基于真实文件名自动生成
```

给「插入模板」绑个快捷键，第③步就是一眨眼的事。

### permalink 防重复规则

模板里 permalink 的最后一段取自文件名，所以有一条简单的链条：**文件名唯一 → permalink 唯一 → 线上 URL 唯一**。写文章前瞄一眼有没有重名文件即可。

## 五、Shell Commands：一键发布

最后一块拼图——在 Obsidian 里直接 push。

著名的 Obsidian Git 插件在这里其实**用不了**：它只认「库根目录」的 git 仓库，而我的结构是仓库在库外面、靠软链接伸进来，插件在 vault 根目录找不到 `.git`，会静默失灵。

替代方案是 **Shell Commands** 插件，它能在 Obsidian 里执行任意终端命令：

```bash
cd ~/code/blog_jxy && git add -A && git commit -m "site: 更新博客" && git push origin hexo
```

配置一条命令、绑一个快捷键（比如 `Cmd + Shift + G`），发布文章就是：写完 → 按快捷键 → CI 自动部署 → 1~2 分钟后线上生效。

> 如果报 `git: command not found`，是图形应用的 PATH 环境变量和终端不同，把命令里的 `git` 换成绝对路径 `/usr/bin/git` 即可。

## 最终工作流

```
📝 Obsidian 的 Blog 文件夹里写文章（模板自动带出 frontmatter）
        ↓
👀 想预览就终端跑 npx hexo s（可选）
        ↓
🚀 Cmd + Shift + G 一键 commit + push
        ↓
🤖 GitHub Actions 自动构建（1~2 分钟）
        ↓
🌐 线上博客更新
```

写作、预览、发布全程只需要 Obsidian 和一个快捷键。

## 总结

这次折腾下来几个心得：

1. **双分支 + CI 部署的博客迁移成本极低**——源码都在远程，新机器 clone + npm install 就完事
2. **软链接是连接两个世界的最优雅方案**——零工具、零同步成本、零数据冗余
3. **Obsidian Git 插件不适用于「仓库在库外」的结构**，Shell Commands 是更好的替代
4. 插件的自动化虽好，但要理解触发时机，不然会得到「未命名」的 permalink 😄

如果你也是 Obsidian + Hexo 用户，欢迎试试这套方案。
