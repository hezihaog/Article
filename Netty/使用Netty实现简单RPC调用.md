#### 使用Netty实现简单RPC调用

说到RPC，一般在微服务的之间调用会使用，因为多个服务之间不再像本地接口直接调用那么直接，需要用到HTTP或其他协议通信。
如果用过SpringCloud的Fegin，你就会知道，虽然远程调用，但是代码写起来还是和本地接口调用一样。

而Netty是高性能的网络通信框架，我们可以让每个服务都成为Netty的服务器端，让其他服务调用方，也通过Netty来调用。

如果直接书写这些代码，每个接口调用就会掺杂网络通信的代码，那我们可以按本地接口调用那样实现吗？

答案是可以的，我们需要用到以下技术
    - 动态代理
    - Netty网络通信
    - 反射

大概实现过，使用动态代理，代理服务API接口，当调用生成的代理方法时，在动态代理的回调中，获取调用方法的方法名、参数、返回值，再使用Netty进行通信，传递参数到对方的服务中，再使用反射调用本地的实现类，获取到返回值后，再通过Netty发送结果回去。

一般服务会做集群多实例，多个服务需要注册到注册中心，保存服务的IP、端口号等信息，以及心跳续租，提供服务列表给服务来调用。

本篇不提供注册中心功能，可在服务调用之前获取到，传给RPCProxy进行服务连接。如不传地址和端口号，默认使用127.0.0.1本地ip和端口号9999，来获取服务。

#### 项目详情

用Maven生成3个Module和一个netty_rpc的父工程

- provider，服务提供方，依赖consumer模块，只使用到它的接口和模型类
- consumer，服务消费方
- 封装的lib模块，被provider和consumer模块依赖
- netty_rpc父工程，负责管理依赖

#### netty_rpc父工程

pom.xml引入2个依赖，分别是netty和一个反射库

```
<dependencies>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>${netty.version}</version>
        <optional>true</optional>
    </dependency>

    <dependency>
        <groupId>org.reflections</groupId>
        <artifactId>reflections</artifactId>
        <version>${reflections.version}</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

#### 调用方的代码

先放一下调用的代码，远程调用都隐藏到了RPCProxy类中，调用和处理方法就和本地接口调用一样。

```
/**
 * 客户端入口类
 */
public class ConsumerMain {
    public static void main(String[] args) {
        //远程调用，获取热门书籍
        BookService bookService = RPCProxy.create(BookService.class);
        Book hotBook = bookService.getHotBook();
        System.out.println("远程RPC调用获取热门书籍：" + hotBook);

        //远程调用，保存书籍
        Book book = new Book();
        book.setId(1);
        book.setName("MySQL必知必会");
        book.setAuthor("MySQL");
        System.out.println("保存书籍：" + book);
        boolean isSuccess = bookService.saveBook(book);
        if (isSuccess) {
            System.out.println("书籍保存 => 成功");
        } else {
            System.out.println("书籍保存 => 失败");
        }
    }
}
```

#### 提供方的代码

我们的服务提供方是一个图书服务，提供获取热门书籍和保存书籍的功能

- Book实体类

```
/**
 * 书籍实体
 */
public class Book implements Serializable {
    /**
     * 书籍Id
     */
    private Integer id;
    /**
     * 书籍名称
     */
    private String name;
    /**
     * 书籍作者
     */
    private String author;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}
```

- 图书服务和实现

```
/**
 * 书籍服务
 */
public interface BookService {
    /**
     * 获取一本热门书籍信息
     */
    Book getHotBook();

    /**
     * 保存一本书籍
     */
    boolean saveBook(Book book);
}

/**
 * 书籍热门服务实现
 */
public class BookServiceImpl implements BookService {
    public Book getHotBook() {
        Book book = new Book();
        book.setId(1);
        book.setName("Java入门到放弃");
        book.setAuthor("Java布道师");
        return book;
    }

    public boolean saveBook(Book book) {
        System.out.println("保存书籍成功：" + book);
        return true;
    }
}
```

- ProviderServer，提供者的Netty服务

```
/**
 * 提供者服务
 */
