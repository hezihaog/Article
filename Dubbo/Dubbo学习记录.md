# Dubbo学习记录

本篇是学习了[Dubbo 视频教程全集（30P）| 4 小时从入门到精通](https://www.bilibili.com/video/av62353311/)后，总结的记录。

## Dubbo是什么？

![Dubbo](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ec48e57e2474ddc9b48552943602e5d~tplv-k3u1fbpfcp-zoom-1.image)

Dubbo作为阿里巴巴提供的分布式RPC和微服务框架，提供了基于接口的远程调用，容错和负载均衡，服务注册与发现功能。

## 什么是RPC

指远程过程调用，是一种进程间的通信方式。这是一种思想，而不是规范。

## Dubbo的三大核心能力

- 面向接口的远程方法调用
- 智能容错和负载均衡
- 服务的自主注册和发现

## Dubbo的架构

![流程图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc6c137aee334937a2106d7acbfe2bfb~tplv-k3u1fbpfcp-zoom-1.image)

- Provider：暴露服务的服务者
- Consumer：调用远程服务的服务消费者
- Registry：服务注册与发现的注册中心
- Monitor：统计服务的调用次数和调用时间的监控中心
- Container：服务运行容器

## Dubbo的服务注册与发现流程

- 服务提供者（Provider）：服务者，负责暴露服务，启动时，向注册中心注册服务。
- 服务消费者（Consumer）：调用远程服务的服务消费者，启动时，向注册中心订阅服务，获取服务者的地址列表，基于软负载均衡算法，选取一个服务进行调用，如果调用失败，选取下一台。
- 注册中心（Registry）：注册中心维护服务提供者的地址列表，如果有变更，通过长连接推送给服务消费者。
- 监控中心（Monitor）：服务消费者和提供者之间的调用次数、调用时间等，定时发送给监控中心。

### 调用关系

- 服务容器负责启动、加载、运行服务提供者。
- 服务提供者启动时，向注册中心注册自己提供的服务。
- 服务消费者启动时，向注册中心订阅自己所需的服务。
- 注册中心返回服务提供者的地址列表给消费者，如果有变更，会通过长连接推送变更数据给消费者。
- 服务消费者，从服务提供者信息列表中，通过软负载均衡算法，选取一个提供者进行调用，如果调用失败，再选取另外一个进行调用。
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时发送一次到监控中心。

## Dubbo环境搭建

### JDK环境

必须有，大部分都已经有了，如果没有的话，自行配置一下

### 注册中心

Dubbo支持4种，分别为`Multicast`、`Zookeeper`、`Redis`、`Simple`，推荐使用`Zookeeper`，这里也只用`Zookeeper`。

[Zookeeper官方](http://zookeeper.apache.org/)

[Zookeeper下载地址](http://zookeeper.apache.org/releases.html#download)，我选择3.4.14版本

#### 配置环境变量

解压压缩包到指定目录，我选择放在User目录下

```
export ZOOKEEPER_HOME=/Users/wally/zookeeper-3.4.14
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

- 更新环境变量

```
source .bash_profile
```

#### 配置Zookeeper

- 配置Zookeeper的配置文件

进入Zookeeper的目录，找到conf子目录，默认会有一个配置文件模板zoo_sample.cfg，复制一份，重命名为zoo.cfg。


1. 编辑zoo.cfg，将dataDir属性的值改为../data，就是将数据目录配置到了Zookeeper根目录下的data目录。
2. 在Zookeeper根目录下新建一个data目录，作为Zookeeper的数据存放目录。

- clientPort，为Zookeeper要使用的端口号

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=../data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

#### Zookeeper常用命令

去到bin目录，终端下使用zkServer.sh

```
//启动
zkServer.sh start
//停止
zkServer.sh stop
//查看状态
zkServer.sh status
```

使用Zookeeper的客户端cli连接Zookeeper

```
zkCli.sh  -server localhost
```

### admin管理控制台

为了让用户更好的管理监控众多的dubbo服务，官方提供了一个可视化的监控程序，不是必须，其实不装，也不影响Dubbo的使用。

- 下载[dubbo-admin](https://github.com/apache/incubator-dubbo-ops)
- 进入`src\main\resources\`目录，修改`application.properties`，指定zookeeper地址：`dubbo.registry.address=zookeeper://127.0.0.1:2181`
- 打包dubbo-admin：`mvn clean package -Dmaven.test.skip=true`
- 运行dubbo-admin：java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
- 登录admin，用户名和密码都为root

## 使用案例

有2个服务，`用户服务`和`订单服务`，订单服务依赖用户服务。所以订单服务为`服务消费方`，而用户服务为`提供方`。
订单服务初始化订单时，会传入userId，订单服务调用用户服务，获取用户的收货地址列表，再返回。

### 公共模块

#### 用户地址-实体类

```
public class UserAddress implements Serializable {
    private Integer id;
    /**
     * 用户地址
     */
    private String userAddress;
    /**
     * 用户Id
     */
    private String userId;
    /**
     * 收货人
     */
    private String consignee;
    /**
     * 电话号码
     */
    private String phoneNum;
    /**
     * 是否为默认地址，Y-是 N-否
     */
    private String idDefault;

    public UserAddress(Integer id, String userAddress, String userId, String consignee, String phoneNum, String idDefault) {
        this.id = id;
        this.userAddress = userAddress;
        this.userId = userId;
        this.consignee = consignee;
        this.phoneNum = phoneNum;
        this.idDefault = idDefault;
    }

    //省略get、set方法

    @Override
    public String toString() {
        return "UserAddress{" +
                "id=" + id +
                ", userAddress='" + userAddress + '\'' +
                ", userId='" + userId + '\'' +
                ", consignee='" + consignee + '\'' +
                ", phoneNum='" + phoneNum + '\'' +
                ", idDefault='" + idDefault + '\'' +
                '}';
    }
}
```

#### Service层接口

- 订单服务

```
public interface OrderService {
    /**
     * 初始化订单
     *
     * @param userId 用户Id
     */
    List<UserAddress> initOrder(String userId);
}
```

- 用户服务

```
public interface UserService {
    /**
     * 按照用户Id，获取用户的所有收货地址
     *
     * @param userId 用户Id
     */
    List<UserAddress> getUserAddressList(String userId);
}
```

#### 建立服务提供者Provider

服务者模块名：user-service-provider

- pom文件

配置Dubbo依赖，Zookeeper依赖、日志等

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.gmail</groupId>
    <artifactId>user-service-provider</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- 公共API接口 -->
        <dependency>
            <groupId>com.atguigu.gmail</groupId>
            <artifactId>gmall-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- 日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <!-- dubbo依赖 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>
        <!-- 注册中心zookeeper客户端 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
    </dependencies>
</project>
```

- log4j日志配置

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

- 编写用户服务实现

dubbo支持多个实现类来做多版本，一般用来做灰度发布使用。

注意，这个`@Service`注解是Spring的，不是Dubbo的，不需要导错包了。

```
//1.0.0版本，用户接口实现类
@Service
public class UserServiceImpl implements UserService {
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("---------- 线上版本订单服务，版本1.0.0 ----------");

        UserAddress userAddress1 = new UserAddress(1,
                "北京市昌平区xx科技园综合楼3层",
                userId,
                "李老师",
                "010-yyyyy",
                "Y");
        UserAddress userAddress2 = new UserAddress(2,
                "深圳市宝安区西部硅谷大厦B座3层",
                userId,
                "王老师",
                "010-xxxxx",
                "N"
        );
        return Arrays.asList(userAddress1, userAddress2);
    }
}
//2.0.0版本的实现
@Service
public class UserServiceImpl2 implements UserService {
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("---------- 灰度订单服务，版本2.0.0 ----------");

        UserAddress userAddress1 = new UserAddress(1,
                "北京市昌平区xx科技园综合楼3层",
                userId,
                "李老师",
                "010-yyyyy",
                "Y");
        UserAddress userAddress2 = new UserAddress(2,
                "深圳市宝安区西部硅谷大厦B座3层",
                userId,
                "王老师",
                "010-xxxxx",
                "N"
        );
        return Arrays.asList(userAddress1, userAddress2);
    }
}
```

- provider配置文件

配置一个服务提供者，需要6步，最后一步不是必须的，可以不配置监控中心。

1. 配置当前服务/应用的名字
2. 指定注册中心的位置，有2种写法，第一种是协议名和地址合起来一起写，第二种是分开属性写
3. 指定通信的规则，就是协议和端口
4. 暴露服务，就是暴露刚才上面写的接口实现，并且标签里面指定暴露的服务方法
5. dubbo支持多版本发布，常用来做灰度发布，和第四步一样，就是`version`属性指定不同的版本
6. 指定配置中心的地址和端口，可以直接使用一个registry关键字，让dubbo自己查找

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 1.配置当前服务/应用的名字 -->
    <dubbo:application name="user-service-provider"/>

    <!-- 2.指定注册中心的位置 -->
    <!--    <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"/>

    <!-- 3.指定通信规则（通信协议、通信端口） -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 4.暴露服务，interface属性指定服务接口，ref指定服务具体实现 -->
    <dubbo:service interface="com.atguigu.gmail.service.UserService" ref="userServiceImpl1"
                   version="1.0.0">
        <dubbo:method name="getUserAddressList" timeout="1000"/>
    </dubbo:service>
    <!-- 服务实现 -->
    <bean id="userServiceImpl1" class="com.atguigu.gmail.service.impl.UserServiceImpl"/>

    <!-- 5.配置一个灰度服务实现 -->
    <dubbo:service interface="com.atguigu.gmail.service.UserService" ref="userServiceImpl2"
                   version="2.0.0"/>
    <bean id="userServiceImpl2" class="com.atguigu.gmail.service.impl.UserServiceImpl2"/>

    <!-- 6.配置监控中心，2种方式，protocol属性写协议，如果写registry，则代表自动从注册中心查找监控中心的位置 -->
    <dubbo:monitor protocol="registry"/>
    <!-- 或者手动指定位置 -->
    <!--    <dubbo:monitor address="127.0.0.1:7070"/>-->
</beans>
```

- 启动类

提供者启动，为了后续的消费方能调用到，先用`System.in.read()`阻塞，不马上结束程序。

```
public class ProviderApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml");
        context.start();
        //阻塞
        System.in.read();
    }
}
```

#### 建立服务消费者Consumer

- pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.gmail</groupId>
    <artifactId>order-service-consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- 公共API -->
        <dependency>
            <groupId>com.atguigu.gmail</groupId>
            <artifactId>gmall-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- 日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <!-- dubbo依赖 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>
        <!-- 注册中心zookeeper客户端 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
    </dependencies>
</project>
```

- log4j配置

和上面服务提供者的一样，拷贝一份即可，不再赘述

- 编写订单服务实现

```
@Service
public class OrderServiceImpl implements OrderService {
    /**
     * 注入用户服务
     */
    @Autowired
    private UserService userService;

    public List<UserAddress> initOrder(String userId) {
        //1.查询用户的搜索地址
        return userService.getUserAddressList(userId);
    }
}
```

- 本地存根

本地存根，用于调用远程服务时，先走一遍自己的本地实现，可以在这里做参数验证、拦截、容错以及结果缓存等操作。
类似于一个静态代理的拦截器，或者说AOP的效果。

```
public class UserServiceStub implements UserService {
    private final UserService userService;

    /**
     * 必须提供一个有参构造器，dubbo会传入远程服务的代理对象，通过这个对象，来进行调用远程方法
     *
     * @param userService 远程服务的代理对象
     */
    public UserServiceStub(UserService userService) {
        this.userService = userService;
    }

    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceStub.getUserAddressList() => 本地存根被调用...");
        try {
            if (StringUtils.isEmpty(userId)) {
                throw new RuntimeException("传入的用户Id不能为空！");
            }
            return userService.getUserAddressList(userId);
        } catch (Exception e) {
            e.printStackTrace();
            //容错处理，如果调用出错，返回空
            return new ArrayList<UserAddress>();
        }
    }
}
```

- consumer配置文件

配置一个服务消费者，需要5步，最后一步监控中心可省略。

1. 开启包扫描，这个是Spring的配置
2. 配置服务消费者的应用名称
3. 指定注册中心的地址。有一个`check`属性，配置注册中心不存在时，是否报错，默认为true，就是会报错
4. 声明需要远程调用的远程服务接口，生成远程服务代理
5. 配置监控中心的地址和端口，也可以指定为registry，代表dubbo自动搜索监控中心

```
<?xml version="1.0" encoding="UTF-8"?>
<beans
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 1.配置包扫描 -->
    <context:component-scan base-package="com.atguigu.gmail.service.impl"/>

    <!-- 2.配置服务消费者的应用名称 -->
    <dubbo:application name="order-service-consumer"/>

    <!-- 3.指定注册中心的地址 -->
    <!-- 3.1check属性，配置注册中心不存在时，是否报错，默认为true，就是会报错 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" check="false"/>

    <!-- 4.声明需要远程调用的远程服务接口，生成远程服务代理 -->
    <!-- 4.1 check属性，启动时是否检查依赖的服务是否启动，默认为true，启动会检查，会导致如果依赖的服务没有启动，就会报错，一般我们设置为false，调用的时候，才去检查对方服务状态 -->
    <!-- 4.2 timeout属性，配置消费者调用服务提供者的方法超过一定时间没响应，就判定为超时，避免大量的阻塞，单位为毫秒，默认值为1000，就是一秒 -->
    <!-- 4.3 retries属性，重试次数，调用远程方法超时时，进行重试，不包含第一次调用（如果服务提供方有多个服务，重试请求会分散到不同的服务中） -->
    <!-- 幂等方法（设置重试次数，来提升系统的性能），非幂等方法（不能设置重试次数） -->
    <!-- 幂等：就是方法无论运行多少次，都是一个效果，例如数据库的查询、删除、修改 -->
    <!-- 非幂等：每次方法运行都是产生一个新的效果，例如数据库新增 -->
    <!-- 如不需要重试，设置值为0即可，代表不重试 -->
    <!-- 4.4 dubbo还支持多版本，就是灰度，version属性指定使用调用哪个版本的实现，还支持*通配符来表示随机选一个来调用，来达到灰度 -->
    <!-- 4.5 配置本地存根，使用stub属性配置本地存根的全路径类名 -->
    <dubbo:reference id="userService" interface="com.atguigu.gmail.service.UserService"
                     check="false" timeout="3000" retries="3"
                     version="*" stub="com.atguigu.gmail.service.stub.UserServiceStub"/>

    <!-- 4.2每个去配置check属性是很麻烦的，我们可以统一配置所有消费者的缺省配置，我们可以默认所有服务都不检查，如果有某一项服务不一样，则单独配置即可 -->
    <!-- 4.3 timeout属性，为所有的消费者提供缺省设置，而不需要去dubbo:reference下每个都配置 -->
    <dubbo:consumer check="false" timeout="1000"/>

    <!-- 5.配置监控中心，2种方式，protocol属性写协议，如果写registry，则代表自动从注册中心查找监控中心的位置 -->
    <dubbo:monitor protocol="registry"/>
    <!-- 或者手动指定位置 -->
    <!--    <dubbo:monitor address="127.0.0.1:7070"/>-->
