---
layout: default
title: Spring AI 1.X 理论
parent: Spring
grand_parent: Java
nav_order: 90
---

Here are Spring AI 1.X record .
{: .fs-6 .fw-300 }


## Here are Spring AI 1.X tutorial .
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 01 · 原理篇：定义特性原理与选型对比

> 阶段：入门 → 精通

> 本篇目标：搞清楚 Spring AI **是什么**（定义与特性）、**怎么跑起来的**（架构与原理）、**该不该用它**（vs LangChain4j 选型），并给出 **1.x → 2.0 升级指引**。

> 技术基线：Spring AI 1.1.8 · Spring Boot 3.5.x · JDK 17+（本篇所有 API 均经 1.1.8 实际 jar 反查校验）



# 第一部分 · 定义与特性（入门）

## 1. Spring AI 是什么

官方定义（spring.io/projects/spring-ai）：

> **Spring AI is an application framework for AI engineering.** Its goal is to apply to the AI domain Spring ecosystem design principles such as portability and modular design and promote using POJOs as the building blocks of an application to the AI domain.
>
> At its core, Spring AI addresses the fundamental challenge of AI integration: **Connecting your enterprise Data and APIs with the AI Models.**

翻译成三句话：

1. **Spring AI 是 Spring 官方推出的 AI 工程应用框架**——是和 Spring MVC、Spring Data 同级的"全家桶成员"。
2. **它把 Spring 的设计哲学搬到 AI 领域**：可移植性（换模型不改代码）、模块化（按需引依赖）、POJO 编程（AI 的输入输出都是普通 Java 对象）。
3. **它解决的核心问题是集成**：把企业的**数据**（文档、数据库）和 **API**（业务接口）与 **AI 模型**（大模型服务）连起来。

一句话记忆：

> **Spring AI 之于大模型，就像 Spring Data 之于数据库。**
>
> 你不需要为 MySQL、MongoDB、Redis 各学一套 API——`JpaRepository` 一个抽象全覆盖；同理，你不需要为 OpenAI、DeepSeek、Ollama 各写一套客户端——`ChatClient` 一个抽象全覆盖，换模型只改配置文件。

### 1.1 它不是什么

| 误区 | 澄清 |
|------|------|
| 不是 Python LangChain 的 Java 移植 | 灵感来自 LangChain/LlamaIndex，但 API 完全按 Spring 习惯重新设计 |
| 不是只能聊天的 SDK | 覆盖对话、嵌入、文生图、语音转写、语音合成、内容审核 6 大模型能力 |
| 不是训练/微调框架 | 只做**推理接入**与**应用编排**，不负责模型训练 |
| 不是必须上云 | 可接云端 API（DeepSeek/OpenAI…），也可接本机 Ollama/vLLM 私有部署模型 |

## 2. 演进历程与版本线

| 时间 | 里程碑 |
|------|--------|
| 2024 年 | Spring 官方立项，陆续发布 1.0.0-M 系列里程碑版本 |
| 2025 年 5 月 | **1.0 GA** 正式发布，进入生产可用（ChatClient / Advisor / RAG / Tool Calling 全套能力定型） |
| 2025 年下半年 | **1.1 发布**（当前维护版 **1.1.8**），配套 Spring Boot 3.4.x / 3.5.x，新增/强化 `spring.ai.model.chat` 选择器、ToolCallAdvisor、Ollama 思考模式等 |
| 2026 年 | **2.0 发布**（当前稳定版 2.0.1），配套 Spring Boot 4.x，工具调用架构重构、配置属性扁平化——详见本篇 §14 升级指引 |

当前版本线（截至本教程写作时）：

| 版本线 | 最新稳定版 | 配套 Spring Boot | 说明 |
|--------|-----------|------------------|------|
| **1.1.x** | **1.1.8** | 3.4.x / 3.5.x | **本教程基线**，1.x 最新维护线 |
| 1.0.x | 1.0.9 | 3.4.x / 3.5.x | 首个大版本的维护线 |
| 2.0.x | 2.0.1 | 4.0.x / 4.1.x | 新主线，有破坏性变更（见 §14） |
| 2.1.x | 2.1.0-M1（里程碑） | — | 预览 |

> ⚠️ **本教程全部代码基于 Spring AI 1.1.8 + Spring Boot 3.5.16。**
> 若你计划升级 2.0，请重点阅读第二部分末尾的「1.x → 2.0 升级指引」；
> 若你的项目是 Spring Boot 4.x，请直接使用配套的 [2.0 版教程]()。

## 3. 核心特性全表

| # | 特性 | 一句话说明 | 解决什么问题 |
|---|------|-----------|-------------|
| 1 | **可移植 API** | 同一套 `ChatClient`/`EmbeddingModel` 接口，20+ 供应商多实现 | 换模型不改业务代码 |
| 2 | **多能力覆盖** | 对话补全、嵌入向量、文生图、音频转写、语音合成、内容审核 | 一个框架接全 AI 能力 |
| 3 | **同步 + 流式** | 每种模型调用都有 `call()` 与 `stream()` 双形态 | 打字机效果、长文本生成 |
| 4 | **ChatClient 流式 API** | 语法风格与 `RestClient`/`WebClient` 一脉相承 | 链式调用，可读性高 |
| 5 | **Advisor API** | 拦截每次模型调用，注入记忆、RAG、日志、敏感词过滤 | 横切能力不侵入业务 |
| 6 | **结构化输出** | AI 返回的 JSON 自动映射成 Java POJO | 告别手写 JSON 解析 |
| 7 | **Tool Calling** | 用 `@Tool` 注解把 Java 方法注册给模型调用 | AI 操作数据库/查接口/发工单 |
| 8 | **向量库 + RAG** | 20+ 向量数据库统一 API + `QuestionAnswerAdvisor` | 企业知识库问答 |
| 9 | **对话记忆** | `ChatMemory` 多会话窗口记忆，可落库（JDBC/Cassandra/Neo4j/Redis） | 多轮对话"记得住" |
| 10 | **MCP 协议** | 客户端/服务端 Starter，把 Spring Bean 暴露为 MCP 工具 | 接入 AI 工具生态 |
| 11 | **可观测性** | Micrometer 指标 + Tracing 链路，覆盖模型调用每一步 | 生产排障与成本核算 |
| 12 | **Boot 自动配置** | 所有模型/向量库都有 Starter，start.spring.io 勾选即用 | 零样板接入 |

