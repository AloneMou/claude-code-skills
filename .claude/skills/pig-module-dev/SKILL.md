---
name: pig-module-dev
description: "在 pig 微服务项目中新增或修改业务模块。当用户要求创建新服务、扩展现有模块、配置 Gateway 路由或编写 Spring Cloud 微服务代码时使用。"
---

# PIG 模块开发

在 pig 项目中开发微服务模块。

## TRIGGER

用户需要在 pig 项目中：
- 新增业务微服务模块
- 扩展现有模块功能
- 配置 Gateway 路由
- 编写 Controller / Service / Mapper

## 技术栈

| 技术 | 版本 |
|------|------|
| JDK | 17+ |
| Spring Boot | 3.5 |
| Spring Cloud | 2025 |
| Spring Cloud Alibaba | 2025 |
| Spring Authorization Server | 1.5 |
| MyBatis Plus | 3.5 |
| Nacos | 注册中心/配置中心 |
| Seata | 分布式事务 |
| Vue | 3.5 + Element Plus 2.8 |

## 模块结构

```
pig
├── pig-boot              -- 单体启动器 [9999]
├── pig-auth              -- 授权服务 [3000]
├── pig-common            -- 公共模块
│   ├── pig-common-bom       -- 依赖管理
│   ├── pig-common-core      -- 工具类
│   ├── pig-common-mybatis   -- MyBatis 扩展
│   ├── pig-common-security  -- 安全鉴权
│   ├── pig-common-feign     -- Feign 扩展
│   ├── pig-common-swagger   -- 接口文档
│   ├── pig-common-log       -- 日志
│   ├── pig-common-datasource-- 动态数据源
│   ├── pig-common-seata     -- 分布式事务
│   ├── pig-common-websocket -- WebSocket
│   ├── pig-common-oss       -- 文件上传
│   └── pig-common-xss       -- XSS 防护
├── pig-register          -- Nacos [8848]
├── pig-gateway           -- 网关 [9999]
├── pig-upms              -- 用户权限
│   ├── pig-upms-api       -- API 模块
│   └── pig-upms-biz       -- 业务 [4000]
└── pig-visual
    ├── pig-monitor       -- 监控 [5001]
    ├── pig-codegen       -- 代码生成 [5002]
    └── pig-quartz        -- 定时任务 [5007]
```

## 开发规则

### 1. 分层架构

- **Controller**：参数接收、校验、调用 Service、返回 `R<T>`，不含业务逻辑
- **Service**：接口 + 实现，继承 `IService<Entity>`，事务标注在实现类
- **Mapper**：继承 `BaseMapper<Entity>`，复杂查询用 XML 或 `LambdaQueryWrapper`

### 2. 新增模块步骤

1. 在根 `pom.xml` 的 `<modules>` 中添加
2. 创建 `pig-{module}-api` 和 `pig-{module}-biz` 子模块
3. API 模块依赖：`pig-common-core` + `pig-common-feign`
4. BIZ 模块依赖：`pig-{module}-api` + `pig-common-mybatis` + `pig-common-security` + `pig-common-swagger`
5. 配置 `bootstrap.yml`（Nacos 注册）和 `application.yml`
6. 在 Gateway 添加路由，在 `ServiceNameConstants` 注册服务名

### 3. 禁止

- 在 Controller 层编写业务逻辑
- 使用字符串拼接 SQL
- 在日志中打印密码、Token
- 捕获 `Exception` 或 `Throwable`
- biz 模块间直接依赖（应通过 Feign）
- 使用 `System.out.println`

### 4. 必须

- 提交前执行 `mvn spring-javaformat:apply`
- 统一返回 `R<T>` 类型
- Service 使用 `@RequiredArgsConstructor` 构造注入
- 接口方法必须有 Swagger 注解
- 使用 Lombok 简化代码

## Entity 模板

```java
@Data
@TableName("table_name")
@Schema(description = "描述")
public class Xxx {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    @TableLogic
    private String delFlag;
}
```

## Controller 模板

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/xxx")
@Tag(description = "xxx", name = "模块名")
@SecurityRequirement(name = HttpHeaders.AUTHORIZATION)
public class XxxController {
    private final XxxService xxxService;

    @GetMapping("/{id}")
    @Operation(summary = "获取详情")
    public R<Xxx> getById(@PathVariable Long id) {
        return R.ok(xxxService.getById(id));
    }

    @PostMapping
    @Operation(summary = "新增")
    public R<Void> save(@Valid @RequestBody XxxDTO dto) {
        xxxService.save(dto);
        return R.ok();
    }

    @PutMapping
    @Operation(summary = "修改")
    public R<Void> update(@Valid @RequestBody XxxDTO dto) {
        xxxService.update(dto);
        return R.ok();
    }

    @DeleteMapping("/{id}")
    @Operation(summary = "删除")
    public R<Void> removeById(@PathVariable Long id) {
        xxxService.removeById(id);
        return R.ok();
    }
}
```

## 常用命令

```bash
mvn clean install                    # 编译
mvn spring-javaformat:apply          # 格式化代码
mvn spring-javaformat:validate       # 检查格式
mvn spring-boot:run -pl pig-boot     # 单体运行
```

## 参考

- [编码规范](../../CLAUDE.md) — 命名、注释、分层规范
- [包结构规则](../../CLAUDE.md) — 依赖引入、模块间依赖原则
- [安全审查](../../CLAUDE.md) — 鉴权、输入验证、数据安全
- [代码审查清单](../../CLAUDE.md) — 提交前自查