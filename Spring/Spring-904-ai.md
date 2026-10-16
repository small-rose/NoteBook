---
layout: default
title: Spring AI 调用链
parent: Spring
grand_parent: Java
nav_order: 94
---

# Spring AI 调用链
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


# 03 · 理解篇：场景与调用过程

> 阶段：理解调用链
> 本篇目标：理解spring ai 完整且覆盖多种场景的调用链过程

---

# 展示一次用户请求从进入 Spring AI 到最终返回的全过程

## 1. 场景覆盖

- 普通对话
- RAG（检索增强生成）
- Tool Calling（工具调用）
- 记忆（ChatMemory）
- 流式与非流式两种模式

为了清晰，采用缩进层级 + 箭头 + 注释的方式，模拟时序图的阅读顺序（从上到下，从左到右表示调用方向）。


## 2. 场景设定

用户提问：

    “今天北京天气怎么样？顺便帮我查一下订单 ORD-123 的状态。”

系统已有：

- WeatherTool（查天气，需调用第三方 API）
- OrderTool（查订单状态，查数据库）
- VectorStore 中存有公司产品手册（RAG 用）
- ChatMemory 存储了最近 10 轮对话

## 3. 场景覆盖的组件

| 组件 | 在本时序图中的角色                     |
|-----|-------------------------------|
|ChatClient	| 入口，组装 Prompt，触发执行             |
|MessageChatMemoryAdvisor	| 注入历史对话，保存新对话                  |
|QuestionAnswerAdvisor	| 执行 RAG 检索，拼入上下文               |
|ToolCallingAdvisor	| 管理工具定义、解析 tool_call、执行工具、二次调用 |
|EmbeddingModel	| 将问题转为向量                       |
|VectorStore	| 存储文档向量，执行相似度搜索                |
|OpenAiChatModel	| 厂商适配，转换消息格式，发起 HTTP 请求        |
|WeatherTool / OrderTool	| 用户自定义 @Tool 方法 |
|ChatMemory	| 存储对话历史（InMemory 或 Redis） |


## 4. 调用时序图

