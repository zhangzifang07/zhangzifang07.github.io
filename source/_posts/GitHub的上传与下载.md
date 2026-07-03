---
title: GitHub的上传与下载
excerpt: 使用Trae和Git命令行进行GitHub代码上传与下载的操作方法，涵盖SSH配置、克隆、提交、推送、分支、冲突处理等完整流程
date: 2026-06-29 16:00:00
permalink: /2026/06/29/github-upload-download/
tags:
  - GitHub
categories:
  - 技术学习
---

# 1、SSH 配置（首次使用配置一次即可）

Git 与 GitHub 之间通过 SSH 协议通信，配置一次后免密操作：

```bash
# 生成密钥（ed25519 是当前推荐的算法，比 rsa 更安全更快）
ssh-keygen -t ed25519 -C "你的邮箱"
# 一路回车即可，密钥文件生成在 ~/.ssh/ 目录下

# 查看并复制公钥内容
cat ~/.ssh/id_ed25519.pub

# 将公钥添加到 GitHub
# GitHub → Settings → SSH and GPG keys → New SSH key
# Title 随便填（如 "我的电脑"），Key 粘贴公钥内容，点击 Add SSH key

# 测试连接（成功会显示 "Hi 用户名! You've successfully authenticated"）
ssh -T git@github.com
```

> **提示**：如果公司或学校网络屏蔽了 SSH 的 22 端口，连接超时，可以配置使用 443 端口：
> ```bash
> # 编辑 ~/.ssh/config 文件，添加以下内容
> Host github.com
>   Hostname ssh.github.com
>   Port 443
>   User git
> ```

# 2、克隆仓库（下载到本地）

## 2.1、克隆方式

GitHub 支持两种克隆方式，推荐使用 SSH：

```bash
# SSH 方式（推荐，配置好密钥后免密操作）
git clone git@github.com:用户名/仓库名.git

# HTTPS 方式（每次 push 需要输入用户名和 Token）
git clone https://github.com/用户名/仓库名.git
```

## 2.2、克隆到指定目录

```bash
# 克隆到当前目录下的指定文件夹
git clone git@github.com:用户名/仓库名.git 目标文件夹名

# 只克隆最近一次提交（大型仓库加速下载）
git clone --depth=1 git@github.com:用户名/仓库名.git
```

克隆完成后进入目录，后续所有操作都在此目录下进行：

```bash
cd 仓库名
```

# 3、日常上传流程（三步走）

每次修改文件后，按以下三步提交推送：

```bash
# ① 添加到暂存区
git add .                # 添加所有修改
git add test.txt         # 或只添加指定文件
git add src/             # 或添加整个目录

# ② 提交到本地仓库
git commit -m "提交说明"  # -m 后的说明必填，简要描述本次改动

# ③ 推送到远程仓库
git push
```

## 3.1、查看当前状态

在 add 之前，建议先看看改了什么：

```bash
# 查看哪些文件被修改了
git status

# 查看具体修改了什么内容（红色为删除，绿色为新增）
git diff

# 查看暂存区的修改内容（add 之后用）
git diff --staged
```

## 3.2、撤销操作

```bash
# 撤销工作区的修改（文件恢复到上次 commit 的状态，慎用！修改会丢失）
git checkout -- 文件名

# 撤销暂存区（把已 add 的文件移出暂存区，文件内容不变）
git reset HEAD 文件名

# 修改上次提交说明（还没 push 时使用）
git commit --amend -m "新的提交说明"

# 撤销最近一次提交（保留文件修改，回退到 add 之前）
git reset --soft HEAD~1

# 撤销最近一次提交（丢弃所有修改，慎用！）
git reset --hard HEAD~1
```

> **注意**：`git reset --hard` 会丢失所有未提交的修改，执行前务必确认！

## 3.3、忽略文件（.gitignore）

有些文件不需要上传到 GitHub（如编译产物、配置文件中的密码等），在项目根目录创建 `.gitignore` 文件：

```bash
# .gitignore 文件示例
node_modules/        # 依赖目录
.DS_Store            # macOS 系统文件
*.log                # 所有日志文件
.env                 # 环境变量文件（可能含密码）
dist/                # 构建输出目录
```

如果文件已经被 Git 跟踪了，需要先取消跟踪：

```bash
git rm -r --cached 文件名    # 从 Git 中移除，但保留本地文件
git commit -m "移除不需要跟踪的文件"
git push
```

# 4、新项目首次上传

如果本地已有项目文件夹，GitHub 上也新建了空仓库，需要首次关联推送：

```bash
# 1. 进入项目文件夹
cd 本地文件夹路径

# 2. 初始化 Git 仓库
git init

# 3. 添加并提交文件
git add .
git commit -m "首次提交：上传项目文件"

# 4. 关联远程仓库（origin 是远程仓库别名，可自定义但一般用 origin）
git remote add origin git@github.com:用户名/仓库名.git

# 5. 推送并关联分支（-u 只需首次使用，之后直接 git push 即可）
git push -u origin main
```

> **常见问题**：如果 GitHub 创建仓库时勾选了 README，远程已有内容，push 会被拒绝。解决方法：
> ```bash
> # 先拉取远程内容并合并，再推送
> git pull origin main --allow-unrelated-histories
> git push -u origin main
> ```

# 5、删除与重命名文件

## 5.1、删除文件

