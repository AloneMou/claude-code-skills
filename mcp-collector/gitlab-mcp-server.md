# GitLab Duo MCP Server 接口文档

> **文档来源**: [GitLab MCP Server](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server/) / [GitLab MCP Server Tools](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server_tools/)
> **MCP 端点**: GitLab SaaS 实例内置（自部署版为 `<your-gitlab-instance>/api/mcp`）
> **协议**: Streamable HTTP / SSE（取决于 GitLab 版本）

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

GitLab Duo MCP Server 是 GitLab 官方提供的 MCP 服务，内置于 GitLab SaaS 和自部署实例中。AI 助手可通过 MCP 协议直接与 GitLab 交互，实现 Issue 管理、Merge Request 操作、流水线控制、代码搜索等功能。

| 子系统          | 能力                                       |
| --------------- | ------------------------------------------ |
| Issue           | 获取、创建、更新、列表、搜索               |
| Merge Request   | 获取、创建、更新、列表、查看版本           |
| Project         | 获取项目信息、列出项目                     |
| Pipeline/CI-CD  | 获取流水线、列出流水线、管理流水线（创建/取消/重试/删除） |
| Code Search     | 语义代码搜索、文件内容搜索                 |
| Work Items      | 工作项管理                                 |

---

## 2. 认证与配置

### 2.1 MCP 客户端配置

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "http",
      "url": "https://gitlab.com/api/mcp"
    }
  }
}
```

**自部署实例：**

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "http",
      "url": "https://your-gitlab-instance.com/api/mcp"
    }
  }
}
```

### 2.2 认证方式

#### OAuth 2.0（推荐）

- GitLab MCP 支持 **OAuth 2.0 动态客户端注册**（Dynamic Client Registration）
- AI 工具可向 GitLab 实例自行注册
- 首次连接触发浏览器授权流程
- 注册的应用会出现在 GitLab 用户设置的 **Applications** 列表中

#### Personal Access Token

