# 1.了解Spring MVC
### 什么是 Spring MVC？
**1. MVC 设计模式**
MVC 是软件工程中的一种架构设计模式，把系统分为三部分：
- **Model（模型）**：处理业务逻辑和数据（比如后厨做饭）。
- **View（视图）**：与用户交互，展示数据（比如服务员接待）。
- **Controller（控制器）**：接收请求，调用模型，返回视图（比如前厅调度）。

**2. Spring MVC 是什么**
- **定义**：Spring MVC 是 Spring 框架中的一个 **Web 模块**，全称 **Spring Web MVC**。
- **核心作用**：它把 MVC 模式落地到 Web 开发中。作为“前端调度中心”（DispatcherServlet），负责把浏览器的 HTTP 请求准确分发到你的 Java 方法上，并处理参数和返回值。
- **核心**：学 Spring MVC 就是学三件事：**建立连接**（路由映射）、**请求**（获取参数）、**响应**（返回数据）。


### 概念辨析与关系（重点）

**1. Spring（大工厂）**
- **定位**：整个 Spring 生态的**基石框架**。
- **作用**：提供核心的“容器”能力（如 IoC 依赖注入、AOP 面向切面编程、事务管理等）。
- **特点**：非常底层且庞大，早期使用它需要手动配置大量文件。
**2. Spring MVC（Web 办事员）**
- **定位**：Spring 框架里专门负责 **Web 层开发**的模块。
- **作用**：处理 HTTP 请求、URL 路由映射（如你之前用的 `@RestController`、`@RequestMapping`）、参数绑定、返回 JSON 数据等。
- **缺点**：早期纯 Spring MVC 开发时，需要手动配置 Tomcat、写大量 XML 文件，极其繁琐。
**. Spring Web MVC（模块别名）**
- **定位**：**Spring MVC 的全称**（Spring Web MVC）。
- **区别**：两者指代同一个东西。在代码依赖中，它的包名和依赖名通常叫 `spring-webmvc`。

- **全称**：Spring Web MVC（因为它既是 Web 模块，又实现了 MVC 模式）。
- **简称**：Spring MVC（大家为了省事，把“Web”省略了）。
- 就像“北京大学”和“北大”的关系，指的都是同一个学校。
##### 3.spring web和spring web mvc区别

| **<br><br>模块<br><br>** | **<br><br>定位<br><br>** | **<br><br>日常会直接接触到的东西<br><br>**                                               |
| ---------------------- | ---------------------- | ----------------------------------------------------------------------------- |
| **spring-web**​        | **底层基础设施**​            | `RestTemplate`（发 HTTP 请求调别人的接口）、`MultipartFile`（文件上传）、`WebUtils` 等工具类。        |
| **spring-webmvc**​     | **Web MVC 框架**​        | `@RestController`、`@RequestMapping`、`DispatcherServlet` 等，用来**接收并处理**别人发来的请求。 |
**`spring-webmvc` 的运行强依赖于 `spring-web`**。

当引入 `spring-webmvc` 时，Maven 会自动把 `spring-web` 也拉进来。所以你不需要手动写 `spring-web` 的依赖，它就在后台默默提供着最底层的 Web 支持。

- **Spring Web**（`spring-web`）= 提供 Web 开发需要的**基础工具和底层能力**。
- **Spring Web MVC**（`spring-webmvc`）= 在基础能力之上，实现了 **MVC 模式**，让你能优雅地写接口。

**4. Spring Boot（一站式管家）**
- **定位**：基于 Spring 的**快速开发脚手架**。
- **作用**：**自动配置**（Auto-configuration）和**起步依赖**（Starters）。它帮你把 Spring 和 Spring MVC 运行所需的环境（如内嵌 Tomcat 服务器、默认配置）全部打包搞定。
- **关键结论**：**Spring Boot 不是替代 Spring MVC，而是封装和增强了它**。当你用 Spring Boot 创建项目并勾选 `Spring Web` 时，它其实引入了 `spring-boot-starter-web`，而这个依赖里面**已经包含了 Spring MVC 和 内嵌 Tomcat**。
# 2学习Spring MVC
### 2.1建立连接
#### 2.1.1注解
**以 `@` 开头、帮后端和前端建立连接、表现 URL 路径、区分 GET/POST**。

#### Spring MVC 的注解大体可以分成 **4 个阵营**

