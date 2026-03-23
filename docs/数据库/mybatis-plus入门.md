# mybatis plus入门

本文面向已经具备 MyBatis 使用经验的 Java 工程师，目标并非从零讲解持久层基础，而是帮助读者以较低的认知成本完成从 MyBatis 到 MyBatis-Plus 的入门过渡。全文将围绕实际开发中最常见的几个问题展开：为何要引入 MyBatis-Plus、其核心能力体现在哪里、在 Spring Boot 项目中应当如何快速落地，以及在已有 MyBatis 基础之上应当如何取舍与使用。

## 1. MyBatis 与 MyBatis-Plus 的关系

- MyBatis 是一套 SQL 映射框架，开发者通常通过 XML 或注解手写 SQL，因此拥有较高的灵活性与可控性。
- MyBatis-Plus 是构建在 MyBatis 之上的增强工具，并非替代品。它在保留 MyBatis 原有能力的前提下，提供了通用 CRUD、条件构造器、分页插件、代码生成等能力，用以减少重复性开发工作。
- 因而，从工程实践的角度看，`MyBatis 并不会因为 MyBatis-Plus 的引入而失去价值`。更准确地说，MyBatis-Plus 所解决的是大量样板化、重复性的单表操作问题，而复杂 SQL 的表达能力依然建立在 MyBatis 之上。

## 2. 你会立即感受到的收益

在具备 MyBatis 使用经验之后，初次接触 MyBatis-Plus 时，最直观的收益通常体现在以下几个方面：

- 对于单表 CRUD 场景，通常无需再编写 XML，即可完成大部分基础操作。
- 在条件查询方面，可以使用 `LambdaQueryWrapper` 组织查询条件，从而降低硬编码字段名带来的维护成本。
- 分页能力由官方插件直接提供，不再像传统 MyBatis 项目那样必须额外引入 PageHelper 一类的插件。
- 对逻辑删除、自动填充、乐观锁等企业项目中的常见能力，MyBatis-Plus 已经提供了较为成熟的支持。

如果用一句话概括其价值，那么可以理解为：`它并未改变 MyBatis 的本质，只是让常规数据库访问层的开发效率显著提升。`

## 3. 快速开始（Spring Boot）

### 3.1 引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
        <version>3.5.7</version>
    </dependency>

    <!-- 数据库驱动示例：MySQL -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

> Spring Boot 2.x 通常使用 `mybatis-plus-boot-starter`。

### 3.2 application.yml 最小配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 3.3 启动类扫描 Mapper

```java
@SpringBootApplication
@MapperScan("com.example.demo.mapper")
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

至此，一个最小可运行的接入环境便已经具备。接下来只需要定义实体、Mapper 和基础配置，即可开始使用 MyBatis-Plus 所提供的通用能力。

## 4. 第一个可运行 CRUD

### 4.1 建表（示例）

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(64) NOT NULL,
    age INT,
    email VARCHAR(128),
    create_time DATETIME,
    update_time DATETIME,
    deleted TINYINT DEFAULT 0
);
```

### 4.2 实体类

```java
@Data
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
    private String email;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    @TableLogic
    private Integer deleted;
}
```

### 4.3 Mapper 接口

```java
public interface UserMapper extends BaseMapper<User> {
}
```

在传统 MyBatis 中，开发者往往需要自行定义接口方法，并在 XML 中编写对应 SQL。而在 MyBatis-Plus 中，只要让 Mapper 继承 `BaseMapper<T>`，便可以直接获得一组常用的基础方法，例如：

- `insert`
- `deleteById`
- `updateById`
- `selectById`
- `selectList`

### 4.4 Service 层（推荐）

```java
public interface UserService extends IService<User> {
}
```

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

在服务层中，若继续使用 `IService + ServiceImpl` 这一套通用抽象，还可以进一步获得 `save`、`list`、`page` 等方法。对于常见业务系统中的基础数据维护模块而言，这种写法通常已经足够高效。

## 5. 条件查询：从 XML 转向 Wrapper

