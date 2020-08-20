# SpringBoot自动配置原理

SpringBoot现在基本是标配了，除非是老旧的项目，或者保守一点的企业，都会选择SpringBoot。

在SpringBoot出现之前，使用SSH或者SSM，都要很多的XML配置文件，而这些配置，在每个项目中，大部分都是相同的。

虽然都一样，但项目都要配置，可能会出现配置几小时，写代码几分钟的情况，把项目启动拖慢了。SpringBoot则是为了解决这种问题而生的，提高开发效率。

用过SpringBoot的小伙伴都知道，在IDEA使用SpringBoot Initializer，快速配置项目，写一个Controller就可以，快速搭建起Web项目。

SpringBoot给我们提供了大量的starter，里面已经帮我们配置了常用配置，如果我们需要改动，则在application.yml中配置即可。

SpringBoot之所以可以这样做，是因为它的设计策略，开箱即用和约定大于配置。

下面我们看下SpringBoot帮我们做了什么吧！

## 开箱即用原理

要使用SpringBoot，我们需要指定parent父工程

### pom指定parent父工程

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

点进去会发现，spring-boot-starter-parent也有父工程，就是`spring-boot-dependencies`，继续点进去

```
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-dependencies</artifactId>
<version>2.2.0.RELEASE</version>
<relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

`spring-boot-dependencies`看来是管理依赖和版本号的，所以我们依赖第三方库时，如果在这个依赖列表中有，则不需要写版本号了

```
<properties>
    <activemq.version>5.15.10</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.76</appengine-sdk.version>
    <artemis.version>2.10.1</artemis.version>
    <aspectj.version>1.9.4</aspectj.version>
    <assertj.version>3.13.2</assertj.version>
    <atomikos.version>4.0.6</atomikos.version>
    <awaitility.version>4.0.1</awaitility.version>
    <bitronix.version>2.1.4</bitronix.version>
    <build-helper-maven-plugin.version>3.0.0</build-helper-maven-plugin.version>
    <byte-buddy.version>1.10.1</byte-buddy.version>
    ...太多了，省略其他
</properties>

<dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot</artifactId>
        <version>2.2.0.RELEASE</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-test</artifactId>
        <version>2.2.0.RELEASE</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-test-autoconfigure</artifactId>
        <version>2.2.0.RELEASE</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-actuator</artifactId>
        <version>2.2.0.RELEASE</version>
      </dependency>
    <dependencies>
    ...太多了，省略其他
<dependencyManagement>
```

我们的父工程 `spring-boot-starter-parent`，还帮我们指定了配置文件的格式

```
<build>
    <resources>
      <resource>
        <filtering>true</filtering>
        <directory>${basedir}/src/main/resources</directory>
        <!-- 指定了配置文件的格式，加载顺序为yml => yaml => properties -->
        <includes>
          <include>**/application*.yml</include>
          <include>**/application*.yaml</include>
          <include>**/application*.properties</include>
        </includes>
      </resource>
      <resource>
        <directory>${basedir}/src/main/resources</directory>
        <excludes>
          <exclude>**/application*.yml</exclude>
          <exclude>**/application*.yaml</exclude>
          <exclude>**/application*.properties</exclude>
        </excludes>
      </resource>
    </resources>
<build>
```

### 启动器

SpringBoot将每种使用场景所需要的依赖和依赖，封装成一个启动器starter，我们需要引入某种领域的功能时，直接依赖对应的starer即可。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

例如我们常用的Web开发，需要依赖SpringMVC等，SpringBoot提供了 `spring-boot-starter-web` 启动器

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

我们点进行该starter，他给我们定义了以下依赖：

1. spring-boot-starter SpringBoot基础启动器
2. spring-boot-starter-json json序列化、反序列化的启动器
3. spring-boot-starter-tomcat 内嵌Tomcat
4. spring-web和spring-webmvc，就是我们的SpringMVC

```
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.2.0.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.2.0.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.2.0.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
      <version>2.2.0.RELEASE</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>
          <artifactId>tomcat-embed-el</artifactId>
          <groupId>org.apache.tomcat.embed</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.2.0.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.0.RELEASE</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```

### 启动类

SpringBoot要求我们提供一个启动类，并且类头加上 @SpringBootApplication注解，该注解就是SpringBoot启动的核心。

```
@SpringBootApplication
public class SpringbootEsApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootEsApplication.class, args);
        log.info("项目启动成功，访问地址：http://localhost:8081/");
    }
}
```

我们点进去 `@SpringBootApplication`注解

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
@ConfigurationPropertiesScan
public @interface SpringBootApplication {
    //省略属性...
}
```

