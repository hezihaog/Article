# MyBatis使用

本篇来学习MyBatis的入门使用。

### 准备工作

- 导入数据库表和数据

1. 数据库名为eesy_mybatis。
2. account表，用户账户表（账户表和用户表时多对一关系）
3. user表，用户表
4. role表，角色表
5. user_role，用户和角色中间表（用户和角色是多对多关系）

```
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime default NULL COMMENT '生日',
  `sex` char(1) default NULL COMMENT '性别',
  `address` varchar(256) default NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`username`,`birthday`,`sex`,`address`) values (41,'老王','2018-02-27 17:47:08','男','北京'),(42,'小二王','2018-03-02 15:09:37','女','北京金燕龙'),(43,'小二王','2018-03-04 11:34:34','女','北京金燕龙'),(45,'传智播客','2018-03-04 12:04:06','男','北京金燕龙'),(46,'老王','2018-03-07 17:37:26','男','北京'),(48,'小马宝莉','2018-03-08 11:44:00','女','北京修正');

DROP TABLE IF EXISTS `account`;

CREATE TABLE `account` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `UID` int(11) default NULL COMMENT '用户编号',
  `MONEY` double default NULL COMMENT '金额',
  PRIMARY KEY  (`ID`),
  KEY `FK_Reference_8` (`UID`),
  CONSTRAINT `FK_Reference_8` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `account`(`ID`,`UID`,`MONEY`) values (1,41,1000),(2,45,1000),(3,41,2000);

DROP TABLE IF EXISTS `role`;

CREATE TABLE `role` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `ROLE_NAME` varchar(30) default NULL COMMENT '角色名称',
  `ROLE_DESC` varchar(60) default NULL COMMENT '角色描述',
  PRIMARY KEY  (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `role`(`ID`,`ROLE_NAME`,`ROLE_DESC`) values (1,'院长','管理整个学院'),(2,'总裁','管理整个公司'),(3,'校长','管理整个学校');

DROP TABLE IF EXISTS `user_role`;

CREATE TABLE `user_role` (
  `UID` int(11) NOT NULL COMMENT '用户编号',
  `RID` int(11) NOT NULL COMMENT '角色编号',
  PRIMARY KEY  (`UID`,`RID`),
  KEY `FK_Reference_10` (`RID`),
  CONSTRAINT `FK_Reference_10` FOREIGN KEY (`RID`) REFERENCES `role` (`ID`),
  CONSTRAINT `FK_Reference_9` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user_role`(`UID`,`RID`) values (41,1),(45,1),(41,2);
```

- 导入依赖

1. mybatis：3.4.5
2. mysql驱动：5.1.6
3. log4j：1.2.12
4. junit：4.10

```
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
        <scope>runtime</scope>
    </dependency>
    <!-- 日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.12</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

- 导入log4j配置文件

resources目录下，创建log4j配置文件，log4j.properties

```
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=d:\axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```

- 创建数据库配置文件

resources目录下，创建数据库配置文件，jdbcConfig.properties