```
用户 HTTP 请求 /ask?q=今天北京天气？查订单ORD-123
  │
  ▼
【ChatClient.prompt().user(...).call().content()】         ← 入口
  │
  ├── 1. 构造 Prompt（UserMessage + 历史记忆？）
  │     └─ 此时 Prompt 内只有当前用户消息
  │
  ├── 2. 【Advisor 链 - 前置处理】
  │     │
  │     ├── 2a. MessageChatMemoryAdvisor.before(prompt)
  │     │     ├─ 从 ChatMemory 加载最近 5 条对话（system/user/assistant）
  │     │     └─ 将历史消息插入 Prompt 头部
  │     │
  │     ├── 2b. QuestionAnswerAdvisor.before(prompt)
  │     │     ├─ 提取用户问题（去除历史干扰）
  │     │     ├─ 调用 EmbeddingModel 将问题转为向量
  │     │     ├─ 调用 VectorStore.similaritySearch(queryVector, topK=3)
  │     │     ├─ 得到 3 个相关文档片段
  │     │     └─ 将文档片段作为 SystemMessage 拼入 Prompt（如：“以下是相关知识：...”）
  │     │
  │     └── 2c. ToolCallingAdvisor.before(prompt)
  │           ├─ 扫描所有 @Tool 方法（WeatherTool.getWeather, OrderTool.getOrderStatus）
  │           ├─ 生成 tools 定义（JSON Schema 数组）
  │           └─ 将 tools 附加到 Prompt 的 options 中
  │
  ├── 3. 【第一次调用 ChatModel】
  │     │
  │     ├── 3a. OpenAiChatModel.call(prompt)
  │     │     ├─ 将 Spring AI 的 Prompt 转为 OpenAI API 请求体（JSON）
  │     │     │   { model: "gpt-4o", messages: [...], tools: [...] }
  │     │     ├─ HTTP POST https://api.openai.com/v1/chat/completions
  │     │     └─ 等待响应（约 1~2 秒）
  │     │
  │     └── 3b. 模型返回响应（非最终回答，而是 tool_call）
  │           ├─ choices[0].message.content = null
  │           ├─ choices[0].message.tool_calls = [
  │           │     { id: "call_xxx", function: { name: "getWeather", arguments: '{"city":"北京"}' } },
  │           │     { id: "call_yyy", function: { name: "getOrderStatus", arguments: '{"orderId":"ORD-123"}' } }
  │           │   ]
  │           └─ finish_reason = "tool_calls"
  │
  ├── 4. 【ToolCallingAdvisor 处理 tool_call】
  │     │
  │     ├── 4a. 遍历 tool_calls
  │     │     ├─ 调用 WeatherTool.getWeather(city="北京") 
  │     │     │     └─ 内部：HTTP 调用天气 API → 返回 "晴，22°C"
  │     │     └─ 调用 OrderTool.getOrderStatus(orderId="ORD-123")
  │     │           └─ 内部：查询数据库 → 返回 "已发货，预计明天到达"
  │     │
  │     ├── 4b. 构造 ToolMessage 列表
  │     │     ├─ ToolMessage(tool_call_id="call_xxx", content="晴，22°C")
  │     │     └─ ToolMessage(tool_call_id="call_yyy", content="已发货，预计明天到达")
  │     │
  │     └── 4c. 将 ToolMessage 追加到 Prompt 中
  │           └─ 新的 Prompt 包含：历史消息 + 原始用户消息 + 工具结果
  │
  ├── 5. 【第二次调用 ChatModel】（携带工具结果）
  │     │
  │     ├── 5a. OpenAiChatModel.call(updatedPrompt)
  │     │     └─ 再次 HTTP POST（同上，但 messages 多了 tool 结果）
  │     │
  │     └── 5b. 模型返回最终回答（finish_reason = "stop"）
  │           └─ content = "北京今天晴天，气温22°C。您的订单 ORD-123 已发货，预计明天送达。"
  │
  ├── 6. 【Advisor 链 - 后置处理】
  │     │
  │     ├── 6a. ToolCallingAdvisor.after(response) — 无操作
  │     ├── 6b. QuestionAnswerAdvisor.after(response) — 无操作
  │     └── 6c. MessageChatMemoryAdvisor.after(response)
  │           └─ 将本次对话（用户消息 + 助手回答）存入 ChatMemory
  │
  └── 7. 【返回结果】
        └─ content() 提取 AssistantMessage.content → "北京今天晴天..."
```

流式模式的区别

如果使用 .stream() 而不是 .call()，流程变化如下：

```
├── 3'. 【第一次调用 ChatModel（流式）】
  │     ├─ 使用 WebClient 发起 SSE 请求
  │     ├─ 逐 chunk 接收 data: {...} 事件
  │     └─ 每收到一个 chunk，就通过 Flux 发射给调用方
  │
  ├── 4'. 【ToolCallingAdvisor 处理 tool_call】
  │     └─ 需要等到所有 chunk 接收完毕，才能判断是否是 tool_call
  │        （因为 tool_call 可能出现在最后一个 chunk 中）
  │
  ├── 5'. 【第二次调用 ChatModel（流式）】
  │     └─ 同样逐 chunk 发射最终回答
  │
  └── 6'. 【Advisor 后置处理】
        └─ 在 Flux 完成时执行（如保存记忆）
```
关键差异：流式模式下，Advisor 的前置处理是一样的，但后置处理需要等到整个 Flux 结束。Tool Calling 的判断需要等待第一个响应的完整 JSON（因为 tool_call 可能在最后一个 delta 中出现）。


总结

{: .tips }
> 一次看似简单的 chatClient.prompt().user(...).call().content()，背后可能经历了 两次甚至多次 LLM 调用，穿插着 向量检索、工具执行、记忆读写，全部由 Spring AI 的 Advisor 链​ 有序编排。这就是它“智能编排”的本质。


