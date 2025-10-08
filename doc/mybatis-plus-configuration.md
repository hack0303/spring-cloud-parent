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
    
    <!-- MyBatis Plus Generator (代码生成器) -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
    </dependency>
    
    <!-- 数据库驱动 (根据实际数据库选择) -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- 代码生成器模板引擎 -->
    <dependency>
        <groupId>org.freemarker</groupId>
        <artifactId>freemarker</artifactId>
        <version>2.3.32</version>
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

#### 完整的代码生成器配置类

```java
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.OutputFile;
import com.baomidou.mybatisplus.generator.config.rules.DbColumnType;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.po.LikeTable;

import java.util.Collections;

public class CodeGenerator {
    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/your_database", "username", "password")
                .globalConfig(builder -> {
                    builder.author("your_name") // 设置作者
                            .enableSwagger() // 开启 swagger 模式
                            .fileOverride() // 覆盖已生成文件
                            .outputDir(System.getProperty("user.dir") + "/src/main/java"); // 指定输出目录
                })
                .packageConfig(builder -> {
                    builder.parent("com.yourpackage") // 设置父包名
                            .moduleName("system") // 设置父包模块名
                            .entity("entity") // 实体类包名
                            .service("service") // Service包名
                            .serviceImpl("service.impl") // Service实现类包名
                            .mapper("mapper") // Mapper包名
                            .controller("controller") // Controller包名
                            .pathInfo(Collections.singletonMap(OutputFile.xml, 
                                System.getProperty("user.dir") + "/src/main/resources/mapper")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("user", "role", "permission") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_") // 设置过滤表前缀
                            .entityBuilder() // 实体类配置
                                .enableLombok() // 启用lombok
                                .enableTableFieldAnnotation() // 启用字段注解
                                .logicDeleteColumnName("deleted") // 逻辑删除字段名
                                .versionColumnName("version") // 乐观锁字段名
                            .serviceBuilder() // Service配置
                                .formatServiceFileName("%sService") // Service接口文件名格式
                                .formatServiceImplFileName("%sServiceImpl") // Service实现类文件名格式
                            .controllerBuilder() // Controller配置
                                .enableRestStyle() // 启用Rest风格
                                .enableHyphenStyle(); // 启用驼峰转连字符
                })
                .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板
                .execute();
    }
}
```

#### 高级代码生成器配置（支持更多自定义）

```java
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.rules.DbColumnType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.Collections;

public class AdvancedCodeGenerator {
    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/your_database", "username", "password")
                // 全局配置
                .globalConfig(builder -> {
                    builder.author("your_name")
                            .outputDir(System.getProperty("user.dir") + "/src/main/java")
                            .enableSwagger()
                            .fileOverride()
                            .dateType(DateType.TIME_PACK)
                            .commentDate("yyyy-MM-dd")
                            .disableOpenDir(); // 生成后不打开文件夹
                })
                // 包配置
                .packageConfig(builder -> {
                    builder.parent("com.yourpackage")
                            .moduleName("system")
                            .entity("entity")
                            .service("service")
                            .serviceImpl("service.impl")
                            .mapper("mapper")
                            .controller("controller")
                            .xml("mapper.xml")
                            .pathInfo(Collections.singletonMap(OutputFile.xml, 
                                System.getProperty("user.dir") + "/src/main/resources/mapper"));
                })
                // 策略配置
                .strategyConfig(builder -> {
                    builder.addInclude("user", "role", "permission") // 表名
                            .addTablePrefix("t_", "sys_") // 过滤表前缀
                            
                            // 实体类策略配置
                            .entityBuilder()
                                .enableLombok()
                                .enableTableFieldAnnotation()
                                .naming(NamingStrategy.underline_to_camel)
                                .columnNaming(NamingStrategy.underline_to_camel)
                                .logicDeleteColumnName("deleted")
                                .versionColumnName("version")
                                .addTableFills(new TableFill("create_time", FieldFill.INSERT))
                                .addTableFills(new TableFill("update_time", FieldFill.INSERT_UPDATE))
                                .idType(IdType.AUTO)
                            
                            // Controller策略配置
                            .controllerBuilder()
                                .enableRestStyle()
                                .enableHyphenStyle()
                            
                            // Service策略配置
                            .serviceBuilder()
                                .formatServiceFileName("%sService")
                                .formatServiceImplFileName("%sServiceImpl")
                            
                            // Mapper策略配置
                            .mapperBuilder()
                                .enableBaseResultMap()
                                .enableBaseColumnList()
                                .formatMapperFileName("%sMapper")
                                .formatXmlFileName("%sMapper");
                })
                // 模板配置
                .templateConfig(builder -> {
                    builder.entity("/templates/entity.java")
                            .service("/templates/service.java")
                            .serviceImpl("/templates/serviceImpl.java")
                            .mapper("/templates/mapper.java")
                            .xml("/templates/mapper.xml")
                            .controller("/templates/controller.java");
                })
                // 注入配置
                .injectionConfig(builder -> {
                    builder.beforeOutputFile((tableInfo, objectMap) -> {
                        System.out.println("tableInfo: " + tableInfo.getEntityName() + " objectMap: " + objectMap.size());
                    });
                })
                .templateEngine(new FreemarkerTemplateEngine())
                .execute();
    }
}
```

#### 使用 Maven 插件生成代码

在子模块的 `pom.xml` 中添加插件配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator-maven-plugin</artifactId>
            <version>3.5.5</version>
            <configuration>
                <configFile>src/main/resources/generator-config.yml</configurationFile>
            </configuration>
        </plugin>
    </plugins>
</build>
```

创建 `src/main/resources/generator-config.yml`：

```yaml
# 代码生成器配置
globalConfig:
  # 输出目录
  outputDir: src/main/java
  # 作者
  author: your_name
  # 开启 swagger 模式
  swagger2: true
  # 文件覆盖
  fileOverride: true

packageConfig:
  # 父包名
  parent: com.yourpackage
  # 实体类包名
  entity: entity
  # service包名
  service: service
  # serviceImpl包名
  serviceImpl: service.impl
  # mapper包名
  mapper: mapper
  # controller包名
  controller: controller

strategyConfig:
  # 表名
  include:
    - user
    - role
    - permission
  # 表前缀过滤
  tablePrefix:
    - t_
    - sys_
  entityLombokModel: true
  restControllerStyle: true
  controllerMappingHyphenStyle: true

templateConfig:
  # 禁用默认模板
  entity: /templates/entity.java
  service: /templates/service.java
  serviceImpl: /templates/serviceImpl.java
  mapper: /templates/mapper.java
  xml: /templates/mapper.xml
  controller: /templates/controller.java
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