```java
LambdaQueryWrapper<User> qw = Wrappers.lambdaQuery();
qw.like(User::getName, "zhang")
  .ge(User::getAge, 18)
  .orderByDesc(User::getId);

List<User> users = userMapper.selectList(qw);
```

对于有 MyBatis 使用经验的开发者而言，MyBatis-Plus 最需要适应的，往往不是 CRUD，而是 Wrapper 的使用方式。它本质上是一种以 Java API 组织查询条件的方式，可用于替代相当一部分简单 XML 动态 SQL。

常见方法包括：

- `eq/ne/gt/ge/lt/le`
- `like/likeLeft/likeRight`
- `in/notIn`
- `between`
- `orderByAsc/orderByDesc`

相较于直接书写字符串字段名，`User::getName` 这一类 Lambda 写法的优势在于：当实体字段发生重构时，编译器即可帮助我们发现问题，从而降低因字段名变更而引发的隐性错误。

## 6. 分页查询（MP 官方插件）

### 6.1 分页插件配置

```java
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

### 6.2 使用分页

```java
Page<User> page = new Page<>(1, 10); // 第1页，每页10条
LambdaQueryWrapper<User> qw = Wrappers.lambdaQuery(User.class)
        .ge(User::getAge, 18)
        .orderByDesc(User::getId);

Page<User> result = userMapper.selectPage(page, qw);

long total = result.getTotal();
List<User> records = result.getRecords();
```

分页是 MyBatis-Plus 在实际项目中极高频的一项能力。与 MyBatis 生态中常见的第三方分页方案相比，官方分页插件的接入方式更加直接，且与 Wrapper、Service 体系结合得更自然。

## 7. 自动填充（createTime/updateTime）

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

在企业项目中，`createTime`、`updateTime`、`createUser`、`updateUser` 一类字段通常具有稳定的填充规则。若每次都在业务代码中手工赋值，不仅重复，而且容易遗漏。通过自动填充机制，可以将这类通用逻辑统一收敛到框架层处理。

## 8. 常见实践建议（有 MyBatis 经验的人重点看）

引入 MyBatis-Plus 之后，一个常见误区是试图将所有数据库访问都改写为 Wrapper 风格。这样的做法并不总是合理。更稳妥的实践通常是：

- 对于简单的单表增删改查，优先使用 MyBatis-Plus，以充分发挥其开发效率优势。
- 对于复杂联表、复杂统计、较强业务语义的 SQL，继续采用自定义 SQL（XML 或注解）往往更加清晰，也更便于后期排查与优化。
- 避免滥用 `selectList(null)` 这一类“全表查询”方式，在线上环境中应当对查询条件与返回规模保持明确控制。
- 在 Wrapper 中尽量避免前端参数直接参与字段拼接，尤其是排序字段，通常应采用白名单方式进行约束。
- 分页插件应确认只注册一次，以免出现插件冲突或行为异常的问题。

从团队协作的角度看，最值得建立的并不是“所有人都统一使用 MyBatis-Plus”，而是“哪些场景适合 MyBatis-Plus，哪些场景仍应坚持手写 SQL”的共识。

## 9. 学习路径（建议 1~2 天）

如果读者已经具备 MyBatis 基础，那么 MyBatis-Plus 的入门成本其实并不高。一个相对务实的学习路径如下：

1. 先跑通本文中的最小示例，理解实体、Mapper、分页插件之间的基本协作方式。
2. 从现有项目中挑选一个结构简单的 MyBatis Mapper，尝试改写为 `BaseMapper` + Wrapper 的形式。
3. 在此基础上补充逻辑删除、自动填充、乐观锁等常见能力，并通过少量单元测试验证行为是否符合预期。
4. 最后结合团队实际情况，沉淀出一份简单规范，明确“哪些 SQL 建议继续手写，哪些场景交由 MyBatis-Plus 承担”。

完成上述过程之后，通常已经能够胜任大多数业务系统中的 MyBatis-Plus 开发工作。

## 参考

- <https://baomidou.com/>
- <https://baomidou.com/pages/24112f/>
- <https://baomidou.com/pages/2976a3/>
