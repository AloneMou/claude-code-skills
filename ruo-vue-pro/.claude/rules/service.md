# Service 开发规范

> 定义 Service 层的职责边界、实现模式和代码规范。

## 分层结构

必须有 Interface + Impl 两层：

```java
// 接口
public interface XxxService {
    Long createXxx(XxxSaveReqVO createReqVO);
    void updateXxx(XxxSaveReqVO updateReqVO);
    void deleteXxx(Long id);
    void deleteXxxList(List<Long> ids);
    XxxDO getXxx(Long id);
    PageResult<XxxDO> getXxxPage(XxxPageReqVO pageReqVO);
}

// 实现
@Service
@Validated
public class XxxServiceImpl implements XxxService {
    @Resource
    private XxxMapper xxxMapper;

    // 实现方法...
}
```

## 方法实现模式

### 创建方法

```java
@Override
public Long createXxx(XxxSaveReqVO createReqVO) {
    // 1. 校验（唯一性、关联存在性等）
    validateXxxForCreateOrUpdate(null, createReqVO.getName());
    // 2. 转换
    XxxDO xxx = BeanUtils.toBean(createReqVO, XxxDO.class);
    // 3. 插入
    xxxMapper.insert(xxx);
    return xxx.getId();
}
```

### 更新方法

```java
@Override
public void updateXxx(XxxSaveReqVO updateReqVO) {
    // 1. 校验存在
    validateXxxExists(updateReqVO.getId());
    // 2. 校验唯一性
    validateXxxForCreateOrUpdate(updateReqVO.getId(), updateReqVO.getName());
    // 3. 转换并更新
    XxxDO updateObj = BeanUtils.toBean(updateReqVO, XxxDO.class);
    xxxMapper.updateById(updateObj);
}
```

### 删除方法

```java
@Override
public void deleteXxx(Long id) {
    validateXxxExists(id);
    xxxMapper.deleteById(id);
}

@Override
public void deleteXxxList(List<Long> ids) {
    xxxMapper.deleteByIds(ids);
}
```

### 查询方法

```java
@Override
public XxxDO getXxx(Long id) {
    return xxxMapper.selectById(id);
}

@Override
public PageResult<XxxDO> getXxxPage(XxxPageReqVO pageReqVO) {
    return xxxMapper.selectPage(pageReqVO);
}
```

## 校验方法规范

- 私有方法，命名 `validateXxxXxx`
- 校验不通过时抛出 `ServiceException`，使用 `throw exception(ERROR_CODE)` 语法
- 校验逻辑包括：存在性校验、唯一性校验、状态合法性校验等

```java
private void validateXxxForCreateOrUpdate(Long id, String name) {
    XxxDO existing = xxxMapper.selectByName(name);
    if (existing != null && !existing.getId().equals(id)) {
        throw exception(XXX_NAME_DUPLICATE);
    }
}

private void validateXxxExists(Long id) {
    if (xxxMapper.selectById(id) == null) {
        throw exception(XXX_NOT_FOUND);
    }
}
```

## Service 禁止行为

1. **禁止** 直接操作 `HttpServletRequest`/`HttpServletResponse`
2. **禁止** 在 Service 方法中直接拼装 SQL — 使用 Mapper 的条件构造器
3. **禁止** 返回 VO 给 Controller — 分页返回 `PageResult<DO>`，单个返回 `DO`
4. **禁止** 绕过校验直接操作数据库
5. **禁止** 在 Service 中直接调用其他模块的 Mapper — 必须通过跨模块 API

## 转换规则

- 简单对象转换: `BeanUtils.toBean(source, TargetClass.class)`
- 列表转换（复杂场景）: 使用 MapStruct `XxxConvert.INSTANCE.convertList(list)`
- DO → VO 转换: Controller 层负责，Service 不处理 VO