public class ProviderServer {
    public static void main(String[] args) {
        //创建并开启远程RPC服务
        RPCServer rpcServer = new RPCServer(9999);
        rpcServer.start();
    }
}
```

#### 封装公共类库

rpc_lib主要是存放封装rpc功能的相关类。封装分为2类，client(调用方)和server（API提供方）。

- client包
	- RPCProxy，获取远程服务的门面类，提供create()方法来获取远程方法接口的实例
	- ResultHandler，处理远程方法调用结果的处理类，外部不需要接触，被RPCProxy使用

- server包
	- CallInfo，调用的远程信息，封装方法名、形参、返回值等信息
	- InvokeHandler，收到远程调用后，反射调用本地方法的处理类
	- RPCServer，使用Netty作为服务器端的类，每个服务提供方，都要有一个类继承该类，调用start()方法来启动服务

#### 服务调用方的封装

- CallInfo

```
/**
 * 封装远程调用方法的方法名、参数、返回值等信息的实体
 */
public class CallInfo implements Serializable {
    private static final long serialVersionUID = 1L;

    /**
     * 类名
     */
    private String className;
    /**
     * 方法名
     */
    private String methodName;
    /**
     * 参数类型
     */
    private Class<?>[] types;
    /**
     * 参数列表
     */
    private Object[] objects;

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public String getMethodName() {
        return methodName;
    }

    public void setMethodName(String methodName) {
        this.methodName = methodName;
    }

    public Class<?>[] getTypes() {
        return types;
    }

    public void setTypes(Class<?>[] types) {
        this.types = types;
    }

    public Object[] getObjects() {
        return objects;
    }

    public void setObjects(Object[] objects) {
        this.objects = objects;
    }

    @Override
    public String toString() {
        return "ClassInfo{" +
                "className='" + className + '\'' +
                ", methodName='" + methodName + '\'' +
                ", types=" + Arrays.toString(types) +
                ", objects=" + Arrays.toString(objects) +
                '}';
    }
}
```

- RPCProxy，门面类，提供获取接口实例的方法

```
/**
 * RPC远程调用的代理生成类
 */
public class RPCProxy {
    public static <T> T create(Class<T> target) {
        return (T) proxy(target, "127.0.0.1", 9999);
    }

    public static <T> T create(Class<T> target, String host, int port) {
        return (T) proxy(target, host, port);
    }

    /**
     * 根据接口创建代理对象
     *
     * @param target 要获取的服务接口
     * @param host   服务提供的ip地址
     * @param port   服务提供的端口号
     */
    private static Object proxy(Class<?> target, String host, int port) {
        return Proxy.newProxyInstance(target.getClassLoader(),
                new Class[]{target}, new RemoteMethodCall(target, host, port));
    }

    private static class RemoteMethodCall implements InvocationHandler {
        /**
         * 接口的Class
         */
        private final Class<?> targetClass;
        /**
         * 远程服务的地址
         */
        private final String host;
        /**
         * 远程服务的端口号
         */
        private final int port;

        public RemoteMethodCall(Class<?> targetClass, String host, int port) {
            this.targetClass = targetClass;
            this.host = host;
            this.port = port;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            //封装调用信息到ClassInfo
            CallInfo callInfo = new CallInfo();
            callInfo.setClassName(targetClass.getName());
            callInfo.setMethodName(method.getName());
            callInfo.setObjects(args);
            callInfo.setTypes(method.getParameterTypes());
            //使用Netty发送数据到接口提供方
            EventLoopGroup group = new NioEventLoopGroup();
            ResultHandler resultHandler = new ResultHandler();
            try {
                Bootstrap bootstrap = new Bootstrap();
                bootstrap.group(group)
                        .channel(NioSocketChannel.class)
                        .handler(new ChannelInitializer<SocketChannel>() {
                            @Override
                            public void initChannel(SocketChannel ch) throws Exception {
                                ChannelPipeline pipeline = ch.pipeline();
                                //编码器
                                pipeline.addLast("encoder", new ObjectEncoder());
                                //解码器  构造方法第一个参数设置二进制数据的最大字节数  第二个参数设置具体使用哪个类解析器
                                pipeline.addLast("decoder", new ObjectDecoder(Integer.MAX_VALUE, ClassResolvers.cacheDisabled(null)));
                                //客户端业务处理类
                                pipeline.addLast("handler", resultHandler);
                            }
                        });
                ChannelFuture future = bootstrap.connect(host, port).sync();
                future.channel().writeAndFlush(callInfo).sync();
                future.channel().closeFuture().sync();
            } finally {
                group.shutdownGracefully();
            }
            return resultHandler.getResponse();
        }
    }
}
```

- ResultHandler，处理远程方法回调结果

```
/**
 * 远程方法的回调结果处理类
 */
