#### Java8新特性之方法引用

上一节，我们介绍了lambda表达式，例如点击事件中，我们使用lambda表达式来创建匿名方法，但有时候我们只是调用了一个存在方法。

```
View.OnClickListener listener = view -> {
		//点击处理代码量比较大，抽取成另外一个函数进行调用
      OnItemClick();
}
view.setOnClickListener(listener);
```

但其实在Java8中，还可以进一步简写，就是使用Java8中的新特性，方法引用

```
view.setOnClickListener(this::listener);
```

#### 方法引用的格式

| 类型 | 示例格式 |
| ------ | ------ |
| 引用静态方法 | ClassName::staticMethodName |
| 引用某个对象的实例方法 | object::instanceMethodName |
| 引用构造方法 | ClassName::new |

- 注意方法名后**不需要**添加括号()

#### 举个栗子🌰

```
public class Person {
    private int age;
    private String name;

    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }

	//...省略set\get方法

    /**
     * 提供静态方法比较2个对象，按年龄比较
     */
    public static int compareByAge(Person o1, Person o2) {
        return Integer.compare(o1.getAge(), o2.getAge());
    }
}
```

#### 引用静态方法

我们在Person类中，添加一个compareByAge()静态方法用于按年龄比较

```
Person wally = new Person(18, "Wally");
Person barry = new Person(22, "Barry");

//传统方式书写
Arrays.asList(wally, barry).sort(new Comparator<Person>() {
    @Override
    public int compare(Person o1, Person o2) {
        return Person.compareByAge(o1, o2);
    }
});

//lambda表达式书写
Arrays.asList(wally, barry).sort((o1, o2) -> Person.compareByAge(o1, o2));

//方法引用书写
Arrays.asList(wally, barry).sort(Person::compareByAge);
```

#### 引用某个对象的实例方法

新建一个Provider类，添加一个实例方法用于按年龄比较

```
/**
 * 比较方法提供类
 */
private static class Provider {
    /**
     * 提供一个实例方法比较
     */
    public int compareByAge(Person o1, Person o2) {
        return Integer.compare(o1.getAge(), o2.getAge());
    }
}

//再调用该实例方法
Person wally = new Person(18, "Wally");
Person barry = new Person(22, "Barry");
Arrays.asList(wally, barry).sort(new Provider()::compareByAge);
```

#### 引用构造方法

```
Person wally = new Person(18, "Wally");
Person barry = new Person(22, "Barry");

/**
 * 提供一个静态方法，传入Provider进行排序
 */
private static void sort(List<Person> list, Supplier<Provider> provider) {
    list.sort(new Comparator<Person>() {

        @Override
        public int compare(Person o1, Person o2) {
            return provider.get().compareByAge(o1, o2);
        }
    });
}

List<Person> people = Arrays.asList(wally, barry);
sort(people, Provider::new);
```