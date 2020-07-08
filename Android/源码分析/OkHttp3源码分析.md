#### OkHttp3源码分析

OkHttp3是目前Android热门的网络请求框架之一，本篇来分析一下OkHttp3最主要的几部分：

- 同步、异步请求流程
- 拦截器对请求的补充和拦截

连接池的复用连接和缓存连接也是一大亮点，不过水平有限，暂时先不分析

#### OkHttpClient构建

OkHttp3请求前需要创建一个OkHttpClient，所有的配置都在OkHttpClient的构建时配置，它使用了构建者模式（Builder模式）来具体化每个配置，并且提供默认配置。

- 如果想使用默认配置，可直接使用new关键字创建（也是使用默认配置的Builder来创建），需要自定义配置则创建OkHttpClient.Builder配置类传入client进行构造。

```
OkHttpClient client = new OkHttpClient();

//使用默认Builder配置来构造client对象
public OkHttpClient() {
    this(new Builder());
}

//自定义Builder配置对象传入构造client
OkHttpClient(Builder builder) {
	//...
}

public static final class Builder {
/**
     * 请求分发器
     */
    Dispatcher dispatcher;
    @Nullable
    Proxy proxy;
    List<Protocol> protocols;
    List<ConnectionSpec> connectionSpecs;
    /**
     * 全局请求拦截器列表
     */
    final List<Interceptor> interceptors = new ArrayList<>();
    /**
     * 非WebSocket时会添加的请求拦截器
     */
    final List<Interceptor> networkInterceptors = new ArrayList<>();
    EventListener.Factory eventListenerFactory;
    ProxySelector proxySelector;
    CookieJar cookieJar;
    @Nullable
    Cache cache;
    /**
     * 内部缓存，实际是缓存的Api接口，内部还是调用Cache对象中的方法来实现缓存
     */
    @Nullable
    InternalCache internalCache;
    SocketFactory socketFactory;
    /**
     * HTTPS使用的安全套接字Socket工厂
     */
    @Nullable
    SSLSocketFactory sslSocketFactory;
    @Nullable
    CertificateChainCleaner certificateChainCleaner;
    HostnameVerifier hostnameVerifier;
    CertificatePinner certificatePinner;
    Authenticator proxyAuthenticator;
    Authenticator authenticator;
    /**
     * 连接池
     */
    ConnectionPool connectionPool;
    Dns dns;
    /**
     * 安全套接字层重定向
     */
    boolean followSslRedirects;
    /**
     * 本地重定向
     */
    boolean followRedirects;
    /**
     * 重试连接失败
     */
    boolean retryOnConnectionFailure;
    int connectTimeout;
    int readTimeout;
    int writeTimeout;
    int pingInterval;
    
    /**
     * 默认的配置
     */
    public Builder() {
        dispatcher = new Dispatcher();
        protocols = DEFAULT_PROTOCOLS;
        connectionSpecs = DEFAULT_CONNECTION_SPECS;
        eventListenerFactory = EventListener.factory(EventListener.NONE);
        proxySelector = ProxySelector.getDefault();
        cookieJar = CookieJar.NO_COOKIES;
        socketFactory = SocketFactory.getDefault();
        hostnameVerifier = OkHostnameVerifier.INSTANCE;
        certificatePinner = CertificatePinner.DEFAULT;
        proxyAuthenticator = Authenticator.NONE;
        authenticator = Authenticator.NONE;
        connectionPool = new ConnectionPool();
        dns = Dns.SYSTEM;
        followSslRedirects = true;
        followRedirects = true;
        retryOnConnectionFailure = true;
        connectTimeout = 10_000;
        readTimeout = 10_000;
        writeTimeout = 10_000;
        pingInterval = 0;
    }
    
    //...
}
```

#### 同步请求流程分析

同步请求就是直接在当前线程请求，会阻塞线程，所以如果在主线程请求，必须开启一个子线程进行请求。

```
OkHttpClient client = new OkHttpClient();
//创建请求配置类Request
Request request = 
	new Request.Builder()
    .url(url)
    .build();
//newCall创建请求对象Call，再通过execute()开始执行，获取请求结果Response对象
Response response = client.newCall(request).execute();
```

- 先来看下client.newCall(request)方法，调用RealCall.newRealCall()静态方法，创建一个RealCall对象，传入request请求配置类对象，并指定请求不是WebSocket请求。

