---
layout: default
title: Spring AI 1.X 应用
parent: Spring
grand_parent: Java
nav_order: 91
---

# Spring AI 2.X 应用
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# 02 · 实战篇：场景应用与两大接入案例

> 阶段：进阶 → 实战 → 精通
> 本篇目标：先用五大核心能力打通「场景 → 代码」的映射，再交付**两个完整可运行工程**：
> 案例一接入 DeepSeek（云端），案例二接入自定义本地模型（Ollama/vLLM，含多模型路由）。
> 技术基线：Spring AI 1.1.8 · Spring Boot 3.5.x · JDK 17+（本篇所有代码均基于 1.1.8 已校验 API）

---

# 第一部分 · 应用场景与核心能力实战（进阶）

## 1. 六大应用场景总览

| # | 场景 | 业务描述 | 所需 Spring AI 能力 | 本篇落点 |
|---|------|---------|--------------------|---------|
| 1 | **智能客服 / 对话助手** | 7×24 应答，记得用户上一句说了什么 | ChatClient + ChatMemory + Advisor | §3、案例一 |
| 2 | **企业知识库 RAG** | 基于公司文档回答，"让 AI 只按我们的资料答" | VectorStore + QuestionAnswerAdvisor + EmbeddingModel | §6 |
| 3 | **内容生成与审核** | 写文案/邮件，过滤违规输出 | 同步/流式生成 + 自定义 Advisor 拦截 | 案例一（流式）、§5（拦截思路） |
| 4 | **数据结构化提取** | 把自由文本变 JSON/POJO 入库 | 结构化输出 `.entity()` | §4、案例一 |
| 5 | **Tool Calling 业务自动化** | AI 查订单、退票、发工单 | `@Tool`（ChatModel 内部循环 / ToolCallAdvisor） | §5 |
| 6 | **多模态理解** | 图片描述/截图理解/文档 OCR 辅助 | Media 附件 + 多模态模型 | 案例二（视觉模型） |

> 选型口诀：**先判断场景 → 对照上表找到能力 → 直接翻对应小节的代码**。

## 2. 能力① Prompt：人设、模板与参数

### 2.1 System 人设定基调

```java
String answer = chatClient.prompt()
        .system("你是一名资深 Java 工程师，回答控制在 100 字以内，不解释背景知识")
        .user(question)
        .call()
        .content();
```

SystemMessage 决定"AI 是谁、遵守什么规则"，UserMessage 是用户输入。**同一问题，人设不同答案完全不同**——这是提示词工程的最小单位。

### 2.2 提示词模板（占位符传参）

模板写在代码里、参数运行时传入，避免字符串拼接：

```java
String answer = chatClient.prompt()
        .system(s -> s.text("你是{company}的客服助手，说话风格：{style}")
                      .param("company", "观世伞科技")
                      .param("style", "简洁专业，不超过 100 字"))
        .user(u -> u.text("问题：{question}")
                    .param("question", question))
        .call()
        .content();
```

- 默认模板渲染器是 `StTemplateRenderer`，占位符语法为 `{name}`。
- 内部由 `PromptTemplate` 完成替换——模板也可以指向外部 `.st` 文件（进阶用法）。

### 2.3 运行时覆盖模型参数

```java
chatClient.prompt()
        .user("写一首关于杭州的诗")
        .options(DeepSeekChatOptions.builder().temperature(0.9).build()) // 请求级覆盖
        .call().content();
```

> 原则：**全局默认放 yml，单次请求特调放 `.options()`**。
> 1.x 修改已有 Options 实例用 `options.copy()` 拷贝后改（2.0 改为 `mutate()`，见 01 篇 §14）。

## 3. 能力② 多轮对话记忆

AI 模型本身无状态，"记得上一句"靠框架把历史消息重新拼进 Prompt：

```java
@Service
public class ChatService {

    private final ChatClient chatClient;

    public ChatService(ChatClient.Builder builder) {
        ChatMemory chatMemory = MessageWindowChatMemory.builder()
                .maxMessages(20)                 // 滑动窗口：只保留最近 20 条
                .build();

        this.chatClient = builder
                .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build())
                .build();
    }

    public String chat(String conversationId, String userText) {
        return chatClient.prompt()
                .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, conversationId)) // 会话隔离
                .user(userText)
                .call()
                .content();
    }
}
```

```java
// 第一句
chatService.chat("conv-1001", "我叫小明，是一名 Java 工程师");
// 第二句 —— AI 记得：
chatService.chat("conv-1001", "我叫什么？做什么工作？");   // → "你叫小明，是 Java 工程师"
chatService.chat("conv-2002", "我叫什么？");              // → 不知道（会话隔离生效）
```

要点：

| 要点 | 说明 |
|------|------|
| 窗口淘汰 | `maxMessages(20)` 超出后丢最旧消息 |
| 会话隔离 | 必须传 `ChatMemory.CONVERSATION_ID`，否则所有用户共享一段记忆 |
| 执行顺序 | `MessageChatMemoryAdvisor` 默认顺序 = `MIN_VALUE + 1000`（01 篇 §12.3），可用 `.order()` 调整 |
| 持久化 | 默认内存实现（`MessageWindowChatMemory`）重启即失；生产用 JDBC/Redis/Cassandra 的 `ChatMemoryRepository` 挂到 `chatMemoryRepository(...)` |

## 4. 能力③ 结构化输出：文本 → POJO

```java
public record FlightTicket(String passenger, String flightNo, String date, String seat) {}

FlightTicket ticket = chatClient.prompt()
        .user("解析这段话为机票信息：张三，CA1234 航班，10 月 20 日，座位 32A")
        .call()
        .entity(FlightTicket.class);          // ← 自动 JSON Schema 指令 + 反序列化
```

泛型集合用 `ParameterizedTypeReference`：

```java
List<FlightTicket> tickets = chatClient.prompt()
        .user("从下面文本解析出所有机票：李四 CA1234 10-20 32A；王五 MU5678 10-21 15C")
        .call()
        .entity(new ParameterizedTypeReference<List<FlightTicket>>() {});
```

原理回顾（01 篇 §11）：框架把目标类型的 JSON Schema 写进 Prompt → 模型回 JSON → Jackson 转 POJO。
1.x 阶段建议在 system 提示词里补一句字段约束（如"缺少的填 unknown"），提升解析成功率。

## 5. 能力④ Tool Calling：让 AI 调你的 Java 方法

