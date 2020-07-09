#### SpringMVC入门使用

本篇记录SpringMVC的使用，初学Java后端，各种大佬轻拍。

#### 配置工作

- 导入依赖

使用Spring5.0.2的版本，主要依赖是spring-web，其他都是学习过程中需要jar包，单纯只要SpringMVC不需要导入。

```
<properties>
    <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
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

    <!-- json转换 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.0</version>
    </dependency>

    <!-- 文件上传 -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.4</version>
    </dependency>

    <!-- 跨服务器上传 -->
    <dependency>
        <groupId>com.sun.jersey</groupId>
        <artifactId>jersey-core</artifactId>
        <version>1.18.1</version>
    </dependency>
    <dependency>
        <groupId>com.sun.jersey</groupId>
        <artifactId>jersey-client</artifactId>
        <version>1.18.1</version>
    </dependency>
</dependencies>
```

- 配置web.xml

不管SpringMVC还是其他MVC框架，都是封装servlet的API，而SpringMVC分发请求是通过一个DispatcherServlet，再转发请求到Controller，所以我们需要在web.xml中添加配置。

1. 配置DispatcherServlet前端控制器，在初始化参数中，配置SpringMVC的配置文件为springmvc.xml。
2. 配置解决中文乱码的过滤器，CharacterEncodingFilter，指定编码为utf-8。

```
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!--配置前端控制器-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 加载SpringMVC配置文件 -->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 启动就加载这个Servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--配置解决中文乱码的过滤器-->
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
</web-app>
```

- 配置SpringMVC配置文件

上面DispatcherServlet中，我们指定SpringMVC的配置文件为springmvc.xml，所以我们需要在resources目录下，添加springmvc.xml文件。

1. 配置spring创建容器时要扫描的包
2. 配置视图解析器，指定jsp存放的配置，后缀等，这里指定为webapp下的WEB-INF里面的pages
3. 配置静态资源不拦截，如html、css、js等
4. 配置spring开启注解mvc的支持，开启后，就可以使用SpringMVC的注解

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.itheima"/>

    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 视图文件都从pages文件夹下找 -->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <!-- 文件后缀为jsp -->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 配置静态资源不拦截 -->
    <mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 -->
    <mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 -->
    <mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->

    <!-- 配置spring开启注解mvc的支持 -->
    <mvc:annotation-driven />
</beans>
```

- 编写通用页面

一般项目都有通用的成功、失败页面。我们都放在webapp下的WEB-INF里面的pages下。我们就简单的写一些文字来代替。

1. 通用错误页面

```
<%--
  Created by IntelliJ IDEA.
  User: wally
  Date: 2020/6/23
  Time: 2:51 下午
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>错误页面</title>
</head>
<body>
<h3>友好的错误页面</h3>
${ errorMsg }
</body>
</html>
```

2. 通用成功页面

```
<%--
  Created by IntelliJ IDEA.
  User: wally
  Date: 2020/6/22
  Time: 3:22 下午
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>成功页面</title>
</head>
<body>
<h3>成功</h3>
</body>
</html>
```

- 编写Controller，输出HelloWord

1. 使用@Controller注解，标识当前是一个Controller类
2. 使用@RequestMapping注解，指定当前控制器管理的一级路径，这里为/user
3. 使用@RequestMapping注解，指定sayHello()接口方法的响应的请求路径为/hello，完整路径为：http://localhost:8081/springmvc_sample_war_exploded/user/hello
4. sayHello()接口方法，返回通用成功页面，直接返回success字符串，视图解析器会去pages下查找教success.jsp的文件

```
@Controller
@RequestMapping("/user")
public class HelloController {
    /**
     * 测试请求映射
     */
    @RequestMapping(path = "/hello")
    public String sayHello() {
        System.out.println("Hello Spring MVC");
        return "success";
    }
}
```

- index.html中，添加请求跳转

a标签，只能发起GET请求，href超链接中指定请求url，点击就会跳转到success.jsp。

```
<%--
  Created by IntelliJ IDEA.
  User: wally
  Date: 2020/6/22
  Time: 3:05 下午
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>首页</title>
</head>
<body>
<h3>入门案例</h3>
<a href="user/hello">入门案例</a>
</body>
</html>
```

#### 表单数据绑定到JavaBean

一般前端页面会有表单提交，表单数据很多时，一般我们会封装到一个JavaBean，而SpringMVC可以将数据映射到JavaBean中，省去我们从Request对象中做很多次get操作来获取。

这里我们前端页面，请求保存账户信息以及它的用户信息

- 用户实体

```
/**
 * 用户实体
 */
