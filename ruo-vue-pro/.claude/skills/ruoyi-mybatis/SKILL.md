---
name: ruoyi-mybatis
description: Guide MyBatis-Plus data access patterns for RuoYi-Vue-Pro including LambdaQueryWrapperX, BaseMapperX, pagination, joins, and type handlers
trigger: When the user works with database queries, MyBatis-Plus, Mapper methods, complex conditions, multi-table joins, or says "写查询", "数据库操作", "Mapper"
---

# RuoYi MyBatis-Plus Guide

Data access patterns for the 芋道 (Yudao) RuoYi-Vue-Pro framework.

## BaseMapperX Methods

`BaseMapperX<T>` extends MyBatis-Plus-Join's `MPJBaseMapper<T>` and provides:

### Single Entity Operations

| Method | Description |
|--------|-------------|
| `insert(T)` | Insert single record |
| `insertBatch(Collection<T>)` | Batch insert |
| `insertBatch(Collection<T>, int batchSize)` | Batch insert with custom size |
| `updateById(T)` | Update by ID |
| `update(T, Wrapper<T>)` | Update by condition |
| `updateBatch(Collection<T>)` | Batch update |
| `updateBatch(Collection<T>, int batchSize)` | Batch update with custom size |
| `deleteById(Long)` | Delete by ID (logical) |
| `deleteByIds(Collection<Long>)` | Batch delete (logical) |
| `delete(Wrapper<T>)` | Delete by condition |
| `selectById(Long)` | Get by ID |
| `selectBatchIds(Collection<Long>)` | Batch get by IDs |
| `selectOne(Wrapper<T>)` | Get single record |
| `selectCount(Wrapper<T>)` | Count by condition |
| `selectList(Wrapper<T>)` | List by condition |
| `selectPage(PageParam, Wrapper<T>)` | Paginated query |

### LambdaQueryWrapperX — Condition Methods

All `*IfPresent` methods skip the condition when the value is null/empty:

```java
.eqIfPresent(Entity::getField, value)       // = value (skip if null)
.neIfPresent(Entity::getField, value)       // != value
.likeIfPresent(Entity::getField, value)     // LIKE %value%
.likeLeftIfPresent(Entity::getField, value) // LIKE %value
.likeRightIfPresent(Entity::getField, value)// LIKE value%
.inIfPresent(Entity::getField, collection)  // IN (values)
.betweenIfPresent(Entity::getField, array)  // BETWEEN start AND end
.geIfPresent(Entity::getField, value)       // >= value
.leIfPresent(Entity::getField, value)       // <= value
.gtIfPresent(Entity::getField, value)       // > value
.ltIfPresent(Entity::getField, value)       // < value
.isNullIfPresent(Entity::getField, flag)    // IS NULL (when flag is true)
.orderByDesc(Entity::getField)              // ORDER BY field DESC
.orderByAsc(Entity::getField)               // ORDER BY field ASC
```

## Common Query Patterns

### Paginated List Query

```java
default PageResult<XxxDO> selectPage(XxxPageReqVO reqVO) {
    return selectPage(reqVO, new LambdaQueryWrapperX<XxxDO>()
            .likeIfPresent(XxxDO::getName, reqVO.getName())
            .eqIfPresent(XxxDO::getStatus, reqVO.getStatus())
            .betweenIfPresent(XxxDO::getCreateTime, reqVO.getCreateTime())
            .orderByDesc(XxxDO::getId));
}
```

### Batch Query by IDs

```java
default List<XxxDO> selectByIds(Collection<Long> ids) {
    return selectList(new LambdaQueryWrapperX<XxxDO>()
            .in(XxxDO::getId, ids));
}
```

### Unique Query

```java
default XxxDO selectByName(String name) {
    return selectOne(XxxDO::getName, name);
}
```

### Count Query

```java
default Long selectCountByStatus(Integer status) {
    return selectCount(XxxDO::getStatus, status);
}
```

### Batch Update by Condition

```java
default int updateStatusByIds(Collection<Long> ids, Integer status) {
    return update(null, new LambdaUpdateWrapperX<XxxDO>()
            .in(XxxDO::getId, ids)
            .set(XxxDO::getStatus, status));
}
```

### Conditional Insert (avoid duplicates)

```java
default int insertIfAbsent(XxxDO entity) {
    Long count = selectCount(XxxDO::getCode, entity.getCode());
    if (count > 0) {
        return 0;
    }
    return insert(entity);
}
```

## Multi-Table Joins (MPJLambdaWrapperX)

For queries involving multiple tables:

```java
default PageResult<XxxRespVO> selectPageWithJoin(XxxPageReqVO reqVO) {
    MPJLambdaWrapperX<XxxDO> wrapper = new MPJLambdaWrapperX<XxxDO>()
            .selectAll(XxxDO.class)
            .selectAs(OtherDO::getName, XxxRespVO::getOtherName)
            .leftJoin(OtherDO.class, OtherDO::getId, XxxDO::getOtherId)
            .eqIfPresent(XxxDO::getStatus, reqVO.getStatus());

    return selectJoinPage(reqVO, XxxRespVO.class, wrapper);
}
```

## Type Handlers

The framework provides built-in type handlers in `yudao-spring-boot-starter-mybatis`:

| Type Handler | Use Case |
|-------------|----------|
| `IntegerListTypeHandler` | `List<Integer>` stored as JSON array |
| `LongListTypeHandler` | `List<Long>` stored as JSON array |
| `StringListTypeHandler` | `List<String>` stored as JSON array |
| `JsonSetTypeHandler` | `Set<String>` stored as JSON array |

Usage in DO:
```java
@TableField(typeHandler = IntegerListTypeHandler.class)
private List<Integer> categoryIds;
```

## Pagination Setup

Pagination is handled automatically by MyBatis-Plus interceptor. The flow:

1. Controller receives `XxxPageReqVO extends PageParam`
2. Passes to Service/Mapper
3. Mapper calls `selectPage(reqVO, wrapper)` — pagination is auto-applied
4. Returns `PageResult<XxxDO>`

**PageParam defaults**: `pageNo = 1`, `pageSize = 10`, max `pageSize = 200`

## Important Rules

1. **Always use `LambdaQueryWrapperX`** — never raw `QueryWrapper`
2. **Use `*IfPresent` methods** — automatically skip null conditions
3. **Default methods over XML** — simple queries don't need XML files
4. **Logical delete** — `deleteById` sets `deleted = true`, doesn't remove row
5. **Batch operations** — prefer `insertBatch`/`updateBatch` over loops
6. **Join queries** — use `MPJLambdaWrapperX`, not handwritten SQL
7. **Transaction management** — add `@Transactional` to Service methods that modify multiple tables
8. **No SELECT \*** — always specify fields, use `selectAll()` only in join wrappers
