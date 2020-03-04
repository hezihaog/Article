#### Dart系列-枚举、泛型

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的枚举、泛型。

#### 枚举

在日常开发中，我们经常会需要定义多个类型的常量值，而枚举的作用，就是为了替代常量来定义一个更有语义的类型。

- 枚举通过enum关键字进行声明。
- 枚举有一个index值，从0开始。
- 枚举不能手动指定原始值。
- 枚举中不能声明方法。

dart的枚举相比Java中的枚举会简单得多，它的作用就是用来替代常量值的。

- 例如四个季节的例子，设定当前季节，通过switch判断，打印对应的季节月份范围。

```
void main() {
  var currentSeason = Season.spring;
  //index属性，从0开始
  print(currentSeason.index);

  switch (currentSeason) {
    case Season.spring:
      print('1-3月');
      break;
    case Season.summer:
      print('4-6月');
      break;
    case Season.autumn:
      print('7-9月');
      break;
    case Season.winter:
      print('10-12月');
      break;
  }
}

//枚举使用enum关键字来定义
enum Season {
  spring,
  //不能指定原始值
//  spring = 10,
  summer,
  autumn,
  winter

  //不能添加方法
//  void test() {
//
//  }
}
```

#### 泛型

泛型，前面我们讲过dynamic类型，它的作用相当于Object类型。而像List，Map这种容器类型，如果不指定泛型，都为dynamic类型，可以添加任何类型的对象进入容器，这显然是不好的。dart中的容器也支持设置泛型。

- 容器类使用泛型

```
  //var list = new List();
  var list = new List<String>();
  //加入泛型后，就不能添加其他类型
  //list.add(1);
  list.add('1');
```

- 自定义类，类上使用泛型。在类名后<Type>声明泛型。

```
class Utils<T> {
  T element;

  void put(T element) {
    this.element = element;
  }
}

void main() {
	var utils = new Utils<String>();
	//var utils = new Utils<int>();
	utils.put('1');
	//指定泛型类型后，就只能设置对应类型的数据了
	//utils.put(1);
	print(utils.element);
}
```

- 自定义类，方法上使用泛型。在方法名后使用<Type>声明泛型类型，这和Java是不一样的，Java是在最前面。

```
class Utils<T> {

  //方法泛型
  V echo<V>(V value) {
    return value;
  }
}

void main() {
	//方法泛型
	utils.echo<int>(1);
	//加上泛型，就只能传入对应泛型的类型
	//utils.echo<String>(1);
	var value = utils.echo<String>('hello');
	print(value);
}
```

#### 总结
dart系列博客，本篇是最后一篇了，dart语言融合了Java和JavaScript的语言特点和特性，会Java或JavaScript的小伙伴学习dart起来会很容易掌握，因为他们实在太像了。dart系列博客只讲了基本的语法和相关Api以及dart的语言特性、语法糖，具体还是需要在项目中实践~