- 使用 GitLab Personal Access Token 进行认证
- Token 需包含对应项目/资源的访问权限（`read_api`, `write_api`, `api` scope）
- 在 MCP 客户端配置中通过 `env` 或 `headers` 传递 Token

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "http",
      "url": "https://gitlab.com/api/mcp",
      "headers": {
        "Authorization": "Bearer <your-personal-access-token>"
      }
    }
  }
}
```

### 2.3 版本要求

| 版本    | 说明                                     |
| ------- | ---------------------------------------- |
| 18.0+   | MCP Server 实验性引入                    |
| 18.6+   | 增加更多工具（如 `list_projects` 等）     |
| 18.10+  | 持续扩展工具集                           |

---

## 3. 工具列表

### 3.1 Issue 工具

---

#### `get_issue`

获取单个 Issue 的完整详情。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `issue_iid`  | string | ✅   | Issue 的内部 ID（非全局 ID）            |

---

#### `create_issue`

创建新的 Issue。

| 参数          | 类型   | 必填 | 说明                                  |
| ------------- | ------ | ---- | ------------------------------------- |
| `project_id`  | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `title`       | string | ✅   | Issue 标题                             |
| `description` | string | ❌   | Issue 描述（支持 Markdown）            |
| `labels`      | array  | ❌   | 标签列表，如 `["bug", "frontend"]`     |
| `assignee_ids`| array  | ❌   | 指派人用户 ID 数组                     |
| `milestone_id`| integer| ❌   | 关联的里程碑 ID                        |

---

#### `update_issue`

更新已有 Issue。

| 参数          | 类型   | 必填 | 说明                                  |
| ------------- | ------ | ---- | ------------------------------------- |
| `project_id`  | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `issue_iid`   | string | ✅   | Issue 的内部 ID                        |
| `title`       | string | ❌   | 新标题                                 |
| `description` | string | ❌   | 新描述                                 |
| `state_event` | string | ❌   | 状态变更：`"close"` / `"reopen"`       |
| `labels`      | array  | ❌   | 更新的标签列表                         |

---

#### `list_issues`

列出项目中的 Issue。

| 参数         | 类型    | 必填 | 说明                                  |
| ------------ | ------- | ---- | ------------------------------------- |
| `project_id` | string  | ✅   | 项目 ID 或 URL 编码的路径              |
| `state`      | string  | ❌   | 过滤状态：`"opened"` / `"closed"`      |
| `labels`     | array   | ❌   | 按标签过滤                             |
| `author_id`  | integer | ❌   | 按作者过滤                             |
| `assignee_id`| integer | ❌   | 按指派人过滤                           |
| `milestone`  | string  | ❌   | 按里程碑名称过滤                       |

---

#### `search_issues`

跨项目搜索 Issue。

| 参数      | 类型   | 必填 | 说明                              |
| --------- | ------ | ---- | --------------------------------- |
| `query`   | string | ✅   | 搜索关键词                        |
| `state`   | string | ❌   | 过滤状态                          |
| `labels`  | array  | ❌   | 按标签过滤                        |

---

### 3.2 Merge Request 工具

---

#### `get_merge_request`

获取单个 Merge Request 的完整详情，包括变更内容。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `merge_request_iid` | string | ✅ | MR 的内部 ID                     |

---

#### `create_merge_request`

创建新的 Merge Request。

| 参数             | 类型    | 必填 | 说明                                  |
| ---------------- | ------- | ---- | ------------------------------------- |
| `project_id`     | string  | ✅   | 项目 ID 或 URL 编码的路径              |
| `source_branch`  | string  | ✅   | 源分支名称                             |
| `target_branch`  | string  | ✅   | 目标分支名称                           |
| `title`          | string  | ✅   | MR 标题                                |
| `description`    | string  | ❌   | MR 描述（支持 Markdown）               |
| `target_project_id` | string | ❌ | 目标项目 ID（跨项目 MR）          |
| `assignee_ids`   | array   | ❌   | 审查指派人用户 ID 数组                 |
| `reviewer_ids`   | array   | ❌   | 审查人用户 ID 数组                     |
| `labels`         | array   | ❌   | 标签列表                               |
| `milestone_id`   | integer | ❌   | 关联的里程碑 ID                        |

---

#### `update_merge_request`

更新已有 Merge Request。

| 参数             | 类型   | 必填 | 说明                                  |
| ---------------- | ------ | ---- | ------------------------------------- |
| `project_id`     | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `merge_request_iid` | string | ✅ | MR 的内部 ID                     |
| `title`          | string | ❌   | 新标题                                 |
| `description`    | string | ❌   | 新描述                                 |
| `state_event`    | string | ❌   | `"merge"` / `"close"`                  |
| `target_branch`  | string | ❌   | 修改目标分支                           |
| `assignee_ids`   | array  | ❌   | 更新指派人                             |
| `reviewer_ids`   | array  | ❌   | 更新审查人                             |

---

#### `list_merge_requests`

列出项目中的 Merge Request。

| 参数             | 类型   | 必填 | 说明                                  |
| ---------------- | ------ | ---- | ------------------------------------- |
| `project_id`     | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `state`          | string | ❌   | `"opened"` / `"closed"` / `"merged"`   |
| `author_id`      | integer| ❌   | 按作者过滤                             |
| `target_branch`  | string | ❌   | 按目标分支过滤                         |
| `source_branch`  | string | ❌   | 按源分支过滤                           |

---

#### `list_merge_request_versions`

列出 Merge Request 的版本/修订历史。

| 参数             | 类型   | 必填 | 说明                                  |
| ---------------- | ------ | ---- | ------------------------------------- |
| `project_id`     | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `merge_request_iid` | string | ✅ | MR 的内部 ID                     |

---

### 3.3 Project 工具

---

#### `get_project`

获取项目详情。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |

---

#### `list_projects`

列出用户可访问的项目。

| 参数          | 类型   | 必填 | 说明                                  |
| ------------- | ------ | ---- | ------------------------------------- |
| `search`      | string | ❌   | 搜索项目名称                           |
| `membership`  | boolean| ❌   | 仅列出用户有成员资格的项目              |
| `visibility`  | string | ❌   | `"public"` / `"internal"` / `"private"`|

---

### 3.4 Pipeline / CI-CD 工具

---

#### `get_pipeline`

获取流水线详情。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `pipeline_id`| string | ✅   | 流水线 ID                              |

---

#### `list_pipelines`

列出项目中的流水线。

| 参数         | 类型    | 必填 | 说明                                  |
| ------------ | ------- | ---- | ------------------------------------- |
| `project_id` | string  | ✅   | 项目 ID 或 URL 编码的路径              |
| `ref`        | string  | ❌   | 按分支/标签过滤                        |
| `status`     | string  | ❌   | `"running"` / `"success"` / `"failed"` / `"pending"` / `"created"` |
| `source`     | string  | ❌   | 触发源：`"push"` / `"web"` / `"api"` / `"schedule"` |

---

#### `manage_pipeline`

管理流水线（创建、取消、重试、删除）。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `action`     | string | ✅   | `"create"` / `"cancel"` / `"retry"` / `"delete"` |
| `pipeline_id`| string | ❌   | 流水线 ID（cancel/retry/delete 时必填） |
| `ref`        | string | ❌   | 分支/标签名（create 时必填）            |

---

#### `list_pipeline_bridges`

列出流水线中的桥接任务。

| 参数          | 类型   | 必填 | 说明                                  |
| ------------- | ------ | ---- | ------------------------------------- |
| `project_id`  | string | ✅   | 项目 ID 或 URL 编码的路径              |
| `pipeline_id` | string | ✅   | 流水线 ID                              |

---

### 3.5 Code Search 工具

---

#### `semantic_code_search`

语义代码搜索（AI 驱动的跨仓库代码理解搜索）。

| 参数      | 类型   | 必填 | 说明                              |
| --------- | ------ | ---- | --------------------------------- |
| `project_id` | string | ✅ | 项目 ID 或 URL 编码的路径         |
| `query`   | string | ✅   | 自然语言搜索查询                    |

---

#### `blob_search`

文件内容/代码片段搜索。

| 参数      | 类型   | 必填 | 说明                              |
| --------- | ------ | ---- | --------------------------------- |
| `project_id` | string | ❌ | 项目 ID（不填则为 Group/Instance 级别搜索） |
| `query`   | string | ✅   | 搜索关键词（支持正则）              |
| `scope`   | string | ❌   | 搜索范围：`"group"` / `"instance"` |

---

### 3.6 Work Items 工具

工作项管理工具（GitLab 18.x 引入，具体工具名可能随版本变化）。

| 参数         | 类型   | 必填 | 说明                                  |
| ------------ | ------ | ---- | ------------------------------------- |
| `project_id` | string | ✅   | 项目 ID 或 URL 编码的路径              |

---

## 4. 使用示例

### 4.1 基础 MCP 配置

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "http",
      "url": "https://gitlab.com/api/mcp"
    }
  }
}
```

