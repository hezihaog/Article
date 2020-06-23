#### SpringMVC表单提交中文乱码问题

SpringMVC的form表单提交中文出现乱码，一般是因为服务端和页面的编码不一致导致的。

- jsp页面设置编码

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
```

- tomcat服务器，server.xml配置文件，添加编码

```
<Connector URIEncoding="UTF-8" port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

- web.xml中，添加编码过滤配置，可以尽量靠前配置

```
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!-- 初始化参数，指定编码 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```