##### 一、声明身份的注解（告诉 Spring“我是干什么的”）

这类注解用在**类上**，解决的是“这个类归不归 Spring MVC 管”的问题。

|注解|作用|说明|
|---|---|---|
|`@Controller`|标记这是一个控制器|方法返回值默认被当成**视图名**（如返回 `"home"` 去找 `home.html`）|
|`@RestController`|标记这是一个 REST 控制器|等价于 `@Controller + @ResponseBody`，方法返回值**直接变成数据**（JSON/字符串）返回给前端|
|`@ResponseBody`|标记返回值直接写入响应体|可以放在**类上**（整个类都生效）或**方法上**（只对该方法生效）|

写前后端分离的项目，99% 的情况直接用 `@RestController` 就完事了。
##### 二、建立连接 & 映射路径的注解（URL 路由）

这类注解解决的是“**前端的请求到底交给哪个方法处理**”的问题。你提到的“有些方法注解也可以给类用”，主要就是指下面这个：

##### @RequestMapping —— 万金油

它**既可以用在类上，也可以用在方法上**：

```
@RestController
@RequestMapping("/user")          // 类上：统一前缀
public class UserController {

    @RequestMapping("/list")       // 方法上：子路径
    public String list() {
        return "用户列表";
    }
}
```

实际访问路径：`/user/list`

**它还能指定请求方式：**
```
@RequestMapping(value = "/login", method = RequestMethod.POST)
public String login() { ... }
```

#### 快捷注解 —— 日常开发的首选

因为 `@RequestMapping` 写起来太长了，Spring MVC 提供了 4 个“快捷方式”，**它们只能用在方法上**（不能用在类上）：

|注解|等价写法|用途|
|---|---|---|
|`@GetMapping`|`@RequestMapping(method = GET)`|查询数据|
|`@PostMapping`|`@RequestMapping(method = POST)`|提交/创建数据|
|`@PutMapping`|`@RequestMapping(method = PUT)`|更新数据|
|`@DeleteMapping`|`@RequestMapping(method = DELETE)`|删除数据|

```
@GetMapping("/hello")      // 只接收 GET 请求
public String hello() { ... }

@PostMapping("/hello")     // 只接收 POST 请求，和上面互不冲突
public String helloPost() { ... }
```

##### 三、参数绑定注解（从请求中获取数据）
路径映射好了，接下来就是**从前端传来的请求里拿数据**。Spring MVC 提供了一组注解来干这件事：
##### 1. @RequestParam —— 获取 URL 查询参数
```
// 前端访问：/hello?name=张三&age=18
@GetMapping("/hello")
public String hello(@RequestParam String name, 
                    @RequestParam int age) {
    return name + "今年" + age + "岁";
}
```

> 如果参数名和前端传的 key 不一致，可以指定：`@RequestParam("user_name") String name`

##### 2. @PathVariable —— 获取 URL 路径中的参数
```
// 前端访问：/user/1001
@GetMapping("/user/{id}")
public String getUser(@PathVariable Long id) {
    return "用户ID是：" + id;
}
```
##### 3. @RequestBody —— 获取请求体中的 JSON 数据

```
// 前端发送一个 JSON：{"name":"张三", "age":18}
@PostMapping("/user")
public String createUser(@RequestBody User user) {
    return "创建用户：" + user.getName();
}
```

> 这是前后端分离项目里**最常用**的注解之一，前端通过 POST 发送 JSON，后端用对象直接接收。

##### 4. 其他获取请求信息的注解

|注解|作用|
|---|---|
|`@RequestHeader`|获取请求头中的某个值|
|`@CookieValue`|获取 Cookie 中的某个值|
|`@RequestPart`|获取文件上传（MultipartFile）|

##### 四、其他常用注解

|注解|作用|
|---|---|
|`@RequestParam(defaultValue = "1")`|给参数设默认值|
|`@RequestParam(required = false)`|标记参数不是必须的|
|`@Valid / @Validated`|对接收到的参数做数据校验（如不能为空、长度限制等）|

##### 全景串联：一个完整的 Controller
把上面这些串在一起，一个典型的 Spring MVC 控制器长这样：
```
@RestController
@RequestMapping("/api/users")
public class UserController {

    // GET /api/users/1001
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id) {
        return "查询用户：" + id;
    }

    // POST /api/users
    @PostMapping
    public String createUser(@RequestBody User user) {
        return "创建用户：" + user.getName();
    }

    // GET /api/users/list?page=1&size=10
    @GetMapping("/list")
    public String listUsers(@RequestParam(defaultValue = "1") int page,
                            @RequestParam(defaultValue = "10") int size) {
        return "第" + page + "页，每页" + size + "条";
    }
}
```

