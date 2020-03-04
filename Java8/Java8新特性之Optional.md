#### Java8新特性之Optional

说起Java中最常见也最让人头痛的异常，莫属**NullPointerException**空指针异常了。Android开发中，接口json中，对象里面有对象，对象中有属性。每次使用或作为参数传参时，都要!=null非空判断。一个还好，当属性嵌套得非常深的时候，就会出现以下代码。

```
public class Person {
    private int age;
    private String name;
    private Address address;

    public Person(int age, String name, Address address) {
        this.age = age;
        this.name = name;
        this.address = address;
    }

    //...省略set、get方法

    public static class Address {
        private String roomNum;
        private String areaNum;

        public Address(String roomNum, String areaNum) {
            this.roomNum = roomNum;
            this.areaNum = areaNum;
        }
        
        //...省略set、get方法
    }
}

//错误码
String errorNum = "error";

if (barry.getAddress() != null 
        && barry.getAddress().getAreaNum() != null 
        && !barry.getAddress().getAreaNum().equals(errorNum)) {
    String areaNum = barry.getAddress().getAreaNum();
    //最后使用areaNum属性做事情
    System.out.println("areaNum = " + areaNum);
}
```

总之，就是不好看，臃肿，也不优雅

#### Java8中的解决方案

Java8中，为了解决以上问题，引入了一个叫Optional的类。它的解释如下：

> 这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

#### 创建

既然Optional可以包裹null值，那么Optional怎么创建呢？

- 通过Optional.of()

```
//of包裹非null值
Optional<String> optional = Optional.of("wally");
//of包裹null值，会抛出NullPointerException
Optional<Object> nullOptional = Optional.of(null);
```

- 通过Optional.ofNullable()

```
//ofNullable传入null不会抛出NullPointerException
Optional<Object> optional = Optional.ofNullable(null);
```

Optional.of()和Optional.ofNullable()很类似，唯一区别就是Optional.of()不能接受null值，传入null值会抛出NullPointerException，而Optional.ofNullable()可以接受。

#### isPresent

isPresent，判断Optional中包裹的值，如果不为null，返回true，为null返回false。

```
Optional<Object> optional = Optional.ofNullable(null);
if (optional.isPresent()) {
    Object value = optional.get();
    //做操作
    System.out.println(value);
}
```


#### get

获取Optional中包裹的值，如果值为null，抛出NoSuchElementException异常，一般会配合isPresent()进行判断后，再使用get获得值。

```
Optional<String> optional = Optional.ofNullable("wally");
if (optional.isPresent()) {
    String value = optional.get();
    //做操作...
}
```

#### orElse

判断Optional中包裹的值，如果值为null，返回orElse()中传入的默认值。

```
Optional<Object> optional = Optional.ofNullable(null);
Object value = optional.orElse("value is null");
System.out.println("value: " + value);
```

#### orElseGet

和orElse类似，但是默认值通过Supplier接口的get()方法返回

```
//对象方式
Object value = optional.orElseGet(new Supplier<Object>() {
    @Override
    public Object get() {
        return "value is null";
    }
});

//lambda方式
Object value = optional.orElseGet(() -> "value is null");
//最后输出结果
System.out.print("value: " + value);
```

#### orElseThrow

判断Optional中包裹的值，如果为null，抛出Supplier接口中get()返回的Throwable异常对象。否则返回Optional中包裹的值。

```
try {
	//对象方式
    Object value = optional.orElseThrow(new Supplier<Throwable>() {
        @Override
        public Throwable get() {
            return new NullPointerException("value is null");
        }
    });
    //lambda方式
    Object value = optional.orElseThrow((Supplier<Throwable>) () -> new NullPointerException("value is null"));
    //输出
    System.out.println("value: " + value);
} catch (Throwable throwable) {
    throwable.printStackTrace();
}
```

#### map

判断Optional中包裹的值，如果不为null，调用Function接口的apply()方法，获得新值，生成新的一个Optional并包裹新值。如果为null，则返回一个null值的Optional。

```
Optional<String> newOptional = Optional.of("hello").map(new Function<Object, String>() {
    @Override
    public String apply(Object o) {
        return "hi";
    }
});
if (newOptional.isPresent()) {
    String value = newOptional.get();
    System.out.println(value);
}
```