## 4. Spring AI 1.1 关键能力（相对 1.0 GA 的主要演进）

| 能力 | 说明 |
|------|------|
| **`spring.ai.model.chat` 选择器** | 顶层属性按供应商 id 选择对话模型自动装配（缺省时全部生效）——多供应商共存的关键（详见 §8，jar 实证 `havingValue` + `matchIfMissing=true`） |
| **ToolCallAdvisor** | 顾问式工具执行：把工具循环从 `ChatModel` 内部接管到 Advisor 链，可与记忆 Advisor 组合（详见 §13） |
| **Ollama 思考模式** | `OllamaChatOptions.builder().enableThinking()/thinkHigh()`，支持 `ThinkOption`（`ENABLED/DISABLED` 与 `LOW/MEDIUM/HIGH`） |
| **结构化输出强化** | `StructuredOutputChatOptions`/`outputSchema` 等供应商侧结构化能力 |
| **工具流式响应** | `ToolCallAdvisor` 的 `streamToolCallResponses`/`suppressToolCallStreaming` 控制工具结果的流式行为 |

> 注意：DeepSeek 的"深度思考"在 1.1.8 中**不是属性开关**，而是直接换用推理模型
> `deepseek-reasoner`，再从 `DeepSeekAssistantMessage.getReasoningContent()` 读思维链（见 02 篇案例一）。

## 5. 核心概念字典

学 Spring AI 只需记住下面这张表（后文反复使用）：

| 概念 | 是什么 | 类比 |
|------|--------|------|
| **ChatClient** | 与 AI 对话的**高层流式 API**，`prompt().user().call()` 链式调用 | `RestClient`/`WebClient` |
| **ChatModel** | 与 AI 对话的**底层接口**，`call(Prompt)` / `stream(Prompt)` | `JpaRepository`（接口）与 MySQL 实现 |
| **Prompt** | 一次请求的完整内容 = 若干条 Message + 运行时选项（模型、温度…） | HTTP Request |
| **Message** | 单条消息，三种角色：`SystemMessage`（人设规则）、`UserMessage`（用户输入）、`AssistantMessage`（AI 回复） | 请求体里的 role/content |
| **ChatResponse** | 模型返回：`AssistantMessage` 正文 + `Usage`（token 用量）+ `Metadata` | HTTP Response |
| **Advisor** | 拦截模型调用的**责任链组件**，在请求前后注入记忆、RAG 上下文、日志等 | Spring AOP / Servlet Filter |
| **Tool** | 用 `@Tool` 标注的 Java 方法，模型可请求执行它 | 后端提供的"给 AI 用的 API" |
| **EmbeddingModel** | 把文本转成向量的模型接口 | 文本 → 坐标点 |
| **VectorStore** | 向量数据库统一接口（存/查语义相似的文档） | "语义版"的 `JpaRepository` |
| **ChatMemory** | 多轮对话记忆，按 `conversationId` 隔离，窗口策略淘汰旧消息 | 有状态的 HttpSession（但可换实现） |

### 5.1 最小可运行示例

依赖（BOM + DeepSeek Starter，完整版见 02 篇案例一）：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-deepseek</artifactId>
</dependency>
```

配置（**1.x 嵌套属性**，注意 `.options` 段）：

```yaml
spring:
  ai:
    deepseek:
      api-key: ${DEEPSEEK_API_KEY}
      chat:
        options:
          model: deepseek-chat        # 1.x 写法；2.0 为 .chat.model
```

代码：

```java
@RestController
public class HelloController {

    private final ChatClient chatClient;

    public HelloController(ChatClient.Builder builder) {   // 自动配置注入
        this.chatClient = builder.build();
    }

