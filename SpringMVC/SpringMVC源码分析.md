# SpringMVC源码分析

SpringMVC在Java后端中，是一个很重要也很常用的框架，本篇来分析一下SpringMVC的源码。版本为5.2.0。

- web.xml配置

虽然使用SpringMVC不需要我们写Servlet，但SpringMVC是封装了Servlet，提供 `DispatcherServlet` 来帮我们处理的。
所以需要在 `web.xml` 配置 `DispatcherServlet`。

可以看出 `DispatcherServlet`，映射的url是 `/`，所以所有的请求都会被它拦截，再处理给我们。我们进去看一下。

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

    //省略其他配置...
</web-app>
```

## 初始化

我们知道，Servlet初始化时，Servlet的 `init()`方法会被调用。我们进入 `DispatcherServlet`中，发现并没有该方法，那么肯定在它集成的父类上。
`DispatcherServlet` 继承于 `FrameworkServlet`，结果还是没找到，继续找它的父类 `HttpServletBean`。

### HttpServletBean

终于找到了，`HttpServletBean` 继承于 `HttpServlet`，我们来看下这个 `init()` 方法。

```
@Override
public final void init() throws ServletException {
    //获取配置web.xml中的参数
    PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
    if (!pvs.isEmpty()) {
        try {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        }
        catch (BeansException ex) {
            if (logger.isErrorEnabled()) {
                logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
            }
            throw ex;
        }
    }
    //重点：一个空方法，模板模式，子类FrameworkServlet，重写了它
    initServletBean();
}
```

### FrameworkServlet

`initServletBean()`方法，就是一个初始化。
方法内主要是调用了 `initWebApplicationContext()` 初始化WebApplicationContext，
以及调用 `initFrameworkServlet()` ，这个是一个空方法，可以提供给以后的子类复写，做一些初始化的事情，暂时没有被复写。

```
@Override
protected final void initServletBean() throws ServletException {
    getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
    if (logger.isInfoEnabled()) {
        logger.info("Initializing Servlet '" + getServletName() + "'");
    }
    long startTime = System.currentTimeMillis();

    try {
        //重点：初始化WebApplicationContext
        this.webApplicationContext = initWebApplicationContext();
        //一个空方法，可以提供给以后的子类复写，做一些初始化的事情，暂时没有被复写
        initFrameworkServlet();
    }
    catch (ServletException | RuntimeException ex) {
        logger.error("Context initialization failed", ex);
        throw ex;
    }

    //省略无关代码...
}
```

下面分析 `initWebApplicationContext()`方法。

- initWebApplicationContext()方法

该方法是初始化 `WebApplicationContext`的，而它集成于 `ApplicationContext`，所以它也是一个IoC容器。

所以 `FrameworkServlet`类的职责是将 `Spring` 和 `Servler` 进行一个关联。

这个方法，除了初始化WebApplicationContext外，还调用了一个 `onRefresh()`方法，又是模板模式，空方法，让子类复写进行逻辑处理，例如子类DispatcherServlet重写了它

```
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
            WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;
    //有参数构造方法，传入webApplicationContext对象，就会进入该判断
    if (this.webApplicationContext != null) {
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            //还没初始化过，容器的refresh()还没有调用
            if (!cwac.isActive()) {
                //设置父容器
                if (cwac.getParent() == null) {
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        //获取ServletContext，之前通过setAttribute设置到了ServletContext中，现在通过getAttribute获取到
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        //创建WebApplicationContext，设置环境environment、父容器，本地资源文件
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        synchronized (this.onRefreshMonitor) {
            //刷新，也是模板模式，空方法，让子类重写进行逻辑处理，而子类DispatcherServlet重写了它
            onRefresh(wac);
        }
    }

    //用setAttribute()，将容器设置到ServletContext中
    if (this.publishContext) {
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }
    return wac;
}

//WebApplicationContext
public interface WebApplicationContext extends ApplicationContext {
    //...
}
```

接来下，我们看子类 `DispatcherServlet` 复写的 `onRefresh()`方法。

### DispatcherServlet

`FrameworkServlet`类的职责是将 `Spring` 和 `Servler` 进行一个关联。而对于 `DispatcherServlet` 来说，它初始化方法是 `onRefresh()`。

`onRefresh()` 方法，调用 `initStrategies()` 方法，进行各种组件的初始化工作。

我们重点看 `initHandlerMappings()` 后面的流程！

```
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    //解析请求
    initMultipartResolver(context);
    //国际化
    initLocaleResolver(context);
    //主题
    initThemeResolver(context);
    //处理Controller的方法和url映射关系
    initHandlerMappings(context);
    //初始化适配器，多样写法Controller的适配处理，实现最后返回都是ModelAndView
    initHandlerAdapters(context);
    //初始化异常处理器
    initHandlerExceptionResolvers(context);
    //初始化视图转发
    initRequestToViewNameTranslator(context);
    //初始化视图解析器，将ModelAndView保存的视图信息，转换为一个视图，输出数据
    initViewResolvers(context);
    //初始化映射处理器
    initFlashMapManager(context);
}
```

### 总结3个Servlet类的作用和职责

- HttpServletBean

主要做一些初始化工作，解析 `web.xml` 中配置的参数到Servlet上，比如`init-param`中配置的参数。提供 `initServletBean()` 模板方法，给子类 `FrameworkServlet`实现。

- FrameworkServlet

将Servlet和SpringIoC容器关联。主要是初始化其中的 `WebApplicationContext`，它代表SpringMVC的上下文，它有一个父上下文，就是 `web.xml` 配置文件中配置的 `ContextLoaderListener` 监听器，监听器进行初始化容器上下文。
提供了 `onRefresh()` 模板方法，给子类 `DispatcherServlet` 实现，作为初始化入口方法。

- DispatcherServlet

最后的子类，作为前端控制器，初始化各种组件，比如请求映射、视图解析、异常处理、请求处理等。

## 组件分析

下面开始分发流程分析。

### HandlerMapping 分析

初始化Controller的Url映射关系。

主要是 `Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);`，这句代码从容器中，获取所有 `HandlerMapping`。

```
//处理器映射集合
@Nullable
private List<HandlerMapping> handlerMappings;
//一个开关，标识是否获取所有的处理器映射，如果为false，则搜寻指定名为的 handlerMapping 的 Bean实例
private boolean detectAllHandlerMappings = true;
//指定的Bean的名称
public static final String HANDLER_MAPPING_BEAN_NAME = "handlerMapping";

