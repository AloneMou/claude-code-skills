---
name: ruoyi-review
description: 审查芋道 RuoYi-Vue-Pro 项目代码，检查规范违反、缺失模式、安全问题和架构不一致性
trigger: 当用户要求审查代码、检查代码质量、审计模块、或提到 "review this"、"代码审查"、"check this code"、"代码规范检查"
---

# RuoYi Code Reviewer

审查代码是否符合芋道 (Yudao) RuoYi-Vue-Pro 框架规范和最佳实践。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取完整的项目级规则。

## TRIGGER

- 用户要求审查代码质量
- 提到 "review this"、"代码审查"、"代码规范检查"
- 新建模块后自动检查生成的代码

## Review Checklist

### Controller 层

- [ ] 返回 `CommonResult<T>`，非裸类型
- [ ] 有 `@Tag(name = "...")` 和 `@Operation(summary = "...")`
- [ ] 每个接口有 `@PreAuthorize("@ss.hasPermission('...')")`
- [ ] 请求体使用 `@Valid` / `@Validated` 校验
- [ ] Controller 无业务逻辑 — 全部委托给 Service
- [ ] 查询参数使用 `@RequestParam`，ID 不用 PathVariable
- [ ] 导出接口设置 `pageSize = PAGE_SIZE_NONE`
- [ ] `@RequestMapping` 遵循 `/{模块前缀}/{资源}` 格式

### Service 层

- [ ] Interface + Impl 分离
- [ ] Impl 有 `@Service` 和 `@Validated`
- [ ] 私有 `validateXxx` 方法做业务校验
- [ ] 业务异常用 `throw exception(ERROR_CODE)`
- [ ] 简单转换用 `BeanUtils.toBean()`
- [ ] 复杂转换用 MapStruct
- [ ] Controller 不直接调用 Mapper

### Mapper 层

- [ ] 继承 `BaseMapperX<T>`，非 `BaseMapper<T>`
- [ ] 自定义查询用 `default` 方法，非 XML
- [ ] 使用 `LambdaQueryWrapperX` 的 `*IfPresent` 方法
- [ ] 有 `@Mapper` 注解
- [ ] 分页查询指定了 `orderByDesc`

### DO (实体) 层

- [ ] 继承 `BaseDO`（含审计字段）
- [ ] 有 `@TableName` 且表名正确
- [ ] 有 `@KeySequence`（兼容 Oracle/PG 等）
- [ ] ID 字段有 `@TableId`
- [ ] 不手动赋值审计字段
- [ ] 状态/类型字段有枚举引用 JavaDoc
- [ ] 使用 Lombok `@Data`、`@EqualsAndHashCode(callSuper = true)`

### VO 层

- [ ] `PageReqVO` 继承 `PageParam`
- [ ] `SaveReqVO` 有 `id` 字段（更新时非空）
- [ ] `RespVO` 有 `createTime` 字段
- [ ] 所有字段有 `@Schema(description = "...")`
- [ ] 校验注解（`@NotBlank`、`@NotNull`、`@Size`）齐全
- [ ] 导出字段有 `@ExcelProperty`

### 错误码

- [ ] 错误码定义在 `ErrorCodeConstants` 接口
- [ ] 代码使用 `1_XXX_YYY_ZZZ` 下划线格式
- [ ] 使用描述性常量名（`POST_NOT_FOUND`，非 `ERROR_001`）
- [ ] 每个 `throw exception(...)` 都有对应错误码

### 安全

- [ ] 无硬编码凭据或密钥
- [ ] 无 SQL 注入风险（仅参数化查询）
- [ ] 管理端点有权限校验
- [ ] 敏感数据脱敏（手机号、邮箱、身份证）
- [ ] 使用 `jakarta.*` 而非 `javax.*`

## Common Issues to Flag

1. **缺少权限** — Controller 接口无 `@PreAuthorize`
2. **裸返回类型** — 返回 `T` 而非 `CommonResult<T>`
3. **Controller 含业务逻辑** — 数据处理而非服务委托
4. **缺少校验** — 请求体无 `@Valid`
5. **手动审计字段** — 手动设置 createTime/updateTime
6. **硬编码错误信息** — `"用户不存在"` 而非 `exception(USER_NOT_FOUND)`
7. **缺少 API 文档** — 无 `@Operation` 注解
8. **错误的 Wrapper** — 用 `QueryWrapper` 而非 `LambdaQueryWrapperX`
9. **XML 写简单查询** — 可用 default 方法替代的 SQL
10. **循环依赖** — 模块 A 调 B，B 调 A