```
/**
 * 发起一个请求，指定不是WebSocket请求
 *
 * @param request 请求的配置
 */
@Override
public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}
```

- RealCall.newRealCall()静态方法，实际就是创建了RealCall对象，并配置了一个事件回调。

```
/**
 * 静态方法配置一个RealCall，RealCall实现了Call接口，暂时是唯一实现，这么设计是为了后续拓展不同的Call
 *
 * @param client          请求客户端
 * @param originalRequest 请求配置类
 * @param forWebSocket    是否是WebSocket请求
 * @return 返回RealCall实例
 */
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    //为每个请求配置上外部设置的事件监听回调
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
}
```

- RealCall类，实现了Call接口，Call接口规定了请求可用的Api，内部还有一个Factory接口，newCall()创建Call对象的方法。看到newCall()方法，那么OkHttpClient也肯定实现了这个接口（果然没猜错）

```
public interface Call extends Cloneable {
  //获取配置的Request对象
  Request request();

  //异步请求Api
  void enqueue(Callback responseCallback);

  //取消任务
  void cancel();

  //任务是否执行过了
  boolean isExecuted();

  //是否取消了
  boolean isCanceled();

  //克隆自己
  Call clone();

  //Call对象生成工厂
  interface Factory {
    Call newCall(Request request);
  }
}

//OkHttpClient实现了Call.Factory，所以它充当了请求工厂的角色
public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory {
	//...
}

//Call接口的实现
final class RealCall implements Call {
	//构造方法就是保存了构造参数，创建了一个拦截器，后面讲
	private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
	        this.client = client;
	        this.originalRequest = originalRequest;
	        this.forWebSocket = forWebSocket;
	        this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client, forWebSocket);
	    }
}
```

- execute执行方法，注意一个RealCall对象只能执行一次，重点在getResponseWithInterceptorChain()方法，调用它后就获得了一个Response对象，那么请求肯定是在这个方法做的。try-catch异常处理，finally块调用分发器dispatcher的finished()方法通知分发器。

```
/**
 * 马上执行，为同步调用（非线程池）
 *
 * @return 请求结果类
 */
@Override
public Response execute() throws IOException {
    synchronized (this) {
        //执行过一次就不可以再执行了，否则抛出异常
        if (executed) {
            throw new IllegalStateException("Already Executed");
        }
        //请求开始，将标志位设置为true
        executed = true;
    }
    captureCallStackTrace();
    //回调外部的事件监听
    eventListener.callStart(this);
    try {
        //通知分发器，开始执行
        client.dispatcher().executed(this);
        //重点：开始责任链调用，让每个拦截器处理请求request，得出请求结果Response
        Response result = getResponseWithInterceptorChain();
        if (result == null) {
            throw new IOException("Canceled");
        }
        return result;
    } catch (IOException e) {
        //异常，回调事件监听对象
        eventListener.callFailed(this, e);
        throw e;
    } finally {
        //结束请求，调用分发器，让分发器继续执行下一个任务
        client.dispatcher().finished(this);
    }
}
```

- 责任链模式的拦截器，具体就是创建拦截器到集合列表，再创建了一个RealInterceptorChain拦截器链，通过proceed()方法处理Request对象，最后得出一个Response请求结果对象，整体很干净，具体工作都交给了拦截器。
	1. 创建了一个拦截器列表，List<Interceptor>。
	2. 首先，先添加我们在Client中配置的拦截器（全局的）
	3. 添加失败和重定向处理的拦截器RetryAndFollowUpInterceptor
	4. 添加请求Header配置和Cookie的拦截器BridgeInterceptor
	5. 添加本地缓存和请求拦截的拦截器CacheInterceptor
	6. 添加服务请求连接的拦截器ConnectInterceptor
	7. 如果不是WebSocket请求，添加我们在Client配置的networkInterceptors()拦截器（局部拦截器，在WebSocket中不会添加的拦截器）
	8. 添加真正请求服务器的拦截器CallServerInterceptor
	9. 创建拦截器链RealInterceptorChain，它实现了Interceptor.Chain接口
	10. 调用proceed()方法开始遍历拦截器处理请求配置类Request