数据库，我们使用mysql，库名为：eesy_mybatis
用户名和密码使用大家自己的

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy_mybatis
jdbc.username=root
jdbc.password=hezihao123
```

- 创建MyBatis配置文件

resources目录下，创建MyBatis配置文件，SqlMapConfig.xml

1. MyBatis的所有配置，都放在configuration标签内
2. properties标签，引入刚才的mysql配置，jdbcConfig.properties
3. typeAliases标签，配置模型别名，可以使用typeAlias标签单独指定，但一般我们会使用package标签，指定统一的一个包
4. environments标签，配置数据库环境，标签内部可以指定多个数据源，default属性指定要使用的，这里我们就一个mysql
5. transactionManager标签，指定事务类型，我们使用MyBatis封装了JDBC，所以我们也是使用JDBC
6. dataSource标签，指定数据源，也是指定连接池，POOLED为MyBatis内置的连接池，UNPOOLED为不使用连接池，一般我们都会使用POOLED
7. mappers标签，resource属性指定mapper.xml文件路径，resource属性指定注解类的类名。一般不用，需要一个个指定。一般使用package标签，指定一个包。
8. package标签，在mappers标签内指定mapper.xml文件路径和注解类的包名。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- MyBatis的主配置 -->
<configuration>
    <!--
        引入外部数据库配置信息
            resource属性：指定配置文件的位置，是按照类路径的写法来写，并且必须存在于类路径下
            url属性：是按照按照Url的写法来写
    -->
    <properties resource="jdbcConfig.properties"/>

    <typeAliases>
        <!-- typeAlias，配置别名，当指定了别名时，就不再区分大小写 -->
        <typeAlias type="com.itheima.domain.User" alias="user"/>
        <!-- package，上面使用typeAlias标签一个个去配，很麻烦，而package标签可以指定一个包，为包里的所有实体都配置别名，不再区分大小写 -->
        <package name="com.itheima.domain"/>
    </typeAliases>

    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- MySql环境 -->
        <environment id="mysql">
            <!-- 配置事务的类型 -->
            <transactionManager type="JDBC"/>
            <!--
                配置数据源（连接池）
                    种类：
                        1.POOLED，采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现
                        2.UNPOOLED，采用传统获取连接的概念，虽然也实现了javax.sql.DataSource接口，但没有池的概念，所以每次获取连接都是新的，并没有复用
                        3.JNDI，采用服务器提供的JNDI技术实现，不同的服务器，获取到的实现都不一样
                            注意：如果不是web工程或maven工程，是不能使用的！
                            项目中使用的是Tomcat服务器，采用的连接池就是dbcp连接池
            -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--
    指定映射配置文件的位置，映射配置指的是每个dao独立的配置文件
        注意如果使用注解方式，对应的xml不能存在，否则会报错
    -->
    <mappers>
        <!-- xml方式，配置resource属性，指定文件路径 -->
        <!--        <mapper resource="com/itheima/dao/IUserDao.xml"/>-->
        <!-- 注解方式，配置class属性，指定类的全限定类名 -->
        <!--        <mapper class="com.itheima.dao.IUserDao"/>-->

        <!-- package属性，用于指定Dao层接口所在的包，当指定了之后，就不需要再写上面的mapper标签和resource或class属性 -->
        <package name="com.itheima.dao"/>
    </mappers>
</configuration>
```

### 账户实体Account

```
/**
 * 账户实体
 */
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
    
    //省略get、set和toString方法
}
```

1. 建立Java的Dao层包，例如我的项目是com.itheima.dao。
2. 在resources目录下，建立和Dao包名相同目录。（要一个个建立！）

- 账户表的单表操作

```
/**
 * 账户Dao层接口
 */
public interface IAccountDao {
    /**
     * 查询所有账户
     */
    List<Account> findAll();

    /**
     * 根据用户Id查询账户
     */
    List<Account> findAccountByUid(Integer id);
}
```

- 创建Mapper文件：IAccountDao.xml，名字要和接口名一致

1. mapper标签，持久层操作方法，都必须写在里面
2. namespace属性，命名空间，填入Dao层的全限定类名（重要！这是mapper中的方法的唯一标识的前缀，每个方法时由命名空间+方法名，来标识的）
3. select标签，查询标签，id为Dao层声明的接口方法名，必须要一样！
4. resultType，方法返回值的映射类型，可以使用别名或全限定类名，别名就是SqlMapConfig.xml中typeAliases标签配置的
5. parameterType，当接口方法有一个参数时，需要指定入参的java类型，同样可以使用别名和类的全限定类名
6. 获取接口参数，使用#{}占位符的方式，名字为接口参数的名字

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace，命名空间必须和Dao接口一样 -->
<mapper namespace="com.itheima.dao.IAccountDao">

    <!-- 配置查询所有 -->
    <select id="findAll" resultType="account">
        select * from account
    </select>

    <select id="findAccountByUid" resultType="com.itheima.domain.Account" parameterType="int">
        select * from account where uid = #{id}
    </select>
