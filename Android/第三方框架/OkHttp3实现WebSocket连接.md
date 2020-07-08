#### OkHttp3实现WebSocket连接

项目中有一个IM模块，是使用了WebSocket来做的，特此记录一下。

WebSocket的框架有很多，了解到OkHttp3也有支持WebSocket，就采用了Okhttp来实现。

一个是不需要再引入多一个WebSocket的第三方库，一个是Okhttp3口碑和稳定性都非常好，而且还一直在更新。

#### 添加依赖

```
implementation 'com.squareup.okhttp3:okhttp:3.8.1'
```

#### 实现步骤

- 构建OkHttpClient配置初始化一些参数。

- 使用WebSocket的Url地址连接。

- 设置WebSocket的连接状态回调和消息回调。

- 解析消息json处理业务等。

- 连接成功后，使用WebSocket发送消息

-----------------------------------------

1. 配置OkHttpClient

```
OkHttpClient mClient = new OkHttpClient.Builder()
        .readTimeout(3, TimeUnit.SECONDS)//设置读取超时时间
        .writeTimeout(3, TimeUnit.SECONDS)//设置写的超时时间
        .connectTimeout(3, TimeUnit.SECONDS)//设置连接超时时间
        .build();
```

2. 使用Url，构建WebSocket请求（一般是后端接口返回连接的Url地址）

```
//连接地址
String url = "ws://xxxxx"
//构建一个连接请求对象
Request request = new Request.Builder().get().url(url).build();
```

3. 发起连接，配置回调。

	- onOpen()，连接成功
	- onMessage(String text)，收到字符串类型的消息，一般我们都是使用这个
	- onMessage(ByteString bytes)，收到字节数组类型消息，我这里没有用到
	- onClosed()，连接关闭
	- onFailure()，连接失败，一般都是在这里发起重连操作

```
//开始连接
WebSocket websocket = mClient.newWebSocket(request, new WebSocketListener() {
    @Override
    public void onOpen(WebSocket webSocket, Response response) {
        super.onOpen(webSocket, response);
        //连接成功...
    }

    @Override
    public void onMessage(WebSocket webSocket, String text) {
        super.onMessage(webSocket, text);
        //收到消息...（一般是这里处理json）
    }

    @Override
    public void onMessage(WebSocket webSocket, ByteString bytes) {
        super.onMessage(webSocket, bytes);
        //收到消息...（一般很少这种消息）
    }

    @Override
    public void onClosed(WebSocket webSocket, int code, String reason) {
        super.onClosed(webSocket, code, reason);
        //连接关闭...
    }

    @Override
    public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
        super.onFailure(webSocket, throwable, response);
        //连接失败...
    }
});
```

4. 使用WebSocket对象发送消息，msg为消息内容（一般是json，当然你也可以使用其他的，例如xml等），send方法会马上返回发送结果。

```
//发送消息
boolean isSendSuccess = webSocket.send(msg);
```

#### 配合RxJava封装

配置RxJava，我们可以为WebSocket增强数据转换，线程切换和重连处理等功能。

#### 实现步骤

1. 定义Api调用接口，外部只需要接触Api无无需关心内部实现逻辑。

```
public interface WebSocketWorker {
    /**
     * 获取连接，并返回观察对象
     */
    Observable<WebSocketInfo> get(String url);

    /**
     * 设置一个超时时间，在指定时间内如果没有收到消息，会尝试重连
     *
     * @param timeout  超时时间
     * @param timeUnit 超时时间单位
     */
    Observable<WebSocketInfo> get(String url, long timeout, TimeUnit timeUnit);

    /**
     * 发送，url的WebSocket已打开的情况下使用，否则会抛出异常
     *
     * @param msg 消息，看具体和后端协商的格式，一般为json
     */
    Observable<Boolean> send(String url, String msg);

    /**
     * 发送，同上
     *
     * @param byteString 信息类型为ByteString
     */
    Observable<Boolean> send(String url, ByteString byteString);

    /**
     * 不关心WebSocket是否连接，直接发送
     */
    Observable<Boolean> asyncSend(String url, String msg);

    /**
     * 同上，只是消息类型为ByteString
     */
    Observable<Boolean> asyncSend(String url, ByteString byteString);

    /**
     * 关闭指定Url的连接
     */
    Observable<Boolean> close(String url);

    /**
     * 马上关闭指定Url的连接
     */
    boolean closeNow(String url);

    /**
     * 关闭当前所有连接
     */
    Observable<List<Boolean>> closeAll();

    /**
     * 马上关闭所有连接
     */
    void closeAllNow();
}
```

2. 构建者模式，大量的配置参数，我们先使用一个Builder类保存，再使用build()方法生成RxWebSocket对象。

