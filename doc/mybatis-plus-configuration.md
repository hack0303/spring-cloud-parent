# MyBatis Plus 配置指南

## 已配置的依赖

在父项目 `pom.xml` 中已经配置了以下 MyBatis Plus 相关依赖：

- **mybatis-plus**: 3.5.5
- **mybatis-plus-boot-starter**: 3.5.5
- **mybatis-plus-generator**: 3.5.5
- **mybatis-spring-boot-starter**: 3.0.3

## 在子模块中使用 MyBatis Plus

### 1. 添加依赖到子模块

在子模块的 `pom.xml` 中添加：

```xml
<dependencies>
    <!-- MyBatis Plus Starter -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
    </dependency>
    
    <!-- 数据库驱动 (根据实际数据库选择) -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- 或者使用其他数据库驱动 -->
    <!--
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.6.0</version>
    </dependency>
    -->
</dependencies>
```

### 2. 配置数据源

在 `application.yml` 或 `application.properties` 中配置数据源：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/your_database?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
    username: your_username
    password: your_password

# MyBatis Plus 配置
mybatis-plus:
  configuration:
    # 开启驼峰命名自动映射
    map-underscore-to-camel-case: true
    # 日志实现
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      # 主键类型
      id-type: auto
      # 逻辑删除字段名
      logic-delete-field: deleted
      # 逻辑删除值
      logic-delete-value: 1
      # 逻辑未删除值
      logic-not-delete-value: 0
  # mapper 文件位置
  mapper-locations: classpath*:/mapper/**/*.xml
```

### 3. 创建实体类

```java
import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String username;
    
    private String email;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    
    @TableLogic
    private Integer deleted;
}
```

### 4. 创建 Mapper 接口

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 可以添加自定义方法
}
```

### 5. 创建 Service 层

```java
import com.baomidou.mybatisplus.extension.service.IService;

public interface UserService extends IService<User> {
    // 自定义业务方法
}

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // 实现自定义业务方法
}
```

### 6. 配置 MyBatis Plus 分页插件和自动填充

```java
import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.LocalDateTime;

@Configuration
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }

    /**
     * 自动填充配置
     */
    @Bean
    public MetaObjectHandler metaObjectHandler() {
        return new MetaObjectHandler() {
            @Override
            public void insertFill(MetaObject metaObject) {
                this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
                this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
            }

            @Override
            public void updateFill(MetaObject metaObject) {
                this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
            }
        };
    }
}
```

### 7. 代码生成器配置

创建代码生成器配置类：

```java
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.OutputFile;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.Collections;

public class CodeGenerator {
    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/your_database", "username", "password")
                .globalConfig(builder -> {
                    builder.author("your_name") // 设置作者
                            .enableSwagger() // 开启 swagger 模式
                            .fileOverride() // 覆盖已生成文件
                            .outputDir("D://"); // 指定输出目录
                })
                .packageConfig(builder -> {
                    builder.parent("com.yourpackage") // 设置父包名
                            .moduleName("system") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.xml, "D://")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("user") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_"); // 设置过滤表前缀
                })
                .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
                .execute();
    }
}
```

## 常用功能

### 基本 CRUD 操作

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    // 查询所有
    public List<User> listAll() {
        return userMapper.selectList(null);
    }
    
    // 根据ID查询
    public User getById(Long id) {
        return userMapper.selectById(id);
    }
    
    // 插入
    public boolean save(User user) {
        return userMapper.insert(user) > 0;
    }
    
    // 更新
    public boolean update(User user) {
        return userMapper.updateById(user) > 0;
    }
    
    // 删除
    public boolean remove(Long id) {
        return userMapper.deleteById(id) > 0;
    }
}
```

### 条件查询

```java
// 条件构造器查询
public List<User> listByCondition(String username) {
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.like("username", username)
           .eq("deleted", 0)
           .orderByDesc("create_time");
    return userMapper.selectList(wrapper);
}

// Lambda 条件构造器
public List<User> listByConditionLambda(String username) {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.like(User::getUsername, username)
           .eq(User::getDeleted, 0)
           .orderByDesc(User::getCreateTime);
    return userMapper.selectList(wrapper);
}
```

### 分页查询

```java
public Page<User> pageList(int current, int size, String username) {
    Page<User> page = new Page<>(current, size);
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.like(User::getUsername, username)
           .eq(User::getDeleted, 0);
    return userMapper.selectPage(page, wrapper);
}
```

## 注意事项

1. 确保在子模块中正确配置数据源
2. 实体类需要添加 `@TableName` 注解指定表名
3. 主键字段需要添加 `@TableId` 注解
4. 逻辑删除字段需要添加 `@TableLogic` 注解
5. 自动填充字段需要添加 `@TableField(fill = FieldFill.INSERT)` 等注解
