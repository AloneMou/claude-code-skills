---
name: ruoyi-crud
description: 在芋道 RuoYi-Vue-Pro 项目中生成完整的 CRUD 业务模块（DO/Mapper/Service/Controller/VO/ErrorCode），遵循框架分层架构和命名约定
trigger: 当用户要求新建业务模块、创建增删改查、生成 CRUD 代码、或提到 "新建CRUD"、"创建XX模块"、"新增增删改查"、"scaffold a module"
---

# RuoYi CRUD Module Generator

在芋道 (Yudao) RuoYi-Vue-Pro 项目中生成完整的 CRUD 模块，覆盖所有分层文件。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取项目级命名约定、技术栈和禁止事项。

## TRIGGER

当用户满足以下任一条件时激活此技能：

- 要求为某个业务实体创建完整的增删改查
- 提供表结构或实体字段描述，要求生成对应代码
- 提到 "新建 XX 模块"、"生成 CRUD"、"scaffold"、"create a module for"
- 在已有模块下新增一个业务实体

## SKIP

以下情况不使用此技能：

- 用户只要求修改单个方法或字段，不涉及完整模块生成
- 纯数据库脚本或 SQL 迁移任务
- 前端页面或 Vue 组件生成（此为后端技能）

## Workflow

1. **确认模块信息** — 实体名称（英文 PascalCase）、中文名称、所属模块、子模块分组、字段列表
2. **按顺序生成文件**: DO → Mapper → Service 接口 → Service 实现 → VOs (SaveReqVO/PageReqVO/RespVO) → Controller → ErrorCode
3. **校验** — 检查命名约定、包路径、文件完整性

## File Templates

### 1. DO (Data Object)

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/dal/dataobject/{submodule}/{Name}DO.java`

```java
package cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule};

import cn.iocoder.yudao.framework.mybatis.core.dataobject.BaseDO;
import com.baomidou.mybatisplus.annotation.*;
import lombok.*;

/**
 * {业务中文名称}
 *
 * @author {author}
 */
@TableName("{table_name}")
@KeySequence("{table_name}_seq")
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class {Name}DO extends BaseDO {

    /**
     * 主键ID
     */
    @TableId
    private Long id;

    // 根据用户提供的字段列表生成字段，使用 Java 驼峰命名
}
```

### 2. Mapper

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/dal/mysql/{submodule}/{Name}Mapper.java`

```java
package cn.iocoder.yudao.module.{module}.dal.mysql.{submodule};

import cn.iocoder.yudao.framework.common.pojo.PageResult;
import cn.iocoder.yudao.framework.mybatis.core.mapper.BaseMapperX;
import cn.iocoder.yudao.framework.mybatis.core.query.LambdaQueryWrapperX;
import cn.iocoder.yudao.module.{module}.controller.admin.{submodule}.vo.{Name}PageReqVO;
import cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule}.{Name}DO;
import org.apache.ibatis.annotations.Mapper;

/**
 * {业务中文名称} Mapper
 *
 * @author {author}
 */
@Mapper
public interface {Name}Mapper extends BaseMapperX<{Name}DO> {

    default PageResult<{Name}DO> selectPage({Name}PageReqVO reqVO) {
        return selectPage(reqVO, new LambdaQueryWrapperX<{Name}DO>()
                .likeIfPresent({Name}DO::getName, reqVO.getName())
                .eqIfPresent({Name}DO::getStatus, reqVO.getStatus())
                .betweenIfPresent({Name}DO::getCreateTime, reqVO.getCreateTime())
                .orderByDesc({Name}DO::getId));
    }
}
```

### 3. Service 接口

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/service/{submodule}/{Name}Service.java`

必须包含的方法：
- `Long create{Name}(SaveReqVO)` — 返回新建 ID
- `void update{Name}(SaveReqVO)` — 更新
- `void delete{Name}(Long id)` — 删除
- `void delete{Name}List(List<Long> ids)` — 批量删除
- `{Name}DO get{Name}(Long id)` — 按 ID 查询
- `PageResult<{Name}DO> get{Name}Page(PageReqVO)` — 分页查询

### 4. Service 实现

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/service/{submodule}/{Name}ServiceImpl.java`

