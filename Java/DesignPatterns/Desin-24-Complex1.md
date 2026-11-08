---
layout: default
title: Strategy & Factory
parent: Design Patterns
has_children: false
nav_order: 34
---

Here are java Strategy & Factory Design Pattern experience .
{: .fs-6 .fw-300 }

1. TOC
{:toc}


## 策略模式 + 工厂模式

常见应用场景

需要根据运行时条件（如用户等级、促销活动）动态选择算法，且算法种类可能频繁增加。工厂负责创建策略对象，策略负责执行算法，二者结合实现“算法选择与创建”的解耦。

策略和实现：
```java
// ========== 场景一：折扣策略 ==========
public interface DiscountStrategy {
    double apply(double price);
}
// ===========具体的折扣计算
public class NewUserDiscount implements DiscountStrategy {
    public double apply(double p) { return p * 0.85; }
}
public class VIPDiscount implements DiscountStrategy {
    public double apply(double p) { return p * 0.70; }
}
public class FestivalDiscount implements DiscountStrategy {
    public double apply(double p) { return p * 0.80; }
}

```

策略工厂：
```java
// ========== 通用工厂（用泛型实现不同策略族的创建） ==========
class StrategyFactory<T> {
    private Map<String, T> strategies = new HashMap<>();

    public void register(String key, T strategy) {
        strategies.put(key, strategy);
    }

    public T getStrategy(String key) {
        T s = strategies.get(key);
        if (s == null) throw new IllegalArgumentException("Unknown strategy: " + key);
        return s;
    }
}
```

注册：
```java
// ========== 初始化工厂并注册策略 ==========
public class AppConfig {
    // 折扣策略工厂
    public static StrategyFactory<DiscountStrategy> discountFactory(){
        StrategyFactory<DiscountStrategy> factory = new StrategyFactory<>();
        factory.register("NEW_USER", new NewUserDiscount());
        factory.register("VIP", new VIPDiscount());
        factory.register("FESTIVAL", new FestivalDiscount());
        return factory;
    }
}
```

订单上下文
```java
// ========== 上下文1：订单 ==========
public class Order {
    private double amount;
    private StrategyFactory<DiscountStrategy> discountFactory;

    public Order(double amount, StrategyFactory<DiscountStrategy> df) {
        this.amount = amount;
        this.discountFactory = df;
    }

    public double pay(String couponType) {
        DiscountStrategy strategy = discountFactory.getStrategy(couponType);
        return strategy.apply(amount);
    }
}
```

客户端
```java

// ========== 客户端 ==========
public class ScenarioDemo {
    public static void main(String[] args) {
        StrategyFactory<DiscountStrategy> discountFactory = AppConfig.discountFactory();
        // 场景1：订单折扣
        Order order = new Order(200, discountFactory);
        System.out.println("订单折扣:");
        System.out.println("  NEW_USER: " + order.pay("NEW_USER")); // 170
        System.out.println("  VIP:      " + order.pay("VIP"));      // 140
        System.out.println("  FESTIVAL: " + order.pay("FESTIVAL")); // 160

    }
}
```
输出：
```text
订单折扣:
  NEW_USER: 170.0
  VIP:      140.0
  FESTIVAL: 160.0

```

## 其他场景

### 场景 1：多渠道支付网关

- 背景：电商平台对接支付宝、微信、银联、PayPal 等多种支付渠道。
- 策略角色：每种支付渠道封装为一个策略，包含下单、退款、查询接口。
- 工厂角色：根据支付方式代码（alipay、wxpay、unionpay）动态创建对应支付策略对象。工厂可集成第三方 SDK 初始化逻辑（如密钥加载、证书管理）。
- 关键差异：不同策略的接口签名一致，但内部调用的外部 API 完全不同（REST、SOAP、自定义协议）。

