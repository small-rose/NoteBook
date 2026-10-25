---
layout: default
title: 分布式锁
parent: Cloud
nav_order: 1
---

# Distribute Global Lock
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## 分布式锁解决方案

### 1、数据库方案（兜底/低并发）

原理：

 - 利用数据库唯一索引的机制，锁表插记录，唯一键冲突即占锁。
 - 乐观锁机制（版本号）
 - 悲观锁（行锁）
 - 会话锁 PG pg_advisory_lock、MySQL GET_LOCK

优点：不引入新中间件；

缺点：性能差（QPS 通常 <1k）、有死锁风险

典型应用： RedLock组件的jdbc模式

```sql
CREATE TABLE distributed_lock (
lock_key   VARCHAR(128) PRIMARY KEY,
lock_value VARCHAR(64)  NOT NULL,
expire_at  DATETIME     NOT NULL
);
```

加锁案例
```
// 加锁
try {
    jdbc.update("INSERT INTO distributed_lock(lock_key,lock_value,expire_at) " +
                "VALUES (?,?,DATE_ADD(NOW(),INTERVAL 30 SECOND))", key, holder);
} catch (DuplicateKeyException e) {
    return false; // 已被占用
}

try {
    // 业务处理区
} finally {
    // 释放：必须校验持有者
    jdbc.update("DELETE FROM distributed_lock " +
                "WHERE lock_key=? AND lock_value=?", key, holder);
}
```
### 2、 Redis 方案

原理: 使用Nx命令 + PX自动续期 的原子命令。

```
SET lock:order:123 <uuid> NX PX 30000
```
注意点：

- 加锁用 NX（不存在才设）+ PX（自动过期）一条原子命令
- 释放必须用 Lua 脚本校验 uuid 再删除，避免误删别人的锁
- 可重入、看门狗续期由 Redisso 封装好了：默认 30s 租期，每 10s 续一次

生产首选 Redisson 

所需依赖

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson-spring-boot-starter</artifactId>
  <version>3.27.2</version> <!-- 选择与项目搭配的版本 -->
</dependency>
```

配置

```java
@Configuration
public class RedissonConfig {
    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer()
              .setAddress("redis://127.0.0.1:6379");
        // 看门狗默认 30s，可改：config.setLockWatchdogTimeout(30000L);
        return Redisson.create(config);
    }
}
```

使用案例：

```java
@Service
public class OrderService {
    @Autowired
    private RedissonClient redisson;

