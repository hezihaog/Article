#### Dart系列-方法

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的方法。

函数，或者叫方法，在dart中也是一种对象，所以他也可以作为一个变量，它的类型为Function，这个和Java是不同的。

#### 方法定义

dart中的方法定义。

- 基本格式。

```
返回值 方法名(方法参数) {
   return 方法体
}
```

- 无参，无返回值方法

```
void printPerson() {
	print('hello person');
}
```

- 无参，带返回值方法

```
String printPerson() {
	return 'hello person';
}
```

- 有参，无返回值方法

```
void printPerson(String name) {
	print('name=$name');
}
```

- 有参、有返回值方法

```
String printPerson(String name) {
	return 'hello $name person';
}
```

- 其中，如果没有返回值，返回值是可以省略的。
- 并且，参数类型是可以省略的。

```
printPerson(name, age) {
  print('name=$name,age=$age');
}
```

#### 可选参数

可选参数，Java我们使用重载进行参数的可选，而dart中则有可选参数。

- 方式一：使用一对{}号，包裹参数声明默认值。并且设置值时，需要写明参数名，格式为参数名:值。

```
//可选参数，需要用{}括起来
printPerson(String name, {int age, String gender}) {
  print('name=$name,age=$age, gender=$gender');
}

printPerson('李四');
//调用时，需要传可选参数，参数需要参数名:+值来传入
printPerson('李四', age: 18);
printPerson('李四', age: 18, gender: 'Male');
//因为有参数名标志，所以参数不同位置都可以
printPerson('李四', gender: 'Male', age: 18);
```

- 方式二：使用一对[]括起来。这种方式声明，参数传递不需要标明参数名，但需要按位置设置。参数是不能乱的。

```
//另外一种方式，用[]括起来
printPerson2(String name, [int age, String gender]) {
  print('name=$name,age=$age, gender=$gender');
}

//调用[]方式的方法，就是按位置来的，不需要用参数名:+值
printPerson2('张三');
printPerson2('张三', 20);
printPerson2('张三', 20, 'Male');
//[]的方式只能按位置来，传入不能乱
//printPerson2('张三', 'Male', 20);
```

#### 默认参数值

同样，方法参数的默认值，Java一般是使用重载实现，而dart不支持方法重载，它使用参数默认值来进行方法参数的默认值使用。

- 默认参数，在参数的后面用=值来设置。

```
printPerson(String name, {int age = 30, String gender = 'Female'}) {
  print('name=$name,age=$age, gender=$gender');
}

//默认参数
printPerson('李四');
//调用时，使用了方式一的可选参数，所以参数需要参数名:+值来传入。
printPerson('李四', age: 18);
printPerson('李四', age: 18, gender: 'Male');
```

#### 匿名方法

匿名方法，上面说到方法也是一种对象，他的类型为Function。声明方法除了上面的在类中显示声明外，还可以匿名声明，就是匿名方法。

- 声明

```
//匿名方法声明，和显示声明方法一样，也需要返回值、方法名、参数。
//它的格式为：
返回值类型 方法名(参数)
```

- 匿名方法的调用

```
//匿名方法，可以将方法赋值给一个变量
var fun = (str) {
print('Hello$str');
};
//调用
fun(30);
```

- 简写方式（不推荐使用，不好区分）

```
//也可以用一对()号来包裹，最后使用()来调用，不推荐使用，不好区分
(() {
print('Hello Dart');
})();
```

- 匿名方法使用

```
//匿名方法使用
List<String> list2 = ['h', 'e', 'l', 'l', 'o'];
//传入一个匿名方法
var result = listTimes(list2, (str) {
	//每个元素复制3次，再返回
	return str * 3;
});
print(result);

//需要传入一个fun方法，每次循环时传入元素，并获取元素设置到元素中
List listTimes(List list, String fun(str)) {
  for (var index = 0; index < list.length; index++) {
    list[index] = fun(list[index]);
  }
  return list;
}
```

- 或者将匿名方法，定义在内部，再调用

```
List<String> list2 = ['h', 'e', 'l', 'l', 'o'];
print(listTimes2(list2));
List listTimes2(List list) {
  //直接内部定义一个方法来使用
  var fun = (str) {
    return str * 3;
  };
  for (var index = 0; index < list.length; index++) {
    list[index] = fun(list[index]);
  }
  return list;
}
```

#### 闭包

闭包，就是方法中的方法，再作为返回值返回出去。使用闭包一般是为了方法内的方法能访问方法内的变量，作为返回值返回后，就能访问到方法中的变量了。

- 例如，需要在外部访问到方法a()中的count变量，就可以在a()方法内定义一个printCount()方法，将count值自增，再作为返回值返回。最后main()方法中获取，再调用，即可操作a()方法中的count值。

```
a() {
  int count = 0;
  //内部定义一个方法，count变量会被传递到方法内
//  printCount() {
//    print(count++);
//  }

  //将方法返回出去
  //return printCount;

  //也可以直接匿名方法，这种会用得比较多
  return () {
    print(count++);
  };
}

void main() {
  //闭包
  //a()方法返回内部的一个方法，内部的方法抓住了count变量，然后返回这个内部方法
  //内部方法是一个对象，所以每次调用，都会将内部变量进行+1
  var func = a();
  //每次调用，count值都会+1
  func();
  func();
  func();
  func();
}
```