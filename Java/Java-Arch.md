---
layout: default
title: Java Arch
has_children: true
nav_order: 10
---


 Here are commonly used Java examples .
{: .fs-6 .fw-300 }

### 为什么要演进架构

架构不是设计出来的，⽽是"⻓"出来的。每⼀次架构升级，背后都是业务规模和技术约束的变化。

Java 应⽤部署架构的演进，本质上是围绕四个核⼼⽭盾展开：
- 规模⽭盾：⽤⼾量、数据量、请求量呈指数增⻓，单机性能天花板被不断突破。
- 交付⽭盾：业务迭代速度加快，发布频率从"⽉"到"天"甚⾄"⼩时"，⼈⼯部署成为瓶颈。
- 可⽤⽭盾：单点故障代价越来越⾼，系统需要⾃愈、容灾、多活。
- 成本⽭盾：资源利⽤率低导致浪费，需要在弹性伸缩与成本控制间找到平衡。

演进路线总览

![架构演进](https://docs.zhangxiaocai.cn/Assets/images/arch/arch1.png)

架构演进不是"⾮此即彼"的替换，⽽是层层叠加、平滑过渡。⼤多数企业会⻓期共存多个阶段的架构，关
键是明确每个阶段的适⽤边界。

### 第一阶段 单体应用 + 物理机部署

最原始也最朴素：⼀个 War/Jar 包，部署在⼀台物理服务器上，所有功能模块耦合在⼀起。

![单体架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch2.png)

适⽤场景与痛点

单体物理机部署的优劣分析

|维度 |优势 | 痛点                |
|----|----|-------------------|
|开发 |结构简单，IDE 直接运⾏，调试⽅便 | 代码耦合严重，模块边界模糊     |
|部署 |⼀个包搞定，⽆需复杂编排 | 全量发布，任⼀模块改动需整体重启  |
|性能 |本地⽅法调⽤，⽆⽹络开销 | 单 JVM 内存上限，⽆法⽔平扩展 |
|可⽤性 |故障⾯⼩，排查链路短 | 单点故障，宕机即全站不可⽤     |
|成本 |硬件投⼊固定，运维⼈⼒少 | 资源利⽤率低，⾼峰期束⼿无策    |

>典型适用：
> 
>创业初期，内部⼯具、流量稳定的⼩型系统（⽇ PV  10 万）。当团队 < 10 ⼈、功能模块 < 20 个时，单
体架构是最经济合理的选择。

### 第二阶段 单体应⽤ + 虚拟化 / 集群部署

架构形态：

引⼊物理服务器集群 + 前置负载均衡，通过多实例部署提升并发能⼒和可⽤性。

![单体+集群架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch3.png)

关键技术点

- 负载均衡策略：轮询、IP Hash、最少连接数
- Session 共享：Redis 集中存储 / Spring Session
- 动静分离：Nginx 直接处理静态资源
- 数据库读写分离：主库写、从库读

>本质局限:
> 
>集群化解决了"能扛多少"的问题，但没有解决"能多快交付"的问题。所有实例仍是同⼀份代码，任何⼀个
Bug 会导致全集群故障。


### 第三阶段 垂直拆分 + 反向代理

架构形态：

按业务域将单体拆分为多个独⽴应⽤，各⾃部署在独⽴服务器上，通过反向代理统⼀⼊⼝。

![垂直拆分架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch4.png)

核⼼价值与代价

|价值 |代价|
|----|----|
|不同业务可按需独⽴扩容 |跨库事务难以保证|
|故障隔离，单模块宕机不影响全局 |模块间调⽤从⽅法变为 HTTP/RPC|
|团队可独⽴开发部署 |需要统⼀ API ⽹关规范|
|技术栈可按业务选择 |数据⼀致性复杂度上升|


### 第四阶段 SOA 服务化架构

架构形态：

引⼊企业服务总线（ESB），将公共能⼒抽象为可复⽤的服务，通过标准协议（SOAP/REST）进⾏服务编排。

![SOA服务化架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch5.png)

关键组件
- ESB（Enterprise Service Bus）：异构系统集成的"中枢神经"，负责协议适配、消息转换、路由分发
- 服务注册中⼼：如 ZooKeeper，管理服务地址
- 消息中间件：ActiveMQ / RabbitMQ，异步解耦

> SOA 的局限:
> 
> ESB 逐渐成为性能瓶颈和单点故障源；WebService 协议（SOAP）笨重、开发体验差。这正是微服务架构
兴起的历史原因。


### 第五阶段  微服务 + 容器化（Docker + K8s）

架构形态：

服务粒度更细、去中⼼化治理，配合容器编排实现弹性伸缩和⾃动化运维。

![微服务 + 容器化架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch6.png)

核⼼技术栈

|层次 |技术选型 |作⽤|
|-----|-------|---|
|服务框架 |Spring Boot 3.x + Spring Cloud Alibaba |快速开发 + 微服务全家桶|
|注册配置 |Nacos / Consul |服务注册发现 + 动态配置|
|API ⽹关 |Spring Cloud Gateway / APISIX |统⼀⼊⼝、路由、限流|
|负载均衡 |OpenFeign + Spring Cloud LoadBalancer |声明式服务调⽤|
|熔断限流 |Sentinel / Resilience4j |流量控制、熔断降级|
|链路追踪 |SkyWalking / OpenTelemetry |分布式链路可视化|
|消息队列 |RocketMQ / Kafka |异步解耦、事件驱动|
|容器化 |Docker + containerd |环境⼀致性、快速交付|
|编排调度 |Kubernetes + Helm |⾃动伸缩、滚动更新、⾃愈|
|CI/CD |Jenkins / GitLab CI / ArgoCD |流⽔线⾃动化|
|可观测 |Prometheus + Grafana + Loki |指标/⽇志/链路三位⼀体|

DockerFile

```
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-XX:+UseZGC","-jar","app.jar"]
```

Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: order-service
spec:
    replicas: 3
    selector:
        matchLabels: { app: order }
    template:
      spec:
        containers:
          - name: order
        image: registry.example.com/order:v1.2.0
        resources:
          requests: { cpu: "500m", memory: "512Mi" }
          limits: { cpu: "2000m", memory: "2Gi" }
        readinessProbe:
          httpGet: { path: /actuator/health, port: 8080 }
```

### 第六阶段  Service Mesh 服务⽹格

架构形态：

将服务间通信的治理逻辑从应⽤代码中剥离，下沉到基础设施层（Sidecar 代理），实现业务代码与治理逻辑彻底解
耦。

![微服务 + 容器化架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch7.png)

核⼼价值
- 语⾔⽆关：治理逻辑下沉到 Sidecar，Java 应⽤⽆需引⼊ SDK
- 精细化流量管理：⾦丝雀发布、A/B 测试、流量镜像
- 零信任安全：⾃动 mTLS 双向认证、细粒度授权
- 可观测性：⾃动⽣成服务拓扑和⻩⾦指标

> 实施⻔槛：
> 
> Service Mesh 引⼊额外的⽹络跳数和运维复杂度。对中⼩规模集群（< 50 个服务），Spring Cloud 全家
桶的性价⽐通常更⾼。
>

### 第七阶段 Serverless / 函数计算

架构形态：

开发者只关注代码逻辑，完全不感知服务器。应⽤按请求⾃动伸缩，缩容到零，按实际⽤量计费。

![Serverless 架构](https://docs.zhangxiaocai.cn/Assets/images/arch/arch8.png)

Serverless 适⽤场景分析


| ✅ 适合 | ❌ 不适合|
|---------|---------|
|事件驱动任务（⽂件处理、消息消费） |⻓连接 / WebSocket 服务|
|流量波动⼤、峰⾕差异明显的场景 |有状态、需要本地缓存的服务|
|定时批处理任务 |对冷启动极度敏感的实时交易|
|轻量级 API 后端 |需要精细控制 JVM 参数的场景|

>Java 在 Serverless 中的挑战:
> 
>JVM 启动慢、内存占⽤⼤，在冷启动场景下不如 Go/Node.js/Python。解决⽅案：GraalVM Native
Image 编译为原⽣可执⾏⽂件，可将启动时间从秒级降⾄毫秒级。


### 各阶段横向对⽐与选型建议

七个阶段架构对⽐

|维度 | 单体物理机  | 虚拟化集群 |垂直拆分 |SOA |微服务+容器 |Service Mesh | Serverless |
|----|--------|-------|--------|----|--------|--------|-----|
|复杂度 | ★      | ★★    |★★★ |★★★★ |★★★★★ |★★★★★ |★★★|
|弹性能⼒ | ⽆      | 有限    |模块级 |服务级 |实例级 |实例级 |请求级|
|交付速度 | 慢      | 慢     |中 |中 |快 |快 |极快|
|运维成本 | 低      | 中     |中 |⾼ |⾼ |极⾼ |低|
|资源利⽤率 | 低      | 中     |中 |中 |⾼ |⾼ |极⾼|
|团队规模适配 | 大于10⼈  | 大于20⼈ |20-50⼈ |50-100⼈ |100⼈+ |200⼈+ |任意|
|典型技术 | Tomcat | Nginx |HTTP/RPC |ESB |K8s |Istio |Knative|


选型决策树

>选型逻辑
>
>团队 < 10 ⼈ + ⽇活 < 10 万 → 单体 + 集群即可
> 
>10-50 ⼈ + 多业务域 → 垂直拆分 / 轻量微服务
>
>50-200 ⼈ + ⾼并发 → 微服务 + Docker + K8s（推荐基线）
> 
>200 ⼈+ + 多语⾔混合 → 引⼊ Service Mesh
> 
>事件驱动 + 流量波峰明显 → Serverless 补充

### 演进策略与落地路径

- 渐进式演进：不要⼀次性重写，采⽤绞杀者模式（Strangler Fig）逐步替换
- 数据先⾏：先拆分数据库，再拆分应⽤，避免分布式单体
- 可观测先⾏：在拆分前先建⽴监控和链路追踪，否则排查问题会陷⼊地狱
- 组织适配：康威定律——架构反映组织结构，微服务需要⼩团队⾃治


架构演进的本质是在复杂度、成本、速度之间寻找当前阶段的最优解。不存在"最好的架构"，只有"最适合
当前团队和业务阶段的架构"。 好的架构师不是追求技术时髦，⽽是让架构适度领先业务半步——既不过度
设计，也不成为瓶颈。


**常见的反模式**

演进中的典型反模式

|反模式 |问题 |应对|
|------|-----|---|
|分布式单体 |服务间强耦合，⼀个服务挂全挂 |异步事件驱动 + 定义清晰契约|
|过早微服务化 |团队不⾜却被微服务复杂度拖垮 |先⽤模块化单体，等规模到了再拆|
|数据库共享 |服务间通过 DB 耦合，失去独⽴性 |每服务独⽴库，⾛ API 交互|
|忽视可观测 |出问题靠猜，排查时间指数增⻓ |先建监控、⽇志、链路三件套|

还有一些需要结合业务特征在不同架构阶段进行局部调整适应行业需要。