---
name: ruoyi-review
description: Review RuoYi-Vue-Pro code for convention violations, missing patterns, security issues, and architectural inconsistencies following Yudao framework standards
trigger: When the user asks to review code, check code quality, audit a module, or says "review this", "代码审查"
---

# RuoYi Code Reviewer

Reviews code against 芋道 (Yudao) RuoYi-Vue-Pro conventions and best practices.

## Checklist

### Controller Layer

- [ ] Returns `CommonResult<T>`, not raw types
- [ ] Has `@Tag(name = "...")` and `@Operation(summary = "...")` for API docs
- [ ] Has `@PreAuthorize("@ss.hasPermission('...')")` on every endpoint
- [ ] Uses `@Valid` / `@Validated` on request bodies
- [ ] No business logic in Controller — delegates to Service
- [ ] Uses `@RequestParam` for query parameters, not path variables for IDs
- [ ] Export endpoint sets `pageSize = PAGE_SIZE_NONE`
- [ ] `@RequestMapping` follows pattern `/{module-prefix}/{resource}`

### Service Layer

- [ ] Interface + Impl separation exists
- [ ] Impl has `@Service` and `@Validated`
- [ ] Private `validateXxx` methods for business validation
- [ ] Uses `throw exception(ERROR_CODE)` for business errors
- [ ] Uses `BeanUtils.toBean()` for simple conversion
- [ ] Uses MapStruct for complex conversion (list operations, nested objects)
- [ ] No direct Mapper calls from Controller

### Mapper Layer

- [ ] Extends `BaseMapperX<T>`, not `BaseMapper<T>`
- [ ] Custom queries are `default` methods, not XML
- [ ] Uses `LambdaQueryWrapperX` with `*IfPresent` methods
- [ ] Has `@Mapper` annotation
- [ ] `orderByDesc` is specified for pagination queries

### DO (Entity) Layer

- [ ] Extends `BaseDO` (has createTime/updateTime/creator/updater/deleted)
- [ ] Has `@TableName` with correct table name
- [ ] Has `@KeySequence` for sequence-supporting databases
- [ ] Has `@TableId` on the ID field
- [ ] No manual audit field assignments
- [ ] Status/type fields have enum reference in JavaDoc
- [ ] Uses Lombok `@Data`, `@EqualsAndHashCode(callSuper = true)`

### VO Layer

- [ ] `PageReqVO` extends `PageParam`
- [ ] `SaveReqVO` has `id` field (nullable for create, required for update)
- [ ] `RespVO` has `createTime` field
- [ ] All fields have `@Schema(description = "...")` 
- [ ] Validation annotations (`@NotBlank`, `@NotNull`, `@Size`) present
- [ ] Excel annotations (`@ExcelProperty`) on export fields

### Error Codes

- [ ] Error codes defined in `ErrorCodeConstants` interface
- [ ] Codes follow `1_XXX_YYY_ZZZ` format with underscores
- [ ] Codes use descriptive constants (`POST_NOT_FOUND`, not `ERROR_001`)
- [ ] Code exists for every `throw exception(...)` call

### Security

- [ ] No hardcoded credentials or secrets
- [ ] No SQL injection risks (parameterized queries only)
- [ ] Permission checks on all admin endpoints
- [ ] Sensitive data masked in responses (phone, email, ID card)
- [ ] Uses `jakarta.*` not `javax.*`

### Common Issues to Flag

1. **Missing permission** — controller endpoint without `@PreAuthorize`
2. **Raw return type** — method returns `T` instead of `CommonResult<T>`
3. **Business logic in Controller** — data manipulation instead of service delegation
4. **No validation** — request body without `@Valid`
5. **Manual audit fields** — setting createTime/updateTime manually
6. **Hardcoded error strings** — `"用户不存在"` instead of `exception(USER_NOT_FOUND)`
7. **Missing API docs** — no `@Operation` annotation
8. **Wrong wrapper** — using `QueryWrapper` instead of `LambdaQueryWrapperX`
9. **XML for simple queries** — SQL in XML that could be a default method
10. **Circular dependency** — module A calls B, B calls A
