---
layout: default
title: Gradle 
parent: DevNotes
nav_order: 3
---

Here are Gradle  experience .
{: .fs-6 .fw-300 }


## Here are Gradle experience .
{: .no_toc .text-delta }

1. TOC
{:toc}


# Gradle

## 什么的gradle

Gradle Build Tool 是一款快速、可靠且适应性强的开源 构建自动化 工具，具有优雅且可扩展的声明式构建语言。 

Gradle 构建工具缩写为 Gradle 。



Gradle 是一种广泛使用且成熟的工具，拥有活跃的社区和强大的开发人员生态系统。

- Gradle 是 JVM 最常用的构建系统，也是 Android 和 Kotlin 多平台项目的默认系统。 它拥有丰富的社区插件生态系统。

- Gradle 可以使用其内置功能、第三方插件或自定义构建逻辑来自动化各种软件构建场景。

- Gradle 提供了一种高级、声明性且富有表现力的构建语言，使您可以轻松读取和编写构建逻辑。

- Gradle 速度快、可扩展，可以构建任何规模和复杂程度的项目。

- Gradle 可产生可靠的结果，同时受益于增量构建、构建缓存和并行执行等优化。


**支持的语言和框架**

Gradle 支持 Android、Java、Kotlin Multiplatform、Groovy、Scala、Javascript 和 C/C++。

**兼容的 IDE**

所有主要的 IDE 都支持 Gradle，包括 Android Studio、IntelliJ IDEA、Visual Studio Code、Eclipse 和 NetBeans。


## Gradle 安装

>需要先安装 Java JDK 版本 8 或更高版本

第 1 步 - 下载 最新的 Gradle 发行版

下载文件有两种风格：

- 仅二进制 （bin）
- 完整 （全部） 包含文档和源

我们建议下载 bin 文件。  [Gradle All Releases ](https://gradle.org/releases/)

第 2 步 - 解压缩分配

创建新目录 `d:\Gradle`。

解压 `gradle-7.0-bin.zip` 到目录 `d:\Gradle` 


第 3 步 - 配置您的系统环境

添加环境变量 GRADLE_HOME 并将其指向解压缩的目录。 

添加 %GRADLE_HOME%\bin 发送到您的 Path。 

变量名: GRADLE_HOME

变量名: D:\apache\gradle-7.0

编辑 path （系统变量[全局]或用户变量[局部]）

增加 `%GRADLE_HOME%\bin`

> 升级到其他版本的 Gradle 时，只需将 GRADLE_HOME 环境变量。

## Gradle 安装

核心概念包括：

第 1 部分 Gradle 概述
第 2 部分 Gradle 的包装
第 3 部分 Gradle 的命令行界面
第 4 部分 设置文件
第 5 部分 构建文件
第 6 部分 依赖关系管理
第 7 部分 任务
第 8 部分 插件
第 9 部分 构建扫描
第 10 部分 Gradle 优化 



本教程涵盖：

第 1 部分。 初始化项目
第 2 部分。 运行任务
第 3 部分。 了解依赖关系
第 4 部分。 应用插件
第 5 部分。 探索增量构建
第 6 部分。 启用缓存
第 7 部分。 使用参考资料 