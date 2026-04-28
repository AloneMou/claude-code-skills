# PIG 模块开发规则

## 1. 新增业务模块流程

### 1.1 创建模块

1. 在根 `pom.xml` 的 `<modules>` 中添加新模块
2. 创建模块目录结构（参考 [package-rules.md](./package-rules.md)）
3. 创建模块 `pom.xml`，继承父 POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig</artifactId>
        <version>${revision}</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>pig-{新模块}</artifactId>
    <packaging>pom</packaging>
    <description>新模块描述</description>
    <modules>
        <module>pig-{新模块}-api</module>
        <module>pig-{新模块}-biz</module>
    </modules>
</project>
```

### 1.2 API 模块 pom.xml

```xml
<dependencies>
    <!-- 核心依赖 -->
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-common-core</artifactId>
    </dependency>
    <!-- Feign 依赖（跨服务调用） -->
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-common-feign</artifactId>
    </dependency>
</dependencies>
```

### 1.3 BIZ 模块 pom.xml

```xml
<dependencies>
    <!-- API 模块 -->
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-{模块}-api</artifactId>
    </dependency>
    <!-- 按需引入功能模块 -->
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-common-mybatis</artifactId>
    </dependency>
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-common-security</artifactId>
    </dependency>
    <dependency>
        <groupId>com.pig4cloud</groupId>
        <artifactId>pig-common-swagger</artifactId>
    </dependency>
</dependencies>
```

## 2. 各模块开发要点

### 2.1 pig-gateway（网关）

- 使用 Spring Cloud Gateway
- 路由配置在 `application.yml` 或 Nacos 配置中心
- 全局过滤器实现：认证、限流、日志
- 不编写业务代码，只做路由转发和过滤器

### 2.2 pig-auth（授权服务）

- 基于 Spring Authorization Server
- 实现 OAuth2 授权码模式、密码模式等
- 自定义用户认证逻辑
- 注意 token 的生成、刷新、验证
- 修改鉴权逻辑需同步考虑 pig-gateway 的过滤器

### 2.3 pig-upms（用户权限管理）

- 用户、角色、菜单、部门等 RBAC 模型
- 数据权限处理（部门数据隔离）
- 菜单权限树构建
- 与 pig-common-security 配合实现权限校验

### 2.4 pig-register（注册中心）

- 基于 Nacos 定制
- 一般不需要修改，直接使用官方版本

### 2.5 pig-visual（可视化模块）

- **pig-monitor**：服务监控，基于 Spring Boot Admin
- **pig-codegen**：代码生成，模板引擎
- **pig-quartz**：定时任务，基于 Quartz

## 3. MyBatis Plus 使用规则

### 3.1 Entity 实体类

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

### 3.2 Mapper 接口

```java
@Mapper
public interface SysUserMapper extends BaseMapper<SysUser> {
    // 简单 CRUD 直接使用 BaseMapper
    // 复杂查询写在 XML 或使用 LambdaQueryWrapper
}
```

### 3.3 Service 接口

```java
public interface SysUserService extends IService<SysUser> {
    UserDTO getUserInfo(Long userId);
    void updateUser(UserDTO userDTO);
}
```

### 3.4 Service 实现

```java
@Service
@RequiredArgsConstructor
public class SysUserServiceImpl extends ServiceImpl<SysUserMapper, SysUser>
        implements SysUserService {

    private final PasswordEncoder passwordEncoder;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void updateUser(UserDTO userDTO) {
        // 业务逻辑
    }
}
```

## 4. Controller 编写规范

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/sys-user")
@Tag(description = "sys-user", name = "用户管理")
@SecurityRequirement(name = HttpHeaders.AUTHORIZATION)
public class SysUserController {

    private final SysUserService sysUserService;

    @GetMapping("/{id}")
    @Operation(summary = "获取用户信息")
    public R<SysUser> getById(@PathVariable Long id) {
        return R.ok(sysUserService.getById(id));
    }

    @PostMapping
    @Operation(summary = "新增用户")
    public R<Void> save(@Valid @RequestBody UserDTO userDTO) {
        sysUserService.saveUser(userDTO);
        return R.ok();
    }

    @PutMapping
    @Operation(summary = "修改用户")
    public R<Void> update(@Valid @RequestBody UserDTO userDTO) {
        sysUserService.updateUser(userDTO);
        return R.ok();
    }

    @DeleteMapping("/{id}")
    @Operation(summary = "删除用户")
    public R<Void> removeById(@PathVariable Long id) {
        sysUserService.removeUser(id);
        return R.ok();
    }
}
```

## 5. Feign 远程调用

### 5.1 定义接口（api 模块）

```java
@FeignClient(contextId = "remoteUserService", value = ServiceNameConstants.UPMS_SERVICE)
public interface RemoteUserService {
    @GetMapping("/sys-user/info/{username}")
    R<UserInfo> info(@PathVariable("username") String username,
                     @RequestHeader(SecurityConstants.FROM) String from);
}
```

### 5.2 使用接口

```java
@Service
@RequiredArgsConstructor
public class SomeService {
    private final RemoteUserService remoteUserService;

    public void doSomething(String username) {
        R<UserInfo> result = remoteUserService.info(username, SecurityConstants.FROM_IN);
        // 处理结果
    }
}
```

## 6. 全局返回结果

统一使用 `R<T>` 作为返回类型：

```java
// 成功
return R.ok(data);
return R.ok();

// 失败
return R.failed("错误信息");
```
