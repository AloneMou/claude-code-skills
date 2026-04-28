# PIG 项目规则

> 基于 [pig-mesh/pig](https://github.com/pig-mesh/pig) — 企业级快速开发平台

## 技术栈

| 技术 | 版本 |
|------|------|
| JDK | 17+ |
| Spring Boot | 3.5 |
| Spring Cloud | 2025 |
| Spring Cloud Alibaba | 2025 |
| Spring Authorization Server | 1.5 |
| MyBatis Plus | 3.5 |
| Vue | 3.5 + Element Plus 2.8 |
| Nacos | 注册中心/配置中心 |
| Seata | 分布式事务 |
| 代码格式 | spring-javaformat（强制） |

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

## 编码规则

### 命名

| 类型 | 规则 | 示例 |
|------|------|------|
| Controller | `XxxController` | `SysUserController` |
| Service | `XxxService` / `XxxServiceImpl` | `SysUserService` |
| Mapper | `XxxMapper` | `SysUserMapper` |
| Entity | `Xxx` / `SysXxx` | `SysUser` |
| DTO | `XxxDTO` / `XxxReq` | `UserDTO` |
| VO | `XxxVO` | `UserVO` |
| 常量 | `XxxConstants` | `SecurityConstants` |
| 工具类 | `XxxUtil` | `SecurityUtil` |
| 配置类 | `XxxConfiguration` | `MybatisPlusConfig` |

### 分层

- **Controller**：参数接收、校验、调用 Service、返回 `R<T>`，不含业务逻辑
- **Service**：接口 + 实现，继承 `IService<Entity>`，事务标注在实现类
- **Mapper**：继承 `BaseMapper<Entity>`，复杂查询用 XML 或 `LambdaQueryWrapper`

### Lombok

```java
@Data              // getter/setter/toString/equals/hashCode
@Builder           // 建造者
@NoArgsConstructor // 无参构造
@AllArgsConstructor// 全参构造
@Slf4j             // 日志
@RequiredArgsConstructor // 构造注入（Service 层首选）
@Accessors(chain = true) // 链式调用
```

### Entity 模板

```java
@Data
@TableName("sys_user")
@Schema(description = "用户信息")
public class SysUser {
    @TableId(type = IdType.ASSIGN_ID)
    private Long userId;
    private String username;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    @TableLogic
    private String delFlag;
}
```

### Controller 模板

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
    public R<Xxx> getById(@PathVariable Long id) { return R.ok(xxxService.getById(id)); }

    @PostMapping
    @Operation(summary = "新增")
    public R<Void> save(@Valid @RequestBody XxxDTO dto) { xxxService.save(dto); return R.ok(); }

    @PutMapping
    @Operation(summary = "修改")
    public R<Void> update(@Valid @RequestBody XxxDTO dto) { xxxService.update(dto); return R.ok(); }

    @DeleteMapping("/{id}")
    @Operation(summary = "删除")
    public R<Void> removeById(@PathVariable Long id) { xxxService.removeById(id); return R.ok(); }
}
```

### Feign 接口（api 模块）

```java
@FeignClient(contextId = "remoteUserService", value = ServiceNameConstants.UPMS_SERVICE)
public interface RemoteUserService {
    @GetMapping("/sys-user/info/{username}")
    R<UserInfo> info(@PathVariable("username") String username,
                     @RequestHeader(SecurityConstants.FROM) String from);
}
```

### 全局返回

统一使用 `R<T>` 类型：`R.ok(data)` 成功，`R.failed("错误信息")` 失败

### 注释

- 类注释包含 `@author pigx` 和 `@date`
- 公共 API 方法必须有 Javadoc
- 所有接口使用 `@Operation` Swagger 注解
- Entity/DTO 使用 `@Schema` 说明

## 包结构规则

### 标准模块结构

```
pig-{module}
├── pig-{module}-api/          -- API 模块（DTO、Entity、Feign 接口）
└── pig-{module}-biz/          -- 业务模块（Controller、Service、Mapper）
    └── src/main/resources/mapper/  -- MyBatis XML
```

### 依赖规则

- `pig-common-*` 版本由 `pig-common-bom` 统一管理，子模块只需声明 artifactId
- **biz 依赖 api**：`pig-upms-biz` 依赖 `pig-upms-api`
- **禁止 biz 间直接依赖**，跨服务通过 Feign 调用
- **api 不依赖 biz**

### 配置文件

- `bootstrap.yml`：Nacos 注册配置
- `application.yml`：服务自身配置
- 敏感信息使用 Jasypt 加密：`ENC(密文)`

## 代码格式化

**强制使用 spring-javaformat**，提交前必须执行：

```bash
mvn spring-javaformat:apply
```

- 缩进使用 **Tab**
- 每行最大 **90** 字符
- 导入语句按 spring-javaformat 规则排序

## 安全规则

- 管理接口必须有 `@HasPermission("module:action")` 权限校验
- 参数使用 `@Valid` / `@Validated` 校验
- SQL 使用 `#{}` 参数化，禁止 `${}` 拼接（排序字段除外，需白名单）
- 密码使用 BCrypt 加密，日志中禁止打印密码和 Token
- Feign 调用传递 `SecurityConstants.FROM_IN`
- 敏感操作使用 `@SysLog` 记录审计日志

## 禁止

- 在 Controller 层编写业务逻辑
- 使用字符串拼接 SQL
- 在日志中打印密码、Token
- 捕获 `Exception` 或 `Throwable`
- biz 模块间直接依赖
- 使用 `System.out.println`

## 必须

- 提交前执行 `mvn spring-javaformat:apply`
- 统一返回 `R<T>` 类型
- Service 使用 `@RequiredArgsConstructor`
- 接口方法有 Swagger 注解
- 事务方法标注 `@Transactional(rollbackFor = Exception.class)`

## 常用命令

```bash
mvn clean install                    # 编译
mvn spring-javaformat:apply          # 格式化
mvn spring-javaformat:validate       # 检查格式
mvn spring-boot:run -pl pig-boot     # 单体运行
```

## 代码审查

参考 `code-review/` 目录：

- [checklist.md](code-review/checklist.md) — 通用审查清单
- [security.md](code-review/security.md) — 安全审查
- [performance.md](code-review/performance.md) — 性能审查
