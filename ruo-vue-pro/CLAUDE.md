# CLAUDE.md — 芋道 RuoYi-Vue-Pro AI 编程辅助

> 芋道 RuoYi-Vue-Pro (JDK17) 项目的 AI 编程辅助规则集合。
> 本文档作为路由文件，具体规范请查阅 `.claude/rules/` 下的模块化文件。

---

## 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Java | 17 | |
| Spring Boot | 3.5.9 | Jakarta EE 命名空间 |
| MyBatis-Plus | 3.5.15 | ORM 框架 |
| MyBatis-Plus-Join | 1.5.5 | 关联查询 |
| Druid | 1.2.27 | 连接池 |
| Redis (Redisson) | 3.52.0 | 缓存/分布式锁 |
| Dynamic Datasource | 4.5.0 | 多数据源 |
| Lombok | 1.18.42 | |
| MapStruct | 1.6.3 | 对象转换 |
| Hutool | 5.8.42 | 工具库 |
| SpringDoc / Knife4j | 2.8.14 / 4.5.0 | API 文档 |
| Flowable | 7.2.0 | 工作流 |
| FastExcel | 1.3.0 | Excel 导入导出 |

---

## 项目结构

```
yudao-dependencies/          # BOM，统一管理所有依赖版本
yudao-framework/             # 框架 starter（web/security/mybatis/redis 等）
yudao-server/                # 启动入口，聚合业务模块
yudao-module-system/         # 系统核心（用户/角色/菜单/部门/租户等）
yudao-module-infra/          # 基础设施（代码生成/文件/配置/API日志）
yudao-module-*/              # 业务模块（ai/bpm/crm/erp/iot/mall/member/pay 等）
sql/                         # 数据库初始化脚本（多种数据库）
script/                      # 部署脚本（Docker 等）
```

**基础包名**: `cn.iocoder.yudao`

---

## 目录说明

```
ruo-vue-pro/
├── CLAUDE.md                    # 本项目规则路由文件（本文件）
├── README.md                    # 项目索引
└── .claude/
    ├── skills/                  # 可复用技能
    │   ├── ruoyi-crud/          # CRUD 模块生成（DO/Mapper/Service/Controller/VO/ErrorCode）
    │   ├── ruoyi-api/           # 跨模块 API 设计（Api接口 + DTO + MapStruct + ApiImpl）
    │   ├── ruoyi-mybatis/       # MyBatis-Plus 数据访问模式
    │   ├── ruoyi-review/        # 代码规范审查
    │   ├── ruoyi-architect/     # 架构设计与方案规划
    │   ├── ruoyi-security/      # 安全审计与防护
    │   ├── ruoyi-test/          # 单元测试生成
    │   └── ruoyi-config/        # 配置与部署
    ├── rules/                   # 模块化规则文件（自动加载）
    │   ├── architecture.md      # 分层架构与模块边界
    │   ├── naming.md            # 命名约定
    │   ├── controller.md        # Controller 开发规范
    │   ├── service.md           # Service 开发规范
    │   ├── data-access.md       # 数据访问规范
    │   ├── vo-dto.md            # VO/DTO 规范
    │   ├── error-codes.md       # 错误码规范
    │   └── prohibited.md        # 禁止事项清单
    ├── subagents/               # 专用子代理
    │   ├── architect.md         # 架构师：设计方案和模块拆分
    │   ├── reviewer.md          # 审查员：代码审查和质量检查
    │   └── generator.md         # 生成器：代码骨架生成
    └── settings.json            # 项目设置（权限、钩子等）
```

---

## 技能列表

| 技能 | 触发关键词 | 说明 |
|------|-----------|------|
| `ruoyi-crud` | 新建CRUD、创建XX模块、增删改查 | 生成完整 CRUD 模块 |
| `ruoyi-api` | 跨模块API、创建API接口、暴露服务 | 跨模块服务调用 |
| `ruoyi-mybatis` | 数据库查询、分页、多表联查 | MyBatis-Plus 数据操作 |
| `ruoyi-review` | 代码审查、review、检查规范 | 代码规范检查 |
| `ruoyi-architect` | 架构设计、方案规划、模块拆分 | 架构方案设计 |
| `ruoyi-security` | 安全审计、安全检查、XSS、SQL注入 | 安全漏洞检查 |
| `ruoyi-test` | 单元测试、编写测试 | 测试代码生成 |
| `ruoyi-config` | 配置文件、Docker、部署 | 环境配置 |

---

## 核心规则索引

具体开发规范请查阅 `.claude/rules/` 下的文件：

- [架构分层](.claude/rules/architecture.md) — 模块边界、包结构、依赖方向
- [命名约定](.claude/rules/naming.md) — 类/方法/字段/包命名规则
- [Controller](.claude/rules/controller.md) — REST接口开发规范
- [Service](.claude/rules/service.md) — 业务逻辑层规范
- [数据访问](.claude/rules/data-access.md) — Mapper/DO/SQL规范
- [VO/DTO](.claude/rules/vo-dto.md) — 视图对象/数据传输对象规范
- [错误码](.claude/rules/error-codes.md) — 错误码定义和使用
- [禁止事项](.claude/rules/prohibited.md) — 严格禁止的做法