    @GetMapping("/hello")
    public String hello(@RequestParam(defaultValue = "用一句话介绍 Spring AI") String msg) {
        return chatClient.prompt()
                .system("你是一名简洁的 Java 技术专家")
                .user(msg)
                .call()
                .content();
    }
}
```

## 6. ChatClient 还是 ChatModel？

| 场景 | 推荐 | 原因 |
|------|------|------|
| 日常业务开发（90%） | **ChatClient** | 链式 DSL、自动装配 Advisor、支持记忆/RAG/工具 |
| 聊天机器人 / RAG | **ChatClient** | Advisor 一行接入 |
| 需要完全控制 Prompt 组装 | ChatModel | 手动拼 `Prompt`/`Message` |
| 框架级开发、性能基准测试 | ChatModel | 少一层抽象 |
| 获取原始 `ChatResponse` 元数据 | 两者皆可 | ChatClient 用 `.chatClientResponse()` |

> **经验法则：拿不准就用 ChatClient。** 只有当你明确知道 ChatClient 满足不了时，再下沉到 ChatModel。
> 反向依赖是允许的：`ChatClient.builder(chatModel)` —— ChatClient 建立在 ChatModel 之上。

### 第一部分小结

- Spring AI = Spring 官方 AI 工程框架，核心价值是**可移植抽象**（一套接口换遍模型）。
- 三大支柱：**ChatClient（调用）+ Advisor（增强）+ 自动配置（装配）**。
- 本教程基线是 1.x 最新维护线：Boot 3.5.x + Spring AI 1.1.8 + `.options` 嵌套属性。
- 90% 场景用 ChatClient，底层才是 ChatModel。

---

# 第二部分 · 原理与运行机制（高级）

## 7. 三层 API 架构

```
┌──────────────────────────────────────────────────────────┐
│  应用层    ChatClient（Fluent API）                        │
│           prompt().system().user().advisors().call()      │  ← 日常开发只碰这一层
├──────────────────────────────────────────────────────────┤
│  抽象层    ChatModel / StreamingChatModel（统一接口）        │
│           ├─ DeepSeekChatModel     （案例一）              │
│           ├─ OllamaChatModel       （案例二）              │
│           ├─ OpenAiChatModel       ├ AnthropicChatModel   │
│           └─ …… 共 20+ 实现                               │
│           横向：Advisor 链（CallAdvisor / StreamAdvisor）    │  ← 记忆、RAG 在这里拦截
│           纵向：ChatModel 内部工具执行循环（1.x 特色）        │  ← 见 §13
├──────────────────────────────────────────────────────────┤
│  传输层    RestClient / WebClient + 各供应商私有 API Client   │
│           JSON 序列化 · SSE 流解析 · 重试退避 · 代理/证书      │  ← 自动配置，通常无感
└──────────────────────────────────────────────────────────┘
```

设计意图：**每层只暴露必要复杂度**。你写业务时只见 `ChatClient`；要换供应商时只动配置（抽象层实现切换）；传输细节（超时、重试、SSE 心跳）完全封装在底层。

## 8. 自动配置原理

Spring AI 的接入体验完全遵循 Spring Boot 的"约定优于配置"：

```
spring-ai-starter-model-deepseek          ← ① 你加的依赖
        │ 传递依赖
        ▼
spring-ai-autoconfigure-model-deepseek    ← ② 自动配置模块
        │ @AutoConfiguration + @EnableConfigurationProperties
        ▼
spring.ai.deepseek.*  → DeepSeekChatProperties ← ③ 属性绑定（@ConfigurationProperties）
        │ Builder 装配
        ▼
DeepSeekApi → DeepSeekChatModel → ChatClient.Builder ← ④ 产出 Bean
```

关键点：

1. **Starter 只是"入场券"**：真正的逻辑在 `spring-ai-autoconfigure-*` 模块，通过
   `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
   注册自动配置类。
2. **属性即插即用**：所有 `spring.ai.*` 属性绑定到 `*Properties` 类。1.x 的对话参数全部在
   **`.options` 段**下（`spring.ai.deepseek.chat.options.model`），连接参数（api-key、base-url）
   在供应商根前缀下（`spring.ai.deepseek.api-key`）。
3. **多模型启停是"双机制"**（1.x 特有，两者并存）：

```yaml
# 机制一：供应商选择器——jar 实证各供应商自动配置类标注
#   @ConditionalOnProperty(name = "spring.ai.model.chat",
#                          havingValue = "deepseek", matchIfMissing = true)
# 语义：属性缺省 → 所有已引入的供应商照常自动装配（classpath 只有一个 starter 时无歧义）；
#       设置为某 id  → 只有命中的供应商自动装配，其余 back off（手动构建的模型不受影响）。
spring:
  ai:
    model:
      chat: deepseek          # 可选：openai / deepseek / ollama / anthropic ...

# 机制二：供应商级开关（deepseek、openai 等仍提供；ollama 在 1.1.8 已移除该属性）
spring.ai.deepseek.chat.enabled=true
```

4. **注入即所得**：任何自动配置好的组件都能直接构造注入——`ChatClient.Builder`、
   `DeepSeekChatModel`、`ChatMemory`、`VectorStore`……

## 9. 一次调用的完整链路

以 `chatClient.prompt().user("你好").call().content()` 为例：

```
你的代码
  │ ① 构建请求
  ▼
DefaultChatClient.prompt() 组装 ChatClientRequest { Prompt, advisor 参数 }
  │ ② 进入 Advisor 责任链（按 getOrder() 升序）
  ▼
[MessageChatMemoryAdvisor]  按 conversationId 取出历史消息，拼进 Prompt
  │  chain.nextCall(request)
  ▼
[QuestionAnswerAdvisor]     把用户问题向量化 → 检索向量库 → 注入检索片段
  │  chain.nextCall(request)
  ▼
③ ChatModel.call(Prompt)    ← DeepSeekChatModel 实现
  │  合并默认 Options（模型名、温度…，可用请求级 options 覆盖）
  │  若请求携带了工具定义且 internalToolExecutionEnabled=true（默认）：
  ▼
③' 工具执行循环（ChatModel 内部，1.x 特色）           ← 详见 §13
  │  模型返回 tool_calls → 反射执行 @Tool 方法 → 结果回填 → 再次请求模型
  │  → 直到模型不再请求工具
  ▼
DeepSeekApi (底层客户端)
  │  ④ Jackson 序列化 → RestClient HTTPS POST https://api.deepseek.com/chat/completions
  ▼
DeepSeek 服务端推理
  │  ⑤ 返回 JSON（assistant 消息 + usage + …）
  ▼
反序列化为 ChatResponse { AssistantMessage, Usage, Metadata }
  │ ⑥ 沿 Advisor 链原路返回（每个 Advisor 可在响应后处理：记日志、存记忆…）
  ▼
.chatClientResponse().chatResponse().getResult().getOutput().getText()
  │ ⑦ .content() 取出正文返回给你
```

要点：

- **Advisor 是"双向"的**：既改请求（注入上下文），也改响应（截断、脱敏、落库）。
- **工具循环在 ChatModel 内部**：1.x 里模型若返回 `tool_calls`，循环发生在 ③'（实现内部），
  上层通常无感；也可用 ToolCallAdvisor 把这段循环上移到 Advisor 链（§13）。
- **一切皆可换**：换 `spring.ai.model.chat=ollama`，③ 之后的整条下半段自动变成本地 HTTP 调用，你的代码一行不动。