```
/**
 * 开始责任链处理Request请求
 *
 * @return 请求结果
 */
Response getResponseWithInterceptorChain() throws IOException {
    //拦截器列表
    List<Interceptor> interceptors = new ArrayList<>();
    //1、先添加我们在Client中配置的全局拦截器
    interceptors.addAll(client.interceptors());
    //2.添加失败和重定向处理的拦截器
    interceptors.add(retryAndFollowUpInterceptor);
    //3.对请求添加一些必要的Header，接受响应时移除掉一些Header
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //4.缓存拦截器，对请求前本地缓存查询和拦截，对请求结果进行本地缓存
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //5.连接拦截器，负责请求和服务器建立连接
    interceptors.add(new ConnectInterceptor(client));
    //不是WebSocket请求，添加Client中我们添加的拦截器，不是全局的，对WebSocket无效的拦截器
    if (!forWebSocket) {
        interceptors.addAll(client.networkInterceptors());
    }
    //真正请求服务器，获取响应的拦截器
    interceptors.add(new CallServerInterceptor(forWebSocket));
    //创建拦截器链
    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
            originalRequest, this, eventListener, client.connectTimeoutMillis(),
            client.readTimeoutMillis(), client.writeTimeoutMillis());
    //开始责任链遍历
    return chain.proceed(originalRequest);
}
```

- Interceptor拦截器接口，主要是intercept()方法，每种拦截器处理请求Request对象都在这个intercept()方法中处理。

```
/**
 * 拦截器接口
 */
public interface Interceptor {
    /**
     * 拦截回调方法
     *
     * @param chain 拦截器链
     */
    Response intercept(Chain chain) throws IOException;

    /**
     * 拦截器链接口
     */
    interface Chain {
        Request request();

        Response proceed(Request request) throws IOException;

        /**
         * Returns the connection the request will be executed on. This is only available in the chains
         * of network interceptors; for application interceptors this is always null.
         */
        @Nullable
        Connection connection();

        Call call();

        int connectTimeoutMillis();

        Chain withConnectTimeout(int timeout, TimeUnit unit);

        int readTimeoutMillis();

        Chain withReadTimeout(int timeout, TimeUnit unit);

        int writeTimeoutMillis();

        Chain withWriteTimeout(int timeout, TimeUnit unit);
    }
}
```

- RealInterceptorChain，实现了Interceptor.Chain接口，接下来看proceed()方法

	1. calls变量自增计数，记录这个拦截器被调用proceed()方法的次数。
	2. index变量代表自身拦截器在拦截器列表集合中的角标位置，通过index + 1获取下一个拦截器，调用它的proceed()方法，对Request对象进行处理。
	3. 重新创建一个RealInterceptorChain，将拦截器列表、下一个拦截器角标位置等其他配置传入，继续调用interceptor拦截器的next()方法继续分发Request对象，并返回一个Response请求结果对象，并返回出去。

