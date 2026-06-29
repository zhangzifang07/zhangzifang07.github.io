---
title: GitHub的上传与下载
excerpt: 使用Trae和命令行进行GitHub代码上传与下载的操作方法
date: 2026-06-29 16:00:00
permalink: /2026/06/29/github-upload-download/
tags:
  - GitHub
categories:
  - 技术学习
---

# 1、使用Trae上传代码文件
- 使用Trae软件源代码管理，登录github账号，上传到GitHub，选择要同步的代码，上传即可
# 2、使用git命令进行文件管理
## 2.1、获取仓库地址
- 进入GitHub，选择对应的仓库
- 点击code，复制仓库地址(ssh)
```bash
git@github.com:zhangzifang07/secondev-bayu-archive.git
```
## 2.2、配置SSH（配置一次即可）
```bash
#生成公钥
ssh-keygen -t ed25519 -C "你的邮箱"

#将公钥内容(id_ed25519.pub)配置到GitHub中

#GitHub->setting-> SSH and GPG keys-> New SSH key

#测试连接
ssh -T git@github.com
```
## 2.3、使用SSH克隆仓库到本地
```bash
git clone git@github.com:用户名/仓库名.git
cd 仓库名
```
## 2.4、删除远程文件（本地的文件还是保留）
```bash
#永久删除（本地文件也会删除）
git rm -r 文件夹名
#暂时删除（只从git里删，不动你电脑上的文件）
git rm -r --cached 文件夹名
git commit -m "删除远程文件"
git push
```
# 3、使用git命令进行文件的上传与同步
## 3.1、提前在GitHub上新建一个空的仓库
## 3.2、本地初始化 Git 仓库并添加文件
```bash
# 1. 进入文件所在文件夹 
cd 本地文件夹路径 # 替换为你的实际路径 
# 2. 初始化 Git 仓库（生成 .git 目录，标记为 Git 仓库） 
git init 
# 3. 将所有文件添加到暂存区（. 表示当前目录所有文件） 
git add . 
# 若只想上传指定文件，比如只传 test.txt：
git add test.txt 
# 4. 提交暂存区的文件（-m 后是提交说明，必填） 
git commit -m "首次提交：上传项目文件"
```
## 3.3、关联GitHub空仓库并推送
```bash
#复制ssh地址

# 1. 添加远程仓库（origin 是远程仓库的别名） 
git remote add origin git@github.com:zhangzifang07/weaver.git 
# 2. 推送本地 main 分支到远程（-u 表示关联分支，后续可直接 git push） 
git push -u origin main

```
# 4、删除一个文件
```bash
#删除
git rm test

#提交
git commit -m "删除测试文件"

#推送远程
git push
```
# 5、git命令详解
```bash
#工作区 → 暂存区 → 本地仓库 → 远程仓库
#你写代码 → 准备提交(add) → 正式保存(commit) → 上传到 GitHub(push)
#初始化为git文件夹(标识为git文件夹)
git init

#将文件加入暂存区()
git add

#将文件修改至本地仓库()
git commit -m ""

#将文件推至远程仓库
git push

#查看当前连接了哪些远程仓库
git remote -v

#查看当前分支
git branch

#新建本地main分支 关联到远程仓库上
git checkout -b main origin/main
```