# 架构分层规则

> 定义 RuoYi-Vue-Pro 项目的模块边界、包结构和依赖方向。

## 模块依赖方向

```
yudao-server (启动入口)
  └── yudao-module-* (业务模块，互相独立)
       └── yudao-framework (框架 starter)
            └── Spring Boot 生态
```

**严格规则**:
- 业务模块只能依赖 `yudao-framework` 和自身的 `api/` 包
- 模块 A 可以通过 `api/` 包调用模块 B，但 B 不可反向依赖 A
- 禁止模块间直接 `service/` 层互相调用，必须通过 `api/` 接口暴露
- 所有业务模块的公共接口放在 `yudao-module-*/api/` 包中

## 模块内部包结构

每个业务模块 (`yudao-module-{module}`) 内部结构：

```
cn.iocoder.yudao.module.{module}/
  api/                      # 跨模块 API 接口（RPC/服务暴露）
    {submodule}/
      XxxApi.java            # API 接口定义
      dto/
        XxxReqDTO.java       # 跨模块请求 DTO
        XxxRespDTO.java      # 跨模块响应 DTO
  controller/
    admin/                   # 管理后台 Controller
      {submodule}/
        XxxController.java
        vo/
          XxxPageReqVO.java  # 分页查询请求
          XxxSaveReqVO.java  # 新增/修改请求
          XxxRespVO.java     # 响应 VO
          XxxSimpleRespVO.java # 简化响应（下拉选择）
    app/                     # 用户端 Controller（部分模块）
  convert/                   # MapStruct 转换器
    XxxConvert.java
  dal/
    dataobject/              # 数据库实体（DO = Data Object）
      {submodule}/
        XxxDO.java
    mysql/                   # MyBatis Mapper 接口
      {submodule}/
        XxxMapper.java
    redis/                   # Redis DAO
      XxxRedisDAO.java
  enums/                     # 枚举
    ErrorCodeConstants.java  # 模块错误码
    XxxTypeEnum.java
  service/                   # 业务逻辑
    {submodule}/
      XxxService.java        # 接口
      XxxServiceImpl.java    # 实现
  job/                       # 定时任务
  mq/                        # 消息队列
  util/                      # 模块工具类
```

## 分层调用链

**请求流**: `Controller → VO → Service → Mapper → DO`
**响应流**: `DO → Convert → VO → Controller → CommonResult`

**调用规则**:
- Controller 只能调用 Service，不可跳过 Service 直接调 Mapper
- Service 只能通过 Mapper 访问数据库，不可在 Service 中拼装 SQL
- Mapper 只能操作 DO，不涉及业务逻辑
- 跨模块调用通过 `api/` 接口，不直接依赖其他模块的 Service

## Service 层拆分

复杂业务必须在 Service 内部进行职责分离：

```java
@Service
public class XxxServiceImpl implements XxxService {

    @Resource
    private XxxMapper xxxMapper;

    @Resource
    private XxxHelper helper;  // 辅助 Service，处理复杂逻辑

    // 接口方法实现：只做参数校验 → 委托 → 返回
    @Override
    public Long createXxx(XxxSaveReqVO reqVO) {
        validateBeforeCreate(reqVO);
        return helper.doCreate(reqVO);
    }
}
```