```java
import java.util.HashMap;
import java.util.Map;

// 策略接口
interface PaymentStrategy {
    boolean pay(double amount);
    boolean refund(String orderId);
}

// 具体策略：支付宝
class AlipayStrategy implements PaymentStrategy {
    public boolean pay(double amount) {
        System.out.println("支付宝支付 " + amount + " 元");
        return true;
    }
    public boolean refund(String orderId) {
        System.out.println("支付宝退款 订单:" + orderId);
        return true;
    }
}

// 具体策略：微信支付
class WechatPayStrategy implements PaymentStrategy {
    public boolean pay(double amount) {
        System.out.println("微信支付 " + amount + " 元");
        return true;
    }
    public boolean refund(String orderId) {
        System.out.println("微信退款 订单:" + orderId);
        return true;
    }
}

// 具体策略：银联支付
class UnionPayStrategy implements PaymentStrategy {
    public boolean pay(double amount) {
        System.out.println("银联支付 " + amount + " 元");
        return true;
    }
    public boolean refund(String orderId) {
        System.out.println("银联退款 订单:" + orderId);
        return true;
    }
}

// 工厂
class PaymentFactory {
    private static final Map<String, PaymentStrategy> strategies = new HashMap<>();
    static {
        strategies.put("alipay", new AlipayStrategy());
        strategies.put("wechat", new WechatPayStrategy());
        strategies.put("unionpay", new UnionPayStrategy());
    }
    public static PaymentStrategy getPayment(String channel) {
        PaymentStrategy s = strategies.get(channel.toLowerCase());
        if (s == null) throw new IllegalArgumentException("Unsupported channel: " + channel);
        return s;
    }
}

// 客户端
public class PaymentDemo {
    public static void main(String[] args) {
        PaymentStrategy payment = PaymentFactory.getPayment("alipay");
        payment.pay(199.99);
        payment.refund("ORD12345");
    }
}
```
### 场景 2：多通道消息推送

- 背景：运营系统需要向用户推送通知，支持短信、邮件、App Push、微信公众号、钉钉机器人。
- 策略角色：每种推送方式是一个策略，统一提供 send(userId, message) 方法。
- 工厂角色：根据用户偏好或渠道权重选择策略（如重要通知走短信+邮件，一般通知走 App Push）。工厂可维护渠道健康状态（熔断降级）。
- 关键差异：不同策略的可靠性、成本、送达率差异巨大，工厂需要结合业务规则决策。

```java
import java.util.HashMap;
import java.util.Map;

interface NotifyStrategy {
    void send(String userId, String message);
}

class SmsNotify implements NotifyStrategy {
    public void send(String userId, String message) {
        System.out.println("短信推送给用户 " + userId + ": " + message);
    }
}

class EmailNotify implements NotifyStrategy {
    public void send(String userId, String message) {
        System.out.println("邮件推送给用户 " + userId + ": " + message);
    }
}

class AppPushNotify implements NotifyStrategy {
    public void send(String userId, String message) {
        System.out.println("App Push 推送给用户 " + userId + ": " + message);
    }
}

class NotifyFactory {
    private static final Map<String, NotifyStrategy> map = new HashMap<>();
    static {
        map.put("sms", new SmsNotify());
        map.put("email", new EmailNotify());
        map.put("apppush", new AppPushNotify());
    }
    public static NotifyStrategy getNotifier(String type) {
        return map.get(type.toLowerCase());
    }
}

public class NotifyDemo {
    public static void main(String[] args) {
        NotifyStrategy notifier = NotifyFactory.getNotifier("sms");
        notifier.send("user001", "您的验证码是123456");
    }
}
```

### 场景 3：多格式数据导出

- 背景：报表系统支持导出为 PDF、Excel、CSV、JSON、HTML。
- 策略角色：每种导出格式是一个策略，实现 export(data) -> File。
- 工厂角色：根据用户选择的格式名称创建对应策略。工厂还可注入公共配置（如页眉页脚、字体、水印）。
- 关键差异：PDF 需要布局引擎，Excel 需要行列处理，CSV 最简单，JSON 面向 API 消费。策略内部依赖库完全不同。

