#### Dart系列-数据类型

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的数据类型。

#### 变量和常量

写程序，变量和常量可以说是代码的组成单元之一了。来看下Dart是怎么定义变量和常量的吧~

- 变量

dart声明变量。

- 使用var关键字。
	- 同样也是可以先声明后赋值或声明并赋值。
	- 如果一开始声明时，不能推断类型（没有直接赋值），则类型为dynamic，可以赋值任何类型，dynamic是什么呢？后面下面讲~

```
//先声明，但不赋值，则为null
var a;
print(a);
//后续赋值
a = 10;
print(a);

//重复赋值，类型可以不一致
a = "Hello Dart!";
print(a);

var b = 20;
//已经推断出类型，就不能赋值其他类型了
b = 'Hello';
print(b);
```

- 常量
	- final关键字。声明的常量为只能赋值一次的变量。
	- const关键字。声明的常量为编译时常量，在编译器就决定了。也只能赋值一次。

```
final c = 30;
//final修饰的变量，只能设置一次
//c = 50;

const d = 20;
//const修饰为常量，表示编译时常量，只能赋值一次
//d = 50;
```

#### 数据类型-简介

任何语言，都会有数据类型，本篇来快速过一下Dart语言的数据类型~数据类型主要有Number、String、Bool、List、Map以及dynamic类型。

**注意：**在dart中，所有的类型都是对象，没有基础数据类型（自然没有装拆箱）。

#### 数据类型-数值型

dart中，数值型数据类型有int和double，他们的父类是num类型。

- num类型
	- int类型，整形，只能存储整数。
	- double类型，浮点型，能存储小数和整数（也是转换为有小数点的浮点数）。

```
//num类型为int和double的父类
num a = 10;
a = 20;
//所以num类型可以赋值整形或浮点型都可以
a = 20.5;

//但是如果确定了是子类类型，就不能了，int类型不能赋值浮点型
int b = 20;
//b = 20.5;

//由于浮点型包含整形，所以是可以赋值的
double c = 10.5;
c = 20;
print(c);
```

#### 数值型的一些操作。
- 算术运算。

```
//加、减、乘、除、取余
print(b + c);
print(b - c);
print(b * c);
print(b / c);
print(b % c);
	
//dart中，还有取整运算，使用~/
//~/代表取整
int result = b ~/ c;
print(result);
```
	
- 类型中定义的一些常用方法

```
//是否是数字
print(0.0 / 0.0);
	
//是否是偶数，是
print(b.isEven);
//是否是奇数，不是
print(b.isOdd);
	
//求绝对值
int e = -100;
print(e.abs());
	
double f = 10.5;
//四舍五入
print(f.round());
//向上取整
print(f.ceil());
//向下取整
print(f.floor());
	
//类型转换
print(f.toInt());
  
int d = 11;
print(d.toDouble());
```

#### 数据类型-字符型

在dart中，字符串和字符都为String类型，没有char类型。

- 声明，普通声明、多行字符声明、原始字符串raw声明

```
//普通声明
String str1 = 'Hello'; //双引号也可以""
print(str1);

//多行字符串，使用3个单引号
String str2 = '''Hello
            Dart''';
print(str2);

//存在转义字符
String str3 = 'Hello \n Dard';

//原始字符串，不进行转义，前面加r
str3 = r'Hello \n Dard';
print(str3);
```

- 字符串的拼接、重复复制（dart特有）、取出字符串中字符

```
//字符串拼接
String str4 = "Hello Dart";
print(str4 + 'New');

//重复复制字符串
//输出：Hello DartHello DartHello DartHello DartHello Dart
print(str4 * 5);

//比较字符串内容是否相等
print(str3 == str4);

//角标取出字符串中的字符
print(str4[0]);
```

- 插值表达式

```
//插值表达式
int a = 1;
int b = 2;

//将表达式嵌入到字符串中
print('a + b = ${a + b}');

//直接取值嵌入到字符串中
print('a = $a');
```

- 字符串的常用操作-字符串长度

```
//取出字符串长度
print(str4.length);
```

- 字符串的常用操作-字符串判空

```
//判断字符串是否为空
print(str4.isEmpty);
//判断字符串是否不为空
print(str4.isNotEmpty);
```

- 字符串的常用操作-字符串包含

```
//判断字符串是否包含另外一个字符串
print(str4.contains('Hello'));
```

- 字符串的常用操作-字符串截取

```
//截取字符串
print(str4.substring(0, 3));
```

- 字符串的常用操作-字符串判断开头、结尾

```
//判断字符串是否以某个字符串开头
print(str4.startsWith('H'));

//判断字符串是否以某个字符串结尾
print(str4.endsWith('Dart'));
```

- 字符串的常用操作-字符串查找子串位置

