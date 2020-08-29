# MyBatisPlus 配置错误异常 MalformedByteSequenceException: 2 字节的 UTF-8 序列的字节 2 无效

按照之前学习MyBatisPlus的记录，配置好后，启动却报错了：`MalformedByteSequenceException: 2 字节的 UTF-8 序列的字节 2 无效`。

## 解决方案

网上搜索，都是说是文件编码不对引起的，将文件保存为UTF-8即可，但我所有文件都是UTF-8的，各种清理缓存，重启也不行。

最后发现是MyBatisPlus的`mapper-locations`的配置问题！

- 错误的配置

mapper目录下使用*通配符，但是这里有问题，如果java包下的mapper类也是这个名字，启动时进行依赖注入，就会找到2份，一份是`java`包下的，一份是`resources`下的。

因为启动时，需要找到xml文件，而这样配置，会找到`java`包下的class文件，xml读取自然失败，才会抛出异常：`MalformedByteSequenceException: 2 字节的 UTF-8 序列的字节 2 无效`。

```
mybatis-plus:
  #配置mapper文件的位置
  mapper-locations:
    - classpath*:com/itheima/hchat/mapper/*
```

- 正确的配置

需要明确文件的后缀，指定为xml即可。这样`java`包下的`.class`文件，就会被选中，只会找到`resoures`下的了。

```
mybatis-plus:
  #配置mapper文件的位置
  mapper-locations:
    - classpath*:com/itheima/hchat/mapper/*.xml
```

## 之前的项目为什么可以

之前的项目，`java`包下的类文件的包名是dao，而`resoures`包下的文件夹名是`mapper`，没有重名，所以即使配置错了，也只会找到一份，所以才没有报错。