private void initHandlerMappings(ApplicationContext context) {
    //清空集合
    this.handlerMappings = null;
    
    //一个开关，默认为true，设置为false，才走else的逻辑
    if (this.detectAllHandlerMappings) {
        //重点：在容器中找到所有HandlerMapping
        Map<String, HandlerMapping> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        //找到了，进行排序，保证顺序
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<>(matchingBeans.values());
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    }
    else {
        //指定搜寻指定名为 handlerMapping 的 HandlerMapping 实例
        try {
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerMapping later.
        }
    }
    
    //也找不到映射关系，设置一个默认的
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
        if (logger.isTraceEnabled()) {
            logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
                    "': using default strategies from DispatcherServlet.properties");
        }
    }

    //配置文件名
    private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";

    //从配置文件中获取配置的组件，其他组件找不到时，也是调用这个方法进行默认配置
    protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
        //...
    }
}
```

如果找不到任何一个映射关系，会通过 `getDefaultStrategies` 方法，从配置文件中获取默认配置。其他组件找不到时，也是调用这个方法进行默认配置。

配置文件名：DispatcherServlet.properties。会加入2个默认的映射关系类 `BeanNameUrlHandlerMapping` 、 `RequestMappingHandlerMapping`。

```
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping
```

### HandlerAdapter 初始化

`HandlerAdapter` 的初始化逻辑和上面的 `HandlerMapping` 基本一样。从容器中搜寻所有 `HandlerAdapter` 的实例。
如果找不到，则从配置文件中获取`默认` 的 `HandlerAdapter`。

```
//适配器集合
@Nullable
private List<HandlerAdapter> handlerAdapters;
//和上面HandlerMapping一样，一个开关，是否搜寻容器中所有的HandlerAdapter，如果为false，则搜寻指定名为 handlerAdapter 的Bean
private boolean detectAllHandlerAdapters = true;
//指定的HandlerAdapter实例
public static final String HANDLER_ADAPTER_BEAN_NAME = "handlerAdapter";