---

##### 一句话总结

| 你想做的事                  | 用哪个注解                               |
| ---------------------- | ----------------------------------- |
| 告诉 Spring 这是个控制器，返回数据  | `@RestController`                   |
| 指定访问路径                 | `@RequestMapping` / `@GetMapping` 等 |
| 区分 GET 还是 POST         | `@GetMapping` / `@PostMapping`      |
| 从 URL `?key=value` 拿数据 | `@RequestParam`                     |
| 从 URL `/path/{id}` 拿数据 | `@PathVariable`                     |
| 从请求体拿 JSON             | `@RequestBody`                      |
|                        |                                     |
|                        |                                     |
**请求进来时、处理中、响应回去时**，注解都在起作用：
```
浏览器发请求
    ↓
【连接阶段】DispatcherServlet 根据 @RequestMapping 找到对应的方法
    ↓
【请求阶段】Spring MVC 根据 @RequestParam / @RequestBody 等注解，从请求中提取数据，填入方法参数
    ↓
你的方法执行业务逻辑
    ↓
【响应阶段】Spring MVC 根据 @ResponseBody（或 @RestController）把返回值序列化成 JSON，写入响应体
    ↓
浏览器收到响应
```
### 2.2请求
### 2.2.1r5和r9的区别

```
@RequestMapping("/r5")

    public String r5(Person person){

        return "接受到参数: "+ person;

    }
    
@RequestMapping("/r9")

    public String r9(@RequestBody Person person){

        return "接受到参数: "+ person;

    }
```

|**<br><br>写法<br><br>**|**<br><br>数据从哪来<br><br>**|**<br><br>前端怎么传<br><br>**|
|---|---|---|
|`r5(Person person)` 无注解|从 **URL 查询参数**​ 或 **表单字段**​ 中一个个匹配|`?name=张三&age=18` 或 form 表单提交|
|`r9(@RequestBody Person person)` 有注解|从 **HTTP 请求体（Request Body）**​ 里读一整个 JSON|请求体里发 `{"name":"张三","age":18}`|
#### 详细介绍区别
##### r5：没有 @RequestBody（表单/查询参数绑定）

```
@RequestMapping("/r5")
public String r5(Person person) {
    return "接受到参数: " + person;
}
```

**Spring MVC 做的事：**
1. 看到方法参数 `Person person` 没有注解
2. 它会去 **URL 的查询参数**​ 或 **表单数据**​ 里找和 Person 类属性名一样的字段
3. 找到后通过 setter 方法一个个 set 进去
**前端必须这样传：**
```
GET /r5?name=张三&age=18
```
或者
```
POST /r5 （Content-Type: application/x-www-form-urlencoded）
Body: name=张三&age=18
```

**Person 类需要满足：**
```
public class Person {
    private String name;
    private Integer age;
    
    // 必须有无参构造函数
    // 必须有 setter 方法（Spring 通过 setXxx() 赋值）
    public void setName(String name) { this.name = name; }
    public void setAge(Integer age) { this.age = age; }
    
    // getter 可选，但建议有
}
```

> ⚠️ 如果前端发的是 JSON（`Content-Type: application/json`），但后端没写 `@RequestBody`，Spring MVC **不会**自动把 JSON 转成 Person 对象，person 里的属性会是 `null`。
##### r9：有 @RequestBody（JSON 请求体绑定）
```
@RequestMapping("/r9")
public String r9(@RequestBody Person person) {
    return "接受到参数: " + person;
}
```

**Spring MVC 做的事：**

1. 看到 `@RequestBody`
2. 读取整个 HTTP 请求体里的字符串
3. 检查请求头 `Content-Type` 是不是 `application/json`
4. 用 Jackson 把 JSON 字符串**整体反序列化**成一个 Person 对象

**前端必须这样传：**
```
POST /r9
Content-Type: application/json
Body:
{
    "name": "张三",
    "age": 18
}
```

**Person 类需要满足：**
```
public class Person {
    private String name;
    private Integer age;
    
    // 必须有无参构造函数
    // 必须有 setter（或者类上有 @Data 注解，如 Lombok 的 @Data）
}
```

