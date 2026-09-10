
# 什么需要配置文件？
### 1. 核心目的：解耦
把会变动的东西（端口号、数据库密码、第三方 API 密钥）从 Java 代码里抽出来，放到配置文件中。
**好处**：
- 改配置不用重新编译代码
- 不同环境（开发/测试/生产）用不同配置文件，代码不变
- 运维可以改配置，开发不用管
### 2. Spring Boot 支持哪几种配置文件？
优先级从低到高：`application.yaml` < `application.yml` < `application.properties`
> 官方推荐用 `yml`，但一个项目里**只留一种**，别混用（会冲突覆盖，排查起来很痛苦）。
## 二、`properties` 配置文件详解
### 1. 基本语法
```java
# 格式：key=value
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
```
### 2. 读取方式一：`@Value`（单个读取）
```java
@RestController
public class ReadController {
    @Value("${server.port}")        // 从配置文件读 server.port
    private String port;
    
    @Value("${server.port:9090}")  // 没配就用默认值 9090
    private Integer portWithDefault;
    
    @PostConstruct
    public void print() {
        System.out.println(port);
    }
}
```
### 3. 读取方式二：`@ConfigurationProperties`（批量绑定）
```java
@Component
@Data   // 必须！Spring 靠 setter 注入
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {
    private String url;
    private String username;
    private String password;
}
```
### 4. `properties` 的优缺点

|优点|缺点|
|---|---|
|语法简单，不容易写错|层级多时重复前缀，冗余|
|所有 IDE 都支持|默认 ISO-8859-1，中文容易乱码|
||不支持 List/Map 等复杂结构|
||没有格式校验，写错 key 不提示|

## 三、`yml/yaml` 配置文件详解
### 1. 基本语法规则（⚠️ 极其严格）
```java
# ✅ 正确写法
server:
  port: 8080          # 冒号后必须有空格
  servlet:
    context-path: /api  # 缩进用空格（不能用 Tab）

# ❌ 常见错误
server:
  port:8080          # 冒号后没空格 → 报错
  servlet:
    context-path:/api  # 没空格 → 报错
```
**三条铁律**：
1. `key:` 后面**必须有空格**
2. 缩进**只能用空格**，不能用 Tab
3. 同级必须左对齐
### 2. 数据类型写法大全
**普通值**：
```java
string-value: hello
number-value: 100
boolean-value: true
```
**对象**：
```java
user:
  name: zhangsan
  age: 18
# 行内写法：
user: {name: zhangsan, age: 18}
```
**List/数组**（用 `-` 表示元素）：
```java

dbtypes:
  name:
    - mysql
    - sqlserver
    - db2
# 行内写法：
dbtypes: {name: [mysql, sqlserver, db2]}
```
**Map**：
```java
maptypes:
  map:
    k1: v1
    k2: v2
    k3: v3
# 行内写法：
maptypes: {map: {k1: v1, k2: v2, k3: v3}}
```
### 3. 用`@ConfigurationProperties` 绑定复杂类型
**Java 类**：
```java
@Component
@Data
@ConfigurationProperties(prefix = "dbtypes")
public class ListConfig {
    private List<String> name;   // 绑定 dbtypes.name 列表
}

@Component
@Data
@ConfigurationProperties(prefix = "maptypes")
public class MapConfig {
    private Map<String, String> map;  // 绑定 maptypes.map
}
```
**Controller 中使用**：
```java
@RestController
public class ReadYmlController {
    @Autowired
    private ListConfig listConfig;
    
    @Autowired
    private MapConfig mapConfig;
    
    @RequestMapping("/readList")
    public String readList() {
        return listConfig.toString();   // 打印 List
    }
    
    @RequestMapping("/readMap")
    public String readMap() {
        return mapConfig.toString();    // 打印 Map
    }
}
```

### 4. `yml` 的优缺点

|优点|缺点|
|---|---|
|层级清晰，可读性强|对格式要求极其严格（空格就能让你崩溃）|
|支持 List/Map/对象等复杂结构|层级太深时反而难读|
|默认 UTF-8，中文无乱码|手写容易缩进错|
|官方推荐||

## 四、配置优先级（面试常考）
### 1. 同名配置覆盖顺序（优先级从低到高）
```java
application.yaml
    ↓ 被覆盖
application.yml
    ↓ 被覆盖
application.properties
    ↓ 被覆盖
Java 系统属性（-Dserver.port=9090）
    ↓ 被覆盖
命令行参数（--server.port=9090）
```
**结论**：命令行参数优先级最高，可以在部署时临时改配置。
### 2. 配置文件存放路径优先级
```java
项目根目录/config/        ← 最高
项目根目录/
src/main/resources/config/
src/main/resources/       ← 默认
```
## 五、IDE 黄色警告消除
在 `pom.xml` 中加入：
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```
然后 **Rebuild Project**，yml 里写配置就会有自动提示了。
## 六、综合实战：验证码功能（Hutool）
### 1. 引入依赖
```java
<dependencies>
    <!-- Web MVC -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>
    
    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- Hutool 工具包 -->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.22</version>
    </dependency>