private void initHandlerAdapters(ApplicationContext context) {
    //清空集合
    this.handlerAdapters = null;

    //也是一个开关，默认true，搜寻容器中所有的HandlerAdapter
    if (this.detectAllHandlerAdapters) {
        Map<String, HandlerAdapter> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerAdapter.class, true, false);
        //找到了，进行排序，保证HandlerAdapter是有序的
        if (!matchingBeans.isEmpty()) {
            this.handlerAdapters = new ArrayList<>(matchingBeans.values());
            AnnotationAwareOrderComparator.sort(this.handlerAdapters);
        }
    }
    else {
        //指定找名为 handlerAdapter 的HandlerAdapter
        try {
            HandlerAdapter ha = context.getBean(HANDLER_ADAPTER_BEAN_NAME, HandlerAdapter.class);
            this.handlerAdapters = Collections.singletonList(ha);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerAdapter later.
        }
    }

    //没有找一个HandlerAdapter，从配置文件中获取默认的HandlerAdapter
    if (this.handlerAdapters == null) {
        this.handlerAdapters = getDefaultStrategies(context, HandlerAdapter.class);
        if (logger.isTraceEnabled()) {
            logger.trace("No HandlerAdapters declared for servlet '" + getServletName() +
                    "': using default strategies from DispatcherServlet.properties");
        }
    }
}
```

### 异常处理器 初始化

和上面的一样，从容器中搜寻所有的异常处理器的实例，也有一个开关去搜索指定名称的异常处理器。

```
@Nullable
private List<HandlerExceptionResolver> handlerExceptionResolvers;
//开关，是否梭巡所有的异常处理器，设置为false，就会找下面名为 handlerExceptionResolver 的Bean实例
private boolean detectAllHandlerExceptionResolvers = true;
//指定名为 handlerExceptionResolver 的实例
public static final String HANDLER_EXCEPTION_RESOLVER_BEAN_NAME = "handlerExceptionResolver";

private void initHandlerExceptionResolvers(ApplicationContext context) {
    //清空集合
    this.handlerExceptionResolvers = null;

    //开关，默认true
    if (this.detectAllHandlerExceptionResolvers) {
        //搜寻所有的异常处理器
        Map<String, HandlerExceptionResolver> matchingBeans = BeanFactoryUtils
                .beansOfTypeIncludingAncestors(context, HandlerExceptionResolver.class, true, false);
        //搜寻到了
        if (!matchingBeans.isEmpty()) {
            this.handlerExceptionResolvers = new ArrayList<>(matchingBeans.values());
            //排序
            AnnotationAwareOrderComparator.sort(this.handlerExceptionResolvers);
        }
    }
    else {
        try {
            HandlerExceptionResolver her =
                    context.getBean(HANDLER_EXCEPTION_RESOLVER_BEAN_NAME, HandlerExceptionResolver.class);
            this.handlerExceptionResolvers = Collections.singletonList(her);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, no HandlerExceptionResolver is fine too.
        }
    }

    //一个异常处理器都没有，从配置文件中获取默认的
    if (this.handlerExceptionResolvers == null) {
        this.handlerExceptionResolvers = getDefaultStrategies(context, HandlerExceptionResolver.class);
        if (logger.isTraceEnabled()) {
            logger.trace("No HandlerExceptionResolvers declared in servlet '" + getServletName() +
                    "': using default strategies from DispatcherServlet.properties");
        }
    }
}
```

### 初始化 ViewResolver 视图解析器

视图解析器和上面的解析器逻辑一样，先有开关决定是搜寻容器中所有的，还是搜寻指定名称的。

```
//视图解析器集合
@Nullable
private List<ViewResolver> viewResolvers;
//开关
private boolean detectAllViewResolvers = true;
//指定名称
public static final String VIEW_RESOLVER_BEAN_NAME = "viewResolver";

