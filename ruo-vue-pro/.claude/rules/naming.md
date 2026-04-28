# 命名约定

> 定义 RuoYi-Vue-Pro 项目中所有类、方法、字段的命名规则。

## 类命名

| 类型 | 模式 | 示例 |
|------|------|------|
| 实体类 (Data Object) | `XxxDO` | `PostDO`, `AdminUserDO` |
| Mapper 接口 | `XxxMapper` | `PostMapper`, `AdminUserMapper` |
| Service 接口 | `XxxService` | `PostService`, `DeptService` |
| Service 实现 | `XxxServiceImpl` | `PostServiceImpl` |
| Helper Service | `XxxHelper` / `XxxHelperImpl` | `OrderHelper` |
| Controller | `XxxController` | `PostController` |
| 分页请求 VO | `XxxPageReqVO` | `PostPageReqVO` |
| 保存请求 VO | `XxxSaveReqVO` | `PostSaveReqVO` |
| 响应 VO | `XxxRespVO` | `PostRespVO` |
| 简化响应 VO | `XxxSimpleRespVO` | `PostSimpleRespVO` |
| 跨模块 DTO | `XxxReqDTO` / `XxxRespDTO` | `SmsCodeSendReqDTO` |
| 枚举 | `XxxEnum` | `CommonStatusEnum` |
| 错误码常量接口 | `ErrorCodeConstants` | 无命名后缀 |
| MapStruct 转换器 | `XxxConvert` | `PostConvert` |
| 工具类 | `XxxUtils` | `PostUtils` |
| 定时任务 | `XxxJob` | `DemoJob` |
| 消息生产者 | `XxxProducer` | `OrderProducer` |
| 消息消费者 | `XxxConsumer` | `OrderConsumer` |

## 方法命名

| 操作类型 | 模式 | 示例 |
|----------|------|------|
| 创建 | `createXxx` | `createPost` |
| 更新 | `updateXxx` | `updatePost` |
| 删除 | `deleteXxx` | `deletePost` |
| 批量删除 | `deleteXxxList` | `deletePostList` |
| 按ID查询 | `getXxx` | `getPost` |
| 分页查询 | `getXxxPage` | `getPostPage` |
| 列表查询 | `getXxxList` | `getPostList` |
| 校验方法 | `validateXxxForCreateOrUpdate` | `validatePostForCreateOrUpdate` |
| 校验存在 | `validateXxxExists` | `validatePostExists` |

## 字段命名

- **Java**: 驼峰命名（camelCase）
- **数据库**: 下划线命名（snake_case）
- MyBatis-Plus 自动映射，无需配置

### 状态/类型字段

- 统一用 `Integer` 类型
- JavaDoc 中注明枚举引用
- 示例: `private Integer status; // 状态，参见 CommonStatusEnum`

## URL 路径命名

```
POST   /{module}/{resource}/create        # 创建
PUT    /{module}/{resource}/update        # 修改
DELETE /{module}/{resource}/delete        # 删除
DELETE /{module}/{resource}/delete-list   # 批量删除
GET    /{module}/{resource}/get           # 详情
GET    /{module}/{resource}/page          # 分页
GET    /{module}/{resource}/list          # 列表
GET    /{module}/{resource}/list-all-simple # 下拉选择
GET    /{module}/{resource}/export-excel  # 导出
```

## 权限标识命名

格式: `{模块}:{资源}:{操作}`

| 操作 | 权限标识 |
|------|----------|
| 创建 | `{module}:{resource}:create` |
| 修改 | `{module}:{resource}:update` |
| 删除 | `{module}:{resource}:delete` |
| 查询 | `{module}:{resource}:query` |
| 导出 | `{module}:{resource}:export` |

示例: `system:post:create`, `system:post:update`, `system:post:delete`
