# slf4j和log4j的关系

- slf4j是什么

Simple Logging Facade for Java。为java提供的简bai单日志Facade门面，简单点说就是Java的简单日志门面。
它是把不同的日志系统的实现进行了具体的抽象化，只提供了统一的日志使用接口，使用时只需要按照其提供的接口方法进行调用即可，

- log4j是什么

由于slf4j只是一个接口，并不是一个具体的可以直接单独使用的日志框架，所以最终日志的格式、记录级别、输出方式等都要通过接口绑定的具体的日志系统来实现。
这些具体的日志系统就有log4j,logback,java.util.logging等，它们才实现了具体的日志系统的功能。

## 简单使用

- 导入Maven依赖

```
<!-- slf4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.12</version>
</dependency>
<!-- log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

- 创建log4j.properties配置文件

```
# 日记级别(单个级别) 文件/控制台
log4j.rootLogger=debug, stdout,file

# Redirect log messages to console
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n

# Rirect log messages to a log file
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=test.log
log4j.appender.file.MaxFileSize=5MB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

- 日志API

```
public class Log4jTest {
    private static final Logger logger = LoggerFactory.getLogger(Log4jTest.class);
    
    public static void main(String[] args) {
        logger.debug("debug");
        logger.warn("warm");
        logger.error("error");
    }
}
```