    public void processOrder(String orderId) {
        RLock lock = redisson.getLock("lock:order:" + orderId);
        try {
            // waitTime=10s，leaseTime 不传 → 看门狗自动续期
            boolean ok = lock.tryLock(10, TimeUnit.SECONDS);
            if (!ok) {
                throw new RuntimeException("获取锁失败，请重试");
            }
            // ===== 临界区业务 =====
            deductStock(orderId);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            // 必须判断持有者，避免误删
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```
关键机制

 - lock() 或不指定 leaseTime 的 tryLock → 看门狗启动，每 10s 自动续期到 30s
 - 显式传 leaseTime>0 则会让看门狗不生效，业务必须在租期内跑完
 - 可重入：内部用 Hash 记录 uuid:threadId + 重入计数


### 3、 ZooKeeper：Curator 方案

原理: **基于临时顺序节点 + Watch 监听机制**

（1）在 /lock/resource 下创建 EPHEMERAL_SEQUENTIAL 节点

（2）序号最小者持有锁

（3）其他客户端只监听前一个节点，避免惊群

（4）持锁者崩溃 → 会话断开 → 临时节点自动删除 → 下一节点被唤醒

优点：
- 会话级自动释放，不用猜 TTL
- ZAB 共识，强一致，自带 zxid 作为 fencing token
- 天然公平锁

缺点：
- 吞吐量中等（~5-20k QPS）
- 要维护 ZK 集群，运维重

依赖：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.1.0</version>
</dependency>
```
代码案例：

```java
public class ZkDistributedLock {
    private static final String ZK_ADDR = "127.0.0.1:2181";
    private static final String LOCK_PATH = "/locks/order";

    public void doWork() throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.newClient(
                ZK_ADDR, new ExponentialBackoffRetry(1000, 3));
        client.start();

        InterProcessMutex lock = new InterProcessMutex(client, LOCK_PATH);
        try {
            // 5s 内抢不到就放弃
            if (lock.acquire(5, TimeUnit.SECONDS)) {
                try {
                    // ===== 临界区 =====
                    System.out.println("持锁执行中...");
                } finally {
                    lock.release();
                }
            } else {
                System.out.println("未获取到锁");
            }
        } finally {
            client.close();
        }
    }
}
```
原理要点

- 底层是 /locks/order/00000001 这种临时顺序节点
- 序号最小者持锁，其余监听前一个节点（无惊群）
- 会话断开 → 临时节点自动删除 → 下一节点被唤醒
- acquire/release 必须成对，release 放 finally



### 4. etcd（云原生首选）
   
基于 Lease（租约）+ Raft CAS：

(1)申请带 TTL 的 Lease
(2)用事务 PUT IF_NOT_EXISTS 把 key 绑到 Lease 上
(3)定期 KeepAlive 续租；客户端死了租约过期，key 自动删除
(4)每次写产生全局递增 Revision，天然就是 fencing token

优点：

- Raft 强一致 + 内置 fencing
- K8s 生态自带，云原生项目几乎零额外运维
- gRPC 接口简单

缺点：
- 吞吐比 Redis 低，共识写有延迟


如果在Java 侧应用

```xml
<dependency>
    <groupId>io.etcd</groupId>
    <artifactId>jetcd-core</artifactId>
    <version>0.7.7</version><!-- 以 Maven 中央仓库最新版为准 -->
</dependency>
```

客户端配置（Spring Boot 场景）

```java
@Configuration
public class EtcdConfig {
    @Bean
    public Client etcdClient() {
        return Client.builder()
                .endpoints("http://etcd1:2379", "http://etcd2:2379", "http://etcd3:2379")
                .build();
    }
}
```

基础 KV 读写示例

```java
import io.etcd.jetcd.*;
import io.etcd.jetcd.kv.GetResponse;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.ExecutionException;

public class EtcdKVExample {

    private static final Client client =
            Client.builder().endpoints("http://localhost:2379").build();

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        KV kv = client.getKVClient();

        ByteSequence key   = ByteSequence.from("user:1", StandardCharsets.UTF_8);
        ByteSequence value = ByteSequence.from("{\"name\":\"tom\"}", StandardCharsets.UTF_8);

        // 写入
        kv.put(key, value).get();

        // 读取
        GetResponse resp = kv.get(key).get();
        String v = resp.getKvs().get(0).getValue().toString(StandardCharsets.UTF_8);
        System.out.println("读到: " + v);

        // 删除
        kv.delete(key).get();

        client.close();
    }
}
```

> 关键点：ByteSequence.from() 是 jetcd 的标配，字符串必须显式指定字符集。
 
使用 ETCD 原生自带 Lock API（推荐）

etcd 自带 Lock 服务，不用自己手写 revision 比较和 watch，客户端只要管好租约续期。

```java
import io.etcd.jetcd.*;
import io.etcd.jetcd.lease.LeaseGrantResponse;
import io.etcd.jetcd.lock.LockResponse;
import io.etcd.jetcd.maintenance.LeaseKeepAliveResponse;
import io.grpc.stub.StreamObserver;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class EtcdDistributedLock {

    private final Client client;
    private final Lock lockClient;
    private final Lease leaseClient;

    public EtcdDistributedLock(Client client) {
        this.client = client;
        this.lockClient = client.getLockClient();
        this.leaseClient = client.getLeaseClient();
    }

    public void runWithLock(String lockName, Runnable business) throws Exception {
        String name = lockName;
        ByteSequence lockKey = ByteSequence.from(name, StandardCharsets.UTF_8);

        // 1. 申请租约（30s TTL）
        long ttl = 30;
        LeaseGrantResponse grant = leaseClient.grant(ttl).get();
        long leaseId = grant.getID();

        // 2. 自动续租（看门狗）：服务端按 TTL/3 节奏发心跳
        ScheduledExecutorService watchdog = Executors.newSingleThreadScheduledExecutor();
        leaseClient.keepAlive(leaseId, new StreamObserver<LeaseKeepAliveResponse>() {
            @Override public void onNext(LeaseKeepAliveResponse r) { /* 续租成功 */ }
            @Override public void onError(Throwable t) { /* 续租异常，业务应主动退出 */ }
            @Override public void onCompleted() { }
        });

        // 3. 抢锁（阻塞，直到拿到或线程中断）
        LockResponse lockResp = lockClient.lock(lockKey, leaseId).get();
        ByteSequence lockPath = lockResp.getKey(); // etcd 实际写入的 key

        try {
            // 4. 临界区业务
            business.run();
        } finally {
            // 5. 释放锁
            lockClient.unlock(lockPath).get();
            // 6. 撤销租约 + 关线程
            leaseClient.revoke(leaseId).get();
            watchdog.shutdownNow();
        }
    }
    // tryLock 带超时时间版本
    public boolean tryLock(String lockName, long timeoutMillis) {
        try {
            ByteSequence key = ByteSequence.from(lockName, StandardCharsets.UTF_8);
            long leaseId = leaseClient.grant(30).get().getID();
            // 带超时抢锁
            LockResponse resp = lockClient.lock(key, leaseId)
                    .get(timeoutMillis, TimeUnit.MILLISECONDS);
            // 记录 leaseId / lockPath 到 ThreadLocal，unlock 时用
            return resp != null;
        } catch (Exception e) {
            Thread.currentThread().interrupt();
            return false;
        }
    }
    
    public static void main(String[] args) throws Exception {
        Client client = Client.builder().endpoints("http://localhost:2379").build();
        EtcdDistributedLock lock = new EtcdDistributedLock(client);

        lock.runWithLock("/locks/order:1001", () -> {
            System.out.println("拿到锁，执行扣库存...");
            try { Thread.sleep(2000); } catch (InterruptedException ignored) {}
        });

        client.close();
    }
}
```

Watch 监听示例（配置热更新场景）

```java
import io.etcd.jetcd.*;
import io.etcd.jetcd.watch.WatchEvent;
import io.etcd.jetcd.watch.WatchResponse;
import io.grpc.stub.StreamObserver;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class EtcdWatchExample {

    private final Client client;
    private Watch watchClient;

    public EtcdWatchExample(String endpoints) {
        this.client = Client.builder().endpoints(endpoints).build();
    }

    /**
     * 监听单个 key 的变化（持续阻塞）
     */
    public void watchKey(String keyStr) throws InterruptedException {
        watchClient = client.getWatchClient();
        ByteSequence key = ByteSequence.from(keyStr, StandardCharsets.UTF_8);

        CountDownLatch latch = new CountDownLatch(1); // 用于保持主线程不退出

        // 创建 Watch（返回 Watcher 对象，可用于取消监听）
        Watch.Watcher watcher = watchClient.watch(key, new StreamObserver<>() {
            @Override
            public void onNext(WatchResponse response) {
                // 每次有变更事件时触发
                for (WatchEvent event : response.getEvents()) {
                    String type = event.getEventType().name(); // PUT / DELETE
                    String value = event.getKeyValue().getValue().toStringUtf8();
                    System.out.printf("[%s] key=%s, value=%s%n",
                            type, keyStr, value);
                }
            }

            @Override
            public void onError(Throwable t) {
                System.err.println("Watch 出错: " + t.getMessage());
                latch.countDown(); // 异常时退出
            }

            @Override
            public void onCompleted() {
                System.out.println("Watch 结束");
                latch.countDown();
            }
        });

        System.out.println("开始监听 " + keyStr + "，按 Ctrl+C 停止...");

        // 阻塞等待（实际生产环境会放在后台线程运行）
        latch.await();

        // 关闭 Watcher（释放 gRPC 流）
        watcher.close();
    }

    /**
     * 监听前缀（如 /config/* 下的所有 key）
     */
    public void watchPrefix(String prefixStr) throws InterruptedException {
        watchClient = client.getWatchClient();
        ByteSequence prefix = ByteSequence.from(prefixStr, StandardCharsets.UTF_8);

        CountDownLatch latch = new CountDownLatch(1);

        Watch.Watcher watcher = watchClient.watch(
                prefix,
                WatchOption.withPrefix(prefix), // 关键：加上前缀选项
                new StreamObserver<>() {
                    @Override
                    public void onNext(WatchResponse response) {
                        for (WatchEvent event : response.getEvents()) {
                            String key   = event.getKeyValue().getKey().toStringUtf8();
                            String value = event.getKeyValue().getValue().toStringUtf8();
                            System.out.printf("[%s] %s = %s%n",
                                    event.getEventType(), key, value);
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                        System.err.println("前缀 Watch 出错: " + t.getMessage());
                        latch.countDown();
                    }

                    @Override
                    public void onCompleted() {
                        System.out.println("前缀 Watch 结束");
                        latch.countDown();
                    }
                }
        );

        latch.await();
        watcher.close();
    }

    public void close() {
        if (watchClient != null) watchClient.close();
        if (client != null) client.close();
    }

    // ============ 测试入口 ============
    public static void main(String[] args) throws Exception {
        EtcdWatchExample demo = new EtcdWatchExample("http://localhost:2379");

        // 在另一个线程里模拟修改 etcd 的值
        new Thread(() -> {
            try {
                Thread.sleep(2000);
                KV kv = demo.client.getKVClient();
                kv.put(
                        ByteSequence.from("/config/timeout", StandardCharsets.UTF_8),
                        ByteSequence.from("5000", StandardCharsets.UTF_8)
                ).get();
                System.out.println("已写入 /config/timeout = 5000");

                Thread.sleep(2000);
                kv.put(
                        ByteSequence.from("/config/timeout", StandardCharsets.UTF_8),
                        ByteSequence.from("8000", StandardCharsets.UTF_8)
                ).get();
                System.out.println("已更新 /config/timeout = 8000");

                Thread.sleep(2000);
                kv.delete(
                        ByteSequence.from("/config/timeout", StandardCharsets.UTF_8)
                ).get();
                System.out.println("已删除 /config/timeout");

            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();

        // 监听单个 key
        demo.watchKey("/config/timeout");

        // 如果要监听前缀，注释上面一行，取消下面这行的注释：
        // demo.watchPrefix("/config/");

        demo.close();
    }
}
```

关键点说明
1. 为什么需要 CountDownLatch ? 因为Watch 是异步回调模式，onNext() 不会阻塞主线程。如果不加 latch.await()，main 方法会立刻执行完退出，看不到事件输出。

2. 两种 Watch 方式
 - 精确 key  watch(key, observer), 只监听 /config/timeout 这一个 key
 - 前缀匹配  watch(prefix, WatchOption.withPrefix(prefix), observer) 可以监听 /config/ 下的所有 key

3. 资源释放顺序
```
watcher.close() → watchClient.close() → client.close()
```
watcher.close()：关闭当前 Watch 的 gRPC 流
watchClient.close()：关闭 Watch 客户端工厂（如果有多个 Watcher，会在最后一个 Watcher 关闭后才真正关闭）
client.close()：关闭整个 etcd 客户端

4. 事件类型 WatchEvent.EventType 有三种：
- PUT：key 被创建或更新
- DELETE：key 被删除
- （jetcd 0.7.x 版本还有 UNRECOGNIZED，一般忽略）

5. 生产注意事项

- Watch 是长连接，不要在主线程里 latch.await() 死等，应该放到后台线程池
- onError 会断开 Watch，需要重连策略（如指数退避重建 Watcher）
- 不要在 onNext 里做耗时操作，否则会阻塞 gRPC 的响应处理线程，建议扔到业务线程池
- Watch 的 key 不存在也能监听，后续创建/更新/删除都会收到事件


### 5.应用场景和选型对比

**不同应用场景选择**

|场景  | 直接用|
|------|-----|
|高并发、缓存防击穿、幂等 | Redisson |
|强一致、订单/库存/资金  | etcd 或 ZK + fencing |
|K8s 内部服务  | etcd（concurrency 包）|
|低频后台任务、不想引中间件 | 数据库唯一索引|
|读多写少更新 | 乐观锁版本号|


**不同方案对比**

| 方案         |	一致性 |	吞吐 | 	自动防死锁             | 	fencing token栅栏令牌 | 	运维成本 |
|------------|------|-----|--------------------|--------------------|-------|
| Redis 单节点  | 弱（AP）	|极高	| TTL                | 需自建 | 	低    |
| Redlock	   | 弱-中	| 高	| TTL                | 	需自建	| 高（5节点）     
| ZooKeeper	 | 强（CP）	| 中	| 会话机制	              | 有（zxid）| 	高 
| etcd	      | 强（CP）	| 中	| Lease	| 有（revision）	 | 中 
| 数据库	       | 依赖DB	| 低	| 需超时清理	             | 版本号	| 低       |     



### 5.Fencing token 栅栏令牌

什么是 fencing token ？

无论用哪种锁，都可能遇到这种情况：客户端 A 拿到锁 → GC 暂停 10s → 锁过期 → B 拿到锁 → A 恢复继续写

> 客户端 A 拿到分布式锁（比如 Redis 锁），开始处理订单扣库存。
> 客户端 A 发生 Full GC（停顿 10 秒），锁因为 TTL 到期自动释放了。
> 客户端 B 拿到锁，开始处理同一个订单，扣了一次库存。
> 客户端 A 从 GC 中恢复，它仍然认为自己持有锁，于是也去扣库存。

结果：库存被扣了两次，数据不一致。

这时候两个客户端都以为自己持锁。解决办法不是换锁实现，而是加 Fencing Token（栅栏令牌）：

**问题的根源：**

锁服务（Redis、ZK、etcd）只能保证同一时刻只有一个客户端能拿到锁，但它无法阻止客户端在失去锁后依然继续执行。锁超时、网络分区、GC 停顿都会导致这种“双持锁”现象。

Fencing Token 就是来解决这个问题的。

- 每次加锁返回一个单调递增令牌（etcd revision / ZK zxid / 自定义版本号）
- 写资源时带上令牌
- 存储端只接受“令牌 ≥ 已见过的最大值”的写，旧令牌直接拒绝

```sql
UPDATE resource SET val=? , token=35 
WHERE id=123 AND token < 35
```
锁服务负责调度，资源端负责裁决。

Fencing Token（栅栏令牌） 是分布式系统中用来防止“幽灵”客户端破坏数据一致性的一种机制。简单说，它是一个单调递增的编号，每次获取分布式锁时都会得到一个比之前更大的令牌，然后带着这个令牌去操作共享资源，资源端只接受令牌最新的请求。

**Fencing Token 的工作原理**

1. 获取锁时附带令牌
>  每次成功获取分布式锁时，锁服务返回一个全局唯一且单调递增的整数（例如 etcd 的 CreateRevision、ZK 的 zxid、或者自己用 Redis INCR 生成的序列号）。
2. 操作资源时带上令牌
>  客户端在执行业务写操作（例如更新数据库、调用 API）时，必须把当前持有的 fencing token 一起发送给资源端。
3. 资源端校验令牌
>  资源端（数据库、文件系统等）维护一个“已见过最大令牌”的记录（比如在表中存一个字段 max_fence）。每次写请求到达时，资源端比较：

如果请求携带的 token 大于 max_fence → 允许写入，并更新 max_fence 为新值。

如果请求携带的 token 小于等于 max_fence → 拒绝写入（说明这是个过期的、来自旧锁持有者的请求）。

```
时间线 →
A: 获取锁 (token=1) ──→ GC 停顿 ──→ 锁过期 ──→ 醒来，继续写 (token=1)
B:                      ──→ 获取锁 (token=2) ──→ 写资源 (token=2) 成功，max_fence=2
A:                                                             ──→ 写资源 (token=1) → 资源端发现 token(1) ≤ max_fence(2) → 拒绝！
```
最终只有 B 的写入生效，A 的陈旧请求被安全拦截。

**如何生成 Fencing Token？**

不同锁服务提供不同的单调递增标识:


|锁服务	|可作为 fencing token 的值	|说明  |
|-------|---------------------------|------|
|etcd	|CreateRevision（创建 key 时的全局版本号）	|每个 key 第一次创建时分配，全局递增 |
|ZooKeeper	|zxid（ZooKeeper 事务 ID）	|每次写操作都会产生一个全局递增的 zxid |
|Redis	|自己用 INCR 生成序列号	|需要额外的 Redis key 来维护计数器 |
|数据库	|自增主键或序列	|简单可靠 |

注意：不要用时间戳、UUID 或随机数作为 fencing token，因为它们无法保证严格的单调递增和全局有序性。


特别说明：
{: .important }
> (1)Fencing Token 不是锁的替代品，而是锁的补充——锁负责调度谁可以干活，fencing token 负责验证活是不是“合法”的。
> 
> (2)只有配合资源端的校验才有意义。如果资源端不检查 token，光在客户端生成 token 是没用的。
> 
> (3)不是所有场景都需要 fencing token。如果你的业务允许偶尔的重复执行（比如幂等操作、缓存刷新），或者你对锁的超时设置非常保守且网络/GC 很稳定，可以不引入。但对于资金、库存、订单状态等强一致场景，强烈建议加上。

Fencing Token 本职是一张“通行证编号”，每次拿锁时发一张新的，资源端只认最新编号的请求，旧编号的请求一律作废——从而彻底杜绝“客户端以为锁还在，实际上锁已经丢了”导致的数据错乱。

### 6.应用注意事项

- 释放锁必须校验 owner（Redis 用 isHeldByCurrentThread，DB 用 lock_value），防止误删其他人的锁
- unlock / release 必须放 finally, 确保加锁必解锁。
- 业务耗时可能超过 TTL, 使用用看门狗或 KeepAlive，避免强行设置短过期时间
- 加 fencing token机制预防 GC 停顿/网络分区后的双写问题
- 别用阻塞式死等 → tryLock 带 waitTime，拿不到就快速失败或排队
- ZK Session 超时设置过短有可能导致长任务被提前踢出
- 数据库悲观锁不控制事务粒度，预防死锁或连接打满
