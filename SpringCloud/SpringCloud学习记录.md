# SpringCloud学习记录

本篇是[SpringCloud 视频教程全集（50P）| 10 小时从入门到精通](https://www.bilibili.com/video/av59639535/)，学习记录。

## 总体概念

### SpringCloud是什么？

SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等，它们都可以用SpringBoot开发风格做到一键启动和部署。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来,通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单 易懂、易部署和易维护的分布式系统开发工具包。

### SpringCloud和SpringBoot有什么关系

- SpringBoot
    1. SpringBoot专注于快速方便的开发单个个体微服务。
    2. SpringBoot寺注于快速、方便的幵友単个微服努个体，SpringCloud关注全局的服务治理框架。

- SpringCloud
    1. SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之问提供，配置管理、服务发现、断路器、路由、微代理、事性总线、全局锁、决策竞选、分布式会话等等集成服务。
    2. SpringCloud依赖于SpringBoot。SpringCloud是微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体。
    3. SpringBoot可以离开SpringCloud独立使用开发项目， 但是SpringCloud离不开SpringBoot. 属于依赖的关系。
    
### 微服务有哪些优缺点

- 优点

    1. 每个服务足够内聚，足够小，代码容易理解，这样能聚焦一个指定的业务功能或业务需求。
    2. 开发简单、开发效率提高、一个服务可能就是专一的只干一件事。
    3. 微服务能够被小团队单独开发，这个团队是2到5人的开发人员组成。
    4. 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。
    5. 微服务能够使用不同的语言开发。
    6. 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如Jenkins、Hudson、bamboo。
    7. 微服务易于被一个开发人员理解，修改和维护。这样小团队能够更加关注自己的工作成果，无需通过合作才能体现价值。
    8. 微服务允许你利用融合最新技术。
    9. 微服务只是业务逻辑的代码，不会和HTML、CSS或其他页面组件混合。
    10. 每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一数据库。

- 缺点
    
    1. 开发人员要处理分布式系统的复杂性。
    2. 多服务运维难度，随着服务的增多，运维的压力也在增大。
    3. 系统部署依赖。
    4. 服务间通信成本。
    5. 数据一致性。
    6. 系统集成测试。
    7. 性能监控。
    
## SpringCloud 五大组件

- Eureka 注册中心
- Ribbon和Feign
- hystrix 熔断器
- SpringCloud Config 配置中心
- 网关zuul，服务有多个地址，前端配置那么多地址是不合理的，一般我们会统一地址，给前端调用。除了统一调用，还有统一鉴权，以及多服务的反向代理。

## Eureka服务注册与发现

### Eureka是什么
    1. Eureka在微服务架构中重要的一员，负责服务注册与发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件，功能类似Dubbo的注册中心，比如Zookeeper。
    2. Netflix在设计Eureka时遵守的就是AP原则。（一句话概括：某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该服务的信息进行保存）
    
### Eureka架构
    1. Eureka包含两个组件：Eureka Server和Eureka Client
    2. 各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到
    3. EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的，使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除(默认90秒)

### Eureka的自我保护机制

默认情况下，如果EurekaServer在- 定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例(默认90秒)。
但是当网络分区故障发生时,微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了一 因为微服务本身其实
是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题一 当EurekaServer节点在短时间内丢
失过多客户端时(可能发生了网络分区故障)， 那么这个节点就会进入自我保护模式。一-旦进入该模式，EurekaServer就会保护服
务注册表中的信息，不再删除服务注册表中的数据(也就是不会注销任何微服务)。当网络故障恢复后，该Eureka Server节点会
自动退出自我保护模式。

在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上
时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能
健康的服务实例。-句话讲解:好死不如赖活着.

综上，自我保护模式是一种应对网络异常的安全保护措施。 它的架构哲学是宁可同时保留所有微服务(健康的微服务和不健康的微
服务都会保留)，也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。
在Spring Cloud中,可以使用eureka.server.enable-self-preservation = false禁用自我保护模式。

### 建立项目统一父工程

一般我们会抽取父工程来管理依赖版本，以及一些公共依赖。

- pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.springcloud</groupId>
    <artifactId>microservicecloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.9</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.2</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>microservicecloud</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <delimiters>
                        <delimit>$</delimit>
                    </delimiters>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 搭建单机版的Eureka Server

- 新建microservicecloud-eureka-7001模块，修改pom文件，添加eureka server 依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-eureka-7001</artifactId>

    <dependencies>
        <!-- eureka server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

- resources文件夹，新建application.yml文件

```
server:
  port: 7001

eureka:
  server:
    enable-self-preservation: true #是否开启自我保护机制，默认为true，就是开启
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，不需要检索服务
    service-url:
      #单机
      default-zone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置Eureka Server交互的地址查询服务和注册服务都需要这个依赖
```

- 修改启动类

加上注解`@EnableEurekaServer`，代表当前工程为EurekaServer服务。

```
@SpringBootApplication
//告诉SpringBoot，本文件为Eureka Server启动类
@EnableEurekaServer
public class EurekaServer7001App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001App.class, args);
    }
}
```

### 搭建集群版的Eureka Server

再建立（复制）2个Eureka Server模块，上面第一个叫microservicecloud-eureka-7001，现在这2个就叫microservicecloud-eureka-7002、microservicecloud-eureka-7003。

`启动类`和`pom文件`都是一样的，就不再赘述了。唯一区别就是`application.yml`配置文件

- 我们的eureka服务通过域名访问，为了方便，我们在本机的HOST文件中，配置3个eureka的域名

```
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```

- 修改7001的application.yml

将单机版的url改为集群版的。每个eureka服务要配置另外几个eureka服务的域名，不能配置自己的！

```
//eureka-7001
server:
  port: 7001

eureka:
  server:
    enable-self-preservation: true #是否开启自我保护机制，默认为true，就是开启
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，不需要检索服务
    service-url:
      #集群
      default-zone: http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka

//eureka-7002
server:
  port: 7002

eureka:
  server:
    enable-self-preservation: true #是否开启自我保护机制，默认为true，就是开启
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，不需要检索服务
    service-url:
      #集群
      default-zone: http://eureka7001.com:7001/eureka,http://eureka7003.com:7003/eureka

//eureka-7003
server:
  port: 7003

eureka:
  server:
    enable-self-preservation: true #是否开启自我保护机制，默认为true，就是开启
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，不需要检索服务
    service-url:
      #集群
      default-zone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

### Eureka Client

新建模块，作为Eureka的客户端。

- pom文件

添加EurekaClient的依赖

```
<dependencies>
    <!-- Eureka Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```

- application.yml

添加eureka的地址，如果是集群eureka，则配置多个eureka域名，逗号分隔。

如果当前服务只是消费端，没有提供API，那么`register-with-eureka`设置为false，否则设置为true。

```
server:
  port: 80

eureka:
  client:
    register-with-eureka: false #不注册到注册中心
    # 配置eureka server的地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

- 启动类

添加注解`@EnableEurekaClient`，标识当前服务为Eureka客户端。

```
@SpringBootApplication
//配置Eureka客户端
@EnableEurekaClient
public class DeptConsumer80App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80App.class, args);
    }
}
```

## 搭建服务提供方和服务消费方

由于需要服务间调用，我们建立一个`服务提供方模块provider-dept-8001`和一个`服务消费方模块consurmer-dept-80`。

### 准备数据库数据

- 库名：cloudDB01
- 表名：dept

一张部门表，有3个字段，主键id、部门名称、当前库名。

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for dept
-- ----------------------------
DROP TABLE IF EXISTS `dept`;
CREATE TABLE `dept` (
  `deptno` bigint(20) NOT NULL AUTO_INCREMENT,
  `dname` varchar(60) DEFAULT NULL,
  `db_source` varchar(60) DEFAULT NULL,
  PRIMARY KEY (`deptno`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of dept
-- ----------------------------
BEGIN;
INSERT INTO `dept` VALUES (1, '开发部', 'clouddb01');
INSERT INTO `dept` VALUES (2, '人事部', 'clouddb01');
INSERT INTO `dept` VALUES (3, '财务部', 'clouddb01');
INSERT INTO `dept` VALUES (4, '市场部', 'clouddb01');
INSERT INTO `dept` VALUES (5, '运维部', 'clouddb01');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 公共模块

由于我们有公共的实体类，以及后面的Feign服务，我们抽取为一个单独的模块：microservicecloud-api

- pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 继承统一的父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <!-- 当前Module的名字 -->
    <artifactId>microservicecloud-api</artifactId>

    <!-- 声明子模块需要用的jar包，如果父工程已经有定义了，则不需要声明版本号 -->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

- 实体类

```
@NoArgsConstructor
@Data
//访问，链式访问
@Accessors(chain = true)
public class Dept implements Serializable {
    /**
     * 主键
     */
    private Long deptno;
    /**
     * 部门名臣
     */
    private String dname;
    /**
     * 来自哪个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同的数据库
     */
    private String db_source;

    public Dept(String dname) {
        this.dname = dname;
    }
}
```

### 服务提供方

模块名：microservicecloud-provider-dept-8001，端口为8001。

#### pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-provider-dept-8001</artifactId>
    <description>部门微服务提供者</description>

    <dependencies>
        <!-- 引入通用api模块 -->
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### 启动类

使用注解`@EnableEurekaClient`，表示当前为Eureka的客户端。
使用注解`@EnableDiscoveryClient`，开启服务发现，用于从注册中心获取服务。

```
@SpringBootApplication
//表示当前为Eureka的客户端
@EnableEurekaClient
//开启服务发现
@EnableDiscoveryClient
public class DeptProvider8001App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001App.class, args);
    }
}
```

#### application.yml

配置`mybatis`、`datasource`以及`eureka`。

```
server:
  port: 8001

mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml              #mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities          #所有Entity别名类所在包
  mapper-locations:
    - classpath:mybatis/mapper/**/*.xml                           #mapper映射文件

spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource                  #当前数据源操作类型
    driver-class-name: com.mysql.jdbc.Driver                      #mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01
    username: root
    password: hezihao123
    dbcp2:
      min-idle: 5                                                 #数据库连接池的最小维持连接数
      initial-size: 5                                             #初始化连接数
      max-total: 5                                                #最大连接数
      max-wait-millis: 200                                        #等待连接获取的最大超时时间

eureka:
  client: #客户端注册到Eureka服务端中
    service-url:
      #单机 defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: microservicecloud-dept8001 #自定义服务名称信息
    prefer-ip-address: true #访问路径可以显示IP地址

info: #配置微服务的info信息
  app.name: atguigu.microservicecloud
  company.name: www.atguigu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

#### mybatis配置文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

#### dao层和mapper文件

- dao接口，或者叫Mapper

```
@Repository
@Mapper
public interface DeptDao {
    /**
     * 增加部门
     */
    boolean addDept(Dept dept);

    /**
     * 按部门Id查找部门信息
     */
    Dept findById(Long id);

    /**
     * 查询所有部门
     */
    List<Dept> findAll();
}
```

- Mapper的xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.springcloud.dao.DeptDao">
    <insert id="addDept" parameterType="com.atguigu.springcloud.entities.Dept">
        INSERT INTO dept(dname,db_source) VALUES(#{dname}, DATABASE());
    </insert>

    <select id="findById" resultType="com.atguigu.springcloud.entities.Dept" parameterType="Long">
        SELECT deptno,dname,db_source FROM dept WHERE deptno = #{deptno};
    </select>

    <select id="findAll" resultType="com.atguigu.springcloud.entities.Dept">
        SELECT deptno,dname,db_source FROM dept
    </select>
</mapper>
```

#### Service层

- Service层接口

```
public interface DeptService {
    /**
     * 增加部门
     */
    boolean add(Dept dept);

    /**
     * 按部门Id查找部门信息
     */
    Dept get(Long id);

    /**
     * 查询所有部门
     */
    List<Dept> list();
}
```

- 实现类

```
@Service
public class DeptServiceImpl implements DeptService {
    @Autowired
    private DeptDao deptDao;

    @Override
    public boolean add(Dept dept) {
        return deptDao.addDept(dept);
    }

    @Override
    public Dept get(Long id) {
        return deptDao.findById(id);
    }

    @Override
    public List<Dept> list() {
        return deptDao.findAll();
    }
}
```

#### Controller层

提供部门表的增删查改，以及提供了一个查询自己服务信息的接口`/discovery`。

```
@RestController
@RequestMapping("/dept")
public class DeptController {
    @Autowired
    private DeptService deptService;
    /**
     * 服务发现客户端
     */
    @Autowired
    private DiscoveryClient client;

    /**
     * 增加部门
     */
    @RequestMapping(value = "/add", method = RequestMethod.POST)
    public boolean add(@RequestBody Dept dept) {
        return deptService.add(dept);
    }

    /**
     * 按部门Id查找部门信息
     */
    @RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id) {
        return deptService.get(id);
    }

    /**
     * 查询所有部门
     */
    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public List<Dept> list() {
        return deptService.list();
    }

    /**
     * 测试，从注册中心获取服务列表，查询自己的服务
     */
    @RequestMapping(value = "/discovery", method = RequestMethod.GET)
    public Object discovery() {
        //获取服务列表
        List<String> list = client.getServices();
        System.out.println("*****************" + list);
        //查询自己的服务，因为可能是集群，有多个实例，所以返回是一个列表
        List<ServiceInstance> srvList = client.getInstances("MICROSERVICECLOUD-DEPT");
        for (ServiceInstance element : srvList) {
            System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t" + element.getUri());
        }
        return this.client;
    }
}
```

## Ribbon

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡的工具**。
简单的说，Ribbon是Netflix发 布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。
Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer (简称LB) 后
面所有的机器，Ribbon会自动的帮助你基 于某种规则(如简单轮询，随机连接等)去连接这些机器。我们也很容易使用Ribbon实
现自定义的负载均衡算法。

如果要直接用Ribbon，一般会使用`RestTemplate`，虽然它写起来没有那么方便，不然怎么会有`Feign`呢，但还是可以学习一下。

#### 服务消费方

模块名：microservicecloud-consumer-dept-80，端口为80。

### pom文件

引入通用API、EurekaClient、Ribbon等依赖，因为该模块是消费方，我们将模块作为Web模块，提供接口访问。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-consumer-dept-80</artifactId>
    <description>部门微服务消费者</description>

    <dependencies>
        <!-- 引入通用api模块 -->
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- Eureka Client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <!-- Ribbon相关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

### application.yml

主要配置eureka server的地址。

```
server:
  port: 80

eureka:
  client:
    register-with-eureka: false #不注册到注册中心
    # 配置eureka server的地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

### 启动类

- `@EnableEurekaClient`，配置Eureka客户端
- `@RibbonClient`，配置Ribbon

```
@SpringBootApplication
//配置Eureka客户端
@EnableEurekaClient
//启动该微服务时，自动加载我们的自定义的Ribbon配置类，从而配置生效
@RibbonClient(name = "MICROSERVICECLOUD-DEPT")
public class DeptConsumer80App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80App.class, args);
    }
}
```

### Ribbon配置类

提供`RestTemplate`实例，并且方法上要加上`@LoadBalanced`注解，表示开启Ribbon的负载均衡。

```
@Configuration
public class ConfigBean {
    @Bean
    //开启Ribbon的负载均衡
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

### 使用RestTemplate调用服务提供方

该控制器就是将服务提供方的接口，自己重新提供一套接口，接口中再调用服务提供方来实现。

RestTemplate需要服务提供方的接口，但接口HOST部分我们只需要写为服务名即可，Ribbon会查询到服务列表后，按照策略选择一个服务，拿它的地址替换掉。

```
@RestController
public class DeptControllerConsumer {
    //服务提供方的接口前缀，host部分写服务名即可，Ribbon会查询到服务列表后，按照策略选择一个服务，拿它的地址替换掉
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";

    @Autowired
    private RestTemplate restTemplate;

    /**
     * 增加一个部门
     */
    @RequestMapping(value = "/consumer/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept) {
        String url = REST_URL_PREFIX + "/dept/add";
        //第一个参数：Url地址，第二个参数：请求参数，第三个参数，响应的返回值的Class
        return restTemplate.postForObject(url, dept, Boolean.class);
    }

    /**
     * 按Id查询部门信息
     */
    @RequestMapping(value = "/consumer/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id) {
        String url = REST_URL_PREFIX + "/dept/get/" + id;
        return restTemplate.getForObject(url, Dept.class);
    }

    /**
     * 查询所有部门
     */
    @RequestMapping(value = "/consumer/dept/list", method = RequestMethod.GET)
    public List<Dept> list() {
        String url = REST_URL_PREFIX + "/dept/list";
        return restTemplate.getForObject(url, List.class);
    }

    /**
     * 测试服务发现
     */
    @RequestMapping(value = "/consumer/dept/discovery", method = RequestMethod.GET)
    public Object discovery() {
        String url = REST_URL_PREFIX + "/dept/discovery";
        return restTemplate.getForObject(url, Object.class);
    }
}
```

### 自定义Ribbon策略

- 提供策略配置类

提供一个配置类`MySelfRule`，返回IRule的实例，你可以选择已经定义好的策略，或者自定义自己的策略类

```
@Configuration
public class MySelfRule {
    /**
     * 配置负载均衡算法策略
     */
    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```

- 修改启动类

`@RibbonClient`注解上，配置`configuration`属性，指定上面的自定义策略配置类`MySelfRule`。

需要注意配置类的包名和启动类不能同包，例如启动类的包为`com.atguigu.springcloud`，那么`MySelfRule`就不能在这个包内，否则所有的微服务都用同一个策略了，你可以单独一个包，例如`com.atguigu.myrule`。

```
@SpringBootApplication
//配置Eureka客户端
@EnableEurekaClient
//启动该微服务时，自动加载我们的自定义的Ribbon配置类，从而配置生效
//注意MySelfRule配置类不能放到包扫描的包及其自包下，否则锁引用到所有微服务，就不能单独配置了，所以我们新建一个myrule包来放
@RibbonClient(name = "MICROSERVICECLOUD-DEPT", configuration = MySelfRule.class)
public class DeptConsumer80App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80App.class, args);
    }
}
```

## Feign

Feign是一个声明式WebService客户端。 使用Feign能让编写Web Service客户端更加简单,它的使用方法是定义一个接口,然后
在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支 持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，
使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

一句话讲：Feign就是以接口+注册形式，使用Web服务客户端通信的工具。

### Feign能干什么

Feign旨在使编写Java Http客户端变得更容易。

前面在使用Ribbon + RestTemplate时，利用RestTemplate对http请求的封装处理，形成了-套模版化的调用方法。但是在实际
开发中，由于对服务依赖的调用可能不止一处, 往往一个接口会被多处调用， 所以通常都会针对每个微服务自行封装一些客户端
类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装， 由他来帮助我们定义和实现依赖服务接口的定义。在
Feign的实现下，我们只需创建-个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口
上面标注一个Feign注 解即可)，即可完成对服务提供方的接口绑定,简化了使用Spring cloud Ribbon时，自动封装服务调用客户
端的开发量。