```java
import java.util.*;

interface ExportStrategy {
    byte[] export(List<String[]> data);
}

class PdfExport implements ExportStrategy {
    public byte[] export(List<String[]> data) {
        System.out.println("导出为 PDF，共 " + data.size() + " 行数据");
        return new byte[]{'P','D','F'};
    }
}

class ExcelExport implements ExportStrategy {
    public byte[] export(List<String[]> data) {
        System.out.println("导出为 Excel，共 " + data.size() + " 行数据");
        return new byte[]{'X','L','S','X'};
    }
}

class CsvExport implements ExportStrategy {
    public byte[] export(List<String[]> data) {
        System.out.println("导出为 CSV，共 " + data.size() + " 行数据");
        return new byte[]{'C','S','V'};
    }
}

class ExportFactory {
    private static final Map<String, ExportStrategy> map = new HashMap<>();
    static {
        map.put("pdf", new PdfExport());
        map.put("excel", new ExcelExport());
        map.put("csv", new CsvExport());
    }
    public static ExportStrategy getExporter(String format) {
        return map.get(format.toLowerCase());
    }
}

public class ExportDemo {
    public static void main(String[] args) {
        List<String[]> data = Arrays.asList(
            new String[]{"name","age"},
            new String[]{"Alice","30"}
        );
        ExportStrategy exporter = ExportFactory.getExporter("pdf");
        exporter.export(data);
    }
}
```

### 场景 4：多后端文件存储

- 背景：云存储服务支持本地磁盘、阿里云 OSS、AWS S3、MinIO。
- 策略角色：每种存储后端是一个策略，提供 upload(file), download(path), delete(path)。
- 工厂角色：根据配置（local、oss、s3）创建存储策略。工厂负责初始化客户端连接池、重试策略。
- 关键差异：本地存储走 IO，OSS 走 HTTP，S3 走 AWS SDK，MinIO 兼容 S3 但需单独配置 endpoint。

```java
import java.util.*;

interface StorageStrategy {
    void upload(String path, byte[] content);
    byte[] download(String path);
    void delete(String path);
}

class LocalStorage implements StorageStrategy {
    public void upload(String path, byte[] content) {
        System.out.println("本地存储上传: " + path + " (" + content.length + " bytes)");
    }
    public byte[] download(String path) {
        System.out.println("本地存储下载: " + path);
        return new byte[0];
    }
    public void delete(String path) {
        System.out.println("本地存储删除: " + path);
    }
}

class OssStorage implements StorageStrategy {
    public void upload(String path, byte[] content) {
        System.out.println("阿里云OSS上传: " + path);
    }
    public byte[] download(String path) {
        System.out.println("阿里云OSS下载: " + path);
        return new byte[0];
    }
    public void delete(String path) {
        System.out.println("阿里云OSS删除: " + path);
    }
}

class S3Storage implements StorageStrategy {
    public void upload(String path, byte[] content) {
        System.out.println("AWS S3上传: " + path);
    }
    public byte[] download(String path) {
        System.out.println("AWS S3下载: " + path);
        return new byte[0];
    }
    public void delete(String path) {
        System.out.println("AWS S3删除: " + path);
    }
}

class StorageFactory {
    private static final Map<String, StorageStrategy> map = new HashMap<>();
    static {
        map.put("local", new LocalStorage());
        map.put("oss", new OssStorage());
        map.put("s3", new S3Storage());
    }
    public static StorageStrategy getStorage(String backend) {
        return map.get(backend.toLowerCase());
    }
}

public class StorageDemo {
    public static void main(String[] args) {
        StorageStrategy storage = StorageFactory.getStorage("oss");
        storage.upload("/images/logo.png", "fake-image-content".getBytes());
        storage.download("/images/logo.png");
        storage.delete("/images/logo.png");
    }
}
```

### 场景 5：多因素身份认证

- 背景：登录系统支持密码、短信验证码、邮箱验证码、生物识别（指纹/人脸）、OAuth2.0（微信/Google）。
- 策略角色：每种认证方式是一个策略，实现 authenticate(credentials) -> AuthResult。
- 工厂角色：根据用户选择的认证类型（password、sms、oauth_wechat）创建策略。工厂可串联多个策略（多因素认证）。
- 关键差异：密码策略需要哈希比对，短信策略需要调用第三方短信服务，OAuth 需要跳转授权页面。

