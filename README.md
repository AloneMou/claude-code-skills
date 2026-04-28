# Claude Code Skills — 芋道 RuoYi-Vue-Pro

基于 [ruoyi-vue-pro (master-jdk17)](https://github.com/YunaiV/ruoyi-vue-pro/tree/master-jdk17) 项目结构编写的 Claude Code 规则与技能。

## 项目规则

| 文件 | 说明 |
|------|------|
| [CLAUDE.md](CLAUDE.md) | 项目级规则（Claude Code 自动读取），包含技术栈、命名约定、Controller/Service/Mapper/DO/VO 规范、错误码规则等 |

## Skills

| Skill | 触发条件 | 说明 |
|-------|----------|------|
| `ruoyi-crud` | "新建CRUD"、"创建业务模块"、"新增增删改查" | 生成完整的 CRUD 模块（DO/Mapper/Service/Controller/VO/ErrorCode） |
| `ruoyi-api` | "跨模块API"、"创建API接口"、模块间调用 | 生成跨模块 API 接口 + DTO + 转换器 + 实现类 |
| `ruoyi-review` | "review this"、"代码审查"、检查代码质量 | 审查代码是否符合芋道框架规范 |
| `ruoyi-mybatis` | "写查询"、"数据库操作"、"Mapper"、复杂条件查询 | MyBatis-Plus 查询模式指南（LambdaQueryWrapperX、分页、联表、类型处理器） |

## 技能使用方式

在 Claude Code 中通过 `/ruoyi-crud`、`/ruoyi-api`、`/ruoyi-review`、`/ruoyi-mybatis` 调用。

## 技术栈

- **Java 17** / **Spring Boot 3.5.9** (Jakarta EE)
- **MyBatis-Plus 3.5.15** / **MyBatis-Plus-Join 1.5.5**
- **Druid** / **Redis (Redisson)** / **Dynamic Datasource**
- **Lombok** / **MapStruct** / **Hutool**
- **SpringDoc** / **Knife4j**
