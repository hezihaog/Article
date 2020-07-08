#### SSM整合

本篇来整理一下，Spring、SpringMVC、MyBatis，这3大框架的整合，本文使用Maven做依赖管理，分别说明一下3个框架的角色

- Spring，业务层，负责依赖注入、AOP
- SpringMVC，表现层，负责请求和响应扭转流程、以及3层的注解
- MyBatis，持久层，负责数据之间的操作

整合顺序
1. 先Spring
2. 再整合SpringMVC
3. 最后整合MyBatis

#### 案例

整合过程，我们需要进行对3大框架进行测试，确保整合生效，所以有一个单表的案例，就是金额账户的保存和查找

- 准备数据和SQL，数据库工具导入即可

数据库名：ssm
表名：account

```
create database ssm;
use ssm;
create table account(
    id int primary key auto_increment,
    name varchar(20),
    money double 
);

INSERT INTO `account`(`id`, `name`, `money`) VALUES (1, '美美', 200);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (2, '小凤', 300);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (4, '小明', 500);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (5, 'Wally', 230);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (6, 'Barry', 350);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (11, '小米', 187);
INSERT INTO `account`(`id`, `name`, `money`) VALUES (12, '小罗', 19);
```

- 准备Account实体

字段分别是，账户表的id、账户所属人的名称、账户金额

```
/**
 * 账户实体
 */
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Double money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

- pom文件依赖和版本管理

```
<!-- 版本统一管理 -->
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>5.1.6</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
</properties>

<dependencies>
    <!-- spring -->
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
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>

        <version>4.12</version>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

    <!-- log start -->
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
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>

    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
        <type>jar</type>
        <scope>compile</scope>
    </dependency>
</dependencies>

<build>
    <finalName>ssm</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
        <plugins>
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <version>3.1.0</version>
            </plugin>
            <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
            </plugin>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-install-plugin</artifactId>
                <version>2.5.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

#### Spring

- Spring配置文件

在resources文件夹下创建Spring的配置文件：applicationContext.xml

1. 开始Spring的注解扫描，并且扫描时忽略Controller，引入Controller是由SpringMVC管理的，所以这里的Spring不用扫描
2. 配置Spring的声明式事务，分3步
    - 声明事务管理器：transactionManager
    - 声明事务AOP通知：txAdvice
    - 配置AOP增强，应用刚才的txAdvice通知和事务管理器transactionManager

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 1.开启注解扫描 -->
    <context:component-scan base-package="cn.itcast">
        <!-- 忽略Controller扫描 -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 2.配置Spring声明式事务管理 -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
            <tx:method name="*" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>
    <!-- 配置AOP增强 -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* cn.itcast.service.impl.*ServiceImpl.*(..))"/>
    </aop:config>
</beans>
```

- log4j.properties

Spring中的Log输出使用了log4j，所以将配置文件拷贝到resources目录下

```
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=info, CONSOLE, LOGFILE

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

- 准备Service层接口以及实现

现在没有整合MyBatis，所以Dao层操作直接输出Log先

```
/**
 * 账户业务层接口
 */
public interface AccountService {
    /**
     * 查询所有账户信息
     */
    List<Account> findAll();

    /**
     * 保存账户信息
     */
    void saveAccount(Account account);
}

/**
 * 账户业务层接口实现
 */
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    @Override
    public List<Account> findAll() {
        return new ArrayList<>()
    }

    @Override
    public void saveAccount(Account account) {
        System.out.println(account);
    }
}
```

- 测试Spring

新建测试类TestSpring，提供一个run1方法，获取Spring容器，获取Service的实例，测试依赖注入，没有报错，就证明整合成功了

```
/**
 * 测试Spring框架整合
 */
public class TestSpring {
    @Test
    public void run1() {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");
        accountService.findAll();
    }
}
```

#### 整合SpringMVC

Spring整合完了，接下来就是SpringMVC

- 配置web.xml

SpringMVC是封装了Servlet等WebAPI的库，所以web.xml作为Servlet接口的配置文件，我们也需要做配置，配置内容如下

1. 配置Spring的监听器，加载Spring的配置文件，它默认只加载WEB-INF目录下载applicationContext.xml配置文件。以及配置Spring的监听器的初始化参数
2. 配置解决中文乱码的过滤器以及初始化参数，如果不配置，请求有中文，编码默认不是UTF-8，就会乱码
3. 配置DispatcherServlet前端控制器：配置服务器时启动就必须加载，加载springmvc.xml配置文件