```
/**
 * 拦截器链，管理所有拦截器
 */
public final class RealInterceptorChain implements Interceptor.Chain {
    /**
     * 拦截器列表
     */
    private final List<Interceptor> interceptors;
    private final StreamAllocation streamAllocation;
    /**
     * Http版本处理，Http1和Http2
     */
    private final HttpCodec httpCodec;
    private final RealConnection connection;
    /**
     * 当前遍历到的拦截器角标，每次自增获取下一个拦截器
     */
    private final int index;
    /**
     * 请求配置对象，给每个拦截器处理
     */
    private final Request request;
    /**
     * 请求对象
     */
    private final Call call;
    /**
     * 事件回调对象
     */
    private final EventListener eventListener;
    /**
     * 连接超时事件
     */
    private final int connectTimeout;
    /**
     * 读超时事件
     */
    private final int readTimeout;
    /**
     * 写超时事件
     */
    private final int writeTimeout;
    /**
     * 已调用的拦截器数量
     */
    private int calls;

    public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
                                HttpCodec httpCodec, RealConnection connection, int index, Request request, Call call,
                                EventListener eventListener, int connectTimeout, int readTimeout, int writeTimeout) {
        this.interceptors = interceptors;
        this.connection = connection;
        this.streamAllocation = streamAllocation;
        this.httpCodec = httpCodec;
        this.index = index;
        this.request = request;
        this.call = call;
        this.eventListener = eventListener;
        this.connectTimeout = connectTimeout;
        this.readTimeout = readTimeout;
        this.writeTimeout = writeTimeout;
    }

    //...省略其他方法

    /**
     * 请求处理
     *
     * @param request 请求对象
     * @return 请求结果
     */
    @Override
    public Response proceed(Request request) throws IOException {
        return proceed(request, streamAllocation, httpCodec, connection);
    }

    /**
     * 遍历拦截器进行请求处理，请求处理
     *
     * @param request 请求对象
     */
    public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
                            RealConnection connection) throws IOException {
        //超出界限抛出异常
        if (index >= interceptors.size()) {
            throw new AssertionError();
        }
        //记录调用过的拦截器数量
        calls++;
        // If we already have a stream, confirm that the incoming request will use it.
        if (this.httpCodec != null && !this.connection.supportsUrl(request.url())) {
            throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
                    + " must retain the same host and port");
        }
        // If we already have a stream, confirm that this is the only call to chain.proceed().
        if (this.httpCodec != null && calls > 1) {
            throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
                    + " must call proceed() exactly once");
        }
        //创建一个新的拦截器链
        RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
                connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
                writeTimeout);
        //位置自增，index + 1，获取下一个拦截器
        Interceptor interceptor = interceptors.get(index);
        //调用下一个拦截器处理请求，获取处理结果
        Response response = interceptor.intercept(next);
        // Confirm that the next interceptor made its required call to chain.proceed().
        if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
            throw new IllegalStateException("network interceptor " + interceptor
                    + " must call proceed() exactly once");
        }
        // Confirm that the intercepted response isn't null.
        if (response == null) {
            throw new NullPointerException("interceptor " + interceptor + " returned null");
        }
        if (response.body() == null) {
            throw new IllegalStateException(
                    "interceptor " + interceptor + " returned a response with no body");
        }
        return response;
    }
}
```

#### 异步请求流程分析

- 创建Request请求配置类，调用client的newCall(request)方法创建Call对象，调用Call对象的enqueue()将请求加入请求队列，enqueue(callback)需要传入Callback回调对象。

```
OkHttpClient client = new OkHttpClient();
//创建请求配置类Request
Request request = 
	new Request.Builder()
    .url(url)
    .build();
//newCall创建请求对象Call，再通过enqueue()将请求加入到请求队列
Response response = client.newCall(request).enqueue(new Callback() {
    @Override 
    public void onFailure(Call call, IOException e) {
      	e.printStackTrace();
      	//请求失败
    }

    @Override 
    public void onResponse(Call call, Response response) throws IOException {
        //请求成功，请求响应保存再Response中
    }
);
```

- enqueue(callback)方法流程
	1. 判断executed标志，不允许多次执行
	2. 获取client的dispatcher请求分发器，调用enqueue()，新建一个AsyncCall()包装callback回调对象。

```
/**
 * 异步执行，将请求交给分发器队列处理
 *
 * @param responseCallback 请求的回调对象
 */
@Override
public void enqueue(Callback responseCallback) {
    synchronized (this) {
        //同样判断是否执行过了，不允许多次执行
        if (executed) {
            throw new IllegalStateException("Already Executed");
        }
        //标记
        executed = true;
    }
    captureCallStackTrace();
    //回调事件回调对象
    eventListener.callStart(this);
    //将请求交给分发器入队
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
```

- 先来看下dispatcher请求分发器

	1. Dispatcher内部有3个ArrayDeque双端队列，分别为
		- 准备执行异步任务队列readyAsyncCalls
		- 正在执行异步任务队列runningAsyncCalls
		- runningSyncCalls同步请求任务的队列

	2. enqueue()方法，将包装了callback的AsyncCall对象按规则处理添加不同的队列
		- runningAsyncCalls正在运行的异步任务的队列小于最大请求数
		- runningCallsForHost，异步任务的host主机没有同时5个在允许	
	3. 上面2个条件
		- 满足条件，将任务添加到（正在执行异步任务队列）runningAsyncCalls，调用executorService()获取线程池执行器，将任务提交给执行器马上执行
		- 不满足条件，将任务添加到（准备执行异步任务队列）readyAsyncCalls，等待任务被执行

