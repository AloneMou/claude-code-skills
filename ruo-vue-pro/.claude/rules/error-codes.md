# 错误码规范

> 定义项目中错误码的格式、分配和使用规则。

## 错误码结构

### 全局错误码 (0-999)

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 未登录 |
| 403 | 无权限 |
| 500 | 系统异常 |
| 400 | 请求参数错误 |

### 业务错误码格式

格式: `1-XXX-YYY-ZZZ`

| 位 | 说明 | 示例 |
|----|------|------|
| 第1位 | 类型（1=业务错误） | `1` |
| 第2-4位 | 系统编号 | `001`=infra, `002`=system |
| 第5-7位 | 模块编号 | `001`-`999` |
| 第8-10位 | 具体错误 | `000`, `001`, `002`... |

### 模块系统编号

| 系统 | 编号 |
|------|------|
| infra | 001 |
| system | 002 |
| member | 003 |
| pay | 004 |
| bpm | 005 |
| mall | 006 |
| crm | 007 |
| ai | 008 |

### 代码定义

每个模块在 `enums/ErrorCodeConstants.java` 中定义：

```java
package cn.iocoder.yudao.module.system.enums;

import cn.iocoder.yudao.framework.common.exception.ErrorCode;

public interface ErrorCodeConstants {

    // ========== 岗位 1-002-001-000 ==========
    ErrorCode POST_NOT_FOUND = new ErrorCode(1_002_001_000, "岗位不存在");
    ErrorCode POST_NAME_DUPLICATE = new ErrorCode(1_002_001_001, "岗位名称已存在");
    ErrorCode POST_CODE_DUPLICATE = new ErrorCode(1_002_001_002, "岗位编码已存在");
}
```

### 错误码命名规则

1. 常量名用全大写下划线分隔: `XXX_NOT_FOUND`, `XXX_NAME_DUPLICATE`
2. 描述用中文，简明扼要: `"岗位不存在"`, `"岗位名称已存在"`
3. 按模块分组，用注释分隔: `// ========== 岗位 1-002-001-000 ==========`
4. 每个业务实体至少定义: `_NOT_FOUND`, `_NAME_DUPLICATE`（如有唯一编码则加 `_CODE_DUPLICATE`）

### 使用方式

```java
// 抛出业务异常
throw exception(POST_NOT_FOUND);
throw exception(POST_NAME_DUPLICATE);

// 带参数的错误消息
throw exception(POST_NOT_FOUND, id);  // 自动格式化消息
```

### 新增模块时

1. 在对应模块的 `ErrorCodeConstants.java` 中追加定义
2. 分配一个独立的 3 位模块编号（在系统编号下递增）
3. 错误码从 `000` 开始编号，每个业务实体分配 3-5 个错误码