private void initViewResolvers(ApplicationContext context) {
    //清空集合
    this.viewResolvers = null;

    if (this.detectAllViewResolvers) {
        //搜寻所有视图解析器
        Map<String, ViewResolver> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, ViewResolver.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.viewResolvers = new ArrayList<>(matchingBeans.values());
            //排序
            AnnotationAwareOrderComparator.sort(this.viewResolvers);
        }
    }
    else {
        try {
            //搜寻指定名为 viewResolver 的视图解析器Bean
            ViewResolver vr = context.getBean(VIEW_RESOLVER_BEAN_NAME, ViewResolver.class);
            this.viewResolvers = Collections.singletonList(vr);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default ViewResolver later.
        }
    }

    //没有找到任何一个视图解析器，从配置文件中读取
    if (this.viewResolvers == null) {
        this.viewResolvers = getDefaultStrategies(context, ViewResolver.class);
        if (logger.isTraceEnabled()) {
            logger.trace("No ViewResolvers declared for servlet '" + getServletName() +
                    "': using default strategies from DispatcherServlet.properties");
        }
    }
}
```

## 请求流程分析

当请求进入时，我们都知道会调用Servlet的 `service()` 方法，我们试着去 `DispatchServlet` 中搜索，发现没有。我们去到父类 `FrameworkServlet` 找到了。

因为父类是 `HttpServlet`，所以每种请求方法，都会在 `service` 中区分成对应的回调方法。

1. GET请求：doGet
2. POST请求：doPost
3. PUT请求：doPut
4. DELETE请求：doDelete
5. OPTIONS请求：doOptions
6. Trace请求：doTrace

可以看到，每种请求都被复写了，调用 `processRequest()`方法进行处理请求。

```
@Override
protected void service(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    //获取请求方式
    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    //如果是PATCH或获取不到，则走自己的processRequest()方法
    if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
        processRequest(request, response);
    }
    //其他情况，走父类HttpServlet的逻辑，调用其他区分出来的请求方法，例如 doGet、doPost
    else {
        super.service(request, response);
    }
}

//处理GET请求
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}

//处理POST请求
@Override
protected final void doPost(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}

//出路PUT请求
@Override
protected final void doPut(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}

//处理DELETE请求
@Override
protected final void doDelete(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}

//处理OPTION请求
@Override
protected void doOptions(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    if (this.dispatchOptionsRequest || CorsUtils.isPreFlightRequest(request)) {
        processRequest(request, response);
        if (response.containsHeader("Allow")) {
            // Proper OPTIONS response coming from a handler - we're done.
            return;
        }
    }
    // Use response wrapper in order to always add PATCH to the allowed methods
    super.doOptions(request, new HttpServletResponseWrapper(response) {
        @Override
        public void setHeader(String name, String value) {
            if ("Allow".equals(name)) {
                value = (StringUtils.hasLength(value) ? value + ", " : "") + HttpMethod.PATCH.name();
            }
            super.setHeader(name, value);
        }
    });
}

//处理Trace请求
@Override
protected void doTrace(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    if (this.dispatchTraceRequest) {
        processRequest(request, response);
        if ("message/http".equals(response.getContentType())) {
            // Proper TRACE response coming from a handler - we're done.
            return;
        }
    }
    super.doTrace(request, response);
}
```

- processRequest 处理请求

一大片的处理和设置，不是我们的重点，主要是 `doService()` 这个方法，它是一个抽象方法，强制让子类进行复写。
所以最终子类 `DispatcherServlet` 肯定会复写 `doService()` 方法。

```
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
        //重点，doService()是一个抽象方法，强制让子类进行复写
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }
    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

### DispatcherServlet

- doService()

`doService()` 方法中，主要的组件分发处理逻辑在 `doDispatch()` 方法中。