## 10. 流式输出原理

同步调用一次性拿完整 JSON；流式调用基于 **SSE（Server-Sent Events）**：

```
你的 Flux<String>          ← ⑥ 逐步发射文本增量
   ▲
   │ map(chunk -> chunk delta)
ChatModel.stream(Prompt)   ← ⑤ Flux<ChatResponse>（每个元素是一个增量片段）
   ▲
   │ 解析每个 data: {...} 事件
HTTP POST /chat/completions (stream=true)  ← ④ 服务端持续推送 SSE 行
   │    data: {"choices":[{"delta":{"content":"你"}}]}
   │    data: {"choices":[{"delta":{"content":"好"}}]}
   │    data: [DONE]
```

代码形态对照：

```java
// 同步：阻塞等待完整结果
String answer = chatClient.prompt().user(q).call().content();

// 流式：逐块发射（打字机效果），StreamAdvisor 链同样生效
Flux<String> answerStream = chatClient.prompt().user(q).stream().content();
```

Controller 层以 SSE 输出（完整案例见 02 篇）：

```java
@GetMapping(value = "/ai/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<String>> stream(@RequestParam String message) {
    return chatClient.prompt().user(message).stream().content()
            .map(chunk -> ServerSentEvent.builder(chunk).build());
}
```

## 11. 结构化输出原理

模型原生只会吐**文本**。把文本变 POJO 的机制是 `BeanOutputConverter`：

```
① 你声明目标类型            Ticket(String title, String priority, String ticketNo)
② 框架用 Jackson 推导 JSON Schema
③ 把 Schema 作为指令追加进 Prompt   "请返回符合以下 Schema 的 JSON：{...}"
④ 模型返回 JSON 文本
⑤ Jackson 反序列化 → POJO            （解析失败直接抛出转换异常）
```

调用方式（一行）：

```java
public record Ticket(String title, String priority, String ticketNo) {}

Ticket ticket = chatClient.prompt()
        .user("帮我把这段投诉转成工单：快递丢件了，单号 SF123，很急")
        .call()
        .entity(Ticket.class);        // ← 结构化输出入口
```

泛型集合用 `ParameterizedTypeReference`。1.x 阶段的可靠性主要靠**提示词约束 + Schema 指令**，
Schema 校验自动重试自纠与供应商原生 Structured Output 是 2.0 的增强（见 §14）。

## 12. Advisor 责任链机制

Advisor 是 Spring AI 最有"Spring 味"的设计——本质就是 **AOP/Filter 链**在 AI 场景的翻版。

### 12.1 核心接口（1.1.8 实际签名）

```java
public interface Advisor extends Ordered {
    String getName();                               // 链中标识
    // getOrder() 来自 Ordered：值越小越先执行
    // 常量 DEFAULT_CHAT_MEMORY_PRECEDENCE_ORDER = Integer.MIN_VALUE + 1000
}

// 同步场景
public interface CallAdvisor extends Advisor {
    ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain);
}

// 流式场景（1.1.8 中同一接口亦可实现流式方法；BaseAdvisor 同时覆盖两者）
public interface StreamAdvisor extends Advisor {
    Flux<ChatClientResponse> adviseStream(ChatClientRequest request, StreamAdvisorChain chain);
}

// 请求/响应载体（record 类型）
public record ChatClientRequest(Prompt prompt, Map<String, Object> context) {
    // prompt() 待发送的完整 Prompt；context() 责任链内共享的上下文
}
```

最后一个 Advisor 由框架自动追加，负责真正调用 LLM。

> 说明：1.0 GA 已完成更名（`AdvisedRequest` → `ChatClientRequest` 等），**1.1.8 的 Advisor API
> 与 2.0 完全同构**——本节代码在两个版本通用，是升级时最"无痛"的部分。

### 12.2 自定义一个日志 Advisor（完整代码）

```java
@Slf4j
@Component
public class LoggingAdvisor implements CallAdvisor {

    @Override
    public ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain) {
        String userText = request.prompt().getInstructions();      // 待发给模型的消息列表
        log.info(">>> 请求模型: {}", userText);
        long start = System.currentTimeMillis();

        ChatClientResponse response = chain.nextCall(request);     // 放行，进入下一个 Advisor

        long cost = System.currentTimeMillis() - start;
        Usage usage = response.chatResponse() != null
                ? response.chatResponse().getMetadata().getUsage() : null;
        log.info("<<< 耗时 {} ms, tokens={}", cost, usage);
        return response;                                           // 可在此改写响应
    }

    @Override
    public String getName() {
        return "LoggingAdvisor";
    }

    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE + 100;                   // 尽早记录
    }
}
```

注册与使用：

```java
// 方式一：全局默认（Builder）
ChatClient chatClient = ChatClient.builder(chatModel)
        .defaultAdvisors(new LoggingAdvisor())
        .build();

// 方式二：单次调用生效
chatClient.prompt()
        .advisors(new LoggingAdvisor())
        .user("你好")
        .call().content();
```

### 12.3 内置 Advisor 与执行顺序（1.1.8）

| Advisor | 职责 | 默认顺序（HIGHEST_PRECEDENCE = Integer.MIN_VALUE） |
|---------|------|--------------------------------------------------|
| `ToolCallAdvisor` | 工具执行循环（**可选启用**，见 §13） | **+ 300**（`.advisorOrder(int)` 可调） |
| `MessageChatMemoryAdvisor` | 注入/写回对话记忆 | **+ 1000**（= `Advisor.DEFAULT_CHAT_MEMORY_PRECEDENCE_ORDER`，`.order(int)` 可调） |
| `QuestionAnswerAdvisor` | RAG 检索片段注入 | 按注册顺序（实现 `BaseAdvisor`） |

两点须知：

