#### Spring学习之XML方式

Spring，Java界的Web开发的必备框架，主要功能是IoC和AOP，Spring的IoC容器，提供了对象管理、对象注入等实用功能，而对象配置有3种方式：

- XML文件配置
- 注解配置
- JavaConfig

本篇是看了[Spring之路系列文章](https://m.imooc.com/article/300915)，而进行的总结。

#### 容器定义和管理对象

- 准备模型

既然要管理对象，就需要一个实体类，我们创建一个司机类

```
/**
 * 司机类
 */
public class Driver {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void work() {
        System.out.println("司机" + this.name + "开始送货啦");
    }
}
```

- 增加配置文件

我们在项目的resources文件夹下新建一个Spring配置的XML。在XML中添加一个bean对象

1. bean标签：定义一个bean对象
2. id属性：这个bean对象的唯一标识
3. class属性：这个bean对象的全类名
4. property标签：定义需要注入到bean对象中属性，基本数据类型时使用，如果是自定义对象，需要使用rel属性，下面会有讲

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
         
    <!-- 定义一个司机类实例 -->
    <bean id="driver1" class="com.zh.springxmlsample.model.Driver">
        <property name="name" value="张三"/>
    </bean>
</beans>
```

- 创建Spring容器

Spring容器的创建和使用，都是使用ClassPathXmlApplicationContext对象，ClassPathXmlApplicationContext构造时可以传入xml文件的名称，即可将模型类对象交给Spring容器管理。

获取bean实例，使用getBean()方法，传入刚才在XML中定义的Driver类实例的id，即可返回Driver类的实例。

```
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
        //获取一个司机实例
        Driver driver1 = (Driver) context.getBean("driver1");
        driver1.work();
    }
}

//输出
司机张三开始送货啦
歌手[周杰伦]开唱啦，快挥舞起你手中的荧光棒吧
歌手[林俊杰]开唱啦，快挥舞起你手中的荧光棒吧
```

#### 显示装配

上面我们将司机类的实例定义在XML中使用bean标签配置，并在Java代码中获取到，其实就是Spring的显示装配，但只司机类只配置了一个基础类型的name，但往往我们实际中需要配置一些自定义对象，下面我们来学习一下怎么配置自定义对象。

- 定义2个模型类，歌手类和舞者类，以及一个舞台

舞台上有一个歌手类实例和一个舞者类实例

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

- XML配置中，配置实例，舞台依赖歌手和舞者的实例，由于舞台实例中需要依赖2个自定义对象，我们使用ref属性来指定。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 歌手：刀郎 -->
    <bean id="daolang" class="com.zh.springxmlsample.model.Singer">
        <property name="name" value="刀郎"/>
    </bean>

    <!-- 舞者：刘迦 -->
    <bean id="liujia" class="com.zh.springxmlsample.model.Dancer">
        <property name="name" value="刘迦"/>
    </bean>

    <!-- 舞台，手动指定装配 -->
    <bean id="stage" class="com.zh.springxmlsample.model.Stage">
        <property name="singer" ref="daolang"/>
        <property name="dancer" ref="liujia"/>
    </bean>
</beans>
```

- 创建容器并获取舞台实例，因为XML中配置了舞台类实例stage中歌手为daolang，舞者实例为dancer，所以我们只需要指定需要获取舞台实例，内部的歌手、舞者都会自动注入对应的实例。

```
public class Main2 {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring2.xml");
        //获取舞台实例
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

自动装配有2种：

- 按成员属性的类型

我们将XML配置中的显示配置去掉，改为以下形式，主要是autowire属性，我们指定为byType，Spring会查找实例中成员的类型，逐个去容器中查找，但如果类型的实例在容器中有多个时，Spring就蒙圈了，不知道要注入谁，就会报错。

```
<!-- 舞台，自动根据类型装配 -->
<bean id="stage" class="com.zh.springxmlsample.model.Stage" autowire="byType">
     autowire="byType"，表示按成员的类型自动装配 
</bean>
```

- 按成员类型的名字

XML配置中，autowire属性指定为byName，Spring会查找实例中的成员名称作为bean的id，逐个去容器中查找。

```
<bean id="stage" class="com.zh.springxmlsample.model.Stage" autowire="byName">
     autowire="byName"，表示按成员的名字自动装配 
</bean>
```

模型类的成员名称要改为容器中定义的实例名称，所以不能乱改，否则就查找不到了

```
/**
 * 舞台类
 */
public class Stage {
    //歌手：按名字装配，对应daolang
    private Singer stage;
    //舞者：按名字装配，对应liujia
    private Dancer liujia;

    //...省略set、get方法
}
```

#### 总结

XML方式使用Spring，主要分3步

1. 定义模型类
2. XML中配置实例和依赖
3. 创建Spring容器，加载配置容器，获取指定的实例

XML方式项目会用得比较少，而使用注解和JavaConfig的方式比较多，但学习、了解下XML方式还是有必要的~