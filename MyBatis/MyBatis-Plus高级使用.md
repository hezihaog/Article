# MyBatis-Plus高级使用

Mybatis-Plus（MP）在 MyBatis 的基础上只做增强不做改变，简化开发、提高效率。

本篇是根据[MyBatis-Plus高级教程](https://www.imooc.com/learn/1171)视频，学习后总结的。有兴趣可以看一下，对于初学很有帮助。

课程是SpringBoot + MyBatis-Plus的方式集成的。

## 项目地址

文章可能描述的不清楚，或者你想看下代码，可以clone仓库

[mybatis-plus-high](https://github.com/hezihaog/mybatis-plus-high)

## 前置配置

### 数据库和表配置

本次学习，只用到1个库，一张表，库名为mp_hign，表名为user。直接拷贝到sql工具导入即可。

```
/*
 Navicat Premium Data Transfer

 Source Server         : mac
 Source Server Type    : MySQL
 Source Server Version : 50716
 Source Host           : localhost:3306
 Source Schema         : mp_high

 Target Server Type    : MySQL
 Target Server Version : 50716
 File Encoding         : 65001

 Date: 31/07/2020 16:30:42
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `name` varchar(30) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  `manager_id` bigint(20) DEFAULT NULL COMMENT '直属上级id',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '修改时间',
  `version` int(11) DEFAULT '1' COMMENT '版本',
  `deleted` int(1) DEFAULT '0' COMMENT '逻辑删除标识(0.未删除,1.已删除)',
  PRIMARY KEY (`id`),
  KEY `manager_fk` (`manager_id`),
  CONSTRAINT `manager_fk` FOREIGN KEY (`manager_id`) REFERENCES `user` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
BEGIN;
INSERT INTO `user` VALUES (1087982257332887553, '大boss', 40, 'boss@baomidou.com', NULL, '2019-01-11 14:20:20', NULL, 1, 0);
INSERT INTO `user` VALUES (1088248166370832385, '王天风', NULL, NULL, NULL, NULL, '2020-07-31 16:26:35', NULL, 0);
INSERT INTO `user` VALUES (1088250446457389058, '李艺伟', 28, 'lyw@baomidou.com', 1088248166370832385, '2019-02-14 08:31:16', NULL, 1, 0);
INSERT INTO `user` VALUES (1094590409767661570, '张雨琪', 31, 'zjq@baomidou.com', 1088248166370832385, '2019-01-14 09:15:15', NULL, 1, 0);
INSERT INTO `user` VALUES (1094592041087729666, '刘红雨', 32, 'lhm@baomidou.com', 1088248166370832385, '2019-01-14 09:48:16', NULL, 1, 0);
INSERT INTO `user` VALUES (1289111378211680257, '李兴华', 35, NULL, 1088248166370832385, '2020-07-31 16:11:18', '2020-07-31 16:27:18', NULL, 1);
INSERT INTO `user` VALUES (1289111378257817602, '杨红', 29, NULL, 1088248166370832385, '2020-07-31 16:11:18', NULL, NULL, 0);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 依赖配置

指定SpringBoot父工程

1. 依赖SpringBootStarter
2. lombok，生成get、set，减少模板代码
3. MyBatis-Plus给我们提供的Starter
4. mysql的驱动包。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mp</groupId>
    <artifactId>mybatis-plus-high</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <mybatis.starter.version>3.1.2</mybatis.starter.version>
    </properties>

    <!-- Spring Boot Starter 父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <dependencies>
        <!-- SpringBoot启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- SpringBoot test启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- lombok简化Java代码 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- MyBatis-Plus启动器 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis.starter.version}</version>
        </dependency>
        <!-- MySQL jdbc 驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 数据库以及MyBaits-Plus的相关配置

在resource目录下，建立application.yml文件。配置：

1. 项目名
2. 数据库的4大配置，自行更改为自己mysql的用户名和密码！
3. log打印级别

```
spring:
  application:
    name: crud
  #配置数据库
  datasource:
    #MySQL的配置
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp_high?useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8

    username: root
    password: hezihao123

#配置Log打印
logging:
  level:
    root: warn
    #配置要输出日志的类的包路径
    com.mp.dao: trace
  pattern:
    console: '%p%m%n'
```

### 应用入口

SpringBoot要求我们提供一个有main方法的入口，例如我们提供一个Application的类，用来启动

注意：要用@MapperScan注解，指定MyBatis扫描的包路径

```
//标识为SpringBoot启动类
@SpringBootApplication
//Mybatis要扫描Mapper接口的包
@MapperScan("com.mp.dao")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 建立表对应的实体类

注解介绍，在上一篇 **MyBatis入门使用** 中有介绍，这里就不再赘述了

```
@Data
public class User {
    /**
     * 主键
     */
    private Long id;
    /**
     * 姓名
     */
    private String name;
    /**
     * 年龄
     */
    private Integer age;
    /**
     * 邮箱
     */
    private String email;
    /**
     * 直属上级
     */
    private Long managerId;
    /**
     * 创建时间
     */
    //注解@TableField，fill属性，配置自动填充，在插入时，自动插入创建时间。默认是不处理的
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    /**
     * 修改时间
     */
    //注解@TableField，fill属性，配置自动填充，在更新时，自动更新时间。默认是不处理的
    @TableField(fill = FieldFill.UPDATE)
    private LocalDateTime updateTime;
    /**
     * 版本
     */
    private Integer version;
    /**
     * 逻辑删除标识（0为未删除，1为已删除）
     */
    @TableField(select = false)
    private Integer deleted;
}
```

### 配置Dao或者叫Mapper

```
public interface UserMapper extends BaseMapper<User> {
}
```

## 高级功能

### 逻辑删除

一般我们数据会用一个删除标记，来代表删除，查询时过滤即可，而不会真的删除数据。

MyBatis-Plus给我们提供了逻辑删除，只需要我们在实体中提供逻辑删除字段，以及一些配置即可。

- application.yml中，配置全局逻辑配置

1. 未删除为0
2. 已删除为1

```
mybatis-plus:
  global-config:
    db-config:
      #逻辑删除，未删除的字段值，0
      logic-not-delete-value: 0
      #逻辑删除，已删除的字段值，1
      logic-delete-value: 1
```

- 实体中配置逻辑删除字段

1. 使用注解@TableLogic，标识字段为逻辑删除字段
2. 如果该表的逻辑删除字段的值和全局配置不同，则使用value属性配置未删除值，delval配置已删除值

```
@Data
public class User {
    //省略其他字段...

    /**
     * 逻辑删除标识（0为未删除，1为已删除）
     */
    //注解@TableLogic，标识该字段为逻辑删除字段
    //你可以使用注解里面的value属性指定未删除的值，delval属性设置已删除的值，一般我们不指定，使用全局配置的，这里配置的是局部配置的
    @TableLogic
    private Integer deleted;
}
```

- 新建配置类，提供逻辑删除注入器（3.1.2版本以下需要配置，以上不需要，因为已经配置了）

```
@Configuration
public class MyBatisConfiguration {
    /**
     * 逻辑删除注入器（3.1.2版本的MyBatis-Plus，默认已经配置了，所以可以不用配置）
     * 注意，如果配置了这句，自定义Sql注入器就会找到2个ISqlInjector的实现类，会导致注入失败而抛异常，所以版本符合的情况下，就不要加了
     */
//    @Bean
//    public ISqlInjector sqlInjector() {
//        return new LogicSqlInjector();
//    }
}
```

- 测试，测试包下，建立MyTest测试类

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class MyTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 逻辑删除，由于我们实体的逻辑删除字段配置了注解@TableLogic，所以使用的是逻辑删除，而不是真正的删除
     */
    @Test
    public void deleteById() {
        int rows = userMapper.deleteById(1094592041087729666L);
        System.out.println("影响行数：" + rows);
    }
}
```

### 自动填充

我们的表中，一般会有create_time创建时间，以及update_time更新时间，我们有2种方式解决：

1. 每个insert插入方法中，对create_time赋值。没给更新方法中，对update_time赋值。
2. MyBatis-Plus提供自动填充功能，当执行新增和更新操作时，回调我们的配置类，对字段进行赋值。

第一种方式，最简单，但是每个新增、更新方法都要写一遍，如果要加逻辑，每个地方都要改。

我们希望有一个统一配置，当指定增、改方法时，回调统一配置进行处理，下面就是MyBatis-Plus自动填充功能的使用。

- 实体类中，标识自动填充字段

 1. 注解@TableField，fill属性，表示自动填充
    - 使用类型为FieldFill.INSERT，在插入时，自动插入创建时间。默认是不处理的
    - 使用类型为FieldFill.UPDATE，在更新时，自动更新时间。默认是不处理的

```
@Data
public class User {
    /**
     * 创建时间
     */
    //注解@TableField，fill属性，配置自动填充，在插入时，自动插入创建时间。默认是不处理的
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    /**
     * 修改时间
     */
    //注解@TableField，fill属性，配置自动填充，在更新时，自动更新时间。默认是不处理的
    @TableField(fill = FieldFill.UPDATE)
    private LocalDateTime updateTime;
}
```

- 提供统一配置类

配置类称之为 **元数据对象处理类**，一般用于公共字段自动写入。

1. metaObject表示元数据对象，我们从该对象中获取实体数据信息
2. hasSetter()，判断是否有某个字段的set方法
3. setInsertFieldValByName()，在新增时，设置指定字段的值，例如这里是创建时间
4. getFieldValByName()，获取实体对象中，是否已经手动设置了值，如果设置了，就可以不进行自动填充
5. setUpdateFieldValByName()，在更新时，设置指定字段的值，例如这里是更新时间

6. 建议设置值前，先判断是否有set方法，有set方法，就代表有字段，避免无意义的设置。
7. 最好设置前，先判断是否已经有值，有值则不覆盖了，建立优先级关系

```
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        String fieldName = "createTime";
        //有创建时间字段时，才进行填充，否则不处理
        boolean hasCreateTimeField = metaObject.hasSetter(fieldName);
        if (hasCreateTimeField) {
            System.out.println("============== insert 触发自动填充... ==============");
            //新增时，插入创建时间（注意传入的字段名是属性中的变量名，不是数据库中的字段名）
            setInsertFieldValByName(fieldName, LocalDateTime.now(), metaObject);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        //存在更新时间字段时，再进行自动填充，否则不处理
        String fieldName = "updateTime";
        boolean hasUpdateTime = metaObject.hasSetter(fieldName);
        //获取实体中，是否手动设置了值，如果设置了值，则不进行自动填充
        Object val = getFieldValByName(fieldName, metaObject);
        if (val == null && hasUpdateTime) {
            System.out.println("============== update 触发自动填充... ==============");
            //更新时，更新时间（注意传入的字段名是属性中的变量名，不是数据库中的字段名）
            setUpdateFieldValByName(fieldName, LocalDateTime.now(), metaObject);
        }
    }
}
```

- 测试，在测试包下，建立FillTest测试类

提供insert()新增方法和updateById()更新方法，测试运行后，创建时间和更新时间是否已赋值。

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class FillTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void inset() {
        User user = new User();
        user.setName("李玉");
        user.setAge(31);
        user.setEmail("lmc@baomidou.com");
        user.setManagerId(1088248166370832385L);
        int rows = userMapper.insert(user);
        System.out.println("影响行数：" + rows);
    }

    @Test
    public void updateById() {
        User user = new User();
        user.setId(1088248166370832385L);
        user.setAge(27);
        //手动设置更新时间，自动填充则不进行填充
        user.setUpdateTime(LocalDateTime.now());
        int rows = userMapper.updateById(user);
        System.out.println("影响行数：" + rows);
    }
}
```