```
public class RxWebSocketBuilder {
    Context mContext;
    /**
     * 是否打印Log
     */
    boolean mIsPrintLog;
    /**
     * Log代理对象
     */
    Logger.LogDelegate mLogDelegate;
    /**
     * 支持外部传入OkHttpClient
     */
    OkHttpClient mClient;
    /**
     * 支持SSL
     */
    SSLSocketFactory mSslSocketFactory;
    X509TrustManager mTrustManager;
    /**
     * 重连间隔时间
     */
    long mReconnectInterval;
    /**
     * 重连间隔时间的单位
     */
    TimeUnit mReconnectIntervalTimeUnit;

    public RxWebSocketBuilder(Context context) {
        this.mContext = context.getApplicationContext();
    }

    public RxWebSocketBuilder isPrintLog(boolean isPrintLog) {
        this.mIsPrintLog = isPrintLog;
        return this;
    }

    public RxWebSocketBuilder logger(Logger.LogDelegate logDelegate) {
        Logger.setDelegate(logDelegate);
        return this;
    }

    public RxWebSocketBuilder client(OkHttpClient client) {
        this.mClient = client;
        return this;
    }

    public RxWebSocketBuilder sslSocketFactory(SSLSocketFactory sslSocketFactory, X509TrustManager trustManager) {
        this.mSslSocketFactory = sslSocketFactory;
        this.mTrustManager = trustManager;
        return this;
    }

    public RxWebSocketBuilder reconnectInterval(long reconnectInterval, TimeUnit reconnectIntervalTimeUnit) {
        this.mReconnectInterval = reconnectInterval;
        this.mReconnectIntervalTimeUnit = reconnectIntervalTimeUnit;
        return this;
    }

    public RxWebSocket build() {
        return new RxWebSocket(this);
    }
}
```

2. Api实现类，这里我使用代理模式定义一个代理对象，隔离内部实现，暂时只是简单转调实现，后续需要增加限制逻辑时，在这里加），也方便使用装饰者模式增强逻辑。

```
public class RxWebSocket implements WebSocketWorker {
    private Context mContext;
    /**
     * 是否打印Log
     */
    private boolean mIsPrintLog;
    /**
     * Log代理对象
     */
    private Logger.LogDelegate mLogDelegate;
    /**
     * 支持外部传入OkHttpClient
     */
    private OkHttpClient mClient;
    /**
     * 支持SSL
     */
    private SSLSocketFactory mSslSocketFactory;
    private X509TrustManager mTrustManager;
    /**
     * 重连间隔时间
     */
    private long mReconnectInterval;
    /**
     * 重连间隔时间的单位
     */
    private TimeUnit mReconnectIntervalTimeUnit;
    /**
     * 具体干活的实现类
     */
    private WebSocketWorker mWorkerImpl;

    private RxWebSocket() {
    }

    RxWebSocket(RxWebSocketBuilder builder) {
        this.mContext = builder.mContext;
        this.mIsPrintLog = builder.mIsPrintLog;
        this.mLogDelegate = builder.mLogDelegate;
        this.mClient = builder.mClient == null ? new OkHttpClient() : builder.mClient;
        this.mSslSocketFactory = builder.mSslSocketFactory;
        this.mTrustManager = builder.mTrustManager;
        this.mReconnectInterval = builder.mReconnectInterval == 0 ? 1 : builder.mReconnectInterval;
        this.mReconnectIntervalTimeUnit = builder.mReconnectIntervalTimeUnit == null ? TimeUnit.SECONDS : builder.mReconnectIntervalTimeUnit;
        setup();
    }

    /**
     * 开始配置
     */
    private void setup() {
        this.mWorkerImpl = new WebSocketWorkerImpl(
                this.mContext,
                this.mIsPrintLog,
                this.mLogDelegate,
                this.mClient,
                this.mSslSocketFactory,
                this.mTrustManager,
                this.mReconnectInterval,
                this.mReconnectIntervalTimeUnit);
    }

    //...Api都是转调mWorkerImpl，mWorkerImpl是具体的实现类
}
```

3. WebSocketInfo消息实体，RxJava发送消息通知订阅者需要一个实体Model类来保存发送的消息，例如发送的消息字符串，标识是否重连，是否连接成功等。

- 需要缓存的模型需要实现的接口

```
public interface ICacheTarget<T> {
    /**
     * 重置方法
     *
     * @return 重置后的对象
     */
    T reset();
}
```

- 缓存池接口

