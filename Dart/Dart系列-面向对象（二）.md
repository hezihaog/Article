#### Dart系列-面向对象（二）

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的面向对象（二）。

#### 继承

dart作为面向对象的语言，所以也有面向对象的三大特性，封装、继承、多态。封装就是之前的_前缀，而dart以库作为单位的，而库则是dart中的文件。接下来我们先看继承。

- 继承。使用extends关键字。格式和Java的是一样的。继承是复用的一种手段，子类继承父类，会继承父类的所有公开属性和公开方法（包括计算属性），私有的属性和方法则不会被继承。子类可以覆写父类的公开方法。

```
class Person {
  String name;
  int age;

  //私有属性，不能被访问
  String _birthday;

  //是否成年，计算属性
  bool get isAdult => age > 18;

  void run() {
    print('run -> name=$name, age=$age');
  }
}

class Student extends Person {
  void study() {
    print('study...');
  }

  //复写计算属性，加上@override注解
  @override
  bool get isAdult => age > 15;

  //复写方法
  @override
  void run() {
    print('student run...');
  }
}

void main() {
  var student = new Student();
  student.study();
  //继承Person的可见属性
  student.name = 'wally';
  student.age = 17;
  print('是否成年:${student.isAdult}');
  student.run();
}
```

- 多态。子类通过覆写覆写中的方法，在构造时，使用父类类型接收，调用父类方法则调用到了子类覆写的方法，则出现了多态。多态后，只能调用父类中公开的方法和属性，不能访问到子类的公开属性和方法了，如果要访问，则需要强转后调用。

```
//多态
Person person = new Student();
person.name = 'barry';
person.age = 19;
//多态后，只能访问父类中的属性和方法
person.run();
//不能访问子类的方法
//person.study();

//如果需要访问子类的方法，就需要强转
if (person is Student) {
	person.study();
}
```

#### 继承中的构造方法

通过继承我们复用了父类的属性和方法，而构造方法，也会被复用，但和Java的有些许不同。

- 子类默认继承父类中的无参构造方法。

```
class Person {
  String name;
  
  //重写无参构造方法
  Person() {
    print("person...");
  }
}

class Student extends Person {
}
```

- 如果父类没有无参构造方法，只有有参构造或命名构造放方法，则子类需要显示调用父类的有参构造方法或者命名构造方法，还可以结合初始化列表进行使用。

```
class Person {
  String name;
  
  //如果父类没有无参构造方法，只有有参构造或命名构造放方法，则子类需要显示调用父类的有参构造方法或者命名构造方法
  Person(this.name);

  Person.withName(this.name);
}

class Student extends Person {
	final String gender;
	//有参构造方法，调用父类的有参构造方法
	Student(String name, String gender)
      : gender = gender,
        super(name);
        
	//子类中的命名构造方法也是一样
	Student.withName(this.name, this.gender)
        super.withName(name);

	//如果gender在初始化列表中初始化也是可以的
Student.withName(String name) : super.withName(name);
  Student.withName(String name, String g)
      //初始化列表
      : gender = g,
        super.withName(name);
}

```

#### 抽象类

1. 抽象类，和Java一样，使用abstract关键字声明为抽象类。
2. 抽象类中声明抽象方法，子类必须实现，而Java的抽象方法必须使用abstract关键字修饰，而dart则不需要，只要方法只有方法声明，没有实现即为抽象方法。
3. 抽象类可以有抽象方法和实例方法，甚至可以没有抽象方法，这个和Java是一致的。
4. 抽象类不能被实例化，只有子类可以。

```
//abstract修饰类，该类则为抽象类
//抽象类可以没有抽象方法
//没有实现的方法的类，必须是抽象类
abstract class Person {
  //没有实现的方法就是抽象方法，不需要加abstract
  void run();
}

class Student extends Person {
  //复写父类的抽象方法
  @override
  void run() {
    print('run...');
  }
}

void main() {
  //Person抽象类，不能实例化
  //Person person = new Person();
  Person person = new Student();
  person.run();
}
```

#### 接口

和Java一样，dart也有接口，但是和Java还是有区别的。

1. 首先，dart的接口没有interface关键字定义接口，而是普通类或抽象类都可以作为接口被实现。
2. 同样使用implements关键字进行实现。
3. 但是dart的接口有点奇怪，如果实现的类是普通类，会将普通类和抽象中的属性的方法全部需要覆写一遍。而因为抽象类可以定义抽象方法，普通类不可以，所以一般如果要实现像Java接口那样的方式，一般会使用抽象类。

```
class Person {
  String name;

  //计算属性
  int get age => 18;

  void run() {
    print('Person run...');
  }
}

//dart的接口，类也可以作为接口，使用implements关键字进行实现，需要将所有属性和方法都实现...
class Student implements Person {
  @override
  String name;

  @override
  int get age => null;

  @override
  void run() {

  }
}
```