</mapper>
```

- 测试

1. Resources资源类，调用getResourceAsStream()方法，加载配置文件
2. SqlSessionFactory，SqlSession工厂类，专门来生产SqlSession对象，它需要用SqlSessionFactoryBuilder来构建
3. 调用SqlSessionFactory的openSession()，获取一个SqlSession
4. 调用SqlSession的getMapper()方法，传入Dao层接口类的Class，动态代理生成接口的实现类

注意点：
1. openSession()时，默认事务是设置为手动提交的，如果传入true，则代表开启自动提交事务，一般我们都是手动控制
1. 操作完，如果是插入、更新等操作，需要调用SqlSession的commit()方法提交事务。
2. 使用完，关闭SqlSession和配置文件的输入流

```
public class AccountTest {
    private InputStream inputStream;
    private SqlSession session;
    private IAccountDao accountDao;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder()
                .build(inputStream);
        //3.使用工厂生产SqlSession对象，autoCommit设置为true时，为自动提交事务，默认为false，一般我们都手动控制事务
//        session = factory.openSession(true);
        session = factory.openSession();
        //4.使用SqlSession创建Dao接口的代理对象
        accountDao = session.getMapper(IAccountDao.class);
    }

    @After
    public void destroy() throws IOException {
        //注意，如果openSession()，没有传autoCommit参数，或者设置为false，则需要自己手动提交事务
        session.commit();
        //6.释放资源
        session.close();
        inputStream.close();
    }

    /**
     * 测试查询所有账户
     */
    @Test
    public void testFindAll() {
        List<Account> resultList = accountDao.findAll();
        for (Account account : resultList) {
            System.out.println(account);
        }
    }
}
```

### 用户表的增删查改

创建实体

```
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    
    //省略get、set和toString方法
}
```

- 创建Dao层接口

```
/**
 * 用户表持久层接口
 */
public interface IUserDao {
    /**
     * 查询所有用户，同时获取到用户下的所有账户信息
     */
    List<User> findAll();

    /**
     * 保存用户
     */
    int saveUser(User user);

    /**
     * 更新用户
     */
    void updateUser(User user);

    /**
     * 删除用户
     *
     * @param userId 用户Id
     */
    void deleteUser(int userId);

    /**
     * 按用户Id查询用户信息
     */
    User findById(int userId);
}
```

- 创建Mapper文件：IUserDao.xml

1. 增：insert标签
2. 删：delete标签
3. 改：update标签

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace，命名空间必须和Dao接口一样 -->
<mapper namespace="com.itheima.dao.IUserDao">
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user;
    </select>

    <insert id="saveUser" parameterType="com.itheima.domain.User">
        <!-- 配置插入数据后，获取插入数据的id -->
        <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
        insert into user(username, address, sex, birthday) values(#{username}, #{address}, #{sex}, #{birthday});
    </insert>

    <update id="updateUser" parameterType="com.itheima.domain.User">
        update user set username = #{username}, address = #{address},sex = #{sex}, birthday = #{birthday} where id = #{id};
    </update>

    <!-- 只有1个参数，占位符中的名称写什么都可以 -->
    <delete id="deleteUser" parameterType="java.lang.Integer">
        select * from user
        where id = #{uid};
    </delete>

    <select id="findById" parameterType="java.lang.Integer" resultType="com.itheima.domain.User">
        select * from user
        where id = #{userId}
    </select>
</mapper>
```

#### 模糊查询

- Dao接口中添加方法

```
public interface IUserDao {
    //...省略其他方法

    /**
     * 根据名称，模糊查询用户信息
     */
    User findByName(int username);
}
```

- Mapper的XML文件中添加方法

