# Controller 开发规范

> 定义管理后台和用户端 Controller 的开发标准。

## 基本结构

```java
@Tag(name = "管理后台 - 岗位")
@RestController
@RequestMapping("/system/post")
@Validated
public class PostController {

    @Resource
    private PostService postService;

    // 接口方法...
}
```

## 接口模板

| 方法 | 路径 | 权限 | 返回类型 |
|------|------|------|----------|
| POST | `/create` | `{module}:{resource}:create` | `CommonResult<Long>` |
| PUT | `/update` | `{module}:{resource}:update` | `CommonResult<Boolean>` |
| DELETE | `/delete?id=` | `{module}:{resource}:delete` | `CommonResult<Boolean>` |
| DELETE | `/delete-list?ids=` | `{module}:{resource}:delete` | `CommonResult<Boolean>` |
| GET | `/get?id=` | `{module}:{resource}:query` | `CommonResult<XxxRespVO>` |
| GET | `/page?...` | `{module}:{resource}:query` | `CommonResult<PageResult<XxxRespVO>>` |
| GET | `/list-all-simple` | 无 | `CommonResult<List<XxxSimpleRespVO>>` |
| GET | `/export-excel` | `{module}:{resource}:export` | `void` (直接写入 response) |

## 注解规范

### 必须注解

| 注解 | 用途 | 示例 |
|------|------|------|
| `@Tag` | API 分组（类级别） | `@Tag(name = "管理后台 - 岗位")` |
| `@Operation` | 接口描述 | `@Operation(summary = "创建岗位")` |
| `@PreAuthorize` | 权限控制 | `@PreAuthorize("@ss.hasPermission('system:post:create')")` |
| `@Valid` | 请求体校验（方法参数） | `@Valid @RequestBody PostSaveReqVO` |
| `@Validated` | 查询参数校验（类级别） | 类上加 `@Validated` |

### 可选注解

| 注解 | 用途 |
|------|------|
| `@PermitAll` | 公开接口，跳过认证 |
| `@ApiAccessLog` | 操作日志记录（导出时标记 `EXPORT`） |
| `@Parameter` | 单个参数描述 |

## 参数传递规则

- **请求体**（POST/PUT）: 用 `@RequestBody` + `@Valid`
- **查询参数**（GET/DELETE）: 用 `@RequestParam`
- **分页参数**: 直接接收 VO 对象，用 `@Validated` 不加 `@Valid`
- **导出参数**: 接收 PageReqVO 后设置 `pageSize = PAGE_SIZE_NONE`

## 响应规则

- 所有接口统一返回 `CommonResult<T>`
- 成功用 `CommonResult.success(data)`
- 失败通过 `ServiceException` 抛出，框架自动包装为 error 响应
- 禁止直接返回裸数据或自定义响应格式

## 导出接口模板

```java
@GetMapping("/export-excel")
@Operation(summary = "导出Excel")
@PreAuthorize("@ss.hasPermission('system:post:export')")
@ApiAccessLog(operateType = EXPORT)
public void exportPostExcel(HttpServletResponse response, @Validated PostPageReqVO reqVO)
        throws IOException {
    reqVO.setPageSize(PageParam.PAGE_SIZE_NONE);
    List<PostDO> list = postService.getPostPage(reqVO).getList();
    ExcelUtils.write(response, "岗位数据.xls", "岗位列表", PostRespVO.class,
            BeanUtils.toBean(list, PostRespVO.class));
}
```

## Controller 禁止行为

1. **禁止** 在 Controller 中写业务逻辑 — 全部委托给 Service
2. **禁止** 直接调用 Mapper — 必须经过 Service
3. **禁止** 绕过 `@PreAuthorize` — 每个接口必须有权限注解
4. **禁止** 返回非 `CommonResult` 类型（导出接口除外）
5. **禁止** 在接口方法签名中暴露 `HttpServletRequest`/`HttpServletResponse`（导出除外）