```java
@Component
public class OrderTools {

    @Tool(description = "根据订单号查询订单的物流状态")
    public String queryOrderStatus(
            @ToolParam(description = "订单号，例如 SF1234567890") String orderNo) {
        // 真实项目里查数据库 / 调内部接口
        return "订单 " + orderNo + " 已到达杭州转运中心，预计 10 月 16 日送达";
    }

    @Tool(description = "为指定订单发起催派")
    public String urgeDelivery(@ToolParam(description = "订单号") String orderNo) {
        return "已为订单 " + orderNo + " 发起催派，24 小时内专人跟进";
    }
}
```

```java
// 注册工具后提问 —— 模型会自己决定调 queryOrderStatus
String answer = chatClient.prompt()
        .user("帮我查下 SF1234567890 到哪了")
        .tools(new OrderTools())                  // 对象注册
        .call()
        .content();
```

也可以按 Bean 名注册或全局注册：

```java
ChatClient chatClient = ChatClient.builder(chatModel)
        .defaultTools(new OrderTools())           // Builder 级全局注册
        // .defaultToolNames("orderTools")        // 按 Spring Bean 名注册（1.x 保留）
        .build();
```

**1.x 行为须知**（01 篇 §13）：

1. 工具执行循环默认在 **ChatModel 内部**完成——注册工具即可，无需任何开关；
2. `internalToolExecutionEnabled`（挂在 Options 上，默认 true）设为 false 可"只下发定义不执行"；
3. 需要与记忆/审计编排、或 `returnDirect` 时，改用 **`ToolCallAdvisor`** 接管循环（01 篇 §13.2）；
4. 1.x **没有** 2.0 的调用限额防护（`spring.ai.tools.limits.*` 为 2.0 新增）——
   对"模型可能反复调工具"的场景要自己加业务层防护。

## 6. 能力⑤ RAG 最小闭环：让 AI 只按你的资料回答

依赖（示例用内存向量库 + Ollama 嵌入模型，零外部服务；注意 RAG Advisor 模块名是
**`spring-ai-advisors-vector-store`**，2.0 改名为 `spring-ai-vector-store-advisor`）：

```xml
<!-- 提供 EmbeddingModel（Ollama 自动装配） -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
<!-- 提供 VectorStore / SimpleVectorStore -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-vector-store</artifactId>
</dependency>
<!-- 提供 QuestionAnswerAdvisor -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-advisors-vector-store</artifactId>
</dependency>
```

```java
@Configuration
public class RagConfig {

    @Bean
    VectorStore vectorStore(EmbeddingModel embeddingModel) {
        // 内存向量库，生产可换 spring-ai-starter-vector-store-pgvector 等，代码不变
        return SimpleVectorStore.builder(embeddingModel).build();
    }
}
```

```java
@Service
public class KnowledgeBaseService {

    private final VectorStore vectorStore;
    private final ChatClient chatClient;

    public KnowledgeBaseService(VectorStore vectorStore, ChatClient.Builder builder) {
        this.vectorStore = vectorStore;
        this.chatClient = builder
                .defaultAdvisors(QuestionAnswerAdvisor.builder(vectorStore).build())
                .build();
    }

    /** 文档摄入：真实项目用 ETL 管道读 PDF/网页 */
    public void ingest() {
        vectorStore.add(List.of(
                new Document("Spring AI 1.1.x 面向 Spring Boot 3.4.x / 3.5.x，JDK 17+。"),
                new Document("接入 DeepSeek 使用依赖 spring-ai-starter-model-deepseek，配置前缀 spring.ai.deepseek。"),
                new Document("Ollama 默认端口 11434，其 OpenAI 兼容端点为 /v1。")));
    }

    /** 提问：Advisor 自动检索相关片段注入 Prompt */
    public String ask(String question) {
        return chatClient.prompt().user(question).call().content();
    }
}
```

```text
问：Spring AI 1.1.x 需要什么 JDK 版本？
答：需要 JDK 17+（检索到的文档片段被注入了 Prompt）
```

> 嵌入模型默认使用 Ollama 的 `nomic-embed-text`——记得先 `ollama pull nomic-embed-text`。

### 第一部分小结

| 能力 | 一行记忆 | 关键 API |
|------|---------|---------|
| Prompt | 人设定调，模板传参 | `.system()/user()`、`.param()` |
| 记忆 | 历史消息重拼 + 会话隔离 | `MessageWindowChatMemory` + `CONVERSATION_ID` |
| 结构化输出 | JSON → POJO | `.entity(Class)` |
| 工具 | `@Tool` 注册，**ChatModel 内部循环默认执行** | `.tools(...)`、`internalToolExecutionEnabled` |
| RAG | 检索片段注入 Prompt | `SimpleVectorStore` + `QuestionAnswerAdvisor`（advisors-vector-store 模块） |

---

# 第二部分 · 案例一：完整接入 DeepSeek（实战）

> 目标：从零跑通一个「同步 / 流式 / 结构化 / 深度思考」四能力的 DeepSeek 问答服务，附网页聊天界面。
> 工程名：`springai-1x-deepseek-demo`

## 7. 工程与依赖

**创建方式**：https://start.spring.io 选择 Spring Boot **3.5.16**、依赖勾选 **Spring Web**，
其余依赖按下表补齐（也可直接手写完整 pom）。

完整 `pom.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.16</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>springai-1x-deepseek-demo</artifactId>
    <version>1.0.0</version>
    <name>springai-1x-deepseek-demo</name>
    <description>Spring AI 1.1.8 + DeepSeek 完整案例</description>

    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.1.8</spring-ai.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- Spring AI BOM：统一管理所有 spring-ai 模块版本 -->
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- DeepSeek 模型 Starter（自动配置 DeepSeekChatModel / ChatClient.Builder） -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-model-deepseek</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> 依赖关系：`spring-ai-starter-model-deepseek` → `spring-ai-autoconfigure-model-deepseek`
> → `spring-ai-deepseek`（内含 `DeepSeekApi` 底层客户端）。**BOM 不写版本号也不会出现版本漂移**。

## 8. 配置

`src/main/resources/application.yml`（**1.x 嵌套属性**，参数都在 `.options` 段下）：

```yaml
server:
  port: 8080

spring:
  ai:
    # 1.1.8 的供应商选择器（jar 实证：havingValue=deepseek + matchIfMissing=true）
    # 本例 classpath 只有 DeepSeek 一个对话 starter，此行可省略；写上便于日后多供应商共存
    model:
      chat: deepseek
    deepseek:
      api-key: ${DEEPSEEK_API_KEY:sk-your-key}
      base-url: https://api.deepseek.com
      # chat.enabled: true            # 供应商级开关（1.x 仍有；ollama 已移除该属性）
      chat:
        options:                      # ← 1.x 的 .options 段（2.0 起扁平化到 chat 下一级）
          model: deepseek-chat        # deepseek-chat（通用）| deepseek-reasoner（深度思考）
          temperature: 0.7
          max-tokens: 2048