有2种方式：
1. `#{}`占位符方式，以预编译、占位符方式嵌入参数，能防止SQL注入（常用）
2. `${}`字符串拼接方式，有一个默认叫value的值，只能适用于单个参数，不能防止SQL注入，限制大，很少用

```
<select id="findByName" parameterType="java.lang.String" resultType="com.itheima.domain.User">
    <!-- select * from user where username like #{name} -->
    select * from user
    where username like '%${value}%'
</select>
```

#### 查询总数量

- Dao层接口

```
public interface IUserDao {
    //...省略其他方法

    /**
     * 查询总用户数
     */
    int findTotal();
}
```

- Mapper的XML文件中添加方法

```
<select id="findTotal" resultType="java.lang.Integer">
    select count(id) from user;
</select>
```

#### 使用多层封装的JavaBean作为接口参数

一般我们结构很复杂的时候，会好几层模型的封装，MyBatis同样可以解析，XML中使用：变量名.属性名，这种方式访问变量的属性，多层的话，多个.就可以了

- 实体类

```
public class QueryVO {
    private User user;

    //省略get、set和toString方法
}
```

- Dao层接口

```
public interface IUserDao {
    //...省略其他方法

    /**
     * 根据QueryVO查询用户信息
     */
    List<User> findUserByVo(QueryVO vo);
}
```

- Mapper的XML文件中添加方法

```
<select id="findUserByVo" parameterType="com.itheima.domain.QueryVO" resultType="com.itheima.domain.User">
    select * from user
    where username like #{user.username}
</select>
```

#### 多个条件组合判断查询

需求：按用户的名称或性别来查询，条件不一定都有

- Dao层接口

```
public interface IUserDao {
    //...省略其他方法

    /**
     * 根据条件查询用户信息
     *
     * @param user 查询条件
     */
    List<User> findUserByCondition(User user);
}
```

- Mapper的XML文件中添加方法

1. where标签，表示where条件
2. if标签，表示每个查询条件，test属性为添加条件时的前置条件

```
<select id="findUserByCondition" parameterType="user" resultType="com.itheima.domain.User">
    select * from user
    <where>
        <if test="username != null">
            and username = #{username}
        </if>
        <if test="sex != null">
            and sex = #{sex}
        </if>
    </where>
</select>
```

#### 查询多个id的用户

修改QueryVO，增加ids属性。SQL查询使用in语句。

```
public class QueryVO {
    private User user;

    /**
     * 要查询多个人的id
     */
    private List<Integer> ids;
    
    //省略get、set、toString方法
}
```

- Dao层接口

```
public interface IUserDao {
    //...省略其他方法

    /**
     * 根据QueryVO中的id集合，查询多个用户信息
     */
    List<User> findUserInIds(QueryVO vo);
}
```

- Mapper的XML文件中添加方法

1. foreach标签，表示需要循环遍历
2. collection属性：指定集合的变量名称
3. open属性，遍历开始前的前缀
4. close属性，遍历结束后的后缀
5. separator属性，每次循环，每个条目之间的分隔符，一般用逗号
6. item属性，每次循环，每个条目的名称

```
<select id="findUserInIds" resultType="user">
    select * from user
    <where>
        <if test="ids != null and ids.size > 0">
            <foreach collection="ids" open="and id in(" close=")" separator="," item="uid">
                #{uid}
            </foreach>
        </if>
    </where>
</select>
```

#### JavaBean和表字段不对应

返回结果我们可以使用resultType来指定返回JavaBean，但是如果JavaBean和表的字段名不一致，那么就要使用resultMap

- Mapper的XML文件

```
<!-- 配置实体类和数据库字段的对应关系 -->
<resultMap id="userMap" type="user">
    <!-- 主键字段对应 -->
    <id property="id" column="id"/>
    <!-- 非主键字段对应，这里嫌麻烦，就不弄了，知道意思就好 -->
    <!-- <result property="username" column="userName"/> -->
    <result property="username" column="username"/>
</resultMap>

<select id="findUserByCondition" parameterType="user" resultMap="userMap">
    <include refid="defaultUser"/>
    <where>
        <if test="username != null">
            and username = #{username}
        </if>
        <if test="sex != null">
            and sex = #{sex}
        </if>
    </where>
</select>
```