#### 对比总结

|对比项|r5（无注解）|r9（有 @RequestBody）|
|---|---|---|
|**数据来源**​|URL 参数 / 表单字段|请求体（Request Body）|
|**Content-Type**​|不需要特别指定 / `x-www-form-urlencoded`|必须是 `application/json`|
|**数据格式**​|`name=张三&age=18`|`{"name":"张三","age":18}`|
|**适合场景**​|简单参数、传统表单提交|前后端分离、复杂对象、RESTful API|
|**前端用 Postman 怎么测**​|Params 标签里加 key-value|Body → raw → JSON 里写对象|
|**如果前端传 JSON 但后端没写 @RequestBody**​|person 属性全为 null|—|
|**如果前端传表单但后端写了 @RequestBody**​|—|报 400 错误（无法反序列化）|

#### 常见坑
代码里写了 `reurn "接受到参数: "+ person;`，如果 Person 类**没有重写 `toString()` 方法**，浏览器上看到的会是：

```
接受到参数: com.bit.springmvc.Person@1a2b3c4d
```

建议在 Person 类里加上 `toString()`：
```
@Override
public String toString() {
    return "Person{name='" + name + "', age=" + age + "}";
}
```

这样返回的就是：
```
接受到参数: Person{name='张三', age=18}
```
- **没注解**​ → 数据来自 **URL 或表单**，一个个字段往对象里塞
- **有 `@RequestBody`**​ → 数据来自 **请求体**，整个 JSON 一次性转成对象

​ 用 JSON 放在请求体里，数据确实**不会出现在 URL 中**。这也是为什么登录、注册、提交敏感信息时，几乎都用 JSON 或表单放在请求体里传，而不是拼在 URL 后面。

URL 只承载**路径和查询参数**（如 `/api/user?name=张三`），而 JSON 是放在**请求体**里的，两者完全分开。所以浏览器地址栏、书签、聊天软件里分享的链接，都看不到请求体里的内容。

#### 但要注意：不在 URL ≠ 安全

|误区|真相|
|---|---|
|"放 Body 里别人就看不见了"|❌ 如果用的是 **HTTP**（不是 HTTPS），用抓包工具（如 Fiddler、Wireshark）可以轻松看到请求体里的 JSON，**完全是明文的**​|
|"密码放 JSON Body 里就安全了"|❌ 没加密的话，中间人一样能截获|
|"URL 里看不到 = 加密了"|❌ 只是不在地址栏显示而已，传输层该明文还是明文|

**真正的安全靠的是 HTTPS**（在传输过程中把整个请求——包括 URL 和请求体——都加密了）。

###  2.2.2 Spring MVC 接收参数的所有常见方式
#### 1. 普通参数连接（从 URL 查询参数获取）
**特点**：直接在方法里写参数名，Spring MVC 自动从 `?key=value` 里取值。也可以配合 `@RequestParam` 做限制。

|方法|访问示例|说明|
|---|---|---|
|`r1(String s1)`|`/request/r1?s1=hello`|直接写参数名，自动绑定|
|`r2(Integer age)`|`/request/r2?age=18`|基本类型包装类，没传就是 null|
|`r3(Boolean flag)`|`/request/r3?flag=true`|布尔值，支持 true/false|
|`r4(String name, Integer age)`|`/request/r4?name=张三&age=18`|多个普通参数，按顺序绑定|
|`r6(@RequestParam(...))`|`/request/r6?sa=hello`|用 `@RequestParam` 指定别名、设非必传|

> 💡 `r6` 里的 `value = "sa"` 意思是：URL 里写 `sa`，方法参数名用 `name`；`required = false` 表示可以不传这个参数。
#### 2. 类作为对象参数（用 Java 对象接收）
**特点**：当参数较多时，直接用一个类（如 `Person`）来接收，Spring MVC 会自动把 URL 参数名和类的属性名匹配，通过 setter 方法注入。

|方法|访问示例| 说明                          |
| ------------------- | ---------------------------- | --------------------------- |
|`r5(Person person)`|`/request/r5?name=张三&age=18`| 无注解，框架自动把参数填充到 Person 对象的属性 

> ⚠️ 前提：`Person` 类必须有**无参构造函数**和**setter 方法**，否则无法赋值（属性为 null）。

