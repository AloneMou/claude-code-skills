# PIG AI 编码助手指南

> 当 AI 辅助编码工具（Claude Code、Copilot 等）在 pig 项目中工作时，请遵循以下指南。

## 1. 项目认知

### 1.1 技术栈

在生成代码时，始终基于以下技术栈：

| 技术 | 版本 | 说明 |
|------|------|------|
| JDK | 17+ | Java 版本 |
| Spring Boot | 3.5 | 核心框架 |
| Spring Cloud | 2025 | 微服务框架 |
| Spring Cloud Alibaba | 2025 | Nacos, Sentinel 等 |
| Spring Authorization Server | 1.5 | OAuth2 鉴权 |
| MyBatis Plus | 3.5 | ORM 框架 |
| Spring Cloud Gateway | — | API 网关 |
| Seata | — | 分布式事务 |
| Vue | 3.5 | 前端框架 |
| Element Plus | 2.8 | UI 组件库 |
| Nacos | — | 注册中心/配置中心 |

### 1.2 架构模式

- **微服务模式**：各功能作为独立服务，通过 Nacos 注册发现
- **单体模式**：使用 `pig-boot` 启动器，适合小规模部署
- **前后端分离**：前端（pig-ui）与后端通过 API 交互
- **API 网关**：所有请求经过 pig-gateway 路由转发

## 2. 编码原则

### 2.1 必须遵循

1. **使用 spring-javaformat 格式化代码**
2. **使用 Lombok 简化代码**
3. **统一返回 `R<T>` 类型**
4. **使用 MyBatis Plus 的 `BaseMapper` 和 `IService`**
5. **Service 实现类使用 `@RequiredArgsConstructor` 构造注入**
6. **接口方法必须有 Swagger 注解**

### 2.2 禁止

1. ~~在 Controller 层编写业务逻辑~~
2. ~~使用字符串拼接 SQL~~
3. ~~在日志中打印密码、Token 等敏感信息~~
4. ~~捕获 `Exception` 或 `Throwable`~~
5. ~~在 biz 模块间直接依赖~~
6. ~~使用 `System.out.println` 代替日志~~

### 2.3 建议

1. 优先使用 `LambdaQueryWrapper` 而非 XML
2. 优先使用 `IService` 提供的内置方法
3. 业务逻辑集中在 Service 层
4. 使用枚举替代魔法数字
5. 异常信息使用中文描述

## 3. 代码生成模板

### 3.1 CRUD 标准代码

当用户要求"创建一个XX的CRUD"时，生成以下文件：

```
controller/   → XxxController.java    (REST API)
service/      → XxxService.java       (接口)
service/impl  → XxxServiceImpl.java   (实现)
mapper/       → XxxMapper.java        (数据访问)
entity/       → Xxx.java              (实体)
resources/mapper → XxxMapper.xml      (SQL 映射，可选)
```

### 3.2 文件模板

参考 [module-rules.md](./rules/module-rules.md) 中的 Controller、Service、Mapper 示例代码。

## 4. 任务类型响应

### 4.1 新增模块

1. 确认模块名称和位置
2. 参考 [module-dev skill](./skills/module-dev.md) 创建模块结构
3. 生成 pom.xml、启动类、配置文件

### 4.2 新增 CRUD

1. 确认表结构
2. 参考 [codegen skill](./skills/codegen.md) 生成代码
3. 执行代码格式化

### 4.3 跨服务调用

1. 在 api 模块定义 Feign 接口
2. 参考 [feign-api skill](./skills/feign-api.md)
3. 在目标服务的 Controller 实现对应接口

### 4.4 代码审查

1. 参考 [checklist.md](./code-review/checklist.md)
2. 参考 [security.md](./code-review/security.md)
3. 参考 [performance.md](./code-review/performance.md)

### 4.5 Bug 修复

1. 定位问题所在模块和类
2. 分析问题原因
3. 给出修复方案
4. 修复后确认不影响其他功能

## 5. 输出格式

### 5.1 代码输出

- 标注文件名和完整路径
- 标注新增、修改、删除的区域
- 包含必要的 import 语句

### 5.2 建议输出

- 先说明方案思路
- 列出涉及的改动文件
- 标注注意事项

## 6. 常用命令参考

```bash
# 编译项目
mvn clean install

# 格式化代码
mvn spring-javaformat:apply

# 检查代码格式
mvn spring-javaformat:validate

# 运行单体
mvn spring-boot:run -pl pig-boot

# 运行指定模块
mvn spring-boot:run -pl pig-upms/pig-upms-biz
```