```
//取字符串在指定字符串中的角标位置
print(str4.indexOf('D'));
//从后面开始找
print(str4.lastIndexOf('t'));
```

- 字符串的常用操作-字符串大小写转换

```
//字符串转大写
print(str4.toUpperCase());
//字符串转小写
print(str4.toLowerCase());
```

- 字符串的常用操作-字符串去除空格

```
//去除字符串前后空格
String str5 = ' Hello ';
print(str5.trim());
//只去除前面的空格
print(str5.trimLeft());
//只去除后面的空格
print(str5.trimRight());
```

- 字符串的常用操作-字符串分割

```
String str6 = "wally,barry,rose";
//字符串分割
List<String> list = str6.split(',');
print(list);
```

- 字符串的常用操作-字符串替换

```
//替换所有
print(str5.replaceAll('l', 'a'));
//替换第一个
print(str5.replaceFirst('l', 'a'));
```

#### 数据类型-布尔型

布尔型，true或者false。

```
//布尔值，只有true和false
bool isTrue = true;
bool isFalse = false;
print('Hello'.isNotEmpty);
```

#### 数据类型-列表List

存储数据的列表容器list。没有数组类型。

- list声明

```
//声明列表
var list1 = [1, 2, 3, 4, 'Dart', true];
print(list1);

//也可以用new关键字
list1 = new List();
```

- 获取list中的值和修改元素

```
//获取数组中的元素
print(list1[0]);

//修改元素的值
list1[1] = 'Hello';
print(list1);
```

- 不可变list

```
//不可变list
var list2 = const [1, 2, 3];
print(list2);
//不可变list不能修改其内容，编译会抛出异常
list2[0] = 100;
```

- 获取list的长度

```
//获取一个list的长度
var list4 = ['Hello', 'Dart'];
print(list4.length);
```

- list添加元素

```
//给list添加元素
list4.add('New');
print(list4);
//指定位置添加元素
list4.insert(1, 'love');
print(list4);
```

- 移除元素

```
//移除某个元素
list4.remove('love');
print(list4);
//按角标移除元素
print(list4.removeAt(0));
```

- 元素查找

```
//查找元素位置，找不到返回-1
print(list4.indexOf('Dart'));
//反向查找
print(list4.lastIndexOf('Hello'));
```

- list排序

```
list4.sort();
print(list4);
```

- 截取list

```
//截取list，从第二个元素开始
print(list4.sublist(1));
```

- 打乱list

```
//打乱
print(list4.shuffle());
```

- 转换为map

```
//转换为键值对Map，会以list中的角标作为map的key，值作为map的value
print(list4.asMap());
```

- 遍历

```
//遍历
list4.forEach(print);
```

#### 数据类型-键值对Map

Map键值对类型，开发也是非常常见的~

- 声明

```
//声明并赋值Map
var map1 = {'first': 'dart', 1: true, true: 2};
//也可以用new关键字声明
map1 = new Map();
```

- 取出map中的元素

```
//按key取值
print(map1['first']);
print(map1[true]);
```

- 修改map中的元素

```
//修改map的值
map1[1] = false;
print(map1);
```

- 不可变map

```
//不可变map
var map2 = const {1: 'Java', 2: 'Dart'};
print(map2);
//修改就会编译不过
map2[0] = 'Python';
```

- 获取map的长度

```
//获取map的长度
var map = {'first': 'Dart', 'second': 'Java', 'third': 'Python'};
print(map.length);
```

- 判断map是否为空、不为空

```
//判断是否为空
print(map.isEmpty);
//判断是否不为空
print(map.isNotEmpty);
```

- 获取map的key列表和value列表

```
//获取全部key
print(map.keys);
//获取全部value
print(map.values);
```

- 判断是否包含某个key和某个value

```
//判断是否包含某个key和某个value
print(map.containsKey('first'));
print(map.containsValue('C'));
```

- 移除某个元素

```
//移除某个元素
map.remove('third');
print(map);
```

- 遍历

```
//遍历，传入一个方法进行遍历
map.forEach(f);

void f(key, value) {
  print('key=$key, value=$value');
}
```

#### 数据类型-特殊的dynamic

终于到我们的dynamic类型了，这种类型可以被任何类型赋值，其实说白了就是相当于Java的Object类型，dart中则为dynamic类型。

- 赋值示例

```
//var 这时候赋值给任何类型都可以
var a;
a = 10;
a = 'Dart';

//其实类型就是dynamic，就是动态的
dynamic b = 20;
b = "JavaScript";
```

- 泛型中使用

```
//list的泛型中传入dynamic，就能存放任何类型
var list = new List<dynamic>();
list.add(1);
list.add('hello');
list.add(true);
print(list);
```

#### 总结

本篇，我们学习了dart中的数据类型，以及他们的常用Api。下篇我们来学习**dart中的运算符**。