```

API Key 通过环境变量注入（**不要把 Key 提交进仓库**）：

```bash
# macOS / Linux
export DEEPSEEK_API_KEY=sk-xxxxxxxx

# Windows CMD
set DEEPSEEK_API_KEY=sk-xxxxxxxx

# PowerShell
$env:DEEPSEEK_API_KEY="sk-xxxxxxxx"
```

Key 获取：https://platform.deepseek.com 注册 → API Keys 页面创建。

## 9. 代码实现

### 9.1 启动类

```java
package com.example.deepseekdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DeepSeekDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DeepSeekDemoApplication.class, args);
    }
}
```

### 9.2 DTO

```java
package com.example.deepseekdemo.dto;

public record ChatRequest(String message) {}
```

```java
package com.example.deepseekdemo.dto;

/** 结构化输出目标：投诉 → 工单 */
public record Ticket(String title, String priority, String ticketNo, String summary) {}
```

### 9.3 Controller：四个接口全覆盖

```java
package com.example.deepseekdemo.controller;

import com.example.deepseekdemo.dto.ChatRequest;
import com.example.deepseekdemo.dto.Ticket;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.deepseek.DeepSeekChatModel;
import org.springframework.ai.deepseek.DeepSeekChatOptions;
import org.springframework.ai.deepseek.DeepSeekAssistantMessage;
import org.springframework.ai.deepseek.api.DeepSeekApi;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

import java.util.Map;

@RestController
@RequestMapping("/ai")
public class ChatController {

    private static final String SYSTEM_PROMPT =
            "你是一名资深 Java 与 AI 工程顾问，回答简洁准确，不超过 150 字。";

    private final ChatClient chatClient;
    private final DeepSeekChatModel deepSeekChatModel;

    // ChatClient.Builder / DeepSeekChatModel 均由 Starter 自动配置注入
    public ChatController(ChatClient.Builder builder, DeepSeekChatModel deepSeekChatModel) {
        this.chatClient = builder.build();
        this.deepSeekChatModel = deepSeekChatModel;
    }

    /** ① 同步对话 */
    @PostMapping("/chat")
    public Map<String, String> chat(@RequestBody ChatRequest request) {
        String content = chatClient.prompt()
                .system(SYSTEM_PROMPT)
                .user(request.message())
                .call()
                .content();
        return Map.of("content", content);
    }

    /** ② 流式输出（打字机效果，SSE） */
    @GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<org.springframework.http.codec.ServerSentEvent<String>> stream(
            @RequestParam String message) {
        return chatClient.prompt()
                .system(SYSTEM_PROMPT)
                .user(message)
                .stream()
                .content()                                  // Flux<String> 逐块文本
                .map(chunk -> org.springframework.http.codec.ServerSentEvent.<String>builder(chunk).build());
    }

    /** ③ 结构化输出：自由文本 → 工单 POJO */
    @PostMapping("/chat/entity")
    public Ticket entity(@RequestBody ChatRequest request) {
        return chatClient.prompt()
                .system("你是工单解析助手，从用户输入中提取字段，缺少的填 unknown，priority 只能是 high/medium/low")
                .user(request.message())
                .call()
                .entity(Ticket.class);                      // BeanOutputConverter
    }

    /** ④ 深度思考：用推理模型拿思维链（reasoningContent）+ 最终答案（content） */
    @GetMapping("/chat/reason")
    public Map<String, String> reason(@RequestParam String message) {
        // 1.1.8 的思考模式 = 直接换 deepseek-reasoner 模型（无 2.0 的 thinking 属性开关）
        Prompt prompt = new Prompt(message, DeepSeekChatOptions.builder()
                .model(DeepSeekApi.ChatModel.DEEPSEEK_REASONER)   // = "deepseek-reasoner"
                .build());

        ChatResponse response = deepSeekChatModel.call(prompt);

        // DeepSeek 返回的助手消息是 DeepSeekAssistantMessage 子类型
        DeepSeekAssistantMessage assistant =
                (DeepSeekAssistantMessage) response.getResult().getOutput();

        return Map.of(
                "reasoning", assistant.getReasoningContent() == null ? "" : assistant.getReasoningContent(),
                "content", assistant.getText());
    }
}
```

四个接口与第一部分能力的对应关系：

| 接口 | 方法 | 能力 |
|------|------|------|
| `/ai/chat` | POST | 同步 + System 人设 |
| `/ai/chat/stream` | GET | 流式 SSE |
| `/ai/chat/entity` | POST | 结构化输出 |
| `/ai/chat/reason` | GET | 深度思考 / 思维链（换 `deepseek-reasoner`） |

## 10. 简易聊天前端

`src/main/resources/static/index.html`（直接复制，零构建）：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Spring AI 1.x × DeepSeek</title>
    <style>
        body { font-family: "Microsoft YaHei", sans-serif; max-width: 860px; margin: 40px auto; padding: 0 16px; }
        h1 { font-size: 22px; }
        #log { border: 1px solid #ccc; border-radius: 8px; min-height: 320px; padding: 12px; background: #fafafa; white-space: pre-wrap; }
        .me { color: #1677ff; font-weight: bold; }
        .ai { color: #333; }
        .ctrl { margin-top: 12px; display: flex; gap: 8px; }
        input { flex: 1; padding: 8px; }
        button { padding: 8px 16px; cursor: pointer; }
    </style>
</head>
<body>
<h1>Spring AI 1.1.8 · DeepSeek 聊天演示</h1>
<div id="log"></div>
<div class="ctrl">
    <input id="msg" placeholder="输入问题，回车发送（流式）" onkeydown="if(event.key==='Enter')streamSend()">
    <button onclick="streamSend()">流式发送</button>
    <button onclick="syncSend()">同步发送</button>
</div>

<script>
    const log = document.getElementById('log');
    const input = document.getElementById('msg');

    function append(who, text) {
        const span = document.createElement('span');
        span.className = who;
        span.textContent = text;
        log.appendChild(span);
        window.scrollTo(0, document.body.scrollHeight);
    }

    // 流式：EventSource 消费 SSE
    function streamSend() {
        const q = input.value.trim();
        if (!q) return;
        input.value = '';
        append('me', '你：' + q + '\nAI：');
        const es = new EventSource('/ai/chat/stream?message=' + encodeURIComponent(q));
        es.onmessage = e => { append('ai', e.data); };
        es.onerror = () => es.close();
        es.onend = () => { append('ai', '\n\n'); es.close(); };
    }

    // 同步：一次拿完整回答
    async function syncSend() {
        const q = input.value.trim();
        if (!q) return;
        input.value = '';
        append('me', '你：' + q + '\nAI：');
        const resp = await fetch('/ai/chat', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ message: q })
        });
        const data = await resp.json();
        append('ai', data.content + '\n\n');
    }
</script>
</body>
</html>
```

