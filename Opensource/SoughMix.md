---
layout: default
title: SoughMix
parent: OpenSource
has_children: false
nav_order: 73
---

Here are opensource experience .
{: .fs-6 .fw-300 }


## Here are  SoughMix .
{: .no_toc .text-delta }

1. TOC
{:toc}


# 枕声 SoughMix：一个隐私优先的开源白噪音 App

> 记录一下「枕声 SoughMix」这个业余项目的来龙去脉，以及一些技术上的固执。

## 起因：一个睡不着的夜

有段时间我经常失眠。不是那种吵闹的失眠，而是脑子关不掉——白天的事像后台进程一样，一个接一个杀不干净。

我试过不少助眠 App。有的是算法合成的白噪音，听久了有一种电子味，而且循环点很明显，隔一会儿就能听出来"又回到开头了"；有的好一点，用了真实录音，但免费版每隔几分钟弹一次广告，本来快睡着了，一下子被吵醒；还有的打开就让你注册登录，一个放声音的工具，凭什么要知道我是谁。

后来我想，这些需求其实很简单，干脆自己写一个。

枕声是一款自然声音混音 App，支持 Android 与 Windows。代码 MIT 开源，音源采用 CC 授权素材，所有数据本地存储。当前版本 v1.2.0。

- 仓库：[https://github.com/small-rose/SoughMix](https://github.com/small-rose/SoughMix)
- 官网：[https://mix.zhangxiaocai.cn](https://mix.zhangxiaocai.cn)
- 下载：[https://github.com/small-rose/SoughMix/releases](https://github.com/small-rose/SoughMix/releases)（arm64 / armv7 / x86_64 三架构，正式签名）

## 功能特性

| 项目 | 说明 |
|---|---|
| 音源 | 27 个真实采集录音，分自然（13）/ 田园（8）/ 生活（3）/ 噪音（3）四组 |
| 混音 | 最多 6 轨同时播放，每轨独立音量与静音，总音量统一调节 |
| 内置方案 | 11 组，覆盖雨天篝火、森林晨间、海边夜晚、粉噪深眠等场景 |
| 睡眠 / 休息计时 | 5–480 分钟定时，结束前 5 分钟渐弱；休息模式反向渐强 |
| 后台播放 | 前台服务 + 系统媒体通知，息屏持续播放，锁屏与通知栏可控制 |
| 外观 | 4 套内置主题、强调色自定义、主题导入导出、浅色/深色/跟随系统三种桌面图标 |
| 桌面端 | 同一套 Flutter 代码，Windows 系统托盘常驻 |

## 隐私设计

无账号体系、无埋点上报、无第三方 SDK。方案、主题与设置仅存于设备本地 SharedPreferences，卸载即无痕。

## 技术实现

**音频处理**：素材来自 Wikimedia Commons 等平台的 CC 授权真实录音，经响度归一化（loudnorm）与 1 秒交叉淡化无缝循环处理。处理脚本开源于 `app/tools/`，可完整复现。

**媒体通知**：未使用 `audio_service` 的默认通知——其在 HyperOS / MIUI 上存在通知无法推送的问题。改为 Android 侧自建 MediaSession、直接发送 MediaStyle 前台通知，Dart 侧每秒推送状态；标题显示当前混音，副标题实时显示倒计时，支持上一首/下一首切换方案与播放暂停。

**计时精度**：倒计时基于系统墙钟计算剩余时间，定时器仅负责刷新界面，避免息屏后 Doze 节流导致的时间漂移。

**跨端**：Android 与 Windows 共用一套 Dart 代码，桌面端通过系统托盘常驻。

## 开源许可

- 代码：[MIT License](LICENSE)
- 音频素材：不受 MIT 覆盖，遵循上游 CC0 / CC BY / CC BY-SA 许可，逐条出处见 [`docs/音频素材授权清单.csv`](音频素材授权清单.csv)。商用或再分发时需分别处理。

欢迎提 Issue 与 Pull Request。
