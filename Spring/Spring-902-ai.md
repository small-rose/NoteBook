---
layout: default
title: Spring AI 2.X 理论
parent: Spring
grand_parent: Java
nav_order: 92
---

# Spring AI 2.X 理论
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 01 · 理论篇：定义特性原理与选型对比

> 阶段：入门 → 精通
> 本篇目标：搞清楚 Spring AI **是什么**（定义与特性）、**怎么跑起来的**（架构与原理）、**该不该用它**（vs LangChain4j 选型）。
> 技术基线：Spring AI 2.0.1 · Spring Boot 4.1.x · JDK 17+

---

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

## 2. 演进历程与版本线

| 时间 | 里程碑 |
|------|--------|
| 2024 年 | Spring 官方立项，陆续发布 1.0.0-M 系列里程碑版本 |
| 2025 年 5 月 | **1.0 GA** 正式发布，进入生产可用（ChatClient / Advisor / RAG / Tool Calling 全套能力定型） |
| 2025 年 | 1.1.x 持续演进（当前维护版 1.1.8），配套 Spring Boot 3.5.x |
| 2026 年 | **2.0 发布**（当前稳定版 **2.0.1**），配套 **Spring Boot 4.0.x / 4.1.x**，工具调用架构重构、配置扁平化、MCP/结构化输出能力大幅增强 |

当前版本线（截至本教程写作时）：

| 版本线 | 最新稳定版 | 配套 Spring Boot | 说明 |
|--------|-----------|------------------|------|
| **2.0.x** | **2.0.1** | 4.0.x / 4.1.x | **本教程基线**，新特性只在这里 |
| 1.1.x | 1.1.8 | 3.5.x | 维护线，适合存量 Boot 3.5 项目 |
| 1.0.x | 1.0.9 | 3.4.x / 3.5.x | 维护线 |
| 2.1.x | 2.1.0-M1（里程碑） | — | 预览 |

> ⚠️ **本教程全部代码基于 Spring AI 2.0.1 + Spring Boot 4.1.1。** 若你维护的是 1.x 老项目，
> 请重点阅读第二部分末尾的「1.x → 2.0 迁移要点速查表」。

## 3. 核心特性全表

| # | 特性 | 一句话说明 | 解决什么问题 |
|---|------|-----------|-------------|
| 1 | **可移植 API** | 同一套 `ChatClient`/`EmbeddingModel` 接口，20+ 供应商多实现 | 换模型不改业务代码 |
| 2 | **多能力覆盖** | 对话补全、嵌入向量、文生图、音频转写、语音合成、内容审核 | 一个框架接全 AI 能力 |
| 3 | **同步 + 流式** | 每种模型调用都有 `call()` 与 `stream()` 双形态 | 打字机效果、长文本生成 |
| 4 | **ChatClient 流式 API** | 语法风格与 `RestClient`/`WebClient` 一脉相承 | 链式调用，可读性高 |
| 5 | **Advisor API** | 拦截每次模型调用，注入记忆、RAG、日志、敏感词过滤 | 横切能力不侵入业务 |
| 6 | **结构化输出** | AI 返回的 JSON 自动映射成 Java POJO（含 Schema 校验与自纠） | 告别手写 JSON 解析 |
| 7 | **Tool Calling** | 用 `@Tool` 注解把 Java 方法注册给模型调用 | AI 操作数据库/查接口/发工单 |
| 8 | **向量库 + RAG** | 20+ 向量数据库统一 API + ETL 摄入管道 | 企业知识库问答 |
| 9 | **对话记忆** | `ChatMemory` 多会话窗口记忆，可落库（JDBC/Cassandra/Neo4j/Redis） | 多轮对话"记得住" |
| 10 | **MCP 协议** | 客户端/服务端 Starter，注解式把 Spring Bean 暴露为 MCP 工具 | 接入 AI 工具生态 |
| 11 | **可观测性** | Micrometer 指标 + Tracing 链路，覆盖模型调用每一步 | 生产排障与成本核算 |
| 12 | **评估工具** | 相关性评估、事实核查 Evaluator | 防幻觉、测 RAG 质量 |
| 13 | **Boot 自动配置** | 所有模型/向量库都有 Starter，start.spring.io 勾选即用 | 零样板接入 |

