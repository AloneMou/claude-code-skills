# CLAUDE.md — 芋道 RuoYi-Vue-Pro (JDK17) 项目规则

> 本文档由 Claude Code 自动读取，定义本项目的编码规范、架构约定和开发规则。

---

## 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Java | 17 | |
| Spring Boot | 3.5.9 | Jakarta EE 命名空间 |
| MyBatis-Plus | 3.5.15 | ORM 框架 |
| MyBatis-Plus-Join | 1.5.5 | 关联查询 |
| Druid | 1.2.27 | 连接池 |
| Redis (Redisson) | 3.52.0 | 缓存/分布式锁 |
| Dynamic Datasource | 4.5.0 | 多数据源 |
| Lombok | 1.18.42 | |
| MapStruct | 1.6.3 | 对象转换 |
| Hutool | 5.8.42 | 工具库 |
| SpringDoc / Knife4j | 2.8.14 / 4.5.0 | API 文档 |
| Flowable | 7.2.0 | 工作流 |
| FastExcel | 1.3.0 | Excel 导入导出 |

---

## 项目结构

```
yudao-dependencies/          # BOM，统一管理所有依赖版本
yudao-framework/             # 框架 starter（web/security/mybatis/redis 等）
yudao-server/                # 启动入口，聚合业务模块
yudao-module-system/         # 系统核心（用户/角色/菜单/部门/租户等）
yudao-module-infra/          # 基础设施（代码生成/文件/配置/API日志）
yudao-module-*/              # 业务模块（ai/bpm/crm/erp/iot/mall/member/pay 等）
sql/                         # 数据库初始化脚本（多种数据库）
script/                      # 部署脚本（Docker 等）
```

**基础包名**: `cn.iocoder.yudao`

---

## 模块内部包结构

每个业务模块 (`yudao-module-{module}`) 内部结构：

```
cn.iocoder.yudao.module.{module}/
  api/                      # 跨模块 API 接口（RPC/服务暴露）
    {submodule}/
      XxxApi.java            # API 接口定义
      dto/
        XxxReqDTO.java       # 跨模块请求 DTO
        XxxRespDTO.java      # 跨模块响应 DTO
  controller/
    admin/                   # 管理后台 Controller
      {submodule}/
        XxxController.java
        vo/
          XxxPageReqVO.java  # 分页查询请求
          XxxSaveReqVO.java  # 新增/修改请求
          XxxRespVO.java     # 响应 VO
          XxxSimpleRespVO.java # 简化响应（下拉选择）
    app/                     # 用户端 Controller（部分模块）
  convert/                   # MapStruct 转换器
    XxxConvert.java
  dal/
    dataobject/              # 数据库实体（DO = Data Object）
      {submodule}/
        XxxDO.java
    mysql/                   # MyBatis Mapper 接口
      {submodule}/
        XxxMapper.java
    redis/                   # Redis DAO
      XxxRedisDAO.java
  enums/                     # 枚举
    ErrorCodeConstants.java  # 模块错误码
    XxxTypeEnum.java
  service/                   # 业务逻辑
    {submodule}/
      XxxService.java        # 接口
      XxxServiceImpl.java    # 实现
  job/                       # 定时任务
  mq/                        # 消息队列
  util/                      # 模块工具类
```

---

## 命名约定

| 类型 | 命名模式 | 示例 |
|------|----------|------|
| 实体类 (Data Object) | `XxxDO` | `PostDO`, `AdminUserDO` |
| Mapper 接口 | `XxxMapper` | `PostMapper`, `AdminUserMapper` |
| Service 接口 | `XxxService` | `PostService`, `DeptService` |
| Service 实现 | `XxxServiceImpl` | `PostServiceImpl` |
| Controller | `XxxController` | `PostController` |
| 分页请求 VO | `XxxPageReqVO` | `PostPageReqVO` |
| 保存请求 VO | `XxxSaveReqVO` | `PostSaveReqVO` |
| 响应 VO | `XxxRespVO` | `PostRespVO` |
| 简化响应 VO | `XxxSimpleRespVO` | `PostSimpleRespVO` |
| 跨模块 DTO | `XxxReqDTO` / `XxxRespDTO` | `SmsCodeSendReqDTO` |
| 枚举 | `XxxEnum` | `CommonStatusEnum` |
| 错误码常量接口 | `ErrorCodeConstants` | `POST_NOT_FOUND` |

---

## Controller 规范