</beans>
```

- 启动类

我们为了演示方便，直接在启动时，获取服务实例，调用远程服务。

```
public class ConsumerApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("consumer.xml");
        OrderService orderService = context.getBean(OrderService.class);
        orderService.initOrder("1");
        System.out.println("调用完成");
        //阻塞
        System.in.read();
    }
}
```

## 整合SpringBoot的改造

上面的Dubbo使用比较原始，一般在SpringBoot出现前使用，现在基本都用SpringBoot，所以Dubbo也提供了和SpringBoot整合的方式。

#### 建立服务提供者Provider

建立模块：boot-user-service-provider

- pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.gmail</groupId>
    <artifactId>boot-user-service-provider</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- 依赖基础接口 -->
        <dependency>
            <groupId>com.atguigu.gmail</groupId>
            <artifactId>gmall-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- spring boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--热启动-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!-- Dubbo Spring Boot 的 Starter -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
        <!-- 服务容错 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.0.4.RELEASE</version>
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
</project>
```

- application.properties

相当于将之前的provider.xml的内容挪到这里

```
#配置应用名称
dubbo.application.name=boot-user-service-provider

#配置注册中心的位置和注册中心协议
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper

#配置通信规则，协议名以及端口
dubbo.protocol.name=dubbo
dubbo.protocol.port=20881

#配置监控中心
dubbo.monitor.protocol=registry

#配置包扫描，@EnableDubbo注解是配置这个包扫描功能
#dubbo.scan.base-packages=com.atguigu.gmail
```