### Feign和Ribbon有什么不同

Feign集成了Ribbon。利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息, 并且通过轮询实现了客户端的负载均衡。
而与Ribbon不同的是,通过feign只需要定义服务绑定接口且以声明式的方法,优雅而简单的实现了服务调用。

### 编写使用Feign的客户端

模块名：microservicecloud-consumer-dept-feign，端口为80。

#### pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-consumer-dept-feign</artifactId>

    <dependencies>
        <!-- 引入通用api模块 -->
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <!-- Eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <!-- Ribbon相关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

主要是配置eureka的服务地址。

```
server:
  port: 80

eureka:
  client:
    register-with-eureka: false #不注册到注册中心
    # 配置eureka server的地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

#### 启动类

- `@EnableEurekaClient`注解，配置Eureka客户端
- `@EnableFeignClients`注解，配置使用Feign客户端，配置要扫描的包

```
@SpringBootApplication
//配置Eureka客户端
@EnableEurekaClient
//配置使用Feign客户端，配置要扫描的包
@EnableFeignClients(basePackages = "com.atguigu.springcloud")
public class DeptConsumer80FeignApp {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80FeignApp.class, args);
    }
}
```

#### 编写Feign接口

Feign接口，就是将服务提供方的提供的接口，在接口再重新写一遍，并且接口上使用注解`@FeignClient`，`value`属性指定服务的名称。

```
//注解@FeignClient，value为要访问的微服务的名称
@FeignClient(value = "MICROSERVICECLOUD-DEPT")
public interface DeptClientService {
    /**
     * 按部门Id查找部门信息
     */
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    Dept get(@PathVariable("id") Long id);