1. `ToolCallAdvisor` **默认不在链中**——需要时手动构建并注册（1.x 是"顾问式可选"，
   2.0 的 `ToolCallingAdvisor` 才是自动注册的默认行为）。
2. 与记忆 Advisor 组合时，官方建议记忆在前、工具循环在后（顺序值小者先执行）：

```java
var chatMemoryAdvisor = MessageChatMemoryAdvisor.builder(chatMemory)
        .order(Ordered.HIGHEST_PRECEDENCE + 200)      // 先拼历史
        .build();
var toolCallAdvisor = ToolCallAdvisor.builder()
        .advisorOrder(Ordered.HIGHEST_PRECEDENCE + 300)  // 再进工具循环
        .disableMemory()                                 // 历史交给 ChatMemory 管
        .build();
```

运行时给 Advisor 传参（多会话隔离必用）：

```java
chatClient.prompt()
        .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, "user-666"))   // 会话隔离
        .user("我刚才说了什么？")
        .call().content();
```

### 12.4 为什么 Advisor 值得学透

| 横切需求 | 用 Advisor 实现 |
|----------|-----------------|
| 多轮对话记忆 | `MessageChatMemoryAdvisor` |
| 知识库问答 | `QuestionAnswerAdvisor` |
| 敏感词过滤/审计日志 | 自定义 Advisor，请求前拦截、响应后扫描 |
| 提示词防注入改写 | 自定义 Advisor 规整用户输入 |
| Token 成本统计 | 自定义 Advisor 读取 `Usage` 累加 |

## 13. 工具调用架构：1.x 的机制与进阶控制

### 13.1 默认路径：ChatModel 内部循环

1.x 中每个 `ChatModel` 实现**内部**都藏着一个工具执行循环——这是**默认行为**：

```
chatClient.prompt().tools(new WeatherTools()).call()
   │
   ▼
ChatModel.call(prompt with tools)
   → ① 工具定义（JSON Schema）随请求发给模型
   → ② 模型返回 tool_calls
   → ③ ChatModel 内部用 ToolCallingManager 反射执行 @Tool 方法
   → ④ 结果回填 Prompt，再次请求模型            ← 循环藏在实现里
   → ⑤ 直到模型给出最终回答才返回
```

```java
// 1.x 标准用法：注册工具即可，内部循环默认开启
String answer = ChatClient.create(chatModel)
        .prompt("北京现在天气怎么样？")
        .tools(new WeatherTools())              // 对象注册；也支持 .toolNames("weatherTools") 按 Bean 名注册
        .call()
        .content();
```

**可控开关**（挂在 Options 上）：

| 机制 | 默认值 | 说明 |
|------|--------|------|
| `internalToolExecutionEnabled` | **true**（框架默认执行） | 设为 false 则只下发工具定义、拿到 `tool_calls` 就返回，由你自己处理 |
| `spring.ai.chat.client.*` 限额 | **无** | 1.x 没有 2.0 的调用限额防护（`spring.ai.tools.limits.*` 是 2.0 新增，见 §14） |

### 13.2 进阶路径：ToolCallAdvisor 接管循环

若希望工具循环**可观测、可与其他 Advisor 编排**，用 `ToolCallAdvisor` 显式接管
——它会**自动禁用模型内部执行**，循环逻辑收敛到 Advisor 链（1.1.8 实际 API）：

```java
var toolCallAdvisor = ToolCallAdvisor.builder()
        // .toolCallingManager(...)         // 不传则用默认 DefaultToolCallingManager
        .advisorOrder(Ordered.HIGHEST_PRECEDENCE + 300)
        .disableMemory()                    // 循环内历史交给外部 ChatMemory 管理
        // .conversationHistoryEnabled(true) // 默认 true：Advisor 内部维护循环全程历史
        // .suppressToolCallStreaming()      // 工具阶段不向客户端流式透传
        .build();

ChatClient chatClient = ChatClient.builder(chatModel)
        .defaultAdvisors(chatMemoryAdvisor, toolCallAdvisor)   // 记忆在前（+200）
        .build();
```

| Builder 方法 | 作用 |
|--------------|------|
| `toolCallingManager(…)` | 指定工具执行器（反射调用 `@Tool` 方法的组件） |
| `advisorOrder(int)` | 链中位置，默认 `HIGHEST_PRECEDENCE + 300`（jar 字节码实证） |
| `conversationHistoryEnabled(boolean)` / `disableMemory()` | 循环内是否自管历史；与 ChatMemory Advisor 组合时用 `disableMemory()` |
| `streamToolCallResponses(boolean)` / `suppressToolCallStreaming()` | 工具结果是否随流式响应透传 |
| `@Tool(returnDirect = true)` | 工具结果不回给模型、直接返回客户端（Advisor 跳出循环） |

**选型建议**：简单场景用默认内部循环（13.1，零配置）；需要与记忆/审计深度编排、
或要 `returnDirect` 时上 ToolCallAdvisor（13.2）。

### 13.3 注解与历史包袱

```java
@Component
public class WeatherTools {

    @Tool(description = "查询指定城市的当前天气")                    // org.springframework.ai.tool.annotation.Tool
    public String getWeather(
            @ToolParam(description = "城市名，如 北京") String city) { // @ToolParam 控制必填与描述
        return city + "：晴，22℃";
    }
}
```

- 1.0 起工具 API 统一为 **`@Tool` + `.tools(...)` + `ToolCallback`**；
  更早里程碑里的 `.functions(...)`/`FunctionCallback` 已废弃删除。
- 1.1.8 仍保留 `.toolNames(...)`（按 Spring Bean 名注册）；2.0 收紧为仅显式挂载的对象工具。

## 14. 1.x → 2.0 升级指引

升级 2.0 前，按下面清单逐项核对（全部基于两边 jar 实证；**不变项先安心，再看变更项**）：

**先安心：这些完全不用改**