- dubbo.properties

dubbo还提供了全局配置文件，该文件一般是配置公共配置，application.xx的配置可以覆盖该配置。

```
#配置公共端口号，当在虚拟机参数、xml或application.properties覆盖相同属性时，会进行覆盖
#虚拟机参数：-Ddubbo.protocol.port，值为20880
#优先级：虚拟机参数 > xml或application.properties > dubbo.properties
#一般dubbo.properties文件配置的是公共配置
dubbo.protocol.port=20882
```

- 编写用户服务实现类

SpringBoot抛弃了XML，推荐注解和JavaConfig的方式配置，所以Dubbo也有对应的注解进行配置。

这里的`@Service`注解，是dubbo的注解，用来暴露服务的，同时我们也要Spring管理该Bean，为了和Spring的`@Service`不重名，我们使用`@Component`注解。

```
//1.0.0版本

//注解@Service，标识暴露给dubbo的服务，timeout属性配置远程服务调用超时时间，version属性配置多版本功能，可实现灰度
@Service(timeout = 3000, version = "*")
//因为和spring的@Service注解同名，这里用@Component注解也是一样的
@Component("userService1")
public class BootUserServiceImpl implements UserService {
    /**
     * Hystrix容错处理，该方法会被Hystrix代理，来处理容错异常
     *
     * @param userId 用户Id
     */
    @HystrixCommand
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("---------- 线上版本订单服务，版本1.0.0 ----------");

        UserAddress userAddress1 = new UserAddress(1,
                "北京市昌平区xx科技园综合楼3层",
                userId,
                "李老师",
                "010-yyyyy",
                "Y");
        UserAddress userAddress2 = new UserAddress(2,
                "深圳市宝安区西部硅谷大厦B座3层",
                userId,
                "王老师",
                "010-xxxxx",
                "N"
        );
        return Arrays.asList(userAddress1, userAddress2);
    }
}

//2.0.0版本
//注解@Service，标识暴露给dubbo的服务，timeout属性配置远程服务调用超时时间，version属性配置多版本功能，可实现灰度
@Service(timeout = 3000, version = "*")
//因为和spring的@Service注解同名，这里用@Component注解也是一样的
@Component("userService2")
public class BootUserServiceImpl2 implements UserService {
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("---------- 灰度订单服务，版本2.0.0 ----------");

        UserAddress userAddress1 = new UserAddress(1,
                "北京市昌平区xx科技园综合楼3层",
                userId,
                "李老师",
                "010-yyyyy",
                "Y");
        UserAddress userAddress2 = new UserAddress(2,
                "深圳市宝安区西部硅谷大厦B座3层",
                userId,
                "王老师",
                "010-xxxxx",
                "N"
        );
        return Arrays.asList(userAddress1, userAddress2);
    }
}
```

