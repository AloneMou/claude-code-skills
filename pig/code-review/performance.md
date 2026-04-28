# PIG 性能审查规则

## 1. 数据库性能

### 1.1 查询优化

- 禁止在循环中执行数据库查询（N+1 问题）
- 批量操作使用 `saveBatch` / `updateBatchById` / `removeBatchByIds`
- 只查询需要的字段，避免 `SELECT *`
- 分页查询必须使用 `Page<T>` 对象，避免全表扫描

### 1.2 索引

- 频繁查询字段必须建立索引
- 外键字段必须建立索引
- 排序字段建立索引避免 filesort
- 使用 `EXPLAIN` 分析慢查询

### 1.3 连接池

- 使用 HikariCP 作为连接池
- 合理配置连接池参数：
  - `maximum-pool-size`：根据并发量调整（默认 10）
  - `minimum-idle`：建议与 maximum-pool-size 相同
  - `connection-timeout`：30000ms
  - `idle-timeout`：600000ms

## 2. 缓存策略

### 2.1 Redis 使用

- 热点数据使用 Redis 缓存
- 设置合理的过期时间（TTL）
- 避免缓存穿透：使用布隆过滤器或空值缓存
- 避免缓存雪崩：过期时间增加随机值
- 避免缓存击穿：使用分布式锁

### 2.2 缓存一致性

- 数据更新时及时清除/更新缓存
- 先更新数据库，再删除缓存（Cache Aside Pattern）
- 分布式事务场景考虑缓存的最终一致性

## 3. 代码性能

### 3.1 避免常见问题

- 循环中避免对象创建（尤其大对象）
- 字符串拼接使用 `StringBuilder` 而非 `+`
- 避免不必要的同步（`synchronized`）
- 大集合初始化指定容量

### 3.2 Stream API

- 合理使用 Stream API，避免过度使用
- 大数据量场景考虑并行流（`parallelStream`），但注意线程安全
- 避免在 Stream 中执行数据库操作

### 3.3 日志性能

```java
// 避免：字符串拼接在 debug 级别也会执行
log.debug("user info: " + user);

// 推荐：使用占位符
log.debug("user info: {}", user);
```

## 4. 网关性能

### 4.1 Gateway 配置

- 启用连接复用
- 配置合理的超时时间
- 路由规则尽量精简
- 避免在网关中执行复杂逻辑

### 4.2 限流降级

- Gateway 层配置限流过滤器
- 使用 Sentinel 实现服务降级
- 合理配置线程池大小

## 5. Feign 调用性能

- 配置合理的超时时间
- 启用请求压缩（`feign.compression.*`）
- 使用连接池（Apache HttpClient 或 OkHttp）
- 避免跨服务调用链路过长

## 6. 异步处理

### 6.1 异步任务

- 非关键路径操作使用 `@Async` 异步执行
- 配置独立的线程池（不使用默认的 `SimpleAsyncTaskExecutor`）
- 异步方法返回值使用 `CompletableFuture<T>`

### 6.2 消息队列

- 耗时操作（如发送邮件、导出报表）通过消息队列解耦
- 消息消费需要幂等性保证
- 消息失败需要重试机制

## 7. 线程安全

- 共享可变状态需要同步控制
- 优先使用不可变对象
- `ConcurrentHashMap` 替代 `HashMap` 在并发场景
- 注意 `@Scheduled` 任务的线程池配置

## 8. 内存管理

- 避免大对象常驻内存
- 及时关闭资源（使用 try-with-resources）
- 避免内存泄漏：`ThreadLocal` 使用后必须 `remove()`
- 大文件处理使用流式读取，避免一次性加载到内存

## 9. 监控与排查

- 启用 Spring Boot Actuator 健康检查
- 使用 Spring Boot Admin 监控服务状态
- 配置 JVM 参数：`-Xms`, `-Xmx`, `-XX:+HeapDumpOnOutOfMemoryError`
- 使用 Arthas 排查线上性能问题
