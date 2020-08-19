# MyBatis-Plus入门使用

Mybatis-Plus（MP）在 MyBatis 的基础上只做增强不做改变，简化开发、提高效率。

本篇是根据[MyBatis-Plus入门教程](https://www.imooc.com/learn/1130)视频，学习后总结的。有兴趣可以看一下，对于初学很有帮助。

课程是SpringBoot + MyBatis-Plus的方式集成的。

## 项目地址

文章可能描述的不清楚，或者你想看下代码，可以clone仓库

[immoc-mybatis-plus](https://github.com/hezihaog/immoc-mybatis-plus)

## 前置配置

### 数据库和表配置

本次学习，只用到1个库，一张表，库名为mp，表名为mp_user。直接拷贝到sql工具导入即可。

```
/*
 Navicat Premium Data Transfer

 Source Server         : mac
 Source Server Type    : MySQL
 Source Server Version : 50716
 Source Host           : localhost:3306
 Source Schema         : mp

 Target Server Type    : MySQL
 Target Server Version : 50716
 File Encoding         : 65001

 Date: 30/07/2020 23:12:46
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for mp_user
-- ----------------------------
DROP TABLE IF EXISTS `mp_user`;
CREATE TABLE `mp_user` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `name` varchar(30) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  `manager_id` bigint(20) DEFAULT NULL COMMENT '直属上级id',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `manager_fk` (`manager_id`),
  CONSTRAINT `manager_fk` FOREIGN KEY (`manager_id`) REFERENCES `mp_user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of mp_user
-- ----------------------------
BEGIN;
INSERT INTO `mp_user` VALUES (1087982257332887553, '大boss', 40, 'boss@baomidou.com', NULL, '2019-01-11 14:20:20');
INSERT INTO `mp_user` VALUES (1088248166370832385, '王天风', 26, 'wtf2@baomidou.com', 1087982257332887553, '2019-02-05 11:12:22');
INSERT INTO `mp_user` VALUES (1088250446457389058, '李艺伟', 32, 'lyw2019@baomidou.com', 1088248166370832385, '2019-02-14 08:31:16');
INSERT INTO `mp_user` VALUES (1094590409767661570, '张雨琪', 31, 'zjq@baomidou.com', 1088248166370832385, '2019-01-14 09:15:15');
INSERT INTO `mp_user` VALUES (1094592041087729666, '刘红雨', 32, 'lhm@baomidou.com', 1088248166370832385, '2019-01-14 09:48:16');
INSERT INTO `mp_user` VALUES (1288460195017072641, '刘明强', 31, NULL, 1088248166370832385, '2020-07-29 21:03:44');
INSERT INTO `mp_user` VALUES (1288842439342780417, '张强', 29, 'lh@baomidou.com', 1088248166370832385, '2020-07-30 22:23:33');
INSERT INTO `mp_user` VALUES (1288848935841488897, '刘花', 29, 'lh@baomidou.com', 1088248166370832385, '2020-07-30 22:48:27');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 依赖配置

指定SpringBoot父工程

1. 依赖SpringBootStarter
2. lombok，生成get、set，减少模板代码
3. MyBatis-Plus给我们提供的Starter
4. 最后是mysql的驱动包。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.immoc.mybatisplus</groupId>
    <artifactId>immoc-mybatis-plus</artifactId>
    <version>1.0-SNAPSHOT</version>

    <modules>
        <module>first</module>
        <module>crud</module>
    </modules>
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <mybatis.starter.version>3.1.0</mybatis.starter.version>
    </properties>

    <!-- Spring Boot Starter 父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
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

在resource目录下，建立`application.yml`文件。配置：

1. 项目名
2. 数据库的4大配置，自行更改为自己mysql的用户名和密码！
3. log打印级别
4. MyBatis-Plus配置等，注释都写得很清楚了，就不在赘述了

```
spring:
  application:
    name: crud
  #配置数据库
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp?useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: hezihao123

#配置Log打印
logging:
  level:
    root: warn
    com.imooc.mybatisplus: trace
  pattern:
    console: '%p%m%n'

mybatis-plus:
  #配置mapper文件的位置，注意，如果你是maven多模块下使用，路径前需要加classpath*，即加载多个jar包下的xml文件
  mapper-locations:
    - classpath*:com/imooc/mybatisplus/mapper/*
  global-config:
    db-config:
      #全局id策略
      id-type: id_worker
      #字段生成sql的where策略，默认实体中的字段为null时，不添加到sql中，如果想为null，也添加到sql语句中，则使用ignored
      #一般不设置为ignored，因为在update语句中，如果不设置值，就会用null覆盖掉原有的值。一般我们是希望设置值的才更新，为null则不更新
      #not_empty，如果字段值为null或者空字符串，会忽略掉，不添加到sql语句中
      field-strategy: default
      #统一表名前缀
      #table-prefix: mp_
      #数据库表是否使用下划线间隔命名，默认为true
      table-underline: true
  #传统的mybatis配置文件位置
  #config-location: classpath:mybatis-config.xml
  #实体别名包配置
  type-aliases-package: com.imooc.mybatisplus.entity
  #注意configuration不能和config-location同时出现，不然会报错
  configuration:
    #驼峰转下划线（实体类用驼峰，数据库表字段用下划线），默认为true
    map-underscore-to-camel-case: true
```

### 配置原始MyBatis的配置文件

基本上MyBatis的配置都会配置到`application.yml`中，但有一些MyBatis的特有配置，还是可以放到原始的配置文件中的，例如二级、三级缓存等
在resources文件夹内创建`mybatis-config.xml`文件，暂时先不做配置。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

### 配置MyBatis-Plus的分页插件

建立一个configuration包，增加MyBatisPlusConfig类，该类为一个配置类，提供PaginationInterceptor分页插件实例。

```
@Configuration
public class MyBatisPlusConfig {
    /**
     * MyBatisPlus的分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

### 应用入口

SpringBoot要求我们提供一个有main方法的入口，例如我们提供一个Application的类，用来启动

注意：要用@MapperScan注解，指定MyBatis扫描的包路径

```
@SpringBootApplication
//指定MyBatis扫描的包路径
@MapperScan("com.imooc.mybatisplus.dao")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 建立表对应的实体类

1. @Data注解，生成实体类的get、set方法
2. @EqualsAndHashCode注解，生成equals和hashcode方法，并指定不调用父类的equals和hashcode方法
3. @TableName注解，指定实体类对应的表名，如果表名和实体类名一致，可以不加，但如果不一致，就需要该注解指定表名
4. @TableId注解，指定实体类上成员变量为数据库表中的主键字段，value属性指定表中主键的名称。默认主键名为id，2个名字和数据库一致时，可以不加，不一致时需要加
5. @TableField注解，表示成员变量为数据库对应的字段，如果成员变量和表字段的名称一致可以不加，如果不一致，就要加
    - value属性，用于指定对应的表字段名
    - strategy属性，可指定字段策略，默认当字段非null时，进行插入和更新操作，才会加入sql语句的数据字段中，可以配置多种策略，例如FieldStrategy.NOT_EMPTY，当字段为非null和非空字符串时，才加入sql语句
    - exist属性，标识该成员变量不是数据库表中字段，CRUD时，不会将该字段加入sql

```
@Data
@EqualsAndHashCode(callSuper = false)
@TableName("mp_user")
public class User {
    /**
     * 主键
     * 注解@TableId，标识字段为主键，如变量名和数据库字段名不一致，可指定
     */
    //@TableId(value = "id")
    /**
     * 主键策略
     * 1.IdType.AUTO：数据库自增
     * 2.IdType.NONE：不配置，默认雪花算法
     * 以下3种策略，主键的Id不能设置值，才会生效
     * 3.IdType.ID_WORKER：雪花算法，数值类型
     * 4.IdType.UUID：UUID，字符串类型
     * 5.IdType.ID_WORKER_STR，雪花算法，字符串类型
     */
    @TableId(value = "id")
    private Long userId;
    /**
     * 姓名
     * 注解@TableField，标识为数据库字段，如变量名和数据库字段名不一致，可指定
     * condition属性，用作在使用实体作为查询条件给MyBatis-Plus直接查询时，可以指定字段是作为等值还是模糊查询，或不等于等条件（默认是等值）
     */
    //@TableField(value = "name", condition = SqlCondition.LIKE)
    /**
     * strategy属性，字段策略，NOT_EMPTY会忽略为null和空字符串的值
     */
    //@TableField(value = "name", strategy = FieldStrategy.NOT_EMPTY)
    @TableField(value = "name")
    private String readName;
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
    private LocalDateTime createTime;
    /**
     * 备注，非数据库字段，默认MyBatis-Plus会将实体的所有变量名作为数据库字段，如果没有字段就会报错
     * 标识为非数据库字段的3种方式：
     * 1.transient关键字，标识该字段不参与序列化
     * 2.将字段使用static静态标识，但会导致全类共用一个属性，一般不会用
     * 3.使用@TableField注解，将exist属性设置为false，来表示不是数据库字段
     */
    //private transient String remark;
    //private static String remark;
    @TableField(exist = false)
    private String remark;
}
```

### 配置Dao或者叫Mapper

UserMapper是User类对应的Mapper接口。每个自定义Mapper接口都继承BaseMapper接口，泛型指定操作的实体类。

BaseMapper中提供了很多单表操作，批量操作、条件查询、分页查询等，常用方法。
我们的自定义Mapper接口只写特殊的操作方法，例如自定义的多表联查等，现在暂时不加，后面讲解自定义方法时，再加。

```
public interface UserMapper extends BaseMapper<User> {
}
```

## 测试CRUD

- 插入测试，测试包下，新建InsertTest测试类

插入调用UserMapper的insert()方法，这是BaseMapper提供的。

```
//指定可以在Spring环境下使用Junit测试
@RunWith(SpringRunner.class)
//标识该类是SpringBoot测试类，并指定启动类
@SpringBootTest(classes = Application.class)
public class InsertTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void insert() {
        //插入
        User user = new User();
        user.setReadName("刘明强");
        user.setAge(31);
        user.setManagerId(1088248166370832385L);
        user.setCreateTime(LocalDateTime.now());
        int rows = userMapper.insert(user);
        System.out.println("影响记录数：" + rows);
    }

    @Test
    public void insert2() {
        //插入
        User user = new User();
        user.setReadName("向中");
        user.setAge(25);
        user.setManagerId(1088248166370832385L);
        user.setEmail("xd@baomidou.com");
        user.setCreateTime(LocalDateTime.now());
        user.setRemark("我是一个备注信息");
        int rows = userMapper.insert(user);
        System.out.println("影响记录数：" + rows);
    }
}
```

- 删除测试，测试包下，新建DeleteTest测试类

删除方法，BaseMapper提供了好几种

1. deleteById，根据主键进行删除
2. deleteByMap，将删除条件放到Map中，进行删除
3. deleteBatchIds，批量Id删除
4. delete，传入Wrapper，Wrapper类是一个条件构造器，构造器中有or、get大于等条件添加方法。例如这里是删除年龄是27岁的，或者年龄大于41岁的记录

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class DeleteTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 根据主键Id删除
     */
    @Test
    public void deleteById() {
        int rows = userMapper.deleteById(1288677289033801729L);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 提供一个Map，Map中记录删除条件，进行删除
     */
    @Test
    public void deleteByMap() {
        HashMap<String, Object> columnMap = new HashMap<>();
        columnMap.put("id", 1288676754557825026L);
        columnMap.put("age", 25);
        int rows = userMapper.deleteByMap(columnMap);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 传入多个id，进行批量删除
     */
    @Test
    public void deleteIds() {
        int rows = userMapper.deleteBatchIds(Arrays.asList(1288461659789660161L,
                1288671248736870402L, 1288676285735272450L));
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 使用条件构造器，进行删除
     */
    @Test
    public void deleteByWrapper() {
        LambdaQueryWrapper<User> lambdaQuery = Wrappers.lambdaQuery();
        lambdaQuery.eq(User::getAge, 27)
                .or()
                .gt(User::getAge, 41);
        int rows = userMapper.delete(lambdaQuery);
        System.out.println("影响记录数：" + rows);
    }
}
```

- 更新测试，测试包下，建立UpdateTest测试类

BaseMapper提供了如下更新方式：

1. updateById，根据实体的主键Id进行更新
2. update，传入实体和Wrapper类，实体设置更新后的值，Wrapper配置条件
3. update，和第二种差不多，只是将Wrapper类构造时，传入实体类
4. update，只用Wrapper，实体类传null，更新值用Wrapper的set方法设置
5. update，和第四种一样，Wrapper类换成LambdaXXXWrapper，支持Lambda表达式
6. update方法不在Mapper类中，而是在Wrapper构造时传入Mapper，调用Wrapper的update方法时，间接调用欧冠Mapper类

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UpdateTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 更新条件和更新信息放在实体，按主键Id更新
     */
    @Test
    public void updateById() {
        User user = new User();
        user.setUserId(1088248166370832385L);
        user.setAge(26);
        user.setEmail("wtf2@baomidou.com");
        int rows = userMapper.updateById(user);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 更新信息放在实体，更新条件使用条件构造器，进行更新
     */
    @Test
    public void updateByWrapper() {
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("name", "李艺伟")
                .eq("age", 28);
        User user = new User();
        user.setEmail("lyw2019@baomidou.com");
        user.setAge(29);
        int rows = userMapper.update(user, updateWrapper);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * UpdateWrapper创建时传入更新实体
     */
    @Test
    public void updateByWrapper2() {
        User whereUser = new User();
        whereUser.setReadName("李艺伟");
        whereUser.setEmail("lyw2019@baomidou.com");
        whereUser.setAge(29);
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>(whereUser);
        int rows = userMapper.update(whereUser, updateWrapper);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 通过UpdateWrapper进行条件设置，并且通过set方法设置新值
     */
    @Test
    public void updateByWrapper3() {
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.eq("name", "李艺伟")
                .eq("age", 29)
                .set("age", 30);
        int rows = userMapper.update(null, updateWrapper);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 通过UpdateWrapper进行条件设置，并且通过set方法设置新值
     */
    @Test
    public void updateByWrapperLambda() {
        LambdaUpdateWrapper<User> updateWrapper = Wrappers.lambdaUpdate();
        updateWrapper.eq(User::getReadName, "李艺伟")
                .eq(User::getAge, 30)
                .set(User::getAge, 31);
        int rows = userMapper.update(null, updateWrapper);
        System.out.println("影响记录数：" + rows);
    }

    /**
     * 另外一种Lambda表达式方式
     */
    @Test
    public void updateByWrapperLambdaChain() {
        LambdaUpdateChainWrapper<User> updateWrapper = new LambdaUpdateChainWrapper<>(userMapper);
        boolean isUpdateSuccess = updateWrapper.eq(User::getReadName, "李艺伟")
                .eq(User::getAge, 31)
                .set(User::getAge, 32)
                .update();
        System.out.println("是否更新成功：" + isUpdateSuccess);
    }
}
```

- 查询测试，在测试包下，新增RetrieveTest测试类

1. selectById，按主键Id查询
2. selectBatchIds，批量Id查询
3. selectByMap，查询条件放到Map中，进行查询
4. selectList，提供QueryWrapper类实例，作为查询条件构造器使用
5. selectMaps，提供QueryWrapper类实例，作为查询条件构造器使用，但返回的查询结果，List中不是实体类对象，而是一个Map，Map中存放着每一条属性和值
6. selectObjs，提供QueryWrapper类实例，作为查询条件构造器使用，该方法只会拿出查询结果的第一列的属性和值返回（sql中是会查询其他字段的，但是selectObjs()方法只选择第一列的数据，只返回第一列的时候可以用它）
7. selectCount，统计查询，统计查询结果，多少条记录
8. selectOne，只查询出1条数据（必须查询结果只有1条，多条会报错）
9. selectList，使用LambdaQueryWrapper，以使用Lambda表达式（好处：仿误写，如果是普通方式，传入数据库字段名，如果写错了就会报错，Lambda表达式使用方法引用来获取字段信息）

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class RetrieveTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 根据主键Id查询
     */
    @Test
    public void selectById() {
        User user = userMapper.selectById(1094590409767661570L);
        System.out.println(user);
    }

    /**
     * 一次使用多个Id进行查询
     */
    @Test
    public void selectIds() {
        List<Long> idList = Arrays.asList(1088248166370832385L, 1288460195017072641L, 1094590409767661570L);
        List<User> userList = userMapper.selectBatchIds(idList);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用Mao存放查询字段和条件来进行查询
     */
    @Test
    public void selectByMap() {
        //Map存放查询条件，注意key存放的是数据库的字段名，而不是实体中的变量名
        HashMap<String, Object> columnMap = new HashMap<>();
        columnMap.put("name", "王天风");
        columnMap.put("age", 25);
        List<User> userList = userMapper.selectByMap(columnMap);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 需求：
     * 名字中包含"雨"，并且年龄小于40
     * sql: name like %雨% and age < 40
     */
    @Test
    public void selectByWrapper() {
        //直接创建一个条件构造器，获取使用Wrappers工具类
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //QueryWrapper<User> queryWrapper = Wrappers.query();
        queryWrapper
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 需求：
     * 名字中包含"雨"，并且年龄大于等于20，且小于等于40，并且email不为空
     * sql: name like '%雨%' and age between 20 and 40 and email is not null
     */
    @Test
    public void selectByWrapper2() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //年龄大于20，并且小于40
                .between("age", 20, 40)
                //不为null
                .isNotNull("email");
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 需求：
     * 名字为王姓，或者年龄大于等于25，按照年龄降序排列，年龄相同按照id升序排列
     * name like '王%' or age>=25 order by age desc,id asc
     */
    @Test
    public void selectByWrapper3() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //模糊查询，只包含右边一个%，注意这里的字段都是数据库字段，而不是实体的变量名
                .likeRight("name", "王")
                .or()
                //年龄大于等于25
                .ge("age", 25)
                //先按年龄降序排（从大到小）
                .orderByDesc("age")
                //年龄相同的，再按id的升序排（从小到大）
                .orderByAsc("id");
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 创建日期为2019年2月14日，并且直属上级为名字为王姓
     * sql: date_format(create_time,'%Y-%m-%d')='2019-02-14' and manager_id in (select id from user where name like '王%')
     */
    @Test
    public void selectByWrapper4() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //直接用，不使用占位符，可能会有sql注入的风险
                //.apply("date_format(create_time,'%Y-%m-%d')=2019-02-14")
                //apply占位符查询，目的是为了防止sql注入
                .apply("date_format(create_time,'%Y-%m-%d')={0}", "2019-02-14")
                //inSql子查询
                .inSql("manager_id", "select id from mp_user where name like '王%'");
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 名字为王姓并且（年龄小于40或邮箱不为空）
     * sql: name like '王%' and (age<40 or email is not null)
     */
    @Test
    public void selectByWrapper5() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.likeRight("name", "王")
                //函数式编程
                //.and(wrapper-> wrapper.lt("age", 40).or().isNotNull("email"))
                .and(new Function<QueryWrapper<User>, QueryWrapper<User>>() {
                    @Override
                    public QueryWrapper<User> apply(QueryWrapper<User> wrapper) {
                        //年龄小于40
                        return wrapper.lt("age", 40)
                                //或者邮箱不为空
                                .or()
                                .isNotNull("email");
                    }
                });

        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 名字为王姓，或者（年龄小于40并且年龄大于20并且邮箱不为空）
     * sql: name like '王%' or (age<40 and age>20 and email is not null)
     */
    @Test
    public void selectByWrapper6() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //王姓开头
                .likeRight("name", "王")
                //or()使用Function参数的，将获取年龄小于40，并且年龄大于20
                .or(wrapper -> wrapper.lt("age", 40)
                        .gt("age", 20)
                        //并且邮箱不为空的条件用括号包裹起来
                        .isNotNull("email"));
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * （年龄小于40或邮箱不为空）并且名字为王姓
     * sql: (age<40 or email is not null) and name like '王%'
     */
    @Test
    public void selectByWrapper7() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //nested()，嵌套，就是加括号
                .nested(wrapper -> wrapper.lt("age", 40).or().isNotNull("email"))
                //邮箱不为null
                .likeRight("name", "王");
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 年龄为30、31、34、35
     * sql: age in (30、31、34、35)
     */
    @Test
    public void selectByWrapper8() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .in("age", Arrays.asList(30, 31, 34, 35));
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 9、只返回满足条件的其中一条语句即可
     * sql: limit 1
     */
    @Test
    public void selectByWrapper9() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //字符串拼接到sql，会有sql注入的风险
        queryWrapper.last("limit 1");
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 需求：
     * select中字段不全部出现的查询，例如只查询出id和姓名（默认会查询出实体中的所有字段）
     * sql: select id,name from user where name like '%雨%' and age<40
     */
    @Test
    public void selectByWrapperSupper() {
        //直接创建一个条件构造器，获取使用Wrappers工具类
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //QueryWrapper<User> queryWrapper = Wrappers.query();
        queryWrapper
                //<重点>，相比上面的selectByWrapper，多了调用select()方法，传入需要查询的列名
                .select("id", "name")
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用条件构造器进行查询
     * <p>
     * 需求：
     * select中字段不全部出现的查询，例如只查询出id和姓名、年龄、邮箱（默认会查询出实体中的所有字段）
     * sql: select id,name,age,email from user where name like '%雨%' and age<40
     */
    @Test
    public void selectByWrapperSupper2() {
        //直接创建一个条件构造器，获取使用Wrappers工具类
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //QueryWrapper<User> queryWrapper = Wrappers.query();
        queryWrapper
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40)
                //如果字段比较多，我们每个都写上会比较麻烦，我们可以使用排除法，毕竟只是去掉少量的字段，其他字段都保留
                //参数一：实体类的Class
                //参数二：Predicate函数式接口，test()方法返回boolean，表示是否保留当前遍历到的字段，返回true代表需要，false代表不需要
                .select(User.class, info -> !info.getColumn().equals("create_time")
                        && !info.getColumn().equals("manager_id"));
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 测试动态条件
     */
    @Test
    public void testCondition() {
        String name = "王";
        String email = "";
        condition(name, email);
    }

    /**
     * 查询条件，条件可传可不传
     *
     * @param name  姓名
     * @param email 邮箱
     */
    private void condition(String name, String email) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //手动判空后，加入条件
//        if (StringUtils.isNotEmpty(name)) {
//            queryWrapper.like("name", name);
//        }
//        if (StringUtils.isNotEmpty(email)) {
//            queryWrapper.like("email", email);
//        }
        //上面不够优雅，代码量大
        queryWrapper.like(StringUtils.isNotEmpty(name), "name", name)
                .like(StringUtils.isNotEmpty(email), "email", email);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 使用实体Entity中的字段作为查询条件
     */
    @Test
    public void selectByWrapperEntity() {
        //使用实体作为查询条件加入到where中
        User whereUser = new User();
        whereUser.setReadName("刘雨红");
        whereUser.setAge(32);
        QueryWrapper<User> queryWrapper = new QueryWrapper<>(whereUser);
        //再给查询条件加like等操作也是可以的
        queryWrapper
                .like("name", "雨")
                .lt("age", 40);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User result : userList) {
            System.out.println(result);
        }
    }

    /**
     * 使用map作为sql的查询条件，map中的所有非空属性会作为sql的等于条件来拼接
     */
    @Test
    public void selectByWrapperAllEq() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        Map<String, Object> params = new HashMap<>();
        params.put("name", "王天风");
        //age参数，如果为null，会给生成的sql加上age is null，如果想过滤掉为null的属性字段，则将allEq的null2IsNull属性设置为false，默认为true
        params.put("age", null);
        //queryWrapper.allEq(params, false);

        //函数式方式，传入BiPredicate过滤器，test()方法返回当前遍历到的键值对是否加入到条件中，返回true表示加入条件中，返回false代表不加入到条件中
        queryWrapper.allEq(new BiPredicate<String, Object>() {
            @Override
            public boolean test(String key, Object value) {
                //例如过滤掉name的字段
                return !key.equals("name");
            }
        }, params);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User result : userList) {
            System.out.println(result);
        }
    }

    /**
     * 查询，返回列表，但列表里面的元素不是实体，而是一个Map，每个Map就是一条记录的所有属性以键值对的形式存在
     * 当我们查询的字段相比实体字段少很多的时候，使用实体去存放，会有很多属性是null，不是很优雅，我们使用Map存放会更加明确有什么属性和值
     */
    @Test
    public void selectByWrapperMaps() {
        //直接创建一个条件构造器，获取使用Wrappers工具类
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //QueryWrapper<User> queryWrapper = Wrappers.query();
        queryWrapper
                .select("id", "name")
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40);
        List<Map<String, Object>> userList = userMapper.selectMaps(queryWrapper);
        for (Map<String, Object> map : userList) {
            System.out.println(map);
        }
    }

    /**
     * 按照直属上级分组，查询每组的平均年龄、最大年龄、最小年龄。并且只取年龄总和小于500的组。
     * sql:
     * select avg(age) avg_age,min(age) min_age,max(age) max_age
     * from user
     * group by manager_id
     * having sum(age) <500
     */
    @Test
    public void selectByWrapperMaps2() {
        //直接创建一个条件构造器，获取使用Wrappers工具类
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //QueryWrapper<User> queryWrapper = Wrappers.query();
        queryWrapper
                //字段起别名：数据库字段名 别名
                .select("avg(age) avg_age", "min(age) min_age", "max(age) max_age")
                .groupBy("manager_id").having("sum(age) < {0}", 500);

        List<Map<String, Object>> userList = userMapper.selectMaps(queryWrapper);
        for (Map<String, Object> map : userList) {
            System.out.println(map);
        }
    }

    /**
     * selectObjs()，只拿出数据的第一列的数据，其他列都被舍弃（sql中是会查询其他字段的，但是selectObjs()方法只选择第一列的数据）
     * 场景：只返回第一列的时候可以用它
     */
    @Test
    public void selectByWrapperObjs() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .select("id", "name")
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40);
        List<Object> userList = userMapper.selectObjs(queryWrapper);
        for (Object o : userList) {
            System.out.println(o);
        }
    }

    /**
     * 统计查询
     */
    @Test
    public void selectByWrapperCount() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                //会帮我们在sql中添加 COUNT( 1 )，所以我们就不能使用select()方法来指定要查询的列了，否则会作为count的参数来使用
                //.select("id", "name")
                //除非你想count其他字段，就可以使用
                .select("id")
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .like("name", "雨")
                //小于
                .lt("age", 40);
        Integer count = userMapper.selectCount(queryWrapper);
        System.out.println("count：" + count);
    }

    /**
     * 只查询出1条数据（必须查询结果只有1条，多条会报错）
     */
    @Test
    public void selectByWrapperOne() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .select("id", "name")
                //模糊查询，注意这里的字段都是数据库字段，而不是实体的变量名
                .eq("name", "刘雨红");
        User user = userMapper.selectOne(queryWrapper);
        System.out.println(user);
    }

    /**
     * Lambda条件构造器
     * 好处：仿误写，如果是普通方式，传入数据库字段名，如果写错了就会报错，Lambda表达式使用方法引用来获取字段信息
     */
    @Test
    public void selectByWrapperLambda() {
        //Lambda条件构造器的3种创建方式
//        LambdaQueryWrapper<User> queryWrapper = new QueryWrapper<User>().lambda();
//        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        LambdaQueryWrapper<User> queryWrapper = Wrappers.lambdaQuery();
        queryWrapper
                //where name like '%雨%'
                .like(User::getReadName, "雨")
                //and age < 40
                .lt(User::getAge, 40);
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 查询，姓名为王姓，并且（年龄小于40岁或者邮箱不为空）
     */
    @Test
    public void selectByWrapperLambda2() {
        LambdaQueryWrapper<User> queryWrapper = Wrappers.lambdaQuery();
        queryWrapper
                //where name like '%王%'
                .likeRight(User::getReadName, "王")
                //and (age < 40 or email is not null)
                .and(lqw -> lqw.lt(User::getAge, 40).or().isNotNull(User::getEmail));
        List<User> userList = userMapper.selectList(queryWrapper);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 查询，姓名为王姓，并且年龄大于等于20
     */
    @Test
    public void selectByWrapperLambda3() {
        LambdaQueryChainWrapper<User> chainWrapper = new LambdaQueryChainWrapper<>(userMapper);
        List<User> userList = chainWrapper
                //姓名为王姓
                .like(User::getReadName, "雨")
                //年龄大于等于20
                .ge(User::getAge, 20)
                .list();
        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

- 分页查询

BaseMapper还提供了分页查询的相关方法。

1. selectPage，传入分页参数Page对象，以及QueryWrapper条件构造器，返回IPage对象，IPage内有总页数、总记录数等分页参数
2. selectMapsPage，第二种分页方式，分页对象的泛型是Map，不是实体类对象，就是将实体类的字段和值封装到了Map中
3. selectPage，传入的Page分页对象，构造方法配置isSearchCount参数为false，表示分页但不查询总记录数，在不需要总记录数的场景下，可以减少一次查询，例如App端，加载更多是不需要总记录数做判断的

```
/**
 * 分页查询，分页对象的泛型是实体类
 */
@Test
public void selectPage() {
    //直接创建一个条件构造器，获取使用Wrappers工具类
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2);

    //分页
    IPage<User> iPage = userMapper.selectPage(page, queryWrapper);

    System.out.println("总页数：" + iPage.getPages());
    System.out.println("总记录数：" + iPage.getTotal());
    List<User> userList = iPage.getRecords();
    for (User user : userList) {
        System.out.println(user);
    }
}

/**
 * 第二种分页方式，分页对象的泛型是Map
 */
@Test
public void selectPage2() {
    //直接创建一个条件构造器，获取使用Wrappers工具类
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2);

    //第二种分页，但返回的IPage泛型类型是Map类型，就是将实体类的字段和值封装到了Map中
    IPage<Map<String, Object>> iPage = userMapper.selectMapsPage(page, queryWrapper);

    System.out.println("总页数：" + iPage.getPages());
    System.out.println("总记录数：" + iPage.getTotal());
    List<Map<String, Object>> userList = iPage.getRecords();
    for (Map<String, Object> map : userList) {
        System.out.println(map);
    }
}

/**
 * 分页，不查询总记录数（默认会查，会查询总数和分页查询，会查询2次，例如不断上拉加载的场景是不需要查总记录数的，就可以不进行查询）
 */
@Test
public void selectPage3() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    //isSearchCount传false，代表不进行查询总记录数，少一次查询
    Page<User> page = new Page<>(1, 2, false);

    //分页
    IPage<User> iPage = userMapper.selectPage(page, queryWrapper);

    System.out.println("总页数：" + iPage.getPages());
    List<User> userList = iPage.getRecords();
    for (User user : userList) {
        System.out.println(user);
    }
}
```

## 自定义Mapper方法

### 提供自定义Mapper方法

给UserMapper增加2个自定义方法。

自定义方法有2种方式，注解和XML，这里主要用XML，但也提供了一条注解，因为他们不能同时存在，所以注释掉，有需要时，再参考即可

1. selectAll，自定义查询参数，查询条件由传入的wrapper提供，参数要加上@Param注解，并且参数名是固定的，都是Constants.WRAPPER
2. selectUserPage，自定义查询参数，带分页功能，需要传个2参数，Page对象和Wrapper对象，同样参数要加上@Param注解，并且参数名是固定的，都是Constants.WRAPPER

```
public interface UserMapper extends BaseMapper<User> {
    /**
     * 自定义Sql查询，让Wrapper中配置的条件和自定义sql进行组合
     */
    //@Select("select * from mp_user ${ew.customSqlSegment}")
    List<User> selectAll(@Param(Constants.WRAPPER) Wrapper<User> wrapper);

    /**
     * 自定义Sql，进行分页查询
     */
    IPage<User> selectUserPage(Page<User> page, @Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
```

### 配置Mapper的XML文件

有自定义Mapper接口，要么用注解，要么就用XML，一般会使用XML。
在resource目录下，建立多层文件夹，com/imooc/mybatisplus.mapper，文件夹内新建XML文件：UserMapper.xml

注意：Mapper接口和对应XML文件的名称要相同！以及命名空间是Mapper的全类名

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace，命名空间必须和Dao接口一样 -->
<mapper namespace="com.imooc.mybatisplus.dao.UserMapper">
    <!-- 查询所有 -->
    <select id="selectAll" resultType="User">
        select * from mp_user ${ew.customSqlSegment}
    </select>

    <!-- 分页查询 -->
    <select id="selectUserPage" resultType="User">
        select * from mp_user ${ew.customSqlSegment}
    </select>
</mapper>
```

### 测试自定义SQL的方法

条件在Wrapper实例中配置，查询的字段由自定义的SQL语句提供。

分别是查询selectAll()所有和selectMyPage()分页查询。

```
/**
 * 自定义sql
 */
@Test
public void selectMy() {
    LambdaQueryWrapper<User> lambdaQuery = Wrappers.lambdaQuery();
    lambdaQuery
            //where name like '%王%'
            .likeRight(User::getReadName, "王")
            //and (age < 40 or email is not null)
            .and(lqw -> lqw.lt(User::getAge, 40).or().isNotNull(User::getEmail));
    List<User> userList = userMapper.selectAll(lambdaQuery);
    for (User user : userList) {
        System.out.println(user);
    }
}

/**
 * 自定义查询分页
 */
@Test
public void selectMyPage() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2);
    //自定义分页查询
    IPage<User> iPage = userMapper.selectUserPage(page, queryWrapper);
    System.out.println("总页数：" + iPage.getPages());
    System.out.println("总记录数：" + iPage.getTotal());
    List<User> userList = iPage.getRecords();
    for (User user : userList) {
        System.out.println(user);
    }
}
```

## 通用Service

上面我们使用的是通用Mapper，而一般我们会有Service组合多个Mapper来完成业务逻辑。

MyBatis-Plus也给我们提供了通用Service的方案。

### 建立Service接口和实现

建立service包，UserService接口，继承IService接口，泛型写实体类的类型

```
public interface UserService extends IService<User> {
}
```

建立UserServiceImpl实现类，继承ServiceImpl，泛型传入Mapper接口和实体类类型

```
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

### 测试

1. getOne，获取1个结果，如果有结果有多个，会报错。如果不想报错，默认取第一个，则将后面的throwEx属性设置为false
2. saveBatch，保存多个对象到数据库表，如果实体对象id字段有值，则为更新，为null，则代表新增
3. lambdaQuery，调用list，链式编程查询
4. lambdaUpdate，调用update，链式编程更新
5. lambdaUpdate，调用remove，链式编程删除

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ServiceTest {
    @Autowired
    private UserService userService;

    /**
     * 获取一个结果
     */
    @Test
    public void getOne() {
        //查询1个，如果有多个结果，会报错。
        //后面的throwEx属性代表如果查询出多个，是否抛出异常，默认为true，如果有多个就抛出异常
        //如果设置为false，则不抛出异常，打印警告，并且拿取第一个返回
        User result = userService.getOne(Wrappers.<User>lambdaQuery()
                .gt(User::getAge, 25), false);
        System.out.println(result);
    }

    /**
     * 批量插入
     */
    @Test
    public void batch() {
        User user1 = new User();
        user1.setReadName("徐丽1");
        user1.setAge(28);

        User user2 = new User();
        user2.setReadName("徐丽2");
        user2.setAge(29);
        List<User> userList = Arrays.asList(user1, user2);
        boolean isSuccess = userService.saveBatch(userList);
        System.out.println("是否批量插入成功：" + isSuccess);
    }

    /**
     * 批量操作，如果设置了id，则先查询，有则更新，无则做插入
     */
    @Test
    public void batch2() {
        User user1 = new User();
        user1.setReadName("徐丽3");
        user1.setAge(28);

        User user2 = new User();
        user2.setUserId(1289004540795330562L);
        user2.setReadName("徐力");
        user2.setAge(30);
        List<User> userList = Arrays.asList(user1, user2);
        boolean isSuccess = userService.saveOrUpdateBatch(userList);
        System.out.println("是否批量插入成功：" + isSuccess);
    }

    /**
     * 链式编程，查询
     */
    @Test
    public void chain() {
        List<User> userList = userService.lambdaQuery()
                .gt(User::getAge, 25)
                .like(User::getReadName, "雨")
                .list();
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * 链式编程，更新
     */
    @Test
    public void chain2() {
        boolean isSuccess = userService.lambdaUpdate()
                .eq(User::getAge, 25)
                .set(User::getAge, 26)
                .update();
        System.out.println("是否更新成功：" + isSuccess);
    }

    /**
     * 链式编程，删除
     */
    @Test
    public void chain3() {
        boolean isSuccess = userService.lambdaUpdate()
                .eq(User::getAge, 24)
                .remove();
        System.out.println("是否删除成功：" + isSuccess);
    }
}
```

## AR模式

AR模式，即ActiveRecord模式，用实体类操作增删查改，需要对实体类做修改。

1. 实体类继承Model类，泛型传入实体类的类型
2. 生成序列化的UID

```
@Data
@EqualsAndHashCode(callSuper = false)
@TableName("mp_user")
public class User extends Model<User> {
    private static final long serialVersionUID = 1L;

    //...省略字段，和之前的没区别
}
```

### 测试

1. insert，插入数据
2. selectById，根据实体上的主键查询，也可以在方法参数上传入
3. updateById，根据实体主键更新实体上设置的数据
4. deleteById，根据实体主键删除，也可以在方法参数上传入
5. insertOrUpdate，新增或更新，根据实体主键是否有值判断，有值则为更新， 为null则为新增

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ARTest {
    /**
     * 插入数据
     */
    @Test
    public void insert() {
        User user = new User();
        user.setReadName("刘花");
        user.setAge(29);
        user.setEmail("lh@baomidou.com");
        user.setManagerId(1088248166370832385L);
        user.setCreateTime(LocalDateTime.now());
        boolean isSuccess = user.insert();
        System.out.println("是否插入成功：" + isSuccess);
    }

    /**
     * 按主键进行查询
     */
    @Test
    public void selectById() {
        User user = new User();
        User result = user.selectById(1288460195017072641L);
        System.out.println(result);
    }

    /**
     * 按实体类上的主键属性进行查询
     */
    @Test
    public void selectById2() {
        User user = new User();
        user.setUserId(1288839790966910978L);
        User result = user.selectById();
        System.out.println(result);
    }

    /**
     * 更新，按实体类上的主键属性
     */
    @Test
    public void updateById() {
        User user = new User();
        user.setUserId(1288839790966910978L);
        user.setReadName("刘草");
        boolean isSuccess = user.updateById();
        System.out.println("是否更新成功：" + isSuccess);
    }

    /**
     * 按主键删除
     */
    @Test
    public void deleteById() {
        User user = new User();
        boolean isSuccess = user.deleteById(1288839790966910978L);
        System.out.println("是否删除成功：" + isSuccess);
    }

    /**
     * 按实体类上的主键属性进行删除
     */
    @Test
    public void deleteById2() {
        User user = new User();
        user.setUserId(1288839790966910978L);
        boolean isSuccess = user.deleteById();
        System.out.println("是否删除成功：" + isSuccess);
    }

    /**
     * 先查询主键Id的记录，如果有则更新，无则新增
     */
    @Test
    public void insertOrUpdate() {
        User user = new User();
        //设置了主键Id，则会先查询，有记录则更新，无则删除
        //user.setUserId(1288842439342780417L);
        user.setReadName("张强");
        user.setAge(29);
        user.setEmail("lh@baomidou.com");
        user.setManagerId(1088248166370832385L);
        user.setCreateTime(LocalDateTime.now());
        boolean isSuccess = user.insertOrUpdate();
        System.out.println("是否插入成功：" + isSuccess);
    }
}
```