public class User {
    private String uname;
    private Integer age;

    //省略get、set
}
```

- 账户实体

账户实体中，包含一个User实体

```
/**
 * 账户实体
 */
public class Account implements Serializable {
    private String username;
    private String password;
    private Double money;
    /**
     * 嵌套自定义引用类型
     */
    private User user;

    //省略get、set
}
```

- Controller

Controller类中添加一个saveAccount()方法，Account实体作为方法形参即可。

```
/**
 * 测试数据绑定到JavaBean
 */
@RequestMapping("/saveAccount")
public String saveAccount(Account account) {
    System.out.println("---- saveAccount() 调用成功 ----");
    System.out.println(account);
    return "success";
}
```

- index.jsp

index.jsp添加一个表单，由于我们的User实体是嵌入到Account实体的，所以input标签的name属性，如user.name、user.age，级联写即可。

```
<form action="user/saveAccount" method="post"><br>
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    金额：<input type="text" name="money"><br>
    用户姓名：<input type="text" name="user.uname"><br>
    用户年龄：<input type="text" name="user.age"><br>
    <input type="submit" value="提交">
</form>
```

#### 自定义类型转换器

SpringMVC内置了一些转换器，例如时间转换，前端表单传递时间，使用字符串，也可以自动映射为实体的Date字段。
但例如有一些自定义格式，超过内置转换器的转换范围时，就需要我们自己自定义转换器了。

需求：例如我们使用2020-12-28这种格式，SpringMVC是不能帮我们转换的，这就需要我们自己提供转换器了。

- User实体增加date字段，表示出生日期

```
/**
 * 用户实体
 */
public class User {
    private String uname;
    private Integer age;
    private Date date;

    //省略get、set
}
```

- 自定义转换器

自定义转换器，需要实现Converter接口，有2个泛型，第一个泛型为转换前类型，第二个泛型为转换后的类型，这里我们是从String转为Date类型。
复写convert()转换方法，处理年-月-日的日期格式，例如：2020-12-28

```
/**
 * 字符串转日期类型
 */
public class StringToDateConverter implements Converter<String, Date> {
    public Date convert(String source) {
        try {
            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
            return format.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
            throw new IllegalArgumentException("数据类型转换异常");
        }
    }
}
```

- 在SpringMVC配置文件中，配置转换器

所有自定义转换器，都要在ConversionServiceFactoryBean，这个转换服务下，通过set标签排列。并将服务命名为conversionService。
最后，将conversionService服务，交给mvc:annotation-driven标签的conversion-service属性。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    //...省略其他配置

    <!-- 注册自定义类型转换器 -->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.itheima.util.StringToDateConverter"/>
            </set>
        </property>
    </bean>

    <!-- 配置spring开启注解mvc的支持，配置自定义类型转换器 -->
    <mvc:annotation-driven
            conversion-service="conversionService"/>
</beans>
```

- Controller，添加接口

```
/**
 * 测试自定义类型转换器
 */
@RequestMapping("/saveUser")
public String saveUser(User user) {
    System.out.println("---- saveUser() 调用成功 ----");
    System.out.println(user);
    return "success";
}
```

- 前端页面index.jsp

添加表单，在date的input中填入：2020-12-28，点击提交，在后台能收到转成成功的Date即可。

```
<h3>保存用户</h3>
<form action="user/saveUser" method="post">
    用户姓名：<input type="text" name="uname"><br>
    用户年龄：<input type="text" name="age"><br>
    用户生日：<input type="text" name="date"><br>
    <input type="submit" value="保存">
</form>
```

#### 支持原生Servlet的API

虽然SpringMVC给我们提供了良好的封装，但如果我们需要用到原始的HttpServletRequest、HttpServletResponse对象时，要怎么做呢，同样很简单，只要在接口方法形参上写上即可。