## 11. 运行验证与排错

```bash
mvn spring-boot:run
# 浏览器打开 http://localhost:8080/index.html
```

命令行验证四个接口：

```bash
# ① 同步
curl -X POST http://localhost:8080/ai/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"用一句话介绍 Spring AI"}'

# ② 流式（-N 关闭缓冲，逐块输出）
curl -N "http://localhost:8080/ai/chat/stream?message=讲一个 Java 冷笑话"

# ③ 结构化输出
curl -X POST http://localhost:8080/ai/chat/entity \
  -H "Content-Type: application/json" \
  -d '{"message":"急件！客户张三投诉快递丢失，单号 SF8888888888，要求今天必须解决"}'

# ④ 深度思考（返回 reasoning + content 两段）
curl "http://localhost:8080/ai/chat/reason?message=9.11 和 9.8 哪个大？为什么？"
```

**常见问题排查**：

| 现象 | 原因 | 解决 |
|------|------|------|
| 401 Unauthorized | API Key 错误/未注入 | 检查 `DEEPSEEK_API_KEY` 环境变量是否在启动进程生效 |
| `NoSuchBeanDefinitionException: ChatClient.Builder` | 没引 Starter 或 BOM 没生效 | 确认依赖为 `spring-ai-starter-model-deepseek`，`mvn dependency:tree` 查版本 |
| 属性不生效 / 模型名仍是默认值 | 写成了 2.0 扁平属性 | 1.x 必须写 `spring.ai.deepseek.chat.options.model`（有 `.options` 段） |
| 调用超时 / 429 | 网络或限流 | 调 `spring.ai.retry.*`（默认最多重试 10 次、指数退避） |
| 流式接口一次性吐完 | 代理/缓冲截断 | 用 `curl -N` 验证；检查中间层是否缓冲 `text/event-stream` |
| 流式无输出但不报错 | 前端用 POST + EventSource | EventSource 只支持 GET，本案例流式接口即 GET |
| reasoning 为空 | 默认模型不是 reasoner | `/chat/reason` 必须用 `deepseek-reasoner`（本案例已显式指定） |

## 12. 进阶：接入对话记忆

在案例一基础上给 `ChatClient` 挂记忆（完整说明见第一部分 §3）：

```java
public ChatController(ChatClient.Builder builder, DeepSeekChatModel deepSeekChatModel) {
    ChatMemory chatMemory = MessageWindowChatMemory.builder()
            .maxMessages(20)
            .build();                                      // 1.x 手动构建（可注入自定义 Bean）
    this.chatClient = builder
            .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build())
            .build();
    this.deepSeekChatModel = deepSeekChatModel;
}

// 请求时带会话 ID（也可以从 Header/X-User-Id 取）
chatClient.prompt()
        .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, "conv-" + userId))
        .user(request.message())
        .call()
        .content();
```

其他进阶方向：

- **换推理模型**：yml 里 `spring.ai.deepseek.chat.options.model: deepseek-reasoner`
  （或运行时 `.options()` 覆盖）；
- **只下发工具不执行**：`.options(DeepSeekChatOptions.builder().internalToolExecutionEnabled(false).build())`，
  自己消费 `tool_calls` 做审计后再决定是否执行；
- **多供应商共存**：引入另一个 starter 后用 `spring.ai.model.chat` 选择激活谁——
  这正是第三部分多模型路由的基础。

### 第二部分小结

案例一交付了生产可用的最小形态：**一个 pom + 一份 yml + 四个接口 + 一个页面**。
其中每个接口都对应第一部分的一项能力——先懂场景，再抄代码，最后按需拼装。

---

# 第三部分 · 案例二：接入自定义本地模型（精通）

> 目标：不依赖任何云端 API，用本机（或内网服务器）模型跑通对话、多模态，
> 并实现「云端 DeepSeek ↔ 本地 Ollama」一键路由。
> 工程名：`springai-1x-local-demo`

两条技术路线：

| | 方案 A：OpenAI 兼容协议 | 方案 B：Ollama 专用 Starter |
|---|---|---|
| 原理 | 用 OpenAI starter 指向任何 OpenAI 兼容端点 | 用 Ollama 原生 API（`/api/chat`） |
| 适用 | Ollama、vLLM、LM Studio、Xinference、企业自建网关……**通吃** | 只连 Ollama，但可用 Ollama 专有能力 |
| 依赖 | `spring-ai-starter-model-openai` | `spring-ai-starter-model-ollama` |
| 属性前缀 | `spring.ai.openai.*` | `spring.ai.ollama.*` |
| 本地方案差异 | `keep_alive`、模型预拉取、思考模式等需手动传参 | 原生支持 `enableThinking()`、启动自动拉模型 |

## 13. 前置：安装 Ollama 并拉取模型

1. 安装：https://ollama.com 下载对应平台安装包（Windows/macOS/Linux），安装后 `ollama serve` 自动运行。
2. 拉取模型：

```bash
ollama pull qwen3                 # 通用对话（本案例默认，约 4.7GB）
ollama pull qwen3:4b              # 小显存机器（约 2.5GB）
ollama pull deepseek-r1:7b        # 本地推理模型
ollama pull llama3.2-vision       # 视觉模型（15 方案用）
ollama pull nomic-embed-text      # 嵌入模型（RAG 用）
ollama list                       # 查看已装模型
```

3. 验证服务与 **OpenAI 兼容端点**（方案 A 的原理依据）：

```bash
# Ollama 原生端点
curl http://localhost:11434/api/chat -d '{"model":"qwen3","messages":[{"role":"user","content":"你好"}]}'

# OpenAI 兼容端点（/v1/chat/completions）
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3","messages":[{"role":"user","content":"你好"}]}'
```

> 两个端点返回格式不同，但**后者与 OpenAI 官方 API 完全一致**——这就是方案 A 只改
> `base-url` 就能接入的全部秘密。

## 14. 方案 A：OpenAI 兼容协议通用接入

### 14.1 依赖与配置

