# 芋道 RuoYi-Vue-Pro — AI 编程辅助规则

> 基于 Claude Code `.claude/` 目录规范，为 RuoYi-Vue-Pro 项目定制的 AI 编程辅助工具集。

---

## 快速开始

将 `ruo-vue-pro/` 下的文件复制到你的项目根目录：

```bash
# 复制 CLAUDE.md
cp ruo-vue-pro/CLAUDE.md /path/to/your/ruoyi-vue-pro/

# 复制 .claude/ 目录
cp -r ruo-vue-pro/.claude/ /path/to/your/ruoyi-vue-pro/
```

---

## 目录结构

```
.claude/
├── rules/                    # 模块化规则文件（自动加载）
│   ├── architecture.md       # 分层架构与模块边界
│   ├── naming.md             # 命名约定
│   ├── controller.md         # Controller 开发规范
│   ├── service.md            # Service 开发规范
│   ├── data-access.md        # 数据访问规范
│   ├── vo-dto.md             # VO/DTO 规范
│   ├── error-codes.md        # 错误码规范
│   └── prohibited.md         # 禁止事项清单
├── skills/                   # 可复用技能
│   ├── ruoyi-crud/           # CRUD 模块生成
│   ├── ruoyi-api/            # 跨模块 API 设计
│   ├── ruoyi-mybatis/        # MyBatis-Plus 数据访问
│   ├── ruoyi-review/         # 代码审查
│   ├── ruoyi-architect/      # 架构设计与方案规划
│   ├── ruoyi-security/       # 安全审计
│   ├── ruoyi-test/           # 单元测试生成
│   └── ruoyi-config/         # 配置与部署
├── subagents/                # 专用子代理
│   ├── architect.md          # 架构师：方案设计
│   ├── reviewer.md           # 审查员：代码审查
│   └── generator.md          # 生成器：代码骨架
└── settings.json             # 项目设置
```

---

## 技能使用指南

| 场景 | 使用技能 | 触发方式 |
|------|---------|----------|
| 新建一个业务模块 | `ruoyi-crud` | 说 "创建用户管理模块" |
| 跨模块调用服务 | `ruoyi-api` | 说 "暴露用户查询 API 给其他模块" |
| 写复杂数据库查询 | `ruoyi-mybatis` | 说 "写一个多表联查的分页" |
| 代码审查 | `ruoyi-review` | 说 "审查这段代码" |
| 新模块架构设计 | `ruoyi-architect` | 说 "设计一个订单模块" |
| 安全漏洞检查 | `ruoyi-security` | 说 "安全检查" |
| 生成测试用例 | `ruoyi-test` | 说 "为 OrderService 写测试" |
| 配置和部署 | `ruoyi-config` | 说 "生成 Docker 配置" |

---

## 子代理

在需要深度分析或大规模代码生成时，主代理会自动分派给专用子代理：

- **架构师** (`architect`) — 需求分析 → 数据库设计 → 文件清单 → 实现步骤
- **审查员** (`reviewer`) — 规范检查 + 安全检查 + 性能检查 → 审查报告
- **生成器** (`generator`) — 按架构方案逐文件生成代码
