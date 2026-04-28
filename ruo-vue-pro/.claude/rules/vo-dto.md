# VO/DTO 规范

> 定义请求 VO、响应 VO 和跨模块 DTO 的开发标准。

## VO 类型

所有 VO 位于模块的 `controller/admin/{submodule}/vo/` 包下。

### SaveReqVO（新增/修改请求）

```java
@Schema(description = "管理后台 - 岗位新增/修改")
@Data
public class PostSaveReqVO {

    @Schema(description = "岗位编号", example = "1")
    private Long id;

    @Schema(description = "岗位编码", requiredMode = Schema.RequiredMode.REQUIRED, example = "CEO")
    @NotBlank(message = "岗位编码不能为空")
    private String postCode;

    @Schema(description = "岗位名称", requiredMode = Schema.RequiredMode.REQUIRED, example = "CEO")
    @NotBlank(message = "岗位名称不能为空")
    private String name;

    @Schema(description = "显示顺序", requiredMode = Schema.RequiredMode.REQUIRED, example = "1")
    @NotNull(message = "显示顺序不能为空")
    private Integer sort;

    @Schema(description = "状态", example = "0")
    private Integer status;

    @Schema(description = "备注", example = "CEO")
    private String remark;
}
```

### PageReqVO（分页请求）

```java
@Schema(description = "管理后台 - 岗位分页查询")
@Data
@EqualsAndHashCode(callSuper = true)
public class PostPageReqVO extends PageParam {

    @Schema(description = "岗位编码", example = "CEO")
    private String postCode;

    @Schema(description = "岗位名称", example = "CEO")
    private String name;

    @Schema(description = "状态", example = "0")
    private Integer status;

    @Schema(description = "创建时间")
    @DateTimeFormat(pattern = FORMAT_YEAR_MONTH_DAY_HOUR_MINUTE_SECOND)
    private LocalDateTime[] createTime;
}
```

- 必须继承 `PageParam`（提供 `pageNo` 和 `pageSize`）
- 查询字段加 `@Schema` 描述，不加校验注解（由 Mapper 层处理）
- 时间范围查询用 `LocalDateTime[]` + `@DateTimeFormat`

### RespVO（响应 VO）

```java
@Schema(description = "管理后台 - 岗位信息")
@Data
public class PostRespVO {

    @Schema(description = "岗位编号", example = "1")
    private Long id;

    @Schema(description = "岗位编码", example = "CEO")
    @ExcelProperty("岗位编码")
    private String postCode;

    @Schema(description = "岗位名称", example = "CEO")
    @ExcelProperty("岗位名称")
    private String name;

    @Schema(description = "显示顺序", example = "1")
    @ExcelProperty("显示顺序")
    private Integer sort;

    @Schema(description = "状态", example = "0")
    @ExcelProperty("状态")
    private Integer status;

    @Schema(description = "创建时间")
    @ExcelProperty("创建时间")
    private LocalDateTime createTime;
}
```

### SimpleRespVO（简化响应，用于下拉选择）

```java
@Schema(description = "管理后台 - 岗位简化信息")
@Data
public class PostSimpleRespVO {

    @Schema(description = "岗位编号", example = "1")
    private Long id;

    @Schema(description = "岗位名称", example = "CEO")
    private String name;
}
```

## DTO 类型

跨模块 DTO 位于模块的 `api/{submodule}/dto/` 包下。

### RespDTO

```java
@Schema(description = "{业务名称} Response DTO")
@Data
public class XxxRespDTO {

    @Schema(description = "编号", example = "1")
    private Long id;

    @Schema(description = "名称", example = "CEO")
    private String name;

    // 仅暴露调用方需要的字段
}
```

### DTO 规则

1. **不得引用** 任何模块的 DO 或 VO — DTO 必须独立
2. **最小暴露** — 只包含调用方需要的字段
3. 使用 `@Data` 注解（Lombok）
4. 每个字段加 `@Schema` 描述

## 注解规范

| 注解 | 用途 | 示例 |
|------|------|------|
| `@Schema` | API 文档描述 | `@Schema(description = "岗位名称")` |
| `@NotNull` | 非空校验 | `@NotNull(message = "状态不能为空")` |
| `@NotBlank` | 字符串非空非空白 | `@NotBlank(message = "名称不能为空")` |
| `@Size` | 长度限制 | `@Size(max = 30, message = "名称不能超过30个字符")` |
| `@Email` | 邮箱格式 | `@Email(message = "邮箱格式不正确")` |
| `@ExcelProperty` | Excel 导出列名 | `@ExcelProperty("岗位名称")` |
| `@DateTimeFormat` | 时间格式 | `@DateTimeFormat(pattern = FORMAT_YEAR_MONTH_DAY_HOUR_MINUTE_SECOND)` |