#### 3. 数据结构（数组、List）
**特点**：前端可以一次传多个同名参数，后端用数组或集合接收。

|方法|访问示例|说明|
|---|---|---|
|`r7(String[] array)`|`/request/r7?array=aaa&array=bbb`|用数组接收，同名参数自动变成数组元素|
|`r8(@RequestParam List<String> list)`|`/request/r8?list=aaa&list=bbb`|用 List 接收，必须加 `@RequestParam` 告诉框架这是请求参数|

> 💡 浏览器里多次写同一个 key（`?array=1&array=2`），后端就能收到多个值。

#### 4. JSON 作为参数连接（请求体传参）
**特点**：前端通过 POST 发送 JSON 字符串放在请求体里，后端用 `@RequestBody` 接收并自动转成 Java 对象。

|方法|访问方式|说明|
|---|---|---|
|`r9(@RequestBody Person person)`|`POST /request/r9`  <br>Body: `{"name":"张三","age":18}`|`@RequestBody` 把 JSON 反序列化成 Person 对象|

> ⚠️ 必须设置请求头 `Content-Type: application/json`，否则报 415 错误。

#### 5. 路径参数 & 文件上传

##### 路径参数（@PathVariable）
**特点**：把参数直接写在 URL 路径里，而不是 `?` 后面，RESTful 风格。

|方法|访问示例|说明|
|---|---|---|
|`r9(@PathVariable Integer articleId)`|`/request/1`|路径里的 `1` 被绑定到 `articleId`|
|`r10(@PathVariable Integer articleId, @PathVariable String type)`|`/request/blog/1`|多个路径参数，type=blog，articleId=1|

##### 文件上传（MultipartFile）
**特点**：接收前端上传的文件，可以获取文件名、类型，并保存到服务器。

|方法|访问方式|说明|
|---|---|---|
|`r11(MultipartFile file)`|`POST /request/r11`  <br>Body: form-data，key=file，选一个文件|用 `MultipartFile` 接收，调用 `transferTo()` 保存文件|

> ⚠️ 文件上传需要在配置里设置最大文件大小，且前端 form 的 `enctype` 必须是 `multipart/form-data`。
#### 所有参数接收方式

|你想接收的数据来源|用什么|示例|
|---|---|---|
|URL `?key=value`|直接写参数 或 `@RequestParam`|`/r1?s1=hi`|
|多个同名参数|数组 或 `@RequestParam List`|`/r7?array=1&array=2`|
|对象属性（URL 参数）|写 Java 类作为参数|`/r5?name=xx&age=18`|
|JSON 请求体|`@RequestBody` + 对象|POST Body: `{"name":"xx"}`|
|URL 路径里|`@PathVariable`|`/request/1`|
|上传的文件|`MultipartFile`|form-data 上传|
### 2.2.3cookie与session（网络原理提到过）

#### 一、起点：HTTP 是无状态的

整个 Cookie 和 Session 的故事，起点只有一个：**HTTP 协议本身没有记忆**。引入一种机制，让无状态的 HTTP 具备"记住用户"的能力。这就是 Cookie 和 Session 出现的根本原因。
#### 二、Cookie：浏览器端的"会员卡"
Cookie 的本质，是**浏览器提供的一种本地存储数据的机制**，它以键值对的形式存在。

但它不是普通的本地存储，它是 **HTTP 协议体系内的一部分**，具体体现在它和 HTTP 头部（Header）深度绑定：

- **服务器想给浏览器存数据**：通过响应头 `Set-Cookie` 告诉浏览器"帮我存一下"。
- **浏览器后续访问时**：通过请求头 `Cookie` 自动把之前存的数据带回去。
比如服务器响应时带一句：
```
Set-Cookie: sessionid=abc123
```

浏览器收到后，就把 `sessionid=abc123` 存在本地硬盘（或内存）里。下次再访问同一个网站时，浏览器会**自动**在请求头里加上：
```
Cookie: sessionid=abc123
```
Cookie 本质上就是一种特殊的 Header。它和其他 Header 的区别在于：它是从本地硬盘读出来、自动附加到请求里的，具备"记忆功能"。
#### 三、Session：服务器端的"档案袋"
Cookie 解决了"浏览器能带数据回来"的问题，但它有一个致命缺陷：**数据存在客户端，不安全，而且容量有限**。
你不能把用户的密码、权限信息直接存在 Cookie 里发给浏览器。所以需要 Session。
Session 是**服务器端**的概念。你可以把它理解为一个 Map（键值对集合），存在服务器的内存（或 Redis、数据库）里。每个用户来访问时，服务器为这个用户单独建一个"档案"，里面存着这个用户的状态数据——比如用户 ID、昵称、登录时间、权限等。
这个档案本身和 HTTP 请求没有关系，它就是服务器内存里的一段数据。
#### 四、SessionID：连接两端的桥梁
现在问题来了：服务器上有成千上万个 Session 档案，浏览器发请求过来时，服务器怎么知道"你对应的是哪一个档案"？

