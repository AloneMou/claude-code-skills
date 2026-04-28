# PIG 包结构与依赖规则

## 1. 标准模块包结构

每个业务模块（如 pig-upms）应遵循以下结构：

```
pig-{module}
├── pig-{module}-api              -- API 模块（对外暴露的接口、DTO、实体）
│   └── src/main/java/com/pig4cloud/pig/{module}/api
│       ├── dto/                  -- 数据传输对象
│       │   ├── XxxDTO.java
│       │   └── XxxReq.java
│       ├── vo/                   -- 视图对象
│       │   └── XxxVO.java
│       ├── entity/               -- 实体类
│       │   └── Xxx.java
│       ├── feign/                -- Feign 远程调用接口
│       │   └── RemoteXxxService.java
│       └── constant/             -- 常量定义
│           └── XxxConstants.java
└── pig-{module}-biz              -- 业务模块（内部实现）
    └── src/main/java/com/pig4cloud/pig/{module}
        ├── controller/           -- 控制器
        │   └── XxxController.java
        ├── service/              -- 服务层
        │   ├── XxxService.java
        │   └── impl/
        │       └── XxxServiceImpl.java
        ├── mapper/               -- 数据访问层
        │   └── XxxMapper.java
        ├── config/               -- 配置类
        │   └── XxxConfig.java
        └── bootstrap/            -- 启动类
            └── PigXxxApplication.java
    └── src/main/resources
        ├── application.yml
        ├── bootstrap.yml
        └── mapper/               -- MyBatis XML 映射文件
            └── XxxMapper.xml
```

## 2. 依赖引入规则

### 2.1 全局依赖

以下依赖在根 `pom.xml` 中统一管理，子模块无需重复声明：

```xml
<!-- 已在父 POM 中声明 -->
spring-boot-starter-test
lombok
spring-boot-configuration-processor
jasypt-spring-boot-starter
spring-boot-starter-actuator
spring-boot-admin-starter-client
```

### 2.2 BOM 管理

所有 `pig-common-*` 子模块版本由 `pig-common-bom` 统一管理：

```xml
<!-- 子模块只需声明 artifactId，版本由 BOM 控制 -->
<dependency>
    <groupId>com.pig4cloud</groupId>
    <artifactId>pig-common-core</artifactId>
</dependency>
```

### 2.3 常用模块依赖

| 需要功能 | 引入依赖 |
|----------|----------|
| 核心工具 | `pig-common-core` |
| MyBatis Plus | `pig-common-mybatis` |
| 安全鉴权 | `pig-common-security` |
| 日志记录 | `pig-common-log` |
| 动态数据源 | `pig-common-datasource` |
| 分布式事务 | `pig-common-seata` |
| 接口文档 | `pig-common-swagger` |
| Feign 调用 | `pig-common-feign` |
| WebSocket | `pig-common-websocket` |
| 文件上传 | `pig-common-oss` |
| XSS 防护 | `pig-common-xss` |

### 2.4 模块间依赖原则

1. **biz 模块依赖 api 模块**：`pig-upms-biz` 依赖 `pig-upms-api`
2. **api 模块不依赖 biz 模块**
3. **跨服务调用通过 api 模块的 Feign 接口**
4. **禁止 biz 模块之间直接依赖**（应通过 Feign 调用）

## 3. 配置文件规则

### 3.1 bootstrap.yml

服务注册相关配置：

```yaml
spring:
  application:
    name: '@artifactId@'
  cloud:
    nacos:
      server-addr: ${NACOS_HOST:127.0.0.1}:${NACOS_PORT:8848}
```

### 3.2 application.yml

服务自身配置：

```yaml
server:
  port: 4000
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/pigx?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: root
```

### 3.3 敏感信息

- 密码等敏感信息使用 Jasypt 加密：`ENC(加密后的密文)`
- 开发环境可使用明文，生产环境必须加密

## 4. 资源文件规则

- MyBatis XML 映射文件放在 `src/main/resources/mapper/` 目录
- Mapper XML 文件名与 Mapper 接口同名：`SysUserMapper.xml`
- SQL 脚本放在 `src/main/resources/db/` 目录

## 5. 前端项目结构（pig-ui）

```
pig-ui
├── src/
│   ├── api/                    -- 接口定义
│   │   └── admin/
│   │       └── user.js
│   ├── views/                  -- 页面
│   │   └── admin/
│   │       └── user/
│   │           ├── index.vue
│   │           └── form.vue
│   ├── components/             -- 公共组件
│   ├── router/                 -- 路由配置
│   ├── store/                  -- 状态管理
│   ├── utils/                  -- 工具函数
│   └── styles/                 -- 全局样式
├── public/
├── package.json
└── vite.config.js
```