```
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>

    <!-- 1.配置Spring的监听器，加载Spring的配置文件，它默认只加载WEB-INF目录下载applicationContext.xml配置文件 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 所以，我们要配置初始化参数，加载resources目录下的配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <!-- 2.配置解决中文乱码的过滤器 -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 初始化参数，指定编码 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 3.配置前端控制器：服务器启动必须加载，需要加载springmvc.xml配置文件 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <!-- 配置初始化参数，创建完DispatcherServlet对象，加载springmvc.xml配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param> <!-- 服务器启动的时候，让DispatcherServlet对象创建 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

- 准备配置文件

上面web.xml中，我们配置了服务器在启动时，就加载SpringMVC的配置文件springmvc.xml，所以接下来我们开始配置

在resources目录下，新建springmvc.xml，作为SpringMVC的配置文件

1. 开启注解扫描，只扫描Controller
2. 配置视图解析器，指定jsp的存放位置和后缀
3. 配置不过滤静态资源，不过滤html、css、js等，否则前端页面拉取静态资源都会被SpringMVc拦截掉
4. 开启SpringMVC注解支持

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 开启注解扫描，只扫描Controller -->
    <context:component-scan base-package="cn.itcast">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 配置视图解析器 -->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 视图文件位置 -->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <!-- 视图文件后缀 -->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 配置不过滤静态资源 -->
    <mvc:resources location="/css/" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/js/" mapping="/js/**"/>

    <!-- 开启SpringMVC注解支持 -->
    <mvc:annotation-driven/>
</beans>
```

- 编写页面

在webapp建立文件夹

1. css，存放css文件
2. images，存放图片文件
3. js，存放JavaScript文件
4. WEB-INF，里面建立pages目录，主要存放项目的jsp文件
5. index.jsp，项目首页

- 首页：index.jsp

```
<%--
  Created by IntelliJ IDEA.
  User: wally
  Date: 2020/6/23
  Time: 4:48 下午
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="utf-8" %>
<html>
<head>
    <title>首页</title>
</head>
<body>
<a href="account/findAll">查询所有账户</a>
<br>
<form action="account/saveAccount" method="post">
    名称：<input type="text" name="name"><br>
    金额：<input type="text" name="money"><br>
    <input type="submit" value="保存">
</form>
</body>
</html>
```

- 结果页面：list.jsp，存放在pages下

```
<%--
  Created by IntelliJ IDEA.
  User: wally
  Date: 2020/6/23
  Time: 4:50 下午
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>账户列表</title>
</head>
<body>
<h3>查询了所有的账户信息</h3>
<br>
<c:forEach items="${list}" var="account">
    ${account.id} - ${account.name}：${account.money}<br>
</c:forEach>
</body>
</html>
```

- 准备Controller

1. 使用@Controller注解，标识为Controller类
2. 使用@RequestMapping注解，指定当前控制器的一级目录为/account
3. 提供接口方法， 使用@RequestMapping注解，标识为接口方法，同时指定接口路径，请求方式（GET、POST等）
4. 使用@Autowired注解，依赖注入AccountService接口的实例

```
@Controller
@RequestMapping("/account")
public class AccountController {
    @Autowired
    private AccountService accountService;

    /**
     * 查询所有账户
     */
    @RequestMapping(value = "/findAll", method = RequestMethod.GET)
    public ModelAndView findAll() {
        List<Account> accounts = accountService.findAll();
        ModelAndView mv = new ModelAndView();
        //跳转到结果页面，并传递accounts结果集合
        mv.addObject("list", accounts);
        mv.setViewName("list");
        return mv;
    }

    /**
     * 保存账户
     */
    @RequestMapping(value = "/saveAccount", method = RequestMethod.POST)
    public ModelAndView saveAccount(Account account) {
        accountService.saveAccount(account);
        return findAll();
    }
}
```

- 测试

Idea配置Tomcat，需要填写请求根路径，我配置为ssm，端口号为8080。测试查询和保存这2个接口，没有问题即可

- 默认服务器启动后，自动打开index.jsp文件
- 查询所有账户，GET请求，请求路径为http://localhost:8080/ssm/account/findAll
- 保存金额POST请求，，请求路径为http://localhost:8080/ssm/account/saveAccount

#### 整合MyBatis

整合完Spring和SpringMVC，就查MyBatis，下面我们继续

- 配置文件

同样在resources文件夹下，创建SqlMapConﬁg.xml文件