#### 多表联查

多表联查，MyBatis提供了ResultMap，可以轻松配置一对多、多对一、多对多关系。

而MyBatis将多对一看做一对一，多对多看为一对多，提供以下2个标签，标签的属性，在后续实例中会讲解

1. 一对多和多对多：使用collection标签
2. 多对一：使用association标签

#### 多对一的SQL联查

需求：查询所有账户，并查询出账户所属的用户信息（多个账户能被一个用户拥有，多对一关系）

- 修改账户实体，增加一个User对象

```
/**
 * 账户实体
 */
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
    /**
     * 多对一关系映射，从表实体包含一个主表实体的引用
     */
    private User user;
}
```

- IAccountDao增加方法

```
public interface IAccountDao {
    /**
     * 查询所有账户，同时包含用户的信息
     */
    List<Account> findAllAccount();
}
```

- IAccountDao.xml

1. association标签，一对一的关系映射
2. property属性：表示封装User信息到Account的哪个属性上
3. column属性：表示user和account关联的外键属性
4. javaType属性：封装的数据类型

```
<!-- 定义封装Account和User的resultMap -->
<resultMap id="accountUserMap" type="account">
    <id property="id" column="aid"/>
    <result property="uid" column="uid"/>
    <result property="money" column="money"/>
    <!--
        一对一的关系映射，配置封装的User的内容
        1.property属性：表示封装User信息到Account的哪个属性上
        2.column属性：表示user和account关联的外键属性
        3.javaType属性：封装的数据类型
    -->
    <association property="user" column="uid" javaType="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="address" column="address"/>
        <result property="sex" column="sex"/>
        <result property="birthday" column="birthday"/>
    </association>
</resultMap>

<select id="findAllAccount" resultMap="accountUserMap">
        select a.*,u.username,u.address from account a, user u where u.id = a.uid;
</select>
```

### 一对多的SQL联查

需求：查询用户表下所有用户，以及用户下的所有账户（一个用户下有多个账户，一对多关系）。

- 修改User实体，增加一个List<Account>属性，表示当前用户拥有的所有角色

```
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    /**
     * 一对多关系映射，主表实体应该包含从表实体的集合引用
     */
    private List<Account> accounts;
    
    //省略set、get和toString方法
}
```

- 修改IUserDao.java的findAll()方法，user表 left join account表

提供resultMap，userAccountMap

1. collection属性：指定集合属性名称
2. ofType属性：指定集合中的元素的类型
3. property属性指定JavaBean中的字段属性
4. column属性指定SQL查询中的字段名

```
<!-- 定义用户和账户的映射 -->
<resultMap id="userAccountMap" type="user">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="address" column="address"/>
    <result property="sex" column="sex"/>
    <result property="birthday" column="birthday"/>
    <!--
    配置User对象中的accounts集合的映射
        property属性：指定集合属性名称
        ofType属性：指定集合中的元素的类型
    -->
    <collection property="accounts" ofType="account">
        <id property="id" column="aid"/>
        <result property="uid" column="uid"/>
        <result property="money" column="money"/>
    </collection>
</resultMap>

<!-- 配置查询所有 -->
<select id="findAll" resultMap="userAccountMap">
    select * from user u left join account a on u.id = a.uid
</select>
```

### 多对多的SQL联查

需求：查询所有用户，以及用户的所有角色。（多对多关系，多个用户都可以拥有同一个角色，同时角色能被多个用户拥有）

- 角色实体

```
/**
 * 角色实体
 */
public class Role implements Serializable {
    private Integer roleId;
    private String roleName;
    private String roleDesc;

    //省略set、get和toString
}
```

- IUserDao新增方法

