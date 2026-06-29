---
title: ARTHAS在E10中的使用
excerpt: 使用Arthas在线诊断E10运行中的Java进程，包括反编译源码、搜索类、追踪方法调用等常用操作
date: 2026-06-29 16:00:00
permalink: /2026/06/29/arthas-in-e10/
tags:
  - E10
  - ARTHAS
  - 运维
categories:
  - 技术学习
---

## 一、查看源码，反编译类
```bash
#找路径 获取进程pid-1111176
ps -ef | grep E10

#启动 Arthas 并绑定目标进程
/data/weaver/jdk/bin/java -jar /data/weaver/e-monitor/app/arthas/arthas-boot.jar 1111176

#反编译目标类代码
jad com.weaver.intcenter.hr.dataInterface.source.impl.beisen.util.BeiSenApiUtil
```

## 二、常用命令
```bash
#搜索类
sc *BeiSen*

#查看类的信息
sc -d com.weaver.intcenter.hr.dataInterface.source.impl.beisen.util.BeiSenApiUtil

#查看类有哪些方法（`sm` = **show method**）
sm com.weaver.intcenter.hr.dataInterface.source.impl.beisen.util.BeiSenApiUtil

#查看这个方法具体代码
jad com.weaver.ebuilder.form.view.list.dao.datarecycle.EbCommonDataRecycle

#方法调用追踪（看入参、返回值、异常）
watch com.weaver.intcenter.hr.dataInterface.source.impl.beisen.util.BeiSenApiUtil getAccessToken '{params,returnObj,throwExp}' -n 5

```