#### 解决Maven依赖Oracle驱动ojdbc14找不到的问题

- 错误原因

Oracle的驱动，在Maven中依赖，会报错说找不到，所以需要手动下载jar包，再同步到本地的仓库。
再次同步Maven的依赖就可以了

```
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc14</artifactId>
    <version>10.2.0.4.0</version>
</dependency>
```

- 解决方案

先手动下载jar包，在jar包的目录下开启终端，执行以下命令，出现build success，就为成功了！

```
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc14 -Dversion=10.2.0.4.0 -Dpackaging=jar -Dfile=ojdbc14.jar
```

![](https://user-gold-cdn.xitu.io/2020/7/5/1731c7b9635203e5?w=570&h=452&f=png&s=63229)

- 同步到的本地Maven仓库目录

本地的仓库一般在用户目录下的.m2目录下，里面有一个repository子目录，再进去就是下载的各种jar包的包名的根目录
我的是Mac系统，仓库的目录是：（xxx是你的用户名）

```
/Users/xxx/.m2/repository/com/oracle/ojdbc14/10.2.0.4.0
```