### 乐观锁插件

- 乐观锁的概念

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，只在更新的时候会判断一下在此期间别人有没有去更新这个数据。

- 悲观锁的概念

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞，直到它拿到锁。

- 乐观锁实现方案

表字段提供一个version字段，从1开始。

1. 取出记录时，获取当前的version。
2. 更新时，带上这个version
3. 就是where更新条件中，比对这个version，如果version和表中记录的version对不上，代表记录已经被别人修改过了，则更新失败，否则更新成功。

#### 配置步骤

- 配置乐观锁插件

```
@Configuration
public class MyBatisConfiguration {
    //省略之前的配置...

    /**
     * 乐观锁插件
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```

- 实体类中，标识version字段

在版本字段中，加上注解@Version，标识当前字段为乐观锁字段

```
@Data
public class User {
    //省略其他字段...    

    /**
     * 版本
     */
    //注解@Version，标识当前字段为乐观锁字段
    @Version
    private Integer version;
}
```

- 测试，在测试包下，新建OptTest测试类

注意：
1. version版本字段，支持的数据类型只有：int、integer、long、Long、Date、Timestamp、LocalDateTime
2. 整数类型下，newVersion = oldVersion + 1
3. newVersion更新后，会自动回写到实体类entity中
4. 更新方法，仅只支持updateById(id) 和 update(entity, wrapper)方法
5. 在update(entity, wrapper)方法下，wrapper不能复用！！！

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class OptTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void update() {
        //模拟查询出来版本
        int version = 2;

        User user = new User();
        user.setEmail("ly3@baomidou.com");
        user.setVersion(version);
        QueryWrapper<User> query = Wrappers.query();
        query.eq("name", "李玉");
        int rows = userMapper.update(user, query);
        System.out.println("影响行数：" + rows);

        //复用会有问题！！！version字段会and多次！！
        int version2 = 3;
        User user2 = new User();
        user2.setEmail("ly4@baomidou.com");
        user2.setVersion(version2);
        query.eq("age", 25);
        int rows2 = userMapper.update(user2, query);
        System.out.println("影响行数：" + rows2);
    }

    @Test
    public void updateById() {
        //模拟查询出来版本
        int version = 1;

        User user = new User();
        user.setId(1289027494564343810L);
        user.setEmail("ly2@baomidou.com");
        //设置版本号，Mybatis-Plus会将版本号 +1 进行设置
        user.setVersion(version);
        //手动设置更新时间，自动填充则不进行填充
        user.setUpdateTime(LocalDateTime.now());

        //注意，乐观锁能生效的只有updateById()和update()方法，Wrapper的方式不能服用Wrapper，否则会有问题！！version字段会and多次！！
        int rows = userMapper.updateById(user);
        System.out.println("影响行数：" + rows);
    }
}
```

### 性能分析插件

性能分析插件，用于输出每条SQL语句的执行时间。

注意：生产环境不要用，每次性能分析，是会有开销的，所以一般只用于开发和测试环境。

#### 配置步骤

- 配置类中，增加性能分析插件

```
@Configuration
public class MyBatisConfiguration {
    /**
     * SQL性能分析插件，一般测试环境开启，生产环境不开启
     */
    @Bean
    //@Profile注解，配置开发环境、测试环境下开启
    @Profile({"dev", "test"})
    public PerformanceInterceptor performanceInterceptor() {
        PerformanceInterceptor interceptor = new PerformanceInterceptor();
        //设置是否格式化，默认为false
        interceptor.setFormat(true);
        //设置执行超过指定的毫秒数，就停止操作（SQL执行过慢）
        //interceptor.setMaxTime(5L);
        return interceptor;
    }
}
```

- 增加性能分析插件p6spy依赖（注意MyBatis-Plus要3.1.0版本以上才能使用）

pom文件中，添加依赖

```
<dependencies>
    <!-- 性能分析插件 -->
    <dependency>
        <groupId>p6spy</groupId>
        <artifactId>p6spy</artifactId>
        <version>3.8.2</version>
    </dependency>