## 4. Spring AI 2.0 新增能力（相对 1.x）

| 能力 | 说明 |
|------|------|
| **ToolCallingAdvisor** | 工具执行循环从 `ChatModel` 内部抽离，由自动注册的 Advisor 统一承担（详见第二部分 §13） |
| **配置属性扁平化** | `.options` 段全部移除，如 `spring.ai.deepseek.chat.model`（1.x 为 `.chat.options.model`） |
| **多模型启停属性** | 新增顶层 `spring.ai.model.chat=<provider>`，一个应用可同时装配多个供应商模型 |
| **Options 严格不可变** | `copy()` 废弃 → 用 `mutate()`；`getDefaultOptions()` → `getOptions()` |
| **Tool 调用限额** | 单工具每轮默认上限 40 次、总计 150 次，防模型失控死循环 |
| **Tool Search Advisor** | 按关键词/语义检索，每次只把最相关的工具定义发给模型（`spring-ai-starter-tool-search-advisor`） |
| **结构化输出增强** | Schema 校验 + 自动重试自纠、供应商原生 Structured Output 支持 |
| **MCP 增强** | 注解式 MCP 服务端/客户端、Streamable-HTTP、Security 支持 |
| **开发期服务** | Docker Compose/Testcontainers 一键拉起向量库等依赖（Development-time Services） |
| **DeepSeek 思考模式** | `Thinking.ENABLED/DISABLED`、`ReasoningEffort`、思维链 `reasoningContent` 获取（见 02 篇案例一） |

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

配置（2.0 扁平化属性）：

```yaml
spring:
  ai:
    deepseek:
      api-key: ${DEEPSEEK_API_KEY}
      chat:
        model: deepseek-v4-flash
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
| 聊天机器人 / RAG / Agent | **ChatClient** | Advisor 一行接入 |
| 需要完全控制 Prompt 组装 | ChatModel | 手动拼 `Prompt`/`Message` |
| 框架级开发、性能基准测试 | ChatModel | 少一层抽象 |
| 获取原始 `ChatResponse` 元数据 | 两者皆可 | ChatClient 用 `.chatClientResponse()` |

> **经验法则：拿不准就用 ChatClient。** 只有当你明确知道 ChatClient 满足不了时，再下沉到 ChatModel。
> 反向依赖是允许的：`ChatClient.builder(chatModel)` —— ChatClient 建立在 ChatModel 之上。

### 第一部分小结

- Spring AI = Spring 官方 AI 工程框架，核心价值是**可移植抽象**（一套接口换遍模型）。
- 三大支柱：**ChatClient（调用）+ Advisor（增强）+ 自动配置（装配）**。
- 2.0 是当前主线：Boot 4.x、配置扁平化、工具调用架构重构。
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
│           横向：Advisor 链（CallAdvisor / StreamAdvisor）    │  ← 记忆、RAG、工具在这里拦截
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
spring.ai.deepseek.*  → DeepSeekProperties ← ③ 属性绑定（@ConfigurationProperties）
        │ Builder 装配
        ▼
DeepSeekApi → DeepSeekChatModel → ChatClient.Builder ← ④ 产出 Bean
```

关键点：

1. **Starter 只是"入场券"**：真正的逻辑在 `spring-ai-autoconfigure-*` 模块，通过
   `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
   注册自动配置类（Boot 4.x 机制不变）。
2. **属性即插即用**：所有 `spring.ai.*` 属性绑定到 `*Properties` 类，再喂给各层 Builder。
3. **多模型启停（2.0 新增）**：顶层属性决定装配哪个供应商——

```yaml
# 同时存在多个供应商时，显式声明启用谁（默认值即各 starter 自身 id）
spring:
  ai:
    model:
      chat: deepseek        # 可选：openai / deepseek / ollama / anthropic / none ...