```
public interface ICachePool<T extends ICacheTarget<T>> {
    /**
     * 创建缓存时回调
     */
    T onCreateCache();

    /**
     * 设置缓存对象的最大个数
     */
    int onSetupMaxCacheCount();

    /**
     * 获取一个缓存对象
     *
     * @param cacheKey 缓存Key，为了应对多个观察者同时获取缓存使用
     */
    T obtain(String cacheKey);

    /**
     * 当获取一个缓存后回调，一般在该回调中重置对象的所有字段
     *
     * @param cacheTarget 缓存对象
     */
    T onObtainCacheAfter(ICacheTarget<T> cacheTarget);
}
```

- 统一的缓存模型包裹类

```
public class CacheItem<T> implements Serializable {
    private static final long serialVersionUID = -401778630524300400L;

    /**
     * 缓存的对象
     */
    private T cacheTarget;
    /**
     * 最近使用时间
     */
    private long recentlyUsedTime;

    public CacheItem(T cacheTarget, long recentlyUsedTime) {
        this.cacheTarget = cacheTarget;
        this.recentlyUsedTime = recentlyUsedTime;
    }

    public T getCacheTarget() {
        return cacheTarget;
    }

    public void setCacheTarget(T cacheTarget) {
        this.cacheTarget = cacheTarget;
    }

    public long getRecentlyUsedTime() {
        return recentlyUsedTime;
    }

    public void setRecentlyUsedTime(long recentlyUsedTime) {
        this.recentlyUsedTime = recentlyUsedTime;
    }
}
```

- 基础缓存池

```
public abstract class BaseCachePool<T extends ICacheTarget<T>> implements ICachePool<T>, Comparator<CacheItem<T>> {
    /**
     * 缓存池
     */
    private ConcurrentHashMap<String, LinkedList<CacheItem<T>>> mPool;

    public BaseCachePool() {
        mPool = new ConcurrentHashMap<>(8);
    }

    @Override
    public T obtain(String cacheKey) {
        //缓存链
        LinkedList<CacheItem<T>> cacheChain;
        //没有缓存过，进行缓存
        if (!mPool.containsKey(cacheKey)) {
            cacheChain = new LinkedList<>();
        } else {
            cacheChain = mPool.get(cacheKey);
        }
        if (cacheChain == null) {
            throw new NullPointerException("cacheChain 缓存链创建失败");
        }
        //未满最大缓存数量，生成一个实例
        if (cacheChain.size() < onSetupMaxCacheCount()) {
            T cache = onCreateCache();
            CacheItem<T> cacheItem = new CacheItem<>(cache, System.currentTimeMillis());
            cacheChain.add(cacheItem);
            mPool.put(cacheKey, cacheChain);
            return cache;
        }
        //达到最大缓存数量。按最近的使用时间排序，最近使用的放后面，每次取只取最前面（最久没有使用的）
        Collections.sort(cacheChain, this);
        CacheItem<T> cacheItem = cacheChain.getFirst();
        cacheItem.setRecentlyUsedTime(System.currentTimeMillis());
        //重置所有属性
        T cacheTarget = cacheItem.getCacheTarget();
        cacheTarget = onObtainCacheAfter(cacheTarget);
        return cacheTarget;
    }
    
    @Override
    public T onObtainCacheAfter(ICacheTarget<T> cacheTarget) {
        //默认调用reset方法进行重置，如果有其他需求，子类再进行复写
        return cacheTarget.reset();
    }

    @Override
    public int compare(CacheItem<T> o1, CacheItem<T> o2) {
        return Long.compare(o1.getRecentlyUsedTime(), o2.getRecentlyUsedTime());
    }
}
```

- 实现我们的缓存模型

```
public class WebSocketInfo implements Serializable, ICacheTarget<WebSocketInfo> {
    private static final long serialVersionUID = -880481254453932113L;

    private WebSocket mWebSocket;
    private String mStringMsg;
    private ByteString mByteStringMsg;
    /**
     * 连接成功
     */
    private boolean isConnect;
    /**
     * 重连成功
     */
    private boolean isReconnect;
    /**
     * 准备重连
     */
    private boolean isPrepareReconnect;

    /**
     * 重置
     */
    @Override
    public WebSocketInfo reset() {
        this.mWebSocket = null;
        this.mStringMsg = null;
        this.mByteStringMsg = null;
        this.isConnect = false;
        this.isReconnect = false;
        this.isPrepareReconnect = false;
        return this;
    }

	 //省略get、set方法
}
```

4. 具体实现。

- 将连接WebSocket封装到Observable的订阅回调中。
- 以Map缓存Url和数据源，多个Url共享同一个连接对象。
- 使用share操作符，让多个观察者同时订阅一个数据源。所有订阅者都取消订阅时，再断开连接。