```java
package cn.iocoder.yudao.module.{module}.service.{submodule};

import cn.iocoder.yudao.framework.common.pojo.PageResult;
import cn.iocoder.yudao.module.{module}.controller.admin.{submodule}.vo.*;
import cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule}.{Name}DO;
import cn.iocoder.yudao.module.{module}.dal.mysql.{submodule}.{Name}Mapper;
import org.springframework.stereotype.Service;
import org.springframework.validation.annotation.Validated;

import jakarta.annotation.Resource;
import java.util.List;

import static cn.iocoder.yudao.framework.common.exception.util.ServiceExceptionUtil.exception;
import static cn.iocoder.yudao.module.{module}.enums.ErrorCodeConstants.*;

/**
 * {业务中文名称} Service 实现类
 *
 * @author {author}
 */
@Service
@Validated
public class {Name}ServiceImpl implements {Name}Service {

    @Resource
    private {Name}Mapper {name}Mapper;

    @Override
    public Long create{Name}(SaveReqVO createReqVO) {
        // 1. 校验
        validateUnique(null, createReqVO.getName());
        // 2. 转换并插入
        {Name}DO {name} = BeanUtils.toBean(createReqVO, {Name}DO.class);
        {name}Mapper.insert({name});
        return {name}.getId();
    }

    @Override
    public void update{Name}(SaveReqVO updateReqVO) {
        // 1. 校验存在
        validateExists(updateReqVO.getId());
        // 2. 校验唯一性
        validateUnique(updateReqVO.getId(), updateReqVO.getName());
        // 3. 更新
        {Name}DO updateObj = BeanUtils.toBean(updateReqVO, {Name}DO.class);
        {name}Mapper.updateById(updateObj);
    }

    @Override
    public void delete{Name}(Long id) {
        // 校验存在
        validateExists(id);
        {name}Mapper.deleteById(id);
    }

    @Override
    public void delete{Name}List(List<Long> ids) {
        {name}Mapper.deleteByIds(ids);
    }

    @Override
    public {Name}DO get{Name}(Long id) {
        return {name}Mapper.selectById(id);
    }

    @Override
    public PageResult<{Name}DO> get{Name}Page(PageReqVO pageReqVO) {
        return {name}Mapper.selectPage(pageReqVO);
    }

    private void validateExists(Long id) {
        if ({name}Mapper.selectById(id) == null) {
            throw exception({NAME_UPPER}_NOT_FOUND);
        }
    }

    private void validateUnique(Long id, String name) {
        {Name}DO existing = {name}Mapper.selectByName(name);
        if (existing != null && !existing.getId().equals(id)) {
            throw exception({NAME_UPPER}_NAME_DUPLICATE);
        }
    }
}
```

### 5. VOs

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/controller/admin/{submodule}/vo/`

**SaveReqVO** — 包含 `id` 字段（更新时非空）+ 所有可编辑字段，每个字段加 `@Schema(description = "...")` 和校验注解
**PageReqVO** — 继承 `PageParam`，包含查询条件字段
**RespVO** — 包含所有字段 + `createTime`，加 `@Schema` 和 `@ExcelProperty` 注解

### 6. Controller

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/controller/admin/{submodule}/{Name}Controller.java`

必须包含的接口：
| 方法 | 路径 | 权限 |
|------|------|------|
| POST | `/create` | `{module}:{resource}:create` |
| PUT | `/update` | `{module}:{resource}:update` |
| DELETE | `/delete?id=` | `{module}:{resource}:delete` |
| DELETE | `/delete-list?ids=` | `{module}:{resource}:delete` |
| GET | `/get?id=` | `{module}:{resource}:query` |
| GET | `/page?...` | `{module}:{resource}:query` |
| GET | `/export-excel` | `{module}:{resource}:export` |

### 7. ErrorCode

追加到: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/enums/ErrorCodeConstants.java`

```java
// {RESOURCE} 1-XXX-YYY-ZZZ
ErrorCode {NAME_UPPER}_NOT_FOUND = new ErrorCode(1_XXX_YYY_000, "{业务}不存在");
ErrorCode {NAME_UPPER}_NAME_DUPLICATE = new ErrorCode(1_XXX_YYY_001, "{业务}名称已存在");
```

## Field Type Mapping

| 数据库类型 | Java 类型 | 查询方式 |
|-----------|-----------|----------|
| VARCHAR | String | `likeIfPresent` |
| INT/INTEGER | Integer | `eqIfPresent` |
| BIGINT | Long | `eqIfPresent` |
| TINYINT | Integer | `eqIfPresent`（状态/类型枚举） |
| DATETIME | LocalDateTime | `betweenIfPresent` |
| DECIMAL | BigDecimal | `eqIfPresent` |
| TEXT | String | 通常不参与查询 |

## Rules

- 使用 `jakarta.*` 而非 `javax.*`（Spring Boot 3）
- 状态/类型字段用 `Integer`，JavaDoc 中注明枚举引用
- 简单转换用 `BeanUtils.toBean()`，复杂列表转换用 MapStruct
- 错误码使用 `1_XXX_YYY_ZZZ` 下划线格式
- 表名用 snake_case，Java 类名用 PascalCase
- `@KeySequence` 必须添加（Oracle/PG/Kingbase/DB2/H2 兼容）
- 每个 Controller 方法必须有 `@Operation`、`@PreAuthorize` 注解