## 5、案例代码


### 5.1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.1</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>spring-ai-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-ai-demo</name>
    <description>Spring AI 完整案例 - RAG + Tool Calling + ChatMemory</description>

    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M6</spring-ai.version>
    </properties>

    <dependencyManagement>
        <dependencies>
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
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring AI OpenAI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-model-openai</artifactId>
        </dependency>

        <!-- Spring AI Tika Document Reader (用于读取文档做 RAG) -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-tika-document-reader</artifactId>
        </dependency>

        <!-- 内存向量存储 (SimpleVectorStore) -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-advisors-vector-store</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- 测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 5.2 启动类

```java
package com.example.springaidemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringAiDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringAiDemoApplication.class, args);
    }
}
```
### 5.3 配置类

```java
package com.example.springaidemo.config;

import com.example.springaidemo.tool.BusinessTools;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.MessageChatMemoryAdvisor;
import org.springframework.ai.chat.client.advisor.QuestionAnswerAdvisor;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * ============================================================
 * ChatClient 配置 - 组装 Advisor 链
 * ============================================================
 *
 * 这是整个 Spring AI 的核心配置类，负责：
 * 1. 创建 ChatClient Bean（应用注入使用）
 * 2. 组装 Advisor 链（执行顺序就是下面 addAdvisors 的顺序）
 * 3. 配置默认系统提示词
 *
 * Advisor 执行顺序（请求方向）：
 *   MessageChatMemoryAdvisor → QuestionAnswerAdvisor → ToolCallingAdvisor(内置) → ChatModel
 *
 * Advisor 执行顺序（响应方向 - 倒序）：
 *   ChatModel → ToolCallingAdvisor → QuestionAnswerAdvisor → MessageChatMemoryAdvisor
 * ============================================================
 */
@Slf4j
@Configuration
public class ChatClientConfig {

    /**
     * 创建配置了完整 Advisor 链的 ChatClient
     *
     * @param chatClientBuilder Spring AI 自动配置的 Builder
     * @param chatMemory        聊天记忆（用于多轮对话）
     * @param vectorStore       向量存储（用于 RAG 检索）
     * @param businessTools     业务工具（用于 Tool Calling）
     */
    @Bean
    public ChatClient chatClient(
            ChatClient.Builder chatClientBuilder,
            ChatMemory chatMemory,
            VectorStore vectorStore,
            BusinessTools businessTools) {

        return chatClientBuilder
                // 默认系统提示词
                .defaultSystem("""
                        你是一个智能企业助手，可以：
                        1. 回答关于公司产品的各种问题（基于知识库）
                        2. 查询天气信息（调用天气工具）
                        3. 查询订单状态（调用订单工具）
                        4. 进行多轮对话（记住上下文）
                        
                        请根据用户问题选择合适的工具或知识库信息来回答。
                        回答时使用中文，保持友好和专业。
                        """)

                // ===== Advisor 链配置 =====
                // 执行顺序：请求时从上到下，响应时从下到上

                // 1️⃣ 记忆 Advisor（最先执行前置，最后执行后置）
                //    - 前置：从 ChatMemory 加载历史消息，注入 Prompt
                //    - 后置：将本轮对话保存到 ChatMemory
                .defaultAdvisors(
                        MessageChatMemoryAdvisor.builder(chatMemory)
                                .build()
                )

                // 2️⃣ RAG Advisor（Question Answer）
                //    - 前置：将用户问题向量化 → 检索 VectorStore → 拼入 System Prompt
                //    - 后置：无操作
                .defaultAdvisors(
                        QuestionAnswerAdvisor.builder(vectorStore)
                                .build()
                )

                // 3️⃣ 工具配置（Tool Calling）
                //    - 内部由 ToolCallingAdvisor 处理
                //    - 前置：扫描 @Tool 方法，生成 tools JSON Schema，附加到请求
                //    - 后置：解析 LLM 返回的 tool_call → 反射调用 Java 方法 → 结果回传
                .defaultTools(businessTools)

                // 构建
                .build();
    }
}
```