```

`spring.ai.model.chat=none` 可禁用某模型的对话自动配置（引了依赖但不想自动装配时）。

4. **配置扁平化（2.0 破坏性变更）**：

```yaml
# ❌ 1.x 写法
spring.ai.deepseek.chat.options.model=deepseek-chat
# ✅ 2.0 写法
spring.ai.deepseek.chat.model=deepseek-chat
```

5. **注入即所得**：任何自动配置好的组件都能直接构造注入——`ChatClient.Builder`、
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
[ToolCallingAdvisor]        附上 @Tool 工具定义（本轮暂不执行）
  │  chain.nextCall(request)
  ▼
③ ChatModel.call(Prompt)    ← DeepSeekChatModel 实现
  │  合并默认 Options（模型名、温度…，可用请求级 options 覆盖）
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
- **工具调用是一段循环**：若模型返回 `hasToolCalls()`，`ToolCallingAdvisor` 会执行 Java 方法、
  把结果回填 Prompt、再次请求模型，直到产出最终回答（详见 §13）。
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
⑤ Jackson 反序列化 → POJO            （失败时 2.0 支持 Schema 校验 + 自动重试自纠）
```

调用方式（一行）：

```java
public record Ticket(String title, String priority, String ticketNo) {}

Ticket ticket = chatClient.prompt()
        .user("帮我把这段投诉转成工单：快递丢件了，单号 SF123，很急")
        .call()
        .entity(Ticket.class);        // ← 结构化输出入口
```

2.0 还支持**供应商原生 Structured Output**（模型端直接按约束解码，而非提示词约束），
以及校验失败后的自动纠错重试——细节见 02 篇实战。

## 12. Advisor 责任链机制

Advisor 是 Spring AI 最有"Spring 味"的设计——本质就是 **AOP/Filter 链**在 AI 场景的翻版。

### 12.1 核心接口（2.0 实际签名）

```java
public interface Advisor extends Ordered {
    String getName();                               // 链中标识
    // getOrder() 来自 Ordered：值越小越先执行
}

// 同步场景
public interface CallAdvisor extends Advisor {
    ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain);
}

// 流式场景
public interface StreamAdvisor extends Advisor {
    Flux<ChatClientResponse> adviseStream(ChatClientRequest request, StreamAdvisorChain chain);
}

// 责任链：调用 chain.nextCall(request) 进入下一个 Advisor
public interface CallAdvisorChain extends AdvisorChain {
    ChatClientResponse nextCall(ChatClientRequest request);
    List<CallAdvisor> getCallAdvisors();
}
```

最后一个 Advisor 由框架自动追加，负责真正调用 LLM。

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

### 12.3 内置 Advisor 与执行顺序（2.0）

| Advisor | 职责 | 默认顺序（HIGHEST_PRECEDENCE = Integer.MIN_VALUE） |
|---------|------|--------------------------------------------------|
| `MessageChatMemoryAdvisor` | 注入/写回对话记忆 | **+ 200** |
| `ToolCallingAdvisor` | 工具定义下发 + 工具执行循环 | **+ 300** |
| `QuestionAnswerAdvisor` | RAG 检索片段注入 | 按注册顺序 |
| `ToolSearchToolCallingAdvisor` | 替代上者，按相关性筛选工具 | 需显式启用 |

2.0 的一个关键调整：**记忆 Advisor（+200）排在工具循环（+300）之外**——
`ToolCallingAdvisor` 在循环内部自行管理会话历史，只把最终的用户/助手消息交给记忆存储
（多数存储实现不支持中间的 tool-call 消息类型）。需要"循环内记忆"时可自行调序并
`disableInternalConversationHistory()`，一般场景用默认值即可。

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

## 13. 工具调用架构：2.0 的关键演进

### 13.1 1.x 的做法（已废弃）

1.x 中每个 `ChatModel` 实现**内部**都藏着一个工具执行循环：