```
public interface IUserDao {
    /**
     * 查询所有用户信息和角色信息
     */
    List<User> findUserRoles();
}
```

- IUserDao.xml

resultMap中配置和上面的一对多一样，也是使用collection属性，其他属性就不在赘述了。

```
<!-- 定义用户和角色的映射 -->
<resultMap id="userRoleMap" type="user">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="address" column="address"/>
    <result property="sex" column="sex"/>
    <result property="birthday" column="birthday"/>
    <!-- 配置角色的属性 -->
    <collection property="roles" ofType="role">
        <id property="roleId" column="aid"/>
        <result property="roleName" column="role_name"/>
        <result property="roleDesc" column="role_desc"/>
    </collection>
</resultMap>

<select id="findUserRoles" resultMap="userRoleMap">
    SELECT u.*, r.id as rid, r.role_name, r.role_desc
    FROM user u
    LEFT JOIN user_role ur on u.id = ur.uid
    LEFT JOIN role r on r.id = ur.rid
</select>
```

需求：查询所有角色，并把拥有当前角色的所有用户都查询出来

- 修改角色实体，增加用户集合，表示拥有该角色的所有用户

```
/**
 * 角色实体
 */
public class Role implements Serializable {
    private Integer roleId;
    private String roleName;
    private String roleDesc;
    /**
     * 多对多关系映射，一个角色可以赋予给多个用户
     */
    private List<User> users;

    //省略set、get和toString
}
```

- 增加IRoleDao，提供持久层接口

```
/**
 * 角色表的Dao层接口
 */
public interface IRoleDao {
    /**
     * 查询所有角色
     */
    List<Role> findAll();
}
```

- IRoleDao.xml

同样使用collection属性来配置用户列表。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace，命名空间必须和Dao接口一样 -->
<mapper namespace="com.itheima.dao.IRoleDao">
    <!-- 配置实体类和数据库字段的对应关系 -->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="id"/>
        <result property="roleName" column="role_name"/>
        <result property="roleDesc" column="role_desc"/>
        <collection property="users" ofType="user">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
            <result property="address" column="address"/>
        </collection>
    </resultMap>

    <!-- 配置查询所有 -->
    <select id="findAll" resultMap="roleMap">
        SELECT u.*, r.id as rid, r.role_name, r.role_desc
        FROM role r
        LEFT JOIN user_role ur on r.id = ur.rid
        LEFT JOIN user u on u.id = ur.uid
    </select>
</mapper>
```

### 注解方式

MyBatis除了提供XML方式编写Mapper外，还提供了注解方式，注解相比XML会少写一些代码，但是将SQL写到Java类中，一般不推荐。

- 基本CRUD，注解内写的SQL和XML配置是一样的，所以就不再赘述了

1. @Select，查询
1. @Insert，插入
1. @Update，更新
1. @Delete，删除

- ResultMap配置

1. 如果实体和数据库字段不同，则可以使用@Results注解，使用@Results注解，配置映射关系
2. @Results注解的id属性，为ResultMap的id，可以被其他方法引用。
3. @Results注解的value属性，为@Result注解的数组，所以可以配置多个@Result注解
4. @Result注解，id属性，布尔值，默认为false，代表是否是id属性，只有id属性，才会设置true
5. @Result注解，column属性为数据库字段，property属性为实体字段

- 一对多关系配置

1. @Result注解，many属性，存放@Many注解，用于配置一对多关系
2. @Many注解，select属性，指定查询哪个Mapper（Dao层接口）的方法来查询，命名空间+方法名，例如这里需要调用IAccountDao的findAccountByUid()方法，值就为com.itheima.dao.IAccountDao.findAccountByUid。
3. @Many注解，fetchType属性，指定查询时机，有3个值，LAZY（懒加载），EAGER（饥饿，马上查询），DEFAULT（默认值，就是马上查询）

关于懒加载，一般一对多时使用懒加载，而多对一选择马上加载

```
/**
 * MyBatis注解练习
 * 注解一共有4个：
 * 1.@Select
 * 2.@Inser
 * 3.@Update
 * 4.@Delete
 */