| 项 | 说明 |
|----|------|
| Advisor API | `CallAdvisor.adviseCall(ChatClientRequest, CallAdvisorChain)`、`getName()` 两版同构 |
| ChatClient DSL | `.prompt()/.system()/.user()/.tools()/.entity()/.stream()/.advisors(a->a.param(...))` 全部不变 |
| Starter 命名 | `spring-ai-starter-model-deepseek/openai/ollama` 两版相同 |
| 记忆/RAG 用法 | `MessageWindowChatMemory`/`MessageChatMemoryAdvisor`/`QuestionAnswerAdvisor` 用法相同 |

**必须逐项处理的变更**

| # | 变更 | 1.x 写法（1.1.8） | 2.0 写法 |
|---|------|------------------|----------|
| 1 | 配置属性扁平化 | `spring.ai.deepseek.chat.options.model` | `spring.ai.deepseek.chat.model` |
| 2 | 多模型启停 | `spring.ai.model.chat`（选择器，缺省全开）+ `spring.ai.<p>.chat.enabled`（deepseek/openai 仍有，ollama 已移除） | 统一为 `spring.ai.model.chat=<provider>/none`，`chat.enabled` 全部移除 |
| 3 | ChatModel Builder | `.builder().…​.defaultOptions(options)` | `.options(options)` |
| 4 | Options 可变性 | `options.copy()` 后修改；`getDefaultOptions()` | 严格不可变：`options.mutate().….build()`；`getOptions()` |
| 5 | 工具执行循环 | `ChatModel` 内部自动执行（`internalToolExecutionEnabled` 默认 true） | 循环移至自动注册的 **`ToolCallingAdvisor`**；`ChatModel` 只返回原始响应 |
| 6 | 顾问式工具 | `ToolCallAdvisor`（手动构建才生效） | 更名 `ToolCallingAdvisor` 且自动注册（`spring.ai.chat.client.tool-calling.enabled` 可关） |
| 7 | 工具调用限额 | 无 | 单工具每轮 40 次、单请求总计 150 次（`spring.ai.tools.limits.*`） |
| 8 | RAG 模块名 | `spring-ai-advisors-vector-store` | `spring-ai-vector-store-advisor`（类名不变） |
| 9 | Ollama 思考控制 | 程序化 `.enableThinking()/thinkHigh()`（`ThinkOption`；Options 绑定名 `options.think-option`，1.1 文档推荐程序化控制） | 扁平属性 `spring.ai.ollama.chat.think` |
| 10 | DeepSeek 思考 | 换模型 `deepseek-reasoner` + `getReasoningContent()` | v4 系列 + `thinking.type`/`reasoning-effort` 属性 + `enableThinking()` |
| 11 | 结构化输出 | 提示词约束 + Schema 指令 | Schema 校验失败自动重试自纠、供应商原生 Structured Output |
| 12 | `.N(…)` 方法名 | 个别 Builder 方法带大写首字母残留 | 统一为小写 `.n(…)` 风格 |

> 升级口诀：**先全局搜 `.chat.options.` 和 `.defaultOptions(`，再搜 `internalToolExecutionEnabled`；
> Advisor/ChatClient 代码基本原样搬迁。** 迁移后跑一遍 02 篇两个案例的接口即可确认链路完整。

## 15. 可观测性

Spring AI 通过 Micrometer（OpenTelemetry 兼容）暴露：

- **Metrics**：每次模型调用的时长、token 消耗（输入/输出分开计）、失败次数——按模型、
  供应商打标签，直接接 Grafana 看板核算成本。
- **Tracing**：一次 RAG 问答会拆成「检索 → 组装 → 模型调用」多个 Span，跨服务全链路可见。
- **日志**：`spring.ai.chat.client` 相关日志开关可打印出入 Prompt。

接入方式与 Spring Boot 应用完全一致——引入 `micrometer-registry-prometheus` 即可，
这就是"Spring 全家桶亲和"的具体含义。

### 第二部分小结

- 三层架构：**ChatClient（DSL）→ ChatModel（多实现）→ RestClient（传输）**，换供应商只换实现。
- 自动配置四步走：Starter → AutoConfiguration → 属性绑定 → Builder 装配 Bean；
  多模型启停 = `model.chat` 选择器 + 供应商级 `chat.enabled` 双机制。
- Advisor = AI 场景的责任链/AOP：`adviseCall` + `chain.nextCall`，双向改写请求与响应。
- 1.x 工具调用 = **ChatModel 内部循环为默认**，`ToolCallAdvisor` 为可选接管；无 2.0 的限额防护。
- 升级 2.0 时优先核对 §14 的 12 条变更；Advisor API/DSL/Starter 命名不变，可放心。

---

# 第三部分 · Spring AI vs LangChain4j（精通）

> 本节对比基于 **Spring AI 1.1.8** 与 **LangChain4j 1.22.0**（均为写作时最新稳定版）。

## 16. 两者定位与背景

| | Spring AI | LangChain4j |
|---|-----------|-------------|
| 出身 | **Spring 官方项目**（spring-projects/spring-ai） | 社区项目，Python **LangChain 的 Java 对齐实现** |
| 哲学 | 把可移植性、模块化、POJO、依赖注入等 **Spring 生态设计原则**搬到 AI 领域 | 把 LangChain 的"链式编排 + 丰富集成"思想带进 Java |
| 首个 GA | 1.0（2025 年 5 月） | 1.0（2025 年） |
| 当前稳定版 | **1.1.8**（配套 Boot 3.5.x；另有 2.0.1 配套 Boot 4.x） | **1.22.0** |
| 许可 | Apache 2.0 | Apache 2.0 |
| 一句话概括 | "**AI 界的 Spring Data**"，服务 Spring 全家桶用户 | "**Java 界的 LangChain**"，服务所有 Java 用户 |

