#### Spring学习之注解方式

Spring，Java界的Web开发的必备框架，主要功能是IoC和AOP，Spring的IoC容器，提供了对象管理、对象注入等实用功能，而对象配置有3种方式：

- XML文件配置
- 注解配置
- JavaConfig

本篇是看了[Spring之路系列文章](https://m.imooc.com/article/300915)，而进行的总结。

#### 注解包扫描配置

要使用Spring的注解，必须要在XML配置中增加包扫描的配置，我们的模型类定义的包名为：com.zh.spring.anno

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置包扫描 -->
    <context:component-scan base-package="com.zh.spring.anno"/>
</beans>
```

#### 使用注解注入

我们需要使用的注解有@Component、@Value

- @Component注解，标识一个Bean，可以传入一个实例的Id，如果不传，则以类名全小写作为这个实例的Id
- @Value注解，给Bean的一个基础类型的成员属性设置值，如果需要注入自定义对象，则需要使用下面的@Autowired注解
- @Autowired注解，注入自定义对象时使用，但是如果有多个实例时，Spring则不知道注入哪个，就会报错，这时需要使用@Qualifier注解来指定注入具体的实例
- @Qualifier注解，可以传入一个实例的id，指定注入特定id的实例

下面，我们将3个模型类，重新使用Spring的注解进行重写

```
/**
 * 歌手类
 */
@Component(value = "zhoujielun")
public class Singer {
    @Value("周杰伦")
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
@Component("liujia")
public class Dancer {
    //设置值
    @Value("刘迦")
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
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
@Component("stage")
public class Stage {
    //注入
    @Autowired
    @Qualifier("zhoujielun")
    private Singer singer;
    
    @Autowired
    @Qualifier("liujia")
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

- 创建容器，获取Bean实例

```
public class Main3 {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring3.xml");
        //获取注解标识的Bean实例
        Stage stage = (Stage) context.getBean("stage");
        stage.getSinger();
        System.out.println("舞台歌手：" + stage.getSinger());
        System.out.println("舞台舞者：" + stage.getDancer());
    }
}

//输出
歌手[周杰伦]开唱啦，快挥舞起你手中的荧光棒吧
舞台歌手：Singer{name='周杰伦'}
舞台舞者：Dancer{name='刘迦'}
```

#### 自动装配

自动装配依旧是2种：

- 按成员的类型查找

当我们需要注入的实例只有一个时才可以使用，上面的例子中，直接使用@Autowired就是按类型查找

- 按成员的名称查找

当我们的实例有多个时，就不能按类型查找了，我们需要增加一个@Qualifier注解，指定实例的Id即可

#### 问题

其实通过上面的例子，我们发现一个问题，如果多个歌手实例，就需要创建多个类实例，而每个类之间的差别只是一些成员变量的值不同，会造成类的冗余。我们在下一篇[Spring学习之JavaConfig方式](https://www.jianshu.com/p/1170ad6e3b1d)文章中，使用JavaConfig可以解决这个问题。

#### 总结

从上面的例子可见，按类型和按名称，只相差一个@Qualifier和@Component注解加一个实例Id，修改起来是相当方便的。

**需要注意**的是，即使使用注解的方式，XML还是不能抛弃的，最少也要指定一个注解扫描的包名。

注解的可读性也比XML的好，所以也比较常用。