答案是：**SessionID**。

SessionID 是一个随机字符串，由服务器生成。它的角色就是一把"钥匙"：

1. 用户第一次登录，服务器验证密码成功后：
    - 在服务器端创建一个 Session，存好用户数据。
    - 生成一个 SessionID（比如 `abc123`），把这个 ID 和刚才的 Session 关联起来。
    - 通过响应头 `Set-Cookie: sessionid=abc123` 把这个 ID 交给浏览器。
2. 浏览器收到后，把 `sessionid=abc123` 存进 Cookie。
3. 以后每次浏览器访问这个网站，都会自动在请求头里带上 `Cookie: sessionid=abc123`。
4. 服务器收到请求，从 Cookie 里取出 `sessionid=abc123`，去自己的 Session 存储里一查，找到对应的用户数据，就知道"哦，是小明，已经登录了"。

**所以三者的关系可以浓缩为一句话：**

> Cookie 是浏览器存钥匙的地方，Session 是服务器存档案的地方，SessionID 是钥匙上的编号，用来在档案室里找到对应的档案。
#### 五、免重复登的完整过程结合你画的时序图，我把这个过程用文字再讲一遍：
##### 第一次登录（还没有 Cookie 和 Session）

```
浏览器                              服务器
  |  POST /login                     |
  |  Body: username=xxx, pwd=xxx     |
  |--------------------------------->|
  |                                  |
  |                          1. 验证账号密码
  |                          2. 验证通过
  |                          3. 创建 Session：
  |                             SessionID = "abc123"
  |                             Session数据 = { userId: 888, name: "小明" }
  |                          4. 把 SessionID 存起来
  |                                  |
  |  HTTP 200 OK                     |
  |  Set-Cookie: sessionid=abc123    | ← 告诉浏览器存好这把钥匙
  |<---------------------------------|
  |
  |  浏览器把 sessionid=abc123 写入 Cookie
```

##### 后续访问（浏览器自动带 Cookie）

```
浏览器                              服务器
  |  GET /profile                    |
  |  Cookie: sessionid=abc123        | ← 浏览器自动带上
  |--------------------------------->|
  |                                  |
  |                          1. 从请求头提取 sessionid
  |                          2. 用 "abc123" 去 Session 存储里查
  |                          3. 查到了！用户是小明，已登录
  |                                  |
  |  HTTP 200 OK                     |
  |  Body: { name: "小明" }          |
  |<---------------------------------|
```

这就是为什么你登录一次之后，刷新页面、点链接、甚至关掉浏览器再打开（如果 Cookie 没过期），都不需要重新输入密码——因为每次请求浏览器都自动把 SessionID 带过去了，服务器一查就知道你是谁。

|概念|存在哪|是什么|作用|
|---|---|---|---|
|**Cookie**​|浏览器（客户端）|一小段键值对数据|在每次请求时自动把数据带给服务器|
|**Session**​|服务器（内存/Redis）|服务器为某个用户保存的状态数据|让服务器"记住"用户是谁、登录了没有|
|**SessionID**​|服务器生成，通过 Cookie 传给浏览器|一个随机字符串，Session 的唯一索引|服务器用它查找对应的 Session 数据|

**一句话串起来：**
> HTTP 无状态 → 需要记住用户 → 服务器用 Session 存用户状态 → 服务器用 SessionID 标记这个 Session → 通过 Set-Cookie 把 SessionID 交给浏览器 → 浏览器每次请求自动带 Cookie → 服务器用 SessionID 找到 Session → 识别用户身份 → 实现免重复登录。

这就是 Cookie 和 Session 的完整逻辑。你之前的分析方向完全正确，我上面的讲解只是用更口语化的方式重新走了一遍这个流程。
### 2.3响应