    /**
     * 查询所有部门
     */
    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    List<Dept> list();

    /**
     * 增加部门
     */
    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    boolean add(Dept dept);
}
```

#### Controller层

注入Feign接口，提供自己的接口，声明式调用服务提供者。

```
@RestController
public class DeptControllerConsumer {
    /**
     * 注入Feign接口，声明式调用服务提供者
     */
    @Autowired
    private DeptClientService service;

    /**
     * 增加一个部门
     */
    @RequestMapping(value = "/consumer/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept) {
        return service.add(dept);
    }

    /**
     * 按Id查询部门信息
     */
    @RequestMapping(value = "/consumer/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id) {
        return service.get(id);
    }

    /**
     * 查询所有部门
     */
    @RequestMapping(value = "/consumer/dept/list", method = RequestMethod.GET)
    public List<Dept> list() {
        return service.list();
    }
}
```

## hystrix熔断器

Hystrix是一个用于处理分布式系统的延迟和容错的开源库, 在分布式系统里，许多依赖不可避免的会调用失败,比如超时、异常等
，Hystrix能够保证在一 个依赖出问题的情况下，不会导致整体服务失败,避免级联故障,以提高分布式系统的弹性。

"断路器”本身是一种开关装置，当某个服务单元发生故障之后,通过断路器的故障监控(类似熔断保险丝)， 向调用方返回-个
符合预期的、可处理的备选响应(FallBack) ，而不是长时间的等待或者抛出调用方无法处理的异常,这样就保证了服务调用方的
线程不会被长时间、不必要地占用,从而避免了故障在分布式系统中的蔓延，乃至雪崩。

### 什么是Hystrix的服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制。

当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误”的响应
信息。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。 Hystrix会监控微
服务间调用的状况，当失败的调用到一定阈值,缺省是 5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand

### 什么是服务降级

整体资源快不够时，忍痛将某些服务关闭，待渡过难关，再开启回来。关闭时，使用服务降级处理，返回可处理的提示信息，而不会挂起耗死服务器。

### 服务熔断和服务降级的区别

- 服务熔断，从服务端处理，当服务器发生故障或异常引起，类似保险丝，当某个异常条件被触发，直接熔断整个服务，而不是一直等此服务超时。
- 服务降级，从客户端进行处理，当服务不可用，可能挂掉了，或者响应超时，使用本地的fallback方法进行处理，发挥一个缺省值，这样做，虽然服务水平下降，但好歹可用，比直接挂掉强。

### （服务熔断）使用hystrix熔断器

新建模块：microservicecloud-provider-dept-hystrix-8001

服务熔断，是在`服务方`处理的，与`客户端`无关。

#### pom文件

主要是配置`Hystrix熔断器`的依赖。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-provider-dept-hystrix-8001</artifactId>

    <dependencies>
        <!-- 引入通用api模块 -->
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- Hystrix熔断器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### Controller层

我们以`get()`方法为例，如果获取不到部门信息，就产生熔断，走我们的兜底方法`processHystrixGet`。

注意兜底方法的`入参`必须和`接口一致`，否则会映射不起来，Hystrix会报错。

我们调`get`接口，传入一个不存在的id，即可测试。

```
@RestController
@RequestMapping("/dept")
public class DeptController {
    @Autowired
    private DeptService deptService;

