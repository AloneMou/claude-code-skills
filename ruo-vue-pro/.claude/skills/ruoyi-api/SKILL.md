---
name: ruoyi-api
description: 在芋道 RuoYi-Vue-Pro 项目中设计跨模块 API 接口（Api 接口 + DTOs + MapStruct 转换器 + 实现类），遵循框架模块间通信约定
trigger: 当用户需要创建跨模块调用、暴露模块服务给其他模块调用、设计模块间接口、或提到 "跨模块API"、"创建API接口"、"暴露服务"、"module API"
---

# RuoYi Cross-Module API Designer

在芋道 (Yudao) RuoYi-Vue-Pro 项目中生成跨模块 API 接口。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取项目级命名约定、技术栈和禁止事项。

## TRIGGER

当用户满足以下任一条件时激活此技能：

- 模块 A 需要调用模块 B 的服务
- 要求将某个服务暴露为跨模块 API
- 提到 "跨模块调用"、"创建 API"、"暴露服务"、"expose as API"

## SKIP

以下情况不使用此技能：

- 模块内部的 Controller → Service 调用（同一模块内）
- 纯 HTTP/REST 接口设计（应使用 Controller 规范）
- 仅修改已有 API 的单个字段

## Architecture

跨模块 API 使用**直接服务调用**（非 HTTP RPC）。调用方模块依赖被调用方模块的 `api/` 包。

```
Module A
  └── depends on ── yudao-module-system (API package only)
       └── cn.iocoder.yudao.module.system.api
            ├── user/
            │   ├── UserApi.java          # API 接口
            │   └── dto/
            │       ├── UserRespDTO.java  # 响应 DTO
            │       └── UserReqDTO.java   # 请求 DTO
            └── dept/
                ├── DeptApi.java
                └── dto/
                    └── DeptRespDTO.java
```

## Workflow

1. **明确 API 契约** — 调用方需要哪些数据？只暴露必要字段
2. **创建 API 接口** — 在提供方的 `api/{submodule}/` 包下
3. **创建 DTO** — 仅暴露调用方需要的字段，不引用 DO/VO
4. **创建 MapStruct 转换器** — DO → DTO 转换
5. **创建 API 实现** — ApiImpl 调用内部 Service + Convert 转换

## File Templates

### API 接口

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/{Name}Api.java`

```java
package cn.iocoder.yudao.module.{module}.api.{submodule};

import cn.iocoder.yudao.module.{module}.api.{submodule}.dto.{Name}RespDTO;

import java.util.Collection;
import java.util.List;

/**
 * {业务名称} API 接口
 *
 * @author {author}
 */
public interface {Name}Api {

    /**
     * 获得{业务名称}
     *
     * @param id 编号
     * @return {业务名称}
     */
    {Name}RespDTO get{Name}(Long id);

    /**
     * 批量获得{业务名称}列表
     *
     * @param ids 编号列表
     * @return {业务名称}列表
     */
    List<{Name}RespDTO> get{Name}List(Collection<Long> ids);
}
```

### DTO

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/dto/{Name}RespDTO.java`

```java
package cn.iocoder.yudao.module.{module}.api.{submodule}.dto;

import lombok.Data;

/**
 * {业务名称} Response DTO
 *
 * @author {author}
 */
@Data
public class {Name}RespDTO {

    /**
     * 编号
     */
    private Long id;

    /**
     * 名称
     */
    private String name;

    // 仅暴露调用方需要的字段
}
```

### Convert (MapStruct)

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/convert/{submodule}/{Name}Convert.java`

```java
package cn.iocoder.yudao.module.{module}.convert.{submodule};

import cn.iocoder.yudao.module.{module}.api.{submodule}.dto.{Name}RespDTO;
import cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule}.{Name}DO;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

import java.util.List;

/**
 * {业务名称} 转换器
 *
 * @author {author}
 */
@Mapper
public interface {Name}Convert {

    {Name}Convert INSTANCE = Mappers.getMapper({Name}Convert.class);

    {Name}RespDTO convert({Name}DO bean);

    List<{Name}RespDTO> convertList(List<{Name}DO> list);
}
```

### API 实现

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/{Name}ApiImpl.java`

```java
package cn.iocoder.yudao.module.{module}.api.{submodule};

import cn.iocoder.yudao.module.{module}.convert.{submodule}.{Name}Convert;
import cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule}.{Name}DO;
import cn.iocoder.yudao.module.{module}.service.{submodule}.{Name}Service;
import org.springframework.stereotype.Service;

import jakarta.annotation.Resource;
import java.util.Collection;
import java.util.List;

/**
 * {业务名称} API 实现类
 *
 * @author {author}
 */
@Service
public class {Name}ApiImpl implements {Name}Api {

    @Resource
    private {Name}Service {name}Service;

    @Override
    public {Name}RespDTO get{Name}(Long id) {
        {Name}DO {name} = {name}Service.get{Name}(id);
        return {Name}Convert.INSTANCE.convert({name});
    }

    @Override
    public List<{Name}RespDTO> get{Name}List(Collection<Long> ids) {
        List<{Name}DO> list = {name}Service.get{Name}List(ids);
        return {Name}Convert.INSTANCE.convertList(list);
    }
}
```

## Rules

1. **API 接口是契约** — 修改前需确认所有调用方
2. **DTO 独立** — 不得引用任何模块的 DO 或 VO
3. **无循环依赖** — 如果 A 调 B，B 不得调 A 的 API
4. **最小暴露** — 只暴露调用方需要的字段
5. **使用 MapStruct** 做 DO → DTO 转换（非 BeanUtils，性能更好）
6. **API 方法应幂等** — 调用方可能在超时后重试
7. **使用 `jakarta.annotation.Resource`** 而非 `javax.annotation.Resource`