```
public final class Dispatcher {
	/**
     * 准备执行异步任务的队列
     */
    private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

    /**
     * 正在执行异步任务的队列
     */
    private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

    /**
     * 同步请求任务的队列
     */
    private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
    
    /**
     * 分发器将异步任务入队
     *
     * @param call 异步请求对象
     */
    synchronized void enqueue(AsyncCall call) {
        //入队前限制规则：
        //1、runningAsyncCalls正在运行的异步任务的队列小于最大请求数
        //2、runningCallsForHost，异步任务的host主机没有同时5个在允许
        //3、如果上面2个条件都满足，那么马上将任务排到允许任务队列，并且马上执行
        if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
            //任务添加到正在执行异步任务的队列
            runningAsyncCalls.add(call);
            //获取线程池，马上执行，执行到AsyncCall时，会回调AsyncCall的execute()方法！
            executorService().execute(call);
        } else {
            //不满足条件，添加到准备执行异步任务的队列中
            readyAsyncCalls.add(call);
        }
    }
}
```

- 接下来来分析，AsyncCall类

	1. AsyncCall继承NamedRunnable类
	2. execute()方法为异步执行时调用的方法，和同步一样，调用getResponseWithInterceptorChain()责任链方式遍历拦截器处理Request对象，返回请求结果Response。
	3. 通过retryAndFollowUpInterceptor.isCanceled()处理任务被取消的情况，取消则调用callback.onFailure()表示请求失败。
	4. 没有被取消，则为成功，调用callback.onResponse()表示请求成功。
	5. try-catch处理抛出异常的情况，同样调用callback.onFailure()表示请求失败。
	6. 最后finall块，调用分发器dispatcher的finished方法，表示任务完成，继续执行下一个任务。

```
/**
 * 异步请求，继承NamedRunnable类，NamedRunnable类实现了Runnable接口
 */
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;

    AsyncCall(Callback responseCallback) {
        super("OkHttp %s", redactedUrl());
        this.responseCallback = responseCallback;
    }

    //省略一些方法...

    @Override
    protected void execute() {
        boolean signalledCallback = false;
        try {
            //和同步一样，被线程池执行时，进行责任链处理请求，获取请求结果
            Response response = getResponseWithInterceptorChain();
            //处理被取消的情况
            if (retryAndFollowUpInterceptor.isCanceled()) {
                signalledCallback = true;
                //回调回调接口对象
                responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
            } else {
                //请求成功，回调回调对象
                signalledCallback = true;
                responseCallback.onResponse(RealCall.this, response);
            }
        } catch (IOException e) {
            //处理抛出异常
            if (signalledCallback) {
                // Do not signal the callback twice!
                Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
            } else {
                //回调事件回调接口
                eventListener.callFailed(RealCall.this, e);
                //回调接口回调为失败
                responseCallback.onFailure(RealCall.this, e);
            }
        } finally {
            //结束请求，调用分发器，让分发器继续执行下一个任务
            client.dispatcher().finished(this);
        }
    }
}
```

- 分析任务结束调用的finished()

	1. finished()方法有2个
		- finished(AsyncCall)，异步任务结束后调用。
		- finished(RealCal)同步任务结束后调用。
	2. 2个finished()都会调用到第三个参数的finished。
		- calls，任务队列，异步任务传runningAsyncCalls队列，同步任务传runningSyncCalls队列，
		- call，任务类，异步任务为AsyncCall，同步任务为RealCal。
		- promoteCalls，代表是否推动下一个异步任务执行，同步任务执行完就完了，没有下一个所以为false，异步任务执行完会有执行下一个异步任务，为true。
	3. 先从队列中移除任务（已经已经完成了），再调用promoteCalls()推动下一个任务执行。
	4. 获取同步任务和异步任务的数量，都为0，则回调idleCallback表示任务闲置。

```
/**
 * 异步任务执行结束，会推动下一个异步任务执行，因为可能有等待的异步任务在队列中等待
 *
 * @param call 异步请求对象
 */
void finished(AsyncCall call) {
    finished(runningAsyncCalls, call, true);
}

/**
 * 同步任务执行结束，同步任务没有等待队列，所以执行完就完了
 */
void finished(RealCall call) {
    finished(runningSyncCalls, call, false);
}

/**
 * 任务结束处理
 *
 * @param calls        任务队列
 * @param call         异步任务
 * @param promoteCalls 是否推动下一个异步任务执行
 */
private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
        //从队列中移除异步请求
        if (!calls.remove(call)) {
            throw new AssertionError("Call wasn't in-flight!");
        }
        //推动下一个异步任务执行
        if (promoteCalls) {
            promoteCalls();
        }
        //获取正在运行的异步任务和同步任务的总数量
        runningCallsCount = runningCallsCount();
        idleCallback = this.idleCallback;
    }
    //正在运行的异步任务和同步任务的数量之和为0，则回调空闲回调
    if (runningCallsCount == 0 && idleCallback != null) {
        idleCallback.run();
    }
}
```

