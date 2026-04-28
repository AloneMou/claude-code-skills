# PIG 代码审查清单

## 1. 架构与设计

- [ ] 新代码是否遵循了模块分层结构（Controller → Service → Mapper）
- [ ] Controller 层是否只处理参数接收、校验和返回，不包含业务逻辑
- [ ] 跨服务调用是否使用了 Feign 接口而非直接依赖
- [ ] 是否正确地使用了 `pig-common-*` 公共模块
- [ ] 新增的类是否放在了正确的包路径下

## 2. 编码规范

- [ ] 命名是否符合 [coding-standards.md](./coding-standards.md) 规范
- [ ] 类注释、方法 Javadoc 是否完整
- [ ] 是否使用了 spring-javaformat 格式化
- [ ] 代码是否可以通过 `mvn spring-javaformat:validate`
- [ ] Lombok 注解是否正确使用

## 3. 数据库相关

- [ ] Entity 是否使用了 `@TableName` 注解
- [ ] 主键策略是否正确（`IdType.ASSIGN_ID` 雪花算法）
- [ ] 逻辑删除字段是否标注 `@TableLogic`
- [ ] 自动填充字段是否标注 `@TableField(fill = FieldFill.INSERT/UPDATE)`
- [ ] SQL 是否存在 N+1 查询问题
- [ ] 批量操作是否使用了批量方法（`saveBatch` / `updateBatchById`）
- [ ] 分页查询是否使用了 `Page` 对象

## 4. 安全

- [ ] 接口是否有权限校验（`@HasPermission`）
- [ ] 是否对用户输入进行了校验（`@Valid` / `@Validated`）
- [ ] SQL 是否使用了参数化查询（防 SQL 注入）
- [ ] 输出是否做了 XSS 过滤
- [ ] 密码是否加密存储
- [ ] 敏感信息是否使用 Jasypt 加密
- [ ] Feign 调用是否传递了 `SecurityConstants.FROM` 标识

## 5. 事务

- [ ] 涉及多个数据库操作的方法是否标注了 `@Transactional`
- [ ] `@Transactional` 是否指定了 `rollbackFor = Exception.class`
- [ ] 分布式事务场景是否使用了 Seata
- [ ] 事务是否过于宽泛（应尽量缩小事务范围）

## 6. 异常处理

- [ ] 是否正确抛出业务异常（`BusinessException`）
- [ ] 异常信息是否有意义
- [ ] 是否避免了捕获 `Exception` 或 `Throwable`
- [ ] `@RestControllerAdvice` 是否正确处理了自定义异常

## 7. 日志

- [ ] 关键业务流程是否打了 `log.info`
- [ ] 错误日志是否使用 `log.error` 并包含异常堆栈
- [ ] 日志中是否避免了打印敏感信息（密码、token）
- [ ] 是否正确使用了 `@Slf4j`

## 8. API 文档

- [ ] Controller 是否使用了 `@Tag` 注解
- [ ] 每个接口方法是否使用了 `@Operation` 描述
- [ ] 参数是否使用了 `@Parameter` 说明
- [ ] DTO/Entity 是否使用了 `@Schema` 说明

## 9. 性能

- [ ] 是否有不必要的数据库查询
- [ ] 循环中是否避免了数据库操作
- [ ] 大数据量是否做了分页处理
- [ ] 是否使用了适当的缓存（Redis）
- [ ] 是否存在内存泄漏风险

## 10. 配置

- [ ] 配置文件中的密码是否已加密
- [ ] 端口号是否与已有服务冲突
- [ ] 服务名称是否在 `ServiceNameConstants` 中定义
- [ ] Nacos 配置是否正确

## 11. 测试

- [ ] 新增的核心逻辑是否有对应的单元测试
- [ ] 测试是否覆盖了正常和异常流程
- [ ] 测试是否可以独立运行（不依赖外部服务）

## 12. Git 提交

- [ ] 提交信息是否清晰地描述了变更内容
- [ ] 是否只提交了必要的文件
- [ ] 是否避免了提交本地配置文件
