# Mac下安装nacos

nacos是spring cloud alibaba推出的注册中心和配置中心，可用于替代netfix的eureka。\

- 下载nacos的压缩包，并解压

```
https://github.com/alibaba/nacos/releases
```

- 进入解压目录的bin目录下，打开终端，输入命令启动，输出`nacos is starting with standalone`即为成功

```
sh startup.sh -m standalone
```

- 进入可视化页面，账号密码都是nacos，进行登录即可，nacos的端口为8848

```
http://127.0.0.1:8848/nacos/#/login
```

- 关闭nacos

```
sh shutdown.sh
```

- 但发现关闭后，仍然能在可视化页面连接nacos，所以需要杀死8848端口的进程

```
//查询8848端口的进程，获取到进程id，例如是45025
lsof -i:8848
//杀死45025进程
kill -9 45025
```