- 启动类

1. dubbo整合springboot的三种方式
    - 导入dubbo的starter，在application.properties配置属性，使用@Service暴露服务，使用@Reference引用服务，并且Application入口类必须加上@EnableDubbo注解来开启注解功能
    - 导入dubbo的starter，保留dubbo的xml配置文件，再在Application入口类上使用@ImportResource注解，引入xml配置文件，使用原始方式，就不需要用注解了
    - 使用注解API的方式，就是SpringJavaConfig的方式提供类对象，将每个组件手动创建放到Spring容器中，让dubbo扫描出组件

2. 注意！！！上面3种配置方式，不能共存，只能选一种，否则会出现重复定义的异常，例如注解和JavaConfig的方式都做了，而@EnableDubbo内部又使用@DubboComponentScan注解，导致2种配置方式都开启了，出现了重复定义
服务容错，使用Hystrix，使用@EnableHystrix注解开启Hystrix，在需要保护的方法上使用@HystrixCommand注解，在调用方，也依赖Hystrix，同样使用@EnableHystrix注解开启Hystrix，在调用的方法上，也加上@HystrixCommand注解

推荐使用第一种和第三种方式，就是注解方式或者JavaConfig的方式，我更倾向于第一种，配置文件方式会比JavaConfig简洁一些。

