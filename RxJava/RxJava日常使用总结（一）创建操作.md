#### RxJava日常使用总结（一）创建操作

#### 为什么要记录

- 在项目中已经用了好几个月RxJava，越用越不离手，特此记录一下，用到的一些RxJava片段。

- Observable，实现了ObservableSource接口，被观察者，我一般叫他为数据源，Observable会源源不断发出事件。类似流水线，不断生产零部件，经过多个Operators，相当于流水线的工人，不断给这些零件加工，最后交到我们的被观察者Observer。

#### 使用Create创建

从头创建一个Observable，其实这个不是经常使用，一般都会使用RxJava的几个工厂型的静态方法创建Observable，例如Just，fromIterable等，我只在特定需要定制返回结果时使用，例如在使用RxJava封装WebSocket时，外部监听WebSocket的事件，则封装了一个WebSocketInfo消息模型时，使用到了Create操作符。
![image.png](https://upload-images.jianshu.io/upload_images/1641428-0fed29c0fdf2aafd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Observable.create()，需要传入一个ObservableOnSubscribe参数，可以理解为Observable被监听时的一个回调，实现subscribe()方法，该方法在被观察者观察时回调，方法传入一个ObservableEmitter的形参，代表一个发射器，调用emitter的onNext()发射数据，onError()发射异常，onComplete()发射完成事件，注意onError()和onComplete()只能有一个发送，发送后就代表这条数据源结束。

```
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
						emitter.onNext(createConnect(webSocket));
                    }
                    isReconnecting = false;
                }

                @Override
                public void onMessage(WebSocket webSocket, String text) {
                    super.onMessage(webSocket, text);
                    //收到消息
                    if (!emitter.isDisposed()) {
                        emitter.onNext(createReceiveStringMsg(webSocket, text));
                    }
                }

                @Override
                public void onMessage(WebSocket webSocket, ByteString bytes) {
                    super.onMessage(webSocket, bytes);
                    //收到消息
                    if (!emitter.isDisposed()) {
                        emitter.onNext(createReceiveByteStringMsg(webSocket, bytes));
                    }
                }

                @Override
                public void onClosed(WebSocket webSocket, int code, String reason) {
                    super.onClosed(webSocket, code, reason);
                    if (!emitter.isDisposed()) {
                        emitter.onNext(createClose());
                    }
                }

                @Override
                public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
                    super.onFailure(webSocket, throwable, response);
					//...
                    if (!emitter.isDisposed()) {
                        emitter.onNext(createPrepareReconnect());
                        //失败发送onError，让retry操作符重试
                        emitter.onError(new ImproperCloseException());
                    }
                }
            });
        }
    }
}
    
//最后做注册等操作
Observable.create(new WebSocketOnSubscribe(url))
.subscribe()...
```

#### Empty操作符，创建一个不发事件，直接发送onComplete()事件的数据源。

看似没什么用，一开始我也是这么觉得的，不发onNext()，没有事件输出，那这条数据源有什么用？直到我使用它做了一件事，才知道，原来是可以这么用的~

- 项目中有一个搜索功能，界面很常见，第一个Item是热门搜索，第二Item是历史记录。2个都是异步操作，所以肯定要开子线程，一异步操作就肯定有接口回调，多个接口回调就容易形成回调地域，RxJava就能化解这种情况。多个数据联合我们会用到concat()操作符，这里先简单理解为将多个数据源的发射的数据按顺序返回。

- 所以就有了以下代码~

```
Observable. concat(
//热门搜索
mClient.requestTagsInObservable(TAG)
        .filter(new Predicate<HttpModel<List<SearchTagModel>>>() {
            @Override
            public boolean test(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                return httpModel.getData() != null && httpModel.getData().size() > 0;
            }
        })
        .map(new Function<HttpModel<List<SearchTagModel>>, BaseSearchModel>() {
            @Override
            public BaseSearchModel apply(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                return new HotSearchModel(httpModel.getData());
            }
        }), //本地历史
mClient.getHistoryTagsInObservable()
.filter(new Predicate<List<SearchTagModel>>() {
    @Override
    public boolean test(List<SearchTagModel> tagModels) throws Exception {
        return tagModels.size() > 0;
    }
})
//线程切换，子线程执行，主线程回调
.compose(RxSchedulerUtil.ioToMain())
.map(new Function<List<SearchTagModel>, HistorySearchModel>() {
    @Override
    public HistorySearchModel apply(List<SearchTagModel> tagModels) throws Exception {
        mHistoryTagList.clear();
        mHistoryTagList.addAll(tagModels);
        return new HistorySearchModel(tagModels);
    }
})
//聚合在一起为List接口再一起发射
.toList()
```

- 最后满心欢喜，的确能实现~结果一关网络，纳尼？怎么全没了，debug一看，回调到了error的回调，一看Throwable对象，则是AddressNoFound的异常，才恍然大悟，没有网络，连不放地址，抛异常直接为Rx抓到异常直接回调了！

经过查看Rx的文档，发现Rx提供了异常处理的操作，例如这里用到的onErrorResumeNext()，意识就是当抛出onError()时，拦截掉，再回调onErrorResumeNext()，onErrorResumeNext()传入一个ObservableSource对象，其实就是Observable，就是一个数据源，那我么那就可以使用Observable.empty()，创建一个正常结束的Observable，来结束这个热门搜索的Observable，这样就能保证最后数据中就算没有热门搜索，也肯定有本地历史~

```
Observable. concat(
//热门搜索
mClient.requestTagsInObservable(TAG)
        .filter(new Predicate<HttpModel<List<SearchTagModel>>>() {
            @Override
            public boolean test(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                return httpModel.getData() != null && httpModel.getData().size() > 0;
            }
        })
        .map(new Function<HttpModel<List<SearchTagModel>>, BaseSearchModel>() {
            @Override
            public BaseSearchModel apply(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                return new HotSearchModel(httpModel.getData());
            }
        }), //本地历史
mClient.getHistoryTagsInObservable()
.filter(new Predicate<List<SearchTagModel>>() {
    @Override
    public boolean test(List<SearchTagModel> tagModels) throws Exception {
        return tagModels.size() > 0;
    }
})
//线程切换，子线程执行，主线程回调
.compose(RxSchedulerUtil.ioToMain())
.map(new Function<List<SearchTagModel>, HistorySearchModel>() {
    @Override
    public HistorySearchModel apply(List<SearchTagModel> tagModels) throws Exception {
        mHistoryTagList.clear();
        mHistoryTagList.addAll(tagModels);
        return new HistorySearchModel(tagModels);
    }
})
.onErrorResumeNext(Observable.empty())
//聚合在一起为List接口再一起发射
.toList()
```

#### never\error操作符，和empty类似，empty为只发送一个onComplete()事件，正常结束序列，never为创建一个不发送事件也不结束的数据源，error则是创建一个只发送onError()事件的数据源。

- never项目中还没用到过，error()倒是有，例如flatMap中，我们判断到传入的数据为null，需要返回一个Observable，则可以使用Observable.error()生成一个发送error事件的数据源，来发送onError()。

#### from操作符。
就是前面说create()方法时的工厂方法，from操作符有多个变种

![image.png](https://upload-images.jianshu.io/upload_images/1641428-1375d7ce203b3f24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- fromIterable()，传入一个Iterable迭代器接口，最常用就是传入List集合对象，它的作用就是遍历，就是forEach。一般我会使用它来配合flatMap操作符来发送多次事件。它的作用和create()其实是一样的，create()需要我们手动控制什么时候onNext()，什么时候onComplete。其实一般情况，我们不需要每次都控制得这么仔细，只需要正常发送，正常结束，这时候from系列的操作符就是一个很好的帮手~那from操作符什么时候onNext()，什么时候onComplete呢？其实很显而易见了，遍历时onNext，遍历结束后onComplete~

例如我遇到的一个很常见的场景，多张图片压缩，一起压缩，再一张张上传，最后输出一批返回结果~

```
Observable.just(filePaths)
.flatMap(new Function<List<String>, ObservableSource<String>>() {
    @Override
    public ObservableSource<String> apply(List<String> strings) throws Exception {
        //1、压缩图片，得到压缩文件的path，遍历
        List<String> compressFilePaths = Flora
                .with(activity)
                .bitmapConfig(Bitmap.Config.ARGB_8888)
                .compressTaskNum(filePaths.size())
                .load(filePaths)
                .compressSync();
        return Observable.fromIterable(compressFilePaths);
    }
})
.flatMap(new Function<String, ObservableSource<Images>>() {
    @Override
    public ObservableSource<Images> apply(String path) throws Exception {
        //2、对每张压缩图片进行上传
        return startUploadImage(tag, path, otherParams);
    }
})
//3、结果堆积为List
.toList()
.toObservable();
```

- fromArray，传入一个数组，和fromIterable作用一样，只是参数为数组。

- fromFuture\fromCallable，Java多线程框架中提供的有参返回值的接口，对应无返回值就是我们的Runnable接口，我很少使用，这里不细讲。

#### interval操作符，按固定时间间隔，发送整数序列的Observable。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-2eb4c168c07aaaf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


很常见的一个场景，定时器！或者是倒计时功能，还记得我们用Handler来倒计时的日子吗~

```
/**
 * Rx倒计时
 *
 * @param startDelayTime 开始前的延时时间，例如开始前有1秒缓冲
 * @param cycle          周期，每隔多久重复执行，例如1秒执行一次的倒计时功能
 * @param time           执行多久，例如倒计时3秒，则为3
 * @param unit           时间单位，倒计时3秒，单位为秒
 */
public static Observable<Integer> countdown(long startDelayTime, long cycle, int time, TimeUnit unit) {
    final int countTime = time < 0 ? 0 : time;
    return Observable.interval(startDelayTime, cycle, unit)
            .subscribeOn(AndroidSchedulers.mainThread())
            .observeOn(AndroidSchedulers.mainThread())
            .map(new Function<Long, Integer>() {
                @Override
                public Integer apply(Long increaseTime) throws Exception {
                    return countTime - increaseTime.intValue();
                }
            })
            //take指定到多少次就停止，这里指定到时间后就结束
            .take(countTime + 1);
}
```

#### timer操作符。
延时执行，效果就是new Handler().postDelayed()。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-96269e5fcfb6368d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
/**
 * 延时操作
 */
public static Observable<Long> delayed(int time, TimeUnit unit) {
    return Observable.timer(time, unit);
}
```

#### Just操作符

相比from系列，最常用就是Just了，相当于发送一个值，或多个值，Just有多个重载方法，最多支持10个，效果相当于调用onNext()发送传入的数据，发送完毕再调用onComplete()。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-a552c05831075bef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 还是上传图片的例子，我们一般会现有一个文件的file地址，要开启一个数据源可以直接使用just发送这个file的path，再map、flatMap转换，就不需要使用create()操作符了，create在这种简单场景真的没必要~

```
Observable
	.just(path)
	.map(new Function<String, File>() {
	    @Override
	    public File apply(String path) throws Exception {
	        return new File(path);
	    }
	})
	.map(new Function<File, File>() {
	    @Override
	    public File apply(File file) throws Exception {
	        String compressFile = Flora.with(activity)
	                .bitmapConfig(Bitmap.Config.ARGB_8888)
	                .load(file).compressSync();
	        return new File(compressFile);
	    }
	})...
```

#### range操作符。
指定开始位置和结束位置，发射这个区间的Int或Long值，这个操作符我不是很多用，不过range可以用来结合遍历，range(0, list.size-1)，再结合zip操作符来配对组合，就可以拿到遍历时的位置，为什么这么干呢？因为有时候需要过滤某个位置的数据时，fromIterable只能拿到数据不能拿到遍历的角标，所以这种情况，我们可以range配合zip配合使用。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d190bc68469806bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如遍历一个Items集合，过滤角标为15的数据，最后输出~

```
//包裹数据和角标位置的模型，一般用于Rx，遍历数据，过滤指定类型数据后，需要用到数据和角标的情况
public class DataIndexModel<T> {
    private T data;
    private int index;

    public DataIndexModel(T data, int index) {
        this.data = data;
        this.index = index;
    }

    public T getData() {
        return data;
    }

    public int getIndex() {
        return index;
    }
}
```

```
Observable.zip(Observable.range(0, mItems.size() - 1), Observable.fromIterable(mItems),
        new BiFunction<Integer, Object, DataIndexModel<Object>>() {
    @Override
    public DataIndexModel<Object> apply(Integer index, Object data) throws Exception {
        return new DataIndexModel<>(data, index);
    }
})
        .filter(new Predicate<DataIndexModel<Object>>() {
            @Override
            public boolean test(DataIndexModel<Object> model) throws Exception {
                return model.getIndex() != 15;
            }
        })
        .map(new Function<DataIndexModel<Object>, Object>() {
            @Override
            public Object apply(DataIndexModel<Object> model) throws Exception {
                return model.getData();
            }
        })
        .subscribe();
```

#### repeat操作符。
指定重复次数，重复发送数据源的数据。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-df8873a696f6ea39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如发送1，2，3，3个数据，鬼畜5次，哈哈~平时可能有这种需求，发送一批，数据重复发送多少次，再结束。

```
Observable.fromArray(1,2,3)
            .repeat(5)
            .subscribe();
```