#### Java8新特性之lambda表达式

Java8，自2014年3月18日发布至今，出来已经有5年时间了，Java12都已经发布，但其实在后端中，在兼容性上，一直都是Java7，到现在Java8现在才比较用得比较多。在Android中，JDK的版本更加是限制在安卓版本中，在Android7.0中，JDK版本才升级到Java8，一般我们都需要兼容到4.4一下，而4.4的JDK版本才到JDK6，所以Java8的新特性基本都没用过。但是我们依旧需要进行学习。

#### 配置环境

因为JDK的限制，我们不能使用lambda表达式，但我们又希望学习lambda表达式，AndroidStudio3.0已经可以使用兼容的lambda表达式（在JDK8一下版本使用，7甚至更
低）

- 在app的Gradle文件中，进行配置

```
android {
    defaultConfig {
        ...
        //开启jack编译
        jackOptions {
            enabled true
        }
    }
   ...
   //将编译选项设置为Java1.8
    compileOptions {
        targetCompatibility 1.8
        sourceCompatibility 1.8
    }
}
```

#### 点击事件匿名内部类，lambda改造

Android中事件监听是很常见的，例如点击事件，关闭掉当前弹窗。

```
close.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dismiss();
            }
        });
```

而使用lambda可以怎么改造呢？

```
close.setOnClickListener(v -> dismiss());
```

#### lambda的格式

- 参数名+操作，(argument)->(operation)

```
(arg1, arg2) -> {System.out.println("总和: " + arg1 + arg2)}
```

- - 单个参数

```
(String arg1) -> {System.out.println("参数" + arg1)}
```
- 单个参数，可省略括号和参数类型

```
arg1 -> {System.out.println("参数" + arg1)}
```

- 没有参数

```
() -> {System.out.println("无参调用")}
```

- 多行语句，只针对接口只有一个方法，多个方法不可以使用lambda

```
View.OnClickListener listener = (View view) -> {
      Log.d(TAG,"我被点击啦");
      Log.d(TAG,"哈喽");
}

View.OnClickListener listener = view -> {
      Log.d(TAG,"我被点击啦");
      Log.d(TAG,"哈喽");
}
```

#### 函数式接口（functional interface）

- 函数式接口，存在一个抽象方法接口。并且使用@FunctionalInterface标识，其实不使用@FunctionalInterface标识也可以，这个注解只是给编译器去识别，进行检查是否存在一个抽象方法。

#### Java8中新增的函数式接口

|接口名 | 参数 | 返回值 | 用途 |
| ------ | ------ | ------ | ------ |
| Predicate | T | boolean | 断言
| Consumer  | T | void | 消费
|Function<T,R> | T | R | 函数
| Supplier | None | T | 工厂方法
| UnaryOperator | T | T | 逻辑非
| BinaryOperator | (T,T) | T | 二元操作