</dependencies>
```

- 在application.yml中，修改数据设置

1. 将数据库驱动类，从MySQL的Driver改为p6spy的P6SpyDriver
2. 将jdbc连接，从mysql改为p6spy

```
spring:
  application:
    name: crud
  #配置数据库
  datasource:
    #原本MySQL的配置
    #driver-class-name: com.mysql.cj.jdbc.Driver
    #url: jdbc:mysql://localhost:3306/mp_high?useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8

    #性能分析插件的配置，驱动替换为p6的，url要在jdbc后面加p6spy
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://localhost:3306/mp_high?useSSL=false&characterEncoding=UTF-8&serverTimezone=GMT%2B8

    username: root
    password: hezihao123

# 省略其他配置...
```

- 在resource文件夹下，建立p6spy的配置文件 spy.properties

```
#3.2.1以上使用
#modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```

- UserMapper中，增加自定义SQL方法

```
public interface UserMapper extends BaseMapper<User> {
    /**
     * 自定义查询，是不会加逻辑删除的限定条件的
     * <p>
     * 有2种方式解决：
     * 1.有Wrapper对象传入，直接在Wrapper类中添加限定条件
     * 2.如果没有Wrapper对象传入，则需要写在下方的sql语句中，要特别注意
     */
    //@SqlParser注解，filter属性设置为true，让多租户配置不应用到这个方法上
    @SqlParser(filter = true)
    @Select("select * from user ${ew.customSqlSegment}")
    List<User> mySelectList(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
```

