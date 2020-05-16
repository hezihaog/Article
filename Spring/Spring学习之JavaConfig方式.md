#### Spring学习之JavaConfig方式

Spring，Java界的Web开发的必备框架，主要功能是IoC和AOP，Spring的IoC容器，提供了对象管理、对象注入等实用功能，而对象配置有3种方式：

- XML文件配置
- 注解配置
- JavaConfig

本篇是看了[Spring之路系列文章](https://m.imooc.com/article/300915)，而进行的总结。

#### 为什么有JavaConfig方式

其实通过上一篇[Spring学习之注解方式](https://www.jianshu.com/p/2b3dcd98a6a0)，我们会发现一个问题，不同参数的实体类，都要新建一个类，会导致类剧增和冗余，而这些类不同之处只是注入的数据不同，所以就有了JavaConfig的方式，将只是参数不同的实体类，统一放到一个配置类中，提供返回值不同参数的实例，来达到配置。

除了类不会冗余外，还有一个原因，如果需要注入的实例是别的框架的类对象时，@Component注解是加不了到别人的类中的，这时就需要用到JavaConfig的方式，手工返回第三方库中的对象。

注意：JavaConfig的方式，实体类不需要增加注解，而是干净的实体类，我们将创建不同实例的职责放到一个Configuration类中。

#### 使用

JavaConfig方式，不需要新建XML配置文件，而是需要创建一个配置的Java类。

- 干干净净的实体类

```
/**
 * 歌手类
 */
public class Singer {
    private String name;

    public void sing() {
        System.out.println("歌手[" + name + "]开唱啦，快挥舞起你手中的荧光棒吧");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Singer{" +
                "name='" + name + '\'' +
                '}';
    }
}

/**
 * 舞者类
 */
public class Dancer {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Dancer{" +
                "name='" + name + '\'' +
                '}';
    }
}

/**
 * 舞台类
 */
public class Stage {
    /**
     * 歌手
     */
    private Singer singer;
    /**
     * 舞者
     */
    private Dancer dancer;

    public Singer getSinger() {
        return singer;
    }

    public void setSinger(Singer singer) {
        this.singer = singer;
    }

    public Dancer getDancer() {
        return dancer;
    }

    public void setDancer(Dancer dancer) {
        this.dancer = dancer;
    }

    @Override
    public String toString() {
        return "Stage{" +
                "singer=" + singer +
                ", dancer=" + dancer +
                '}';
    }
}
```

- 新建配置类

	- @Configuration，表示这个类为配置类，Spring需要扫描该类中的bean
	- @Bean，标识提供不同参数实例的方法。
		- 可传入一个autowire属性，值有NO、BY_TYPE、BY_NAME。
			- Autowire.BY_TYPE，表示里面的成员，按成员类型，在容器中查询，装配该Bean
			- autowire = Autowire.BY_NAME，表示里面的成员，按成员的变量名，在容器中查询，装配该Bean
		- 可传入一个name属性，标识返回该Bean实例的Id，如果不传，则以方法名作为Id。

```
/**
 * Configuration注解，表示这个类为配置类，Spring需要扫描该类中的bean
 */
@Configuration
public class BeanConfiguration {
    @Bean
    public Singer daolang() {
        Singer singer = new Singer();
        singer.setName("刀郎");
        return singer;
    }

    @Bean
    public Dancer liujia() {
        Dancer dancer = new Dancer();
        dancer.setName("刘迦");
        return dancer;
    }

    /**
     * 因为我们的Stage模型，并没有加@Component注解，并没有id，所以只能按照类型来查找
     * autowire = Autowire.BY_TYPE，表示里面的成员，按成员类型，在容器中查询，装配该Bean
     * autowire = Autowire.BY_NAME，表示里面的成员，按成员的变量名，在容器中查询，装配该Bean
     */
    @Bean(autowire = Autowire.BY_TYPE)
    public Stage stage() {
        return new Stage();
    }
}
```

- 创建容器和获取实例

需要注意，JavaConfig的方式，创建的Context容器是AnnotationConfigApplicationContext，而不是ClassPathXmlApplicationContext了，并且需要将配置类的class传入。

```
public class Main4 {
    public static void main(String[] args) {
        //这里需要使用AnnotationConfigApplicationContext，而不是ClassPathXmlApplicationContext了
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfiguration.class);
        Stage stage = (Stage) context.getBean("stage");
        System.out.println("舞台歌手：" + stage.getSinger());
        System.out.println("舞台舞者：" + stage.getDancer());
    }
}

//输出
舞台歌手：Singer{name='刀郎'}
舞台舞者：Dancer{name='刘迦'}
```

#### 自动装配

- 按成员变量的类型自动装配

上面的Stage类中，有Singer和Dancer需要注入，我们在提供Stage类实例的方法上使用的@Bean注解，autowire属性指定的事Autowire.BY_TYPE，则按照Singer和Dancer去容器中匹配查找。

- 按成员变量的名称自动装配

以上面Stage类来看，我们只要将Singer和Dancer的变量名，分别改为daolang和liujia，并且autowire属性指定的事Autowire.BY_NAME即可。

```
/**
 * 舞台类
 */
public class Stage {
    /**
     * 歌手
     */
    private Singer daolang;
    /**
     * 舞者
     */
    private Dancer liujia;

    //省略get、set、toString方法
}
```

#### 总结

JavaConfig和注解方式是共存的，我们应该看情况使用，如果需要注入的类是第三方库中的对象，或者是多种只是不同参数的实例，则使用JavaConfig方式。如果是自己自定义类，每个实例之间处理差别很大，则可以使用注解的方式注入。