### 5.4 模型类

订单信息

```java
package com.example.springaidemo.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 订单信息模型
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderInfo {
    private String orderId;
    private String status;
    private String productName;
    private String shippingAddress;
    private String estimatedDelivery;
    private double amount;
}
```

天气信息
```java
package com.example.springaidemo.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 天气信息模型
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class WeatherInfo {
    private String city;
    private String condition;
    private int temperature;
    private String humidity;
    private String wind;
}
```

## 5.5 service类

### 5.5.1 ChatMemoryService

```java
package com.example.springaidemo.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.chat.memory.InMemoryChatMemory;
import org.springframework.ai.chat.messages.Message;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

/**
 * ============================================================
 * 聊天记忆服务 - 管理多轮对话上下文
 * ============================================================
 *
 * 核心原理：
 * - 每个会话（conversationId）维护一个独立的消息列表
 * - 每次请求时，将历史消息注入到 Prompt 中
 * - 响应返回后，将新的对话追加到历史记录
 * - 支持设置最大保留消息数，防止上下文过长
 *
 * 生产环境可替换为 RedisChatMemory 或 JdbcChatMemory
 * ============================================================
 */
@Slf4j
@Configuration
public class ChatMemoryService {

    @Value("${app.chat.max-history:20}")
    private int maxHistory;

    /**
     * 注册 ChatMemory Bean
     * Spring AI 1.0.0-M6 使用 InMemoryChatMemory
     * 每个 conversationId 对应一个独立的消息列表
     */
    @Bean
    @Primary
    public ChatMemory chatMemory() {
        log.info("📝 初始化 ChatMemory，最大保留消息数: {}", maxHistory);
        return new InMemoryChatMemory();
    }
}
```

### 5.5.2 ChatService

ChatService 聊天服务类

```java
package com.example.springaidemo.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

/**
 * ============================================================
 * 聊天服务 - 核心业务逻辑层
 * ============================================================
 *
 * 封装了与 ChatClient 的交互，提供：
 * - 普通对话（自动包含 RAG + Tool + Memory）
 * - 流式对话
 * - 带会话隔离的多轮对话
 * ============================================================
 */
@Slf4j
@Service
public class ChatService {

    private final ChatClient chatClient;

    public ChatService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    /**
     * ============================================================
     * 核心方法：发送消息并获取回复
     * ============================================================
     *
     * 当你调用这个方法时，底层完整流程：
     *
     * ① chatClient.prompt() → 创建 Prompt 构造器
     * ② .user(message)      → 添加用户消息
     * ③ .advisors(...)      → 动态设置会话ID（实现多用户隔离）
     * ④ .call()             → 冻结构造器，准备执行
     * ⑤ .content()          → 触发完整执行链：
     *     ├─ MessageChatMemoryAdvisor.before() → 注入历史
     *     ├─ QuestionAnswerAdvisor.before()   → RAG 检索
     *     ├─ ToolCallingAdvisor.before()      → 附加工具定义
     *     ├─ ChatModel.call()                 → 调用 LLM
     *     ├─ ToolCallingAdvisor.after()       → 如有 tool_call，执行并再次调用
     *     ├─ QuestionAnswerAdvisor.after()    → 无操作
     *     └─ MessageChatMemoryAdvisor.after() → 保存对话
     * ============================================================
     */
    public String chat(String conversationId, String message) {
        log.info("💬 [请求] 会话ID: {}, 消息: {}", conversationId, message);

        String response = chatClient.prompt()
                .user(message)
                // 通过 AdvisorContext 传递 conversationId
                // 这是实现多用户/多会话隔离的关键
                .advisors(a -> a
                        .param(ChatMemory.CONVERSATION_ID, conversationId)
                )
                .call()
                .content();

        log.info("💬 [响应] 会话ID: {}, 回复: {}", conversationId,
                response != null ? response.substring(0, Math.min(50, response.length())) + "..." : "null");
        return response;
    }

    /**
     * ============================================================
     * 流式对话 - 逐 token 返回 Flux<String>
     * ============================================================
     *
     * 流式模式下的 Tool Calling 处理差异：
     * - 第一次响应需要等完整 JSON 才能判断是否是 tool_call
     * - 因为 tool_call 的 function name 和 arguments 可能在最后一个 delta chunk 中
     * - ToolCallingAdvisor 内部会缓存 chunk 直到能确定响应类型
     * - 如果是 tool_call → 执行工具 → 二次调用（这次也是流式）
     * - 如果是普通文本 → 直接 emit 每个 token
     * ============================================================
     */
    public Flux<String> getFluxResponse(String conversationId, String message) {
        log.info("🌊 [流式请求] 会话ID: {}, 消息: {}", conversationId, message);

        return chatClient.prompt()
                .user(message)
                .advisors(a -> a
                        .param(ChatMemory.CONVERSATION_ID, conversationId)
                )
                .stream()
                .content();
    }

    /**
     * 清空指定会话的对话历史
     */
    public void clearHistory(String conversationId, ChatMemory chatMemory) {
        log.info("🗑️ 清空会话历史: {}", conversationId);
        chatMemory.clear(conversationId);
    }
}
```