我们会发现SpringBootApplication是一个复合注解，当中最重要的是 `@SpringBootConfiguration` 和 `@EnableAutoConfiguration`，这2个注解。
@ComponentScan注解是包扫描，因为没有配置扫描包，默认是扫描标识该注解的类的包，以及它以下的子包，所以启动类一般在根包下。

- @SpringBootConfiguration注解

我们发现 `@SpringBootConfiguration`注解 ，最主要是加上了 `@Configuration`注解。
我们知道 `@Configuration`注解 就代表了一个JavaConfig方式的Spring的容器，所以我们启动器类也相当于一个容器。

`SpringBootConfiguration`注解没什么可看了，我们看下一个注解

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration(
    proxyBeanMethods = false
)
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

- @EnableAutoConfiguration注解

`@EnableAutoConfiguration`注解中，主要注解是 `@Import(AutoConfigurationImportSelector.class)`。
@Import注解，帮我们导入了`AutoConfigurationImportSelector`这个类

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

Class<?>[] exclude() default {};

String[] excludeName() default {};
}
```

- AutoConfigurationImportSelector类

AutoConfigurationImportSelector类实现了`DeferredImportSelector`接口，该接口继承 `ImportSelector`接口 ，会要求复写`selectImports()`方法。

`ImportSelector`接口，主要是为了导入`@Configuration`配置的，而`DeferredImportSelector`是延期导入，当所有的`@Configuration`都处理完成后，再调用`DeferredImportSelector`进行处理。

所以`AutoConfigurationImportSelector`类是延迟导入的，所有`@Configuration`都处理完后，再调用它的`selectImports()`方法。

`selectImports()`方法，调用了`getAutoConfigurationEntry()`方法，而`getAutoConfigurationEntry()`又调用了`getCandidateConfigurations()`方法。
而`getCandidateConfigurations()`方法是重点！

```
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}

    protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
    			AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        configurations = removeDuplicates(configurations);
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationEntry(configurations, exclusions);
    }

	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
}
```

- getCandidateConfigurations()方法

方法中，调用SpringFactoriesLoader.loadFactoryNames()，传入2个参数，EnableAutoConfiguration的Class和Bean的ClassLoader。

`loadFactoryNames()`方法，返回一个集合，如果集合为空，进入下一句的Assert断言，就会抛出异常。

最后返回这个配置集合。

```
//AutoConfigurationImportSelector
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
            getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
            + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}

protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}

protected ClassLoader getBeanClassLoader() {
    return this.beanClassLoader;
}
```

- SpringFactoriesLoader

`loadFactoryNames()`方法，获取了传入的`EnableAutoConfiguration`注解的Class，调用loadSpringFactories()方法。
而`loadSpringFactories()`方法，会读取jar包中`META-INF`目录的`spring.factories`配置文件。

如果读取不到，则返回一个空集合。

```
public final class SpringFactoriesLoader {
    //jar包中的META-INF目录下，spring.factories配置文件
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        String factoryTypeName = factoryType.getName();
        return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
    }
    
    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = cache.get(classLoader);
        if (result != null) {
            return result;
        }
    
        try {
            Enumeration<URL> urls = (classLoader != null ?
                    classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                    ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
            result = new LinkedMultiValueMap<>();
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                for (Map.Entry<?, ?> entry : properties.entrySet()) {
                    String factoryTypeName = ((String) entry.getKey()).trim();
                    for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                        result.add(factoryTypeName, factoryImplementationName.trim());
                    }
                }
            }
            cache.put(classLoader, result);
            return result;
        }
        catch (IOException ex) {
            throw new IllegalArgumentException("Unable to load factories from location [" +
                    FACTORIES_RESOURCE_LOCATION + "]", ex);
        }
    }
}
```

- spring.factories配置文件

我们选一个starter，例如spring-boot-autoconfigure，找到它的META-INF目录，找到spring.factories文件，打开。

我们发现文件里，配置了很多自动配置属性（内容有删减，实在太多了！）。
它的形式是Key-Value，例如其中一个Key是EnableAutoConfiguration的全类名，它的Value是好几个名字以AutoConfiguration结尾的类，每个类之间用逗号分隔。

刚才我们跟踪的`loadFactoryNames()`方法，传入的EnableAutoConfiguration的Class，就是要从`spring.factories`配置文件中找到它对应的那一组Value。

我们以 `ServletWebServerFactoryAutoConfiguration` 为例，点进去看一下

```
# 省略其他配置...