    //省略其他接口方法...

    /**
     * 按部门Id查找部门信息
     */
    @RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
    //熔断配置，一般服务方法抛出异常，自动调用配置的兜底fallback方法，返回可处理的异常信息
    @HystrixCommand(fallbackMethod = "processHystrixGet")
    public Dept get(@PathVariable("id") Long id) {
        Dept dept = deptService.get(id);
        if (dept == null) {
            throw new RuntimeException("查找不到id:" + id + "的部门信息");
        }
        return dept;
    }

    /**
     * 兜底处理方法
     */
    private Dept processHystrixGet(@PathVariable("id") Long id) {
        Dept dept = new Dept();
        dept.setDeptno(id)
                .setDname("查找不到id:" + id + "的部门信息")
                .setDb_source("no this database in MySQL");
        return dept;
    }

    //省略其他接口方法...
}
```

### （服务降级）使用hystrix熔断器改造feign调用

在`microservicecloud-consumer-dept-feign`模块，增加hystrix使用。

服务降级，是在`客户端`处理的，和服务端无关。

#### application.yml

```
server:
  port: 80

feign:
  # 开启hystrix熔断器
  hystrix:
    enabled: true

eureka:
  client:
    register-with-eureka: false #不注册到注册中心
    # 配置eureka server的地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