1. 配置数据源，数据库的4大属性
2. 配置Mapper，一般我们指定mapper的包名即可

```
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- Spring整合后，该文件就可以不要了，将配置在spring的配置文件中配置即可 -->
<configuration>
    <!-- 1.数据源 -->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///ssm"/>
                <property name="username" value="root"/>
                <property name="password" value="hezihao123"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 2.配置Mapper -->
    <mappers>
        <!-- 如果使用的是mapper.xml -->
        <!-- <mapper resource="cn/itcast/dao/xxx.xml"/> -->

        <!-- 如果使用的是注解，指定Dao接口的位置，但需要一个个加，麻烦 -->
        <!-- <mapper class="cn.itcast.dao.AccountDao"/> -->

        <!-- 注解，直接扫描指定某个包下的即可，该包下所有的dao接口都可以使用 -->
        <package name="cn.itcast.dao"/>
    </mappers>
</configuration>
```

2. 创建Dao接口

创建AccountDao接口，这里使用MyBatis的注解开发。

如果是xml做配置，则需要在resources文件夹下，创建Dao包名路径一样的文件夹。

例如AccountDao的包名是cn.itcast.dao。那么创建文件夹的路径就为cn/itcast/dao

最后创建一个AccountDao.xml文件，进行Dao层的开发

```
/**
 * 账号Dao接口
 */
@Repository
public interface AccountDao {
    /**
     * 查询所有账户信息
     */
    @Select("select * from account")
    List<Account> findAll();

    /**
     * 保存账户信息
     */
    @Insert("insert into account(name, money) value(#{name}, #{money})")
    void saveAccount(Account account);
}
```

- 测试框架

创建TestMyBatis类，测试查询和保存的Dao持久层方法，运行成功即可

```
public class TestMyBatis {
    /**
     * 测试查询
     */
    @Test
    public void run1() throws Exception {
        //加载MyBatis配置文件
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConﬁg.xml");
        //创建工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder()
                .build(inputStream);
        //创建session
        SqlSession session = factory.openSession();
        //创建Dao代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        List<Account> accounts = dao.findAll();
        for (Account account : accounts) {
            System.out.println(account);
        }
        //关闭资源
        session.close();
        inputStream.close();
    }

    /**
     * 测试保存
     */
    @Test
    public void run2() throws Exception {
        //加载MyBatis配置文件
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConﬁg.xml");
        //创建工厂
        SqlSessionFactory factory = new SqlSessionFactoryBuilder()
                .build(inputStream);
        //创建session
        SqlSession session = factory.openSession();
        //创建Dao代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        Account account = new Account();
        account.setName("小明");
        account.setMoney(500d);
        dao.saveAccount(account);
        //提交事务
        session.commit();
        //关闭资源
        session.close();
        inputStream.close();
    }
}
```

- 整合Spring

上面MyBatis框架我们测试通过了，但还没有整合到Spring，上面我们看到我们需要手写SqlSessionFactory，那么整合到Spring，我们配置上就可以了

整合到Spring，上面的SqlMapConfig.xml配置文件就可以不用了，我们将配置写到Spring的配置文件中。

1. 配置数据源和连接池，这里使用c3p0连接池
2. 配合SqlSessionFactory工厂，传递数据源
3. 配置Dao接口的包

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    省略上面Spring的配置...

    <!-- Spring整合MyBatis框架 -->
    <!-- 1.配置数据源和连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql:///ssm"/>
        <property name="user" value="root"/>
        <property name="password" value="hezihao123"/>
    </bean>
    <!-- 2.配合SqlSessionFactory工厂 -->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 3.配置Dao接口的包 -->
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.itcast.dao"/>
    </bean>
</beans>
```

- 在Service层使用AccountDao

前面我们配置好了AccountDao，还需要配置到AccountService的实现类中去，才能连接SpringMVC

1. @Autowired依赖注入Dao层接口
2. 实现findAll()查询所有账户、saveAccount()保存账户

```
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;

    @Override
    public List<Account> findAll() {
        return accountDao.findAll();
    }

    @Override
    public void saveAccount(Account account) {
        accountDao.saveAccount(account);
    }
}
```

- 最后

3大框架，我们连接完成了，启动服务器，在index.jsp中，操作查询所有、保存这2个操作，数据库中数据正确即可！

#### 总结

Spring、SpringMVC、MyBatis，3大框架，到这里就整合结束了，整合的源码我已经上传到[码云](https://gitee.com/hezihao/ssm)，有兴趣的同学可以clone来看下。