- 测试，MyTest测试类中，增加mySelect方法，测试自定义SQL查询

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class MyTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void mySelect() {
        List<User> userList = userMapper.mySelectList(
                Wrappers.<User>lambdaQuery()
                        .gt(User::getAge, 25)
                        //自定义sql查询，不会加逻辑删除的限定条件，所以需要自己加
                        .eq(User::getDeleted, 0)
        );
        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

- 性能分析打印结果

```
Consume Time：14 ms 2020-08-19 15:24:25
Execute SQL：select * from user WHERE age > 25 AND deleted = 0
```

#### 将性能分析信息输出到文件

默认性能分析信息是输出到控制台的，我们可以配置让它输出到文件中，方便我们查看。

- 修改p6spy的配置文件

1. 将appender配置注释
2. 再增加一个logfile配置，例如配置到项目的根目录log.log
3. 如果不注释掉appender，就增加logfile，会报错


```
# 省略其他配置，主要是下面2句...

# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger

#配置将日志输出到文件中（项目根目录，注意要将上面的appender，输出到控制台的配置注释掉）
logfile=log.log
```

### 多租户SQL解析器

多租户技术是一种软件架构技术，是实现如何在多用户环境下。多用户指的一般是企业的用户，共用相同的系统，并且可确保各用户间，数据的隔离。

简单来说，多租户是一种架构，目的是为了，多用户环境下使用同一套程序，且要保证用户间数据隔离。

多租户一般有3种隔离方案：

1. 独立数据库，一个租户一个数据库，这种方案，数据隔离界别最高，优点是每个租户独立一个数据库，有助于数据模型的扩展设计，满足独特租户的不同需求。如果出现故障，恢复数据比较简单。缺点是增加了数据的安装数量，随之代来了安装成本和购置成功的增加。
2. 共享数据库，独立Schema，就是所有租户共享DataBase，但是每个租户有一个Schema，也可以叫一个user，例如Oracle、db2都可以设置同一个数据库下，设置多个Schema。优点是为安全性要求较高的用户，提供了较高的一定程度的逻辑数据隔离，它并不是完全的隔离，数据可支持更多的租户数量。缺点是如果出现了故障，数据恢复比较困难，因为数据库涉及到其他租户的数据。
3. 共享数据库，共享Schema，共享数据表，就是租户共享一个DataBase，同一个Schema，但是在表中增加多租户的租户Id这个数据字段，这个是共享程度最高，隔离级别最低的模式。简单来说，就是每插入一条数据时，都要有一个客户的标识，比如说tenantId这个区分不同租户，这样才能在一张表中，区分出不同的租户数据。优点是在这3种方案中，维护成本和购置成本最低的，也允许没给数据库支持的用户数最多，缺点是隔离级别最低，安全性最低，需要在设计开发时，加大对安全的开发量，数据备份和恢复最困难，甚至可能需要逐条备份和恢复还原。

#### 实现步骤

- 配置类中，配置分页插件

多租户是依赖于分页插件的，所以需要在配置类中，配置分页插件

1. 创建SQL解析器列表sqlParserList，每个解析器都要add到该列表中
2. 创建多租户解析器TenantSqlParser，设置一个TenantHandler 多租户处理器
3. TenantHandler处理器有3个方法需要复写
    - getTenantIdColumn()，返回多租户Id的字段名
    - getTenantId()，租户信息Id，一般情况下，可能会从session中取、配置文件中取、或者从静态变量等取出
    - doTableFilter()，可以过滤某些表的操作才加上多租户条件，返回true，代表过滤，就不加，如果返回false，代表不过滤，就是加

```
@Configuration
public class MyBatisConfiguration {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor interceptor = new PaginationInterceptor();
        ArrayList<ISqlParser> sqlParserList = new ArrayList<>();

        //多租户解析器
        TenantSqlParser tenantSqlParser = new TenantSqlParser();
        tenantSqlParser.setTenantHandler(new TenantHandler() {
            @Override
            public String getTenantIdColumn() {
                //返回多租户的字段名，注意是数据库字段名，而不是实体的变量名
                return "manager_id";
            }

            @Override
            public Expression getTenantId() {
                //租户信息Id，一般情况下，可能会从session中取、配置文件中取、或者从静态变量等取出
                //这里不做演示，先写死
                return new LongValue(1088248166370832385L);
            }

            @Override
            public boolean doTableFilter(String tableName) {
                //可以指定哪些加上租户信息，哪些不加上
                //返回true，代表过滤，就不加
                //返回false，代表不过滤，就是加
                if (tableName.equals("role")) {
                    //如果是角色表，就不加租户条件
                    return true;
                }
                return false;
            }
        });
        //将多租户解析器添加到解析器列表中
        sqlParserList.add(tenantSqlParser);

        //配置解析器列表到插件中
        interceptor.setSqlParserList(sqlParserList);
        return interceptor;
    }
}
```

- 特定SQL过滤

就是表中特定（CRUD）的方法，不加多租户的条件，其他方法则加上。

例如要过滤UserMapper的selectById()方法。

1. SqlParserHelper.getMappedStatement()，传入metaObject，获取当前要执行的方法的statement对象
2. statement.getId()，获取执行方法的Id，它是Mapper全路径+方法名，所以我们判断它是否是UserMapper的selectById即可

```
@Configuration
public class MyBatisConfiguration {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        //省略上面的多租户解析器配置...

        //添加过滤器，可以让指定方法不添加多租户条件
        interceptor.setSqlParserFilter(new ISqlParserFilter() {
            @Override
            public boolean doFilter(MetaObject metaObject) {
                MappedStatement statement = SqlParserHelper.getMappedStatement(metaObject);
                //过滤UserMapper的selectById()方法，不添加租户条件
                //多个Mapper的方法，都在这里配置，会很多，一般使用@SqlParser注解，配置到Mapper的方法上来进行！例如UserMapper就配置了
                if ("com.mp.dao.UserMapper.selectById".equals(statement.getId())) {
                    return true;
                }
                return false;
            }
        });

        return interceptor;
    }
}
```

- 测试，在MyTest测试类中，测试selectById()方法

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class MyTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void selectById() {
        User user = userMapper.selectById(1087982257332887553L);
        System.out.println(user);
    }
}
```

- 输出结果

```
//where中，没有多租户的判断条件
SELECT id,name,age,email,manager_id,create_time,update_time,version FROM user WHERE id=? AND deleted=0
```

#### 另外一种过滤方式

如果有好几个表，有2、3个方法需要过滤，每个都要去配置类中配置方法名，太麻烦了，如果方法重命名了，还要记住去配置类中同步修改，不然就会过滤失败

其实还有另外一种方式，我们更常用，就是在Mapper中的对指定方法上，加上注解@SqlParser，例如这里对mySelect()方法进行过滤

- @SqlParser注解，filter属性设置为true，让多租户配置不应用到这个方法上

```
public interface UserMapper extends BaseMapper<User> {
    /**
     * 自定义查询，是不会加逻辑删除的限定条件的
     * <p>
     * 有2种方式解决：
     * 1.有Wrapper对象传入，直接在Wrapper类中添加限定条件
     * 2.如果没有Wrapper对象传入，则需要写在下方的sql语句中，要特别注意
     */
    //@SqlParser注解，filter属性设置为true，让多租户配置不应用到这个方法上
    @SqlParser(filter = true)
    @Select("select * from user ${ew.customSqlSegment}")
    List<User> mySelectList(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
```

- application.yml中进行配置

在mybatis-plus配置中，3.1.1版本之前，配置多租户过滤，需要将该属性设置为true，否则就可以不配置

```
mybatis-plus:
  global-config:
    db-config:
      #逻辑删除，未删除的字段值，0
      logic-not-delete-value: 0
      #逻辑删除，已删除的字段值，1
      logic-delete-value: 1
    # 3.1.1版本之前，配置多租户过滤，需要将该属性设置为true
    sql-parser-cache: true
```

- 测试，MyTest测试类中，测试mySelect()方法

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class MyTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void mySelect() {
        List<User> userList = userMapper.mySelectList(
                Wrappers.<User>lambdaQuery()
                        .gt(User::getAge, 25)
                        //自定义sql查询，不会加逻辑删除的限定条件，所以需要自己加
                        .eq(User::getDeleted, 0)
        );
        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

- 输出结果

```
//没有加上多租户的查询条件
DEBUG==>  Preparing: select * from user WHERE age > ? AND deleted = ? 
DEBUG==> Parameters: 25(Integer), 0(Integer)
```

### 动态表名SQL解析器

#### 应用场景

一个项目，有多个表，这些表都是存储同类型的数据。字段都是一样的，一般是由于存到同一个数据表中，数据量比较大。所以进行了分表存储。

例如某些日志数据的表，一个月一张，命名规则是xxx_年月，比如说19年7月份，就是xxx_201907。

还有某些业务表，是针对不同机构的，比如一张机构一张表，命名规则是xxx_ + organizationId等等。

反正无论什么规则，都是要有规则的，一般规则，前缀都是相同的，后面根据不同情况拼接不同的内容，当在程序中进行查询的时候，或者其他操作，在调用的时候，才能确定操作哪张表。要动态的拼接表名.

这时候，动态表名的解析器就派上了用场。

#### 动态表名的实现步骤

动态表名插件也是要写在分页插件中的，和多租户的SQL解析器是配置过程类似。

- 配置类中，在分页插件中配置动态表名SQL解析器

1. 创建SQL解析器列表sqlParserList，每个解析器都要add到该列表中
2. 创建动态表名解析器DynamicTableNameParser，解析器需要配置一个TableNameHandlerMap，就是一个Map
3. TableNameHandlerMap，key是原表名，value是ITableNameHandler，也相当于一个处理器，处理器要求复写一个dynamicTableName()方法，该方法返回动态规则生成后的表名，如果返回null，则代表不替换

这里为了演示效果，不新建表，只是通过一个ThreadLocal变量，

```
@Configuration
public class MyBatisConfiguration {
    public static ThreadLocal<String> myTableName = new ThreadLocal<>();

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor interceptor = new PaginationInterceptor();
        ArrayList<ISqlParser> sqlParserList = new ArrayList<>();

        //省略上面的多租户解析器配置...

        //动态表名解析器，可用于一定规则的分表，例如表数据量很大，表名按一定前缀+日期+表名，来分开不同的表来查询
        DynamicTableNameParser dynamicTableNameParser = new DynamicTableNameParser();
        Map<String, ITableNameHandler> tableNameHandlerMap = new HashMap<>();
        tableNameHandlerMap.put("user", new ITableNameHandler() {
            @Override
            public String dynamicTableName(MetaObject metaObject, String sql, String tableName) {
                //如果返回为null，则不进行替换
                return myTableName.get();
            }
        });
        dynamicTableNameParser.setTableNameHandlerMap(tableNameHandlerMap);
        sqlParserList.add(dynamicTableNameParser);

        //配置解析器列表到插件中
        interceptor.setSqlParserList(sqlParserList);
        return interceptor;
    }
}
```

- 测试，测试select方法

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class MyTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void select() {
        //这里测试一下，例如我们让表名是按年份规则进行区分
        MyBatisConfiguration.myTableName.set("user_2019");

        List<User> userList = userMapper.selectList(null);
        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

- 输出，因为没有表，所以会报表不存在，原因是因为在SQL中，已经用上了我们的动态表名

```
//报错信息
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Table 'mp_high.user_2019' doesn't exist
//SQL语句
SQL: SELECT  id,name,age,email,manager_id,create_time,update_time,version FROM user_2019 WHERE deleted=0
```

#### 注意事项

1. 如果分页插件的过滤器和动态表名，2个配置同时存在，当我们指定一个被过滤的方法，表名是不会进行动态替换的。
2. 如果是在Mapper中使用@SqlParser注解，配置filter属性为true，和上面在分页插件中配置过滤器一样，表名是不会进行动态替换的。

### SQL注入器

使用SQL注入器，我们就可以自定义通用方法，就像BaseMapper中的selectById、insert等。自定义的方法要添加SQL注入器中，例如MP给我们提供的那些好用的通用方法，也是添加到SQl注入器中。
如果MP提供的方法，不能完全覆盖我们的需求，我们就可以编写自定义方法。

#### 自定义方法的实现步骤

- 创建定义方法的类

定义deleteAll()方法，

```
public class DeleteAllMethod extends AbstractMethod {
    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        //执行的sql
        String sql = "delete from " + tableInfo.getTableName();
        //Mapper接口方法名
        String method = "deleteAll";
        SqlSource sqlSource = languageDriver.createSqlSource(configuration, sql, modelClass);
        return addDeleteMappedStatement(mapperClass, method, sqlSource);
    }
}
```

- 创建注入器

自定义注入器，需要实现ISqlInjector接口，我们直接继承一个默认的注入器：DefaultSqlInjector，重写getMethodList()方法

将我们的自定义方法类DeleteAllMethod，加入到methodList，并返回即可。

注意：

1. super调用getMethodList()，不能去掉，否则父类中那一堆通用方法都会丢失
2. 注入器的类头上，必须加上@Component注解，否则不生效

```
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        //加入自定义的删除所有的方法
        methodList.add(new DeleteAllMethod());
        return methodList;
    }
}
```

- 在Mapper中加入自定义方法

```
public interface UserMapper extends BaseMapper<User> {
    //省略其他方法...

