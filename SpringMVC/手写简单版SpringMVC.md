# 手写简单版SpringMVC

本篇是看了慕课网上的教程[仅需2小时 手写MINI Spring MVC框架](https://www.imooc.com/learn/1203)，跟着手写了一次简单版的SpringMVC，项目由Gradle做项目依赖管理。

项目实现如下功能：

1. Bean扫描
2. 控制翻转
3. 依赖注入（循环依赖先不处理）
4. 请求分发和响应

### 大体流程

1. 入口类扫描Class，并启动Tomcat服务
2. Bean工厂扫描Class中的注解，创建实例，并且处理依赖注入
3. 扫描Controller类，创建控制器内的方法和Url的映射关系
4. 建立DispatchServlet，统管请求，请求来到时，遍历Controller类中的映射，找到后，反射调用控制器的方法，获取返回数据，写到浏览器

- 新建Gradle项目，选择普通java项目即可。再新建framework模块，依赖如下

集成Tomcat，Tomcat支持内嵌式在Java项目中。

```
plugins {
    id 'java'
}

group 'zbs.mooc.com'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    //集成Tomcat
    compile group: 'org.apache.tomcat.embed', name: 'tomcat-embed-core', version: '8.5.23'
}
```

### 嵌入Tomcat以及统一请求入口DispatcherServlet

- DispatcherServlet类

建立web包，再在里面创建一个servlet包，新建DispatcherServlet类，用于分发请求。

```
/**
 * 分发请求的Servlet
 */
public class DispatcherServlet implements Servlet {
    private ServletConfig config;

    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        System.out.println("DispatcherServlet => init()... 初始化");
    }

    @Override
    public ServletConfig getServletConfig() {
        return config;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        //...后续会在这里做文章，先留空
    }

    @Override
    public String getServletInfo() {
        return "";
    }

    @Override
    public void destroy() {
        System.out.println("DispatcherServlet => destroy()... 销毁");
    }
}
```

- TomcatServer类

web包下，新建TomcatServer类，作为Tomcat的启动类，同时注册DispatcherServlet，让DispatcherServlet处理所有请求。

1. 端口号是6699
2. DispatcherServlet注册的请求路径为/，表示统配所有请求

```
public class TomcatServer {
    /**
     * Tomcat实例
     */
    private Tomcat tomcat;
    /**
     * 启动参数，后续可以获取启动参数来进行配置
     */
    private String[] args;

    public TomcatServer(String[] args) {
        this.args = args;
    }

    /**
     * 开启Tomcat服务
     */
    public void startServer() throws LifecycleException {
        tomcat = new Tomcat();
        tomcat.setPort(6699);
        tomcat.start();
        //初始化容器
        Context context = new StandardContext();
        context.setPath("");
        context.addLifecycleListener(new Tomcat.FixContextListener());
        DispatcherServlet dispatcherServlet = new DispatcherServlet();
        //注册DispatcherServlet
        Tomcat.addServlet(context, "dispatcherServlet", dispatcherServlet)
                //设置支持异步
                .setAsyncSupported(true);
        //设置Servlet和URI的映射
        context.addServletMappingDecoded("/", "dispatcherServlet");
        //注册默认Host容器
        tomcat.getHost().addChild(context);

        //声明等待线程
        Thread awaitThread = new Thread(new Runnable() {
            @Override
            public void run() {
                //让Tomcat一直在等待
                tomcat.getServer().await();
            }
        }, "tomcat_await_thread");
        //设置为非守护进程
        awaitThread.setDaemon(false);
        awaitThread.start();
    }
}
```

- MiniApplication类

新建starter包，新建MiniApplication类，作为框架的入口。
需要调用方在main函数中调用MiniApplication的run()方法，传入启动入口类的Class和参数args。

```
/**
 * 框架入口类
 */
public class MiniApplication {
    public static void run(Class<?> cls, String[] args) {
        try {
            //创建Tomcat服务，启动服务
            TomcatServer tomcatServer = new TomcatServer(args);
            tomcatServer.startServer();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 获取当前包下的所有Class

- ClassScanner扫描类

建立core包，新建ClassScanner类，用于获取当前包下的所有Class。

```
/**
 * 类扫描器，将指定包下的所有Class收集起来
 */
public class ClassScanner {
    /**
     * 扫描指定包下的所有Class
     */
    public static List<Class<?>> scanClasses(String packageName) throws IOException, ClassNotFoundException {
        List<Class<?>> classList = new ArrayList<>();
        //将类的全路径名转换为文件路径
        String path = packageName.replace(".", "/");
        //获取类加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        Enumeration<URL> resources = classLoader.getResources(path);
        while (resources.hasMoreElements()) {
            URL resource = resources.nextElement();
            //如果是jar包，则获取jar包绝对路径
            if (resource.getProtocol().contains("jar")) {
                JarURLConnection jarURLConnection = (JarURLConnection) resource.openConnection();
                String jarFilePath = jarURLConnection.getJarFile().getName();
                //通过jar包的路径，获取jar包下所有的类
                classList.addAll(getClassesFromJar(jarFilePath, path));
            } else {
                //非jar包类型
            }
        }
        return classList;
    }

    /**
     * 通过jar包的路径，获取jar包下所有的类
     *
     * @param jarFilePath jar包的绝对路径
     * @param path        需要获取的类的相对路径，用来过滤
     */
    private static List<Class<?>> getClassesFromJar(String jarFilePath, String path) throws IOException, ClassNotFoundException {
        ArrayList<Class<?>> classes = new ArrayList<>();
        JarFile jarFile = new JarFile(jarFilePath);
        Enumeration<JarEntry> jarEntries = jarFile.entries();
        while (jarEntries.hasMoreElements()) {
            JarEntry jarEntry = jarEntries.nextElement();
            //com/mooc/zbs/test/Test.class
            String entryName = jarEntry.getName();
            if (entryName.startsWith(path) && entryName.endsWith(".class")) {
                //获取类的全类名
                String classFullName = entryName.replace("/", ".")
                        .substring(0, entryName.length() - 6);
                classes.add(Class.forName(classFullName));
            }
        }
        return classes;
    }
}
```

- MiniApplication启动类中，添加调用

```
/**
 * 框架入口类
 */
public class MiniApplication {
    public static void run(Class<?> cls, String[] args) {
        try {
            //创建Tomcat服务，启动服务
            TomcatServer tomcatServer = new TomcatServer(args);
            tomcatServer.startServer();
            //获取所有的Class
            List<Class<?>> classList = ClassScanner.scanClasses(cls.getPackage().getName());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 提供控制翻转和依赖注入

- Bean注解

建立beans包，建立Bean注解，用于控制翻转

```
/**
 * Bean注解，标识一个类被框架容器管理
 */
@Documented
//作用于类上
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Bean {
}
```

- Controller注解

和Bean注解类似，特指控制器的注解，给容器管理

```
/**
 * 控制器注解
 */
@Documented
//保留到运行时
@Retention(RetentionPolicy.RUNTIME)
//作用到类上
@Target(ElementType.TYPE)
public @interface Controller {
}
```

- Service注解

和Bean注解类似，特指业务层的注解，给容器管理，后续可能会添加特定功能

```
/**
 * Service注解，标识这个类是Service层的对象
 */
@Documented
//作用于类上
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Service {
}
```

- AutoWired注解

建立AutoWired注解，用于依赖注入

```
/**
 * 依赖注入注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
//作用于类属性上
@Target(ElementType.FIELD)
public @interface AutoWired {
}
```

- BeanFactory工厂

建立BeanFactory工厂类，该类主要是扫描类上的注解，遇到Bean、Service、Controller等注解，创建实例，并存入到容器中。

```
/**
 * Bean工厂
 */
public class BeanFactory {
    /**
     * Bean容器
     */
    private static final Map<Class<?>, Object> classToBean = new ConcurrentHashMap<>();

    /**
     * 获取一个Bean
     */
    public static Object getBean(Class<?> cls) {
        return classToBean.get(cls);
    }

    /**
     * 初始化Bean的方法
     *
     * @param classList 所有类列表
     */
    public static void initBean(List<Class<?>> classList) throws InstantiationException, IllegalAccessException {
        ArrayList<Class<?>> toCreate = new ArrayList<>(classList);
        while (toCreate.size() != 0) {
            int remainSize = toCreate.size();
            for (int i = 0; i < toCreate.size(); i++) {
                //创建完，就要移除掉
                if (finishCreate(toCreate.get(i))) {
                    toCreate.remove(i);
                }
            }
            //陷入循环依赖的死循环，抛出异常
            if (toCreate.size() == remainSize) {
                throw new RuntimeException("cycle dependency!");
            }
        }
    }

    /**
     * 初始化Bean
     */
    private static boolean finishCreate(Class<?> cls) throws IllegalAccessException, InstantiationException {
        boolean hasBeanAnno = cls.isAnnotationPresent(Bean.class);
        boolean hasControllerAnno = cls.isAnnotationPresent(Controller.class);
        boolean hasServiceSAnno = cls.isAnnotationPresent(Service.class);
        //忽略，没有使用Bean注解和不是Controller、Service的类
        if (!hasBeanAnno && !hasControllerAnno && !hasServiceSAnno) {
            return true;
        }
        //创建Bean，处理对象中的属性，查看是否需要依赖注入
        Object bean = cls.newInstance();
        for (Field field : cls.getDeclaredFields()) {
            if (field.isAnnotationPresent(AutoWired.class)) {
                //获取属性的类型
                Class<?> fieldType = field.getType();
                //从工厂里面获取，获取不到，先返回
                Object reliantBean = BeanFactory.getBean(fieldType);
                if (reliantBean == null) {
                    return false;
                }
                //从工厂获取到了，设置属性字段可接触
                field.setAccessible(true);
                //反射将对象设置到属性上
                field.set(bean, reliantBean);
            }
        }
        //缓存实例到工厂中
        classToBean.put(cls, bean);
        return true;
    }
}
```

- 在启动类中，添加调用

```
/**
 * 框架入口类
 */
public class MiniApplication {
    public static void run(Class<?> cls, String[] args) {
        try {
            //创建Tomcat服务，启动服务
            TomcatServer tomcatServer = new TomcatServer(args);
            tomcatServer.startServer();
            //获取所有的Class
            List<Class<?>> classList = ClassScanner.scanClasses(cls.getPackage().getName());
            //创建Bean工厂,扫描Class，创建被注解标注的类
            BeanFactory.initBean(classList);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 控制器接口方法和Url映射

经过上面的代码，我们先使用ClassScanner获取到所有的类的Class，再通过BeanFactory，实例化所有注解标识的实例。接下来就是在扫描出使用了Controller注解的控制器。
让控制器上的接口方法和Url产生映射关系。

- RequestMapping注解

该注解标识在Controller器的接口方法上，为每个接口方法绑定一个Url。

```
/**
 * 接口方法注解，需要指定Url
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequestMapping {
    /**
     * Url
     */
    String value();
}
```

- RequestParam注解

该注解标识在接口方法的形参上，标识每个形参变量对应的字段值

```
/**
 * 请求参数注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
//需要作用于方法参数上
@Target(ElementType.PARAMETER)
public @interface RequestParam {
    /**
     * 指定请求参数的key
     */
    String value();
}
```

- MappingHandler

新建MappingHandler类，用来保存Controller中接口方法和Url之间的关系，以及反射调用接口方法需要参数，例如形参列表、控制器Class、方法Method对象。

提供一个handle方法，提供给DispatchServlet调用，请求发过来时，判断是否是该接口方法响应。是则反射调用方法，并获取到返回值，响应到请求方。

```
/**
 * 保存每个URL和Controller的映射
 */
public class MappingHandler {
    /**
     * 请求路径Uri
     */
    private final String uri;
    /**
     * Controller中对应的方法
     */
    private final Method method;
    /**
     * Controller类对象
     */
    private final Class<?> controller;
    /**
     * 调用方法时传递的参数
     */
    private final String[] args;

    public MappingHandler(String uri, Method method, Class<?> controller, String[] args) {
        this.uri = uri;
        this.method = method;
        this.controller = controller;
        this.args = args;
    }

    /**
     * 处理方法
     *
     * @param req 请求对象
     * @param res 响应对象
     * @return 是否处理了
     */
    public boolean handle(ServletRequest req, ServletResponse res) throws IllegalAccessException, InstantiationException, InvocationTargetException, IOException {
        //获取请求路径
        String requestUri = ((HttpServletRequest) req).getRequestURI();
        //不是当前的Controller处理，直接返回
        if (!requestUri.equals(uri)) {
            return false;
        }
        //是当前Controller要处理的，准备方法参数，从Request对象中获取，获取到的值给反射调用
        Object[] parameters = new Object[args.length];
        for (int i = 0; i < args.length; i++) {
            parameters[i] = req.getParameter(args[i]);
        }
        //从缓存中取出Controller，启动时就已经创建Controller实例了
        Object ctl = BeanFactory.getBean(controller);
        //调用对应的接口方法，并获取响应结果
        Object response = method.invoke(ctl, parameters);
        //将响应结果写到外面
        res.getWriter().println(response.toString());
        return true;
    }
}
```

- HandlerManager

Handler管理器，每个Controller其实就是一个Handler，该管理器负责启动时扫描所有Controller类，组成映射关系，并存储起来，提供给DispatchServlet获取和使用。

```
/**
 * Handler管理类
 */
public class HandlerManager {
    /**
     * Controller类中所有类方法和uri映射关系
     */
    private static final List<MappingHandler> mappingHandlerList = new ArrayList<>();

    /**
     * 找到所有Controller类
     *
     * @param classList 类的Class集合
     */
    public static void resolveMappingHandler(List<Class<?>> classList) {
        for (Class<?> cls : classList) {
            //判断是否使用了Controller注解
            if (cls.isAnnotationPresent(Controller.class)) {
                parseHandlerFromController(cls);
            }
        }
    }

    /**
     * 解析Controller上的注解
     */
    private static void parseHandlerFromController(Class<?> cls) {
        //获取类上的所有方法
        Method[] methods = cls.getDeclaredMethods();
        for (Method method : methods) {
            //判断方法是否使用了RequestMapping注解，如果没有标识，不处理
            if (!method.isAnnotationPresent(RequestMapping.class)) {
                continue;
            }
            //获取RequestMapping注解上标识的uri值
            String uri = method.getDeclaredAnnotation(RequestMapping.class).value();
            //获取形参上的RequestParam注解，拿取注解上定义的值
            ArrayList<String> paramNameList = new ArrayList<>();
            for (Parameter parameter : method.getParameters()) {
                if (parameter.isAnnotationPresent(RequestParam.class)) {
                    String value = parameter.getDeclaredAnnotation(RequestParam.class).value();
                    paramNameList.add(value);
                }
            }
            //参数集合转换为数组
            String[] params = paramNameList.toArray(new String[paramNameList.size()]);
            //参数收集完毕，构建一个MappingHandler
            MappingHandler mappingHandler = new MappingHandler(uri, method, cls, params);
            //保存到列表里
            mappingHandlerList.add(mappingHandler);
        }
    }

    public static List<MappingHandler> getMappingHandlerList() {
        return mappingHandlerList;
    }
}
```

- 启动类中，添加调用

```
/**
 * 框架入口类
 */
public class MiniApplication {
    public static void run(Class<?> cls, String[] args) {
        try {
            //创建Tomcat服务，启动服务
            TomcatServer tomcatServer = new TomcatServer(args);
            tomcatServer.startServer();
            //获取所有的Class
            List<Class<?>> classList = ClassScanner.scanClasses(cls.getPackage().getName());
            //创建Bean工厂,扫描Class，创建被注解标注的类
            BeanFactory.initBean(classList);
            //扫描所有的类，找到所有Controller，建立Controller中每个方法和Url的映射关系
            HandlerManager.resolveMappingHandler(classList);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 在DispatchServlet中补充遍历MappingHandler

```
/**
 * 分发请求的Servlet
 */
public class DispatcherServlet implements Servlet {
    private ServletConfig config;

    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        System.out.println("DispatcherServlet => init()... 初始化");
    }

    @Override
    public ServletConfig getServletConfig() {
        return config;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        //获取所有Controller和内部定义的接口方法列表
        List<MappingHandler> mappingHandlerList = HandlerManager.getMappingHandlerList();
        //找到当前请求Url对应的Controller接口处理方法
        for (MappingHandler mappingHandler : mappingHandlerList) {
            try {
                if (mappingHandler.handle(req, res)) {
                    return;
                }
            } catch (IllegalAccessException | InstantiationException | InvocationTargetException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public String getServletInfo() {
        return "";
    }

    @Override
    public void destroy() {
        System.out.println("DispatcherServlet => destroy()... 销毁");
    }
}
```

#### 测试模块

- 建立一个test模块，依赖framework模块，依赖如下

注意要添加jar的配置，指定启动类，才能打包和运行成功

```
plugins {
    id 'java'
}

group 'zbs.mooc.com'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    //依赖自定义的SpringMVC
    compile(project(':framework'))
}

//标识启动类
jar {
    manifest {
        attributes "Main-Class": "com.mooc.zbs.Application"
    }
    from {
        configurations.compile.collect {
            it.isDirectory() ? it : zipTree(it)
        }
    }
}
```

- 新建Application入口类，并初始化框架

```
/**
 * 测试入口类
 */
public class Application {
    public static void main(String[] args) {
        MiniApplication.run(Application.class, args);
    }
}
```

- 建立工具类，新增SalaryHelper工具类，提供一个按工龄计算工资的方法

```
/**
 * 工资计算类
 */
@Bean
public class SalaryHelper {
    /**
     * 计算工资
     *
     * @param experience 工龄
     */
    public Integer calSalary(Integer experience) {
        return experience * 5000;
    }
}
```

- 建立Service层，提供SalaryService，业务层对象

```
@Service
public class SalaryService {
    @AutoWired
    private SalaryHelper salaryHelper;

    /**
     * 计算工资
     *
     * @param experience 工龄
     */
    public Integer calSalary(Integer experience) {
        return salaryHelper.calSalary(experience);
    }
}
```

- 建立Controller，提供getSalary()接口方法，提供计算工资的功能

```
/**
 * 工资的控制器
 */
@Controller
public class SalaryController {
    /**
     * 依赖注入
     */
    @AutoWired
    private SalaryService salaryService;

    /**
     * 查询工资
     *
     * @param name       员工名称
     * @param experience 工龄
     */
    @RequestMapping("/getSalary")
    public Integer getSalary(@RequestParam("name") String name, @RequestParam("experience") String experience) {
        System.out.println("salaryService => " + salaryService);
        System.out.println("获取到的参数 => name=" + name + ",experience=" + experience);
        return salaryService.calSalary(Integer.parseInt(experience));
    }
}
```

### 启动

- 选择Idea右侧的Gradle命令模板，点击jar命令，命令跑完后，在test模块下的build目录下，有一个libs目录，里面就存放着一个打包好的jar包。
- java -jar test/build/libs/test-1.0-SNAPSHOT.jar，即可启动。
- 浏览器访问：http://localhost:6699/getSalary?experience=5，即可调用到刚才的Controller，输出结果25000。

### 项目源码

项目源码我放到了[Github](https://github.com/hezihaog/mini-spring)，有兴趣的同学可以clone下来看下。