### 4.2 获取项目详情

```
工具: get_project
参数:
  project_id: "group-name/project-name"
```

### 4.3 搜索 Issue

```
工具: list_issues
参数:
  project_id: "group-name/project-name"
  state: "opened"
  labels: ["bug"]
```

### 4.4 创建 Issue

```
工具: create_issue
参数:
  project_id: "group-name/project-name"
  title: "登录页面移动端适配问题"
  description: "## 问题描述\n在移动浏览器上，登录按钮被底部导航遮挡"
  labels: ["bug", "mobile"]
```

### 4.5 创建 Merge Request

```
工具: create_merge_request
参数:
  project_id: "group-name/project-name"
  source_branch: "feature/add-login"
  target_branch: "main"
  title: "feat: 新增移动端登录页面"
  description: "## 变更内容\n- 新增响应式登录组件"
  labels: ["frontend"]
  reviewer_ids: [12345]
```

### 4.6 查看流水线

```
工具: list_pipelines
参数:
  project_id: "group-name/project-name"
  ref: "main"
  status: "failed"
```

### 4.7 管理流水线（重试失败的流水线）

```
工具: manage_pipeline
参数:
  project_id: "group-name/project-name"
  action: "retry"
  pipeline_id: "987654"
```

### 4.8 语义代码搜索