```
//1）注解方式，使用@EnableDubbo注解，开启基于注解的dubbo功能，例如@Service注解才能生效
@EnableDubbo
//2）xml方式，使用@ImportResource注解导入配置
//@ImportResource(locations = "classpath:provider.xml")
//3）JavaConfig配置方式，需要使用@DubboComponentScan注解配置扫描包的包名，用来扫描到Dubbo的Java配置类，用@EnableDubbo也是可以的，@EnableDubbo注解就已经引用了@DubboComponentScan注解
//@DubboComponentScan(basePackages = "com.atguigu.gmail")
//开启服务容错
@EnableHystrix
@SpringBootApplication
public class BootProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(BootProviderApplication.class, args);
    }
}
```

- JavaConfig方式配置Dubbo

注意：不能和其他2种同时存在！！

```
//SpringJavaConfig的方式，配置dubbo的组件
@Configuration
public class MyDubboConfig {
    /**
     * 提供Application标签的功能
     * 相当于：dubbo.application.name=boot-user-service-provider
     */
    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig config = new ApplicationConfig();
        config.setName("boot-user-service-provider");
        return config;
    }

    /**
     * 提供注册中心配置
     * 相当于：
     * dubbo.registry.address=127.0.0.1:2181
     * dubbo.registry.protocol=zookeeper
     */
    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig config = new RegistryConfig();
        config.setAddress("127.0.0.1:2181");
        config.setProtocol("zookeeper");
        return config;
    }

    /**
     * 提供通信协议的配置
     * 相当于：
     * dubbo.protocol.name=dubbo
     * dubbo.protocol.port=20881
     */
    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig config = new ProtocolConfig();
        config.setName("dubbo");
        config.setPort(20881);
        return config;
    }

    /**
     * 提供暴露服务的配置
     * 相当于
     * <dubbo:service interface="com.atguigu.gmail.service.UserService" ref="userServiceImpl1"
     * version="1.0.0">
     * <dubbo:method name="getUserAddressList" timeout="1000"/>
     * </dubbo:service>
     */
    @Bean
    public ServiceConfig<UserService> userServiceConfig(@Qualifier("userService1") UserService userService) {
        ServiceConfig<UserService> serviceConfig = new ServiceConfig<>();
        //配置暴露的服务的接口
        serviceConfig.setInterface(UserService.class);
        //配置暴露的服务的具体实现，spring会自动注入到参数上
        serviceConfig.setRef(userService);
        //设置版本
        serviceConfig.setVersion("1.0.0");
        //配置每一个方法的信息
        MethodConfig methodConfig = new MethodConfig();
        //设置方法名
        methodConfig.setName("getUserAddressList");
        //设置超时时间
        methodConfig.setTimeout(1000);
        //将MethodConfig关联到Service的配置中
        serviceConfig.setMethods(Arrays.asList(methodConfig));
        return serviceConfig;
    }

    /**
     * 提供监控中心的配置
     */
    @Bean
    public MonitorConfig monitorConfig() {
        MonitorConfig config = new MonitorConfig();
        config.setProtocol("registry");
        return config;
    }
}
```