```
public class WebSocketWorkerImpl implements WebSocketWorker {
    private static final String TAG = WebSocketWorkerImpl.class.getName();

    /**
     * 上下文
     */
    private Context mContext;
    /**
     * 支持外部传入OkHttpClient
     */
    private OkHttpClient mClient;
    /**
     * 重连间隔时间
     */
    private long mReconnectInterval;
    /**
     * 重连间隔时间的单位
     */
    private TimeUnit mReconnectIntervalTimeUnit;

    /**
     * 缓存观察者对象，Url对应一个Observable
     */
    private Map<String, Observable<WebSocketInfo>> mObservableCacheMap;
    /**
     * 缓存Url和对应的WebSocket实例，同一个Url共享一个WebSocket连接
     */
    private Map<String, WebSocket> mWebSocketPool;
    /**
     * WebSocketInfo缓存池
     */
    private final WebSocketInfoPool mWebSocketInfoPool;

    public WebSocketWorkerImpl(
            Context context,
            boolean isPrintLog,
            Logger.LogDelegate logDelegate,
            OkHttpClient client,
            SSLSocketFactory sslSocketFactory,
            X509TrustManager trustManager,
            long reconnectInterval,
            TimeUnit reconnectIntervalTimeUnit) {
        this.mContext = context;
        //配置Logger
        Logger.setDelegate(logDelegate);
        Logger.setLogPrintEnable(isPrintLog);
        this.mClient = client;
        //重试时间配置
        this.mReconnectInterval = reconnectInterval;
        this.mReconnectIntervalTimeUnit = reconnectIntervalTimeUnit;
        //配置SSL
        if (sslSocketFactory != null && trustManager != null) {
            mClient = mClient.newBuilder().sslSocketFactory(sslSocketFactory, trustManager).build();
        }
        this.mObservableCacheMap = new HashMap<>(16);
        this.mWebSocketPool = new HashMap<>(16);
        mWebSocketInfoPool = new WebSocketInfoPool();
    }

    @Override
    public Observable<WebSocketInfo> get(String url) {
        return getWebSocketInfo(url);
    }

    @Override
    public Observable<WebSocketInfo> get(String url, long timeout, TimeUnit timeUnit) {
        return getWebSocketInfo(url, timeout, timeUnit);
    }

    @Override
    public Observable<Boolean> send(String url, String msg) {
        return Observable.create(new ObservableOnSubscribe<Boolean>() {
            @Override
            public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
                WebSocket webSocket = mWebSocketPool.get(url);
                if (webSocket == null) {
                    emitter.onError(new IllegalStateException("The WebSocket not open"));
                } else {
                    emitter.onNext(webSocket.send(msg));
                }
            }
        });
    }

    @Override
    public Observable<Boolean> send(String url, ByteString byteString) {
        return Observable.create(new ObservableOnSubscribe<Boolean>() {
            @Override
            public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
                WebSocket webSocket = mWebSocketPool.get(url);
                if (webSocket == null) {
                    emitter.onError(new IllegalStateException("The WebSocket not open"));
                } else {
                    emitter.onNext(webSocket.send(byteString));
                }
            }
        });
    }

    @Override
    public Observable<Boolean> asyncSend(String url, String msg) {
        return getWebSocket(url)
                .take(1)
                .map(new Function<WebSocket, Boolean>() {
                    @Override
                    public Boolean apply(WebSocket webSocket) throws Exception {
                        return webSocket.send(msg);
                    }
                });
    }

    @Override
    public Observable<Boolean> asyncSend(String url, ByteString byteString) {
        return getWebSocket(url)
                .take(1)
                .map(new Function<WebSocket, Boolean>() {
                    @Override
                    public Boolean apply(WebSocket webSocket) throws Exception {
                        return webSocket.send(byteString);
                    }
                });
    }

    @Override
    public Observable<Boolean> close(String url) {
        return Observable.create(new ObservableOnSubscribe<WebSocket>() {
            @Override
            public void subscribe(ObservableEmitter<WebSocket> emitter) throws Exception {
                WebSocket webSocket = mWebSocketPool.get(url);
                if (webSocket == null) {
                    emitter.onError(new NullPointerException("url:" + url + " WebSocket must be not null"));
                } else {
                    emitter.onNext(webSocket);
                }
            }
        }).map(new Function<WebSocket, Boolean>() {
            @Override
            public Boolean apply(WebSocket webSocket) throws Exception {
                return closeWebSocket(webSocket);
            }
        });
    }

    @Override
    public boolean closeNow(String url) {
        return closeWebSocket(mWebSocketPool.get(url));
    }

    @Override
    public Observable<List<Boolean>> closeAll() {
        return Observable
                .just(mWebSocketPool)
                .map(new Function<Map<String, WebSocket>, Collection<WebSocket>>() {
                    @Override
                    public Collection<WebSocket> apply(Map<String, WebSocket> webSocketMap) throws Exception {
                        return webSocketMap.values();
                    }
                })
                .concatMap(new Function<Collection<WebSocket>, ObservableSource<WebSocket>>() {
                    @Override
                    public ObservableSource<WebSocket> apply(Collection<WebSocket> webSockets) throws Exception {
                        return Observable.fromIterable(webSockets);
                    }
                }).map(new Function<WebSocket, Boolean>() {
                    @Override
                    public Boolean apply(WebSocket webSocket) throws Exception {
                        return closeWebSocket(webSocket);
                    }
                }).collect(new Callable<List<Boolean>>() {
                    @Override
                    public List<Boolean> call() throws Exception {
                        return new ArrayList<>();
                    }
                }, new BiConsumer<List<Boolean>, Boolean>() {
                    @Override
                    public void accept(List<Boolean> list, Boolean isCloseSuccess) throws Exception {
                        list.add(isCloseSuccess);
                    }
                }).toObservable();
    }

    @Override
    public void closeAllNow() {
        for (Map.Entry<String, WebSocket> entry : mWebSocketPool.entrySet()) {
            closeWebSocket(entry.getValue());
        }
    }

    /**
     * 是否有连接
     */
    private boolean hasWebSocketConnection(String url) {
        return mWebSocketPool.get(url) != null;
    }

    /**
     * 关闭WebSocket连接
     */
    private boolean closeWebSocket(WebSocket webSocket) {
        if (webSocket == null) {
            return false;
        }
        WebSocketCloseEnum normalCloseEnum = WebSocketCloseEnum.USER_EXIT;
        boolean result = webSocket.close(normalCloseEnum.getCode(), normalCloseEnum.getReason());
        if (result) {
            removeUrlWebSocketMapping(webSocket);
        }
        return result;
    }

    /**
     * 移除Url和WebSocket的映射
     */
    private void removeUrlWebSocketMapping(WebSocket webSocket) {
        for (Map.Entry<String, WebSocket> entry : mWebSocketPool.entrySet()) {
            if (entry.getValue() == webSocket) {
                String url = entry.getKey();
                mObservableCacheMap.remove(url);
                mWebSocketPool.remove(url);
            }
        }
    }

    private void removeWebSocketCache(WebSocket webSocket) {
        for (Map.Entry<String, WebSocket> entry : mWebSocketPool.entrySet()) {
            if (entry.getValue() == webSocket) {
                String url = entry.getKey();
                mWebSocketPool.remove(url);
            }
        }
    }

    public Observable<WebSocket> getWebSocket(String url) {
        return getWebSocketInfo(url)
                .filter(new Predicate<WebSocketInfo>() {
                    @Override
                    public boolean test(WebSocketInfo webSocketInfo) throws Exception {
                        return webSocketInfo.getWebSocket() != null;
                    }
                })
                .map(new Function<WebSocketInfo, WebSocket>() {
                    @Override
                    public WebSocket apply(WebSocketInfo webSocketInfo) throws Exception {
                        return webSocketInfo.getWebSocket();
                    }
                });
    }

    public Observable<WebSocketInfo> getWebSocketInfo(String url) {
        return getWebSocketInfo(url, 5, TimeUnit.SECONDS);
    }

    public synchronized Observable<WebSocketInfo> getWebSocketInfo(final String url, final long timeout, final TimeUnit timeUnit) {
        //先从缓存中取
        Observable<WebSocketInfo> observable = mObservableCacheMap.get(url);
        if (observable == null) {
            //缓存中没有，新建
            observable = Observable
                    .create(new WebSocketOnSubscribe(url))
                    .retry()
                    //因为有share操作符，只有当所有观察者取消注册时，这里才会回调
                    .doOnDispose(new Action() {
                        @Override
                        public void run() throws Exception {
                            //所有都不注册了，关闭连接
                            closeNow(url);
                            Logger.d(TAG, "所有观察者都取消注册，关闭连接...");
                        }
                    })
                    //Share操作符，实现多个观察者对应一个数据源
                    .share()
                    //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
                    .subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread());
            //将数据源缓存
            mObservableCacheMap.put(url, observable);
        } else {
            //缓存中有，从连接池中取出
            WebSocket webSocket = mWebSocketPool.get(url);
            if (webSocket != null) {
                observable = observable.startWith(createConnect(url, webSocket));
            }
        }
        return observable;
    }

    /**
     * 组装数据源
     */
    private final class WebSocketOnSubscribe implements ObservableOnSubscribe<WebSocketInfo> {
        private String mWebSocketUrl;
        private WebSocket mWebSocket;
        private boolean isReconnecting = false;

        public WebSocketOnSubscribe(String webSocketUrl) {
            this.mWebSocketUrl = webSocketUrl;
        }

        @Override
        public void subscribe(ObservableEmitter<WebSocketInfo> emitter) throws Exception {
            //因为retry重连不能设置延时，所以只能这里延时，降低发送频率
            if (mWebSocket == null && isReconnecting) {
                if (Thread.currentThread() != Looper.getMainLooper().getThread()) {
                    long millis = mReconnectIntervalTimeUnit.toMillis(mReconnectInterval);
                    if (millis == 0) {
                        millis = 1000;
                    }
                    SystemClock.sleep(millis);
                }
            }
            initWebSocket(emitter);
        }

        private Request createRequest(String url) {
            return new Request.Builder().get().url(url).build();
        }

        /**
         * 初始化WebSocket
         */
        private synchronized void initWebSocket(ObservableEmitter<WebSocketInfo> emitter) {
            if (mWebSocket == null) {
                mWebSocket = mClient.newWebSocket(createRequest(mWebSocketUrl), new WebSocketListener() {
                    @Override
                    public void onOpen(WebSocket webSocket, Response response) {
                        super.onOpen(webSocket, response);
                        //连接成功
                        if (!emitter.isDisposed()) {
                            mWebSocketPool.put(mWebSocketUrl, mWebSocket);
                            //重连成功
                            if (isReconnecting) {
                                emitter.onNext(createReconnect(mWebSocketUrl, webSocket));
                            } else {
                                emitter.onNext(createConnect(mWebSocketUrl, webSocket));
                            }
                        }
                        isReconnecting = false;
                    }

                    @Override
                    public void onMessage(WebSocket webSocket, String text) {
                        super.onMessage(webSocket, text);
                        //收到消息
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createReceiveStringMsg(mWebSocketUrl, webSocket, text));
                        }
                    }

                    @Override
                    public void onMessage(WebSocket webSocket, ByteString bytes) {
                        super.onMessage(webSocket, bytes);
                        //收到消息
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createReceiveByteStringMsg(mWebSocketUrl, webSocket, bytes));
                        }
                    }

                    @Override
                    public void onClosed(WebSocket webSocket, int code, String reason) {
                        super.onClosed(webSocket, code, reason);
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createClose(mWebSocketUrl));
                        }
                    }

                    @Override
                    public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
                        super.onFailure(webSocket, throwable, response);
                        isReconnecting = true;
                        mWebSocket = null;
                        //移除WebSocket缓存，retry重试重新连接
                        removeWebSocketCache(webSocket);
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createPrepareReconnect(mWebSocketUrl));
                            //失败发送onError，让retry操作符重试
                            emitter.onError(new ImproperCloseException());
                        }
                    }
                });
            }
        }
    }


    private WebSocketInfo createConnect(String url, WebSocket webSocket) {
        return mWebSocketInfoPool.obtain(url)
                .setWebSocket(webSocket)
                .setConnect(true);
    }

    private WebSocketInfo createReconnect(String url, WebSocket webSocket) {
        return mWebSocketInfoPool.obtain(url)
                .setWebSocket(webSocket)
                .setReconnect(true);
    }

    private WebSocketInfo createPrepareReconnect(String url) {
        return mWebSocketInfoPool.obtain(url)
                .setPrepareReconnect(true);
    }

    private WebSocketInfo createReceiveStringMsg(String url, WebSocket webSocket, String stringMsg) {
        return mWebSocketInfoPool.obtain(url)
                .setConnect(true)
                .setWebSocket(webSocket)
                .setStringMsg(stringMsg);
    }

    private WebSocketInfo createReceiveByteStringMsg(String url, WebSocket webSocket, ByteString byteMsg) {
        return mWebSocketInfoPool.obtain(url)
                .setConnect(true)
                .setWebSocket(webSocket)
                .setByteStringMsg(byteMsg);
    }

    private WebSocketInfo createClose(String url) {
        return mWebSocketInfoPool.obtain(url);
    }
}
```