####  feign接口，添加服务降级工厂

- 降级工厂

```
//记得添加注解@Component，否则无效！
@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(Long id) {
                Dept dept = new Dept();
                dept.setDeptno(id)
                        .setDname("查找不到id:" + id + "的部门信息，此刻服务已经关闭")
                        .setDb_source("no this database in MySQL");
                return dept;
            }

            @Override
            public List<Dept> list() {
                return new ArrayList<>();
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

- feign接口中配置上降级工厂

```
//注解@FeignClient，value为要访问的微服务的名称，fallbackFactory为服务降级工厂
@FeignClient(value = "MICROSERVICECLOUD-DEPT", fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {
    /**
     * 按部门Id查找部门信息
     */
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    Dept get(@PathVariable("id") Long id);

    /**
     * 查询所有部门
     */
    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    List<Dept> list();

    /**
     * 增加部门
     */
    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    boolean add(Dept dept);
}
```

## Zuul网关

Zuul包含了对请求的路由和过滤两个最主要的功能:

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过
程进行干预，是实现请求校验、服务聚合等功能的基础。
Zuul和Eureka进行整合,将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意: Zuul服务最终还是会注册进Eureka！！！

Zuul提供：代理+路由+过滤，三大作用

- 默认网关地址组成规则：网关地址 + 微服务名 + url地址

    1. 启用Zuul前：http://localhost:8001/dept/get/1
    2. 启用Zuul后：http://myzuul.com:9527/microservicecloud-dept/dept/get/1
    
不想暴露微服务名称，需要加配置隐藏，并且一般会统一前缀

### 网关服务

#### pom文件

主要是zuul依赖。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-zuul-gateway-9527</artifactId>

    <dependencies>
        <!-- zuul网关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <!-- hystrix熔断器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <!-- hystrix监控 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

主要是zuul的配置路由映射，隐藏服务名称，将某个服务的地址，改为一个规范的名字。

```
server:
  port: 9527

spring:
  application:
    name: microservicecloud-zuul-gateway

eureka:
  client: #客户端注册到Eureka服务端中
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: gateway-9527.com #自定义hystrix相关的服务名称信息
    prefer-ip-address: true #访问路径可以显示IP地址

zuul:
  #统一前缀
  prefix: /atguigu
  # 隐藏微服务名，统一使用别名
  #ignored-services: microservicecloud-dept
  ignored-services: "*" #星号代表通配，隐藏所有
  routes: #路由映射，不再使用微服务名称作为地址一部分，而是自定义一个地址mydept
    mydept:
      serviceId: microservicecloud-dept
      path: /mydept/**

info: #配置微服务的info信息
  app.name: atguigu.microservicecloud
  company.name: www.atguigu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

#### 启动类

`@EnableZuulProxy`注解，标识当前服务为Zuul网关

```
@SpringBootApplication
//标识当前服务为Zuul网关
@EnableZuulProxy
public class Zuul9527StarSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(Zuul9527StarSpringCloudApp.class, args);
    }
}
```

## 配置中心

主要是负责总多服务的一些配置管理，是一个统一的配置中心。

我们的配置文件放在Github的一个单独的仓库中，再将服务配置成从该仓库中读取配置。

### 配置文件仓库

配置文件可以参考：[配置文件仓库](https://github.com/hezihaog/microservicecloud-config)

### 使用配置改造项目

改造模块有以下4个

1. 服务消费方Consumer
2. 服务提供方Provider
3. Eureka注册中心

### 搭建配置中心

#### pom文件

主要是配置中心的依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-3344</artifactId>

    <dependencies>
        <!-- config配置中心 服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <!-- 避免eclipse的Git插件报错 -->
        <dependency>
            <groupId>org.eclipse.jgit</groupId>
            <artifactId>org.eclipse.jgit</artifactId>
            <version>4.6.0.201612231935-r</version>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <!-- hystrix熔断器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <!-- hystrix监控 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

该文件，使用远程仓库中的application.yml

```
server:
  port: 3344

spring:
  application:
    name: microservicecloud-config
  cloud:
    config:
      server:
        git:
          # Github中的配置文件仓库：https://github.com/hezihaog/microservicecloud-config.git
          # 或者使用ssh，则 uri: git@github.com:hezihaog/microservicecloud-config.git #Github上面的git仓库地址
          uri: https://github.com/hezihaog/microservicecloud-config.git #Github上面的git仓库地址
```

#### 远程仓库的application.yml

配置中心自己的配置文件。这里dev测试环境和test测试环境，让服务的名称不同。

```
spring: 
  profiles: 
    actives: 
    - dev
---
spring: 
  profiles: dev #开发环境
  application: 
    name: microservicecloud-config-atguigu-dev
---
spring: 
  profiles: test #测试环境
  application: 
    name: microservicecloud-config-atguigu-test
# 请保存为UTF-8格式
```

#### 启动类

`@EnableConfigServer`注解，标识当前服务是配置中心

```
@SpringBootApplication
//标识为Config配置中心
@EnableConfigServer
public class Config3344StartSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(Config3344StartSpringCloudApp.class, args);
    }
}
```

### 服务消费方Consumer使用配置中心

#### pom文件

依赖要使用配置中心客户端的依赖。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-client-3355</artifactId>

    <dependencies>
        <!-- config配置中心 客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- 避免eclipse的Git插件报错 -->
        <dependency>
            <groupId>org.eclipse.jgit</groupId>
            <artifactId>org.eclipse.jgit</artifactId>
            <version>4.6.0.201612231935-r</version>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <!-- hystrix熔断器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <!-- hystrix监控 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

因为配置都放在远程仓库，所以本地的application.yml文件只配置一些死的配置。例如服务名称，一般都不会动态变更。

```
spring:
  application:
    name: microservicecloud-config-client
```

#### bootstrap.yml

配置远程仓库信息

1. uri：从哪个git仓库拉取
2. name: 拉取的文件名
3. profile: 拉取的环境名称，例如test、dev等
4. label：拉取的仓库的分支

```
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: dev #本次访问的配置项
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号微服务，通过SpringCloud获取Github的服务地址
```

#### microservicecloud-config-client.yml

远程仓库中，服务消费方的配置文件。

```
spring: 
  profiles: 
    actives: 
    - dev
---
server: 
  port: 8201
spring: 
  profiles: dev
  application: 
    name: microservicecloud-config-client
eureka:
  client: 
    server-url: 
      default-zone: http://eureka-dev.com:7001/eureka/

---
server: 
  port: 8202
spring: 
  profiles: test
  application: 
    name: microservicecloud-config-client
eureka:
  client: 
    server-url: 
      default-zone: http://eureka-test.com:7001/eureka/
```

#### 启动类

```
@SpringBootApplication
public class ConfigClient3355StartSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient3355StartSpringCloudApp.class, args);
    }
}
```

#### 使用远程配置的配置信息

- 提供一个Rest控制器，使用接口，获取配置的值

```
@RestController
public class ConfigClientRest {
    /**
     * 服务名称
     */
    @Value("${spring.application.name}")
    private String applicationName;
    /**
     * eureka服务地址
     */
    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServers;
    /**
     * 服务端口号
     */
    @Value("${server.port}")
    private String port;

    /**
     * 获取配置
     */
    @RequestMapping("/config")
    public String getConfig() {
        String str = "applicationName: " + applicationName + "\t eurekaServers: " + eurekaServers + "\t port: " + port;
        System.out.println("********** str: " + str);
        return str;
    }
}
```

- 你会发现启动会异常，因为本地配置文件没有那个配置了，Spring找不到时会报错，我们希望Spring找不到，也不报错，就需要配置一下。

```
//解决Spring的@Value注解在当前的yml配置文件中，没有找到属性时报错
@Configuration
public class PropertySourcePlaceholderConfig {
    /**
     * 忽略没有找到属性，因为我们的配置在拉取远程配置后进行动态配置的，本地文件中没有
     */
    @Bean
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        configurer.setIgnoreUnresolvablePlaceholders(true);
        return configurer;
    }
}
```

### 服务提供方Provider使用配置中心

#### pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-dept-client-8001</artifactId>

    <dependencies>
        <!-- 引入通用api模块 -->
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- 将服务注册到eureka注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- actuator监控和信息配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

同样，application.yml中只配置死的信息，bootstrap.yml中配置动态的配置信息。

```
spring:
  application:
    name: microservicecloud-config-dept-client