- provider.xml

和普通方式配置的provider.xml文件一致，所以不在赘述了。

#### 建立服务消费者Consumer

建立模块：boot-order-service-consumer

- pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.gmail</groupId>
    <artifactId>boot-order-service-consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- 公共API -->
        <dependency>
            <groupId>com.atguigu.gmail</groupId>
            <artifactId>gmall-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- web项目 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--热启动-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!-- Dubbo Spring Boot 的 Starter -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
        <!-- 服务容错 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.0.4.RELEASE</version>
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
</project>
```

- application.properties

```
#配置端口号
server.port=8081
#配置应用名称
dubbo.application.name=boot-order-service-consumer
#配置注册中心地址
dubbo.registry.address=zookeeper://127.0.0.1:2181
#配置监控中心
dubbo.monitor.protocol=registry
```

- 编写订单服务实现

1. 和服务提供者注册的服务类似，也用`@Service`注解进行服务暴露
2. 使用`@Reference`注解，注入依赖的服务，例如这里是用户服务

```
@Service
public class BootOrderServiceImpl implements OrderService {
    /**
     * 依赖注入dubbo的远程服务，因为服务是远程服务，不能用spring的@Autowired注解了
     * 1.url属性，可以实现dubbo直连，直接配置远程服务的ip和端口，不经过注册中心
     * <p>
     * 2.loadbalance属性，负载均衡配置，负载均衡实现有4种，分别是
     * 1）roundrobin：轮训，挨个按顺序分配，第一轮：A-B-C 第二轮：A-B-C
     * 2）leastactive：最小活跃调用数，选中处理速度最快的服务。活跃数指调用前后计数差,优先调用高的，相同活跃数的随机。使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。
     * 3）random：随机抽取，dubbo模式使用这种
     * 4）一致性哈希，只要请求过1次，后面的请求都会找到上次的机器进行处理
     * <p>
     * 3.check属性，是否启动时检查服务提供方是否存在，默认为true，代表检查，如果发现没启动，会报错
     */
//    @Autowired
//    @Reference(url = "127.0.0.1:20881")
//    @Reference(loadbalance = "roundrobin")
    @Reference(check = false)
    private UserService userService;