两者如今都是成熟的生产级框架——**"选哪个都能干活"，选型差异主要在团队技术栈与集成深度**。

## 17. 多维对比总表

| 维度 | Spring AI 1.1.8 | LangChain4j 1.22.0 |
|------|-----------------|---------------------|
| **核心调用 API** | `ChatClient` 流式 DSL（`prompt().user().call()`） | **`AiServices` 声明式接口**（定义 Java 接口 + 注解即得实现） |
| **底层抽象** | `ChatModel` / `EmbeddingModel` / `ImageClient`… | `ChatModel` / `EmbeddingModel` / `ModerationModel`… |
| **编程模型** | 依赖注入、POJO、Builder，深度遵循 Spring 习惯 | 普通 Java 库，Builder/工厂为主，不依赖 Spring |
| **装配方式** | **Boot Starter 自动配置为主**（start.spring.io 勾选即用），也可手工 Builder | 核心是独立库；另有官方 `langchain4j-spring-boot-starter` 可选 |
| **模型供应商** | 20+ 官方模块（OpenAI/Anthropic/Google/Amazon/Azure/**DeepSeek**/Ollama/Groq/Mistral/MiniMax/NVIDIA/Perplexity…） | 同样覆盖主流（OpenAI/Anthropic/Gemini/Azure/Ollama/Mistral/Cohere/HuggingFace…）+ 大量社区模块 |
| **向量数据库** | **20+ 官方支持**（PgVector/Cassandra/Milvus/Qdrant/Chroma/Redis/Weaviate/Elasticsearch/Neo4j/Oracle…）+ 类 SQL 元数据过滤 API | 主流基本覆盖（PgVector/Redis/Milvus/Qdrant/Chroma/Pinecone/Weaviate/Elasticsearch/MongoDB/内存版…） |
| **RAG 能力** | `VectorStore` + `QuestionAnswerAdvisor`（`spring-ai-advisors-vector-store` 模块），组合直觉 | `ContentRetriever`/`ContentAggregator`/查询转换器等 RAG 组件链，组合灵活 |
| **工具调用** | `@Tool` + **ChatModel 内部循环默认执行**（可换 `ToolCallAdvisor` 接管） | `@Tool`（`dev.langchain4j.agent.tool.Tool`）+ AI Services 内自动执行 |
| **对话记忆** | `ChatMemory` + Advisor，可落 JDBC/Cassandra/Neo4j/Redis | `ChatMemory`（`MessageWindowChatMemory` 等）+ 持久化 `ChatMemoryStore` |
| **MCP 支持** | client/server 双 Starter（`spring-ai-starter-mcp-*`），注解式暴露工具 | 有 `langchain4j-mcp` 模块，可用 |
| **可观测性** | Micrometer/OTel **原生集成**，Actuator/Trace 全家桶协同 | 提供 `ChatModelListener`/观测监听，接入生态需自行桥接 |
| **企业特性** | Testcontainers、Docker Compose 开发期服务、与 Spring Security/Authorization Server 组合 | 需自行组合其他库 |
| **学习曲线** | 中偏高：要理解 ChatClient/Advisor/自动配置等抽象 | **偏低**：AiServices 声明式接口直观，"定义接口→拿到实现" |
| **非 Spring 项目** | 能用（手工 Builder），但体验非最优 | **原生主场** |
| **版本治理** | 1.1.x（Boot 3.5）与 2.0.x（Boot 4）双线并行，升级有明确迁移面（见 §14） | 单线迭代，升级以小版本为主 |

## 18. 同一需求，两种写法

需求：**带系统人设的问答**——"你是资深 Java 工程师，回答控制在 100 字以内"。

### Spring AI 写法

```java
@Service
public class QAService {

    private final ChatClient chatClient;

    public QAService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    public String ask(String question) {
        return chatClient.prompt()
                .system("你是一名资深 Java 工程师，回答控制在 100 字以内")
                .user(question)
                .call()
                .content();
    }
}
```

### LangChain4j 写法

```java
public interface Assistant {

    @SystemMessage("你是一名资深 Java 工程师，回答控制在 100 字以内")
    String answer(String userMessage);
}

@Service
public class QAService {

    private final Assistant assistant;

    public Assistant assistant(ChatModel chatModel) {
        // AiServices：接口 + 注解 → 动态代理实现
        return AiServices.builder(Assistant.class)
                .chatModel(chatModel)
                .build();
    }

    public String ask(String question) {
        return assistant.answer(question);
    }
}
```

**差异本质**：

- Spring AI 是**对象协作式**——你显式组装请求（Prompt/Message/Advisor），控制粒度细。
- LangChain4j 是**接口契约式**——把"一次对话"抽象成 Java 接口方法，上手极快，
  但复杂控制（流式细节、响应元数据、分步拦截）要退回到更底层的 API。

再看"带记忆 + RAG + 工具"的完整场景，两者的组件数大体相当，Spring AI 用 Advisor
纵向叠加，LangChain4j 用 `ContentRetriever`/`ChatMemoryProvider`/`tools` 横向挂到
AiServices 上——**没有能力代差，只有组合方式的口味差异**。

## 19. 优缺点诚实评价

### Spring AI

| 优点 | 缺点 |
|------|------|
| Spring 官方维护，与 Security/Actuator/Micrometer/Boot 深度协同 | 抽象层多（ChatClient/Advisor/自动配置），**学习曲线偏陡** |
| 换模型零成本：改配置即可切换供应商 | 非 Spring 项目里优势打折 |
| 企业级特性齐全：可观测、Testcontainers、开发期服务、MCP Starter | 双版本线并行（1.1/2.0），选错基线会带来升级成本 |
| 向量库/模型官方模块数量领先 | 1.x 工具调用循环藏在 ChatModel 内，初学不易理解全貌 |
| 1.1.x 是久经生产的稳定线（Boot 3.5 存量项目首选） | 部分高级用法文档密度仍不及 Python 生态 |

