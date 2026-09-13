
# 8. MyBatis 操作数据库(入门)

**本节目标**：掌握 MyBatis 基本操作，实现数据库增删改查。
**前言**

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。它免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

**JDBC 操作示例回顾**

(简要回顾：加载驱动 -> 获取连接 -> 创建 Statement -> 执行 SQL -> 处理结果 -> 关闭资源。JDBC 代码繁琐，MyBatis 解决了这些痛点。)

## 1. 什么是 MyBatis?

- 半自动的 ORM（对象关系映射）框架。
- 将 SQL 语句与 Java 代码分离（XML 或注解）。
- 屏蔽了底层 JDBC 的繁琐操作。
## 2. MyBatis入门

### 2.1 准备工作

#### 2.1.1 创建工程

在 Spring Boot 中引入依赖：

```java
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### 2.1.2 数据准备

创建数据库表（如 `user_info`）并插入测试数据。
#### 2.1.3 配置数据库连接字符串

在 `application.yml` 中配置：

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test_db?characterEncoding=utf8&useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 2.2 写持久层代码

创建 Mapper 接口：

```java
@Mapper
public interface UserMapper {
    // 注解方式写 SQL
    @Select("SELECT * FROM user_info WHERE id = #{id}")
    UserInfo selectById(Integer id);
}
```
### 2.3 单元测试

在测试类中注入 Mapper 并调用：

```java
@Autowired
private UserMapper userMapper;

@Test
void testSelect() {
    System.out.println(userMapper.selectById(1));
}
```
## 3. MyBatis的基础操作

### 3.1 打印日志

配置日志级别，查看 MyBatis 执行的 SQL：

```java
logging:
  level:
    org.example.springmybatis.mapper: DEBUG
```

### 3.2 参数传递

- 单个参数：直接使用 `#{参数名}`。
- 多个参数：必须使用 `@Param("参数名")` 注解绑定。

```java
@Select("SELECT * FROM user_info WHERE username = #{name} AND age = #{age}")
UserInfo selectByNameAndAge(@Param("name") String username, @Param("age") Integer age);
```
### 3.3 增(Insert)

```java
@Insert("INSERT INTO user_info(username, age) VALUES(#{username}, #{age})")
Integer insertUser(UserInfo user);
```
### 3.4 删(Delete)

```java
@Delete("DELETE FROM user_info WHERE id = #{id}")
Integer deleteById(Integer id);
```
### 3.5 改(Update)
```java
@Update("UPDATE user_info SET username = #{username} WHERE id = #{id}")
Integer updateUser(UserInfo user);
```
### 3.6 查(Select)

#### 3.6.1 起别名

当数据库字段名（如 `user_name`）与 Java 对象属性名（如 `userName`）不一致时，在 SQL 中起别名：

```java
SELECT user_name AS userName FROM user_info
```

#### 3.6.2 结果映射

使用 `@Results` 和 `@Result` 注解手动映射：

```java
@Results({
    @Result(column = "user_name", property = "userName")
})
@Select("SELECT * FROM user_info")
List<UserInfo> selectAll();
```

#### 3.6.3 开启驼峰命名(推荐)

在 `application.yml` 中开启全局自动驼峰映射，免去手动起别名或映射的麻烦：

```java
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

## 4. MyBatis XML配置文件

### 4.1 配置连接字符串和MyBatis

(上述数据库连接和 MyBatis 配置也可统一放在 YAML 中)

### 4.2 写持久层代码

#### 4.2.1 添加 mapper 接口

```java
@Mapper
public interface UserXmlMapper {
    // 方法定义，不带 SQL 注解
    User selectById(Integer id);
}
```

#### 4.2.2 添加 UserInfoXMLMapper.xml

在 `resources/mapper` 目录下创建 XML 文件：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.example.springmybatis.mapper.UserXmlMapper">
    <select id="selectById" resultType="org.example.springmybatis.model.UserInfo">
        SELECT * FROM user_info WHERE id = #{id}
    </select>
</mapper>
```

并在 YAML 中指定 XML 路径：

