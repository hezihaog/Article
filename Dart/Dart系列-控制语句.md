#### Dart系列-控制语句

周末学习了一下Dart语言，按照**慕课网**[Flutter开发第一步-Dart编程语言入门](https://www.imooc.com/learn/1035)教程进行学习，所以记录一下，感觉慕课网的老师辛苦做的视频教程，说得很清楚，有基础学起来很轻松也很快，本篇来学习dart的控制语句。

#### if语句

if、else if、else判断。

```
//int score = 90;
int score = 100;
if (score > 90) {
	if (score == 100) {
  		print('完美');
  	}
} else {
  print('优秀');
}
} else if (score > 60) {
	print('良好');
} else if (score == 60) {
	print('及格');
} else {
	print('不及格');
}
```

#### for语句

for循环，分为传统的for-index循环和for-in循环。

- 传统for-index

```
var list = [1, 2, 3, 4, 5];
for (var index = 0; index < list.length; index++) {
	print(list[index]);
}
```

- for-in循环

```
//for in
for (var item in list) {
	print(item);
}
```

#### while语句

while分为while循环和do-while循环。

- while循环

```
//while语句
int count = 0;
while (count < 5) {
	count++;
	print(count);
}
```

- do-whilde循环

```
do {
	count--;
	print(count);
} while (count > 0 && count < 5);
```

#### break和continue语句

- break当前循环。

```
var list = [1, 2, 3];
for (var item in list) {
	if (item == 2) {
	  break;
	}
	print(item);
}
```

- continue语句。

```
var list = [1, 2, 3];
for (var item in list) {
	//等于2的时候，跳过本次循环
	if (item == 2) {
	  continue;
	}
	print(item);
}
```

#### switch...case语句

switch...case分支和continue + 标签跳转。

- switch...case分支。

```
String language = 'Dart';
switch (language) {
	case 'Dart':
  	print('language is Dart');
	break;
case 'Java':
  	print('language is Java');
  	break;
case 'Python':
  	print('language is Python');
  	break;
default:
  	print('none');
  	break;
}
```

- continue + 标签跳转。先定义一个标签，再使用continue关键字加标签名进行跳转。

```
//continue关键字来跳转到标签位置
switch (language) {
//定义一个跳转标签
D:
case 'Dart':
  print('language is Dart');
  continue D;
case 'Java':
  print('language is Java');
  break;
case 'Python':
  print('language is Python');
  break;
default:
  print('none');
  break;
}
```

#### 总结

本篇，我们学习了dart的流程控制语句，下篇，我们将继续学习dart中的方法。