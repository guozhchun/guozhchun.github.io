---
title: 使用 javadoc2 插件快速生成 javadoc 注释
date: 2019-08-24 10:56:15
tags: [javadoc, plugin]
categories: 工具
---

# 前言

最近项目新增一个指标：代码中每个 protected、public 方法和成员变量都必须有 javadoc 注释，由于存在大量不符合规范的历史代码。如果一个个找出手工添加，必将花费大量的精力。所以寻求插件批量解决。javadoc2 插件刚好可以匹配这些要求。

<!-- more -->

# Javadoc2 插件使用

在 `File -> Settings -> Plugin` 中搜索 `javadoc2`，点击安装，重启 Intellij IDEA。

![javadoc2 plugin](/images/javadoc2-1.png)

在 settings 中找到 javadoc2，根据自己需要进行设置调整

![javadoc2 setting](/images/javadoc2-2.png)

选中要生成 javadoc 注释的包或文件，右键，在弹出菜单中，选择 `JavaDocs -> Create JavaDocs for all elements`

![javadoc2 create javadoc](/images/javadoc2-3.png)

修改前后对比截图如下

修改前

![javadoc2 before update](/images/javadoc2-4.png)

修改后

![javadoc2 after update](/images/javadoc2-5.png)