- Controller添加接口方法

```
/**
 * 测试获取Servlet原生API
 */
@RequestMapping("/testServlet")
public String testServlet(HttpServletRequest request, HttpServletResponse response) {
    System.out.println("---- testServlet() 调用成功 ----");
    System.out.println(request);
    HttpSession session = request.getSession();
    System.out.println(session);
    ServletContext servletContext = session.getServletContext();
    System.out.println(servletContext);
    System.out.println(response);
    return "success";
}
```

- index.jsp添加调用

```
<a href="user/testServlet">测试原生Servlet的API</a>
```

#### @RequestParam注解

上面讲到，表单请求，我们可以封装到一个JavaBean中，一般适用于参数很多和复杂的情况，如果参数很少，封装成JavaBean意义不大。那么我们就可以使用@RequestParam注解。

使用@RequestParam注解，注解内有一个value属性，标识表单字段变量，如果表单字段名和变量名一致，可以不写。
defaultValue属性，指定默认值，不传递参数或传null时生效。

```
/**
 * 测试@RequestParam注解
 */
@RequestMapping("/testParam")
public String testParam(@RequestParam("name") String username,
                        @RequestParam(required = false, defaultValue = "18") Integer age) {
    System.out.println("---- testParam() 调用成功 ----");
    System.out.println("username：" + username);
    System.out.println("age：" + age);
    return "success";
}
```

- index.jsp增加调用

```
<a href="user/testParam?name=哈哈">测试RequestParam注解</a>
```

#### @RequestBody注解

除了上面讲参数对应到形参，SpringMVC还提供了@RequestBody注解，可以将表单的参数按：参数名1=值1&参数名2=值2，这样的格式获取，类似GET请求时，带参数的方式。
除了格式以外，还可以作为json上传的方式封装参数到JavaBean。

- 第一种：按格式获取，Controller添加方法

```
/**
 * 测试@RequestBody注解，会将所有的表单数据按如下格式返回：username=hezihao&password=123
 */
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String body) {
    System.out.println("---- testRequestBody() 调用成功 ----");
    System.out.println(body);
    return "success";
}
```

- index.jsp增加调用

```
<h3>测试RequestBody注解</h3>
<form action="/user/testRequestBody" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="text" name="password"><br>
    <input type="submit" value="提交">
</form>
```

- 第二种：json上传方式，将参数封装到JavaBean

```
/**
 * 测试接收json数据请求
 */
@RequestMapping("/testAjax")
@ResponseBody
public User testAjax(@RequestBody User user) {
    System.out.println("---- testAjax() 调用成功 ----");
    System.out.println(user);
    //模拟结果
    user.setUname("haha");
    user.setAge(40);
    return user;
}
```

- index.jsp增加调用，这里我引入了jQuery，将jq放在webapp目录下的js目录下即可。

点击按钮时，会发起ajax请求，后端程序接收到请求后，返回数据，前端页面alter弹出数据。

```
<head>
    <title>首页</title>
    <script src="js/jquery.min.js"></script>
    <script>
        $(function () {
            $("#btn").click(function () {
                $.ajax({
                    url: "user/testAjax",
                    contentType: "application/json;charset=UTF-8",
                    data: '{"uname": "hehe", "age": 18}',
                    dataType: "json",
                    type: "post",
                    success: function (data) {
                        //data是服务器端返回的数据
                        alert(data)
                        alert(data.uname)
                    }
                })
            });
        });
    </script>
</head>
<body>
<button id="btn">测试ajax请求</button>
</body>
```
#### @PathVariable注解

GET请求除了表单提交外，还可以按Url的路径来请求，这时就需要用到@PathVariable注解。

- Controller增加接口

Url为user/testPathVariable/xxx，在Url后拼上id。

```
/**
 * 测试@testPathVariable注解
 */
@RequestMapping("/testPathVariable/{sid}")
public String testPathVariable(@PathVariable(name = "sid") int id) {
    System.out.println("---- testPathVariable() 调用成功 ----");
    System.out.println("id：" + id);
    return "success";
}
```

- index.jsp增加接口调用