- 分析promoteCalls()方法。由于只有异步任务才会调用promoteCalls()，所以都是异步任务的处理
	1. 先判断runningAsyncCalls执行中的异步任务队列的任务是否已满。满了则不继续，等待其他任务执行完来再来带动任务执行。
	2. 再判断任务队列是否为空，没有任务则执行，这种情况在最后一个任务执行完毕后会出现。
	3. 遍历异步任务队列，获取每个异步对象
	4. 判断这个异步任务的host，在当前请求的队列中，是否小于5个正在请求，小于则开始执行下一个异步任务
	5. 如果满足则将任务添加到runningAsyncCalls正在执行异步任务的队列中，让执行器马上执行

```
/**
 * 推动下一个异步任务执行
 */
private void promoteCalls() {
    //正在运行的异步任务队列满了，不开始下一个任务，跳出
    if (runningAsyncCalls.size() >= maxRequests) {
        return; // Already running max capacity.
    }
    //没有任务执行，跳出
    if (readyAsyncCalls.isEmpty()) {
        return; // No ready calls to promote.
    }
    //遍历准备开始异步任务的队列
    for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        //下一个异步任务
        AsyncCall call = i.next();
        //判断这个异步任务的host，在当前请求的队列中，是否小于5个正在请求，小于则开始执行下一个异步任务
        if (runningCallsForHost(call) < maxRequestsPerHost) {
            //从准备执行异步任务队列中移除任务
            i.remove();
            //将任务添加到正在执行的异步任务
            runningAsyncCalls.add(call);
            //将任务交给线程池执行
            executorService().execute(call);
        }
        //判断当前运行的任务队列是否已满
        if (runningAsyncCalls.size() >= maxRequests) {
            return; // Reached max capacity.
        }
    }
}
```

- 最后来看下NamedRunnable类
	1. NamedRunnable实现了Runnable接口，所以他可以被线程池执行器执行。
	2. 使用模板模式，复写run()方法，调用定义的抽象execute()执行方法，让子类重写，同时将run()方法设置为final防止被重写流程。
	3. 统一设置线程名称。

```
/**
 * 实现了Runnable接口，给每个异步任务定义线程名
 */
public abstract class NamedRunnable implements Runnable {
    protected final String name;

    public NamedRunnable(String format, Object... args) {
        this.name = Util.format(format, args);
    }

    /**
     * 任务run()方法，为被子线程执行的回调方法，设置为final，不允许子类复写修改，并提供execute抽象方法，模板模式处理，定义统一行为
     */
    @Override
    public final void run() {
        //配置线程名
        String oldName = Thread.currentThread().getName();
        Thread.currentThread().setName(name);
        try {
            //调用定义execute抽象方法
            execute();
        } finally {
            Thread.currentThread().setName(oldName);
        }
    }

    /**
     * 将run()方法转化为抽象的execute给子类复写，执行异步任务
     */
    protected abstract void execute();
}
```

#### 拦截器分析

主要分析的拦截器：

1. BridgeInterceptor：对请求添加一些必要的Header，接受响应时移除掉一些Header的拦截器
2. CacheInterceptor：缓存拦截器，对请求前本地缓存查询和拦截，对请求结果进行本地缓存
3. ConnectInterceptor：连接拦截器，负责请求和服务器建立连接

- BridgeInterceptor，对请求添加一些必要的Header，接受响应时移除掉一些Header的拦截器

