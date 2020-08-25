# JPA学习记录

## JPA是什么

Java Persistence API：用于对象持久化的 API。是Java EE5.0平台中Sun为了统一持久层ORM框架而制定的一套标准，注意是一套标准，而不是具体实现。

## JPA和Hibernate的关系

JPA 是 hibernate 的一个抽象（就像JDBC和JDBC驱动的关系）。
JPA 是规范：JPA 本质上就是一种  ORM 规范，不是ORM 框架 —— 
因为 JPA 并未提供 ORM 实现，它只是制订了一些规范，提供了一些编程的API接口，但具体实现则由ORM厂商提供实现。
Hibernate 是实现：Hibernate 除了作为 ORM 框架之外，它也是一种JPA实现。
从功能上来说， JPA 是 Hibernate 功能的一个子集。

## JPA简单使用

### pom文件

主要是Hibernate对JPA的实现依赖，导入它，JPA的规范接口也会连着导入。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itcast</groupId>
    <artifactId>jpa_sample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.hibernate.version>5.0.7.Final</project.hibernate.version>
    </properties>

    <dependencies>
        <!-- junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!-- hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>${project.hibernate.version}</version>
        </dependency>

        <!-- c3p0 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-c3p0</artifactId>
            <version>${project.hibernate.version}</version>
        </dependency>

        <!-- log日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!-- Mysql and MariaDB -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
    </dependencies>
</project>
```

### JPA配置文件

在`resources`目录下，建立一个名为`META-INF`的文件夹，新建一个名为`persistence.xml`的文件，`META-INF`和这个文件名字是固定的。

配置文件主要配置`事务类型`、`JPA的Hibernate实现类`、`数据库4大要素`、以及`Hibernate的可选配置`

```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
    http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">
    <!--
    配置持久化单元
        name：持久化单元名称
        transaction-type：事务管理的方式
            JPA：分布式事务管理
            RESOURCE_LOCAL：本地事务管理
    -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!-- 必选：配置JPA的实现方式 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!-- 必选：配置数据库信息 -->
        <properties>
            <!-- 数据库驱动、地址、用户名、密码 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/jpa"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="hezihao123"/>
            <!-- 可选：
                配置JPA实现方的配置信息
                hibernate.show_sql：显示sql
                hibernate.hbm2ddl.auto：创建数据库表
                    create：程序运行时创建数据库表，如果有表，先删除表，再创建表
                    update：程序运行时创建表，如果有表，不会创建
                    none：不会创建表
            -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

### 导入数据库配置

库名：jpa

有5张表，如下：

- cst_customer：客户表
- cst_linkman：联系人表
- sys_role：角色表
- sys_user：用户表
- sys_user_role：用户和角色关系中间表

表关系：

- cst_customer客户表，和cst_linkman联系人表是一对多关系，一个客户可以有多个联系人
- sys_role角色表，和sys_user用户表是多对多关系，一个用户可以有多个角色