    /**
     * 使用@HystrixCommand注解，保护调用失败时，调用容错方法进行返回
     */
    @HystrixCommand(fallbackMethod = "fallback")
    public List<UserAddress> initOrder(String userId) {
        //1.查询用户的搜索地址
        return userService.getUserAddressList(userId);
    }

    /**
     * 容错方法，注意方法参数必须和容错方法一致，否则会抛出找不到方法的异常
     */
    private List<UserAddress> fallback(String userId) {
        UserAddress address = new UserAddress(
                0,
                "测试",
                userId,
                "测试",
                "xxxx",
                "N"
        );
        return Arrays.asList(address);
    }
}
```

- Controller层

使用`@Autowired`注解，可以注入暴露的服务，提供`initOrder`接口，进行调用

```
@Controller
public class OrderController {
    @Autowired
    private OrderService orderService;

    /**
     * 初始化订单
     */
    @ResponseBody
    @RequestMapping("/initOrder")
    public List<UserAddress> initOrder(@RequestParam("uid") String userId) {
        return orderService.initOrder(userId);
    }
}
```

- 启动类

重点：使用注解@EnableDubbo，开启dubbo的注解功能。

```
//注解@EnableDubbo，开启dubbo的注解功能
@EnableDubbo
//开启服务容错
@EnableHystrix
@SpringBootApplication
public class BootConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(BootConsumerApplication.class, args);
    }
}
```

## 总结

代码我上传到了[Github](https://github.com/hezihaog/dubbo_sample)，有兴趣的同学可以clone。