### 5.5.3 KnowledgeBaseService

知识库服务服务

```java
package com.example.springaidemo.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.reader.tika.TikaDocumentReader;
import org.springframework.ai.vectorstore.SimpleVectorStore;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.stereotype.Service;

import jakarta.annotation.PostConstruct;
import java.io.IOException;
import java.util.List;

/**
 * ============================================================
 * 知识库服务 - 初始化 RAG 所需的向量存储
 * ============================================================
 *
 * RAG 原理（Retrieval-Augmented Generation）：
 *
 * 【离线阶段 - 应用启动时执行】
 * 1. 读取知识库文档（PDF、Word、Markdown等）
 * 2. 将文档切分成适当大小的文本块
 * 3. 使用 EmbeddingModel 将文本块转为向量
 * 4. 将向量和原文存储到 VectorStore
 *
 * 【在线阶段 - 每次请求时执行】
 * 1. 用户提问 → EmbeddingModel 转为查询向量
 * 2. VectorStore 进行相似度搜索，找到 Top-K 相关文档
 * 3. 将相关文档拼接到 Prompt 的 System Message 中
 * 4. LLM 基于这些上下文生成回答
 *
 * 这就是 QuestionAnswerAdvisor 的底层工作逻辑
 * ============================================================
 */
@Slf4j
@Service
public class KnowledgeBaseService {

    @Value("${app.knowledge-base.path}")
    private String knowledgeBasePath;

    private final EmbeddingModel embeddingModel;
    private VectorStore vectorStore;

    public KnowledgeBaseService(EmbeddingModel embeddingModel) {
        this.embeddingModel = embeddingModel;
    }

    /**
     * 应用启动时自动初始化知识库
     * 注意：生产环境应该只在知识库有更新时才重新索引
     */
    @PostConstruct
    public void initKnowledgeBase() {
        log.info("📚 开始初始化知识库...");
        try {
            // 1. 扫描知识库目录下的所有文档
            PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            Resource[] resources = resolver.getResources(knowledgeBasePath + "*");

            if (resources.length == 0) {
                log.warn("⚠️ 知识库路径 {} 下没有找到任何文档", knowledgeBasePath);
                // 创建空的向量存储，避免空指针
                this.vectorStore = SimpleVectorStore.builder(embeddingModel).build();
                return;
            }

            log.info("📚 找到 {} 个文档待处理", resources.length);

            for (Resource resource : resources) {
                log.info("📄 处理文档: {}", resource.getFilename());
            }

            // 2. 读取文档内容
            // TikaDocumentReader 支持 PDF、DOCX、PPTX、HTML 等格式
            TikaDocumentReader reader = new TikaDocumentReader(resources);
            List<Document> documents = reader.get();

            if (documents.isEmpty()) {
                log.warn("⚠️ 文档内容为空，跳过向量化");
                this.vectorStore = SimpleVectorStore.builder(embeddingModel).build();
                return;
            }

            log.info("📄 读取到 {} 个文档片段", documents.size());

            // 3. 文档切分
            // TokenTextSplitter 按 token 数量切分，避免超过模型上下文窗口
            TokenTextSplitter splitter = new TokenTextSplitter();
            List<Document> chunks = splitter.apply(documents);

            log.info("✂️ 切分为 {} 个文本块", chunks.size());

            // 为每个 chunk 添加元数据（来源文件）
            for (int i = 0; i < chunks.size(); i++) {
                Document chunk = chunks.get(i);
                chunk.getMetadata().put("chunk_index", i);
                chunk.getMetadata().put("source", "company-knowledge-base");
            }

            // 4. 创建向量存储并写入文档
            SimpleVectorStore.Builder builder = SimpleVectorStore.builder(embeddingModel);
            this.vectorStore = builder.build();
            this.vectorStore.add(chunks);

            log.info("✅ 知识库初始化完成！共索引 {} 个文本块", chunks.size());

        } catch (IOException e) {
            log.error("❌ 知识库初始化失败", e);
            // 即使失败也不阻塞应用启动，创建空的向量存储
            this.vectorStore = SimpleVectorStore.builder(embeddingModel).build();
        }
    }

    public VectorStore getVectorStore() {
        return vectorStore;
    }
}
```
## 5.6 Tools类