`pom.xml` 只把案例一的 starter 换成 OpenAI：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
</dependency>
```

`application.yml`——**与连真 OpenAI 唯一的区别就是三行**：

```yaml
spring:
  ai:
    model:
      chat: openai
    openai:
      base-url: http://localhost:11434/v1   # ← 关键：指向本地 OpenAI 兼容端点
      api-key: ollama                        # ← 本地不校验 Key，但属性必填，填任意非空值
      chat:
        options:                             # 1.x 嵌套写法
          model: qwen3                       # 本地已拉取的模型名
          temperature: 0.7
```

主流本地推理服务的 `base-url` 对照：

| 服务 | base-url | 说明 |
|------|----------|------|
| **Ollama** | `http://localhost:11434/v1` | 本案例默认 |
| **vLLM** | `http://localhost:8000/v1` | 高并发生产部署首选 |
| **LM Studio** | `http://localhost:1234/v1` | 桌面端实验 |
| **Xinference** | `http://localhost:9997/v1` | 多框架模型管理 |
| OpenAI 官方 | `https://api.openai.com/v1` | 默认值（不改就是连官方） |

### 14.2 代码（同步 + 流式）

```java
package com.example.localdemo;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.http.MediaType;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/local")
public class LocalChatController {

    private final ChatClient chatClient;

    public LocalChatController(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultSystem("你是运行在本地的 AI 助手，数据不出内网。")
                .build();
    }

    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        return chatClient.prompt().user(message).call().content();
    }

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<String>> stream(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .stream()
                .content()
                .map(chunk -> ServerSentEvent.<String>builder(chunk).build());
    }
}
```

```bash
curl "http://localhost:8080/local/chat?message=一句话介绍你自己"
curl -N "http://localhost:8080/local/stream?message=写一首四行小诗"
```

> **方案 A 的精髓**：业务代码与案例一**一字不差**，差异全在 yml——
> 这就是 01 篇说的"可移植抽象"。上线时把 `base-url` 换成内网 vLLM 地址即可平滑迁移。

## 15. 方案 B：Ollama 专用 Starter（含多模态）

### 15.1 依赖与配置

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    model:
      chat: ollama
    ollama:
      base-url: http://localhost:11434     # 无需 /v1，走 Ollama 原生 API
      chat:
        options:                           # 1.x 嵌套写法
          model: qwen3
          temperature: 0.7
      # 启动时自动确保模型存在（可选）
      init:
        pull-model-strategy: when_missing
```

> 思考模式说明：Ollama 的 think 能力在 1.1.8 **没有 yml 属性开关**（jar 里 `OllamaChatProperties`
> 只有 `options` 段，think 通过 `ThinkOption` 程序化控制）——需要时在代码里用
> `OllamaChatOptions.builder().enableThinking()` / `.thinkHigh()`（qwen3、deepseek-r1 等
> 思考型模型在 Ollama 0.12+ 会按模型默认行为自动启用）。

### 15.2 文本对话

```java
@RestController
@RequestMapping("/ollama")
public class OllamaChatController {

    private final OllamaChatModel ollamaChatModel;   // Starter 自动装配

    public OllamaChatController(OllamaChatModel ollamaChatModel) {
        this.ollamaChatModel = ollamaChatModel;
    }

    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        return ollamaChatModel.call(message);        // ChatModel 的 String 便捷方法
    }

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ChatResponse> stream(@RequestParam String message) {
        return ollamaChatModel.stream(new Prompt(message));
    }
}
```

### 15.3 多模态：让本地模型"看图"

```java
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.content.Media;
import org.springframework.core.io.ClassPathResource;
import org.springframework.util.MimeTypeUtils;

@GetMapping("/vision")
public String vision(@RequestParam(defaultValue = "描述这张图片里有什么") String question) {
    var image = new ClassPathResource("cat.png");   // 放一张图到 src/main/resources/cat.png

    UserMessage userMessage = UserMessage.builder()
            .text(question)
            .media(new Media(MimeTypeUtils.IMAGE_PNG, image))   // 文本 + 图片一起发
            .build();

    ChatResponse response = ollamaChatModel.call(new Prompt(userMessage));
    return response.getResult().getOutput().getText();
}
```

配套 yml 把 `spring.ai.ollama.chat.options.model` 换成视觉模型 `llama3.2-vision`
（或在请求级 `.options()` 覆盖）。

### 15.4 方案 A vs 方案 B 实测差异

| 维度 | 方案 A（openai starter） | 方案 B（ollama starter） |
|------|--------------------------|--------------------------|
| 连接端点 | `/v1/chat/completions` | `/api/chat` |
| 思考模式开关 | 需自行构造请求参数 | 程序化 `enableThinking()/thinkHigh()`（1.1.8 无 yml 属性） |
| 模型预热/keep_alive | 手动传参 | `keep-alive` 选项 / init 拉取策略 |
| 换成 vLLM | **改一行 base-url 即可** | 不支持（协议不同） |
| 图片输入 | OpenAI 标准 media 格式 | 原生支持（15.3 示例） |
| 推荐场景 | 供应商无关的通用接入、多后端切换 | 深度使用 Ollama 的团队 |

## 16. 多模型路由：云端 / 本地一键切换

**场景**：默认走云端 DeepSeek（质量优先），云端限流或数据敏感时自动降级到本地 Ollama。

1.1.8 的规则（01 篇 §8）：`spring.ai.model.chat=deepseek` 使**只有 DeepSeek 命中自动装配**
（其余供应商 back off），另一个模型**手动 Builder 构建**——这正是本节的做法。

### 16.1 依赖（两个 Starter 共存）

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-deepseek</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
```

### 16.2 配置：DeepSeek 自动装配，Ollama 手动接管

```yaml
spring:
  ai:
    model:
      chat: deepseek              # 选择器：对话模型自动装配 = DeepSeek（其余供应商 back off）
    deepseek:
      api-key: ${DEEPSEEK_API_KEY:sk-your-key}
      chat:
        options:
          model: deepseek-chat
    ollama:                       # 保留 Ollama 连接配置（手动 Bean 里直接读取）
      base-url: http://localhost:11434

local:
  ollama:
    enabled: true                 # 本地模型开关（机器没装 Ollama 时设为 false 即可）
```

### 16.3 手动构建本地 ChatModel