```java
import java.util.*;

interface AuthStrategy {
    boolean authenticate(Map<String, String> credentials);
}

class PasswordAuth implements AuthStrategy {
    public boolean authenticate(Map<String, String> creds) {
        String user = creds.get("username");
        String pass = creds.get("password");
        System.out.println("密码认证: " + user + "/" + pass);
        return "admin".equals(user) && "123456".equals(pass);
    }
}

class SmsAuth implements AuthStrategy {
    public boolean authenticate(Map<String, String> creds) {
        String phone = creds.get("phone");
        String code = creds.get("code");
        System.out.println("短信验证码认证: " + phone + " 验证码=" + code);
        return "13800000000".equals(phone) && "666666".equals(code);
    }
}

class OAuthAuth implements AuthStrategy {
    public boolean authenticate(Map<String, String> creds) {
        String provider = creds.get("provider");
        String token = creds.get("token");
        System.out.println("OAuth认证: " + provider + " token=" + token);
        return !token.isEmpty();
    }
}

class AuthFactory {
    private static final Map<String, AuthStrategy> map = new HashMap<>();
    static {
        map.put("password", new PasswordAuth());
        map.put("sms", new SmsAuth());
        map.put("oauth", new OAuthAuth());
    }
    public static AuthStrategy getAuth(String type) {
        return map.get(type.toLowerCase());
    }
}

public class AuthDemo {
    public static void main(String[] args) {
        AuthStrategy auth = AuthFactory.getAuth("password");
        Map<String, String> creds = new HashMap<>();
        creds.put("username", "admin");
        creds.put("password", "123456");
        System.out.println("认证结果: " + auth.authenticate(creds));
    }
}
```


### 场景 6：多压缩算法

- 背景：文件上传服务需要自动压缩图片/文档，支持 GZip、Brotli、Zstd、LZ4。
- 策略角色：每种压缩算法是一个策略，实现 compress(bytes) -> bytes。
- 工厂角色：根据文件类型或客户端 Accept-Encoding 头选择压缩策略。工厂可缓存策略实例（算法无状态）。
- 关键差异：GZip 兼容性好，Brotli 压缩率高但慢，Zstd 平衡，LZ4 极快但压缩比低。

```java
import java.util.*;

interface CompressStrategy {
    byte[] compress(byte[] data);
    byte[] decompress(byte[] compressed);
}

class GzipCompress implements CompressStrategy {
    public byte[] compress(byte[] data) {
        System.out.println("GZip压缩: " + data.length + " -> ...");
        return data; // 模拟
    }
    public byte[] decompress(byte[] compressed) {
        System.out.println("GZip解压");
        return compressed;
    }
}

class BrotliCompress implements CompressStrategy {
    public byte[] compress(byte[] data) {
        System.out.println("Brotli压缩: " + data.length + " -> ...");
        return data;
    }
    public byte[] decompress(byte[] compressed) {
        System.out.println("Brotli解压");
        return compressed;
    }
}

class ZstdCompress implements CompressStrategy {
    public byte[] compress(byte[] data) {
        System.out.println("Zstd压缩: " + data.length + " -> ...");
        return data;
    }
    public byte[] decompress(byte[] compressed) {
        System.out.println("Zstd解压");
        return compressed;
    }
}

class CompressFactory {
    private static final Map<String, CompressStrategy> map = new HashMap<>();
    static {
        map.put("gzip", new GzipCompress());
        map.put("brotli", new BrotliCompress());
        map.put("zstd", new ZstdCompress());
    }
    public static CompressStrategy getCompressor(String algo) {
        return map.get(algo.toLowerCase());
    }
}

public class CompressDemo {
    public static void main(String[] args) {
        CompressStrategy compressor = CompressFactory.getCompressor("gzip");
        byte[] original = "Hello World!".getBytes();
        byte[] compressed = compressor.compress(original);
        compressor.decompress(compressed);
    }
}
```

### 场景 7：多序列化协议

- 背景：微服务间 RPC 通信支持 JSON、Protobuf、Avro、Kryo、MessagePack。
- 策略角色：每种序列化方式是一个策略，实现 serialize(obj) -> byte[] 和 deserialize(bytes, class) -> obj。
- 工厂角色：根据服务配置或协商结果（如 HTTP Header Content-Type: application/x-protobuf）创建序列化策略。
- 关键差异：JSON 可读但慢，Protobuf 高效但需预编译 schema，Kryo 快但不跨语言。

