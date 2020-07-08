#### SpringMVC文件上传，找不到FileItemFactory类

#### 前言

今天用SpringMVC做文件上传，启动Tomcat就报错，java.lang.NoClassDefFoundError: org/apache/commons/fileupload/FileItemFactory...

####解决方案 

看到NoClassDefFoundError这个熟悉异常，十有八九是依赖少了，马上在pom.xml上加上

```
<!-- 文件上传 -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```

#### 最终解决

结果启动还是报错，最后发现是Tomcat部署的lib目录，WEB-INF/lib，没有添加上这2个jar包，添加上，再启动即可。

有同学可能不知道在哪里加，打开Project Structure，选择导出的war包，点开WEB-INF/lib，再点击上面的+号，选择fileupload和commons-io，确定就可以了。

![添加lib](https://user-gold-cdn.xitu.io/2020/7/8/1732e798551fe24c?w=1024&h=745&f=png&s=125177)