    /**
     * 自定义sql注入方法，即使不写sql语句，也能删除所有，返回影响行数
     */
    int deleteAll();
}
```

- 测试，测试包下，建立InjectorTest测试类

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class InjectorTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void deleteAll() {
        int rows = userMapper.deleteAll();
        System.out.println("影响行数：" + rows);
    }
}
```

- 抽取自己的通用Mapper

如果其他表也需要上面的deleteAll()方法，每个Mapper都加，肯定是不现实的，所以我们可以像BaseMapper那样，抽取自己的通用Mapper接口。

1. MyMapper接口继承MP的BaseMapper接口，注意泛型要继续传递
2. 最后让UserMapper接口，继承自定义MyMapper接口即可

```
public interface MyMapper<T> extends BaseMapper<T> {
    /**
     * 自定义sql注入方法，即使不写sql语句，也能删除所有，返回影响行数
     */
    int deleteAll();
}

public interface UserMapper extends MyMapper<User> {
}
```

### 选装件

MP从3.0.7开始，从3.1.2版本，增加了3个选装件。

其实选装件也是自定义方法，只不过是MP官方写的。

#### 批量新增数据，自选字段insert

该选装件叫 **InsertBatchSomeColumn**。选装件的方法名为 **insertBatchSomeColumn**。