BusinessTools 业务工具类

```java
package com.example.springaidemo.tool;

import com.example.springaidemo.model.OrderInfo;
import com.example.springaidemo.model.WeatherInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;
import java.util.Random;

/**
 * ============================================================
 * 业务工具类 - 供 LLM 通过 Tool Calling 机制调用的工具
 * ============================================================
 *
 * 核心原理：
 * 1. @Tool 注解的方法会被 Spring AI 自动扫描，生成 JSON Schema
 * 2. Schema 会随 Prompt 一起发送给 LLM
 * 3. LLM 决定调用哪个工具时，返回 tool_call 指令
 * 4. Spring AI 的 ToolCallingAdvisor 拦截到 tool_call
 * 5. 通过反射调用对应的 Java 方法
 * 6. 方法返回值作为 ToolMessage 再次发送给 LLM
 * 7. LLM 拿到结果后生成自然语言回答
 *
 * 注意：LLM 永远不会直接执行 Java 代码，它只发送"调用请求"
 *       真正的方法执行权完全掌握在应用侧
 * ============================================================
 */
@Slf4j
@Component
public class BusinessTools {

    // ==================== 模拟数据库 ====================

    private static final Map<String, OrderInfo> ORDER_DB = new HashMap<>();
    private static final Map<String, WeatherInfo> WEATHER_DB = new HashMap<>();
    private static final Random RANDOM = new Random();

    static {
        // 初始化模拟订单数据
        ORDER_DB.put("ORD-1001", new OrderInfo(
                "ORD-1001", "已发货", "iPhone 16 Pro Max",
                "北京市朝阳区xxx街道", "2025-01-20", 9999.00
        ));
        ORDER_DB.put("ORD-1002", new OrderInfo(
                "ORD-1002", "已签收", "AirPods Pro 3",
                "上海市浦东新区xxx路", "2025-01-15", 1899.00
        ));
        ORDER_DB.put("ORD-1003", new OrderInfo(
                "ORD-1003", "备货中", "MacBook Pro M5",
                "深圳市南山区xxx大道", "预计2025-01-25", 19999.00
        ));

        // 初始化模拟天气数据
        WEATHER_DB.put("北京", new WeatherInfo("北京", "晴", 22, "45%", "东北风3级"));
        WEATHER_DB.put("上海", new WeatherInfo("上海", "多云", 18, "70%", "东南风2级"));
        WEATHER_DB.put("深圳", new WeatherInfo("深圳", "阵雨", 26, "85%", "南风4级"));
        WEATHER_DB.put("广州", new WeatherInfo("广州", "阴", 24, "78%", "北风1级"));
    }

    // ==================== 天气查询工具 ====================

    @Tool(description = "查询指定城市的当前天气信息，包括天气状况、温度、湿度、风力等")
    public WeatherInfo getWeather(
            @ToolParam(description = "城市名称，如：北京、上海、深圳") String city) {
        log.info("🌤️ [Tool调用] 查询天气 - 城市: {}", city);

        // 模拟网络延迟
        simulateDelay(300);

        WeatherInfo info = WEATHER_DB.get(city);
        if (info == null) {
            // 未收录的城市，返回模拟数据
            info = new WeatherInfo(
                    city, "多云", 15 + RANDOM.nextInt(15),
                    (30 + RANDOM.nextInt(50)) + "%",
                    "微风"
            );
        }
        log.info("🌤️ [Tool返回] {}", info);
        return info;
    }

    // ==================== 订单查询工具 ====================

    @Tool(description = "根据订单号查询订单状态、商品名称、配送地址、预计到货时间等信息")
    public OrderInfo getOrderStatus(
            @ToolParam(description = "订单号，格式如 ORD-1001") String orderId) {
        log.info("📦 [Tool调用] 查询订单 - 订单号: {}", orderId);

        // 模拟数据库查询延迟
        simulateDelay(200);

        OrderInfo order = ORDER_DB.get(orderId);
        if (order == null) {
            log.info("📦 [Tool返回] 订单不存在: {}", orderId);
            return new OrderInfo(orderId, "未找到该订单", "", "", "", 0);
        }
        log.info("📦 [Tool返回] {}", order);
        return order;
    }

    // ==================== 辅助方法 ====================

    private void simulateDelay(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

## 5.7 controller 类

```java
package com.example.springaidemo.controller;

