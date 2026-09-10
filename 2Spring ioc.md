
# Spring IoC & DI 
## 一、IoC（控制反转）核心概念
### 1. 什么是 IoC？
**IoC（Inversion of Control，控制反转）**：把对象的创建权、管理权从程序员手里反转给了 Spring 容器。

|传统方式|IoC 方式|
|---|---|
|程序员 `new` 对象|Spring 用反射 `new` 对象|
|程序员手动 `set` 依赖|Spring 自动把依赖"注入"进来|
|对象生命周期由程序员管|对象生命周期由 Spring 容器管|

### 2. 什么是容器？
Spring 容器就是一个 **Map（键值对）**，key 是 Bean 的名字，value 是 Bean 对象。
```java
容器启动后：
Map = {
    "userController" → UserController对象,
    "userService"    → UserService对象,
    "userRepository" → UserRepository对象
}
```
### 3. 什么是 DI？
**DI（Dependency Injection，依赖注入）**：Spring 容器在创建对象时，自动把它依赖的其他对象"塞"进来。
> IoC 是思想，DI 是实现方式。
## 二、IoC 详解——Bean 的注册
### 1. 五大类注解（自动注册 Bean)

|注解|层级|作用|特殊功能|
|---|---|---|---|
|`@Component`|通用层|基础注解，所有衍生注解的"爸爸"|无|
|`@Controller`|控制层|接收 HTTP 请求|Spring MVC 路由支持|
|`@Service`|业务层|写业务逻辑|无|
|`@Repository`|数据层|操作数据库|自动异常翻译|
|`@Configuration`|配置层|配合 `@Bean` 手动声明|CGLIB 代理保证 `@Bean` 单例|

**底层真相**：`@Controller`、`@Service`、`@Repository` 的源码里都包含了 `@Component`。它们本质上是 `@Component` 的"别名"，Spring 扫描到它们时做的事情完全一样——把类实例化放进容器。
### 2. 方法注解 `@Bean`（手动注册 Bean）
用于**第三方库的类**（你改不了源码，不能在类上贴注解）：
```java
@Configuration
public class BeanConfig {
    
    @Bean              // Bean 名字 = 方法名 "userInfo"
    public UserInfo userInfo() {
        return new UserInfo("zhangsan", 16);
    }
    
    @Bean              // Bean 名字 = 方法名 "userInfo2"
    public UserInfo userInfo2() {
        return new UserInfo("suki", 18);
    }
}
```
### 3. ⚠️ 经典踩坑：重复注册导致冲突
**错误写法**（两种注册方式混用）：
```java
// 方式1：类上贴 @Component → 容器里有一个 Bean，名字是 userInfo
@Component
public class UserInfo { ... }

// 方式2：@Bean 方法 → 容器里又有一个 Bean，名字也是 userInfo
@Configuration
public class BeanConfig {
    @Bean
    public UserInfo userInfo() { return new UserInfo(); }
}
```
**结果**：容器里有两个同名同类型的 Bean → 注入时报 `NoUniqueBeanDefinitionException`。
**正确做法**：二选一，不要混用。自己写的类用 `@Component`，第三方类用 `@Bean`。
### 4. Bean 的命名规则

|注册方式|默认 Bean 名|自定义方式|
|---|---|---|
|类注解（`@Component` 等）|类名首字母小写：`UserInfo` → `userInfo`|`@Component("myUser")`|
|`@Bean` 方法|方法名：`userInfo()` → `userInfo`|`@Bean("myUser")`|

### 5. 包扫描路径
`@SpringBootApplication` 默认只扫描**启动类所在包及其子包**。
```
org.example.springiocdemo
├── SpringIocDemoApplication.java   ← 启动类
├── controller/                     ← ✅ 能扫到（子包）
├── service/                       ← ✅ 能扫到（子包）
└── utils/                          ← ✅ 能扫到（子包）
```
如果 Bean 放在启动类的**上级包或平级包**，需要手动指定：
```java
@SpringBootApplication
@ComponentScan("org.example")  // 指定扫描路径
public class SpringIocDemoApplication { ... }
```
## 三、DI 详解——依赖注入的三种方式
### 1. 构造方法注入（✅ 官方推荐）
```java
@Controller
public class UserController {
    private final UserService userService;
    
    // Spring 自动调用这个构造方法，把容器里的 UserService 传进来
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```
**优点**：
- 字段可以是 `final` → 不可变 → 线程安全
- 对象创建后依赖不可能为 null → 空安全
- 循环依赖时启动直接报错 → 逼你写出好设计
- 单元测试不需要 Spring 容器 → 直接 `new` 传参
**现代写法（Lombok 简化）**：
```java
@Controller
@RequiredArgsConstructor  // 自动生成包含所有 final 字段的构造方法
public class UserController {
    private final UserService userService;
    private final OrderService orderService;
    // 不需要手写构造方法
}
```
### 2. Setter 注入
```java
@Controller
public class UserController {
    private UserService userService;
    
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```
**优点**：可选依赖，对象创建后可以换实现。
**缺点**：对象状态可能不完整（忘了调 set 就 NPE），字段不能 final。
### 3. 字段注入（❌ 不推荐）
```java
@Controller
public class UserController {
    @Autowired
    private UserService userService;
}
```
**缺点**：
- 字段不能 final
- 必须依赖 Spring 容器（单元测试困难）
- 用反射暴力突破 private 封装
- 循环依赖被掩盖
### 4. 三种方式对比总表

|维度|构造方法注入|Setter 注入|字段注入|
|---|---|---|---|
|`final` 不可变|✅|❌|❌|
|对象创建后非空|✅|❌|❌|
|循环依赖早暴露|✅ 报错|❌ 掩盖|❌ 掩盖|
|单元测试友好|✅|✅|❌|
|官方推荐度|⭐⭐⭐⭐⭐|⭐⭐|⭐|