#### 定时发送心跳维持连接

因为WebSocket断线后，后端不能马上知道连接已经断开，所以需要一个心跳消息保持双方通信。

实现心跳，本质就是一个定时消息，我们使用RxJava的interval操作符定时执行任务，这里我的消息需要增加一个时间戳，所以我加上了timestamp操作符来给每一次执行结果附加一个时间戳。

- 心跳信息json的生成，我们期望外部进行生成，例如Gson序列化为Json，或者FastJson处理，或者增加其他通用参数等，不应该在WebSocket基础库中写，所以提供了一个HeartBeatGenerateCallback回调进行生成Json。

1. 这里我做了一个优化，当网络没有开启时，则不发送心跳消息。

```
public class NetworkUtil {
    private NetworkUtil() {
    }

    /**
     * 当前是否有网络状态
     *
     * @param context  上下文
     * @param needWifi 是否需要wifi网络
     */
    public static boolean hasNetWorkStatus(Context context, boolean needWifi) {
        NetworkInfo info = getActiveNetwork(context);
        if (info == null) {
            return false;
        }
        if (!needWifi) {
            return info.isAvailable();
        } else if (info.getType() == ConnectivityManager.TYPE_WIFI) {
            return info.isAvailable();
        }
        return false;
    }

    /**
     * 获取活动网络连接信息
     *
     * @param context 上下文
     * @return NetworkInfo
     */
    public static NetworkInfo getActiveNetwork(Context context) {
        ConnectivityManager mConnMgr = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        if (mConnMgr == null) {
            return null;
        }
        // 获取活动网络连接信息
        return mConnMgr.getActiveNetworkInfo();
    }
}
```