```java
@Tag(name = "管理后台 - 岗位")
@RestController
@RequestMapping("/system/post")
@Validated
public class PostController {

    @Resource
    private PostService postService;

    @PostMapping("/create")
    @Operation(summary = "创建岗位")
    @PreAuthorize("@ss.hasPermission('system:post:create')")
    public CommonResult<Long> createPost(@Valid @RequestBody PostSaveReqVO createReqVO) { ... }

    @PutMapping("/update")
    @Operation(summary = "修改岗位")
    @PreAuthorize("@ss.hasPermission('system:post:update')")
    public CommonResult<Boolean> updatePost(@Valid @RequestBody PostSaveReqVO updateReqVO) { ... }

    @DeleteMapping("/delete")
    @Operation(summary = "删除岗位")
    @PreAuthorize("@ss.hasPermission('system:post:delete')")
    public CommonResult<Boolean> deletePost(@RequestParam("id") Long id) { ... }

    @DeleteMapping("/delete-list")
    @Operation(summary = "批量删除")
    @PreAuthorize("@ss.hasPermission('system:post:delete')")
    public CommonResult<Boolean> deletePostList(@RequestParam("ids") List<Long> ids) { ... }

    @GetMapping("/get")
    @Operation(summary = "获得详情")
    @PreAuthorize("@ss.hasPermission('system:post:query')")
    public CommonResult<PostRespVO> getPost(@RequestParam("id") Long id) { ... }

    @GetMapping("/page")
    @Operation(summary = "获得分页列表")
    @PreAuthorize("@ss.hasPermission('system:post:query')")
    public CommonResult<PageResult<PostRespVO>> getPostPage(@Validated PostPageReqVO pageReqVO) { ... }

    @GetMapping("/export-excel")
    @Operation(summary = "导出Excel")
    @PreAuthorize("@ss.hasPermission('system:post:export')")
    @ApiAccessLog(operateType = EXPORT)
    public void export(HttpServletResponse response, @Validated PostPageReqVO reqVO) { ... }
}
```

**关键规则**:
- 所有接口返回 `CommonResult<T>`
- 权限用 `@PreAuthorize("@ss.hasPermission('模块:资源:操作')")`
- 请求体用 `@Valid` 校验，类级别加 `@Validated`
- 查询参数用 `@RequestParam`
- 导出接口设 `pageSize = PAGE_SIZE_NONE`（-1）获取全量
- API 文档用 `@Tag` 和 `@Operation`（SpringDoc）
- 下拉选择接口路径 `/list-all-simple`，返回 `XxxSimpleRespVO`

---

## Entity (DO) 规范

```java
@TableName("system_post")
@KeySequence("system_post_seq")  // Oracle/PG/Kingbase/DB2/H2 序列
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class PostDO extends BaseDO {

    @TableId
    private Long id;
    private String postCode;
    private String name;
    private Integer sort;
    private Integer status;  // 参见 CommonStatusEnum
    private String remark;
}
```

**BaseDO 提供的字段** (自动填充):

| 字段 | 类型 | 填充策略 |
|------|------|----------|
| `createTime` | `LocalDateTime` | INSERT |
| `updateTime` | `LocalDateTime` | INSERT_UPDATE |
| `creator` | `String` | INSERT |
| `updater` | `String` | INSERT_UPDATE |
| `deleted` | `Boolean` | 逻辑删除 `@TableLogic` |

**规则**:
- 所有 DO 必须继承 `BaseDO`
- 字段用 Java 驼峰命名，MyBatis-Plus 自动映射为下划线数据库字段
- 状态/类型字段用 `Integer`，枚举值在 JavaDoc 中注明
- 不要手动设置 `createTime`/`updateTime`/`creator`/`updater`

---

## Mapper 规范

```java
@Mapper
public interface PostMapper extends BaseMapperX<PostDO> {

    default List<PostDO> selectList(Collection<Long> ids, Collection<Integer> statuses) {
        return selectList(new LambdaQueryWrapperX<PostDO>()
                .inIfPresent(PostDO::getId, ids)
                .inIfPresent(PostDO::getStatus, statuses));
    }

    default PageResult<PostDO> selectPage(PostPageReqVO reqVO) {
        return selectPage(reqVO, new LambdaQueryWrapperX<PostDO>()
                .likeIfPresent(PostDO::getCode, reqVO.getCode())
                .likeIfPresent(PostDO::getName, reqVO.getName())
                .eqIfPresent(PostDO::getStatus, reqVO.getStatus())
                .orderByDesc(PostDO::getId));
    }
}
```

**规则**:
- 必须继承 `BaseMapperX<T>`（封装了 MyBatis-Plus Join 的 `MPJBaseMapper`）
- 自定义查询用 `default` 方法，优先避免 XML
- 条件查询用 `LambdaQueryWrapperX` 的 `*IfPresent` 方法（自动跳过 null 值）
- 分页查询用 `selectPage(reqVO, wrapper)` 方法
- 多表 Join 用 `MPJLambdaWrapperX`

---

## Service 规范