public class ResultHandler extends ChannelInboundHandlerAdapter {
    /**
     * 远程调用的返回值结果
     */
    private Object response;

    /**
     * 读取服务器端返回的数据(远程调用的结果)
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object value) throws Exception {
        response = value;
        ctx.close();
    }

    public Object getResponse() {
        return response;
    }
}
```

#### 服务提供封装

- InvokeHandler，其实是Netty消息的回调，当收到远程调用消息时，再反射调用本地方法

```
/**
 * 收到远程调用后，反射调用本地方法
 */
public class InvokeHandler extends ChannelInboundHandlerAdapter {
    //得到某接口下某个实现类的名字
    private String getImplClassName(CallInfo callInfo) throws Exception {
        String className = callInfo.getClassName();
        int lastDot = className.lastIndexOf(".");
        //服务方接口和实现类所在的包路径
        String interfacePath = className.substring(0, lastDot);
        //获取接口名
        String interfaceName = callInfo.getClassName().substring(lastDot);
        //组合成接口的完整路径
        Class<?> superClass = Class.forName(interfacePath + interfaceName);
        Reflections reflections = new Reflections(interfacePath);
        //得到某接口下的所有实现类
        Set<Class<?>> implClassSet = reflections.getSubTypesOf((Class<Object>) superClass);
        if (implClassSet.size() == 0) {
            System.out.println("未找到实现类");
            return null;
        } else if (implClassSet.size() > 1) {
            System.out.println("找到多个实现类，未明确使用哪一个");
            return null;
        } else {
            //把集合转换为数组
            Class<?>[] classes = implClassSet.toArray(new Class[0]);
            //得到实现类的名字
            return classes[0].getName();
        }
    }

    /**
     * 读取客户端发来的数据并通过反射调用实现类的方法
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        CallInfo callInfo = (CallInfo) msg;
        Object clazz = Class.forName(getImplClassName(callInfo)).newInstance();
        Method method = clazz.getClass().getMethod(callInfo.getMethodName(), callInfo.getTypes());
        //通过反射调用实现类的方法
        Object result = method.invoke(clazz, callInfo.getObjects());
        ctx.writeAndFlush(result);
    }
}
```

- RPCServer，服务提供方的远程服务入口，每个服务提供者，需要有一个类继承它，调用start()方法来启动Netty服务

```
/**
 * 服务提供方的服务
 */
public class RPCServer {
    /**
     * 远程服务端口号
     */
    private final int port;

    public RPCServer(int port) {
        this.port = port;
    }

    /**
     * 开启服务
     */
    public void start() {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .localAddress(port).childHandler(
                    new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            //编码器
                            pipeline.addLast("encoder", new ObjectEncoder());
                            //解码器
                            pipeline.addLast("decoder", new ObjectDecoder(Integer.MAX_VALUE, ClassResolvers.cacheDisabled(null)));
                            //服务器端业务处理类
                            pipeline.addLast(new InvokeHandler());
                        }
                    });
            ChannelFuture future = serverBootstrap.bind(port).sync();
            System.out.println("RPC服务启动完毕...");
            future.channel().closeFuture().sync();
        } catch (Exception e) {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

#### 仓库地址

代码在netty_rpc这个Module中。

[Github地址](https://github.com/hezihaog/netty_sample)