- 一般会使用抽象类进行实现。

```
//dart的接口有点奇怪，所以一般会将抽象类作为接口来使用
abstract class Person {
  void run();
}

class Student implements Person {
  @override
  void run() {
    print('student...run...');
  }
}

var student = new Student();
student.run();
```

#### Mixins

Mixins，中文翻译过来为混入。dart和Java等高级语言一样，没有多继承，只有单继承，而dart解决多继承的问题，实现的是Mixins，而Java为接口多实现。

- 要使用Mixins，必须让类继承一个类（Object不算），再使用with关键字将多个类标识为被混入的类。

```
void main() {
  D d = new D();
  d.a();
  d.b();
  d.c();
}

class A {
  void a() {
    print('A.a()...');
  }
}

class B {
  void b() {
    print('B.b()...');
  }
}

class Test {}

//作为mixins的类，只能继承Object
//class C extends Test {
class C {
  //作为mixins的类，不能有构造方法
//  C() {
//
//  }

  void c() {
    print('C.c()...');
  }
}

//mixins，需要先继承一个类，再机上with关键字去混入其他类
//如果混入的类中的方法，多个类中都存在，那么该类继承的的方法为最后一个混入的类
class D extends A with B, C {}
```

1. 然而，被混入的类，不能有构造方法。
2. 被混入的类，不能继承其他类，只能继承Object。
3. 如果多个混入的类中都有同一个方法声明，则宿主类会取最后一个混入的类的方法，所以是按with关键字后写的最后一个类中的方法，是按顺序来的。

#### 汽车例子，使用混入，实现组合功能

- 一辆汽车，有多则部件组成，例如引擎和轮胎，而引擎有电动引擎或油驱动引擎。

```
//引擎抽象类
abstract class Engine {
  //有一个运行的抽象方法
  void work();
}

//油引擎
class OilEngine implements Engine {
  @override
  void work() {
    print('work with Old');
  }
}

//电动引擎
class ElectricEngine implements Engine {
  @override
  void work() {
    print('work with Electric');
  }
}

//轮胎
class Tyre {
  String name;

  //一个跑的方法
  void run() {
  }
}
```

- 声明混入，前面说到mixins需要强制继承一个类后才能通过with关键字进行混入其他类。其实还有将extends关键字换为=号的简写方法。

```
//mixins简写
//class Car = Tyre with ElectricEngine {}

//完整写法
class Car extends Tyre with ElectricEngine {
	//新能源电动车
}

//其实就是组合的方式
class Bus extends Tyre with OilEngine {
	//传统油驱动大巴士
}
```

#### 操作符覆写

- 在比较2个类时，在Java中，我们会使用equas()方法，而dart比较内容是使用==号，Java中==号是比较对象的内存地址。当我们需要比较2个Person对象的年龄时，Java中需要取出age属性再通过比较运算符进行比较。而dart则提供了更强大的，操作符覆写特性。通过operator关键字，覆写指定的操作符。

- 例如比较2个Person类对象的age年龄，使用>号操作符，通过operator覆写，则不需要将age属性取出再进行比较运算符比较。

```
class Person {
  int age;

  Person(this.age);

  //复写>操作符，使用operator关键字
  bool operator >(Person person) {
    return this.age > person.age;
  }
}

//Person person1 = new Person(18);
Person person1 = new Person(20);
Person person2 = new Person(20);
print(person1 > person2);
```

- 再例如我们在map中使用的[]取值符号，在对象中也可以进行复写，从而能将类属性用字符串的方式取出再返回。

```
class Person {
  int age;

  Person(this.age);

  //复写>操作符，使用operator关键字
  bool operator >(Person person) {
    return this.age > person.age;
  }

  //重写取值[]操作符
  int operator [](String str) {
    if (str == 'age') {
      return age;
    }
    return 0;
  }
}

Person person1 = new Person(18);
//取出age字段的值
print(person1['age']);
//没有name字段，返回0
print(person1['name']);
```

- 还可以复写==比较是否相等运算符，==在dart中为比较内容，而我们的自定义类中比较内容是需要我们去自定义的，所以就可以覆写==运算符。并且同时复写hashCode()方法。

```
class Person {
  int age;

  Person(this.age);

  //复写==比较操作符
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Person && runtimeType == other.runtimeType && age == other.age;

  @override
  int get hashCode => age.hashCode;
}

Person person1 = new Person(20);
Person person2 = new Person(20);
//年龄相同时，为相等
print(person1 == person2);
```

#### 总结

本篇我们学习了面向对象（二），学习到了继承、构造方法继承、抽象类、接口、Mixins混入以及操作符复写，下一篇，我们将进行学习枚举和泛型。