```

#### bootstrap.yml

```
spring:
  cloud:
    config:
      name: microservicecloud-config-dept-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: dev #开发环境
      #profile: test #测试环境
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号微服务，通过SpringCloud获取Github的服务地址
```

#### microservicecloud-config-dept-client.yml

远程仓库中，服务提供方的配置文件。

```
spring:
  profiles:
  actives:
    - dev
---
server:
  port: 8001

mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml              #mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities          #所有Entity别名类所在包
  mapper-locations:
    - classpath:mybatis/mapper/**/*.xml                           #mapper映射文件

spring:
  profiles: dev
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource                  #当前数据源操作类型
    driver-class-name: com.mysql.jdbc.Driver                      #mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB03
    username: root
    password: hezihao123
    dbcp2:
      min-idle: 5                                                 #数据库连接池的最小维持连接数
      initial-size: 5                                             #初始化连接数
      max-total: 5                                                #最大连接数
      max-wait-millis: 200                                        #等待连接获取的最大超时时间

eureka:
  client: #客户端注册到Eureka服务端中
    service-url:
      #单机 defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: microservicecloud-dept8001 #自定义服务名称信息
    prefer-ip-address: true #访问路径可以显示IP地址

info: #配置微服务的info信息
  app.name: atguigu.microservicecloud
  company.name: www.atguigu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