```java
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

#### 4.2.3 单元测试

同上文，调用接口方法测试。

### 4.3 增删改查操作

#### 4.3.1 增(Insert)

```java
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user_info(username, age) VALUES(#{username}, #{age})
</insert>
```

(注：`useGeneratedKeys` 用于返回自增主键)

#### 4.3.2 删(Delete)

```java
<delete id="deleteById">
    DELETE FROM user_info WHERE id = #{id}
</delete>
```

#### 4.3.3 改(Update)

```java
<update id="updateUser">
    UPDATE user_info SET username = #{username} WHERE id = #{id}
</update>
```

#### 4.3.4 查(Select)

```java
<select id="selectAll" resultType="org.example.springmybatis.model.UserInfo">
    SELECT * FROM user_info
</select>
```

## 5. 其他查询操作

### 5.1 多表查询

#### 5.1.1 准备工作

准备关联表（如 `order` 表关联 `user` 表）。

#### 5.1.2 数据查询

使用 `resultMap` 中的 `association`（一对一/多对一）或 `collection`（一对多）进行嵌套映射。

```java
<resultMap id="UserOrderMap" type="User">
    <id property="id" column="id"/>
    <result property="userName" column="user_name"/>
    <collection property="orders" ofType="Order">
        <id property="orderId" column="order_id"/>
    </collection>
</resultMap>
```
### 5.2 #{} 和 ${}

#### 5.2.1 #{} 和 ${} 使用

- `#{}`：预编译处理，替换为 `?`，自动加引号。
- `${}`：直接字符串替换，原样拼接。

#### 5.2.2 #{} 和 ${} 区别

|对比项|#{}|${}|
|---|---|---|
|处理方式|预编译(?)占位符|直接字符串替换|
|安全性|✅ 防止 SQL 注入|❌ 存在 SQL 注入风险|
|引号处理|自动添加|需手动添加|
|适用场景|动态值(WHERE条件/INSERT值)|动态 SQL 片段(表名/字段名/排序)|

> **口诀：值用井号(#)，结构用刀($)**​

### 5.3 排序功能

排序字段和规则不能用 `#{}`（会变成字符串带引号报错），必须用 `${}`，**但必须做后端参数白名单校验防注入**：

```java
SELECT * FROM user_info ORDER BY ${sortField} ${sortOrder}
```

### 5.4 like 查询

**错误写法**：`LIKE '%${keyword}%'`（有注入风险）。

**正确写法**：使用 MySQL 的 `CONCAT()` 函数配合 `#{}`：

```java
SELECT * FROM user_info WHERE username LIKE CONCAT('%', #{keyword}, '%')
```

## 6. 数据库连接池

### 6.1 介绍

数据库连接池负责分配、管理和释放数据库连接。Spring Boot 默认使用 **HikariCP**（性能极高）。

### 6.2 使用

在 `application.yml` 中配置 Hikari 参数：

```java
spring:
  datasource:
    hikari:
      maximum-pool-size: 10      # 最大连接数
      minimum-idle: 5             # 最小空闲连接
      idle-timeout: 30000         # 空闲连接超时时间
```


## 7. 总结

### 7.1 MySQL 开发企业规范

- 表名、字段名小写，下划线分隔（`user_info`）。
- 实体类属性名使用驼峰命名（`userName`）。
- 开启 MyBatis 驼峰自动映射。
- 优先使用 `#{}` 防注入。

### 7.2 #{} 和 ${} 区别

(回顾：#{} 预编译安全用于值，${} 字符串替换用于结构且需校验)

# 9. MyBatis 操作数据库(进阶)

**本节目标**：掌握 MyBatis 动态 SQL 标签，灵活组装复杂 SQL。
## 1. 动态SQL

### 1.1 <if> 标签

单条件判断，满足条件才拼接 SQL：

```
<select id="findUser" resultType="User">
    SELECT * FROM user_info WHERE 1=1
    <if test="username != null">
        AND username = #{username}
    </if>
</select>
```

### 1.2 <trim> 标签

自定义去除前后缀（万能标签）：

```
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    <if test="username != null">AND username = #{username}</if>
</trim>
```

### 1.3 <where> 标签

智能处理 WHERE 关键字，自动去除多余的 AND/OR：

```
<where>
    <if test="username != null">AND username = #{username}</if>
    <if test="age != null">AND age = #{age}</if>
</where>
```

### 1.4 <set> 标签

动态更新，自动添加 SET 关键字并去除末尾多余的逗号：

```
<update id="updateUser">
    UPDATE user_info
    <set>
        <if test="username != null">username = #{username},</if>
        <if test="age != null">age = #{age},</if>
    </set>
    WHERE id = #{id}
</update>
```

### 1.5 <foreach> 标签

遍历集合，用于批量操作或 IN 查询：

```
<delete id="batchDelete">
    DELETE FROM user_info WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```

### 1.6 <include> 标签

配合 `<sql>` 抽取复用公共 SQL 片段：

```
<sql id="Base_Column_List">id, username, age</sql>

<select id="selectAll" resultType="User">
    SELECT <include refid="Base_Column_List"/> FROM user_info
</select>
```

