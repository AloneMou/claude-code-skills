# Apifox MCP Server 接口文档

> **仓库**: [apifox/apifox-mcp-server](https://github.com/apifox/apifox-mcp-server)
> **运行方式**: `npx apifox-mcp-server@latest`（本地启动，Streamable HTTP）
> **数据源**: Apifox 项目接口文档 / Swagger/OAS 文件

---

## 目录

1. [概览](#1-概览)
2. [认证与配置](#2-认证与配置)
3. [工具列表](#3-工具列表)
4. [使用示例](#4-使用示例)
5. [高级配置](#5-高级配置)
6. [故障排查](#6-故障排查)
7. [相关文档](#7-相关文档)

---

## 1. 概览

Apifox MCP Server 是一个本地运行的 MCP 服务，将 Apifox 项目内的接口文档作为数据源提供给支持 MCP 的 AI 编程 IDE（Cursor、VSCode + Cline 等）。AI 可通过 MCP 协议直接读取项目接口文档数据，用于生成代码、搜索接口、校验参数等场景。

| 子系统       | 能力                                       |
| ------------ | ------------------------------------------ |
| 接口文档读取 | 读取 Apifox 项目内所有接口文档数据          |
| Swagger/OAS  | 直接读取 Swagger/OpenAPI Specification 文件 |
| 本地缓存     | 接口文档数据缓存在本地电脑                  |

**典型使用场景：**

- 根据接口文档生成或修改代码
- 搜索接口文档内容
- 根据 API 文档为类/模型添加字段和注释
- 根据 API 文档生成特定接口的 MVC 代码

---

## 2. 认证与配置

### 2.1 前置条件

| 条件                | 说明                                          |
| ------------------- | --------------------------------------------- |
| Node.js             | 版本 >= 18，推荐最新 LTS                      |
| MCP 兼容 IDE        | Cursor、VSCode + Cline 插件等                  |
| Apifox Access Token | 在 Apifox 账号设置 → API 访问令牌中创建        |
| Apifox 项目 ID      | 在 Apifox 项目 → 左侧边栏"项目设置" → 基本设置 |

### 2.2 Apifox 项目模式（推荐）

获取 Access Token：
1. 打开 Apifox，鼠标悬停在页面右上角头像上
2. 点击 "账号设置 → API 访问令牌"
3. 创建新的 API 访问令牌

获取项目 ID：
1. 打开 Apifox 中对应的项目
2. 在左侧边栏点击"项目设置"
3. 在"基本设置"页面复制项目 ID

#### macOS / Linux 配置

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=<project-id>"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "<access-token>"
      }
    }
  }
}
```

#### Windows 配置

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=<project-id>"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "<access-token>"
      }
    }
  }
}
```

### 2.3 Swagger/OAS 模式

如果不使用 Apifox 项目，可直接读取 Swagger/OpenAPI 文件：

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--oas=https://petstore.swagger.io/v2/swagger.json"
      ]
    }
  }
}
```

也支持本地文件路径：

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--oas=~/data/petstore/swagger.json"
      ]
    }
  }
}
```

### 2.4 环境变量

| 变量名                | 必填 | 说明                                           |
| --------------------- | ---- | ---------------------------------------------- |
| `APIFOX_ACCESS_TOKEN` | ✅   | Apifox API 访问令牌                             |

### 2.5 启动参数

| 参数                      | 必填 | 说明                                              |
| ------------------------- | ---- | ------------------------------------------------- |
| `--project-id=<id>`       | ✅   | Apifox 项目 ID（Apifox 模式）                      |
| `--oas=<url-or-path>`     | ✅   | Swagger/OAS 文件 URL 或本地路径（OAS 模式）         |
| `--apifox-api-base-url=<url>` | ❌ | 私有化部署服务器的 API 地址（以 http/https 开头） |

> **注意**: `--project-id` 和 `--oas` 二选一使用。

---

## 3. 工具列表

Apifox MCP Server 提供的核心能力如下：

| 工具/能力              | 说明                                              | 输入参数                           |
| ---------------------- | ------------------------------------------------- | ---------------------------------- |
| 读取全部接口文档        | 读取 Apifox 项目或 OAS 文件中所有接口文档数据       | 无（启动时自动缓存）                |
| 按路径/标签搜索接口     | 根据接口路径、标签等搜索特定接口                    | 搜索关键词                          |
| 获取接口详情            | 获取指定接口的完整信息（请求/响应参数、数据结构等）  | 接口 ID 或路径                      |

