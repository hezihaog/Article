#### RxJava日常使用总结（五）错误处理

说到异常处理，Java有try-catach，那在RxJava的世界里，怎么捕获和处理异常呢？一起来看下吧~

#### onErrorReturn操作符
让Observable遇到错误时发射一个特殊的项并且正常终止。其实就是当发生错误时，提供一个回调，返回一个出错时使用的值。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d8b98f5fd7887e19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如订单列表，需要设置一个价格文本，price字段，为了保证价格在json中不会丢失精度，一般会用String类型去存在，再转换为Double等数据类型，但是可能在后端出现了某种原因返回了非数字时，想Double.valueOf(price)肯定会抛出异常，那么RxJava如何处理呢？

```
Observable.just(itemModel.getAskMoney())
                .map(new Function<String, Double>() {
                    @Override
                    public Double apply(String priceStr) throws Exception {
                        return Double.valueOf(priceStr);
                    }
                })
                .onErrorReturn(new Function<Throwable, Double>() {
                    @Override
                    public Double apply(Throwable throwable) throws Exception {
                        return 0.0;
                    }
                })
                .subscribe(new Consumer<Double>() {
                    @Override
                    public void accept(Double price) throws Exception {
                        //价格设置
                        String priceStr = context.getResources().getString(R.string.base_money_text, price);
                        holder.vPrice.setText(priceStr);
                    }
                });
```

- 在map操作符转换数据类型时，可能会抛出异常，RxJava2在map中已经直接加了try-catch捕获，所以这里直接抛出，我们不用做处理，onErrorReturn()操作符，要求返回一个异常后的返回值，例如我们直接返回0.0，代表默认值。

#### onErrorResumeNext操作符
方法返回一个镜像原有Observable行为的新Observable，后者会忽略前者的onError调用，不会将错误传递给观察者，作为替代，它会开始镜像另一个备用的Observable。一句话总结，onErrorReturn操作符返回的是具体值，而onErrorResumeNext则是返回Observable包裹的值，相当于转换成了另外一个Observable，简单理解就是类似map和flatMap。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-b33cef698b0a8acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 像到淘宝的首页，条目类型很多，而且样式也不一致，顶部轮播图，中间网格，底部瀑布流。一般后端不会一个接口给完（因为他们查的表，一般会不一样），所以在使用RxJava时，一般都会使用concatMap操作符连接多个顺序接口，再toList聚合在一起，一次性发送，再渲染界面。

- 例如项目中一个模块首页，顶部轮播图，在一个分类网格，再是一个广告列表，最后是一个老师列表。因为RxJava的链式串联，任何一个串联的Observable数据源出现error都会整个链断裂，整个数据就会没了，像广告这种接口，如果发生错误也不应该影响整个界面，所以使用onErrorResumeNext操作符，返回Observable.empty()，让发生error的这条数据源正常结束，即可不影响整个串联调用链。

```
Observable.concatDelayError(
                Arrays.asList(
                        getMainBanner(context, tag),
                        getTeacherServerCategory(context, tag),
                        //广告接口，不太重要，如果错误，就不显示
                        AskTeacherAdManager.getAdList(context).onErrorResumeNext(Observable.empty())
                        , getAllTeacherList(context, tag, page)
                ))
                .collect(new Callable<List<HttpModel<?>>>() {
                    @Override
                    public List<HttpModel<?>> call() throws Exception {
                        return new ArrayList<>();
                    }
                }, new BiConsumer<List<HttpModel<?>>, HttpModel<?>>() {
                    @Override
                    public void accept(List<HttpModel<?>> list, HttpModel<?> listModel) throws Exception {
                        list.add(listModel);
                    }
                }).toObservable();
```

#### onExceptionResumeNext操作符

和onErrorResumeNext类似，也是接收一个备用的Observable，但是和onErrorResumeNext的区别是，如果产生的异常对象不是Exception，不会进行捕获，也不会使用备用的Observable，会将异常依然传入到原来的Observable，调用它的onError。所以如果异常对象是Throwable，则不会被捕获到，例如我们的OutOfMemoryError，不是Exception的子类而是Throwable的子类。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-3925be26eac527b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 使用和onErrorResumeNext一毛一样的，例如我们需要加载一个图片Url，生成Bitmap，当这个Bitmap内存占用真的超过的可分配的内存时，产生OOM了，使用onErrorResumeNext来不捕获异常，让原来的Observable数据源进行onError回调的处理。其他异常，例如可能在拉取图片途中，网速或者某种原因，发生了TimeoutException，连接超时异常，这时候就会被捕获调，再进行相应的处理，例如重试。