```
public final class BridgeInterceptor implements Interceptor {
    private final CookieJar cookieJar;

    public BridgeInterceptor(CookieJar cookieJar) {
        this.cookieJar = cookieJar;
    }

    /**
     * 将配置的Request对象中的Header配置整理一遍重新创建一个Request
     *
     * @param chain 拦截器链
     * @return 请求结果
     */
    @Override
    public Response intercept(Chain chain) throws IOException {
        //获取原始配置
        Request userRequest = chain.request();
        //新建Request配置
        Request.Builder requestBuilder = userRequest.newBuilder();
        //获取Body
        RequestBody body = userRequest.body();
        if (body != null) {
            MediaType contentType = body.contentType();
            if (contentType != null) {
                requestBuilder.header("Content-Type", contentType.toString());
            }

            long contentLength = body.contentLength();
            if (contentLength != -1) {
                requestBuilder.header("Content-Length", Long.toString(contentLength));
                requestBuilder.removeHeader("Transfer-Encoding");
            } else {
                requestBuilder.header("Transfer-Encoding", "chunked");
                requestBuilder.removeHeader("Content-Length");
            }
        }
        //配置默认Host
        if (userRequest.header("Host") == null) {
            requestBuilder.header("Host", hostHeader(userRequest.url(), false));
        }
        //配置默认Connection为保持连接
        if (userRequest.header("Connection") == null) {
            requestBuilder.header("Connection", "Keep-Alive");
        }

        // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
        // the transfer stream.
        //添加透明的Gzip压缩
        boolean transparentGzip = false;
        if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
            transparentGzip = true;
            requestBuilder.header("Accept-Encoding", "gzip");
        }
        //如果有配置Cookie，添加Cookie到Header
        List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
        if (!cookies.isEmpty()) {
            requestBuilder.header("Cookie", cookieHeader(cookies));
        }
        //配置默认的UA
        if (userRequest.header("User-Agent") == null) {
            requestBuilder.header("User-Agent", Version.userAgent());
        }
        //生成新的Request，执行下一个拦截器获取响应结果
        Response networkResponse = chain.proceed(requestBuilder.build());
        //处理请求头
        HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());
        //重新生成响应配置
        Response.Builder responseBuilder = networkResponse.newBuilder()
                .request(userRequest);
        //响应报文，移除一些Header，压缩响应报文
        if (transparentGzip
                && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
                && HttpHeaders.hasBody(networkResponse)) {
            GzipSource responseBody = new GzipSource(networkResponse.body().source());
            Headers strippedHeaders = networkResponse.headers().newBuilder()
                    .removeAll("Content-Encoding")
                    .removeAll("Content-Length")
                    .build();
            responseBuilder.headers(strippedHeaders);
            String contentType = networkResponse.header("Content-Type");
            responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
        }
        //重新生成响应报文
        return responseBuilder.build();
    }

    /**
     * Returns a 'Cookie' HTTP request header with all cookies, like {@code a=b; c=d}.
     */
    private String cookieHeader(List<Cookie> cookies) {
        StringBuilder cookieHeader = new StringBuilder();
        for (int i = 0, size = cookies.size(); i < size; i++) {
            if (i > 0) {
                cookieHeader.append("; ");
            }
            Cookie cookie = cookies.get(i);
            cookieHeader.append(cookie.name()).append('=').append(cookie.value());
        }
        return cookieHeader.toString();
    }
}
```

- CacheInterceptor，缓存拦截器，对请求前本地缓存查询和拦截，对请求结果进行本地缓存

