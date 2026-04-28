# PIG AI 编码辅助规则

> 基于 [pig-mesh/pig](https://github.com/pig-mesh/pig) — 企业级快速开发平台

## 项目概览

| 维度 | 内容 |
|------|------|
| 架构 | Spring Cloud 微服务 + 单体模式（pig-boot） |
| 后端 | Spring Boot 3.5 + Spring Cloud 2025 + Spring Cloud Alibaba 2025 |
| 前端 | Vue 3.5 + Element Plus 2.8 |
| 鉴权 | Spring Authorization Server 1.5（OAuth2） |
| ORM | MyBatis Plus 3.5 |
| 注册中心 | Nacos |
| 网关 | Spring Cloud Gateway |
| 分布式事务 | Seata |
| JDK | 17+ |
| 构建 | Maven |
| 代码格式 | spring-javaformat（强制） |

## 模块结构

```
pig
├── pig-boot              -- 单体模式启动器 [9999]
├── pig-auth              -- 授权服务 [3000]
├── pig-common            -- 系统公共模块
│   ├── pig-common-bom    -- 全局依赖管理
│   ├── pig-common-core   -- 公共工具类核心包
│   ├── pig-common-datasource -- 动态数据源
│   ├── pig-common-log    -- 日志服务
│   ├── pig-common-oss    -- 文件上传
│   ├── pig-common-mybatis -- MyBatis 扩展
│   ├── pig-common-seata  -- 分布式事务
│   ├── pig-common-websocket -- WebSocket
│   ├── pig-common-security -- 安全工具类
│   ├── pig-common-swagger -- 接口文档
│   ├── pig-common-feign  -- Feign 扩展
│   └── pig-common-xss    -- XSS 安全
├── pig-register          -- Nacos Server [8848]
├── pig-gateway           -- API 网关 [9999]
├── pig-upms              -- 用户权限管理
│   ├── pig-upms-api      -- 公共 API 模块
│   └── pig-upms-biz      -- 业务处理 [4000]
└── pig-visual
    ├── pig-monitor       -- 服务监控 [5001]
    ├── pig-codegen       -- 代码生成 [5002]
    └── pig-quartz        -- 定时任务 [5007]
```

## 目录索引

| 文件 | 用途 |
|------|------|
| `rules/coding-standards.md` | 编码规范、命名约定、注释规范 |
| `rules/package-rules.md` | 包结构规则、依赖引入规范 |
| `rules/spring-format.md` | spring-javaformat 代码格式化要求 |
| `rules/module-rules.md` | 各微服务模块开发规则 |
| `code-review/checklist.md` | 代码审查清单 |
| `code-review/security.md` | 安全审查规则 |
| `code-review/performance.md` | 性能审查规则 |
| `skills/module-dev.md` | 模块开发 Skill |
| `skills/codegen.md` | 代码生成 Skill |
| `skills/feign-api.md` | Feign 远程调用 Skill |
| `ai-guidelines.md` | AI 编码助手总指南 |