```
ChatModel.call(prompt with tools)
   → 模型返回 tool_calls
   → ChatModel 内部执行 Java 方法、回填结果、再次请求      ← 循环藏在实现里
   → 直到模型给出最终回答才返回
```

问题：循环逻辑复制在 20+ 个供应商实现里、行为不一致、上层难以干预、
`internalToolExecutionEnabled` 等开关散落各处。

### 13.2 2.0 的做法：循环外置到 Advisor

```
ChatClient.prompt().tools(new WeatherTools()).call()
   │
   ▼
[ToolCallingAdvisor]  ← 自动注册（spring.ai.chat.client.tool-calling.enabled=true）
   │  下发工具定义 → 模型返回 tool_calls
   │  → ToolCallingManager 反射执行 @Tool 方法
   │  → 结果回填 Prompt → 再次调用模型            ← 循环收敛到唯一入口
   │  → 直到无 tool_calls
   ▼
最终回答返回给调用方
```

**`ChatModel.call(prompt)` 本身不再执行任何工具**——直接返回模型的原始响应。
应用侧写法反而更简单：

```java
// 2.0：无需任何开关，注册工具即可（循环由 ToolCallingAdvisor 承担）
String answer = ChatClient.create(chatModel)
        .prompt("北京现在天气怎么样？")
        .tools(new WeatherTools())          // ← 1.x 叫 .functions(...)
        .call()
        .content();
```

### 13.3 2.0 工具调用的安全收口

| 机制 | 默认值 | 配置 |
|------|--------|------|
| 单工具每轮调用上限 | 40 次 | `spring.ai.tools.limits.max-calls-per-tool` |
| 单次请求工具调用总上限 | 150 次 | `spring.ai.tools.limits.max-total-tool-calls` |
| 未挂载工具的解析回退 | **关闭**（只能执行本次请求显式挂载的工具） | `spring.ai.tools.resolution.fallback.enabled=true` 可恢复旧行为 |
| 全局停用自动工具执行 | 工具仍下发给模型但不自动执行 | `spring.ai.chat.client.tool-calling.enabled=false` |

> 这四项是 2.0 的"生产级加固"：既防模型失控刷接口，也防"应用里随便一个 Bean 都能被模型调到"的越权风险。

## 14. 1.x → 2.0 迁移要点速查表

| # | 变更 | 1.x 写法 | 2.0 写法 |
|---|------|---------|----------|
| 1 | 配置属性扁平化 | `spring.ai.openai.chat.options.model` | `spring.ai.openai.chat.model` |
| 2 | 工具注册 API | `.functions(...)` / `toolNames(...)` / `FunctionCallback` | **`.tools(...)` / `@Tool` / `ToolCallback`**（Function API 全删） |
| 3 | 工具执行循环 | `ChatModel` 内部自动执行（`internalToolExecutionEnabled` 开关） | 循环移至 **`ToolCallingAdvisor`**；`ChatModel` 只返回原始响应 |
| 4 | 顾问式工具类 | `ToolCallAdvisor`（手动 `.advisor(...)` 才生效） | 更名 **`ToolCallingAdvisor`** 且**自动注册**（`spring.ai.chat.client.tool-calling.enabled=false` 可关） |
| 5 | ChatModel Builder | `.builder().…​.defaultOptions(options)` | `.options(options)`（升级时全局搜 `.defaultOptions(`） |
| 6 | Advisor 类名 | `CallAroundAdvisor` 等旧称 | `CallAdvisor` / `StreamAdvisor`，方法名 `adviseCall` / `adviseStream` |
| 7 | Options 可变性 | `options.copy()` 后修改 | 严格不可变，用 `options.mutate().…​.build()` |
| 8 | 默认选项获取 | `chatModel.getDefaultOptions()` | `chatModel.getOptions()` |
| 9 | 记忆 Advisor 顺序 | 记忆在工具循环内 | 记忆默认在循环**外**（+200 vs 工具 +300） |
| 10 | RAG Advisor 模块 | `spring-ai-advisors-vector-store` | `spring-ai-vector-store-advisor`（`QuestionAnswerAdvisor` 类名不变） |
| 11 | Ollama 思考开关 | 程序化 `enableThinking()`（Options 绑定名 `options.think-option`） | `spring.ai.ollama.chat.think` |
| 12 | 多模型启停 | `spring.ai.<provider>.chat.enabled`（1.1.8 的 ollama 已移除该属性；`model.chat` 选择器 1.1.8 已有） | 统一顶层 `spring.ai.model.chat=<provider>/none`，`chat.enabled` 全部移除 |
| 13 | Builder `.N(...)` | `OpenAiChatOptions.builder().N(1)` | `.n(1)` |

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
- 自动配置四步走：Starter → AutoConfiguration → 属性绑定 → Builder 装配 Bean。
- Advisor = AI 场景的责任链/AOP：`adviseCall` + `chain.nextCall`，双向改写请求与响应。
- 2.0 最大结构变化：**工具执行循环从 ChatModel 内部上移到 ToolCallingAdvisor**，并加了限额与越权防护。
- 迁移 1.x 时优先核对 §14 表格中的 13 条变更。