---
server:
  port: 8001

mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml              #mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities          #所有Entity别名类所在包
  mapper-locations:
    - classpath:mybatis/mapper/**/*.xml                           #mapper映射文件

spring:
  profiles: test
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource                  #当前数据源操作类型
    driver-class-name: com.mysql.jdbc.Driver                      #mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB02
    username: root
    password: hezihao123
    dbcp2:
      min-idle: 5                                                 #数据库连接池的最小维持连接数
      initial-size: 5                                             #初始化连接数
      max-total: 5                                                #最大连接数
      max-wait-millis: 200                                        #等待连接获取的最大超时时间

eureka:
  client: #客户端注册到Eureka服务端中
    service-url:
      #单机 defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: microservicecloud-dept8001 #自定义服务名称信息
    prefer-ip-address: true #访问路径可以显示IP地址

info: #配置微服务的info信息
  app.name: atguigu.microservicecloud
  company.name: www.atguigu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

#### 启动类

```
@SpringBootApplication
//表示当前为Eureka的客户端
@EnableEurekaClient
//开启服务发现
@EnableDiscoveryClient
public class ConfigClientDeptProvider8001App {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientDeptProvider8001App.class, args);
    }
}
```

### 注册中心Eureka使用配置中心

#### pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 统一父工程 -->
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-eureka-client-7001</artifactId>

    <dependencies>
        <!-- config配置中心 客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- eureka server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### application.yml