```java
import java.util.*;

interface SerializerStrategy {
    byte[] serialize(Object obj);
    <T> T deserialize(byte[] data, Class<T> clazz);
}

class JsonSerializer implements SerializerStrategy {
    public byte[] serialize(Object obj) {
        System.out.println("JSON序列化: " + obj);
        return obj.toString().getBytes();
    }
    @SuppressWarnings("unchecked")
    public <T> T deserialize(byte[] data, Class<T> clazz) {
        System.out.println("JSON反序列化");
        return (T) new String(data);
    }
}

class ProtobufSerializer implements SerializerStrategy {
    public byte[] serialize(Object obj) {
        System.out.println("Protobuf序列化: " + obj);
        return new byte[]{0x0A, 0x05};
    }
    @SuppressWarnings("unchecked")
    public <T> T deserialize(byte[] data, Class<T> clazz) {
        System.out.println("Protobuf反序列化");
        return (T) ("ProtoObj:" + data.length);
    }
}

class KryoSerializer implements SerializerStrategy {
    public byte[] serialize(Object obj) {
        System.out.println("Kryo序列化: " + obj);
        return new byte[]{0x01, 0x02};
    }
    @SuppressWarnings("unchecked")
    public <T> T deserialize(byte[] data, Class<T> clazz) {
        System.out.println("Kryo反序列化");
        return (T) ("KryoObj:" + data.length);
    }
}

class SerializerFactory {
    private static final Map<String, SerializerStrategy> map = new HashMap<>();
    static {
        map.put("json", new JsonSerializer());
        map.put("protobuf", new ProtobufSerializer());
        map.put("kryo", new KryoSerializer());
    }
    public static SerializerStrategy getSerializer(String protocol) {
        return map.get(protocol.toLowerCase());
    }
}

public class SerializeDemo {
    public static void main(String[] args) {
        SerializerStrategy serializer = SerializerFactory.getSerializer("protobuf");
        byte[] data = serializer.serialize("User{id=1,name='Alice'}");
        String result = serializer.deserialize(data, String.class);
        System.out.println("结果: " + result);
    }
}
```


### 场景 8：多级别日志记录

- 背景：日志框架支持输出到控制台、文件、数据库、Elasticsearch、Kafka。
- 策略角色：每种日志目标是一个策略，实现 log(level, message, context)。
- 工厂角色：根据配置（console、file、es）创建日志策略。工厂可组合多个策略（同时写文件和 ES）。
- 关键差异：控制台同步，文件异步批量写入，ES 需要构建索引，Kafka 生产消息。

```java
import java.time.LocalDateTime;
import java.util.*;

enum LogLevel { DEBUG, INFO, WARN, ERROR }

interface LogStrategy {
    void log(LogLevel level, String message, Map<String, Object> context);
}

class ConsoleLog implements LogStrategy {
    public void log(LogLevel level, String message, Map<String, Object> ctx) {
        System.out.println(LocalDateTime.now() + " [" + level + "] " + message + " " + ctx);
    }
}

class FileLog implements LogStrategy {
    public void log(LogLevel level, String message, Map<String, Object> ctx) {
        System.out.println("[FILE] " + LocalDateTime.now() + " [" + level + "] " + message);
    }
}

class EsLog implements LogStrategy {
    public void log(LogLevel level, String message, Map<String, Object> ctx) {
        System.out.println("[ELASTICSEARCH] index=logs, doc=" + message);
    }
}

class LogFactory {
    private static final Map<String, LogStrategy> map = new HashMap<>();
    static {
        map.put("console", new ConsoleLog());
        map.put("file", new FileLog());
        map.put("elasticsearch", new EsLog());
    }
    public static LogStrategy getLogger(String target) {
        return map.get(target.toLowerCase());
    }
}

public class LogDemo {
    public static void main(String[] args) {
        LogStrategy logger = LogFactory.getLogger("console");
        logger.log(LogLevel.INFO, "用户登录成功", Map.of("userId", 123));
    }
}
```

### 场景 9：多速率限制算法

- 背景：API 网关需要限流，支持令牌桶、漏桶、滑动窗口、计数器。
- 策略角色：每种限流算法是一个策略，实现 tryAcquire(key) -> boolean。
- 工厂角色：根据配置（token_bucket、leaky_bucket、sliding_window）创建限流策略。工厂可注入参数（容量、速率）。
- 关键差异：令牌桶允许突发，漏桶平滑，滑动窗口精确，计数器简单但有临界问题。

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

interface RateLimitStrategy {
    boolean tryAcquire(String key);
}

class TokenBucket implements RateLimitStrategy {
    private final Map<String, Long> tokens = new ConcurrentHashMap<>();
    private final long capacity = 10;
    private final long refillRate = 1; // per second