public interface IUserDao {
    /**
     * 查询所有
     */
    @Select("select * from user")
    //如果实体和数据库字段不同，则可以使用@Results注解，配置映射关系，column为数据库字段，property为实体字段
    @Results(id = "userMap", value = {
            @Result(
                    id = true,
                    column = "id",
                    property = "userId"
            ),
            @Result(
                    column = "username",
                    property = "userName"
            ),
            @Result(
                    column = "address",
                    property = "userAddress"
            ),
            @Result(
                    column = "sex",
                    property = "userSex"
            ),
            @Result(
                    column = "birthday",
                    property = "userBirthday"
            ),
            //配置一对多的关联
            @Result(
                    column = "id",
                    property = "accounts",
                    many = @Many(
                            select = "com.itheima.dao.IAccountDao.findAccountByUid",
                            fetchType = FetchType.LAZY
                    )
            )
    })
    List<User> findAll();

    /**
     * 保存用户
     */
    @Insert("insert into user(username,address,sex,birthday) values(#{username},#{address},#{sex},#{birthday})")
    @ResultMap(value = {"userMap"})
    void saveUser(User user);

    /**
     * 更新用户
     */
    @Update("update user set username=#{username}, address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id}")
    @ResultMap(value = {"userMap"})
    void updateUser(User user);

    /**
     * 删除用户
     */
    @Delete("delete from user where id = #{id}")
    @ResultMap(value = {"userMap"})
    void deleteUser(int id);

    /**
     * 根据用户Id，查询用户信息
     *
     * @param id 用户Id
     */
    @Select("select * from user where id = #{id}")
    @ResultMap(value = {"userMap"})
    User findById(int id);

    /**
     * 根据名称模糊查询，用户列表
     *
     * @param username 用户名
     */
    @Select("select * from user where username like #{username}")
    @ResultMap(value = {"userMap"})
    //或者使用字符串拼接的方式
//    @Select("select * from user where username like '%${value}%'")
    List<User> findUserByName(String username);

    /**
     * 查询总数
     */
    @Select("select count(*) from user")
    @ResultMap(value = {"userMap"})
    int findTotal();
}
```

- 多对一配置

和上面的一对多基本一样，有2个

1. @Result注解，使用one属性，来配置一对一关系的关联
4. @One注解，和@Many注解一样，也有select属性和fetchType属性，含义一样

```
public interface IAccountDao {
    /**
     * 查询所有账户，并且获取每个账户下的所属的用户信息
     */
    @Select("select * from account")
    @Results(id = "accountMap", value = {
            @Result(
                    id = true,
                    column = "id",
                    property = "id"
            ),
            @Result(
                    column = "uid",
                    property = "uid"
            ),
            @Result(
                    column = "money",
                    property = "money"
            ),
            //配置一对一的关联
            @Result(
                    property = "user",
                    //column为调用User的单表查询时传的参数
                    column = "uid",
                    one = @One(
                            //select为去单表查询的方法
                            select = "com.itheima.dao.IUserDao.findById",
                            //多对一，使用立即加载
                            fetchType = FetchType.EAGER
                    )
            )
    })
    List<Account> findAll();

