# PIG 编码规范

## 1. 命名规范

### 1.1 包命名

- 全部小写，使用点号分隔：`com.pig4cloud.pig.admin`
- 模块包结构统一为：`com.pig4cloud.pig.{module}`

### 1.2 类命名

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| Controller | `XxxController` | `SysUserController` |
| Service 接口 | `XxxService` | `SysUserService` |
| Service 实现 | `XxxServiceImpl` | `SysUserServiceImpl` |
| Mapper/DAO | `XxxMapper` | `SysUserMapper` |
| Entity | `Xxx` / `SysXxx` | `SysUser`, `SysRole` |
| DTO | `XxxDTO` / `XxxReq` / `XxxResp` | `UserDTO`, `UserReq` |
| VO | `XxxVO` | `UserVO` |
| 常量类 | `XxxConstants` | `SecurityConstants` |
| 工具类 | `XxxUtil` | `SecurityUtil` |
| 配置类 | `XxxConfiguration` / `XxxConfig` | `MybatisPlusConfig` |
| 过滤器 | `XxxFilter` | `ValidateCodeFilter` |
| 异常类 | `XxxException` | `ValidateCodeException` |

### 1.3 方法命名

- 查询单个：`getById`, `getByXxx`, `selectOne`
- 查询列表：`list`, `listByXxx`, `page`
- 新增：`save`, `add`, `insert`
- 更新：`update`, `updateById`
- 删除：`remove`, `removeById`, `delete`
- 判断：`isXxx`, `hasXxx`, `exists`
- 统计：`count`, `countByXxx`

### 1.4 变量命名

- 普通变量：小驼峰 `userId`, `userName`
- 常量：全大写 + 下划线 `MAX_PAGE_SIZE`, `DEFAULT_ENCODING`
- 布尔变量：使用 `is`, `has`, `can` 前缀 `isAdmin`, `hasPermission`

## 2. 代码风格

### 2.1 spring-javaformat（强制）

项目使用 [spring-javaformat](https://github.com/spring-io/spring-javaformat) 强制代码格式：

```bash
# 提交前执行
mvn spring-javaformat:apply
```

- 缩进使用 **Tab**（非空格）
- 每行最大长度 **90** 字符
- 导入语句按 spring-javaformat 规则自动排序
- **格式化不通过的代码不能合并到主分支**

### 2.2 Lombok 使用

```java
@Data              // getter/setter/toString/equals/hashCode
@Builder           // 建造者模式
@NoArgsConstructor // 无参构造
@AllArgsConstructor// 全参构造
@Slf4j             // 日志
@Accessors(chain = true) // 链式调用
```

### 2.3 注解规范

- Controller 类使用 `@RestController` + `@RequestMapping("/xxx")`
- Service 实现类使用 `@Service` + `@RequiredArgsConstructor`
- Mapper 接口使用 `@Mapper`
- 配置类使用 `@Configuration` + `@EnableXxx`
- 配置属性类使用 `@ConfigurationProperties(prefix = "xxx")` + `@Data`

## 3. 注释规范

### 3.1 类注释

```java
/**
 * 用户信息 Controller
 *
 * @author pigx
 * @date 2024-01-01
 */
```

### 3.2 方法注释

- 公共 API 方法必须有 Javadoc
- 内部方法可使用 `//` 单行注释
- 复杂业务逻辑使用 `//` 分段注释

### 3.3 Swagger 注解

```java
@Operation(summary = "获取用户信息", description = "根据用户ID获取用户详细信息")
@Parameter(name = "userId", description = "用户ID")
```

## 4. 分层规范

### 4.1 Controller 层

- 只做参数接收、参数校验、调用 Service、返回结果
- 不包含任何业务逻辑
- 统一使用 `R<T>` 作为返回类型
- 路径统一使用小写 + 中划线：`/sys-user`

### 4.2 Service 层

- 接口定义方法签名
- 实现类编写业务逻辑
- 继承 `IService<Entity>` 使用 MyBatis Plus 基类
- 事务注解 `@Transactional` 标注在实现类方法上

### 4.3 Mapper 层

- 继承 `BaseMapper<Entity>`
- 自定义查询使用 XML 映射文件或 `@Select` 注解
- 复杂查询使用 `LambdaQueryWrapper`

## 5. 异常处理

- 使用 `@RestControllerAdvice` 全局异常处理器
- 业务异常使用自定义 `BusinessException`
- 不捕获 `Exception`，只捕获具体异常类型
- 异常信息使用有意义的中文描述

## 6. 日志规范

- 使用 `@Slf4j` 注解
- `log.debug`：调试信息
- `log.info`：关键业务流程
- `log.warn`：可恢复的异常情况
- `log.error`：错误信息，必须包含异常堆栈
