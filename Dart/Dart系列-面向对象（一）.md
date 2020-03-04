#### Dart系列-面向对象（一）

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的面向对象（一）。

#### 类和对象

dart也是一门面向对象的语言，所以也有类和对象的概念。

- 类的声明，使用class关键字定义类。

```
class Person {
	//类成员
	String name;
	int age;
	//final修饰的属性，不能被外部重新赋值，只可读，不可写
	final String address = null;
	
	void work() {
    print('Name is $name, Age is $age, He is working');
  }
}
```

- 类的私有方法

不同Java有丰富的权限操作符。dart默认都是公开的，在变量名或方法名前加入_前缀即为私有。

```
class Person {
  //_前缀可以标识属性，代表属性是私有的
  String _name;
  
  //_前缀也可以标识方法，表示方法是私有的
  	void _work() {
    print('Name is $_name, Age is $age, He is working');
  }
}
```

- dart的方法不支持重载。

```
class Person {
  void work() {
    print('Name is $name, Age is $age, He is working');
  }
  
  //不支持重载，这样写是编译不通过的
  void work(int a) {

  }
}
```

- 创建对象，使用new关键字或不写new关键字

```
void main() {
  //var person = new Person();
  var person = Person();
  person.name = 'Tom';
  person.age = 20;
  print(person.name);
  person.work();

  //final属性不能重新赋值
  //person.address = 'china';
}
```

#### 计算属性

例如我们有个矩形类，有宽和高属性，需要计算它的面积时，我们通常会建立一个方法去计算再返回。而dart则提供了计算属性，dart允许申明一个计算属性来通过计算得出的结果做为属性的值。

- 传统创建方法进行计算的方式

```
class Rectangle {
  num width;
  num height;
  
  //自定义方法来实现求面积
  num area() {
    return width * height;
  }
}
```

- 使用dart提供的计算属性

和申明变量的格式一样，加上get关键字，表示为计算属性，右边是箭头函数，写成一对花括号也是一样的。

```
class Rectangle {
  num width;
  num height;
  
  //计算属性，使用get关键字标识，生成一个area属性，他是width和height计算而得来的
  num get area => width * height;
}
```

- 计算属性的设置

计算属性除了内部定义后，进行get操作外，还可以进行赋值操作。例如面积的反算，根据一个高度值，计算出宽度。再将计算出来的值赋值给宽度属性。

```
class Rectangle {
  num width;
  num height;
  
  //计算属性，使用get关键字标识，生成一个area属性，他是width和height计算而得来的
  num get area => width * height;
  
  //给计算属性进行赋值，使用set关键字
  set area(value) {
    //结果赋值给另外一个属性
    width = value / 20;
  }
}

```

#### 构造方法

既然有类的声明，就有类的构造方法。

- 默认编译器会给类添加一个无参构造方法。手动声明也可以，效果是一样的。

```
class Person {
  String name;
  int age;

	//默认无参构造方法
  Person() {
  }
  
  void work() {
    print('Name is $name, Age is $age, Gender is $gender, He is working');
  }
}

void main() {
	Person person = new Person();
	person.name = 'Tom';
    person.age = 20;
    person.work();
}
```

- 有参构造方法。还可以使用dart提供的语法糖进行声明。

```
class Person {
  //定义一个有参构造，就不会默认生成一个无参构造方法了
  Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
  
  //dart提供的语法糖
  //直接赋值的方式，this直接指定，它是在构造方法执行之前进行赋值的！
  //Person(this.name, this.age);
}
```

- 命名构造方法。但如果类已经声明了一个有参构造方法了，编译器就不会给类添加一个无参构造了。那如果我需要一个无参构造方法，还需要一个有参构造方法，像Java那样声明是不可以的，dart没有方法重载，而构造方法也一样，这时候就需要用到命名构造方法，格式：类型.命名(参数)。

```
class Person {
  //...
  
  //如果我想有一个有参构造方法，也有一个无参构造方法，dart是不能重载的，所以不能添加
  //这时候就需要用到命名构造方法，格式为：类名.方法名
  Person.withName(String name) {
    this.name = name;
  }

  //命名构造方法也可以使用直接设置属性的语法糖
  Person.withAge(this.age);

  //...
}
```

#### 常量构造方法

当类的属性设置一次之后，就不会再设置了，那么这个类就可以声明为常量类，常量类的属性使用final修饰，而构造方法使用const修饰。

```
class Person {
  final String name;
  final int age;
  final String gender;

  const Person(this.name, this.age, this.gender);

  void work() {
    print('Name is $name, Age is $age, Gender is $gender, He is working');
  }
}

void main() {
  //如果需要将对象作为常量，就需要将构造方法声明为常量构造方法
  //使用常量构造方法的对象，属性和构造方法都为常量，属性使用final修饰，构造方法使用const修饰
  //常量型对象运行时更快，如果对象的值设置一次后就不会改变，可以使用这种方式
  const person = const Person('Tom', 18, 'Male');
  person.work();
}
```

