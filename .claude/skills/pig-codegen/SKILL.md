---
name: pig-codegen
description: "在 pig 项目中使用代码生成器生成 CRUD 代码。当用户要求生成增删改查代码、使用 pig-codegen 模块或编写代码模板时使用。"
---

# PIG 代码生成

使用 pig-codegen 模块生成标准 CRUD 代码。

## TRIGGER

- 用户要求生成增删改查代码
- 用户提供数据库表结构，要求生成对应代码
- 用户使用 pig-codegen 模块

## 生成的文件

根据一张数据库表生成：

| 文件 | 说明 |
|------|------|
| `Xxx.java` | 实体类（Entity） |
| `XxxMapper.java` | Mapper 接口 |
| `XxxMapper.xml` | MyBatis XML |
| `XxxService.java` | Service 接口 |
| `XxxServiceImpl.java` | Service 实现类 |
| `XxxController.java` | 控制器 |
| `XxxDTO.java` | 数据传输对象 |

## Entity 模板

```java
@Data
@TableName("table_name")
@Schema(description = "描述")
public class Xxx {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    @TableLogic
    private String delFlag;
}
```

## Service 模板

```java
public interface XxxService extends IService<Xxx> {
    void saveXxx(XxxDTO dto);
    void updateXxx(XxxDTO dto);
}
```

```java
@Service
@RequiredArgsConstructor
public class XxxServiceImpl extends ServiceImpl<XxxMapper, Xxx> implements XxxService {

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void saveXxx(XxxDTO dto) {
        Xxx entity = new Xxx();
        BeanUtils.copyProperties(dto, entity);
        save(entity);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void updateXxx(XxxDTO dto) {
        Xxx entity = new Xxx();
        BeanUtils.copyProperties(dto, entity);
        updateById(entity);
    }
}
```

## Controller 模板

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

    @GetMapping("/page")
    @Operation(summary = "分页查询")
    public R<IPage<Xxx>> page(Page page, Xxx query) { return R.ok(xxxService.page(page, Wrappers.lambdaQuery(query))); }

    @PostMapping
    @Operation(summary = "新增")
    public R<Void> save(@Valid @RequestBody XxxDTO dto) { xxxService.saveXxx(dto); return R.ok(); }

    @PutMapping
    @Operation(summary = "修改")
    public R<Void> update(@Valid @RequestBody XxxDTO dto) { xxxService.updateXxx(dto); return R.ok(); }

    @DeleteMapping("/{id}")
    @Operation(summary = "删除")
    public R<Void> removeById(@PathVariable Long id) { xxxService.removeById(id); return R.ok(); }
}
```

## 生成后处理

1. 执行 `mvn spring-javaformat:apply` 格式化代码
2. 根据业务需求补充逻辑
3. 编写单元测试