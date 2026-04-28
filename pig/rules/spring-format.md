# PIG 代码格式化规范

## spring-javaformat 强制要求

本项目使用 [spring-javaformat](https://github.com/spring-io/spring-javaformat) 作为代码格式化工具，**所有代码提交前必须经过格式化**。

## 1. 格式化规则

### 1.1 缩进

- 使用 **Tab** 字符缩进（不是空格）
- IDE 需配置 Tab 宽度为 1 个 Tab 字符

### 1.2 行长度

- 每行最大 **90** 字符
- 超过长度时自动换行

### 1.3 导入排序

导入语句按以下顺序自动排序：

1. `java.*` 和 `javax.*`
2. 第三方库（按字母排序）
3. 项目内部包（按字母排序）

组与组之间有一个空行分隔。

### 1.4 括号

- 左花括号不换行，与前面代码在同一行
- 控制语句括号后无空格：`if (condition)` 而非 `if ( condition )`
- 方法调用括号前无空格：`method(arg)` 而非 `method (arg)`

### 1.5 空格

- 运算符两侧加空格：`a + b` 而非 `a+b`
- 逗号后加空格：`method(a, b)` 而非 `method(a,b)`
- `for` 循环分号后加空格：`for (int i = 0; i < 10; i++)`

## 2. 使用方式

### 2.1 Maven 命令

```bash
# 检查代码格式（CI 使用）
mvn spring-javaformat:validate

# 自动格式化
mvn spring-javaformat:apply
```

### 2.2 IntelliJ IDEA

1. 下载对应版本的插件：
   [spring-javaformat-intellij-idea-plugin](https://repo1.maven.org/maven2/io/spring/javaformat/spring-javaformat-intellij-idea-plugin/)
2. 安装后自动启用格式化

### 2.3 Eclipse

参考 [spring-javaformat Eclipse 配置](https://github.com/spring-io/spring-javaformat#eclipse)

## 3. CI 集成

GitHub Actions / GitLab CI 中必须包含格式化检查：

```yaml
- name: Validate code format
  run: mvn spring-javaformat:validate
```

## 4. 开发者注意事项

1. **提交代码前必须执行 `mvn spring-javaformat:apply`**
2. 格式化不通过的 PR 不能被合并
3. 不要手动调整已格式化好的代码排版
4. IDE 中建议配置保存时自动运行 formatter

## 5. 常见格式化问题

| 问题 | 解决 |
|------|------|
| 导入顺序错误 | 运行 `mvn spring-javaformat:apply` |
| 缩进不一致 | 确保使用 Tab 而非空格 |
| 行超长 | 让 formatter 自动换行，不要手动断行 |
| 注释格式不对 | formatter 会统一调整注释样式 |