import com.example.springaidemo.service.ChatService;
import com.example.springaidemo.service.ChatMemoryService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

/**
 * ============================================================
 * REST 控制器 - 提供 HTTP 接口
 * ============================================================
 *
 * 接口列表：
 * - POST /api/chat          → 普通对话（同步）
 * - GET  /api/chat/stream   → 流式对话（SSE）
 * - POST /api/chat/clear    → 清空会话历史
 * - GET  /api/chat/new-id   → 生成新的会话ID
 *
 * 测试场景：
 * 1. 多轮对话：/api/chat/new-id 获取 ID → 连续发多条消息测试记忆
 * 2. Tool Calling：问"北京天气怎么样"、"查询订单 ORD-1001"
 * 3. RAG：问知识库相关的问题（需先放文档到 resources/knowledge/）
 * ============================================================
 */
@Slf4j
@RestController
@RequestMapping("/api/chat")
public class ChatController {

    @Autowired
    private ChatService chatService;

    @Autowired
    private ChatMemory chatMemory;

    /**
     * 普通对话接口
     */
    @PostMapping
    public Map<String, Object> chat(@RequestBody Map<String, String> request) {
        String conversationId = request.getOrDefault("conversationId",
                UUID.randomUUID().toString());
        String message = request.get("message");

        if (message == null || message.trim().isEmpty()) {
            return Map.of("error", "消息不能为空");
        }

        long startTime = System.currentTimeMillis();
        String response = chatService.chat(conversationId, message);
        long elapsedTime = System.currentTimeMillis() - startTime;

        Map<String, Object> result = new HashMap<>();
        result.put("conversationId", conversationId);
        result.put("response", response);
        result.put("elapsedTime", elapsedTime + "ms");
        return result;
    }

