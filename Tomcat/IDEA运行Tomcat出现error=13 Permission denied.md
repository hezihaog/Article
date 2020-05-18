#### IDEA运行Tomcat出现error=13 Permission denied

IntelliJ IDEA运行Tomcat，出现error=13 Permission denied，原因是文件的权限不足。

- 解决方案

找到Tomcat目录的bin目录，执行以下命令

```
chmod a+x catalina.sh
```