    public boolean tryAcquire(String key) {
        long now = System.currentTimeMillis() / 1000;
        long current = tokens.getOrDefault(key, capacity);
        // 简单模拟：每秒恢复1个令牌
        current = Math.min(capacity, current + (now - lastRefill(key)) * refillRate);
        tokens.put(key, current);
        if (current > 0) {
            tokens.put(key, current - 1);
            System.out.println("令牌桶: 通过, 剩余令牌=" + (current-1));
            return true;
        }
        System.out.println("令牌桶: 拒绝");
        return false;
    }
    private long lastRefill(String key) {
        return System.currentTimeMillis() / 1000; // 简化
    }
}

class SlidingWindow implements RateLimitStrategy {
    private final Map<String, Deque<Long>> windows = new ConcurrentHashMap<>();
    private final int maxRequests = 5;
    private final long windowMs = 1000;

    public boolean tryAcquire(String key) {
        long now = System.currentTimeMillis();
        Deque<Long> deque = windows.computeIfAbsent(key, k -> new LinkedList<>());
        synchronized (deque) {
            while (!deque.isEmpty() && now - deque.peekFirst() > windowMs) {
                deque.pollFirst();
            }
            if (deque.size() < maxRequests) {
                deque.addLast(now);
                System.out.println("滑动窗口: 通过, 当前请求数=" + deque.size());
                return true;
            }
            System.out.println("滑动窗口: 拒绝");
            return false;
        }
    }
}

class RateLimitFactory {
    private static final Map<String, RateLimitStrategy> map = new HashMap<>();
    static {
        map.put("token_bucket", new TokenBucket());
        map.put("sliding_window", new SlidingWindow());
    }
    public static RateLimitStrategy getLimiter(String algorithm) {
        return map.get(algorithm.toLowerCase());
    }
}

public class RateLimitDemo {
    public static void main(String[] args) throws Exception {
        RateLimitStrategy limiter = RateLimitFactory.getLimiter("token_bucket");
        for (int i = 0; i < 12; i++) {
            System.out.println("请求" + i + ": " + limiter.tryAcquire("api:/user"));
            Thread.sleep(100);
        }
    }
}
```

### 场景 10：个性化推荐算法

- 背景：内容推荐系统支持协同过滤、基于内容的推荐、热门排行、随机推荐。
- 策略角色：每种推荐算法是一个策略，实现 recommend(userId, n) -> List<Item>。
- 工厂角色：根据用户画像（新用户用热门，老用户用协同过滤）或 A/B 实验分组创建策略。工厂可维护模型版本。
- 关键差异：协同过滤需要用户行为矩阵，内容推荐依赖物品特征，热门排行实时计算，随机推荐用于冷启动。

```java
import java.util.*;

interface RecommendStrategy {
    List<String> recommend(String userId, int n);
}

class CollaborativeFiltering implements RecommendStrategy {
    public List<String> recommend(String userId, int n) {
        System.out.println("协同过滤推荐 for " + userId);
        return Arrays.asList("商品A", "商品B", "商品C");
    }
}

class ContentBased implements RecommendStrategy {
    public List<String> recommend(String userId, int n) {
        System.out.println("基于内容推荐 for " + userId);
        return Arrays.asList("商品D", "商品E");
    }
}

class HotRank implements RecommendStrategy {
    public List<String> recommend(String userId, int n) {
        System.out.println("热门排行推荐 for " + userId);
        return Arrays.asList("商品X", "商品Y", "商品Z");
    }
}

class RandomRecommend implements RecommendStrategy {
    public List<String> recommend(String userId, int n) {
        System.out.println("随机推荐 for " + userId);
        return Arrays.asList("商品M", "商品N");
    }
}

class RecommendFactory {
    private static final Map<String, RecommendStrategy> map = new HashMap<>();
    static {
        map.put("collaborative", new CollaborativeFiltering());
        map.put("content", new ContentBased());
        map.put("hot", new HotRank());
        map.put("random", new RandomRecommend());
    }
    public static RecommendStrategy getRecommender(String algo) {
        return map.get(algo.toLowerCase());
    }
}

public class RecommendDemo {
    public static void main(String[] args) {
        RecommendStrategy recommender = RecommendFactory.getRecommender("collaborative");
        List<String> items = recommender.recommend("user_001", 3);
        System.out.println("推荐结果: " + items);
    }
}
```