2. 心跳回调接口，让外部生成心跳json

```
public interface HeartBeatGenerateCallback {
    /**
     * 当需要生成心跳信息时回调
     *
     * @param timestamp 当前时间戳
     * @return 要发送的心跳信息
     */
    String onGenerateHeartBeatMsg(long timestamp);
}
```

3. 发送心跳消息，需要制定Url地址、间隔时间，间隔时间单位，心跳消息生成回调。

```
@Override
public Observable<Boolean> heartBeat(String url, int period, TimeUnit unit,
                                     HeartBeatGenerateCallback heartBeatGenerateCallback) {
if (heartBeatGenerateCallback == null) {
    return Observable.error(new NullPointerException("heartBeatGenerateCallback == null"));
}
return Observable
        .interval(period, unit)
        //timestamp操作符，给每个事件加一个时间戳
        .timestamp()
        .retry()
        .flatMap(new Function<Timed<Long>, ObservableSource<Boolean>>() {
            @Override
            public ObservableSource<Boolean> apply(Timed<Long> timed) throws Exception {
                long timestamp = timed.time();
                //判断网络，存在网络才发消息，否则直接返回发送心跳失败
                if (mContext != null && NetworkUtil.hasNetWorkStatus(mContext, false)) {
                    String heartBeatMsg = heartBeatGenerateCallback.onGenerateHeartBeatMsg(timestamp);
                    Logger.d(TAG, "发送心跳消息: " + heartBeatMsg);
                    if (hasWebSocketConnection(url)) {
                        return send(url, heartBeatMsg);
                    } else {
                        //这里必须用异步发送，如果切断网络，再重连，缓存的WebSocket会被清除，此时再重连网络
                        //是没有WebSocket连接可用的，所以就需要异步连接完成后，再发送
                        return asyncSend(url, heartBeatMsg);
                    }
                } else {
                    Logger.d(TAG, "无网络连接，不发送心跳，下次网络连通时，再次发送心跳");
                    return Observable.create(new ObservableOnSubscribe<Boolean>() {
                        @Override
                        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
                            emitter.onNext(false);
                        }
                    });
                }
            }
        });
}
```