```
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    //打印请求
    logRequest(request);

    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    //设置组件到请求域中，给后续的其他组件可以获取到
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    try {
        //重点：主要的组件分发处理逻辑在 doDispatch() 方法
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

- doDispatch() 分发请求给各个组件处理

该方法非常主要，分发逻辑在这里呈现了。重点都在注释标注了。

分发步骤：

1. `getHandler()`，获取本次请求的处理器执行链，包括Controller和拦截器，它们组合成一个执行链 `HandlerExecutionChain`。
2. `getHandlerAdapter()`，获取处理器的适配器，因为有很多种处理器的实现方式，例如直接是Servlet作为处理器、实现Controller接口、使用Controller注解等，每个接口方法的返回值各式各样，所以这里使用了适配器模式，让适配器对处理器的返回值统一输出为ModelAndView。
3. `mappedHandler.applyPreHandle`，责任链模式调用处理器链中的拦截器的 `preHandle()` 方法，代表请求准备进行处理。拦截器可拦截处理。如果拦截器拦截了，则继续往下走。
4. `ha.handle()`，调用适配器的处理方法，传入处理器，调用处理器接口方法，并适配处理器的结果为ModelAndView。
5. `mappedHandler.applyPostHandle`，遍历调用处理器执行链中的拦截器的 `postHandle()` 后置处理方法，代表请求以被处理，但视图还未渲染
6. `processDispatchResult()`，处理视图和结果，调用视图处理器，将真正的视图创建，并对视图数据进行渲染。以及渲染完毕，调用拦截器的 `afterCompletion()`方法，代表视图渲染完毕。
7. `mappedHandler.applyAfterConcurrentHandlingStarted`，调用处理器执行链中的拦截器，无论当前请求处理成功，还是失败，都处理。

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    //本次请求的处理器以及拦截器，它们组合成一个执行链
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            //检查是否是文件上传请求。是的话，做一些处理
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            //重点：找到本次请求的处理器以及拦截器
            mappedHandler = getHandler(processedRequest);
            //找不到处理器处理，响应404
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            //重点：找到本次请求中，处理器的适配器
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            //重点：处理前，责任链模式 回调拦截器的 preHandle() 方法，如果拦截了，则不继续往下走了
            //返回true代表放心，false为拦截
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            //重点：调用适配器的处理方法，传入处理器，让适配器将处理器的结果转换成统一的ModelAndView
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            //如果找不到默认的视图，则设置默认的视图
            applyDefaultViewName(processedRequest, mv);
            //重点：处理完成，调用拦截器的 postHandle() 后置处理方法
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        //重点：分发结果，让视图解析器解析视图，渲染视图和数据
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            //重点：视图渲染完成，调用拦截器的 afterConcurrentHandlingStarted() 方法
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

下面就对上面的每个步骤，进行分析。

### getHandler() 搜寻本次请求的处理器对象

责任链模式，遍历handlerMappings集合，找到处理器和拦截器，会调用到 `AbstractHandlerMapping`的 `getHandler()`方法。
最后将处理器和拦截器都封装到 `HandlerExecutionChain` 这个处理器执行链对象中。

`getHandlerInternal()`，子类实现，主要实现有2个，第一个是AbstractUrlHandlerMapping，一般用它的子类SimpleUrlHandlerMapping，这种方式需要在xml配置文件中配置，已经很少用了。第二个是AbstractHandlerMethodMapping，就是处理我们@Controller和@RequestMapping的。

一般会使用 `AbstractHandlerMethodMapping` 的子类 `RequestMappingHandlerMapping`，其中查询注解的方法为 `isHandler()`，在这里分析的话，篇幅太大了，后续文章，再分析。

```
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}

//处理器映射接口，子类 AbstractHandlerMapping 实现了该方法，它是一个抽象类，所有的HandlerMapping实现类，都继承于它
//而AbstractHandlerMapping只管公共流程和处理，提取了抽象方法给子类实现，也就是模板模式
public interface HandlerMapping {
	@Nullable
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}

