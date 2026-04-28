# 数据访问规范

> 定义 DO、Mapper 和数据库查询的开发标准。

## DO (Data Object) 规范

```java
@TableName("system_post")
@KeySequence("system_post_seq")
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class PostDO extends BaseDO {

    @TableId
    private Long id;
    private String postCode;
    private String name;
    private Integer sort;
    private Integer status;
    private String remark;
}
```

### BaseDO 自动字段

| 字段 | 类型 | 填充策略 | 说明 |
|------|------|----------|------|
| `creator` | `String` | INSERT | 创建者 ID |
| `createTime` | `LocalDateTime` | INSERT | 创建时间 |
| `updater` | `String` | INSERT_UPDATE | 更新者 ID |
| `updateTime` | `LocalDateTime` | INSERT_UPDATE | 更新时间 |
| `deleted` | `Boolean` | `@TableLogic` | 逻辑删除标记 |

### DO 规则

1. 必须继承 `BaseDO`
2. 必须加 `@TableName("table_name")` 和 `@KeySequence("table_name_seq")`
3. 禁止手动设置审计字段（`createTime`, `updateTime`, `creator`, `updater`）
4. 状态/类型字段用 `Integer`，JavaDoc 注明枚举引用
5. 数据库字段为下划线，Java 字段用驼峰，MyBatis-Plus 自动映射

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

### Mapper 规则

1. 必须继承 `BaseMapperX<T>`
2. 自定义查询用 `default` 方法，优先避免 XML
3. 条件查询用 `LambdaQueryWrapperX` 的 `*IfPresent` 方法（自动跳过 null 值）
4. 分页查询用 `selectPage(reqVO, wrapper)` 方法

### BaseMapperX 常用方法

| 方法 | 说明 |
|------|------|
| `selectById(id)` | 按 ID 查询 |
| `selectByIds(ids)` | 按 ID 列表批量查询 |
| `selectList(wrapper)` | 按条件查询列表 |
| `selectPage(reqVO, wrapper)` | 分页查询 |
| `insert(entity)` | 插入记录 |
| `insertBatch(list)` | 批量插入 |
| `updateById(entity)` | 按 ID 更新 |
| `update(entity, wrapper)` | 按条件批量更新 |
| `deleteById(id)` | 按 ID 删除（逻辑删除） |
| `deleteByIds(ids)` | 批量删除（逻辑删除） |
| `selectCount(wrapper)` | 按条件计数 |
| `selectOne(wrapper)` | 查询单条 |

### LambdaQueryWrapperX 条件方法

| 方法 | 说明 |
|------|------|
| `eqIfPresent(field, value)` | 等于（value 为 null 时跳过） |
| `likeIfPresent(field, value)` | 模糊匹配 |
| `inIfPresent(field, value)` | IN 查询 |
| `betweenIfPresent(field, value)` | BETWEEN 查询（如时间范围） |
| `geIfPresent(field, value)` | 大于等于 |
| `leIfPresent(field, value)` | 小于等于 |
| `orderByDesc(field)` | 按字段降序排序 |
| `orderByAsc(field)` | 按字段升序排序 |

## 多表联查

```java
default List<XxxRespVO> selectJoinList(XxxPageReqVO reqVO) {
    return selectJoinList(XxxRespVO.class,
            new MPJLambdaWrapperX<XxxDO>()
                    .selectAll(XxxDO.class)
                    .selectAs(OtherDO::getName, XxxRespVO::getOtherName)
                    .leftJoin(OtherDO.class, OtherDO::getId, XxxDO::getOtherId)
                    .eqIfPresent(XxxDO::getStatus, reqVO.getStatus()));
}
```

## Redis DAO 规范

```java
@Repository
public class XxxRedisDAO {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final String XXX_KEY_PREFIX = "xxx:";

    public XxxDO get(Long id) {
        String key = XXX_KEY_PREFIX + id;
        String json = stringRedisTemplate.opsForValue().get(key);
        return JsonUtils.parseObject(json, XxxDO.class);
    }

    public void set(Long id, XxxDO data, Duration timeout) {
        String key = XXX_KEY_PREFIX + id;
        String json = JsonUtils.toJsonString(data);
        stringRedisTemplate.opsForValue().set(key, json, timeout);
    }

    public void delete(Long id) {
        String key = XXX_KEY_PREFIX + id;
        stringRedisTemplate.delete(key);
    }
}
```