---

# 第三部分 · Spring AI vs LangChain4j（精通）

> 本节对比基于 **Spring AI 2.0.1** 与 **LangChain4j 1.22.0**（均为写作时最新稳定版）。

## 16. 两者定位与背景

| | Spring AI | LangChain4j |
|---|-----------|-------------|
| 出身 | **Spring 官方项目**（spring-projects/spring-ai） | 社区项目，Python **LangChain 的 Java 对齐实现** |
| 哲学 | 把可移植性、模块化、POJO、依赖注入等 **Spring 生态设计原则**搬到 AI 领域 | 把 LangChain 的"链式编排 + 丰富集成"思想带进 Java |
| 首个 GA | 1.0（2025 年 5 月） | 1.0（2025 年） |
| 当前稳定版 | **2.0.1**（配套 Boot 4.x） | **1.22.0** |
| 许可 | Apache 2.0 | Apache 2.0 |
| 一句话概括 | "**AI 界的 Spring Data**"，服务 Spring 全家桶用户 | "**Java 界的 LangChain**"，服务所有 Java 用户 |

两者如今都是成熟的生产级框架——**2025 年之后"选哪个都能干活"，选型差异主要在团队技术栈与集成深度**。

## 17. 多维对比总表

| 维度 | Spring AI 2.0.1 | LangChain4j 1.22.0 |
|------|-----------------|---------------------|
| **核心调用 API** | `ChatClient` 流式 DSL（`prompt().user().call()`） | **`AiServices` 声明式接口**（定义 Java 接口 + 注解即得实现） |
| **底层抽象** | `ChatModel` / `EmbeddingModel` / `ImageClient`… | `ChatModel` / `EmbeddingModel` / `ModerationModel`… |
| **编程模型** | 依赖注入、POJO、Builder，深度遵循 Spring 习惯 | 普通 Java 库，Builder/工厂为主，不依赖 Spring |
| **装配方式** | **Boot Starter 自动配置为主**（start.spring.io 勾选即用），也可手工 Builder | 核心是独立库；另有官方 `langchain4j-spring-boot-starter` 可选 |
| **模型供应商** | 20+ 官方模块（OpenAI/Anthropic/Google/Amazon/Azure/**DeepSeek**/Ollama/Groq/Mistral/MiniMax/NVIDIA/Perplexity…） | 同样覆盖主流（OpenAI/Anthropic/Gemini/Azure/Ollama/Mistral/Cohere/HuggingFace…）+ 大量社区模块 |
| **向量数据库** | **20+ 官方支持**（PgVector/Cassandra/Milvus/Qdrant/Chroma/Redis/Weaviate/Elasticsearch/Neo4j/Oracle…）+ 类 SQL 元数据过滤 API | 主流基本覆盖（PgVector/Redis/Milvus/Qdrant/Chroma/Pinecone/Weaviate/Elasticsearch/MongoDB/内存版…） |
| **RAG 能力** | ETL 摄入管道 + `VectorStore` + `QuestionAnswerAdvisor`/`RetrievalAugmentationAdvisor`（模块化 RAG） | `ContentRetriever`/`ContentAggregator`/查询转换器等 RAG 组件链，组合灵活 |
| **工具调用** | `@Tool` + **`ToolCallingAdvisor` 自动循环**（2.0 外置，带限额与越权防护） | `@Tool`（`dev.langchain4j.agent.tool.Tool`）+ AI Services 内自动执行 |
| **对话记忆** | `ChatMemory` + Advisor，可落 JDBC/Cassandra/Neo4j/Redis | `ChatMemory`（`MessageWindowChatMemory` 等）+ 持久化 `ChatMemoryStore` |
| **MCP 支持** | **一等公民**：client/server 双 Starter、注解式、Streamable-HTTP、Security | 有 `langchain4j-mcp` 模块，可用 |
| **可观测性** | Micrometer/OTel **原生集成**，Actuator/Trace 全家桶协同 | 提供 `ChatModelListener`/观测监听，接入生态需自行桥接 |
| **企业特性** | Spring Security（MCP OAuth）、Spring Authorization Server、Testcontainers、Docker Compose 开发期服务 | 需自行组合其他库 |
| **学习曲线** | 中偏高：要理解 ChatClient/Advisor/自动配置等抽象 | **偏低**：AiServices 声明式接口直观，"定义接口→拿到实现" |
| **非 Spring 项目** | 能用（手工 Builder），但体验非最优 | **原生主场** |
| **Agent/工作流** | 官方文档提供 5 种 Agent 模式（链式/并行/路由/编排-执行/评估-优化）+ MCP Agent 孵化项目 | AI Services 支持带工具的自主循环，Agent 高阶编排靠社区与自建 |

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
| 企业级特性齐全：MCP 安全、可观测、Testcontainers、开发期服务 | 项目年轻，2.0 有破坏性变更，老资料容易过时 |
| 向量库/模型官方模块数量领先 | 部分高级用法文档密度仍不及 Python 生态 |
| 2.0 工具调用安全加固（限额、越权收口）适合生产 | — |

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
├── 是，且企业级要求高（安全、审计、监控、多供应商治理）
│        → ✅ Spring AI（全家桶协同是它最大杀器）
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
└── 需要 MCP 一等公民支持 / Spring Security 管控 AI 进出网关
         → ✅ Spring AI