```
Observable.create(new ObservableOnSubscribe<Bitmap>() {
    @Override
    public void subscribe(ObservableEmitter<Bitmap> emitter) throws Exception {
        try {
            //可能抛出OutOfMemoryError
            Bitmap bitmap = loadImageUrlToBitmap();
            if (bitmap != null) {
                emitter.onNext(bitmap);
            } else {
                emitter.onError(new NullPointerException("load image url bitmap is null"));
            }
        } catch (TimeoutException e) {
            //正常是不需要try-catch再抛出的，subscribe是允许抛出Exception被内部调用时捕获调用onError()，这里只是演示
            e.printStackTrace();
            emitter.onError(e);
        }
    }
})
        //这里只会处理抛出是Exception的情况，OutOfMemoryError是Throwable，不会被捕获，会抛出到源Observable
        .onExceptionResumeNext(Observable.create(new ObservableOnSubscribe<Bitmap>() {
            @Override
            public void subscribe(ObservableEmitter<Bitmap> emitter) throws Exception {
                //这里加载占位图drawable，生成Bitmap
                Bitmap placeholderBitmap = BitmapFactory.decodeResource(mContext.getResources().getDrawable(R.drawable.error_placeholder));
                emitter.onNext(placeholderBitmap);
            }
        }));
```

#### retry操作符
如果原始Observable遇到错误，重新订阅它期望它能正常终止。就是Observable发生异常时（发送onError），捕获，并重新调用Observable的subject订阅方法，重新订阅。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-94b2aa1c57820029.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- retry()，只要发生错误，就重新订阅，无限重试。

例如WebSocket连接，发生异常抛出异常，或我们手动调用发送onError()，因为retry操作符的存在，Observable会被重订阅，重订阅

```
Observable.create(new WebSocketOnSubscribe(url))
        //无限重试
        .retry()
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
        
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
                            //重连成功
                            if (isReconnecting) {
                                emitter.onNext(createReconnect(webSocket));
                            } else {
                                emitter.onNext(createConnect(webSocket));
                            }
                        }
                        isReconnecting = false;
                    }
                    
                    //...省略其他重写方法

                    @Override
                    public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
                        super.onFailure(webSocket, throwable, response);
                        isReconnecting = true;
                        mWebSocket = null;
                        //移除WebSocket缓存，retry重试重新连接
                        removeWebSocketCache(webSocket);
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
```

- retry(int count)，设置一个重试次数，超过次数就不重试了。

如果要限定重试次数，retry操作符传入次数即可。例如retry(100)，重试100次后不重试，将onError()发送给Observable。

```
Observable.create(new WebSocketOnSubscribe(url))
        //重试100后停止重试
        .retry(100)
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());

//后面数据源订阅部分一致，就补贴了...
```

- retry(new Predicate<Throwable>())，接收一个Predicate谓词接口实现，相当于设置一个重试条件，返回true则重试，false则不重试。

如果需要判断指定异常才捕获，就可以使用retry(new Predicate<Throwable>())

```
Observable.create(new WebSocketOnSubscribe(url))
        //重试100后停止重试
        .retry(new Predicate<Throwable>() {
                        @Override
                        public boolean test(Throwable throwable) throws Exception {
                            return throwable instanceof ImproperCloseException;
                        }
                    })
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
```

#### retryUntil

和retry(new Predicate<Throwable>())类似，也是提供一个重试条件，但是返回值是相反的，返回true不重试，返回false重试。

```
Observable.create(new WebSocketOnSubscribe(url))
        //重试100后停止重试
        .retryUntil(new BooleanSupplier() {
                        @Override
                        public boolean getAsBoolean() throws Exception {
                            //有网络，则停止重试，无网络则一直重试
                            return NetworkUtil.hasNetWorkStatus(mContext, false);
                        }
                    })
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
```

#### retryWhen

和上面的retry和retryUntil不一样，retryWhen需要返回一个Observable数据源，这个Observable发出onError则会继续走重试，除非是onNext或者onComplete。使用Observable则可以做一系列转换的事情。例如出错了，返回的Observable中去判断网络环境，或者调用结果查询或查询数据库等操作。（Token过期的401也可以使用retryWhen去处理，flatMap去用RefreshToken刷新AssessToken）

![image.png](https://upload-images.jianshu.io/upload_images/1641428-cbe9e6bafdc537d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.create(new WebSocketOnSubscribe(url))
		 .retryWhen(new Function<Observable<Throwable>, ObservableSource<WebSocketInfo>>() {
                        @Override
                        public ObservableSource<WebSocketInfo> apply(Observable<Throwable> throwableObservable) throws Exception {
                            return throwableObservable.flatMap(new Function<Throwable, ObservableSource<WebSocketInfo>>() {
                                @Override
                                public ObservableSource<WebSocketInfo> apply(Throwable throwable) throws Exception {
                                    //非正常关闭的异常，重试
                                    if (throwable instanceof ImproperCloseException) {
                                        return Observable.just(createPrepareReconnect());
                                    } else {
                                        //其他异常，继续将异常交给原Observable去处理
                                        return Observable.error(throwable);
                                    }
                                }
                            });
                        }
                    })
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
```