```
public final class CacheInterceptor implements Interceptor {
    final InternalCache cache;

    public CacheInterceptor(InternalCache cache) {
        this.cache = cache;
    }

    /**
     * 缓存拦截器
     *
     * @param chain 拦截器链
     * @return 请求结果
     */
    @Override
    public Response intercept(Chain chain) throws IOException {
        //获取缓存容器中请求的缓存，候选缓存
        Response cacheCandidate = cache != null
                ? cache.get(chain.request())
                : null;
        long now = System.currentTimeMillis();
        //缓存策略工厂，传入网络请求的Request和缓存的响应，决定缓存策略
        CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
        Request networkRequest = strategy.networkRequest;
        Response cacheResponse = strategy.cacheResponse;
        if (cache != null) {
            cache.trackResponse(strategy);
        }
        if (cacheCandidate != null && cacheResponse == null) {
            closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
        }
        //请求失败、缓存也失败，返回失败的结果
        if (networkRequest == null && cacheResponse == null) {
            return new Response.Builder()
                    .request(chain.request())
                    .protocol(Protocol.HTTP_1_1)
                    .code(504)
                    .message("Unsatisfiable Request (only-if-cached)")
                    .body(Util.EMPTY_RESPONSE)
                    .sentRequestAtMillis(-1L)
                    .receivedResponseAtMillis(System.currentTimeMillis())
                    .build();
        }
        //缓存不为空，网络请求也不需要，则我们缓存命中
        if (networkRequest == null) {
            return cacheResponse.newBuilder()
                    .cacheResponse(stripBody(cacheResponse))
                    .build();
        }
        //缓存不命中，要请求网络，调用下一个拦截器去请求，并获取结果
        Response networkResponse = null;
        try {
            networkResponse = chain.proceed(networkRequest);
        } finally {
            // If we're crashing on I/O or otherwise, don't leak the cache body.
            if (networkResponse == null && cacheCandidate != null) {
                closeQuietly(cacheCandidate.body());
            }
        }
        //本地缓存是存在的，将本地缓存和网络请求结果比较，决定是否将这个网络请求结果缓存到本地
        if (cacheResponse != null) {
            if (networkResponse.code() == HTTP_NOT_MODIFIED) {
                Response response = cacheResponse.newBuilder()
                        .headers(combine(cacheResponse.headers(), networkResponse.headers()))
                        .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
                        .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
                        .cacheResponse(stripBody(cacheResponse))
                        .networkResponse(stripBody(networkResponse))
                        .build();
                networkResponse.body().close();

                // Update the cache after combining headers but before stripping the
                // Content-Encoding header (as performed by initContentStream()).
                cache.trackConditionalCacheHit();
                //以本次网络请求结果，更新本地缓存
                cache.update(cacheResponse, response);
                return response;
            } else {
                closeQuietly(cacheResponse.body());
            }
        }
        //重新构造输出结果
        Response response = networkResponse.newBuilder()
                .cacheResponse(stripBody(cacheResponse))
                .networkResponse(stripBody(networkResponse))
                .build();

        if (cache != null) {
            if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
                //本地缓存不存在，保存本次网络请求的缓存到本地
                CacheRequest cacheRequest = cache.put(response);
                return cacheWritingResponse(cacheRequest, response);
            }
            //如果缓存是无效的，则移除缓存
            if (HttpMethod.invalidatesCache(networkRequest.method())) {
                try {
                    cache.remove(networkRequest);
                } catch (IOException ignored) {
                    // The cache cannot be written.
                }
            }
        }
        return response;
    }
    
    //...省略其他代码
}
```

- ConnectInterceptor，连接拦截器，负责请求和服务器建立连接

```
public final class ConnectInterceptor implements Interceptor {
    public final OkHttpClient client;

    public ConnectInterceptor(OkHttpClient client) {
        this.client = client;
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        RealInterceptorChain realChain = (RealInterceptorChain) chain;
        Request request = realChain.request();
        //StreamAllocation将连接创建、连接管理的封装类
        StreamAllocation streamAllocation = realChain.streamAllocation();

        // We need the network to satisfy this request. Possibly for validating a conditional GET.
        boolean doExtensiveHealthChecks = !request.method().equals("GET");
        //HttpCodec是对 HTTP 协议操作的抽象，有两个实现：Http1Codec和Http2Codec，顾名思义，它们分别对应 HTTP/1.1 和 HTTP/2 版本的实现。在这个方法的内部实现连接池的复用处理
        //开始连接
        HttpCodec httpCodec = streamAllocation.newStream(client, chain, doExtensiveHealthChecks);
        RealConnection connection = streamAllocation.connection();
        //调用下一个拦截器
        return realChain.proceed(request, streamAllocation, httpCodec, connection);
    }
}
```

#### 总结

本篇分析了OkHttp3的请求流程和拦截器，作为最为核心的就是拦截器部分，将每部分的附加和处理都分类到不同的拦截器进行解耦。

- 异步流程通过Dispatch分发器将任务提交给线程池，每个任务执行完后通知分发器，分发器再继续寻找下一个异步任务进行执行。

- 桥接拦截器对Request做Header补充，并且自动处理Cookie。

- 缓存拦截器，先对Request对象从本地获取缓存，缓存可用时则使用，否则调用下一个拦截器进行请求，请求成功后返回，获取到结果后，再缓存到本地。