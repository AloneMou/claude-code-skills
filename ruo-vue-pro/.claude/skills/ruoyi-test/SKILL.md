---
name: ruoyi-test
description: 为芋道 RuoYi-Vue-Pro 项目生成单元测试代码（Service/Mapper/Controller 层测试）
trigger: 当用户提到单元测试、编写测试、测试用例、test case、ServiceTest、MapperTest
---

# RuoYi Test — 单元测试生成

为芋道 (Yudao) RuoYi-Vue-Pro 项目生成标准化的单元测试。

**共享规范**: 阅读 [CLAUDE.md](../../CLAUDE.md) 获取项目级命名约定和技术栈。

## TRIGGER

- 要求为某个 Service/Controller 写测试
- 提到 "单元测试"、"test case"、"测试覆盖"
- 代码审查时发现缺少测试

## SKIP

- 纯前端测试
- 集成测试（数据库/外部依赖）

## 测试目录

测试文件放在对应模块的 `src/test/java/` 目录下，包路径与源码一致：

```
yudao-module-{module}/src/test/java/cn/iocoder/yudao/module/{module}/
  service/{submodule}/
    XxxServiceImplTest.java
  controller/admin/{submodule}/
    XxxControllerTest.java
```

## Service 测试模板

```java
@Import({XxxServiceImpl.class})
public class XxxServiceImplTest extends BaseDbUnitTest {

    @Resource
    private XxxServiceImpl xxxService;

    @Resource
    private XxxMapper xxxMapper;

    @Test
    public void testCreateXxx_success() {
        // 准备数据
        XxxSaveReqVO reqVO = randomPojo(XxxSaveReqVO.class);
        // 执行
        Long id = xxxService.createXxx(reqVO);
        // 验证
        assertNotNull(id);
        XxxDO dbXxx = xxxMapper.selectById(id);
        assertEquals(reqVO.getName(), dbXxx.getName());
    }

    @Test
    public void testGetXxx_notFound() {
        // 执行并断言异常
        assertServiceException(() -> xxxService.getXxx(0L), XXX_NOT_FOUND);
    }
}
```

## Controller 测试模板

```java
@AutoConfigureMockMvc
@AutoConfigureWebTestClient
public class XxxControllerTest extends BaseMockitoUnitTest {

    @InjectMocks
    private XxxController xxxController;

    @MockBean
    private XxxService xxxService;

    @Test
    public void testCreateXxx_success() throws Exception {
        XxxSaveReqVO reqVO = randomPojo(XxxSaveReqVO.class, o -> o.setId(null));
        Long returnId = 1L;
        when(xxxService.createXxx(eq(reqVO))).thenReturn(returnId);
        Long createId = xxxController.createXxx(reqVO).getCheckedData();
        assertEquals(returnId, createId);
        verify(xxxService).createXxx(eq(reqVO));
    }
}
```

## Rules

1. 每个 Service 公共方法至少一个测试用例（成功路径 + 异常路径）
2. Mock 外部依赖，不测真实数据库（Controller 层）/ 用内存 H2 数据库（Service 层）
3. 用 `randomPojo()` 生成测试数据
4. 用 `assertServiceException()` 断言业务异常
5. 测试方法命名: `test方法名_场景`
