#### SpringBoot整合MyBatisPlus

近年来互联网应用的兴起，MyBatis已逐渐替代Hibernate，而MyBatis作为半ORM，有些单表操作需要手写SQL比较麻烦，开发效率会比Hibernate低。
随后MyBatisPlus出现了，提供了通用的Mapper和单表操作，补足了这个缺点，本篇就用它来和SpringBoot整合来做一个项目。

- 准备数据

本文使用的案例，是一个用户表，提供以下字段

- id：用户id
- name：用户名称
- age：用户年龄
- email：用户邮箱

```
/*
 Navicat Premium Data Transfer

 Source Server         : mac
 Source Server Type    : MySQL
 Source Server Version : 50716
 Source Host           : localhost:3306
 Source Schema         : haoke

 Target Server Type    : MySQL
 Target Server Version : 50716
 File Encoding         : 65001

 Date: 02/07/2020 16:56:05
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name` varchar(30) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
BEGIN;
INSERT INTO `user` VALUES (1, 'Jone', 18, 'test1@baomidou.com');
INSERT INTO `user` VALUES (2, 'Jack', 20, 'test2@baomidou.com');
INSERT INTO `user` VALUES (3, 'Tom', 28, 'test3@baomidou.com');
INSERT INTO `user` VALUES (4, 'Sandy', 21, 'test4@baomidou.com');
INSERT INTO `user` VALUES (5, '罗', 18, '383030391@qq.com');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

- 添加依赖

MyBatisPlus的对SpringBoot的支持是提供了一个starter，叫mybatis-plus-boot-starter，我们使用这个依赖，MyBatis原本的依赖就不需要引入了

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <!-- SpringBoot测试 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--简化代码的工具包-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!--mybatis-plus的springboot支持-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.5</version>
    </dependency>

    <!--mysql驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

- 用户实体类

由于使用lombok插件，可以生成构造方法和属性的get、set，如果不想用，去掉并自己提供即可

```
/**
 * User表实体
 */
@NoArgsConstructor
@AllArgsConstructor
@Slf4j
@Data
public class User {
    /**
     * Id自增长
     */
    @TableId(value = "ID", type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

- 配置文件

resources目录下，新建一个application.yml文件

1. 配置数据源给spring
2. 配置mybatis，配置模型类独的包名，以及mapper的路径，后续的Mapper.xml，都放在resources下的一个叫mapper的文件夹下即可

```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/haoke?useunicode=true&characterEncoding=utf8
    username: root
    password: hezihao123

mybatis:
  type-aliases-package: cn.itheima.mybatisplus.pojo
  mapper-locations: classpath:mapper/*Mapper.xml
```

- 新建Mapper接口

新建UserMapper接口，继承BaseMapper通用Mapper，传入模型到泛型即可，BaseMapper中提供了通用的单表操作

```
/**
 * 用户表的Dao层接口
 */
public interface UserMapper extends BaseMapper<User> {
}
```

- 启动类

1. 使用@MapperScan注解，配置扫描mapper的包
2. 提供分页拦截器，PaginationInterceptor

```
//配置扫描mapper的包
@MapperScan("cn.itcast.mybatisplus.mapper")
//表示为SpringBoot应用
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    /**
     * 提供分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

- 测试CRUD

新建UserMapperTest测试类，测试基本的CRUD

```
/**
 * MyBatisPlus测试
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    /**
     * 基础查询
     */
    @Test
    public void testSelect() {
        List<User> userList = userMapper.selectList(null);
        for (User user : userList) {
            System.out.println(user);
        }
    }

    /**
     * Like查询
     */
    @Test
    public void testSelectByLike() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>(new User());
        //查询名字中包含“o”的用户
        queryWrapper.like("name", "o");
        List<User> users = userMapper.selectList(queryWrapper);
        for (User user : users) {
            System.out.println(user);
        }
    }

    /**
     * 条件查询
     */
    @Test
    public void testSelectByLe() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>(new User());
        //查询年龄小于等于20的用户
        queryWrapper.le("age", 20);
        List<User> users = userMapper.selectList(queryWrapper);
        for (User user : users) {
            System.out.println(user);
        }
    }

    /**
     * 测试保存
     */
    @Test
    public void testSave() {
        User user = new User();
        user.setName("子和");
        user.setAge(24);
        user.setEmail("hezihao@126.com");
        int row = userMapper.insert(user);
        if (row > 0) {
            System.out.println("保存成功");
        } else {
            System.out.println("保存失败");
        }
    }

    /**
     * 测试删除数据
     */
    @Test
    public void testDelete() {
        int row = userMapper.deleteById(6L);
        if (row > 0) {
            System.out.println("删除成功");
        } else {
            System.out.println("删除失败");
        }
    }

    /**
     * 测试更新数据
     */
    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(5L);
        user.setName("罗");
        user.setAge(18);
        user.setEmail("383030391@qq.com");
        int row = userMapper.updateById(user);
        if (row > 0) {
            System.out.println("修改成功");
        } else {
            System.out.println("修改失败");
        }
    }
}
```

- 分页

```
/**
 * 测试分页
 */
@Test
public void testSelectPage() {
    IPage<User> page = new Page<>(1, 2);
    IPage<User> userPage = userMapper.selectPage(page, null);
    System.out.println("总条数：---------->" + userPage.getTotal());
    System.out.println("当前页数：---------->" + userPage.getCurrent());
    System.out.println("当前每页显示数量：---------->" + userPage.getSize());
    List<User> users = userPage.getRecords();
    for (User user : users) {
        System.out.println(user);
    }
}
```