</dependencies>
```

### 2. 配置文件

```java
captcha:
  width: 200
  height: 100
  key: CAPTCHA_SESSION_KEY
  date: CAPTCHA_SESSION_DATE
```

### 3. 配置属性类

```java
@Component
@Data
@ConfigurationProperties(prefix = "captcha")
public class CaptchaProperties {
    private Integer width;
    private Integer height;
    private String key;
    private String date;
}
```
### 4. Controller 完整代码
```java
@RestController
public class HutoolCaptchaController {
    
    @Autowired
    private CaptchaProperties captchaProperties;
    
    // 生成验证码
    @RequestMapping("/captcha/getCaptcha")
    public void getCaptcha(HttpSession session, HttpServletResponse response) {
        // 创建线段干扰验证码
        LineCaptcha captcha = CaptchaUtil.createLineCaptcha(
            captchaProperties.getWidth(),
            captchaProperties.getHeight()
        );
        
        try {
            response.setContentType("image/jpeg");
            captcha.write(response.getOutputStream());
            response.getOutputStream().close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        
        // 存入 Session
        session.setAttribute(captchaProperties.getKey(), captcha.getCode());
        session.setAttribute(captchaProperties.getDate(), new Date());
    }
    
    // 校验验证码
    @RequestMapping("/captcha/check")
    public boolean checkCaptcha(String captcha, HttpSession session) {
        long VALID_TIME = 60 * 1000; // 60秒有效
        
        if (captcha == null || captcha.isEmpty()) return false;
        
        String correctCode = (String) session.getAttribute(captchaProperties.getKey());
        Date createTime = (Date) session.getAttribute(captchaProperties.getDate());
        
        // 无论成功失败，用完就清
        session.removeAttribute(captchaProperties.getKey());
        session.removeAttribute(captchaProperties.getDate());
        
        // 校验：不为空 + 匹配 + 未过期
        if (correctCode != null 
            && correctCode.equalsIgnoreCase(captcha)
            && System.currentTimeMillis() - createTime.getTime() < VALID_TIME) {
            return true;
        }
        return false;
    }
}
```
### 5. 前端页面（index.html，放在 `resources/static/` 下）
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>验证码</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
    <h1>验证码测试</h1>
    <img id="captchaImg" src="/captcha/getCaptcha" onclick="refresh()" style="cursor:pointer;" />
    <br><br>
    <input type="text" id="code" placeholder="请输入验证码" />
    <button onclick="check()">提交</button>

    <script>
        function refresh() {
            $("#captchaImg").attr("src", "/captcha/getCaptcha?" + new Date().getTime());
        }
        function check() {
            $.post("/captcha/check", {captcha: $("#code").val()}, function(res) {
                if (res) {
                    alert("验证成功！");
                    location.href = "success.html";
                } else {
                    alert("验证码错误！");
                    refresh();
                }
            });
        }
    </script>
</body>
</html>
```
## 七、踩过的所有坑（重点复习）

|坑|原因|解决|
|---|---|---|
|`@Value` 注入为 null|没加 `@Value` 注解，或类没被 Spring 管理|加 `@Value("${key}")` + 确保类有 `@Component`/`@RestController`|
|`@Value` 报 `NumberFormatException`|写成 `{key}` 而不是 `${key}`|改为 `${key:默认值}`|
|`@ConfigurationProperties` 绑定为 null|类没加 `@Component`，或没加 `@Data`（无 setter）|两个注解都加上|
|`@Autowired` 报红|注入的类没被 Spring 管理|确保目标类有 `@Component`/`@Service` 等|
|yml 配置不生效|缩进用了 Tab，或冒号后没空格|改为空格缩进，冒号后加空格|
|Maven 依赖爆红|依赖写在 `<dependencies>` 外面|剪切到 `<dependencies>` 内部|
|Session 验证码可重复使用|校验后没清理|校验完立即 `removeAttribute`|

## 八、一句话总结
> **`properties` 简单粗暴，`yml` 结构清晰但要小心空格。读配置用 `@Value` 读单个、用 `@ConfigurationProperties` 批量绑定。记住三条铁律：配置类要加 `@Component`、要加 `@Data`、依赖要写在 `<dependencies>` 里面。验证码案例练的是 IoC 注入 + Session 操作 + Hutool 工具包的综合运用。**