例如：因为我们在数据库中，设置了逻辑删除字段的默认值，所以我们想批量插入时，不将逻辑删除字段作为字段处理。

- 使用步骤

1. 在自定义注入器中，加入InsertBatchSomeColumn选装件。

InsertBatchSomeColumn的构造方法中，传入Predicate类，复写test()方法，返回true，代表加入插入字段中，false则为不加入，所以我们判断传入的TableFieldInfo，是否是逻辑删除字段，如果是，则返回false，代表不加入SQL中。

如果还想排除其他其他字段，则通过TableFieldInfo，调用getColumn()获取到列名，判断列名来过滤即可。

注意：1）该选装件没有过滤的字段，在批量插入时，如果是null值，也会出现在sql语句中，所以例如version字段，数据库默认值为1，但因为该选装件没有排除该字段，会导致插入和更新时，赋值给null，这里一定要注意。
     2）作者只在MySQL数据库中测试过，其他数据库没有测试过，所以要慎用这个选装件。

```
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        //加入自定义的删除所有的方法
        methodList.add(new DeleteAllMethod());
        //1.官方选装件，批量插入，支持排除某些字段不进行批量插入的字段中
        methodList.add(new InsertBatchSomeColumn(new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo info) {
                //返回true，代表加入插入字段中，false则为不加入
                //除了逻辑删除的字段，其他都加入
                return !info.isLogicDelete();
                //如果还想排除其他字段，使用getColumn()方法获取列名
                //return !info.isLogicDelete() && !info.getColumn().equals("age");
            }
        }));
        return methodList;
    }
}
```

