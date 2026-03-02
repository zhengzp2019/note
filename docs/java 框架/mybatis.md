# mybatis

- mybaits：sql 映射框架，运行阶段使用，包括xml 方式、动态 sql 方式和注解方式。
- mybaits generator：mybatis 代码生成器，编码阶段使用。
- mybatis dynamic sql：java api 风格的动态 sql 身生成器。支持使用 java 代码生成动态 sql，而不需要使用 xml 中的若干标签。

## 分页查询

> 使用 PageHelper 插件

**核心类和接口**

- Dialect
- PageHelper
- PageInfo：取出分页结果

**关键方法**

- PageHelper.startPage：开启分页查询，该方法后的第一个 Mybatis 查询方法会被进行分页
- PageHelper.offsetPage：

**使用步骤**

1、引入 pom 依赖

```xml
<!-- PageHelper 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

2、定义Mapper 接口 和 sql

```java
@Mapper
public interface UserMapper {

   List<User> selectAll();
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <select id="selectAll" resultType="com.example.entity.User">
        SELECT id, name, age, email FROM user
    </select>

</mapper>
```

3、在 服务层使用分页

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public PageInfo<User> getUsers(int pageNum, int pageSize) {
        // 开启分页（最关键的一行）
        PageHelper.startPage(pageNum, pageSize);

        // 执行查询（这个查询会被自动拦截并分页）
        List<User> users = userMapper.selectAll();

        // 封装成分页信息对象
        return new PageInfo<>(users);
    }
}
```

4、Controller 返回分页结果

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public PageInfo<User> getUsers(
            @RequestParam(defaultValue = "1") int pageNum,
            @RequestParam(defaultValue = "10") int pageSize) {
        return userService.getUsers(pageNum, pageSize);
    }
}
```

## 参考

- <https://pagehelper.github.io/docs/howtouse/>