```
<a href="user/testPathVariable/123">测试PathVariable注解</a><br>
```

#### @RequestHeader注解

当需要获取请求传递的Header请求头时，我们可以使用@RequestHeader注解来获取指定Key值的Header值。

- Controller增加方法

例如获取默认浏览器都会带的Accept请求头。

```
@RequestMapping("/testRequestHeader")
public String testRequestHeader(@RequestHeader(value = "Accept") String header) {
    System.out.println("---- testRequestHeader() 调用成功 ----");
    System.out.println("header => Accept：" + header);
    return "success";
}
```

- index.jsp增加接口调用

```
<a href="user/testRequestHeader">测试RequestHeader注解</a><br>
```

#### @CookieValue

需要获取请求发过来的Cookie时，可以使用@CookieValue注解。

- Controller添加方法

例如获取Session的ID，在前端浏览器上，这个id是保存在Cookie的，所以我们可以获取到。

```
@RequestMapping("/testCookieValue")
public String testCookieValue(@CookieValue(value = "JSESSIONID") String cookieValue) {
    System.out.println("---- testCookieValue() 调用成功 ----");
    System.out.println("JSESSIONID：" + cookieValue);
    return "success";
}
```

- index.jsp增加调用

```
<a href="user/testCookieValue">测试CookieValue注解</a><br>
```

#### SpringMVC的异常处理器

当我们的接口发生异常时，默认会返回异常信息的页面，这样返回给用户可不好，一般会返回一个漂亮又友好的页面。
SpringMVC给我们提供了全局异常处理器，我们可以捕获特定的异常，统一返回友好的错误页面。

- 自定义异常

SysException系统异常，附带的message为我们自己手动设置的，异常处理器可以拿到异常中的message，返回为用户。

```
/**
 * 系统异常
 */
public class SysException extends Exception {
    private String message;

    public SysException(String message) {
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

- 定义异常处理器

异常处理器需要实现HandlerExceptionResolver接口，复写resolveException()方法。
在这个方法里，我们需要判断异常对象的种类，如果是系统异常，则获取异常中的message，传递到到error错误页面。
如果是其他异常，统一返回请联系系统管理员的消息。

```
/**
 * 异常处理器
 */
public class SysExceptionResolver implements HandlerExceptionResolver {
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        SysException ex;
        if (e instanceof SysException) {
            ex = (SysException) e;
        } else {
            ex = new SysException("请联系系统管理员");
        }
        //跳转到友好的错误页面
        ModelAndView mv = new ModelAndView();
        mv.setViewName("error");
        mv.addObject("errorMsg", ex.getMessage());
        return mv;
    }
}
```

- 配置异常处理器到springmvc.xml

有了异常处理器类还不够，Spring还不知道我们的配置，所以需要在SpringMVC的配置文件springmvc.xml中进行配置。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    //...省略其他配置

    <!-- 配置异常处理器 -->
    <bean id="sysExceptionResolver" class="com.itheima.exception.SysExceptionResolver"/>
</beans>
```

- Controller添加接口方法

例如我们故意制造一个算数异常，try-catch后手动抛出。

```
/**
 * 测试异常处理器
 */
@RequestMapping("/testException")
public String testException() throws SysException {
    try {
        int result = 1 / 0;
    } catch (Exception e) {
        e.printStackTrace();
        throw new SysException(e.getMessage());
    }
    return "success";
}
```

- index.jsp增加接口调用

```
<a href="user/testException">测试异常处理器</a>
```

#### SpringMVC拦截器

SpringMVC还提供了拦截器，给我们在请求前和请求后（响应前）做一些处理。

- 自定义拦截器

拦截器需要实现HandlerInterceptor接口，提供以下方法进行复写。

1. preHandle()，预处理，控制器方法执行前回调。如果返回true，则放行，执行下一个拦截器，如果没有下一个拦截器，则执行控制器中的方法。
2. postHandle()，后处理方法，控制器方法执行后回调，jsp加载之前。
3. afterCompletion()，jsp页面加载之后回调。

执行顺序：preHandle() => postHandle() => afterCompletion()。

我们定义了2个拦截器，当触发回调方法时，返回true，会执行下一个拦截器，返回false则相当于拦截了，后面的拦截器则不回调。