```java
package com.example.localdemo.config;

import org.springframework.ai.deepseek.DeepSeekChatModel;
import org.springframework.ai.ollama.OllamaChatModel;
import org.springframework.ai.ollama.api.OllamaApi;
import org.springframework.ai.ollama.api.OllamaChatOptions;
import org.springframework.ai.ollama.api.OllamaModel;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ConditionalOnProperty;

@Configuration
public class ModelRoutingConfig {

    /** 手动构建本地 Ollama 模型（自动装配的名额被 16.2 的 deepseek 占用） */
    @Bean
    @ConditionalOnProperty(name = "local.ollama.enabled", havingValue = "true", matchIfMissing = true)
    OllamaChatModel localOllamaChatModel(
            @Value("${spring.ai.ollama.base-url:http://localhost:11434}") String baseUrl) {

        OllamaApi api = OllamaApi.builder().baseUrl(baseUrl).build();

        // 1.1.8 的 OllamaChatOptions.Builder 同时支持 model(String) 与 model(OllamaModel)：
        OllamaChatOptions options = OllamaChatOptions.builder()
                .model(OllamaModel.QWEN3_7B)        // 枚举：QWEN3_7B = "qwen3:7b"、LLAMA3 = "llama3"……
                // .model("qwen3")                   // 或任意已 pull 的模型名（fine-tune 模型也行）
                .build();

        return OllamaChatModel.builder()
                .ollamaApi(api)
                .defaultOptions(options)            // ← 1.x 构建方法叫 defaultOptions（2.0 改名 .options）
                .build();
    }

    /**
     * 云端模型 = Starter 自动装配的 DeepSeekChatModel（注入即可）；
     * 本地模型可能被 @ConditionalOnProperty 关掉，用 ObjectProvider 安全获取。
     */
    @Bean
    ModelRouter modelRouter(DeepSeekChatModel deepSeekChatModel,
                            ObjectProvider<OllamaChatModel> localOllama) {
        return new ModelRouter(deepSeekChatModel, localOllama.getIfAvailable());
    }
}
```

> 说明：`spring.ai.model.chat: deepseek` 让 DeepSeek 独占对话模型自动装配名额
> （jar 实证各供应商自动配置类为 `havingValue=<id>` + `matchIfMissing=true`），
> Ollama 的自动装配不命中而 back off，所以这里手动 `@Bean` 不会与 Starter 冲突
> （即便冲突，容器里已存在用户 Bean 时自动配置也会让位）。

### 16.4 路由服务与接口

```java
package com.example.localdemo.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;

public class ModelRouter {

    private final ChatClient cloudClient;    // DeepSeek
    private final ChatClient localClient;    // Ollama（可能为 null）

    public ModelRouter(ChatModel cloud, ChatModel local) {
        this.cloudClient = ChatClient.create(cloud);
        this.localClient = local != null ? ChatClient.create(local) : null;
    }

    public String ask(boolean preferLocal, String message) {
        ChatClient target;
        if (preferLocal && localClient != null) {
            target = localClient;                       // 数据不出内网
        } else {
            target = cloudClient;                       // 质量优先
        }
        return target.prompt().user(message).call().content();
    }
}
```

```java
@RestController
@RequestMapping("/route")
public class RouteController {

    private final ModelRouter router;

    public RouteController(ModelRouter router) {
        this.router = router;
    }

    @GetMapping
    public Map<String, String> route(@RequestParam(defaultValue = "false") boolean local,
                                     @RequestParam String message) {
        long start = System.currentTimeMillis();
        String answer = router.ask(local, message);
        return Map.of(
                "model", local ? "local-ollama" : "cloud-deepseek",
                "costMs", String.valueOf(System.currentTimeMillis() - start),
                "content", answer);
    }
}
```

```bash
curl "http://localhost:8080/route?message=介绍下你自己"                 # 走云端
curl "http://localhost:8080/route?local=true&message=介绍下你自己"      # 走本地
```

**进阶路由策略**（生产常见四种）：手动指定（如上）、按用户/租户路由、
按内容敏感级路由（正则命中敏感词 → 强制本地）、失败降级
（云端 429/超时 → fallback 到本地，可用 Resilience4j/Circuit Breaker 包一层）。

## 17. 生产化要点

| 维度 | 建议 |
|------|------|
| **硬件** | 7B-q4 模型约需 5GB 显存/内存，14B 约 10GB；纯 CPU 可跑但 token 生成慢（先用 `qwen3:4b` 验证链路） |
| **推理引擎** | 开发用 Ollama；**高并发生产换 vLLM**（continuous batching，方案 A 只改 base-url） |
| **并发** | Ollama 请求内部排队，多实例横向扩；vLLM 支持真并发 |
| **超时与重试** | `spring.ai.retry.max-attempts`、RestClient 超时按本地推理实际耗时上调 |
| **模型预热** | Ollama `keep-alive` / vLLM 常驻加载，避免首请求冷启动 |
| **数据安全** | 本地模型最大价值：**Prompt 与业务数据不出内网**，合规场景刚需 |
| **成本** | 云端按 token 计费适合低频高质；本地一次性硬件投入适合高频稳定负载——路由层让你两者兼得 |
| **可观测** | 本地模型同样接入 Micrometer，按 `model` 标签对比两端时延与失败率 |

### 第三部分小结

| 路线 | 一句话 | 核心配置 |
|------|--------|---------|
| 方案 A | OpenAI 兼容协议通吃所有本地推理服务 | `spring.ai.openai.base-url` 指向 `/v1` + `.chat.options.model` |
| 方案 B | Ollama 原生能力全开（思考/预热/自动拉取） | `spring.ai.ollama.*`（1.x 嵌套 `.options`） |
| 多模型路由 | 一个自动装配 + 一个手动 Builder（`defaultOptions`），路由层决策 | `spring.ai.model.chat` + `ChatClient.create(model)` |

---

## 全篇知识图谱

```
场景 ──六大场景──→ 能力 ──五大能力──→ 代码
 │                                    │
 ├ 智能客服      → 记忆/人设      → §2 §3
 ├ 知识库问答    → RAG           → §6（advisors-vector-store）
 ├ 内容生成      → 流式          → 案例一 /stream
 ├ 数据提取      → 结构化输出    → §4 案例一 /entity
 ├ 业务自动化    → Tool Calling  → §5（内部循环 + ToolCallAdvisor）
 └ 多模态        → Media         → 案例二 /vision

案例一（云端 DeepSeek）: pom(Boot 3.5.16) → yml(.options 嵌套) → 四接口 → 前端 → 排错
案例二（本地模型）: Ollama 安装 → 方案A(OpenAI兼容) → 方案B(专用Starter+视觉+enableThinking)
                  → 多模型路由(model.chat 选择器 + defaultOptions 手动 Builder) → 生产化
```

## 面试高频题

**实战操作**