#### 工厂构造方法

在Java中，我们会使用工厂设计模式，生产不同类型的实例，而dart则提供了工厂构造方法的语法糖，方便我们实现工厂模式生产实例。

- 工厂构造方法，使用factory关键字声明，这个工厂以一个String类型的key作为缓存key，类的实例作为缓存value。当没有缓存时构造实例，并缓存起来，如果有缓存则取出并返回。

```
class Logger {
  final String name;

  static final Map<String, Logger> _cache = Map();

  //工厂构造方法使用factory关键字，和构造方法的区别是，工厂构造方法能有返回值，而构造方法不行
  factory Logger(String name) {
    //如果有缓存，直接返回
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      //否则使用命名构造方法生成对象，再放到缓存中
      final logger = Logger._internal(name);
      _cache[name] = logger;
      //再返回出去
      return logger;
    }
  }

  //_前缀也可以修饰命名构造方法，标识为私有的
  Logger._internal(this.name);

  void log(String msg) {
    print(msg);
  }
}
```

#### 初始化列表

声明为final的成员属性，需要在构造方法时就赋值，除了使用dart提供的语法糖进行中直接赋值外，还可以使用初始化列表。

- 初始化列表的格式，在构造方法名后:final属性=值，多个用逗号分隔。

```
class Person {
  String name;
  int age;
  final String gender;

  Person(this.name, this.age, this.gender);

  //命名构造方法+初始化列表，:号后进行final属性的赋值，多个属性中间用,号分隔
  Person.withMap(Map map) : gender = map["gender"] {
    this.name = map["name"];
    this.age = map['age'];
  }

  printPerson() {
    print('name=$name, age=$age, gender=$gender');
  }
}

void main() {
  //初始化列表
  var map = Map();
  map['name'] = 'wally';
  map['age'] = 18;
  map['gender'] = 'Male';
  var person = Person.withMap(map);
  person.printPerson();
}
```

#### 静态成员

类的成员，为静态成员。dart和java的静态成员基本一致。

- 静态方法，只能访问静态成员。
- 实例方法，既能访问静态成员，也可以访问实例成员。

```
class Page {
  //静态常量
  static const int maxPage = 10;
  static int currentPage;
  int firstPage = 1;

  //静态方法能访问静态成员
  static void scrollDown() {
    currentPage = 1;
    //静态方法不能访问实例变量
    //firstPage++;
    print("scroll down -> currentPage: $currentPage");
  }

  //实例也能访问静态成员
  void scrollUp() {
    currentPage++;
    firstPage++;
    firstPage;
    print("scroll up -> currentPage:$currentPage");
  }
}

void main() {
  Page page = Page();
  Page.scrollDown();
  page.scrollUp();
}
```

#### 对象操作符

- 空安全对象操作符。如果调用一个值为null的对象的属性或方法，会抛出异常。传统方式需要判空，而dart提供了空安全的操作符，在调用处加上?即可。

```
Person person;
//?.对象操作符，在对象为null时，不会进行调用
person?.work();
//属性也可以使用?.
person?.name;
```

- as操作符。将对象强转为某个类型。如果类型强转出错则会抛出异常。

```
//没有推断出Person，类型为dynamic
var person2;
//因为类型为dynamic，可以赋值任意类型
person2 = "";
//as操作符，强转类型，这里是String类型，强转为Person类型会抛出异常。
(person2 as Person).work();
```

- is操作符，判断对象是否为某个类型的实例。是返回true，不是则返回false。一般is会配合as使用，但是通过is后，类型会自动转换，所以as操作符就不用了。而Java判断类型后还需要强转类型。

```
if (person2 is Person) {
	print('object is Person');
}

if (person2 is Person) {
	//经过了is操作符判断，就不用as操作符再转换再操作
	person2.work();
}
```

- 级联操作符

当一个类有多个set方法时，一般我们会让set方法后return返回当前对象，来做到链式操作，或者使用builder设计模式进行链式操作，而dart提供了级联操作符。格式为：对象...setXxx()...setYyy();

```
//级联操作符，每次操作后都返回自身，方便链式调用
var person3 = new Person();
person3
..name = 'wally'
..age = 19
..work();
```

#### 对象的call方法

dart中，默认一个规定，对象中存在call方法，那么输出对象时，调用的不是对象的toString()方法，而是声明的call方法。toString()方法不能传入参数，而call方法则不是，输入参数和返回值都可以自定义，只要名字为call即可。

```
void main() {
  var person = new Person();
  //call方法，输出对象默认调用对象的默认方法
  //可以带参数，有返回值，只要名称是call就可以
  String result = person('wally', 19);
  print(result);
}

class Person {
  String name;
  int age;

  String call(String name, int age) {
    return "name=$name,age=$age";
  }
}
```