```

补充判断依据：

| 你在意的点 | 选择 |
|-----------|------|
| "以后可能从 DeepSeek 换到本地 vLLM，要只改配置" | 都支持，Spring AI 的自动配置切换更顺 |
| "我想 10 分钟出原型" | LangChain4j（AiServices） |
| "RAG 要接 PgVector + 权限过滤 + 链路追踪" | Spring AI |
| "项目不引 Spring，还要接 5 个供应商" | LangChain4j |
| "AI 输出要进 Spring Security 审计日志" | Spring AI |

### 第三部分小结

- 两者定位同源思想不同族：**Spring AI = Spring 生态原则在 AI 领域的延伸**；
  **LangChain4j = LangChain 思想的 Java 化**。
- 能力面高度重叠（模型/向量库/RAG/工具/记忆/MCP 都有），差异在**集成深度 vs 上手速度**。
- 选型第一问永远是：**你的技术栈是不是 Spring？** 是 → 优先 Spring AI；否 → 优先 LangChain4j。

---

## 全篇知识图谱

```
Spring AI
├── 是什么：Spring 官方 AI 工程框架，"AI 界的 Spring Data"
├── 特性：可移植 API · 多能力 · ChatClient · Advisor · 结构化输出 · 工具 · RAG · MCP · 可观测
├── 原理
│   ├── 三层架构：ChatClient → ChatModel → RestClient
│   ├── 自动配置：Starter → AutoConfiguration → 属性绑定 → Builder
│   ├── Advisor 责任链：adviseCall + chain.nextCall（双向改写）
│   ├── 工具调用 2.0：循环外置 ToolCallingAdvisor + 限额/越权防护
│   └── 1.x→2.0：配置扁平化 · functions→tools · Options 不可变
└── 选型 vs LangChain4j
    ├── Spring AI：集成深、企业特性全、学习曲线陡
    ├── LangChain4j：AiServices 上手快、零框架绑定
    └── 决策第一问：技术栈是不是 Spring？