### LangChain4j

| 优点 | 缺点 |
|------|------|
| **AiServices 声明式上手快**，原型速度极快 | 非 Spring 项目是主场，Spring 内集成深度不及 Spring AI |
| 纯 Java 库，零框架绑定，任何项目可用 | 可观测/安全等企业能力要自己拼装 |
| 与 Python LangChain 概念对齐，跨语言团队心智成本低 | 官方向量库/模型模块数量略少于 Spring AI |
| API 稳定迭代快，社区活跃 | 官方 Spring Boot Starter 的成熟度不如 Spring AI 自家自动配置 |
| 文档与示例对"入门第一课"更友好 | — |

## 20. 选型决策树

```
你的团队/项目是 Spring Boot 吗？
├── 是（Boot 3.4/3.5），且企业级要求高（安全、审计、监控、多供应商治理）
│        → ✅ Spring AI 1.1.x（本教程；全家桶协同是它最大杀器）
│
├── 是（Boot 4.x）
│        → ✅ Spring AI 2.0.x（直接用 2.0 版教程，不必先学 1.x）
│
├── 是，但只是小工具/轻量服务，追求最少概念
│        → 两者皆可：想少学概念用 LangChain4j，想统一技术栈用 Spring AI
│
├── 否（Quarkus/Micronaut/普通 Java SE/命令行工具）
│        → ✅ LangChain4j（库形态零负担）
│
├── 团队熟 Python LangChain，需要和 Python 服务概念对齐
│        → ✅ LangChain4j
│
└── 需要 MCP Starter / Spring Security 管控 AI 进出网关
         → ✅ Spring AI
```

补充判断依据：

| 你在意的点 | 选择 |
|-----------|------|
| "以后可能从 DeepSeek 换到本地 vLLM，要只改配置" | 都支持，Spring AI 的自动配置切换更顺 |
| "我想 10 分钟出原型" | LangChain4j（AiServices） |
| "RAG 要接 PgVector + 权限过滤 + 链路追踪" | Spring AI |
| "项目不引 Spring，还要接 5 个供应商" | LangChain4j |
| "存量 Boot 3.5 项目，不想动 Boot 大版本" | Spring AI 1.1.x（本教程） |
| "新项目直接 Boot 4" | Spring AI 2.0.x（[2.0 版教程](../SpringAI教程/README.md)） |

### 第三部分小结

- 两者定位同源思想不同族：**Spring AI = Spring 生态原则在 AI 领域的延伸**；
  **LangChain4j = LangChain 思想的 Java 化**。
- 能力面高度重叠（模型/向量库/RAG/工具/记忆/MCP 都有），差异在**集成深度 vs 上手速度**。
- 选型第一问永远是：**你的技术栈是不是 Spring？** 是 → 优先 Spring AI；否 → 优先 LangChain4j。
- Spring AI 内部第二问：**Boot 大版本是 3.5 还是 4.x？** → 决定用 1.1.x 还是 2.0.x。

---

## 全篇知识图谱

```
Spring AI（1.x = 1.1.8 基线）
├── 是什么：Spring 官方 AI 工程框架，"AI 界的 Spring Data"
├── 特性：可移植 API · 多能力 · ChatClient · Advisor · 结构化输出 · 工具 · RAG · MCP · 可观测
├── 原理
│   ├── 三层架构：ChatClient → ChatModel（内部含工具循环）→ RestClient
│   ├── 自动配置：Starter → AutoConfiguration → 属性绑定(.options 嵌套) → Builder
│   ├── 多模型启停：spring.ai.model.chat 选择器 + chat.enabled 双机制
│   ├── Advisor 责任链：adviseCall + chain.nextCall（双向改写，与 2.0 同构）
│   ├── 工具调用 1.x：内部循环默认 + ToolCallAdvisor 可选接管（无限额防护）
│   └── 升级 2.0：属性扁平化 · defaultOptions→options · 循环外置 ToolCallingAdvisor
└── 选型 vs LangChain4j
    ├── Spring AI：集成深、企业特性全、学习曲线陡
    ├── LangChain4j：AiServices 上手快、零框架绑定
    └── 决策：技术栈是不是 Spring？Boot 是 3.5 还是 4.x？
```

## 自测题

**基础（定义与特性）**

1. 用一句话说明 Spring AI 解决的核心问题是什么？
2. Spring AI 之于 Spring Data 的类比关系是什么？
3. 列出 ChatClient、Advisor、ChatMemory 各自的职责。
4. ChatClient 与 ChatModel 的选择标准是什么？

**原理**

5. 描述 `chatClient.prompt().user(x).call().content()` 从你的代码到模型服务再返回的完整链路（至少 5 步）。
6. 同步调用与流式调用在传输层的本质区别是什么？
7. 结构化输出为什么不是"模型天然支持"？框架在其中做了哪三件事？
8. 自定义一个"禁止输出敏感词"的 Advisor，`adviseCall` 中应该在 `chain.nextCall` 的前面还是后面做检查？为什么？
9. 1.x 的工具执行循环默认发生在哪里？如何只下发定义、不自动执行？
10. `MessageChatMemoryAdvisor` 与 `ToolCallAdvisor` 组合时，官方建议的顺序关系是什么？为什么？

**对比与升级**

11. Spring AI 与 LangChain4j 在设计哲学上的根本差异是什么？
12. 为什么说两者"没有能力代差"？举例说明。
13. 1.x 升级 2.0，配置文件里哪两类写法必须最先检查？（提示：§14 表格第 1、3 条）
14. `spring.ai.model.chat` 在 1.1.8 与 2.0 的语义有何细微差别？（提示：matchIfMissing）
15. 并排写：分别用 Spring AI 与 LangChain4j 实现"带 system 人设的问答"，并指出心智模型差异。