> **说明**: 接口文档数据在 MCP Server 启动时自动读取并缓存在本地。如果 Apifox 内的数据有更新，需要告知 AI 刷新接口文档数据，否则 AI 读到的数据可能不是最新的。

---

## 4. 使用示例

### 4.1 基础 MCP 配置（Apifox 模式）

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=123456"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "your-access-token"
      }
    }
  }
}
```

### 4.2 Swagger/OAS 模式配置

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--oas=https://example.com/api/swagger.json"
      ]
    }
  }
}
```

### 4.3 私有化部署配置

```json
{
  "mcpServers": {
    "API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=123456",
        "--apifox-api-base-url=https://your-apifox-server.com"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "your-access-token"
      }
    }
  }
}
```

### 4.4 根据 API 文档生成代码

```
用户提示: "根据 API 文档，生成 /users 接口相关的所有 MVC 代码"
```

### 4.5 根据 API 文档为模型添加字段

```
用户提示: "根据 API 文档，在 Product DTO 里添加 API 文档新增的几个字段"
```

### 4.6 搜索接口

```
用户提示: "搜索所有包含 '用户登录' 的接口"
```

---

## 5. 高级配置

### 5.1 多项目配置

如果需要同时使用多个 Apifox 项目的接口文档，在配置文件中添加多个 MCP Server：

```json
{
  "mcpServers": {
    "订单 API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=111111"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "<access-token-1>"
      }
    },
    "用户 API 文档": {
      "command": "npx",
      "args": [
        "-y",
        "apifox-mcp-server@latest",
        "--project-id=222222"
      ],
      "env": {
        "APIFOX_ACCESS_TOKEN": "<access-token-2>"
      }
    }
  }
}
```

**建议**: MCP Server 名称使用 `"xxx API 文档"` 格式（包含"API文档"字样），AI 更容易正确识别其用途。不建议使用 `"Apifox"` 或 `"Apifox MCP"` 这种名称。

### 5.2 团队安全配置

如果团队习惯将 MCP 配置文件同步到代码仓库，建议删除配置文件中的 `"APIFOX_ACCESS_TOKEN": "<access-token>"`，改为每个成员在自己电脑上设置名为 `APIFOX_ACCESS_TOKEN` 的环境变量，以避免 Token 泄漏。

### 5.3 数据刷新

接口文档数据在 MCP Server 启动时缓存到本地。当 Apifox 中的接口文档有更新时，需要告知 AI 刷新接口文档数据，否则 AI 读到的数据可能不是最新的。

---

## 6. 故障排查

| 问题                        | 原因                                    | 解决方案                              |
| --------------------------- | --------------------------------------- | ------------------------------------- |
| npx 命令执行失败             | 未安装 Node.js 或版本 < 18              | 安装 Node.js >= 18（推荐 LTS）        |
| 无法获取接口文档数据         | Access Token 无效或过期                 | 重新生成 API 访问令牌                  |
| 无法连接 Apifox 服务         | 网络问题或私有化部署地址配置错误         | 检查网络连接和 `--apifox-api-base-url` |
| Windows 下配置不生效         | Windows 需要使用 cmd 包装 npx 命令      | 使用 Windows 专用配置（command: "cmd"）|
| 读取到旧的接口文档数据       | 本地缓存未更新                          | 告知 AI 刷新接口文档数据               |
| 私有化部署无法访问 npm        | 无法访问 www.npm.com                    | 确保网络可访问 npm  registry           |

---

## 7. 相关文档

| 文档                          | 链接                                                                 |
| ----------------------------- | -------------------------------------------------------------------- |
| GitHub 仓库                   | [github.com/apifox/apifox-mcp-server](https://github.com/apifox/apifox-mcp-server) |
| Apifox API 访问令牌帮助文档   | Apifox 账号设置 → API 访问令牌                                       |
| 私有化部署指南                | 添加 `--apifox-api-base-url` 参数，确保可访问 `www.npm.com`          |

---

> **注意**: Apifox MCP Server 目前处于内测阶段，功能和工具接口可能发生变化。建议关注 [GitHub 仓库](https://github.com/apifox/apifox-mcp-server) 获取最新更新。