#### 实现重连

重连配置RxJava，有个天然优势就是RxJava提供了Retry操作符，支持重试，我们在onFailure()连接失败回调中手动发出onError()，让数据源增加retry操作符进行重试，就会重新走到数据源的订阅回调重新连接WebSocket。

```
private final class WebSocketOnSubscribe implements ObservableOnSubscribe<WebSocketInfo> {
        private String mWebSocketUrl;
        private WebSocket mWebSocket;
        private boolean isReconnecting = false;

        public WebSocketOnSubscribe(String webSocketUrl) {
            this.mWebSocketUrl = webSocketUrl;
        }

        @Override
        public void subscribe(ObservableEmitter<WebSocketInfo> emitter) throws Exception {
            //...
        }

        private Request createRequest(String url) {
            return new Request.Builder().get().url(url).build();
        }

        /**
         * 初始化WebSocket
         */
        private synchronized void initWebSocket(ObservableEmitter<WebSocketInfo> emitter) {
            if (mWebSocket == null) {
                mWebSocket = mClient.newWebSocket(createRequest(mWebSocketUrl), new WebSocketListener() {
                    @Override
                    public void onOpen(WebSocket webSocket, Response response) {
                        super.onOpen(webSocket, response);
                        //连接成功
                        if (!emitter.isDisposed()) {
                            mWebSocketPool.put(mWebSocketUrl, mWebSocket);
                            //重连成功
                            if (isReconnecting) {
                                emitter.onNext(createReconnect(mWebSocketUrl, webSocket));
                            } else {
                                emitter.onNext(createConnect(mWebSocketUrl, webSocket));
                            }
                        }
                        isReconnecting = false;
                    }

                    @Override
                    public void onMessage(WebSocket webSocket, String text) {
                        super.onMessage(webSocket, text);
                        //收到消息
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createReceiveStringMsg(mWebSocketUrl, webSocket, text));
                        }
                    }

                    @Override
                    public void onMessage(WebSocket webSocket, ByteString bytes) {
                        super.onMessage(webSocket, bytes);
                        //收到消息
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createReceiveByteStringMsg(mWebSocketUrl, webSocket, bytes));
                        }
                    }

                    @Override
                    public void onClosed(WebSocket webSocket, int code, String reason) {
                        super.onClosed(webSocket, code, reason);
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createClose(mWebSocketUrl));
                        }
                    }

                    @Override
                    public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
                        super.onFailure(webSocket, throwable, response);
                        isReconnecting = true;
                        mWebSocket = null;
                        //移除WebSocket缓存，retry重试重新连接
                        removeWebSocketCache(webSocket);
                        if (!emitter.isDisposed()) {
                            emitter.onNext(createPrepareReconnect(mWebSocketUrl));
                            //失败发送onError，让retry操作符重试
                            emitter.onError(new ImproperCloseException());
                        }
                    }
                });
            }
        }
    }
```