```bash
# 永久删除（本地文件也会被删除）
git rm 文件名
git rm -r 文件夹名

# 只从 Git 中删除，保留本地文件（常用于误提交的文件）
git rm --cached 文件名
git rm -r --cached 文件夹名

# 删除后同样需要提交和推送
git commit -m "删除xxx文件"
git push
```

## 5.2、重命名文件

```bash
# 用 git mv 重命名，Git 会自动识别为重命名而非删除+新增
git mv 旧文件名 新文件名
git commit -m "重命名xxx文件"
git push
```

# 6、分支管理

分支是 Git 最强大的功能之一，可以在不影响主线的情况下开发新功能：

## 6.1、基本操作

```bash
# 查看所有分支（* 标记的是当前分支）
git branch

# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 创建新分支
git branch 新分支名

# 切换分支
git checkout 分支名
# 或使用更直观的 switch 命令（Git 2.23+）
git switch 分支名

# 创建并切换到新分支（一步到位）
git checkout -b 新分支名
# 或
git switch -c 新分支名

# 删除本地分支
git branch -d 分支名

# 删除远程分支
git push origin --delete 分支名
```

## 6.2、合并分支

新功能开发完成后，需要合并回主分支：

```bash
# 1. 切换到目标分支（如 main）
git checkout main

# 2. 合并指定分支到当前分支
git merge 功能分支名

# 3. 推送到远程
git push
```

## 6.3、典型工作流程

```
main（稳定版本）
  └── feature/xxx（新功能开发）
        └── 开发完成 → 合并回 main → 删除 feature 分支
```

# 7、多端同步与冲突处理

## 7.1、日常同步

如果在多台设备上工作，每次开始前先拉取最新代码：

```bash
# 拉取远程更新（每次开始工作前建议先执行）
git pull
```

## 7.2、忘记 pull 就改了本地文件

```bash
git stash        # 暂存本地修改
git pull         # 拉取远程更新
git stash pop    # 恢复本地修改
```

## 7.3、冲突处理

当本地和远程修改了同一个文件的同一行时，`git pull` 或 `git merge` 会产生冲突：

```bash
# 冲突时 Git 会提示哪些文件有冲突
git status       # 查看冲突文件列表

# 打开冲突文件，会看到类似标记：
# <<<<<<< HEAD
# 你的修改内容
# =======
# 远程的修改内容
# >>>>>>> origin/main

# 手动编辑文件，选择保留哪部分（删除标记符号）
# 也可以两者都保留，按需组合

# 解决冲突后
git add 冲突文件名
git commit -m "解决xxx文件冲突"
git push
```

> **避免冲突的技巧**：
> - 每次开发前先 `git pull`
> - 尽量避免多人修改同一文件
> - 用分支隔离不同功能的开发

# 8、查看历史记录

```bash
# 查看提交历史（简洁模式，一行一条）
git log --oneline

# 查看最近 5 条记录
git log --oneline -5

# 查看指定文件的修改历史
git log --oneline 文件名

# 查看某次提交的详细改动
git show 提交ID

# 查看谁修改了文件的每一行
git blame 文件名
```

# 9、使用 Trae 上传代码

使用 Trae（AI 编程工具）内置的源代码管理功能：

1. 在 Trae 中打开项目文件夹
2. 点击左侧源代码管理图标（或 `Ctrl+Shift+G`）
3. 登录 GitHub 账号
4. 选择要提交的文件，填写提交说明，点击提交并推送

# 10、Git 命令速查

## 核心流程

```
工作区 → 暂存区 → 本地仓库 → 远程仓库
你改代码 → git add → git commit → git push
```

## 基础命令

| 命令 | 说明 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone 地址` | 克隆远程仓库 |
| `git status` | 查看当前修改状态 |
| `git add .` | 添加所有修改到暂存区 |
| `git add 文件名` | 添加指定文件到暂存区 |
| `git commit -m "说明"` | 提交暂存区到本地仓库 |
| `git push` | 推送到远程仓库 |
| `git pull` | 拉取远程更新并合并 |

## 分支命令

| 命令 | 说明 |
|------|------|
| `git branch` | 查看本地分支 |
| `git branch 分支名` | 创建新分支 |
| `git checkout 分支名` | 切换分支 |
| `git checkout -b 分支名` | 创建并切换分支 |
| `git merge 分支名` | 合并分支到当前分支 |
| `git branch -d 分支名` | 删除本地分支 |

## 撤销与回退

| 命令 | 说明 |
|------|------|
| `git checkout -- 文件名` | 撤销工作区修改（丢失修改） |
| `git reset HEAD 文件名` | 撤销暂存区（保留修改） |
| `git reset --soft HEAD~1` | 撤销上次提交（保留修改） |
| `git reset --hard HEAD~1` | 撤销上次提交（丢弃修改） |
| `git commit --amend -m "说明"` | 修改上次提交说明 |

## 查看与对比

| 命令 | 说明 |
|------|------|
| `git log --oneline` | 查看提交历史 |
| `git diff` | 查看工作区修改内容 |
| `git diff --staged` | 查看暂存区修改内容 |
| `git show 提交ID` | 查看某次提交详情 |
| `git blame 文件名` | 查看文件每行的修改者 |

## 远程仓库

| 命令 | 说明 |
|------|------|
| `git remote -v` | 查看远程仓库地址 |
| `git remote add origin 地址` | 关联远程仓库 |
| `git push -u origin main` | 首次推送并关联分支 |
| `git push origin --delete 分支名` | 删除远程分支 |