    /**
     * 流式对话接口（SSE）
     */
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public org.reactivestreams.Publisher<String> streamChat(
            @RequestParam String conversationId,
            @RequestParam String message) {

        return chatService.getFluxResponse(conversationId, message);
    }

    /**
     * 清空会话历史
     */
    @PostMapping("/clear")
    public Map<String, Object> clearHistory(@RequestBody Map<String, String> request) {
        String conversationId = request.get("conversationId");
        if (conversationId != null) {
            chatService.clearHistory(conversationId, chatMemory);
            return Map.of("success", true, "message", "会话历史已清空");
        }
        return Map.of("success", false, "message", "conversationId 不能为空");
    }

    /**
     * 生成新的会话ID
     */
    @GetMapping("/new-id")
    public Map<String, String> newConversationId() {
        return Map.of("conversationId", UUID.randomUUID().toString());
    }
}
```

## 6 测试

接口基本信息

```
URL: http://localhost:8080/api/chat
Method: POST
Content-Type: application/json
Request Body:

{
  "message": "你的问题"
}

Response (成功):

{
  "reply": "助手的回答内容"
}
```

###  6.1 场景一：普通对话（无 RAG、无 Tool）

```
POST http://localhost:8080/api/chat
Content-Type: application/json

{
  "message": "介绍一下你自己"
}
```

预期响应：
```
{
  "reply": "我是 AI 助手，由 Spring AI 驱动，可以回答各种问题。"
}
```

###  6.2 场景二：RAG 检索增强（知识库问答）

前提：项目启动时会自动将 resources/knowledge/products.txt 中的内容索引到内存向量库。

请求:
```
POST http://localhost:8080/api/chat
Content-Type: application/json

{
"message": "你们的会员等级有哪些？分别有什么权益？"
}
```
预期响应（基于 products.txt 中的内容，例如）：
```
{
"reply": "我们的会员等级分为普通会员、银卡会员和金卡会员。\n- 普通会员：享受基础折扣 5%。\n- 银卡会员：享受 10% 折扣，生日月双倍积分。\n- 金卡会员：享受 15% 折扣，专属客服，免费升级快递。"
}
```

说明：QuestionAnswerAdvisor 会从向量库检索相关文档片段，拼入 Prompt 后让模型生成答案。

###  6.3  场景三：Tool Calling（工具调用）

请求:
```
POST http://localhost:8080/api/chat
Content-Type: application/json

{
"message": "北京今天天气怎么样？顺便帮我查一下订单 ORD-1001 的状态"
}
```
预期响应（工具执行后生成的回答）：
```
{
"reply": "北京今天天气：晴朗，22°C，适合外出。\n订单 ORD-1001 状态：已发货，预计明天下午送达。"
}
```
说明：模型会依次（或并行）调用 getWeather 和 getOrderStatus 两个工具，拿到结果后再组织最终回答。

###  6.4 场景四（可选）：流式输出


如果需要流式响应，可使用以下端点（已在 Controller 中实现）：

请求:
```
GET http://localhost:8080/api/chat/stream?message=讲个笑话
```


响应：SSE 格式，逐 token 推送，浏览器或 EventSource 客户端可直接消费。

###  6.5  Mock 测试

```java
@SpringBootTest
@AutoConfigureMockMvc
class ChatControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void testNormalChat() throws Exception {
        String requestJson = "{\"message\":\"你好\"}";

        mockMvc.perform(post("/api/chat")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.reply").isNotEmpty());
    }

    @Test
    void testToolCalling() throws Exception {
        String requestJson = "{\"message\":\"北京天气怎么样？查订单 ORD-1001\"}";

        mockMvc.perform(post("/api/chat")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.reply").value(containsString("天气")))
                .andExpect(jsonPath("$.reply").value(containsString("订单")));
    }
}
```