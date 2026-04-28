# PIG 安全审查规则

## 1. 认证与鉴权

### 1.1 OAuth2 安全

- 所有需要认证的接口必须通过 Spring Authorization Server 验证
- 不允许绕过 token 验证直接访问受保护接口
- Token 有效期配置合理（Access Token 不宜过长）
- 刷新 Token 的接口需要安全控制

### 1.2 权限校验

- 每个管理接口必须使用 `@HasPermission("xxx:xxx")` 注解
- 权限标识遵循 `模块:操作` 格式，如 `sys_user:add`
- 数据权限通过 `@DataScope` 实现部门数据隔离
- 不允许在 Controller 层硬编码权限判断

### 1.3 Feign 调用安全

- 内部服务间调用必须传递 `SecurityConstants.FROM_IN` 标识
- Gateway 层需校验请求来源，防止外部直接访问内部服务
- Feign 接口不应暴露敏感操作（如删除、修改密码）

## 2. 输入验证

### 2.1 参数校验

```java
// Controller 方法必须使用 @Valid / @Validated
public R<Void> save(@Valid @RequestBody UserDTO userDTO) { }
```

- DTO 字段必须标注校验注解：`@NotBlank`, `@NotNull`, `@Size`, `@Email`
- 路径参数必须校验合法性
- 列表/分页参数需限制最大值

### 2.2 SQL 注入防护

- 禁止使用字符串拼接构造 SQL
- 统一使用 MyBatis 参数化查询 `#{param}`
- `${param}` 仅用于排序字段和表名，且必须做白名单校验
- Mapper XML 中的动态条件使用 `<if>` 标签

### 2.3 XSS 防护

- 引入 `pig-common-xss` 模块启用 XSS 过滤
- 用户输入在入库前进行 HTML 转义
- 富文本内容使用白名单策略（如只允许指定 HTML 标签）

## 3. 数据安全

### 3.1 密码安全

- 用户密码使用 `PasswordEncoder` 加密存储（BCrypt）
- 禁止明文存储任何密码
- 日志中禁止打印密码和 Token
- API 返回数据中不包含密码字段

### 3.2 配置文件加密

- 使用 Jasypt 加密配置文件中的敏感信息：`ENC(密文)`
- 加密密钥不应硬编码在代码中
- 生产环境密钥通过环境变量传入

### 3.3 文件上传安全

- 文件类型必须做白名单校验
- 文件大小限制合理配置
- 上传目录不允许执行权限
- 文件名应使用 UUID 重命名，避免路径穿越

## 4. 接口安全

### 4.1 验证码

- 登录接口必须有图形验证码
- 验证码有时效和次数限制
- 验证码验证后应立即失效

### 4.2 防重放攻击

- 关键操作接口应增加请求签名或时间戳校验
- Token 应具备唯一性标识（jti）

### 4.3 限流

- 对外暴露的接口应配置合理的限流策略
- Gateway 层配置全局限流
- 验证码接口需要更严格的限流

## 5. 日志审计

- 敏感操作（登录、权限变更、数据删除）必须记录审计日志
- 审计日志使用 `pig-common-log` 的 `@SysLog` 注解
- 日志内容包含：操作人、操作时间、操作内容、IP 地址、结果
- 审计日志不可被普通用户删除

## 6. 依赖安全

- 定期升级 Spring Boot / Spring Cloud 版本修复已知漏洞
- 不使用存在已知漏洞的第三方依赖
- Maven 依赖扫描：`mvn org.owasp:dependency-check-maven:check`

## 7. 常见安全问题清单

| 风险 | 检查点 |
|------|--------|
| SQL 注入 | 是否全部使用 `#{}` 参数化 |
| XSS | 是否引入 XSS 过滤器 |
| CSRF | 是否使用了 CSRF Token |
| 未授权访问 | 接口是否有 `@HasPermission` |
| 密码泄露 | 日志/返回值中是否有密码 |
| 敏感配置 | 配置文件中是否有明文密码 |
| 越权操作 | 是否有数据权限控制 |
| Token 伪造 | 是否正确验证 Token 签名 |