```
/**
 * 自定义拦截器
 */
public class MyInterceptor1 implements HandlerInterceptor {
    /**
     * 预处理，控制器方法执行前回调
     * @return 返回true，则放行，执行下一个拦截器，如果没有下一个拦截器，则执行控制器中的方法
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1 => preHandle()执行了");
        return true;
    }

    /**
     * 后处理方法，控制器方法执行后回调，jsp加载之前
     */
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1 => postHandle()执行了");
    }

    /**
     * jsp页面加载之后回调
     */
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1 => afterCompletion()执行了");
    }
}

public class MyInterceptor2 implements HandlerInterceptor {
    /**
     * 预处理，控制器方法执行前回调
     * @return 返回true，则放行，执行下一个拦截器，如果没有下一个拦截器，则执行控制器中的方法
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor2 => preHandle()执行了");
        return true;
    }

    /**
     * 后处理方法，控制器方法执行后回调，jsp加载之前
     */
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor2 => postHandle()执行了");
    }

    /**
     * jsp页面加载之后回调
     */
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor2 => afterCompletion()执行了");
    }
}
```

- springmvc.xml中配置拦截器

和异常处理器异常，我们还需要告知Spring我们的拦截器配置。

1. mvc:interceptor标签，定义拦截器。
2. mvc:mapping标签，配置拦截的具体方法，可用*作为通配符。
3. mvc:exclude-mapping标签，表示不拦截某个方法，一般我们用上面的mapping标签比较多。
4. bean标签，class属性指定拦截器的类名。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    //...省略其他配置

    <!-- 配置拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <!-- 要拦截的具体的方法 -->
            <mvc:mapping path="/user/*"/>
            <!-- 不要拦截的那个的方法，不必须，只写上面的mvc:mapping，剩下的都不拦截 -->
            <!--            <mvc:exclude-mapping path=""/>-->
            <!-- 配置拦截器的类名 -->
            <bean class="com.itheima.interceptor.MyInterceptor1"/>
        </mvc:interceptor>
    </mvc:interceptors>

    <mvc:interceptors>
        <mvc:interceptor>
            <!-- 要拦截的具体的方法 -->
            <mvc:mapping path="/**"/>
            <!-- 配置拦截器的类名 -->
            <bean class="com.itheima.interceptor.MyInterceptor2"/>
        </mvc:interceptor>
    </mvc:interceptors>
</beans>
```

#### SpringMVC文件上传

- 增加依赖

SpringMVC的文件上传依赖commons-fileupload，commons-fileupload又依赖了commons-io。io包可以不指定，Maven会帮我们依赖传递的导入。

```
<!-- 文件上传 -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```

先来复习一下，传统方式，使用HttpServletRequest文件上传

- index.jsp增加文件上传

1.form表单请求，请求方式必须为POST
2. enctype，必须为multipart/form-data
3. input标签为file类型

```
<h3>传统方式文件</h3>
<form action="upload/testUpload1" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"/><br>
    <input type="submit" value="提交"><br>
</form>s
```

- 新建UploadController控制器

1. 先用HttpServletRequest对象，获取路径，我们将文件放到项目部署的根目录下的uploads文件夹内
2. 创建FileItemFactory文件工厂，解析文件项
3. 遍历文件项，判断到是文件项时，再通过io流写文件

```
@Controller
@RequestMapping("/upload")
public class UploadController {
    /**
     * 传统方式-文件上传
     */
    @RequestMapping(value = "/testUpload1", method = RequestMethod.POST)
    public String testUpload1(HttpServletRequest request) throws Exception {
        //获取文件上传根目录
        String path = request.getSession().getServletContext().getRealPath("/uploads/");
        //创建文件夹
        File pathDir = new File(path);
        if (!pathDir.exists()) {
            pathDir.mkdirs();
        }
        //创建文件工厂
        FileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        //解析请求中的文件项
        List<FileItem> items = upload.parseRequest(request);
        for (FileItem item : items) {
            if (item.isFormField()) {
                //只是普通表单项，不是文件，忽略
            } else {
                //是文件，写文件
                String fileName = item.getName();
                //生成唯一id
                String uuid = UUID.randomUUID().toString().replace("-", "");
                fileName = fileName + "_" + uuid;
                item.write(new File(pathDir, fileName));
                //删除临时文件
                item.delete();
            }
        }
        return "success";
    }
}
```

- SpringMVC方式

表单拷贝一份，请求testUpload2

```
<h3>SpringMVC方式文件</h3>
<form action="upload/testUpload2" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"/><br>
    <input type="submit" value="提交"><br>