现在简单使用，只用到`cst_customer`客户表，其他表后面再使用。

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for cst_customer
-- ----------------------------
DROP TABLE IF EXISTS `cst_customer`;
CREATE TABLE `cst_customer` (
  `cust_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `cust_address` varchar(255) DEFAULT NULL,
  `cust_industry` varchar(255) DEFAULT NULL,
  `cust_level` varchar(255) DEFAULT NULL,
  `cust_name` varchar(255) DEFAULT NULL,
  `cust_phone` varchar(255) DEFAULT NULL,
  `cust_source` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`cust_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of cst_customer
-- ----------------------------
BEGIN;
INSERT INTO `cst_customer` VALUES (1, NULL, 'it教育', 'vip', '传智播客', NULL, NULL);
COMMIT;

-- ----------------------------
-- Table structure for cst_linkman
-- ----------------------------
DROP TABLE IF EXISTS `cst_linkman`;
CREATE TABLE `cst_linkman` (
  `lkm_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `lkm_email` varchar(255) DEFAULT NULL,
  `lkm_gender` varchar(255) DEFAULT NULL,
  `lkm_memo` varchar(255) DEFAULT NULL,
  `lkm_mobile` varchar(255) DEFAULT NULL,
  `lkm_name` varchar(255) DEFAULT NULL,
  `lkm_phone` varchar(255) DEFAULT NULL,
  `lkm_position` varchar(255) DEFAULT NULL,
  `lkm_cust_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`lkm_id`),
  KEY `FKh9yp1nql5227xxcopuxqx2e7q` (`lkm_cust_id`),
  CONSTRAINT `FKh9yp1nql5227xxcopuxqx2e7q` FOREIGN KEY (`lkm_cust_id`) REFERENCES `cst_customer` (`cust_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of cst_linkman
-- ----------------------------
BEGIN;
INSERT INTO `cst_linkman` VALUES (1, NULL, NULL, NULL, NULL, '小王', NULL, NULL, 1);
INSERT INTO `cst_linkman` VALUES (2, NULL, NULL, NULL, NULL, '小李', NULL, NULL, 1);
COMMIT;

-- ----------------------------
-- Table structure for sys_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `role_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`role_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of sys_role
-- ----------------------------
BEGIN;
INSERT INTO `sys_role` VALUES (2, 'Java程序员');
INSERT INTO `sys_role` VALUES (3, 'Java程序员');
COMMIT;

-- ----------------------------
-- Table structure for sys_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user` (
  `user_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `age` int(11) DEFAULT NULL,
  `user_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of sys_user
-- ----------------------------
BEGIN;
INSERT INTO `sys_user` VALUES (2, NULL, '小李');
INSERT INTO `sys_user` VALUES (3, NULL, '小李');
COMMIT;

-- ----------------------------
-- Table structure for sys_user_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_user_role`;
CREATE TABLE `sys_user_role` (
  `sys_user_id` bigint(20) NOT NULL,
  `sys_role_id` bigint(20) NOT NULL,
  PRIMARY KEY (`sys_role_id`,`sys_user_id`),
  KEY `FKsbjvgfdwwy5rfbiag1bwh9x2b` (`sys_user_id`),
  CONSTRAINT `FK1ef5794xnbirtsnudta6p32on` FOREIGN KEY (`sys_role_id`) REFERENCES `sys_role` (`role_id`),
  CONSTRAINT `FKsbjvgfdwwy5rfbiag1bwh9x2b` FOREIGN KEY (`sys_user_id`) REFERENCES `sys_user` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of sys_user_role
-- ----------------------------
BEGIN;
INSERT INTO `sys_user_role` VALUES (2, 2);
INSERT INTO `sys_user_role` VALUES (3, 3);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 实体类

- @Entity：声明为数据库表的Entity实体
- @Table：建立实体类和表的映射关系
- @Id：配置实体属性为主键字段
- @GeneratedValue：配置主键的生成策略
    - GenerationType.IDENTITY：自增长，mysql
    - GenerationType.SEQUENCE：序列。要求：数据库必须支持序列，例如oracle，mysql不支持序列
    - GenerationType.TABLE：jpa提供的一种机制，通过一张数据库表的形式帮我们完成自增
    - GenerationType.AUTO：由程序自动帮我们选择主键生成策略，IDENTITY或SEQUENCE
- @Column：指定实体类属性和数据库表之间的对应关系
    - name：指定数据库表的列名称。
    - unique：是否唯一
    - nullable：是否可以为空
    - inserttable：是否可以插入
    - updateable：是否可以更新
    - columnDefinition: 定义建表时创建此列的DDL
    - secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境[重点]

```
/**
 * 客户表实体
 * 所有的注解都是使用JPA的规范提供的注解
 * 所以在导入注解包的时候，一定要导入javax.persistence下的
 */
//声明为数据库表的Entity实体
@Entity
//建立实体类和表的映射关系
@Table(name = "cst_customer")
public class Customer {
    //配置为主键
    @Id
    /**
     * 配置主键的生成策略
     *      GenerationType.IDENTITY：自增长，mysql
     *      GenerationType.SEQUENCE：序列
     *          要求：数据库必须支持序列，例如oracle，mysql不支持序列
     *      GenerationType.TABLE：jpa提供的一种机制，通过一张数据库表的形式帮我们完成自增
     *      GenerationType.AUTO：由程序自动帮我们选择主键生成策略，IDENTITY或SEQUENCE
     */
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    /**
     * 指定映射关系
     *
     * 作用：指定实体类属性和数据库表之间的对应关系
     *    	属性：
     *      	name：指定数据库表的列名称。
     *      	unique：是否唯一
     *      	nullable：是否可以为空
     *      	inserttable：是否可以插入
     *      	updateable：是否可以更新
     *      	columnDefinition: 定义建表时创建此列的DDL
     *      	secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境[重点]
     */
    @Column(name = "cust_id")
    /**
     * 客户编号(主键）
     */
    private Long custId;
    /**
     * 客户名称(公司名称)
     */
    @Column(name = "cust_name")
    private String custName;
    /**
     * 客户信息来源
     */
    @Column(name = "cust_source")
    private String custSource;
    /**
     * 客户所属行业
     */
    @Column(name = "cust_industry")
    private String custIndustry;
    /**
     * 客户级别
     */
    @Column(name = "cust_level")
    private String custLevel;
    /**
     * 客户联系地址
     */
    @Column(name = "cust_address")
    private String custAddress;
    /**
     * 客户联系电话
     */
    @Column(name = "cust_phone")
    private String custPhone;

    //省略get、set、toString方法
}
```

###工具类

Jpa工具类，解决实体管理器工厂浪费资源和耗时问题。

原理：通过静态代码块，第一次访问工具类时，创建一个公共实体管理器工厂，后续通过getEntityManager()方法，获取实体管理器即可。

```
public class JpaUtil {
    private static EntityManagerFactory factory;

    static {
        //加载配置文件，获取实体管理器工厂
        factory = Persistence.createEntityManagerFactory("myJpa");
    }

    /**
     * 获取实体管理器
     */
    public static EntityManager getEntityManager() {
        return factory.createEntityManager();
    }
}
```

### 增删查改

CRUD的步骤都是类似的，只有第四步不同。

- 步骤：
1. 加载配置文件，创建实体管理器工厂
2. 通过实体管理器工厂，获取实体管理器
3. 获取事务对象，开启事务
4. 执行增删改
5. 提交事务或回滚事务
6. 释放资源

#### 增

```
/**
 * 测试Jpa的保存
 * 步骤：
 * 1.加载配置文件，创建实体管理器工厂
 * 2.通过实体管理器工厂，获取实体管理器
 * 3.获取事务对象，开启事务
 * 4.执行增删改
 * 5.提交事务或回滚事务
 * 6.释放资源
 */
@Test
public void testSave() {
    //1.加载配置文件，创建实体管理器工厂，传入持久化单元名称
    //EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
    //2.通过实体管理器工厂，获取实体管理器
    //EntityManager entityManager = factory.createEntityManager();

    EntityManager entityManager = JpaUtil.getEntityManager();

    //3.获取事务对象，开启事务
    EntityTransaction tx = entityManager.getTransaction();//获取事务对象
    tx.begin();//开启事务
    //4.执行增删改，保存一个客户到数据库
    Customer customer = new Customer();
    customer.setCustName("传智播客");
    customer.setCustIndustry("教育");
    //保存操作
    entityManager.persist(customer);
    //5.提交事务
    tx.commit();
    //6.释放资源
    entityManager.close();
    //factory.close();
}
```

#### 删

```
/**
 * 测试删除
 */
@Test
public void testRemove() {
    //1.获取EntityManager
    EntityManager entityManager = JpaUtil.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.执行操作
    //先查询
    Customer customer = entityManager.find(Customer.class, 3L);
    //再删除
    entityManager.remove(customer);
    System.out.println(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

#### 查

查询有2种，find()和getReference()

- find()
    - 查询对象就是当前客户对象本身
    - 在调用find()方法时，就会发送sql语句
- getReference()
    - 获取的对象是一个动态代理对象
    - 调用getReference()方法，不会立即发送sql语句查询数据库
    - 当调用查询结果对象的属性的时候，才会发送查询sql，什么时候用，什么时候发送sql语句

```
/**
 * 测试根据Id查询客户，使用find()
 * 使用find()方法进行查询
 * 1.查询对象就是当前客户对象本身
 * 2.在调用find()方法时，就会发送sql语句
 * <p>
 * 立即加载
 */
@Test
public void testFind() {
    //1.获取EntityManager
    EntityManager entityManager = JpaUtil.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.执行操作
    /**
     * find()，根据Id查询数据
     *  1.class：查询数据的结果，封装到哪个实体类
     *  2.id：查询的主键值
     */
    Customer customer = entityManager.find(Customer.class, 3L);
    System.out.println(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}

/**
 * 测试根据Id查询客户，使用getReference()
 * 使用使用getReference()方法进行查询
 * 1.获取的对象是一个动态代理对象
 * 2.调用getReference()方法，不会立即发送sql语句查询数据库
 * 3.当调用查询结果对象的属性的时候，才会发送查询sql，什么时候用，什么时候发送sql语句
 * 延迟加载（懒加载）
 */
@Test
public void testReference() {
    //1.获取EntityManager
    EntityManager entityManager = JpaUtil.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.执行操作
    /**
     * getReference()，根据Id查询数据
     *  1.class：查询数据的结果，封装到哪个实体类
     *  2.id：查询的主键值
     */
    Customer customer = entityManager.getReference(Customer.class, 3L);
    System.out.println(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

#### 改

```
/**
 * 测试更新
 */
@Test
public void testUpdate() {
    //1.获取EntityManager
    EntityManager entityManager = JpaUtil.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.执行操作
    //先查询
    Customer customer = entityManager.find(Customer.class, 3L);
    customer.setCustIndustry("it教育");
    //再更新
    entityManager.merge(customer);
    System.out.println(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

## JPA整合Spring使用

一般我们会配合Spring一起使用，而不会单独使用，所以接下来就整合Spring使用吧。

### pom文件

引入Spring相关类库、以及SpringDataJPA。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itcast</groupId>
    <artifactId>spring-jpa</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.version>4.2.4.RELEASE</spring.version>
        <hibernate.version>5.0.7.Final</hibernate.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <c3p0.version>0.9.1.2</c3p0.version>
        <mysql.version>5.1.6</mysql.version>
    </properties>

    <dependencies>
        <!-- junit单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- spring beg -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.8</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- spring end -->

        <!-- hibernate beg -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <!-- hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.1.Final</version>
        </dependency>
        <!-- hibernate end -->

        <!-- c3p0 beg -->
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>${c3p0.version}</version>
        </dependency>
        <!-- c3p0 end -->

        <!-- log end -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- log end -->

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>1.9.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- el beg 使用spring data jpa 必须引入 -->
        <dependency>
            <groupId>javax.el</groupId>
            <artifactId>javax.el-api</artifactId>
            <version>2.2.4</version>
        </dependency>

        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>javax.el</artifactId>
            <version>2.2.4</version>
        </dependency>
        <!-- el end -->
    </dependencies>
</project>
```

### Spring配置文件

1. jpa相关配置，使用`LocalContainerEntityManagerFactoryBean`，Spring整合后的Bean，以及hibernate特殊配置
2. 配置数据库4大要素，以及连接池
3. 配置dao层接口扫描
4. 配置事务管理器、配置spring声明式事务
5. 配置Spring AOP
6. 配置包扫描

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!-- spring和spring data jpa配置文件 -->

    <!-- 1.EntityManagerFactory，交给spring管理 -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <!-- 配置数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置扫描实体类的包 -->
        <property name="packagesToScan" value="cn.itcast.domain"/>
        <!-- 配置jpa的实现，Hibernate -->
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>
        <!-- jpa的供应商适配器 -->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!-- 配置是否自动创建数据库表 -->
                <property name="generateDdl" value="false"/>
                <!-- 指定数据库类型 -->
                <property name="database" value="MYSQL"/>
                <!-- 数据库方言，支持的特有语法 -->
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect"/>
                <!-- 是否显示sql语句 -->
                <property name="showSql" value="true"/>
            </bean>
        </property>
        <!-- jpa的方言，高级特性 -->
        <property name="jpaDialect">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect"/>
        </property>
    </bean>

    <!-- 2.创建数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql:///jpa"/>
        <property name="user" value="root"/>
        <property name="password" value="hezihao123"/>
    </bean>

    <!-- 3.整合spring data jpa，base-package配置扫描dao层接口 -->
    <jpa:repositories base-package="cn.itcast.dao" transaction-manager-ref="transactionManager"
                      entity-manager-factory-ref="entityManagerFactory"/>

    <!-- 4.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!-- 配置spring声明式事务 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 5.配置AOP-->
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* cn.itcast.service.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
    </aop:config>

    <!-- 配置包扫描 -->
    <context:component-scan base-package="cn.itcast"/>
</beans>
```

### 实体类

和之前的一样，不再赘述了

### Dao层

Customer表的Dao层，Jpa的Dao接口需要继承2个接口，`JpaRepository`接口，`JpaSpecificationExecutor`接口。

`JpaRepository`接口，提供一些基础的查询方法，第一个泛型是实体类的类型，第二个是主键Id的类型。

`JpaSpecificationExecutor`接口，提供一些支持条件的查询方法，泛型为实体类的类型。

```
public interface CustomerDao extends JpaRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
}
```

### 测试CRUD

`JpaRepository`接口继承于`PagingAndSortingRepository`接口，而`PagingAndSortingRepository`又继承于`CrudRepository`接口。

`CrudRepository`接口，提供了一些基础的增删查改，例如`findOne()`查询一个、`save()`新增或更新一条记录、`delete()`删除一条记录等。

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class CustomerDaoTest {
    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据id查询
     */
    @Test
    public void testFindOne() {
        Customer customer = customerDao.findOne(3L);
        System.out.println(customer);
    }

    /**
     * 测试保存
     * save：保存或更新
     * 根据传入的对象是否有id主键属性，如果有，则为更新，如没有就是新增
     * 更新：先查询，再更新
     */
    @Test
    public void testSave() {
        Customer customer = new Customer();
        customer.setCustName("黑马程序员");
        customer.setCustLevel("vip");
        customer.setCustIndustry("it教育");
        customerDao.save(customer);
    }

    /**
     * 更新
     */
    @Test
    public void testUpdate() {
        Customer customer = customerDao.findOne(5L);
        customer.setCustName("黑马程序员2");
        customerDao.save(customer);
    }

    /**
     * 测试删除
     */
    @Test
    public void testDelete() {
        customerDao.delete(3L);
    }

    /**
     * 查询所有
     */
    @Test
    public void testFindAll() {
        List<Customer> list = customerDao.findAll();
        for (Customer customer : list) {
            System.out.println(customer);
        }
    }

    /**
     * 测试统计查询：查询客户总数量
     */
    @Test
    public void testCount() {
        long count = customerDao.count();
        System.out.println(count);
    }

    /**
     * 测试，判断id为4的客户是否存在
     */
    @Test
    public void testExists() {
        boolean isExists = customerDao.exists(4L);
        System.out.println(isExists);
    }

    /**
     * 根据id查询
     * 注解@Transactional，作用：保证getOne()正常运行
     *
     * findOne()：使用了em.find()，立即加载
     * getOne()：使用了em.getReference()，延迟加载
     */
    @Test
    @Transactional
    public void testGetOne() {
        Customer customer = customerDao.getOne(4L);
        System.out.println(customer);
    }
}
```

### JPQL

除了基础的ORM操作，在复杂查询中，难免需要自己写SQL，而JPA提供了JPQL给我们查询。

我们在Dao上，加相关注解写上JPQL。

注释写得很清楚，就不再写一遍了，主要是查询操作用JPQL比较多。

```
public interface CustomerDao extends JpaRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
    /**
     * 简单案例
     * <p>
     * 使用Jpql语句进行查询
     * jpql：from Customer where custName = ?
     */
    @Query(value = "from Customer where custName = ?")
    Customer findJpql(String custName);

    /**
     * 多占位符案例
     * <p>
     * 根据客户名称和客户id，查询客户
     * jpql：from Customer where custName = ? and custId = ?
     * <p>
     * 对于多个占位符参数
     * 赋值的时候，默认情况下，占位符的位置需要和方法参数中的位置保持一致
     * <p>
     * 可以指定占位符参数的位置
     * ?索引的方式，指定此占位符的取值，例如参数调换位置，from Customer where custName = ?2 and custId = ?1
     */
//    @Query(value = "from Customer where custName = ?2 and custId = ?1")
//    List<Customer> findCustNameAndId(Long id,String name);
    @Query(value = "from Customer where custName = ? and custId = ?")
    List<Customer> findCustNameAndId(String name, Long id);

    /**
     * 使用Jpql完成更新数据
     * <p>
     * 根据id，更新客户的名称
     * <p>
     * sql：update cust_customer set cust_name =? where cust_id = ?
     * jpql：update Customer set custName = ? where custId = ?
     * <p>
     * 注解@Query是用来查询用的，我们的操作是更新，所以需要再增加一个注解
     * 增加@Modifying注解，表示当前的方法是一个更新操作
     */
    @Query(value = "update Customer set custName = ?2 where custId = ?1")
    @Modifying
    void updateCustomer(long id, String custName);

    /**
     * 使用sql语句查询
     * sql：select * from cst_customer
     * <p>
     * 使用sql语句查询，还是使用@Query注解，但需要设置nativeQuery属性为true，代表value中的是sql语句，默认值为false，就是代表是jpql
     * <p>
     * 注解@Query：配置查询
     * value：sql语句或jpql
     * nativeQuery：查询方式，true为sql，false为jpql
     */
//    @Query(value = "select * from cst_customer", nativeQuery = true)
//    List<Object[]> findSql();
    @Query(value = "select * from cst_customer where cust_name like ?1", nativeQuery = true)
    List<Object[]> findSql(String name);

    /**
     * 约定方式，方法按规则进行命名，就无须使用@Query写语句进行查询
     * findBy开头：查询，后面是对象属性名（首字母大学），查询条件
     * <p>
     * findByCustName()，根据客户名称查询
     * <p>
     * 在运行阶段，会根据方法名称进行解析，findBy from xxx（实体类） 属性名称 where custName = ?
     * <p>
     * 规则：
     *
     * 注意：如果是精准匹配，查询方式可以不写！
     * 1.精确查询：findBy + 属性名称（根据属性名称进行精确匹配的查询，就是=等于）
     * 2.模糊查询：findBy + 属性名称 + 查询方式（Like | isnull），例如：findByCustNameLike
     * 3.多条件查询：
     *      findBy + 属性名 + 查询方式 + 多条件的连接符（and|or） + 属性名 + 查询方式
     */
    Customer findByCustName(String custName);

    /**
     * 模糊查询
     * findBy + 属性名称 + 查询方式（Like | isnull）
     * 模糊查询：findByCustNameLike
     */
    List<Customer> findByCustNameLike(String custName);

    /**
     * 使用客户名称模糊查询，和客户所属行业精准匹配的查询
     */
    Customer findByCustNameLikeAndCustIndustry(String custName, String custIndustry);
}
```

### 测试JPQL

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class JpqlTest {
    @Autowired
    private CustomerDao customerDao;

    /**
     * 测试基本查询
     */
    @Test
    public void testFindJpql() {
        Customer customer = customerDao.findJpql("黑马程序员");
        System.out.println(customer);
    }

    /**
     * 测试多占位符
     */
    @Test
    public void testFindCustNameAndId() {
        List<Customer> customers = customerDao.findCustNameAndId("小码哥", 4L);
        for (Customer customer : customers) {
            System.out.println(customer);
        }
    }

    /**
     * 测试Jpql的更新操作
     * Spring Data jpa，执行更新、删除操作，必须使用事务，就是添加@Transactional注解
     * 但默认执行完毕后，回滚事务，所以需要手动指定不自动回滚
     * <p>
     * 注解@Rollback，value值，true自动回滚，false不自动回滚
     */
    @Test
    @Transactional//添加事务支持，否则执行会报错
    @Rollback(value = false)//设置不自动回滚
    public void testUpdateCustomer() {
        customerDao.updateCustomer(5L, "黑马程序员666");
    }

    /**
     * 测试使用sql查询
     */
    @Test
    public void testFindSql() {
        List<Object[]> list = customerDao.findSql("黑马程序员");
        for (Object[] objects : list) {
            System.out.println(Arrays.toString(objects));
        }
    }

    /**
     * 测试，按方法命名规则查询（精确查询）
     */
    @Test
    public void testFindByCustName() {
        Customer customer = customerDao.findByCustName("黑马程序员");
        System.out.println(customer);
    }

    /**
     * 测试，按方法命名规则模糊查询（模糊查询）
     */
    @Test
    public void testFindByCustNameLike() {
        List<Customer> customers = customerDao.findByCustNameLike("黑马程序员%");
        for (Customer customer : customers) {
            System.out.println(customer);
        }
    }

    /**
     * 测试，使用客户名称模糊查询，和客户所属行业精准匹配的查询（多条件查询）
     */
    @Test
    public void testFindByCustNameLikeAndCustIndustry() {
        Customer customer = customerDao.findByCustNameLikeAndCustIndustry("黑马程序员%", "it教育");
        System.out.println(customer);
    }
}
```

### 条件查询

Dao层接口还继承了一个`JpaSpecificationExecutor`接口，它提供一些条件查询。

主要使用`Specification`对象作为条件对象，复写`toPredicate()`方法，其中有2个对象，我们需要了解。

- Root：获取需要查询的对象属性，通过它可以获取指定key属性的值
- CriteriaBuilder：构造查询条件的对象（相当于条件构造器），内部封装了很多查询条件（模糊查询、精准匹配）

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class SpecTest {
    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据查询，查询单个对象
     */
    @Test
    public void testSpec1() {
        /**
         * 自定义查询条件，创建Specification实例，复写toPredicate()方法，返回查询条件
         * toPredicate()方法参数解释：
         *  1.Root，获取需要查询的对象属性
         *  2.CriteriaBuilder，构造查询条件的，内部封装了很多查询条件（模糊查询、精准匹配）
         *
         *  需求：根据客户名称查询，查询客户名称为黑马程序员的客户
         *  查询条件
         *      1.查询方式，CriteriaBuilder对象
         *      2.比较的属性名称，root对象
         */
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                //1.获取比较的属性
                Path<Object> custName = root.get("custName");
                //2.构造查询条件，select * from cst_customer where cust_name = '黑马程序员'
                //进行精确匹配（参数：比较的属性，比较的属性的值）
                return cb.equal(custName, "黑马程序员");
            }
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }

    /**
     * 多条件查询
     * 需求：根据客户名"黑马程序员"，和客户所属行业（it教育）
     */
    @Test
    public void testSpec2() {
        /**
         * root：获取属性
         *      客户名
         *      所属行业
         * cb：构造查询
         *      1.构造客户名的精准匹配查询
         *      2.构造所属行业的精准匹配查询
         *      3.将以上2个查询联系起来
         */
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                //客户名
                Path<Object> custName = root.get("custName");
                //所属行业
                Path<Object> custIndustry = root.get("custIndustry");
                //构造查询
                Predicate p1 = cb.equal(custName, "黑马程序员");
                Predicate p2 = cb.equal(custIndustry, "it教育");
//                return cb.or(p1, p2);//或关系
                //联合多个查询条件，满足条件1和条件2，与关系
                return cb.and(p1, p2);
            }
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }

    /**
     * 模糊查询
     * 需求：根据客户的名称模糊匹配，返回客户列表
     * <p>
     * equal：直接获取path对象，直接传入进行比较即可
     * gt，lt，ge，like：得到path对象，根据path指定比较的参数类型，再去比较
     * 指定参数类型：path.as(类型的字节码对象)；
     */
    @Test
    public void testSpec3() {
        //构造查询条件
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                Path<Object> custName = root.get("custName");
                return cb.like(custName.as(String.class), "黑马程序员%");
            }
        };
        /**
         * 排序
         *
         * 参数一：
         * Sort.Direction.DESC：倒序
         * Sort.Direction.ASC：正序
         * 参数二：
         * 需要按照排序的字段
         */
        Sort sort = new Sort(Sort.Direction.DESC, "custId");
        List<Customer> list = customerDao.findAll(specification, sort);
//        List<Customer> list = customerDao.findAll(specification);
        for (Customer customer : list) {
            System.out.println(customer);
        }
    }

    /**
     * 分页查询
     * <p>
     * Pageable分页参数
     * findAll(Specification, Pageable)：带有条件的分页
     * findAll(Pageable)：没有条件的分页
     * <p>
     * 返回Page对象
     * Pageable接口，实现类PageRequest
     * 构造方法：
     * 1.第一个参数：当前查询的页数（从0开始）
     * 2.第二参数：每页查询的数量
     */
    @Test
    public void testSpec4() {
        Page<Customer> pages = customerDao.findAll(null, new PageRequest(0, 2));
        //获取总条数
        long totalElements = pages.getTotalElements();
        System.out.println("总条数 totalElements：" + totalElements);
        //获取总页数
        int totalPages = pages.getTotalPages();
        System.out.println("总页数 totalPages：" + totalPages);
        //获取数据集合
        List<Customer> list = pages.getContent();
        System.out.println("数据集合 list：" + list);
    }
}
```

## 一对多查询

前面提到`cst_customer客户表`，和`cst_linkman联系人表`是一对多关系，一个客户可以有多个联系人。

那么在实体上，就可以表达表之间的关系，`Customer`客户表实体类、`LinkMan`联系人表实体类。

### Customer客户表实体类

主要是`linkmans`字段，引用它的多个联系人，JPA推荐我们存储多的一方的实体列表，用Set集合存储。

- `@OneToMany`注解，声明关系，targetEntity对方的实体类类型
    - mappedBy属性：放弃维护外键权，指定维护外键的那一方中使用@JoinColumn注解的属性名
    - cascade属性：级联属性（可以配置到设置多表映射关系的注解上）
        - CascadeType.ALL：所有
        - CascadeType.MERGE：更新
        - CascadeType.PERSIST：保存
        CascadeType.REMOVE：删除
    - fetchType：配置关联对象的加载方式
        - EAGER：立即加载，如果是单个对象，使用立即加载
        - LAZY：延迟加载（懒加载），一般如果是列表，会使用延迟加载
        
- `@JoinColumn`注解，配置外键，name：外键字段名称，referencedColumnName：参照的主表的主键名称
    - 一般让多的一方来使用这个注解，而不会让一的一方来使用，也不会2个同时使用

```
/**
 * 客户表实体
 * 所有的注解都是使用JPA的规范提供的注解
 * 所以在导入注解包的时候，一定要导入javax.persistence下的
 */
//声明为数据库表的Entity实体
@Entity
//建立实体类和表的映射关系
@Table(name = "cst_customer")
public class Customer {
    //配置为主键
    @Id
    /**
     * 配置主键的生成策略
     *      GenerationType.IDENTITY：自增长，mysql
     *      GenerationType.SEQUENCE：序列
     *          要求：数据库必须支持序列，例如oracle，mysql不支持序列
     *      GenerationType.TABLE：jpa提供的一种机制，通过一张数据库表的形式帮我们完成自增
     *      GenerationType.AUTO：由程序自动帮我们选择主键生成策略，IDENTITY或SEQUENCE
     */
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    /**
     * 指定映射关系
     *
     * 作用：指定实体类属性和数据库表之间的对应关系
     *    	属性：
     *      	name：指定数据库表的列名称。
     *      	unique：是否唯一
     *      	nullable：是否可以为空
     *      	inserttable：是否可以插入
     *      	updateable：是否可以更新
     *      	columnDefinition: 定义建表时创建此列的DDL
     *      	secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境[重点]
     */
    @Column(name = "cust_id")
    /**
     * 客户编号(主键）
     */
    private Long custId;
    /**
     * 客户名称(公司名称)
     */
    @Column(name = "cust_name")
    private String custName;
    /**
     * 客户信息来源
     */
    @Column(name = "cust_source")
    private String custSource;
    /**
     * 客户所属行业
     */
    @Column(name = "cust_industry")
    private String custIndustry;
    /**
     * 客户级别
     */
    @Column(name = "cust_level")
    private String custLevel;
    /**
     * 客户联系地址
     */
    @Column(name = "cust_address")
    private String custAddress;
    /**
     * 客户联系电话
     */
    @Column(name = "cust_phone")
    private String custPhone;
    /**
     * 一对多关系映射：客户和联系人，一个客户可有多个联系人
     * <p>
     * 注解@OneToMany：声明关系，targetEntity对方的实体类类型
     * 注解@JoinColumn：配置外键，name：外键字段名称，referencedColumnName：参照的主表的主键名称
     */
    //由于一的一方可以维护外键，会发送多一条update语句，所以放弃维护外键权
//    @OneToMany(targetEntity = LinkMan.class)
    //联系人表的主键lkm_cust_id，引用该表的主键cust_id
//    @JoinColumn(name = "lkm_cust_id", referencedColumnName = "cust_id")

    /**
     * 放弃维护外键权，需要使用mappedBy属性，指定维护外键的那一方中使用@JoinColumn注解的属性名
     *
     * mappedBy属性：对方配置关系的属性名称
     * cascade属性：级联属性（可以配置到设置多表映射关系的注解上）
     *      1.CascadeType.ALL：所有
     *      2.CascadeType.MERGE：更新
     *      3.CascadeType.PERSIST：保存
     *      4.CascadeType.REMOVE：删除
     *
     *  fetchType：配置关联对象的加载方式
     *      1.EAGER：立即加载
     *      2.LAZY：延迟加载（懒加载）
     */
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    private Set<LinkMan> linkmans = new HashSet<LinkMan>(0);

    //省略get、set、toString方法
}
```

### LinkMan联系人表实体类

主要是`customer`属性，引用它的客户实体。

- `@ManyToOne`：配置多对一关系，targetEntity：对象的实体类的类型
- `@JoinColumn`：配置外键，name：外键名称，referencedColumnName参照的主表的主键名称

```
@Entity
@Table(name = "cst_linkman")
public class LinkMan {
    /**
     * 主键，联系人编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "lkm_id")
    private Long lkmId;
    /**
     * 联系人姓名
     */
    @Column(name = "lkm_name")
    private String lkmName;
    /**
     * 联系人性别
     */
    @Column(name = "lkm_gender")
    private String lkmGender;
    /**
     * 联系人办公电话
     */
    @Column(name = "lkm_phone")
    private String lkmPhone;
    /**
     * 联系人手机
     */
    @Column(name = "lkm_mobile")
    private String lkmMobile;
    /**
     * 联系人邮箱
     */
    @Column(name = "lkm_email")
    private String lkmEmail;
    /**
     * 联系人职位
     */
    @Column(name = "lkm_position")
    private String lkmPosition;
    /**
     * 联系人备注
     */
    @Column(name = "lkm_memo")
    private String lkmMemo;
    /**
     * 多对一关系映射：多个联系人对应客户
     * 注解@ManyToOne：配置多对一关系，targetEntity：对象的实体类的类型
     * 注解@JoinColumn，配置外键，name：外键名称，referencedColumnName参照的主表的主键名称
     */
    @ManyToOne(targetEntity = Customer.class, fetch = FetchType.LAZY)
    //外键lkm_cust_id，引用主表的cust_id
    @JoinColumn(name = "lkm_cust_id", referencedColumnName = "cust_id")
    private Customer customer;

    //省略get、set、toString方法
}
```

### Dao层

Jpa的Dao接口需要继承2个接口，JpaRepository接口，JpaSpecificationExecutor接口

#### 客户表Dao层接口

Customer表的Dao层

```
public interface CustomerDao extends JpaRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
}
```

#### 联系人表的Dao层

```
public interface LinkManDao extends JpaRepository<LinkMan, Long>, JpaSpecificationExecutor<LinkMan> {
}
```

### 一对多测试

因为有一对多关系，实体保存时，一的一方，需要加上多的一方的实体列表。

以及级联操作，例如删除一个客户时，它的所有联系人都一起删除。

注意：JPA的测试，要加上事务，并且不自动回滚，否则默认是回滚的，虽然方法执行成功了，数据库是看不到效果的。

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class OneTwoManyTest {
    @Autowired
    private CustomerDao customerDao;
    @Autowired
    private LinkManDao linkManDao;

    /**
     * 测试，保存一个客户，保存一个联系人
     */
    @Test
    @Transactional//配置事务
    @Rollback(value = false)//不自动回滚
    public void testAdd() {
        Customer customer = new Customer();
        customer.setCustName("百度");
        LinkMan linkMan = new LinkMan();
        linkMan.setLkmName("小李");
        //向客户中，添加一个联系人
        //从客户的角度：发送2条insert语句，再发送一条update更新语句，更新外键（由于我们配置了客户到联系人的关系，客户可以对外键进行维护）
        Set<LinkMan> linkmans = customer.getLinkmans();
        linkmans.add(linkMan);
        //保存客户
        customerDao.save(customer);
        //保存联系人
        linkManDao.save(linkMan);
    }

    @Test
    @Transactional//配置事务
    @Rollback(value = false)//不自动回滚
    public void testAdd2() {
        Customer customer = new Customer();
        customer.setCustName("百度");
        LinkMan linkMan = new LinkMan();
        linkMan.setLkmName("小李");

        //联系人中，添加一个客户
        //配置联系人到客户的关系，只发送了2条insert语句（由于我们配置了联系人到客户的映射关系（多对一））
        linkMan.setCustomer(customer);

        //保存客户
        customerDao.save(customer);
        //保存联系人
        linkManDao.save(linkMan);
    }

    /**
     * 会有一条多余的update语句
     *  1.由于一的一方可以维护外键，会发送多一条update语句
     *  2.要解决此问题，只需要在一的一方放弃维护权即可
     */
    @Test
    @Transactional//配置事务
    @Rollback(value = false)//不自动回滚
    public void testAdd3() {
        Customer customer = new Customer();
        customer.setCustName("百度");
        LinkMan linkMan = new LinkMan();
        linkMan.setLkmName("小李");

        //由于配置了多的一方到一的一方的关联关系（当保存的时候，已经对外键进行赋值）
        linkMan.setCustomer(customer);
        //由于配置一的一方到多的一方的关联关系（发送一条update语句）
        customer.getLinkmans().add(linkMan);

        //保存客户
        customerDao.save(customer);
        //保存联系人
        linkManDao.save(linkMan);
    }

    /**
     * 级联添加，保存一个客户的同时，保存客户的所有联系人
     * 需要在操作主体的实体类上，配置cascade
     */
    @Test
    @Transactional//配置事务
    @Rollback(value = false)//不自动回滚
    public void testCascadeAdd() {
        Customer customer = new Customer();
        customer.setCustName("百度1");
        LinkMan linkMan = new LinkMan();
        linkMan.setLkmName("小李1");
        //设置关系
        linkMan.setCustomer(customer);
        customer.getLinkmans().add(linkMan);
        //保存客户，同时保存客户的所有联系人
        customerDao.save(customer);
    }

    /**
     * 级联删除，删除1号客户的同时，删除1号客户的所有联系人
     */
    @Test
    @Transactional//配置事务
    @Rollback(value = false)//不自动回滚
    public void testCascadeRemove() {
        //1.查询1号客户
        Customer customer = customerDao.findOne(1L);
        //2.删除1号客户
        customerDao.delete(customer);
    }
}
```

### 对象导航查询测试

对象导航查询：查询一个对象时，通过此对象，查询所有关联的对象

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class ObjectQueryTest {
    @Autowired
    private CustomerDao customerDao;
    @Autowired
    private LinkManDao linkManDao;

    /**
     * 测试对象导航查询（查询一个对象时，通过此对象，查询所有关联的对象）
     * 添加注解@Transactional，解决could not initialize proxy - no Session错误
     */
    @Test
    @Transactional
    public void testQuery1() {
        //查询1号客户
        Customer customer = customerDao.getOne(1L);
        //对象导航查询
        Set<LinkMan> linkmans = customer.getLinkmans();
        for (LinkMan linkman : linkmans) {
            System.out.println(linkman);
        }
    }

    /**
     * 对象导航查询
     * 导航到的属性是多的一方（一个集合），默认使用的是延迟加载的形式查询，也就是说调用get()方法并不会立即发送查询，而是在使用关联对象时，才会查询
     * 如果不想延迟加载，我们只需要修改配置，将延迟加载改为立即加载即可
     * fetch：需要配置到多表映射关系的注解上，@OneToMany、@ManyToMany，这里去Customer类中添加
     */
    @Test
    @Transactional
    public void testQuery2() {
        //查询1号客户
        Customer customer = customerDao.findOne(1L);
        //对象导航查询
        Set<LinkMan> linkmans = customer.getLinkmans();
        System.out.println(linkmans.size());
    }

    /**
     * 从联系人对象导航到他所属的客户
     * 导航到的属性是一的一方（单个对象属性），默认使用的是立即加载的方式查询
     */
    @Test
    @Transactional
    public void testQuery3() {
        //查询1号客户
        LinkMan linkMan = linkManDao.findOne(1L);
        Customer customer = linkMan.getCustomer();
        System.out.println(customer);
    }
}
```

## 多对多查询

前面提到，`sys_role`角色表，和`sys_user`用户表是多对多关系，一个用户可以有多个角色，那实体类之间是怎么表达的呢？

### Role角色表实体

- `@ManyToMany`注解，配置多对多的映射关系
    - mappedBy属性，放弃维护权，属性值为维护方的属性名
- `@JoinTable`注解，配置外键，一般在维护的那一方加
    - joinColumns属性：当前对象在中间表的外键
    - inverseJoinColumns：对方对象在中间表的外键

```
@Entity
@Table(name = "sys_role")
public class Role {
    /**
     * 角色Id
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long id;
    /**
     * 角色名称
     */
    @Column(name = "role_name")
    private String roleName;
    /**
     * 配置多对多的映射关系
     */
//    @ManyToMany(targetEntity = User.class)
//    @JoinTable(name = "sys_user_role",
//            //joinColumns，当前对象在中间表的外键
//            joinColumns = {@JoinColumn(name = "sys_role_id", referencedColumnName = "role_id")},
//            //inverseJoinColumns，对方对象在中间表的外键
//            inverseJoinColumns = {@JoinColumn(name = "sys_user_id", referencedColumnName = "user_id")})
    //放弃维护权，mappedBy属性：指定维护方的属性名
    @ManyToMany(mappedBy = "roles")
    private Set<User> users = new HashSet<User>();

    //省略get、set、toString方法
}
```

### 用户表实体

- `@ManyToMany`注解：多对多关系
    - targetEntity属性：代表对方的实体类字节码
- `@JoinTable`注解
    - name属性：中间表的名称
    - joinColumns属性（数组）：
        - name：配置当前对象在中间表的外键
        - referencedColumnName：参照的主表的主键名
    - inverseJoinColumns属性（数组）：配置对象对象在中间表的位置
        - name：配置当前对象在中间表的外键
        - referencedColumnName：参照的主表的主键名

```
@Entity
@Table(name = "sys_user")
public class User {
    /**
     * 用户Id
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    private Long id;
    /**
     * 用户名称
     */
    @Column(name = "user_name")
    private String userName;
    /**
     * 用户年龄
     */
    @Column(name = "age")
    private Integer age;
    /**
     * 配置用户到角色的多对多关系
     * 1.配置多对多的映射关系
     *  注解@ManyToMany：多对多关系，targetEntity：代表对方的实体类字节码
     * 2.配置中间表（包含2个外键）
     *
     * 注解@JoinTable：
     *  name属性：中间表的名称
     *  joinColumns属性（数组）：
     *      name：配置当前对象在中间表的外键
     *      referencedColumnName：参照的主表的主键名
     *  inverseJoinColumns属性（数组）：配置对象对象在中间表的位置
     *      name：配置当前对象在中间表的外键
     *      referencedColumnName：参照的主表的主键名
     */
    @ManyToMany(targetEntity = Role.class, cascade = CascadeType.ALL)
    @JoinTable(name = "sys_user_role",
            //joinColumns，当前对象在中间表的外键
            joinColumns = {@JoinColumn(name = "sys_user_id", referencedColumnName = "user_id")},
            //inverseJoinColumns，对方对象在中间表的外键
            inverseJoinColumns = {@JoinColumn(name = "sys_role_id", referencedColumnName = "role_id")})
    private Set<Role> roles = new HashSet<Role>();

    //省略get、set、toString方法
}
```

### Dao层接口

#### 角色表Dao

```
public interface RoleDao extends JpaRepository<Role, Long>, JpaSpecificationExecutor<Role> {
}
```

#### 用户表Dao

```
public interface UserDao extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
}
```

### 多对多测试

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class ManyToManyTest {
    @Autowired
    private UserDao userDao;
    @Autowired
    private RoleDao roleDao;

    /**
     * 保存一个用户，保存一个角色
     * 多对多放弃维护权：被动的一方，放弃维护权
     */
    @Test
    @Transactional
    @Rollback(value = false)
    public void testAdd() {
        User user = new User();
        user.setUserName("小李");
        Role role = new Role();
        role.setRoleName("Java程序员");

        //配置用户到角色的关系，可以对中间表的数据进行维护
        user.getRoles().add(role);
        //配置角色到用户的关系，可以对中间表的数据进行维护，2方都可以维护，会导致主键重复，而抛出异常，所以需要有一方放弃维护权
        role.getUsers().add(user);

        userDao.save(user);
        roleDao.save(role);
    }

    /**
     * 测试级联添加，保存一个用户的同时，保存用户关联的角色
     */
    @Test
    @Transactional
    @Rollback(value = false)
    public void testCascadeAdd() {
        User user = new User();
        user.setUserName("小李");
        Role role = new Role();
        role.setRoleName("Java程序员");

        //配置用户到角色的关系，可以对中间表的数据进行维护
        user.getRoles().add(role);
        //配置角色到用户的关系，可以对中间表的数据进行维护，2方都可以维护，会导致主键重复，而抛出异常，所以需要有一方放弃维护权
        role.getUsers().add(user);

        userDao.save(user);
    }

    /**
     * 级联删除，删除id为1的用户，同时删除他关联的角色
     */
    @Test
    @Transactional
    @Rollback(value = false)
    public void testCascadeDelete() {
        //查询1号用户
        User user = userDao.findOne(1L);
        //删除1号用户
        userDao.delete(user);
    }
}
```

## 总结

文章可能描述得不是很清楚，代码我上传到了[Github](https://github.com/hezihaog/jpa_sample)，有兴趣的同学，可以clone。