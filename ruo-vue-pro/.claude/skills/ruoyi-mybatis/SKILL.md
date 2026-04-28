---
name: ruoyi-mybatis
description: 芋道 RuoYi-Vue-Pro 项目 MyBatis-Plus 数据访问模式指南：LambdaQueryWrapperX 条件查询、BaseMapperX 操作、分页、多表联查、类型处理器
trigger: 当用户编写数据库查询、操作 Mapper 方法、编写复杂条件查询、多表联查、或提到 "写查询"、"数据库操作"、"Mapper"、"分页查询"
---

# RuoYi MyBatis-Plus Guide

芋道 (Yudao) RuoYi-Vue-Pro 项目的数据访问模式参考。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取项目级 Mapper/DO 规范。

## TRIGGER

- 用户需要编写数据库查询方法
- 需要条件分页、批量操作、多表联查
- 提到 "写查询"、"数据库操作"、"Mapper 方法"

## SKIP

- 纯 Redis 缓存操作
- 已有现成方法只需调用

## BaseMapperX 操作表

`BaseMapperX<T>` 继承 MyBatis-Plus-Join 的 `MPJBaseMapper<T>`，提供以下方法：

| 方法 | 说明 |
|------|------|
| `insert(T)` | 单条插入 |
| `insertBatch(Collection<T>)` | 批量插入 |
| `updateById(T)` | 按 ID 更新 |
| `updateBatch(Collection<T>)` | 批量更新 |
| `deleteById(Long)` | 按 ID 删除（逻辑删除） |
| `deleteByIds(Collection<Long>)` | 批量删除（逻辑删除） |
| `selectById(Long)` | 按 ID 查询 |
| `selectOne(Wrapper<T>)` | 单条查询 |
| `selectCount(Wrapper<T>)` | 条件计数 |
| `selectList(Wrapper<T>)` | 列表查询 |
| `selectPage(PageParam, Wrapper<T>)` | 分页查询 |

## LambdaQueryWrapperX — 条件方法

所有 `*IfPresent` 方法在值为 null/空时自动跳过：

```java
.eqIfPresent(Entity::getField, value)       // = value
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
.isNullIfPresent(Entity::getField, flag)    // IS NULL (flag=true)
.orderByDesc(Entity::getField)              // ORDER BY field DESC
.orderByAsc(Entity::getField)               // ORDER BY field ASC
```

## Common Query Patterns

### 分页查询

```java
default PageResult<XxxDO> selectPage(XxxPageReqVO reqVO) {
    return selectPage(reqVO, new LambdaQueryWrapperX<XxxDO>()
            .likeIfPresent(XxxDO::getName, reqVO.getName())
            .eqIfPresent(XxxDO::getStatus, reqVO.getStatus())
            .betweenIfPresent(XxxDO::getCreateTime, reqVO.getCreateTime())
            .orderByDesc(XxxDO::getId));
}
```

### 按 ID 列表批量查询

```java
default List<XxxDO> selectByIds(Collection<Long> ids) {
    return selectList(new LambdaQueryWrapperX<XxxDO>()
            .in(XxxDO::getId, ids));
}
```

### 唯一查询

```java
default XxxDO selectByName(String name) {
    return selectOne(XxxDO::getName, name);
}
```

### 条件计数

```java
default Long selectCountByStatus(Integer status) {
    return selectCount(XxxDO::getStatus, status);
}
```

### 按条件批量更新

```java
default int updateStatusByIds(Collection<Long> ids, Integer status) {
    return update(null, new LambdaUpdateWrapperX<XxxDO>()
            .in(XxxDO::getId, ids)
            .set(XxxDO::getStatus, status));
}
```

### 防重复插入

```java
default int insertIfAbsent(XxxDO entity) {
    Long count = selectCount(XxxDO::getCode, entity.getCode());
    if (count > 0) return 0;
    return insert(entity);
}
```

## 多表联查 (MPJLambdaWrapperX)

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

## 类型处理器

框架内置类型处理器（`yudao-spring-boot-starter-mybatis`）：

| Type Handler | 用途 |
|-------------|------|
| `IntegerListTypeHandler` | `List<Integer>` 存为 JSON 数组 |
| `LongListTypeHandler` | `List<Long>` 存为 JSON 数组 |
| `StringListTypeHandler` | `List<String>` 存为 JSON 数组 |
| `JsonSetTypeHandler` | `Set<String>` 存为 JSON 数组 |

DO 中使用：
```java
@TableField(typeHandler = IntegerListTypeHandler.class)
private List<Integer> categoryIds;
```

## 分页

MyBatis-Plus 拦截器自动处理分页。流程：

1. Controller 接收 `XxxPageReqVO extends PageParam`
2. 传入 Service → Mapper
3. Mapper 调用 `selectPage(reqVO, wrapper)` 自动分页
4. 返回 `PageResult<DO>`

**PageParam 默认值**: `pageNo = 1`, `pageSize = 10`, 最大 `pageSize = 200`

## Rules

1. **始终用 `LambdaQueryWrapperX`** — 不用原生 `QueryWrapper`
2. **使用 `*IfPresent` 方法** — 自动跳过 null 条件
3. **default 方法优先于 XML** — 简单查询不写 XML
4. **逻辑删除** — `deleteById` 设置 `deleted = true`，非物理删除
5. **批量操作** — 优先 `insertBatch`/`updateBatch`，不用循环
6. **联表查询** — 用 `MPJLambdaWrapperX`，不手写 SQL
7. **事务管理** — 修改多表的 Service 方法加 `@Transactional`
8. **不用 SELECT \*** — 明确指定字段，join 中用 `selectAll()` 除外