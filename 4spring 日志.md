
# Spring Boot 日志学习笔记

> 环境：Spring Boot + Java 25 + Lombok
> 
> 核心组合：SLF4J + Logback
## 一、本节目标

1. 掌握日志的概念与作用
2. 学会在 Spring Boot 中打印和使用日志
3. 理解日志框架的底层结构
4. 掌握日志级别、格式、持久化及分割配置
5. 使用 Lombok 简化日志输出
## 二、日志概述

- **什么是日志**：程序运行过程中记录关键信息和错误的文本。
- **为什么不用 `System.out.println()`**：
    - 无法控制输出级别，生产环境删加代码成本极高。
    - 性能差，字符串拼接浪费资源。
    - **规范**：开发中不推荐使用 `System.out.println()` 输出核心/敏感数据。
## 三、日志框架介绍（了解）

|角色|框架名称|说明|
|---|---|---|
|**日志门面**​|**SLF4J**​|提供统一 API 接口，让代码与具体实现解耦。|
|**默认实现**​|**Logback**​|SLF4J 作者设计，Spring Boot 默认集成，性能好。|
|**高性能实现**​|Log4j 2|异步日志性能极佳，适合高并发。|
|**老旧框架**​|JCL / Log4j 1.x|已淘汰或有安全漏洞。|

> **核心口诀**：**SLF4J 写代码（遥控器），Logback 做事情（电视机）**。
> 
> Spring Boot 引入 `spring-boot-starter-web` 时自动引入了 `spring-boot-starter-logging`（包含 SLF4J + Logback）。

## 四、日志使用

### 1. 传统方式打印日志

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserService {
    // 每个类手动声明（繁琐）
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    public void addUser(String username) {
        log.info("开始添加用户，用户名：{}", username);
    }
}
```
### 2. 日志格式说明（控制台默认）
`2026-09-13 12:00:00.123 INFO 12345 --- [main] c.e.s.UserService : 开始添加用户...`
- **日期时间**：`%d{yyyy-MM-dd HH:mm:ss.SSS}`
- **日志级别**：`%-5level`（INFO/WARN 等）
- **进程 ID**：`12345`
- **分隔符**：`---`
- **线程名**：`[main]` / `[http-nio-8080-exec-1]`
- **类名**：`c.e.s.UserService`（全限定类名缩写）
- **消息体**：具体日志内容

## 五、日志级别
### 1. 日志级别的分类（从低到高）

1. **TRACE**：追踪信息（最细粒度）
2. **DEBUG**：调试信息（开发环境用）
3. **INFO**：一般信息（生产环境默认）
4. **WARN**：警告信息（代码可运行但不规范）
5. **ERROR**：错误信息（异常捕获）

### 2. 日志级别的使用规则

- **设置级别后**：只输出**当前级别及更高（更严重）级别**的日志。
- **示例**：配置为 `INFO`，则输出 `INFO`、`WARN`、`ERROR`，屏蔽 `DEBUG`、`TRACE`。
- **开发 vs 生产**：开发设 `DEBUG`/`TRACE`，生产设 `INFO`/`WARN`
## 六、日志配置（application.yml）

### 1. 配置日志级别

```java
logging:
  level:
    root: info                 # 全局根级别
    org.springframework: warn  # Spring 框架只输出 WARN 及以上
    org.example.springmybatis.mapper: debug # Mapper 包设 DEBUG 可打印 SQL
```

### 2. 日志持久化（保存到文件）

默认日志只输出到控制台，重启丢失。需配置持久化：

```java
logging:
  file:
    # 方式一：指定完整文件名（相对/绝对路径）
    name: logger/springboot.log 
    # 方式二：仅指定目录，文件名默认为 spring.log
    # path: D:/temp/logs
```

> ⚠️ **坑点**：`name` 和 `path` 同时配置时，**`name` 优先生效**。

### 3. 配置日志文件分割

默认超过 **10MB**​ 自动分割。可自定义：

```java
logging:
  logback:
    rollingpolicy:
      max-file-size: 10MB          # 单个文件最大大小（测试可设小如 1KB）
      file-name-pattern: ${LOG_FILE}.%d{yyyy-MM-dd}.%i # 分割后命名格式
      max-history: 30              # 保留历史天数
      total-size-cap: 1GB          # 总日志大小限制
```

### 4. 配置日志格式

```java
logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5level) [%thread] %cyan(%logger{36}) - %msg%n" # 控制台（带颜色）
    file: "%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n"       # 文件输出（无色）
```

## 七、更简单的日志输出（Lombok）

### 1. 添加 Lombok 依赖

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

> ⚠️ **坑点**：IDEA 需安装 **Lombok 插件**并开启注解处理，否则 `log` 报红。

### 2. 使用 `@Slf4j` 输出日志

在类上加 `@Slf4j`，自动生成 `log` 对象：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class LogController {
    public void test() {
        log.trace("追踪信息");
        log.debug("调试信息，参数：{}", "test"); // 支持 {} 占位符
        log.info("一般信息");
        log.warn("警告信息");
        
        try {
            int i = 1 / 0;
        } catch (Exception e) {
            log.error("错误信息，异常原因：{}", "除零", e); // 传入异常对象打印堆栈
        }
    }
}
```

> **底层原理**：编译时自动插入 `private static final org.slf4j.Logger log = LoggerFactory.getLogger(LogController.class);`。

## 八、常见踩坑记录

|坑|原因|解决|
|---|---|---|
|`log` 变量报红|没装 Lombok 插件或依赖|装插件 + 加依赖 + 开注解处理|
|日志没输出|级别设太高（如全局 WARN）|调低级别到 `DEBUG` 或 `INFO`|
|MyBatis SQL 没打印|Mapper 包没设 `DEBUG`|`logging.level.你的mapper包=DEBUG`|
|日志文件超大|没配置分割|配置 `max-file-size`|
|用了字符串拼接|`log.info("User " + name)`|**改用占位符**​ `log.info("User {}", name)`，提升性能|
|异常只打消息|`log.error("出错:" + e.getMessage())`|**必须传异常对象**​ `log.error("出错", e)`，否则丢失堆栈轨迹|
|生产环境日志爆炸|开了 `DEBUG` 级别|生产环境务必改为 `INFO` 或 `WARN`|

## 九、核心口诀速记

> 🔹 **SLF4J 门面，Logback 实现**（遥控器与电视机）
> 
> 🔹 **开发用 DEBUG，生产用 INFO**​
> 
> 🔹 **级别越低越详细，设了 INFO 屏蔽 DEBUG**​
> 
> 🔹 **Lombok 加 `@Slf4j`，直接用的 `log` 对象**​
> 
> 🔹 **占位符 `{}` 比拼接字符串好**（防性能浪费）
> 
> 🔹 **报错日志必须把异常对象 `e` 传进去**​
> 
> 🔹 **持久化配 `name`，分割配 `max-file-size`**

---