2. 在自定义的通用Mapper中，添加insertBatchSomeColumn()方法

```
public interface MyMapper<T> extends BaseMapper<T> {
    /**
     * 自定义sql注入方法，即使不写sql语句，也能删除所有，返回影响行数
     */
    int deleteAll();

    /**
     * 批量插入
     */
    int insertBatchSomeColumn(List<T> list);
}
```

3. 测试，InjectorTest测试类中，添加insertBatchSomeColumn()测试方法

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class InjectorTest {
    //省略其他方法...

    /**
     * 官方选装件，批量插入
     */
    @Test
    public void insertBatchSomeColumn() {
        User user1 = new User();
        user1.setName("李兴华");
        user1.setAge(34);
        user1.setManagerId(1088248166370832385L);

        User user2 = new User();
        user2.setName("杨红");
        user2.setAge(29);
        user2.setManagerId(1088248166370832385L);

        List<User> list = Arrays.asList(user1, user2);
        int rows = userMapper.insertBatchSomeColumn(list);
        System.out.println("影响行数：" + rows);
    }
}
```

#### 根据 id 逻辑删除数据，并带字段填充功能

该选装件的类名为 **LogicDeleteByIdWithFill**，方法名为 **deleteByIdWithFill**。

当我们在逻辑删除时，需要对某些字段进行更新，这时该选装件就派上了用场。

比如说，逻辑删除的时候，设置删除人是谁等等。

- 使用步骤

1. 添加 **deleteByIdWithFill** 到自定义的SQL注入器中

```
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);

        //省略其他选装件和自定义方法...

        //2.官方选装件，逻辑删除时，自动填充其他字段（例如逻辑删除时，自动填充删除人是谁，什么时候删除的）
        methodList.add(new LogicDeleteByIdWithFill());        
        return methodList;
    }
}
```

2. 在自定义通用Mapper中添加方法

```
public interface MyMapper<T> extends BaseMapper<T> {
    //省略其他方法...

