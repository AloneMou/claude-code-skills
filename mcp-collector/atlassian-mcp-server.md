# Atlassian Rovo MCP Server 接口文档

> **仓库**: [atlassian/atlassian-mcp-server](https://github.com/atlassian/atlassian-mcp-server)
> **服务端点**: `https://mcp.atlassian.com/v1/mcp`
> **协议**: Streamable HTTP (MCP 2024-11-05)

---

## 目录

1. [概览](#1-概览)
2. [认证与配置](#2-认证与配置)
3. [工具列表](#3-工具列表)
4. [使用示例](#4-使用示例)
5. [预置技能（工作流）](#5-预置技能工作流)
6. [安全说明](#6-安全说明)

---

## 1. 概览

Atlassian Rovo MCP Server 是一个云端桥接服务，将 Atlassian Cloud 站点与外部 MCP 兼容客户端（Claude、ChatGPT、GitHub Copilot CLI、VS Code、Cursor 等）连接起来，提供对 **Jira**、**Confluence** 和 **Compass** 数据的实时访问，并支持通过自然语言创建/更新 Issue 和页面。

| 子系统    | 能力                             |
| --------- | -------------------------------- |
| Jira      | 搜索、创建、获取、评论 Issue      |
| Confluence| 获取、创建、更新页面，搜索空间    |
| Compass   | 服务组件管理、依赖查询            |
| 跨系统搜索| 同时搜索 Jira + Confluence       |

---

## 2. 认证与配置

### 2.1 客户端配置

在 MCP 客户端配置中添加：

```json
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v1/mcp"
    }
  }
}
```

### 2.2 OAuth 2.1（3LO）—— 推荐方式

- 首次使用时触发浏览器 OAuth 2.1 授权流程
- 首次安装需要用户对所请求的所有 Atlassian 应用（Jira、Confluence、Compass）具有访问权限
- 后续用户只需拥有其中一个应用的访问权限即可完成授权
- 授权后在 **Connected apps** 中注册

### 2.3 API Token（无头模式）

- 需要管理员启用 Rovo MCP Server 的 API Token 认证
- 需要获取 **Rovo MCP 范围的 API Token**
- 配置指南: [Atlassian 官方文档](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/configuring-authentication-via-api-token/)

### 2.4 Cloud ID 获取

每个工具都需要 `cloudId` 参数，可通过以下两种方式获取：

1. 调用 `getAccessibleAtlassianResources` 工具获取
2. 从 Atlassian 站点 URL 中提取：`https://<your-site>.atlassian.net` → cloudId 为对应标识符

---

## 3. 工具列表

### 3.1 Jira 工具

---

#### `searchJiraIssuesUsingJql`

使用 JQL 查询语言搜索 Jira Issue。

| 参数            | 类型     | 必填 | 说明                                           |
| --------------- | -------- | ---- | ---------------------------------------------- |
| `cloudId`       | string   | ✅   | Atlassian Cloud 站点标识符                      |
| `jql`           | string   | ✅   | JQL 查询字符串                                  |
| `fields`        | string[] | ❌   | 返回字段，如 `["summary","status","assignee"]` |
| `maxResults`    | number   | ❌   | 最大结果数（建议 10-100）                       |
| `nextPageToken` | string   | ❌   | 分页令牌                                        |

**常用 JQL 示例：**

| 场景                    | JQL                                      |
| ----------------------- | ---------------------------------------- |
| 查询未解决的 Bug        | `project = PROJ AND type = Bug AND status != Done` |
| 分配给当前用户的任务    | `assignee = currentUser() AND status != Done` |
| 最近创建的 Issue        | `project = PROJ AND created >= -7d ORDER BY created DESC` |
| 高优先级未完成任务      | `priority = High AND status != Done`     |

---

#### `getJiraIssue`

获取单个 Jira Issue 的完整详情。

| 参数            | 类型   | 必填 | 说明                              |
| --------------- | ------ | ---- | --------------------------------- |
| `cloudId`       | string | ✅   | Atlassian Cloud 站点标识符         |
| `issueIdOrKey`  | string | ✅   | Issue 编号，如 `"PROJ-123"`        |

---

#### `createJiraIssue`

创建新的 Jira Issue。

| 参数                  | 类型   | 必填 | 说明                                          |
| --------------------- | ------ | ---- | --------------------------------------------- |
| `cloudId`             | string | ✅   | Atlassian Cloud 站点标识符                     |
| `projectKey`          | string | ✅   | 项目 Key，如 `"PROJ"`                          |
| `issueTypeName`       | string | ✅   | Issue 类型：`"Task"` / `"Story"` / `"Bug"` / `"Epic"` |
| `summary`             | string | ✅   | 标题                                           |
| `description`         | string | ❌   | Markdown 格式的描述                            |
| `assignee_account_id` | string | ❌   | 指派人 Jira Account ID                         |
| `parent`              | string | ❌   | 父 Issue Key（关联 Epic 时使用）                |
| `additional_fields`   | object | ❌   | 自定义字段，如 `{"priority": {"name": "High"}}` |

---

#### `addCommentToJiraIssue`

为已有 Jira Issue 添加评论。

| 参数            | 类型   | 必填 | 说明                              |
| --------------- | ------ | ---- | --------------------------------- |
| `cloudId`       | string | ✅   | Atlassian Cloud 站点标识符         |
| `issueIdOrKey`  | string | ✅   | Issue 编号                        |
| `commentBody`   | string | ✅   | 评论内容（Markdown 格式）          |

---

#### `lookupJiraAccountId`

根据姓名或邮箱查找 Jira 用户的 Account ID。

| 参数            | 类型   | 必填 | 说明                                      |
| --------------- | ------ | ---- | ----------------------------------------- |
| `cloudId`       | string | ✅   | Atlassian Cloud 站点标识符                 |
| `searchString`  | string | ✅   | 全名、姓名或邮箱地址                       |

---

#### `getVisibleJiraProjects`

获取用户可访问的 Jira 项目列表。

| 参数      | 类型   | 必填 | 说明                                    |
| --------- | ------ | ---- | --------------------------------------- |
| `cloudId` | string | ✅   | Atlassian Cloud 站点标识符               |
| `action`  | string | ❌   | 过滤操作，如 `"create"`（可创建 Issue）  |

---

#### `getJiraProjectIssueTypesMetadata`

获取项目可用的 Issue 类型元数据。

| 参数             | 类型   | 必填 | 说明              |
| ---------------- | ------ | ---- | ----------------- |
| `cloudId`        | string | ✅   | Cloud 站点标识符   |
| `projectIdOrKey` | string | ✅   | 项目 Key 或 ID    |

---

#### `getJiraIssueTypeMetaWithFields`

获取特定 Issue 类型的必填/可用字段。

| 参数             | 类型   | 必填 | 说明              |
| ---------------- | ------ | ---- | ----------------- |
| `cloudId`        | string | ✅   | Cloud 站点标识符   |
| `projectIdOrKey` | string | ✅   | 项目 Key 或 ID    |
| `issueTypeId`    | string | ✅   | Issue 类型 ID     |

---

### 3.2 Confluence 工具

---

#### `getConfluencePage`

获取 Confluence 页面内容。

| 参数             | 类型   | 必填 | 说明                                  |
| ---------------- | ------ | ---- | ------------------------------------- |
| `cloudId`        | string | ✅   | Cloud 站点标识符                       |
| `pageId`         | string | ✅   | 页面 ID（数字）                        |
| `contentFormat`  | string | ❌   | 输出格式，设为 `"markdown"` 获取 Markdown |

---

#### `createConfluencePage`

创建新的 Confluence 页面。

| 参数             | 类型   | 必填 | 说明                                  |
| ---------------- | ------ | ---- | ------------------------------------- |
| `cloudId`        | string | ✅   | Cloud 站点标识符                       |
| `spaceId`        | string | ✅   | 空间 ID（数字）                        |
| `title`          | string | ✅   | 页面标题                               |
| `body`           | string | ✅   | 页面内容（Markdown 格式）              |
| `contentFormat`  | string | ❌   | 设为 `"markdown"`                      |
| `parentId`       | string | ❌   | 父页面 ID（用于层级嵌套）              |

---

#### `updateConfluencePage`

更新已有 Confluence 页面。

| 参数              | 类型   | 必填 | 说明                                  |
| ----------------- | ------ | ---- | ------------------------------------- |
| `cloudId`         | string | ✅   | Cloud 站点标识符                       |
| `pageId`          | string | ✅   | 页面 ID                                |
| `body`            | string | ✅   | 更新后的内容（Markdown 格式）          |
| `contentFormat`   | string | ❌   | 设为 `"markdown"`                      |
| `versionMessage`  | string | ❌   | 版本提交信息                           |

---

#### `getConfluenceSpaces`

获取可用的 Confluence 空间列表。

| 参数      | 类型   | 必填 | 说明                  |
| --------- | ------ | ---- | --------------------- |
| `cloudId` | string | ✅   | Cloud 站点标识符       |

---

### 3.3 跨系统 / Compass 工具

---

#### `search`

跨 Jira 和 Confluence 同时搜索。

| 参数      | 类型   | 必填 | 说明                                      |
| --------- | ------ | ---- | ----------------------------------------- |
| `cloudId` | string | ✅   | Cloud 站点标识符                           |
| `query`   | string | ✅   | 搜索关键词，支持 CQL 过滤，如 `"type=page AND title~'搜索词'"` |

---

#### `searchConfluenceUsingCql`

使用 CQL 进行 Confluence 精准搜索。

| 参数      | 类型   | 必填 | 说明                                      |
| --------- | ------ | ---- | ----------------------------------------- |
| `cloudId` | string | ✅   | Cloud 站点标识符                           |
| `cql`     | string | ✅   | CQL 查询字符串，如 `"text ~ '搜索词' OR title ~ '搜索词'"` |

---

#### `getAccessibleAtlassianResources`

获取用户可访问的 Atlassian Cloud 资源/站点列表。

| 参数 | 说明 |
| ---- | ---- |
| 无   |      |

返回 cloudId 和站点信息。

---

### 3.4 Compass 功能（说明级）

README 中提及 Compass 支持以下能力，具体工具名称未公开：

- 基于仓库创建服务组件
- 从 CSV/JSON 批量导入组件和自定义字段
- 查询服务依赖关系（如 "什么服务依赖 `api-gateway`？"）

---

## 4. 使用示例

### 4.1 基础 MCP 配置

```json
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v1/mcp"
    }
  }
}
```

### 4.2 搜索 Jira Issue

```
工具: searchJiraIssuesUsingJql
参数:
  cloudId: "your-cloud-id"
  jql: "project = PROJ AND status != Done AND assignee = currentUser()"
  fields: ["summary", "status", "priority", "created"]
  maxResults: 10
```

### 4.3 创建 Jira Bug

```
工具: createJiraIssue
参数:
  cloudId: "your-cloud-id"
  projectKey: "PROJ"
  issueTypeName: "Bug"
  summary: "登录页面在移动端显示异常"
  description: "## 复现步骤\n1. 打开登录页\n2. 缩小浏览器窗口至手机尺寸\n3. 登录按钮被遮挡\n\n## 期望行为: 布局自适应"
  additional_fields:
    priority: { name: "High" }
```

### 4.4 获取 Confluence 页面

```
工具: getConfluencePage
参数:
  cloudId: "your-cloud-id"
  pageId: "123456789"
  contentFormat: "markdown"
```

### 4.5 创建 Confluence 页面

```
工具: createConfluencePage
参数:
  cloudId: "your-cloud-id"
  spaceId: "987654"
  title: "Q2 项目进度报告"
  body: "# Q2 项目进度报告\n\n## 概述\n本季度完成...\n"
  contentFormat: "markdown"
  parentId: "111222333"
```

### 4.6 跨系统搜索

```
工具: search
参数:
  cloudId: "your-cloud-id"
  query: "type=page AND title~'API 设计规范'"
```

### 4.7 为 Issue 添加评论

```
工具: addCommentToJiraIssue
参数:
  cloudId: "your-cloud-id"
  issueIdOrKey: "PROJ-456"
  commentBody: "已复现此问题，确认为前端样式冲突导致，预计明日修复。"
```

### 4.8 获取站点资源（首次使用）

```
工具: getAccessibleAtlassianResources
参数: （无）
```

首次使用时建议先调用此工具获取 `cloudId`。

---

## 5. 预置技能（工作流）

以下 5 个预置技能定义了多工具协作流程：

| 技能                            | 用途                                   | 涉及工具                                           |
| ------------------------------- | -------------------------------------- | -------------------------------------------------- |
| `capture-tasks-from-meeting-notes`  | 解析会议笔记并创建 Jira 任务           | getConfluencePage → lookupJiraAccountId → createJiraIssue |
| `generate-status-report`            | 查询 Jira 进度并发布 Confluence 报告   | searchJiraIssuesUsingJql → createConfluencePage    |
| `search-company-knowledge`          | 跨 Jira + Confluence 搜索知识          | search → searchConfluenceUsingCql → getConfluencePage |
| `spec-to-backlog`                   | 将 Confluence 需求转为 Jira 待办列表   | getConfluencePage → createJiraIssue                |
| `triage-issue`                      | Bug 分派：搜索重复项后创建或评论       | searchJiraIssuesUsingJql → addCommentToJiraIssue   |

---

## 6. 安全说明

- 所有流量通过 HTTPS (TLS 1.2+) 加密
- 数据访问遵循用户已有的 Jira/Confluence 权限
- 支持 IP 白名单
- 关键操作记录在组织审计日志中
- 注意防范 LLM Prompt Injection 攻击，谨慎启用 MCP 客户端/服务端

---

## 附录：推荐配置

在 `AGENTS.md` 中设置默认值以减少发现调用：

```
- 默认 Jira projectKey = YOURPROJ
- 默认 Confluence spaceId = "123456"
- 默认 cloudId 从站点 URL 获取
- 所有搜索默认 maxResults: 10
```

> `/sse` 端点仍然支持但已标记为废弃，建议使用 `/mcp` 端点。