## 四、多个 Bean 冲突的完整解决方案
### 1. 问题场景
```java
@Configuration
public class BeanConfig {
    @Bean
    public UserInfo userInfo() { return new UserInfo("A", 1); }
    
    @Bean
    public UserInfo userInfo2() { return new UserInfo("B", 2); }
}
```

容器里有两个 `UserInfo` 类型的 Bean。此时注入：
```java
@Autowired
private UserInfo userInfo; // ❌ 报错！Spring 不知道选哪个
```
### 2. 解决方案一：`@Qualifier`（精确指定）
写在**注入点**，告诉 Spring 要哪个 Bean：
```java
@Autowired
@Qualifier("userInfo2")   // 指定 Bean 名字
private UserInfo myUser;
```
### 3. 解决方案二：`@Primary`（设默认）
写在**Bean 定义处**，设为全局默认：
```java
@Configuration
public class BeanConfig {
    @Bean
    @Primary                    // 大多数地方默认用这个
    public UserInfo userInfo() { return new UserInfo("A", 1); }
    
    @Bean
    public UserInfo userInfo2() { return new UserInfo("B", 2); }
}
```
这样所有没写 `@Qualifier` 的地方，自动注入 `userInfo()`。
### 4. 解决方案三：`@Resource`（Java 标准）
自带 `name` 属性，一行搞定：
```java
@Resource(name = "userInfo2")  // 不需要 @Qualifier
private UserInfo myUser;
```

### 5. 优先级总规则
```java
@Qualifier（局部精确） > @Primary（全局默认）> 变量名/参数名匹配 > 报错
```
## 五、`Autowired` vs `@Resource` 深度对比

|对比维度|`@Autowired`|`@Resource`|
|---|---|---|
|来源|Spring 框架|Java 标准（JSR-250）|
|默认匹配方式|**按类型**（byType）|**按名称**（byName）|
|指定名字|需要配合 `@Qualifier`|自带 `name` 属性|
|是否认 `@Primary`|✅ 认|❌ 不认|
|能否用于构造方法|✅ 能|❌ 不能|
|能否用于字段|✅ 能|✅ 能|
|能否用于 Setter|✅ 能|✅ 能|
|required 属性|✅ 有|❌ 无|
|type 属性|❌ 无|✅ 有|

## 六、Spring 选 Bean 完整决策流程
### `@Autowired` 的决策路径：
```
按类型找
  → 只有 1 个？→ 注入 ✅
  → 有多个？
      → 有 @Qualifier？→ 按它 → 注入 ✅
      → 有 @Primary？→ 按它 → 注入 ✅
      → 变量名匹配？→ 按匹配 → 注入 ✅
      → 都不行 → 报错 ❌
```

```
按类型查找 Bean
  │
  ├─ 没找到 → 抛异常
  │
  └─ 找到了
        │
        ├─ 只有 1 个 → 自动装配 ✅
        │
        └─ 有多个
              │
              ├─ 配置了 @Qualifier？→ 按 @Qualifier 查找 → 找到装配 / 没找到抛异常
              │
              └─ 没配置 @Qualifier
                    │
                    ├─ 有 @Primary？→ 按 @Primary 装配 ✅   ← 缺这条！
                    │
                    └─ 没有 @Primary
                          │
                          ├─ 按变量名/参数名匹配 → 找到装配 / 没找到抛异常
```
### `@Resource` 的决策路径：

```
有 name 属性？→ 按 name 精确找 → 找到就注入，找不到报错
没写 name？→ 用变量名匹配
  → 匹配上？→ 注入 ✅
  → 匹配不上？→ 退化为按类型
      → 只有 1 个？→ 注入 ✅
      → 多个？→ 报错 ❌
```


## 七、常见面试题
### Q1：什么是 IoC？什么是 DI？它们的关系是什么？
**A**：IoC 是把对象创建权交给容器；DI 是容器在运行时把依赖注入到对象中。DI 是 IoC 的具体实现方式。
### Q2：`@Component` 和 `@Bean` 的区别？
**A**：`@Component` 贴在类上，Spring 自动扫描注册；`@Bean` 写在方法上，手动声明返回值作为 Bean。前者用于自己写的类，后者用于第三方库的类。
### Q3：`@Autowired` 和 `@Resource` 的区别？
**A**：① 来源不同；② `@Autowired` 默认按类型，`@Resource` 默认按名称；③ `@Autowired` 支持 `@Primary`，`@Resource` 不支持；④ `@Autowired` 能用在构造方法上，`@Resource` 不能。
### Q4：构造方法注入为什么比字段注入好？
**A**：① 字段可以 final，不可变，线程安全；② 对象创建后依赖不为 null；③ 循环依赖早暴露；④ 单元测试不需要 Spring 容器。
### Q5：什么是循环依赖？Spring 怎么处理？
**A**：A 依赖 B，B 依赖 A。字段注入/Setter 注入能启动（三级缓存机制掩盖），构造方法注入直接报错。最佳方案是重构代码消除循环依赖。
### Q6：同一个接口有多个实现类，怎么注入指定的那个？
 **A**：三种方式：① `@Qualifier("beanName")` 精确指定；② `@Primary` 设默认；③ 变量名恰好等于 Bean 名。
## 八、一句话总结
> **你只管写类、贴注解（告诉 Spring "管我"），Spring 负责创建、存放、组装（IoC + DI）。遇到多个 Bean 冲突时，`@Qualifier` 精确指定，`@Primary` 设默认，`@Resource` 按名字一行搞定。日常开发用构造方法注入，别用字段注入。**

---