    /**
     * 逻辑删除，并且带自动填充
     */
    int deleteByIdWithFill(T entity);
}
```

3. 实体类中，对自动填充的字段，加上注解@TableField

这里我们不添加字段了，就用age字段，在逻辑删除时，将年龄改为35！

给age字段，加上注解@TableField，配置fill属性为FieldFill.UPDATE，标识更新时，自动填充

```
@Data
public class User {
    //省略其他字段...    

    /**
     * 年龄
     */
    @TableField(fill = FieldFill.UPDATE)
    private Integer age;
}
```

4. 测试，在InjectorTest测试类中，添加deleteByIdWithFill测试方法

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class InjectorTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 逻辑删除，并带自动填充
     */
    @Test
    public void deleteByIdWithFill() {
        User user = new User();
        user.setId(1289111378211680257L);
        //逻辑删除后，更新年龄为35，注意实体属性age要加上@TableField(fill = FieldFill.UPDATE)注解
        user.setAge(35);
        int rows = userMapper.deleteByIdWithFill(user);
        System.out.println("影响行数：" + rows);
    }
}
```

#### 根据 id 更新固定的某些字段

选装件的类名为 **AlwaysUpdateSomeColumnById**，方法名为 **AlwaysUpdateSomeColumnById**

根据 ID 更新固定的那几个字段(但是不包含逻辑删除)，就是逻辑已经排除了。

意思就是：你可以设置哪些字段需要更新，哪些字段不需要更新。如果这些设置需要更新的字段，在实体中没有设置值，那么会再更新时，设置成null。

- 使用步骤

1. 自定义SQL注入器中，添加 **AlwaysUpdateSomeColumnById** 选装件

例如我们在更新时，忽略name字段，其他非逻辑删除的字段会进行更新，而name和逻辑删除字段不会更新

```
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        
        //省略其他选装件和自定义方法...

        //3.官方选装件，根据Id更新固定几个字段
        methodList.add(new AlwaysUpdateSomeColumnById(new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo info) {
                //当更新时，忽略name字段，其他非逻辑删除字段会进行更新
                return !info.getColumn().equals("name");
            }
        }));
        return methodList;
    }
}
```

2. 在自定义Mapper中，添加 **AlwaysUpdateSomeColumnById** 方法

```
public interface MyMapper<T> extends BaseMapper<T> {
    //省略其他方法...

    /**
     * 根据Id，更新固定几个字段
     */
    int alwaysUpdateSomeColumnById(@Param(Constants.ENTITY) T entity);
}
```

3. 测试，InjectorTest测试类中，添加alwaysUpdateSomeColumnById方法

例如我们给某个用户，更改姓名，但是因为我们name字段是被忽略的，sql中不会出现name在更新字段中

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class InjectorTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 根据Id，更新固定几个字段（一般是排除固定的一些字段）
     */
    @Test
    public void alwaysUpdateSomeColumnById() {
        User user = new User();
        user.setId(1088248166370832385L);
        user.setName("王第风");
        int rows = userMapper.alwaysUpdateSomeColumnById(user);
        System.out.println("影响行数：" + rows);
    }
}
```

4. 输出

sql中，没有name字段，其他字段都出现了，但是因为我们没有设置值，而除了update_time更新时间字段，因为我们配置了更新时间的自动填充，所有有值，其他字段都更新成null了

```
DEBUG==>  Preparing: UPDATE user SET age=?, email=?, manager_id=?, create_time=?, update_time=?, version=? WHERE id=? AND deleted=0 
DEBUG==> Parameters: null, null, null, null, 2020-08-19T17:23:13.960(LocalDateTime), null, 1088248166370832385(Long)
```