# Auto Configure !!!!!!!! 重点在这里 !!!!!!!!
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration

# 省略其他配置...
```

- ServletWebServerFactoryAutoConfiguration类

我们看到该类的类头上，有`@EnableConfigurationProperties`注解，该属性表示加载配置属性，这里指定了一个 `ServerProperties` 类。

我们点进去 `ServerProperties` 类看一下

```
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
    //...省略
}
```

- ServerProperties类

这个是一个和配置信息相对应的类，它类头上配置了 `@ConfigurationProperties` 注解，它可以将配置文件中的配置项的内容，映射到我们的类的变量上。

注解上，配置的prefix属性，就代表了server.xxx系列配置，例如我们配置端口：server.port，该注解将我们的配置映射到 `ServerProperties` 上。

```
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
	/**
	 * Server HTTP port.
	 */
	private Integer port;

	/**
	 * Network address to which the server should bind.
	 */
	private InetAddress address;
}
```

到此为止，自动配置的流程基本通了，总结一下：

SpringBoot启动类的main方法启动时，会找@EnableAutoConfiguration注解，而该注解就在@SpringBootApplication上。而@EnableAutoConfiguration注解上，使用了@Import注解，导入了AutoConfigurationImportSelector类。
而该类，会去找META-INF/spring.factories配置文件，这个配置文件中配置了一系列的以AutoConfiguration结尾的类，就是自动配置类。
而每个配置类，都有一个Properties结尾的配置类，它和我们在yml中的配置项时一一对应的，相当于绑定配置到了该对象中。

如果只是想面试了解一下，到这里就可以了，而如果更想深入，就要继续跟一下。

如果要继续跟，就还有一个疑点，自动装配是什么时候开始的呢，其实就是 `AutoConfigurationImportSelector`类上的 `selectImports()`方法，还不知道它什么会被调用。

### 何时开始进行自动装配

我们回归到Spring，Spring应用启动，会在 `AbstractApplicationContext` 类中，调用 `refresh()` 方法。

`refresh()`方法中，调用了 `invokeBeanFactoryPostProcessors()` 方法，该方法是用来处理 `BeanFactoryPostProcessor`接口的，而 `BeanFactoryPostProcessor` 的有一个子接口 `BeanDefinitionRegistryPostProcessor`。

```
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
    @Override
	public void refresh() throws BeansException, IllegalStateException {
        //省略无关代码...

        invokeBeanFactoryPostProcessors(beanFactory);

        //省略无关代码...
    }
}

