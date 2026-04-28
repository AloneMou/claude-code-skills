---
name: ruoyi-api
description: Design cross-module APIs (Api interface + DTOs) for RuoYi-Vue-Pro modules following Yudao inter-module communication conventions
trigger: When the user needs to create cross-module API calls, expose module services to other modules, or design inter-module DTO interfaces
---

# RuoYi Cross-Module API Designer

Generates inter-module API interfaces for the 芋道 (Yudao) RuoYi-Vue-Pro framework.

## When to Trigger

- User wants module A to call module B's service
- "Expose X as API for other modules", "创建跨模块API"
- Need to define `ReqDTO` / `RespDTO` for inter-module communication

## Architecture

Cross-module APIs use **direct service calls** (not HTTP). Module A depends on Module B's API package.

```
Module A
  └── depends on ── yudao-module-system (API package only)
       └── cn.iocoder.yudao.module.system.api
            ├── user/
            │   ├── UserApi.java          # API interface
            │   └── dto/
            │       ├── UserRespDTO.java  # Response DTO
            │       └── UserReqDTO.java   # Request DTO (if needed)
            ├── dept/
            │   ├── DeptApi.java
            │   └── dto/
            │       └── DeptRespDTO.java
```

## Workflow

1. **Identify the API contract** — what data does the caller need?
2. **Create the API interface** in the provider module's `api/` package
3. **Create DTOs** — only expose fields the caller actually needs
4. **Implement the API** — `ApiImpl` calls the internal Service
5. **Register the dependency** — caller module's `pom.xml` adds provider API dependency

## API Interface

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/{Name}Api.java`

```java
package cn.iocoder.yudao.module.{module}.api.{submodule};

import cn.iocoder.yudao.module.{module}.api.{submodule}.dto.{Name}RespDTO;

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

    /**
     * 获得{业务名称}
     *
     * @param {paramName} {param description}
     * @return {业务名称}
     */
    {Name}RespDTO get{Name}ByXxx(String xxx);
}
```

## DTO Conventions

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/dto/`

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

    // Only expose fields needed by callers
}
```

**Rules**:
- DTOs are **flat POJOs** — no business logic, no inheritance from BaseDO
- Only expose fields that external modules need
- Use Java types (not database types) — `LocalDateTime` for datetime
- No validation annotations on DTOs (validation happens at the Service layer)

## API Implementation

Path: `yudao-module-{module}/src/main/java/cn/iocoder/yudao/module/{module}/api/{submodule}/{Name}ApiImpl.java`

```java
package cn.iocoder.yudao.module.{module}.api.{submodule};

import cn.iocoder.yudao.module.{module}.convert.{Name}Convert;
import cn.iocoder.yudao.module.{module}.dal.dataobject.{submodule}.{Name}DO;
import cn.iocoder.yudao.module.{module}.service.{submodule}.{Name}Service;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
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

## Convert (MapStruct)

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

## Dependency Registration

When Module A needs to call Module B's API, add to Module A's `pom.xml`:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-{module}</artifactId>
    <version>${revision}</version>
    <!-- Exclude the provider's transitive dependencies if not needed -->
    <exclusions>
        <exclusion>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## Important Rules

1. **API interfaces are contracts** — don't change them without considering all callers
2. **DTOs are isolated** — they should NOT reference DOs or VOs from any module
3. **No circular dependencies** — if A calls B, B must NOT call A's API
4. **Keep APIs minimal** — expose only what callers need, not the full service
5. **Use MapStruct** for DO → DTO conversion, not BeanUtils (performance)
6. **API methods should be idempotent** — callers may retry on timeout