1. 写出 Spring AI 1.1.8 接入 DeepSeek 的完整三件套（依赖、属性前缀、注入的 Bean）。
2. `spring.ai.deepseek.chat.options.model` 在 2.0 怎么写？为什么改？
3. 流式接口为什么用 GET + `Flux<ServerSentEvent<String>>`？前端怎么消费？
4. `conversationId` 不传会发生什么？如何做到用户间记忆隔离？
5. 本地 Ollama 的 OpenAI 兼容端点地址是什么？方案 A 需要改哪几行配置？

**原理理解**

6. `.entity(Ticket.class)` 背后发生了哪三步？
7. 案例一的 `/ai/chat/reason` 如何拿到思维链？1.1.8 与 2.0 的实现方式有何不同？
8. `spring.ai.model.chat: deepseek` 在 1.1.8 的条件注解语义是什么（提示：havingValue + matchIfMissing）？
9. RAG 示例中 `QuestionAnswerAdvisor` 在调用链的哪一步执行、模块名在 2.0 叫什么？
10. 1.x 的工具调用循环藏在哪里？`ToolCallAdvisor` 与默认行为有什么关系？

**方案设计**

11. 设计一个"云端限流时自动降级本地"的路由，需要哪些组件？
12. 内网 vLLM 与 Ollama 如何用同一套代码接入？
13. 电商项目：订单查询走 AI + Tools，客户数据禁止出内网——怎么落地？
14. 本地模型首 token 延迟高，从哪些角度优化？
15. 本教程的两个案例升级到 Spring AI 2.0 + Boot 4，需要改哪几处？（对照 01 篇 §14 清单回答）

---

## 面试高频题

**实战操作**

1. 写出 Spring AI 1.1.8 接入 DeepSeek 的完整三件套（依赖、属性前缀、注入的 Bean）。

   **答**：①依赖：`spring-ai-bom`（1.1.8）+ `spring-ai-starter-model-deepseek`；②属性：`spring.ai.deepseek.api-key` 必填，模型可选 `spring.ai.deepseek.chat.options.model: deepseek-chat`——注意 1.x **有 `.options` 段**（与 2.0 最直观的区别）；③Bean：starter 自动装配 `ChatClient.Builder`，注入后 `chatClient.prompt().user(...).call().content()` 即用（`DeepSeekChatModel` 也已自动配置）。加分点：多供应商共存时用 `spring.ai.model.chat: deepseek` 指定激活谁。

2. `spring.ai.deepseek.chat.options.model` 在 2.0 怎么写？为什么改？

   **答**：2.0 写作 `spring.ai.deepseek.chat.model`——整体去掉 `.options` 段。原因：2.0 将全部供应商属性扁平化对齐 ChatOptions 字段（`spring.ai.<供应商>.chat.<字段>`），消除 yml 嵌套与 Java 对象的双向映射成本；同时 Java 侧 Options 改为严格不可变，Builder `.defaultOptions()` 改名 `.options()`。升级口诀：全局搜 `.chat.options.` 与 `.defaultOptions(`（见 01 篇 §14）。

3. 流式接口为什么用 GET + `Flux<ServerSentEvent<String>>`？前端怎么消费？

   **答**：SSE（`text/event-stream`）是服务端单向推送的标准格式，浏览器原生 `EventSource` **只支持 GET**——参数就是一句话，放 query string 足够。后端 `Flux<ServerSentEvent<String>>` 异步写出，模型流式 delta 逐条下发不占 Tomcat 线程。前端 `new EventSource('/ai/chat/stream?message=...')`，`onmessage` 里累加 `event.data` 渲染。坑点：`EventSource` 不能带自定义 Header（要 token 用 `fetch` + `ReadableStream`）；前端误用 POST + EventSource 会静默无输出——这正是接口设计为 GET 的原因。

4. `conversationId` 不传会发生什么？如何做到用户间记忆隔离？

   **答**：不传时记忆 Advisor 回落到默认会话 ID，所有用户共享同一段历史——多用户场景会串上下文。隔离靠 `ChatMemory` 的会话分桶：每次请求 `.advisors(a -> a.param(ChatMemory.CONVERSATION_ID, "conv-" + userId))`，以用户维度传唯一 ID。配套：`MessageWindowChatMemory.builder().maxMessages(n)` 限窗防膨胀；生产换 JDBC/Redis 的 `ChatMemoryRepository`；1.1.8 没有自动配置的 `ChatMemory` Bean，需按案例手动构建（2.0/Boot 4 起 classpath 有依赖即可注入）。

5. 本地 Ollama 的 OpenAI 兼容端点地址是什么？方案 A 需要改哪几行配置？

   **答**：地址是 `http://localhost:11434/v1`（**必须带 `/v1`**）。方案 A 用 OpenAI Starter，共改三处：`spring.ai.openai.base-url: http://localhost:11434/v1`、`spring.ai.openai.api-key: ollama`（占位字符串即可）、`spring.ai.openai.chat.options.model: qwen3`（1.x 记得带 `.options` 段）。代码零改动。对比方案 B：Ollama 专用 Starter 的 base-url **不带** `/v1`，模型属性为 `spring.ai.ollama.chat.options.model`。

**原理理解**

6. `.entity(Ticket.class)` 背后发生了哪三步？

   **答**：①`BeanOutputConverter` 把 Ticket 的字段结构生成 JSON Schema 格式指令注入 Prompt（告诉模型怎么答）；②`call()` 拿到模型返回的 JSON 文本；③Jackson 反序列化为 `Ticket` 对象。1.1.8 与 2.0 的差异：1.x 解析失败直接抛异常，需自行 try/catch 决定是否重试；2.0 会把错误回喂模型自动重试自纠。加分：`.entity()` 本质是 `BeanOutputConverter` 的语法糖，也可手动拼 `getFormat()`。

7. 案例一的 `/ai/chat/reason` 如何拿到思维链？1.1.8 与 2.0 的实现方式有何不同？

   **答**：1.1.8 的「思考模式」= 换推理模型：`DeepSeekChatOptions.builder().model(DeepSeekApi.ChatModel.DEEPSEEK_REASONER).build()`（枚举值 `deepseek-reasoner`），**没有** 2.0 的 `thinking.type` / `reasoning-effort` 属性开关。响应中助手消息实际类型为 `DeepSeekAssistantMessage`（`AssistantMessage` 子类），强转后 `getReasoningContent()` 取思维链、`getText()` 取答案。2.0 则是 v4 系模型 + `enableThinking()`/属性开启，同一模型思考可开关。加分：reasoner 对流式与工具调用支持有限，生产上把「是否思考」做成接口开关。