```

## 自测题

**基础（定义与特性）**

1. 用一句话说明 Spring AI 解决的核心问题是什么？

   **答**：用统一的 Spring 式抽象（ChatClient/ChatModel/Advisor + Starter 自动配置 + BOM 版本治理）屏蔽各家模型供应商的 API 差异，让「换模型/换供应商」像换数据库驱动一样只动配置、不动代码。

2. Spring AI 之于 Spring Data 的类比关系是什么？

   **答**：Spring Data 用 Repository 抽象统一访问各家数据库（换库只换依赖与方言实现），Spring AI 用 ChatClient/Model 抽象统一访问各家 LLM——各供应商 Starter 相当于「数据库驱动」，自动配置把对应实现注入容器，业务代码零感知。

3. 列出 ChatClient、Advisor、ChatMemory 各自的职责。

   **答**：ChatClient——面向业务的流式 DSL，组装 Prompt（system/user、Advisor 参数、工具）并发起调用；Advisor——请求/响应双向拦截的责任链插件（记忆、RAG、日志、限流都是它）；ChatMemory——会话历史的存取与窗口管理，按 conversationId 分桶隔离多会话。

4. ChatClient 与 ChatModel 的选择标准是什么？

   **答**：业务开发默认用 ChatClient（DSL + Advisor 生态覆盖绝大多数场景）；只有需要操作底层响应（原始 ChatResponse、元数据、多模态细粒度控制）或开发 Advisor/集成层时才直接用 ChatModel。类比：ChatModel 像 JdbcTemplate，ChatClient 像建立其上的现代门面。

**原理**

5. 描述 `chatClient.prompt().user(x).call().content()` 从你的代码到模型服务再返回的完整链路（至少 5 步）。

   **答**：①DSL 组装 Prompt（system/user/参数）→ ②Advisor 链前置处理（记忆 Advisor 拼历史 → RAG Advisor 检索注入片段 → 自定义改写）→ ③ChatModel 用 Options 拼供应商 HTTP 请求（RestClient/WebClient）→ ④TLS 发往模型服务推理 → ⑤响应反序列化为 ChatResponse → ⑥Advisor 后置处理（写回记忆等）→ ⑦`.content()` 返回文本。2.0 补充：工具场景下 ChatModel 只产出原始响应，执行循环由 ToolCallingAdvisor 完成。

6. 同步调用与流式调用在传输层的本质区别是什么？

   **答**：同步是一次性请求-响应（普通 HTTP，整体写回完整 ChatResponse）；流式基于 SSE 长连接（`text/event-stream`），服务端把生成的 token 分块持续推送，框架聚合为 Flux/回调。差别体现在首 token 延迟（TTFT）与交互体验，而非模型能力。

7. 结构化输出为什么不是"模型天然支持"？框架在其中做了哪三件事？

   **答**：模型原生输出的是自然语言文本，不承诺 JSON 结构。框架（`BeanOutputConverter`）做三件事：①生成 Schema 格式指令注入 Prompt（告诉模型按什么结构输出）；②调用后取回 JSON 文本；③Jackson 反序列化为 POJO——2.0 还会在解析失败时把错误回喂模型自动重试自纠。

8. 自定义一个"禁止输出敏感词"的 Advisor，`adviseCall` 中应该在 `chain.nextCall` 的前面还是后面做检查？为什么？

   **答**：放在 `chain.nextCall` **之后**——敏感词检查的对象是模型输出，请求阶段内容尚未生成；拿到 response 后校验，不通过可抛异常或改写。职责区分：`nextCall` 之前只能拦截/改写「输入」，之后才能审计/过滤「输出」；若输入也要审，就前后各放一个检查点。

9. 2.0 把工具执行循环从 ChatModel 移到 ToolCallingAdvisor，带来了哪三点好处？

   **答**：①**关注点分离**：ChatModel 退回「单次模型调用」本职，工具循环成为可插拔的 `ToolCallingAdvisor`；②**可治理**：循环外置后才能统一施加限额（单工具 40 次/轮、单请求 150 次）、全局开关（`tool-calling.enabled`）与审计埋点；③**安全**：默认只执行显式挂载的工具、拒绝未挂载工具的解析回退，落实最小权限，防提示词注入把容器里任意 Bean 变成攻击面。

10. 1.x 项目升级 2.0，配置文件里哪类写法必须最先检查？

**答**：**配置属性写法**——先全局搜 `.chat.options.`：1.x 的 `spring.ai.deepseek.chat.options.model` 在 2.0 必须改为扁平的 `spring.ai.deepseek.chat.model`（全部供应商同理）。其次搜 Java 侧 `.defaultOptions(` → `.options(`。属性扁平化是 2.0 破坏面最广的变更（§14 第 1 条）。

**对比与选型**

11. Spring AI 与 LangChain4j 在设计哲学上的根本差异是什么？

**答**：Spring AI 走 Spring 工程化路线——自动配置、依赖注入，与 Boot/Security/Micrometer 深度协同，抽象全面、面向企业治理；LangChain4j 走开发者直觉路线——`AiServices` 定义接口 + 注解即得实现，轻量、零框架绑定、原型极快。一个是「生态位优先」，一个是「上手速度优先」。

12. 为什么说两者"没有能力代差"？举例说明。

**答**：两者都完整覆盖对话/流式/嵌入/RAG/工具/多模态。例：RAG 两边同构——Spring AI 是 `VectorStore` + `QuestionAnswerAdvisor`，LC4j 是 `EmbeddingStore` + `ContentRetriever`，都是「嵌入 → 向量检索 → 注入上下文」；工具两边都有 @Tool 式注解与自主执行循环。差异只在组合方式：Advisor 纵向叠加 vs AiServices 横向挂载——能力清单几乎一一对应。

13. 什么情况下应该毫不犹豫选 LangChain4j？

**答**：三种场景：①**非 Spring 项目**（纯 Java SE/Quarkus/Micronaut）——Spring AI 的自动配置生态无从发挥；②**要 10 分钟出原型**——AiServices 接口式开发最快；③**团队已深度使用 LC4j** 且无 Spring 治理诉求。反向：Spring Boot 存量项目、要鉴权/审计/可观测协同的，选 Spring AI。

14. Spring AI 的哪些特性对"金融/政企类项目"最有吸引力？（举三个）

**答**：①**私有化与供应链可信**——官方开源、Maven Central 分发、BOM 统一版本治理，适配内网与等保环境；②**与 Spring 安全/可观测体系天然协同**——Spring Security 鉴权、Micrometer 指标与追踪直接接入现有体系；③**数据可控**——本地模型（Ollama/vLLM）接入 + 工具最小权限治理 + RAG 企业知识库，敏感数据不出域。

15. 并排写：分别用两种框架实现"带 system 人设的问答"，并指出心智模型差异。

**答**：正文 §18 已并排给出——Spring AI：注入 `ChatClient.Builder`，`chatClient.prompt().system("人设").user(q).call().content()`；LangChain4j：定义 `interface Assistant { @SystemMessage("人设") String answer(String q); }`，再 `AiServices.builder(Assistant.class).chatModel(model).build()` 拿实现。心智差异：Spring AI 把「一次调用」建模为可拦截的对象流水线（Prompt/Advisor 显式组装，控制粒度细）；LC4j 建模为一个 Java 接口方法（注解即配置，上手极快）——前者利于复杂编排，后者利于快速表达。

---

