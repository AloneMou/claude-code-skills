---
name: ruoyi-config
description: 为芋道 RuoYi-Vue-Pro 项目生成配置文件、Docker 部署配置和环境变量管理
trigger: 当用户提到配置文件、Docker、部署、环境变量、yudao.yaml、application、docker-compose
---

# RuoYi Config — 配置与部署

为芋道 (Yudao) RuoYi-Vue-Pro 项目生成标准化配置和部署脚本。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取项目级技术栈。

## TRIGGER

- 需要新增配置文件（开发/测试/生产环境）
- 需要 Docker 部署配置
- 需要配置新的 Redis/数据库/消息队列
- 提到 "配置"、"部署"、"Docker"、"环境变量"

## 配置文件结构

```
yudao-server/src/main/resources/
  application.yaml              # 主配置
  application-dev.yaml          # 开发环境
  application-test.yaml         # 测试环境
  application-prod.yaml         # 生产环境
  logback-spring.xml            # 日志配置
```

## 核心配置项

```yaml
spring:
  datasource:
    url: jdbc:mysql://${DB_HOST:127.0.0.1}:${DB_PORT:3306}/${DB_DATABASE:yudao}?useSSL=false
    username: ${DB_USER:root}
    password: ${DB_PASSWORD:123456}
    driver-class-name: com.mysql.cj.jdbc.Driver

  data:
    redis:
      host: ${REDIS_HOST:127.0.0.1}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      database: ${REDIS_DATABASE:0}

# 多租户配置
yudao:
  tenant:
    enable: ${TENANT_ENABLE:false}
    ignore-urls:
      - /system/captcha/get
      - /system/captcha/check
```

## Docker Compose 模板

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  yudao-server:
    build: .
    ports:
      - "48080:48080"
    environment:
      DB_HOST: mysql
      REDIS_HOST: redis
    depends_on:
      - mysql
      - redis

volumes:
  mysql-data:
```

## 环境变量清单

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `DB_HOST` | MySQL 地址 | `127.0.0.1` |
| `DB_PORT` | MySQL 端口 | `3306` |
| `DB_DATABASE` | 数据库名 | `yudao` |
| `DB_USER` | 数据库用户 | `root` |
| `DB_PASSWORD` | 数据库密码 | `123456` |
| `REDIS_HOST` | Redis 地址 | `127.0.0.1` |
| `REDIS_PORT` | Redis 端口 | `6379` |
| `REDIS_PASSWORD` | Redis 密码 | (空) |
| `REDIS_DATABASE` | Redis DB 编号 | `0` |
| `TENANT_ENABLE` | 是否启用多租户 | `false` |
