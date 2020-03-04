#### 重拾Java第四篇-String类
- String类，即为字符串，日常开发中非常常用，我们这篇来学习字符串的处理

-----

- 不可变String
    * String类其实是一个final类，就是说它是不可以被继承的~
    * 怎样才是不可变呢？当String和另外一个String进行拼接时，结果字符串不是原先的字符串对象而是重新创建的~
  
- 字符串新建
    * 字面量，直接双引号将字符串括起来，这种创建方式叫字面量
  
```
String str = "Wally";
```

* new的方式，使用new关键字创建字符串对象

```
String str = new String("Wally");
```

- 他们的区别？ 
  * 字面量的方式，对象是创建在字符串常量池的，所以它是复用的，当多个字符串对象的内容是一致时，对象的地址是一个。
  * 而new方式创建的字符串对象是创建在堆内存的，每次的new都是一个新的对象，不会复用
    
- 字符串操作

- 字符串拼接

    * +号进行拼接
      
    ```
    //结果为：loveyou
    String result = "love" + "you";
    ```
    
    * 使用String类的concat()方法 
        
    ```
    //结果为：loveyou
    String result = "love".concat("you");
    ```
    
    * 字符串查找
    
  -     判断是否包含指定前缀 
  
  ```
  String str = "love you";
  str.startsWith("love");
  ```
  
  - 判断是否包含指定后缀 
  
  ```
  String str = "love you";
  str.endsWith("you"); 
  ```
  
  - 判断是否包含指定字符串 
  ```
  String str = "love you";
  str.contains("y");
  ```
  
  - 获取字符串长度 
  ```
  String str = "love you";
  str.length();
  ```
  
  - 查找字符串在字符串中的位置
  * 从前面向后面开始查找
  ```
  String str = "love you";
  str.indexOf("y");
  ```
  
  * 从后面向前开始查找
  ```
  String str = "love you";
  str.lastIndexOf("y");
  ```
  
  - 字符串替换
  * 单个替换（多个只替换一次）
  ```
  String str = "love you";
  str.replace("love", "very love");
  ```
  - 全部替换，多个出现都会替换掉
  
  ```
  String str = "love you, i love you";
  str.replaceAll("love", "very love");
  ```
  
  - 字符串截取 
  ```
  String str = "love you, i love you";
  //指定开始角标和结束角标进行截取，从一个字截取到第三个字
  //结果为：e you,i love you
  String result = str.substring(0, 2);
  ```
  
- 可变字符串（不是继承于String喔）
* 当需要大量字符串要拼接时，由于String的不可变性，每次拼接都会重新创建一个String对象进行返回，但每次对象的创建都需要耗费内存空间，由此可见直接使用+号和concat()进行拼接是比较耗费内存的

  - 解决方案
    * StringBuilder
    * StringBuffer
    
- StringBuilder，最常用的可变字符串类
- StringBuffer，其实就是StringBuilder的多线程加锁版本，在多线程操作时则需要使用StringBuffer。平时单线程操作字符串使用StringBuilder即可，因为没有了加锁操作，会减少不必要的加锁性能消耗

```
StringBuilder builder = new StringBuilder();
for (int i = 0; i < 15; i++) {
    //使用append方法进行拼接字符串
    builder.append("love: ");
    builder.append(i);
}
//最后toString输出最终结果s
String result = builder.toString();
```

- 清空所有已拼接的字符串，当拼接很频繁时，频繁创建StringBuilder对象也不合理，我们可以使用delete方法，清空掉所有已拼接的字符来复用StringBuilder对象
```
builder.delete(0, builder.length());
```