public abstract class AbstractHandlerMapping extends WebApplicationObjectSupport
		implements HandlerMapping, Ordered, BeanNameAware {
	@Override
	@Nullable
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        //模板方法，获取处理器，具体子类进行实现
		Object handler = getHandlerInternal(request);
        //如果没有获取到，则使用默认的处理器
		if (handler == null) {
			handler = getDefaultHandler();
		}
        //默认的也没有，那就返回null了
		if (handler == null) {
			return null;
		}
		//如果处理器是字符串类型，则在IoC容器中搜寻实例
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = obtainApplicationContext().getBean(handlerName);
		}

        //构成处理器执行链，主要是添加拦截器
		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

		if (logger.isTraceEnabled()) {
			logger.trace("Mapped to " + handler);
		}
		else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
			logger.debug("Mapped to " + executionChain.getHandler());
		}

		if (hasCorsConfigurationSource(handler)) {
			CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
			CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
			config = (config != null ? config.combine(handlerConfig) : handlerConfig);
			executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
		}
		return executionChain;
	}
    
    //组成处理器执行链，添加拦截器
    protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
		HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
				(HandlerExecutionChain) handler : new HandlerExecutionChain(handler));

		String lookupPath = this.urlPathHelper.getLookupPathForRequest(request, LOOKUP_PATH);
		for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
			if (interceptor instanceof MappedInterceptor) {
				MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
                //和请求的url做匹配，匹配才加入
				if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
					chain.addInterceptor(mappedInterceptor.getInterceptor());
				}
			}
			else {
				chain.addInterceptor(interceptor);
			}
		}
		return chain;
	}
}
```

接下来分析适配组件，`getHandlerAdapter()`

### getHandlerAdapter() 获取处理器对应的适配器

责任链模式，遍历调用适配器集合，调用supports()方法，询问每个适配器，是否支持当前的处理器。
如果返回true，则代表找到了，停止遍历，返回适配器。

```
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        for (HandlerAdapter adapter : this.handlerAdapters) {
            //责任链模式，遍历调用适配器集合，调用supports()方法，询问每个适配器，是否支持当前的处理器
            //如果返回true，则代表找到了，停止遍历，返回适配器
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }
    throw new ServletException("No adapter for handler [" + handler +
            "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

我们来看看适配器接口，以及它的子类

1. `HttpRequestHandlerAdapter`，适配 `HttpRequestHandler` 作为handler的适配器。
2. `SimpleServletHandlerAdapter`，适配 `Servlet` 作为handler的适配器
3. `SimpleControllerHandlerAdapter`，适配 `Controller` 接口 作为handler的适配器

```
public interface HandlerAdapter {
    //判断传入的处理器，是否支持适配
	boolean supports(Object handler);

    //上面的supports()方法返回true，才会调用该方法，进行适配，统一返回ModelAndView对象
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

    //作用和Servlet的getLastModified()一样，如果适配的处理器不支持，返回-1即可
	long getLastModified(HttpServletRequest request, Object handler);
}

//适配HttpRequestHandler的适配器
public class HttpRequestHandlerAdapter implements HandlerAdapter {
	@Override
	public boolean supports(Object handler) {
		return (handler instanceof HttpRequestHandler);
	}

	@Override
	@Nullable
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		((HttpRequestHandler) handler).handleRequest(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}
}

//适配Servlet的适配器
public class SimpleServletHandlerAdapter implements HandlerAdapter {
	@Override
	public boolean supports(Object handler) {
		return (handler instanceof Servlet);
	}

	@Override
	@Nullable
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		((Servlet) handler).service(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		return -1;
	}
}

//适配Controller接口的适配器
public class SimpleControllerHandlerAdapter implements HandlerAdapter {
	@Override
	public boolean supports(Object handler) {
		return (handler instanceof Controller);
	}

	@Override
	@Nullable
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return ((Controller) handler).handleRequest(request, response);
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}
}
```

根据流程，获取到对应的适配器后，就可以通知拦截器了

### 拦截器前置通知

遍历拦截器链，调用它的 `preHandle()` 方法，通知拦截器进行请求处理前的拦截和附加处理。
如果有一个拦截器返回false，代表拦截，则处理流程被中断，就是拦截了。

```
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = 0; i < interceptors.length; i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if (!interceptor.preHandle(request, response, this.handler)) {
                triggerAfterCompletion(request, response, null);
                return false;
            }
            this.interceptorIndex = i;
        }
    }
    return true;
}
```

随后，就调用适配器的 `handle()` 方法，进行适配，返回ModelAndView。
处理完后，也代表请求进过Controller处理完毕，接着进行拦截器通知。

### 拦截器后置通知

和前置通知不同，后置通知没有拦截功能，只能是增强。逻辑还是遍历拦截器链，调用拦截器的 `postHandle()` 方法。

```
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
        throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = interceptors.length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            interceptor.postHandle(request, response, this.handler, mv);
        }
    }
}
```

视图、数据都获取到了，就可以进行视图生成以及数据渲染了。

### 结果处理

因为 `doDispatch()` 的处理流程，SpringMVC都帮我们try-catch了，所以能捕获到异常，并传入该方法。

接着首先判断处理过程中，是否产生了异常，有则用异常处理器处理。
没有异常，则继续往下走，判断是否需要渲染，需要渲染，则进行渲染，最后回调拦截器进行通知。

```
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
        @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
        @Nullable Exception exception) throws Exception {
    //是否显示错误页面
    boolean errorView = false;
    //处理异常
    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            //处理异常
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }
    //判断处理器是否需要返回视图
    if (mv != null && !mv.wasCleared()) {
        //重点：渲染
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }
    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }
    //视图渲染完成，回调拦截器
    if (mappedHandler != null) {
        // Exception (if any) is already handled..
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}

//拦截器回调，通知拦截器，视图已被渲染，拦截器可以再做点事情
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
        throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            try {
                interceptor.afterCompletion(request, response, this.handler, ex);
            }
            catch (Throwable ex2) {
                logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
            }
        }
    }
}
```

### 视图解析和渲染

先判断是否需要视图解析器进行视图解析，最后调用解析出来的视图的 `render()` 方法进行渲染操作。

```
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    // Determine locale for request and apply it to the response.
    Locale locale =
            (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
    response.setLocale(locale);
    //真正的视图对象
    View view;
    String viewName = mv.getViewName();
    if (viewName != null) {
        //重点：使用视图解析器，生成正真的视图
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
        if (view == null) {
            throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
                    "' in servlet with name '" + getServletName() + "'");
        }
    }
    else {
        //不需要查找，ModelAndView中已经包含了真正的视图
        view = mv.getView();
        if (view == null) {
            throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
                    "View object in servlet with name '" + getServletName() + "'");
        }
    }

    if (logger.isTraceEnabled()) {
        logger.trace("Rendering view [" + view + "] ");
    }
    try {
        if (mv.getStatus() != null) {
            response.setStatus(mv.getStatus().value());
        }
        //重点：开始渲染
        view.render(mv.getModelInternal(), request, response);
    }
    catch (Exception ex) {
        if (logger.isDebugEnabled()) {
            logger.debug("Error rendering view [" + view + "]", ex);
        }
        throw ex;
    }
}
```

#### 视图解析

遍历视图解析器集合，不同的视图需要不同的解析器进行处理。

ViewResolver解析器是一个接口，他有几个实现类，对应支持的视图技术。

1. `AbstractCachingViewResolver`，抽象类，支持缓存视图，所有的解析器都继承它，它内部有一个Map，缓存解析过的视图对象，解决效率问题。
2. `UrlBasedViewResolver`，继承于 `AbstractCachingViewResolver`，当我们Controller返回一个字符串，例如success，它就会从我们的xml配置文件中，找到prefix前缀和suffix后缀，和url进行拼接，输出一个完成的视图地址。还有一种就是我们返回 `redirect:`前缀的字符串时，会解析为重定向视图View，进行重定向操作。
3. `InternalResourceViewResolver`，内部资源解析器，继承于上面的 `UrlBasedViewResolver`，所以 `UrlBasedViewResolver` 有的功能呢，它都有，主要用于加载 `/WEB-INF/` 目录下的资源。
4. 还有一些其他不太常用的解析器，这里就不介绍了

```
<!-- 配置视图解析器 -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 视图文件都从pages文件夹下找 -->
    <property name="prefix" value="/WEB-INF/pages/"/>
    <!-- 文件后缀为jsp -->
    <property name="suffix" value=".jsp"/>
</bean>
```

```
@Nullable
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
        Locale locale, HttpServletRequest request) throws Exception {
    if (this.viewResolvers != null) {
        //遍历视图解析器集合，不同的视图需要不同的解析器进行处理
        for (ViewResolver viewResolver : this.viewResolvers) {
            View view = viewResolver.resolveViewName(viewName, locale);
            if (view != null) {
                return view;
            }
        }
    }
    return null;
}

public interface ViewResolver {
    //尝试解析视图名称为视图对象，如果不能解析，返回null
	@Nullable
	View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

#### 视图渲染

视图解析完成，生成View视图对象，而View也是一个接口，它有以下实现类：

1. AbstractView，View的抽象类，定义了渲染流程，抽象了一些抽象方法，子类做特殊处理即可，大部分的实现类都继承于它
2. VelocityView，支持Velocity框架生成的页面。
3. FreeMarkerView，支持FreeMarker框架生成的页面。
4. JstlView，支持生成jstl视图。
5. RedirectView，支持生成页面跳转视图。
6.  MappingJackson2JsonView，输出Json的视图，使用Jackson库实现Json序列

视图的本质就是通过 `Response`对象，进行 `write()` 写出到客户端。

```
public interface View {
    String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";
    
    String PATH_VARIABLES = View.class.getName() + ".pathVariables";
    
    String SELECTED_CONTENT_TYPE = View.class.getName() + ".selectedContentType";
    
    //视图对应的的content-type
    @Nullable
    default String getContentType() {
        return null;
    }
    
    //渲染数据到视图
    void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
            throws Exception;
}
```

### 最终通知

`doDispatch()`方法，整体try-catch后，`finally` 代码块，调用拦截器进行最终通知。

遍历到的拦截器必须是 `AsyncHandlerInterceptor` 接口的实现类才行。

```
void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = interceptors.length - 1; i >= 0; i--) {
            if (interceptors[i] instanceof AsyncHandlerInterceptor) {
                try {
                    AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
                    asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
                }
                catch (Throwable ex) {
                    logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
                }
            }
        }
    }
}
```