</form>
```

- Controller新增testUpload2方法

SpringMVC的方式相比传统方式就简单很多，SpringMVC给我们提供了MultipartFile对象，使用这个对象的transferTo即可实现文件复制。

```
/**
 * SpringMVC方式-文件上传
 *
 * @param upload 多文件，变量名要和前端文件上传表单元素的name属性一致才行
 */
@RequestMapping(value = "/testUpload2", method = RequestMethod.POST)
public String testUpload2(HttpServletRequest request, @RequestParam("upload") MultipartFile upload) throws Exception {
    System.out.println("SpringMVC方式文件上传...");
    //获取文件上传路径，并创建目录
    String path = request.getSession().getServletContext().getRealPath("/uploads/");
    File pathDir = new File(path);
    if (!pathDir.exists()) {
        pathDir.mkdirs();
    }
    //重命名文件名
    String originalFilename = upload.getOriginalFilename();
    //生成唯一id
    String uuid = UUID.randomUUID().toString().replace("-", "");
    String fileName = originalFilename + "_" + uuid;
    //上传文件
    upload.transferTo(new File(pathDir, fileName));
    return "success";
}
```

- 跨服务器上传

一般真实开发中，文件服务器是单独的一台服务器，不和业务服务器一起。我们可以使用jersey，来实现跨服务器上传。

- 同样，先拷贝一份表单，请求testUpload3

```
<h3>跨服务器上传文件</h3>
<form action="upload/testUpload3" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"/><br>
    <input type="submit" value="提交"><br>
</form>
```

- Controller添加testUpload3方法

1. 创建Client对象，调用resource方法，指定上传路径，一般是文件服务器的接口地址，返回WebResource对象。
2. 使用WebResource对象的put()方法，获取MultipartFile对象的getBytes()获取Byte数据，再丢进去即可。

```
/**
 * 跨服务器方式-文件上传
 *
 * @param upload 多文件，变量名要和前端文件上传表单元素的name属性一致才行
 */
@RequestMapping(value = "/testUpload3", method = RequestMethod.POST)
public String testUpload3(@RequestParam("upload") MultipartFile upload) throws Exception {
    System.out.println("跨服务器方式文件上传...");
    //文件服务器上传文件请求路径
    String uploadPath = "http://localhost:9090/file_upload/uploads/";
    //重命名文件名
    String originalFilename = upload.getOriginalFilename();
    //生成唯一id
    String uuid = UUID.randomUUID().toString().replace("-", "");
    String fileName = originalFilename + "_" + uuid;
    //创建客户端对象
    Client client = Client.create();
    WebResource webResource = client.resource(uploadPath + fileName);
    //上传文件
    webResource.put(upload.getBytes());
    return "success";
}
```

#### 出现的问题总结

表单请求等一路顺利，但到了文件上传，启动一直报**java.lang.NoClassDefFoundError: org/apache/commons/fileupload/FileItemFactory**的错。

首先想到的是我们缺少jar包，就是上面的commons-fileupload包，但是我们Maven的pom文件已经添加了，而且这个类也是可以跳转过去的。
搜索资料了很久，最后发现是Tomcat部署的lib目录，WEB-INF/lib，没有添加上这2个jar包，添加上，再启动即可。

有同学可能不知道在哪里加，打开Project Structure，选择导出的war包，点开WEB-INF/lib，再点击上面的+号，选择fileupload和commons-io，确定就可以了。

![添加lib](https://user-gold-cdn.xitu.io/2020/7/8/1732e798551fe24c?w=1024&h=745&f=png&s=125177)

#### 项目地址

如果上面的描述不够清楚，我将代码都上传到了[Github](https://github.com/hezihaog/springmvc_sample)，有兴趣的同学可以clone。