//BeanDefinitionRegistryPostProcessor
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
    void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```

- ConfigurationClassPostProcessor类

子接口BeanDefinitionRegistryPostProcessor，有一个实现类 `ConfigurationClassPostProcessor`，它是专门处理 `@Configuration` 注解的。

`processConfigBeanDefinitions()`方法中，就是处理 `@Configuration` 注解的类。主要是使用`ConfigurationClassParser` 类的 `parse()` 方法。

我们进去 `parse()` 方法，看一下

```
public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor,
		PriorityOrdered, ResourceLoaderAware, BeanClassLoaderAware, EnvironmentAware {

    public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
        //省略部分代码        

        // Parse each @Configuration class
        ConfigurationClassParser parser = new ConfigurationClassParser(
                this.metadataReaderFactory, this.problemReporter, this.environment,
                this.resourceLoader, this.componentScanBeanNameGenerator, registry);

        Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
        Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
        do {
            //解析处理@Configuration注解的类
            parser.parse(candidates);
            parser.validate();

            Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
            configClasses.removeAll(alreadyParsed);

            // Read the model and create bean definitions based on its content
            if (this.reader == null) {
                this.reader = new ConfigurationClassBeanDefinitionReader(
                        registry, this.sourceExtractor, this.resourceLoader, this.environment,
                        this.importBeanNameGenerator, parser.getImportRegistry());
            }
            this.reader.loadBeanDefinitions(configClasses);
            alreadyParsed.addAll(configClasses);

            candidates.clear();
            if (registry.getBeanDefinitionCount() > candidateNames.length) {
                String[] newCandidateNames = registry.getBeanDefinitionNames();
                Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
                Set<String> alreadyParsedClasses = new HashSet<>();
                for (ConfigurationClass configurationClass : alreadyParsed) {
                    alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
                }
                for (String candidateName : newCandidateNames) {
                    if (!oldCandidateNames.contains(candidateName)) {
                        BeanDefinition bd = registry.getBeanDefinition(candidateName);
                        if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                                !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                            candidates.add(new BeanDefinitionHolder(bd, candidateName));
                        }
                    }
                }
                candidateNames = newCandidateNames;
            }
        }
        while (!candidates.isEmpty());
    }
}
```

- ConfigurationClassParser类的parse()方法

首先类中有一个内部类 `DeferredImportSelectorHandler`，构造方法 `ConfigurationClassParser` 实例时，就创建该内部类的实例。

在 `parse()` 方法调用时，最后一句调用了 `processDeferredImportSelectors()`方法。

```
class ConfigurationClassParser {
    public void parse(Set<BeanDefinitionHolder> configCandidates) {
            this.deferredImportSelectors = new LinkedList<DeferredImportSelectorHolder>();
        for (BeanDefinitionHolder holder : configCandidates) {
            BeanDefinition bd = holder.getBeanDefinition();
            try {
                if (bd instanceof AnnotatedBeanDefinition) {
                    parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
                }
                else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                    parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
                }
                else {
                    parse(bd.getBeanClassName(), holder.getBeanName());
                }
            }
            catch (BeanDefinitionStoreException ex) {
                throw ex;
            }
            catch (Throwable ex) {
                throw new BeanDefinitionStoreException(
                        "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
            }
        }
        //重点
        processDeferredImportSelectors();
    }
}
```

- processDeferredImportSelectors()方法

重点在 `String[] imports = deferredImport.getImportSelector().selectImports(configClass.getMetadata());`。

调用的是 `DeferredImportSelectorHolder`类，它保存了 `DeferredImportSelector` 的引用，在这个for循环中，调用了 `DeferredImportSelector` 的 `selectImports()`方法，从而调用到了我们之前分析的 `AutoConfigurationImportSelector` 类中的 `selectImports()`方法了。

```
private void processDeferredImportSelectors() {
    List<DeferredImportSelectorHolder> deferredImports = this.deferredImportSelectors;
    this.deferredImportSelectors = null;
    Collections.sort(deferredImports, DEFERRED_IMPORT_COMPARATOR);

    for (DeferredImportSelectorHolder deferredImport : deferredImports) {
        ConfigurationClass configClass = deferredImport.getConfigurationClass();
        try {
            String[] imports = deferredImport.getImportSelector().selectImports(configClass.getMetadata());
            processImports(configClass, asSourceClass(configClass), asSourceClasses(imports), false);
        }
        catch (BeanDefinitionStoreException ex) {
            throw ex;
        }
        catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                    "Failed to process import candidates for configuration class [" +
                    configClass.getMetadata().getClassName() + "]", ex);
        }
    }
}

//该类，保存了配置类和DeferredImportSelector的引用
private static class DeferredImportSelectorHolder {
    private final ConfigurationClass configurationClass;

    private final DeferredImportSelector importSelector;

    public DeferredImportSelectorHolder(ConfigurationClass configClass, DeferredImportSelector selector) {
        this.configurationClass = configClass;
        this.importSelector = selector;
    }

    public ConfigurationClass getConfigurationClass() {
        return this.configurationClass;
    }

    public DeferredImportSelector getImportSelector() {
        return this.importSelector;
    }
}
```

## 参考资料

[SpringBoot：认认真真梳理一遍自动装配原理](https://zhuanlan.zhihu.com/p/95217578)

[Spring Boot面试杀手锏 — 自动配置原理](https://blog.csdn.net/u014745069/article/details/83820511)

[深入理解SpringBoot之自动装配](https://www.cnblogs.com/niechen/p/9027804.html)