```
工具: semantic_code_search
参数:
  project_id: "group-name/project-name"
  query: "查找所有处理用户登录的函数"
```

### 4.9 获取 Merge Request 详情

```
工具: get_merge_request
参数:
  project_id: "group-name/project-name"
  merge_request_iid: "42"
```

---

## 5. 高级配置

### 5.1 工具名前缀

为避免与其他 MCP 服务器冲突，可通过 HTTP Header 设置工具名前缀：

```
X-Gitlab-Mcp-Server-Tool-Name-Prefix: gitlab_
```

配置后，`create_issue` 变为 `gitlab_create_issue`。

### 5.2 连接到 Claude Desktop

参考官方教程：[Connect Claude Desktop to the GitLab MCP server](https://docs.gitlab.com/tutorials/connect_claude_desktop_with_gitlab_mcp_server/)

### 5.3 Duo Agent Platform 集成

GitLab Duo Agent Platform 支持通过 MCP 连接外部 MCP 服务器，扩展 AI Agent 的能力。详见 [Extend GitLab Duo Agent Platform](https://about.gitlab.com/blog/extend-gitlab-duo-agent-platform-connect-any-tool-with-mcp/)。

### 5.4 AI Catalog MCP 服务器

GitLab 的 AI Catalog 中提供了预配置的 MCP 服务器，可直接在 GitLab 内使用。详见 [AI Catalog MCP Servers](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/ai_catalog_mcp_servers/)。

---

## 6. 故障排查

详见官方文档：[Troubleshooting the GitLab MCP server](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server_troubleshooting/)

常见问题：

| 问题                        | 原因                                    | 解决方案                              |
| --------------------------- | --------------------------------------- | ------------------------------------- |
| 连接失败                    | GitLab 版本过低（< 18.0）               | 升级 GitLab 或使用 SaaS 版本           |
| 工具调用返回 401/403        | Token 权限不足                          | 检查 Token 的 `api` scope             |
| 工具名称冲突                | 多个 MCP 服务器有同名工具                | 使用 `X-Gitlab-Mcp-Server-Tool-Name-Prefix` 前缀 |
| 语义搜索不返回结果          | 项目未启用语义搜索索引                  | 检查 GitLab 实例配置                  |
| OAuth 授权循环              | 浏览器 Cookie/缓存问题                   | 清除浏览器缓存后重试                  |

---

## 7. 相关文档

| 文档                          | 链接                                                                 |
| ----------------------------- | -------------------------------------------------------------------- |
| MCP Server 概览               | [docs.gitlab.com/.../mcp_server](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server/) |
| MCP Server 工具参考           | [docs.gitlab.com/.../mcp_server_tools](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server_tools/) |
| MCP 客户端配置                | [docs.gitlab.com/.../mcp_clients](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_clients/) |
| 连接 Claude Desktop           | [docs.gitlab.com/tutorials/...](https://docs.gitlab.com/tutorials/connect_claude_desktop_with_gitlab_mcp_server/) |
| AI Catalog MCP 服务器         | [docs.gitlab.com/.../ai_catalog](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/ai_catalog_mcp_servers/) |
| 故障排查                      | [docs.gitlab.com/.../troubleshooting](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server_troubleshooting/) |
| Duo Agent Platform 扩展       | [about.gitlab.com/blog/...](https://about.gitlab.com/blog/extend-gitlab-duo-agent-platform-connect-any-tool-with-mcp/) |
| 社区实现 (TypeScript)         | [github.com/Alosies/gitlab-mcp-server](https://github.com/Alosies/gitlab-mcp-server) |
| 社区实现 (Python)             | [github.com/wadew/gitlab-mcp](https://github.com/wadew/gitlab-mcp) |
| MCP Marketplace 条目          | [mcps.live/server/gitlab-mcp-server](https://mcps.live/server/gitlab-mcp-server-4803) |

---

> **注意**: GitLab MCP Server 仍在持续迭代中，工具列表和参数可能随版本变化。请以 [官方文档](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server_tools/) 为准。