```java
public interface PostService {
    Long createPost(PostSaveReqVO createReqVO);
    void updatePost(PostSaveReqVO updateReqVO);
    void deletePost(Long id);
    PostDO getPost(Long id);
    PageResult<PostDO> getPostPage(PostPageReqVO reqVO);
}

@Service
@Validated
public class PostServiceImpl implements PostService {

    @Resource
    private PostMapper postMapper;

    @Override
    public Long createPost(PostSaveReqVO createReqVO) {
        // 1. 校验
        validatePostForCreateOrUpdate(null, createReqVO.getName(), createReqVO.getCode());
        // 2. 转换
        PostDO post = BeanUtils.toBean(createReqVO, PostDO.class);
        // 3. 插入
        postMapper.insert(post);
        return post.getId();
    }

    private void validatePostForCreateOrUpdate(Long id, String name, String code) {
        // 校验逻辑：唯一性、存在性等
        PostDO post = postMapper.selectById(id);
        if (post == null) {
            throw exception(POST_NOT_FOUND);
        }
    }
}
```

**规则**:
- 必须有 Interface + Impl 两层
- Impl 加 `@Service` 和 `@Validated`
- Service 方法参数用 VO，返回 VO 或 DO（分页返回 `PageResult<DO>`）
- 私有 `validateXxx` 方法处理业务校验
- 用 `BeanUtils.toBean()` 做简单转换，复杂转换用 MapStruct
- 业务异常用 `throw exception(ERROR_CODE)` 抛出 `ServiceException`

---

## 响应与分页

**统一响应**: `CommonResult<T>`
- `CommonResult.success(data)` — 成功
- `CommonResult.error(code, message)` — 失败
- `code = 0` 表示成功

**分页**:
```java
public class PageParam {
    private Integer pageNo = 1;     // 从 1 开始
    private Integer pageSize = 10;  // 最大 200
    public static final Integer PAGE_SIZE_NONE = -1;  // 不分页
}

public class PageResult<T> {
    private Long total;
    private List<T> list;
}
```

---

## 错误码规范

- 全局错误码: 0-999（0=成功，401=未登录，403=无权限，500=系统异常等）
- 业务错误码: `1-XXX-YYY-ZZZ` 格式
  - 第1位: 类型（1=业务）
  - 第2-4位: 系统（001=infra, 002=system）
  - 第5-7位: 模块
  - 第8-10位: 具体错误

每个模块 `enums/ErrorCodeConstants.java` 中定义：
```java
// POST 模块 1-002-001-000
ErrorCode POST_NOT_FOUND = new ErrorCode(1_002_001_000, "岗位不存在");
ErrorCode POST_NAME_DUPLICATE = new ErrorCode(1_002_001_001, "岗位名称已存在");
ErrorCode POST_CODE_DUPLICATE = new ErrorCode(1_002_001_002, "岗位编码已存在");
```

---

## 安全与权限

- Token 无状态认证，`Authorization: Bearer <token>`
- 权限用 `@PreAuthorize("@ss.hasPermission('模块:资源:操作')")`
- `@PermitAll` 标注公开接口（自动加入白名单）
- 配置项 `yudao.security.permit-all-urls` 可额外配置公开路径
- 登录用户信息通过 `SecurityFrameworkUtils.getLoginUserId()` 获取

---

## 日志

- Logback，`logback-spring.xml` 配置
- 控制台带颜色，文件不带颜色
- 异步 Appender，队列 512
- 日志级别: INFO 默认，WARN 校验错误，ERROR 系统异常

---

## 重要框架类参考

| 类 | 用途 |
|----|------|
| `BaseDO` | 实体基类（含审计字段） |
| `BaseMapperX<T>` | Mapper 基类 |
| `LambdaQueryWrapperX<T>` | 条件构造器（*IfPresent） |
| `MPJLambdaWrapperX<T>` | 多表 Join 构造器 |
| `CommonResult<T>` | 统一响应包装 |
| `PageResult<T>` | 分页结果 |
| `PageParam` | 分页参数基类 |
| `BeanUtils` | 对象转换（Hutool BeanUtil 封装） |
| `ServiceExceptionUtil` | 业务异常工具类 |
| `SecurityFrameworkUtils` | 安全框架工具类 |
| `ValidationUtils` | 校验工具类 |

---

## 禁止事项

1. **不要** 手动设置 `createTime`/`updateTime`/`creator`/`updater`/`deleted` — BaseDO 自动填充
2. **不要** 绕过 `CommonResult` 直接返回裸数据
3. **不要** 在 Controller 写业务逻辑 — 全部放在 Service
4. **不要** 直接用 MyBatis-Plus 原生 `QueryWrapper` — 用 `LambdaQueryWrapperX`
5. **不要** 在 XML 写简单查询 — 用 Mapper 的 `default` 方法
6. **不要** 硬编码错误信息 — 用 `ErrorCodeConstants`
7. **不要** 忽略 `@PreAuthorize` 权限注解
8. **不要** 使用 `javax.*` 包 — Spring Boot 3 用 `jakarta.*`