#### flatMap

flatMap和map类似，但是Function接口的apply方法返回的是Optional对象。注意apply方法返回的Optional中包裹的值不能为null，否则抛出NullPointerException异常。

```
Optional<String> optional = Optional.of("oh no").flatMap(new Function<String, Optional<String>>() {
    @Override
    public Optional<String> apply(String s) {
        return Optional.of("oh shit");
    }
});
optional.ifPresent(System.out::println);
```

#### filter

对Optional中包裹的值，做过滤操作，调用Predicate中的test方法，做过滤操作判断，如果返回true，返回原始的Optional，否则返回一个null值的Optional。配合ifPresent()判断，过滤成功则不会调用输出方法。

```
Optional<String> stringOptional = Optional.of("wally").filter(new Predicate<String>() {
    @Override
    public boolean test(String value) {
        return value.length() > 3;
    }
});
stringOptional.ifPresent(System.out::println);
```

#### 级联判空

json转换成model，可能会model中有model，而model中的某个值才是我们想要的，并且还需要判断值的正确性。

例如我们需要Person类中的Address对象中的areaNum属性，并且要校验areaNum的值的合法性。

- 不使用Optional，嵌套越深，if-else和!=null写得越多，很难看

```
//错误码
String errorNum = "error";

//取出Address属性对象
if (barry.getAddress() != null 
		 //取出Address属性对象中的areaNum属性
        && barry.getAddress().getAreaNum() != null 
        //校验areaNum的合法性
        && !barry.getAddress().getAreaNum().equals(errorNum)) {
    String areaNum = barry.getAddress().getAreaNum();
    //最后使用areaNum属性做事情
    System.out.println("areaNum = " + areaNum);
}
```

- 使用Optional，所有操作都使用flatMap铺平，每个操作都清清楚楚

- 要使用Optional，必须将所有属性都用Optional包一次~所以我们的Person类的Model就要改一下~

```
public class PersonWithOptional {
    private Optional<Integer> age;
    private Optional<String> name;
    private Optional<Address> address;

    public PersonWithOptional(Optional<Integer> age, Optional<String> name, Optional<Address> address) {
        this.age = age;
        this.name = name;
        this.address = address;
    }

    public static class Address {
        private Optional<String> roomNum;
        private Optional<String> areaNum;

        public Address(Optional<String> roomNum, Optional<String> areaNum) {
            this.roomNum = roomNum;
            this.areaNum = areaNum;
        }

        //..省略set、get方法
    }

	//..省略set、get方法
}
```

```
Optional.of(barry).flatMap(new Function<PersonWithOptional, Optional<PersonWithOptional.Address>>() {
    @Override
    public Optional<PersonWithOptional.Address> apply(PersonWithOptional person) {
    	  //取出Address属性对象
        return barry.getAddress();
    }
}).flatMap(new Function<PersonWithOptional.Address, Optional<String>>() {
    @Override
    public Optional<String> apply(PersonWithOptional.Address address) {
        //取出Address属性对象中的areaNum属性
        return address.getAreaNum();
    }
}).filter(new Predicate<String>() {
    @Override
    public boolean test(String areaNum) {
        //校验areaNum的合法性，不合法在下面的ifPresent时被抛弃掉
        return !areaNum.equals(errorNum);
    }
}).ifPresent(new Consumer<String>() {
    @Override
    public void accept(String areaNum) {
        //最后使用areaNum属性做事情
        System.out.println("areaNum = " + areaNum);
    }
});
```

- 这时候，有人又会说了，你这不是纯属增加代码量吗，这比传统写法写多好多代码呀！对于这种情况，终究还是那句话，代码是人看的，代码太复杂不利于阅读，尤其是那种写完几个星期后，只有上帝才知道是什么的代码~况且我们还有lambda这个大杀器！

- 使用lambda,省略了内部类，是不是代码量少了，也依然结构清晰~

```
Optional.of(barry)
.flatMap(person -> barry.getAddress())
.flatMap(PersonWithOptional.Address::getAreaNum)
.filter(areaNum -> !areaNum.equals(errorNum))
.ifPresent(areaNum -> System.out.println("areaNum = " + areaNum));
```