8. `spring.ai.model.chat: deepseek` 在 1.1.8 的条件注解语义是什么（提示：havingValue + matchIfMissing）？

   **答**：底层是 `@ConditionalOnProperty(name = "spring.ai.model.chat", havingValue = "deepseek", matchIfMissing = true)`：配置为 `deepseek` 时只有 DeepSeek 命中自动装配；**不配置时（matchIfMissing=true）所有供应商都满足条件**——先全部装配，再靠 OpenAI 侧 `@ConditionalOnMissingBean` 等退让机制收敛为一个；`=none` 全部禁用。所以 1.1.8 是「缺省全开 + 自我退让链」，2.0 收紧为显式唯一选择器——这是两版多模型行为的关键差异（01 篇 §8）。

9. RAG 示例中 `QuestionAnswerAdvisor` 在调用链的哪一步执行、模块名在 2.0 叫什么？

   **答**：Advisor 链在模型调用之前执行。`QuestionAnswerAdvisor.adviseCall()` 里完成：用户问题 → EmbeddingModel 向量化 → VectorStore 相似检索 top-k 片段 → 拼进 Prompt 作为上下文 → 放行给模型。注册：`.defaultAdvisors(QuestionAnswerAdvisor.builder(vectorStore).build())`，1.1.8 的依赖模块是 **`spring-ai-advisors-vector-store`**——2.0 该模块改名 **`spring-ai-vector-store-advisor`**（`QuestionAnswerAdvisor` 类名不变），升级时 pom 必须同步改（01 篇 §14 第 8 条）。

10. 1.x 的工具调用循环藏在哪里？`ToolCallAdvisor` 与默认行为有什么关系？

**答**：1.x 的循环**藏在 `ChatModel` 实现内部**（`call()`/`stream()` 里）：模型响应含 `toolCalls` 且 Options 上 `internalToolExecutionEnabled`（默认 true）→ 框架自动执行 `@Tool` 方法 → 结果回填 messages → 再次调模型，循环至无 `toolCalls`。`ToolCallAdvisor` 是顾问式的**可选替代路径**：手动 `.advisor(...)` 挂载才生效（默认 order = `MIN_VALUE + 300`），与内部循环互为两套实现，普通业务用默认即可。2.0 把循环统一外置为自动注册的 `ToolCallingAdvisor`——这是两版最大的架构差异（01 篇 §13）。

**方案设计**

11. 设计一个"云端限流时自动降级本地"的路由，需要哪些组件？

**答**：四个组件——①两个 ChatModel：自动装配的云端 DeepSeek + 手动 Builder 的本地 Ollama（`OllamaChatModel.builder().…​.defaultOptions(options).build()` 后 `ChatClient.create`）；②路由器：对云端调用包 `try/catch`（429/超时）或 Resilience4j 熔断器，失败切本地；③统一出口：业务层只认一个 `AiRouter.chat(prompt)` 接口，Controller 不感知底层；④保障件：云端设短超时快失败、本地模型预热（Ollama `init.pull-model-strategy` / keep_alive）、切换次数打监控。加分：降级条件除异常外还可主动读限流响应头；1.x 手动 Builder 记得 `.defaultOptions()`（2.0 改名 `.options()`）。

12. 内网 vLLM 与 Ollama 如何用同一套代码接入？

**答**：两者都提供 OpenAI 兼容接口，同一套代码只换配置：`spring.ai.openai.base-url` 指 `http://vllm-host:8000/v1` 或 `http://localhost:11434/v1`，`spring.ai.openai.chat.options.model` 换成对应模型名（1.x 带 `.options` 段），`api-key` 占位；用 Spring Profile 区分环境即可，代码层完全复用同一 `ChatClient`。只有需要 Ollama 专属能力（think、预热、自动拉取）时才额外引 Ollama 专用 Starter（`spring.ai.ollama.*`，同样 `.options` 嵌套）——那是并行的第二套装配，不影响主路径。加分：provider→baseUrl/model 配置表驱动，新推理服务零代码接入。

13. 电商项目：订单查询走 AI + Tools，客户数据禁止出内网——怎么落地？

**答**：核心原则——**模型在外、数据在内，工具做脱敏代理**。落地三步：①订单查询以内网微服务实现，AI 挂载的 `@Tool` 只是薄壳，入参订单号、出参聚合/脱敏结果，原始客户字段不出内网；②Prompt 只携带非敏感键与脱敏摘要，模型永远看不到完整客户数据；③若合规要求更严，整条链内网化（Ollama/vLLM 本地推理），数据零出网。1.x 额外注意：工具循环在 ChatModel 内部且 `internalToolExecutionEnabled` 默认 true——务必只在请求级显式 `.tools(...)` 挂载所需工具；可用 `.options(…​.internalToolExecutionEnabled(false).build())` 只下发不执行、由业务受控调用（2.0 天然具备该隔离）。再加审计日志记录每次工具调用入参出参。

14. 本地模型首 token 延迟高，从哪些角度优化？

**答**：分四层——①模型侧：更小量化（q4）、延迟友好架构、reasoner 类重推理模型换轻量模型；②服务侧：模型常驻不反复加载、Ollama keep_alive/预热、vLLM 连续批处理与 Paged Attention；③框架侧：用流式输出先吐首 token、Prompt 瘦身（RAG top-k 减半、system 压缩）；④工程侧：GPU offload 拉满、限并发防排队、SSD 加载。排查先分清两段：TTFT（首 token，prefill 慢）与 TPS（decode 慢），优化方向完全不同。加分：预热后用固定 Prompt 循环压测取 P95。

15. 本教程的两个案例升级到 Spring AI 2.0 + Boot 4，需要改哪几处？（对照 01 篇 §14 清单回答）

**答**：对照 01 篇 §14 的 12 条，按操作顺序——①**pom**：Boot 3.5→4.1、BOM 1.1.8→2.0.1、RAG 模块 `spring-ai-advisors-vector-store`→`spring-ai-vector-store-advisor`；②**yml**：全局搜 `.chat.options.` 改扁平（`spring.ai.deepseek.chat.model`、`spring.ai.ollama.chat.model`），多模型启停删 `chat.enabled`、统一 `spring.ai.model.chat`；③**Java**：`defaultOptions(`→`.options(`、`copy()`→`mutate().build()`；④**工具**：确认行为移入自动注册的 `ToolCallingAdvisor`，若曾手动挂 `ToolCallAdvisor` 需换 `ToolCallingAdvisor`，并评估 `spring.ai.tools.limits.*` 限额；⑤**reason 端点**：可保留换模思路，也可改用 v4 + `enableThinking()`；⑥**不变项**：SSE、记忆 Advisor、RAG、`.entity()` 用法照旧。改完对四个接口跑一遍回归即可。

---