application.yml只配置死的信息，bootstrap.yml配置动态配置信息

```
spring:
  application:
    name: microservicecloud-config-eureka-client
```

#### bootstrap.yml

```
spring:
  cloud:
    config:
      name: microservicecloud-config-eureka-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: dev #本次访问的配置项
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号微服务，通过SpringCloud获取Github的服务地址
```

#### microservicecloud-config-eureka-client.yml

远程仓库中，Eureka的配置文件。

```
spring:
  profiles:
  actives:
    - dev
---
server:
  port: 7001 #注册中心占用7001端口，冒号后面必须要有空格

spring:
  profiles: dev
  application:
    name: microservicecloud-config-eureka-client

eureka:
  instance:
    hostname: eureka7001.com #冒号后面必须要有空格
  client:
    register-with-eureka: false #当前eureka-server自己不注册进服务列表中
    fetch-registry: false #不通过eureka获取注册信息
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
---
server:
  port: 7001 #注册中心占用7001端口，冒号后面必须要有空格

spring:
  profiles: test
  application:
    name: microservicecloud-config-eureka-client

eureka:
  instance:
    hostname: eureka7001.com #冒号后面必须要有空格
  client:
    register-with-eureka: false #当前eureka-server自己不注册进服务列表中
    fetch-registry: false #不通过eureka获取注册信息
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

#### 启动类

```
@SpringBootApplication
//告诉SpringBoot，本文件为Eureka Server启动类
@EnableEurekaServer
public class ConfigGitEurekaServerApp {
    public static void main(String[] args) {
        SpringApplication.run(ConfigGitEurekaServerApp.class, args);
    }
}
```

## 总结

记录可能不是很清楚，有兴趣的同学，可以clone

[代码仓库](https://github.com/hezihaog/microservicecloud)

[配置中心](https://github.com/hezihaog/microservicecloud-config)