---
layout: default
title:  gRPC
parent: NET
grand_parent: Java
nav_order: 1
---

Here are about Java gRPC about .
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


---


## gRPC

## gRPC 是什么
gRPC（gRPC Remote Procedure Calls）是由 Google 开源的高性能、跨语言的远程过程调用（RPC）框架。它基于 HTTP/2 协议传输，默认使用 Protocol Buffers（protobuf）作为接口定义语言和序列化工具，支持双向流、流控、头部压缩等特性，具有低延迟、高吞吐的特点。

## 如何理解 gRPC？

可以把它想象成“远程函数调用”——客户端像调用本地函数一样，调用运行在其他服务器上的方法，而底层网络通信细节被框架封装。

具体来说：
 - IDL（接口定义语言）：通过 **.proto** 文件定义服务接口和消息结构，实现语言中立。
 - 序列化：Protobuf 将结构化数据高效地编码为二进制，体积小、解析快。
 - 传输层：HTTP/2 提供多路复用、流式传输、服务器推送等能力，一个连接可并发处理多个请求。

### 四种通信模式：

- 一元 RPC（Unary）：客户端发送一个请求，服务端返回一个响应。
- 服务端流式 RPC：客户端发一个请求，服务端持续返回多个响应（如实时日志）。
- 客户端流式 RPC：客户端连续发送多个请求，服务端汇总后返回一个响应（如批量上传）。
- 双向流式 RPC：双方独立发送流式数据（如聊天、实时游戏）。

### 主要应用场景

- 微服务间通信：云原生架构中，服务之间需要高效、强类型的交互，gRPC 比 RESTful JSON 更轻量、更快。
- 移动端与后端通信：利用 HTTP/2 的多路复用和头部压缩，减少网络开销，适合弱网环境。
- 流式数据处理：实时推送、监控指标、物联网设备数据采集等需要持续传输数据的场景。
- 多语言异构系统集成：同一 .proto 文件可自动生成多种语言（Go、Java、Python、C++、Node.js 等）的客户端/服务端代码。
- 内部 API / 高性能网关：对延迟敏感的内部服务（如推荐系统、搜索引擎）常采用 gRPC。

## 常用框架与生态

- gRPC 官方实现：各语言均有官方库（grpc-go、grpc-java、grpc-node 等），功能完整。
- gRPC-Gateway：可同时暴露 gRPC 和 RESTful API，便于与前端或传统系统对接。
- Envoy / Istio：服务网格中常将 gRPC 流量代理到 sidecar，实现负载均衡、熔断、观测。
- Buf：高效的 protobuf 构建、lint、格式化工具，简化 .proto 管理。
- Connect-Web：专为浏览器设计的 gRPC 实现，支持纯 HTTP/1.1 和 WebSocket，无需 gRPC-Web 代理。