    /**
     * 根据Id查询账户信息
     */
    @Select("select * from account where uid = #{userId}")
    Account findAccountByUid(int userId);
}
```

### 懒加载

MyBatis为了优化多表联查，提供了懒加载，当触发了实体的指定字段时，才触发查询，再将结果集放到最后的结果中。

原理，动态代理返回的模型类，模型类中需要懒加载的字段为null，拦截get方法，动态查询后再设置字段值，最后返回。

#### XML方式开发，懒加载配置

- MyBatis配置文件中开启懒加载支持

```
<configuration>
    <!-- 配置延迟加载策略 -->
    <settings>
        <!-- 开启MyBatis支持延迟加载 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 将积极加载改为消息加载，即按需加载 -->
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
</configuration>
```

- 在Mapper.xml文件中，开启懒加载

1. collection标签，增加fetchType属性，lazy为懒加载，eager为立即加载

```
<!-- 支持懒加载的 -->
<resultMap id="userAccountLazyMap" type="user">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="address" column="address"/>
    <result property="sex" column="sex"/>
    <result property="birthday" column="birthday"/>
    <!--
    配置User对象中的accounts集合的映射
        property属性：指定集合属性名称
        ofType属性：指定集合中的元素的类型
    -->
    <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid"
                column="id" fetchType="lazy"/>

    <!-- 查询所有，并懒加载用户下的所有账户 -->
    <select id="findAllLazy" resultMap="userAccountLazyMap">
        select * from user
    </select>
</resultMap>
```

#### 注解方式开发，懒加载配置

其实上面讲解连表的时候，就提到了，@Many、@One注解都支持设置一个fetchType，设置为lazy则为懒加载

```
public interface IUserDao {
/**
     * 查询所有
     */
    @Select("select * from user")
    //如果实体和数据库字段不同，则可以使用@Results注解，配置映射关系，column属性为数据库字段，property属性为实体字段
    @Results(id = "userMap", value = {
            @Result(
                    id = true,
                    column = "id",
                    property = "userId"
            ),
            @Result(
                    column = "username",
                    property = "userName"
            ),
            @Result(
                    column = "address",
                    property = "userAddress"
            ),
            @Result(
                    column = "sex",
                    property = "userSex"
            ),
            @Result(
                    column = "birthday",
                    property = "userBirthday"
            ),
            //配置一对多的关联
            @Result(
                    //column为调用AccountDao层方法传递的参数
                    column = "id",
                    //property为封装到实体的字段的名称
                    property = "accounts",
                    many = @Many(
                            select = "com.itheima.dao.IAccountDao.findAccountByUid",
                            fetchType = FetchType.LAZY
                    )
            )
    })
    List<User> findAll();
}
```

### 缓存

缓存等级

1. 一级缓存（默认是开的），同一个SqlSession去获取的Mapper实例，查询方法会复用上次查询的结果对象和数据
2. 二级缓存（默认是关的，需要手动开启），不同SqlSession，但同一个SqlSessionFactory，查询方法只复用查询的数据，对象不复用

注意：缓存在调用Mapper的update、insert、delete系列的方法时，会被清空掉，下次select查询会重新从数据库中查询，再缓存起来。

#### XML方式开始，开启二级缓存

- MyBatis配置文件中开启二级缓存支持，settings标签下，增加一个设置，cacheEnable为true

```
<configuration>
    <settings>
        //...省略其他配置
        <!-- 开启二级缓存支持 -->
        <setting name="cacheEnable" value="true"/>
    </settings>
</configuration>
```

- Mapper.xml中，做2步操作

1. 增加一个cache标签，表示支持二级缓存
2. 给支持二级缓存的方法的select标签，增加useCache属性，设置为true

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace，命名空间必须和Dao接口一样 -->
<mapper namespace="com.itheima.dao.IUserDao">
    <!-- cache标签，表示开启User支持二级缓存 -->
    <cache/>

    <!-- useCache="true"，表示该方法的操作，使用二级缓存 -->
    <select id="findById" useCache="true" parameterType="java.lang.Integer" resultType="com.itheima.domain.User">
        <include refid="defaultUser"/>
        where id = #{userId}
    </select>
</mapper>
```

#### 注解方式开发，开启二级缓存

相比XML方法，直接在Mapper类上增加@CacheNamespace注解，设置blocking属性为true即可。

```
//支持二级缓存，blocking表示是否开启，默认为false，改为true，则开启
@CacheNamespace(blocking = true)
public interface IUserDao {
    //...省略其他方法
}
```