#### 使用

- 使用RxWebSocketBuilder，构建RxWebSocket

```
//自定义OkHttpClient
OkHttpClient mClient = new OkHttpClient.Builder()
        .readTimeout(3, TimeUnit.SECONDS)//设置读取超时时间
        .writeTimeout(3, TimeUnit.SECONDS)//设置写的超时时间
        .connectTimeout(3, TimeUnit.SECONDS)//设置连接超时时间
        .build();

//RxWebSocketBuilder构建RxWebSocket
RxWebSocket rxWebSocket = new RxWebSocketBuilder(context)
                //是否打印Log
                .isPrintLog(true)
                //5秒无响应则重连
                .reconnectInterval(5, TimeUnit.SECONDS)
                .client(mClient)
                .build();
```

- 连接Url地址

```
String url = "ws://xxxxxxxxx"
//开始连接
rxWebSocket.get(url)
	//切换到子线程去连接
	.compose(RxSchedulerUtil.ioToMain())
	//绑定生命周期
	.as(RxLifecycleUtil.bindLifecycle(mLifecycleOwner))
	.subscribe(new Consumer<WebSocketInfo>() {
	    @Override
	    public void accept(WebSocketInfo webSocketInfo) throws Exception {
	        String json = webSocketInfo.getStringMsg();
	        //业务层的json解析
	        ...
	    }
    });
```

- 同步发送消息（必须确保连接正常，否则发送失败）

```
rxWebSocket.send(url, "我是消息")
	.compose(RxSchedulerUtil.ioToMain())
	.as(RxLifecycleUtil.bindLifecycle(mLifecycleOwner))
	.subscribe(new Consumer<Boolean>() {
		    @Override
		    public void accept(Boolean isSuccess) throws Exception {
		        if(isSuccess) {
		        	  //发送成功
		        } ele {
		        	  //发送失败
		        }
		    }
	});
```

- 异步发送消息（不需要确保连接正常，如果未连接会连接成功后自动发送）

```
rxWebSocket.asyncSend(url, "我是消息")
	.compose(RxSchedulerUtil.ioToMain())
	.as(RxLifecycleUtil.bindLifecycle(mLifecycleOwner))
	.subscribe(new Consumer<Boolean>() {
		    @Override
		    public void accept(Boolean isSuccess) throws Exception {
		        if(isSuccess) {
		        	  //发送成功
		        } ele {
		        	  //发送失败
		        }
		    }
	});
```

- 发送心跳包

```
rxWebSocket.heartBeat(url, 6 ,TimeUnit.SECONDS, new HeartBeatGenerateCallback() {
    @Override
    public String onGenerateHeartBeatMsg(long timestamp) {
        //生成心跳Json，业务模块处理，例如后端需要秒值，我们除以1000换算为秒。
        //后续可以在这里配置通用参数等
        return GsonUtil.toJson(new HeartBeatMsgRequestModel(WssCommandTypeEnum.HEART_BEAT.getCode(),
                String.valueOf(timestamp / 1000)));
    }
});
```

#### 总结

Okhttp的WebSocket使用比较简单，基本都是发起请求和配置回调2个步骤，再使用send()方法发送消息。

但如果真正使用起来还需要做一层封装，可以配合RxJava将异步回调封装成Observable通知订阅者，并使用RxJava的各种操作符，例如数据转换、线程切换、连接重试和心跳等。