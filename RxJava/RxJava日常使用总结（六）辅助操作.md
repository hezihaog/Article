#### RxJava日常使用总结（六）辅助操作

本篇介绍RxJava的辅助操作。例如Delay延时，Doxx系列事件钩子，线程切换等。

#### delay操作符

延迟一段指定的时间再发射来自Observable的发射物。就是推迟指定发射Observable的事件。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-da2a534698398dfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable
.fromIterable(mItems)
.delay(1, TimeUnit.SECONDS)
.as(RxLifecycleUtil.bindLifecycle(this))
.subscribe();
```

#### do操作符

注册一个动作作为原始Observable生命周期事件的一种占位符。

- Observable发射的事件，我们可以在注册subscribe中处理，但有时候Observable是提供出去的，在哪里注册都是未知的，不可能每个subscribe的地方都写一套。这时候do操作符就用处了，do系列的操作符相当于事件的钩子，在执行时调用使用do系列的操作的回调。

#### doOnEach操作符

doOnEach操作符让你可以注册一个回调，它产生的Observable每发射一项数据就会调用它一次。参数是Notification。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-e0ff573d43e5804c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.fromIterable(mItems)
    .doOnEach(new Consumer<Notification<Object>>() {
        @Override
        public void accept(Notification<Object> notification) throws Exception {
            //...
        }
    })
    .as(RxLifecycleUtil.bindLifecycle(this))
    .subscribe();
```

#### doOnNext操作符

doOnNext操作符类似于doOnEach(Action1) ，但是它的Action不是接受一个Notification参数，而是接受发射的数据项。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-047ec7c4ae349b0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.fromIterable(mItems)
    .doOnNext(new Consumer<Object>() {
        @Override
        public void accept(Object o) throws Exception {
            L.d(o.toString());
        }
    })
    .as(RxLifecycleUtil.bindLifecycle(this))
    .subscribe();
```

#### doOnSubscribe操作符

doOnSubscribe操作符注册一个动作，当观察者订阅它生成的Observable它就会被调用。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d927a601c7ccf263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 注册时就调用，例如接口请求的Observable，注册时就弹出等待框。

```
Observable.fromIterable(mItems)
    .doOnSubscribe(new Consumer<Disposable>() {
        @Override
        public void accept(Disposable disposable) throws Exception {
        	  //准备开始
            showLoading();
        }
    })
    .as(RxLifecycleUtil.bindLifecycle(this))
    .subscribe();
```

#### doOnComplete操作符

![image.png](https://upload-images.jianshu.io/upload_images/1641428-8aa2f8441d13a81d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注册一个完成回调

```
Observable.fromIterable(mItems)
        .doOnComplete(new Action() {
            @Override
            public void run() throws Exception {
                //完成...
            }
        })
        .as(RxLifecycleUtil.bindLifecycle(this))
        .subscribe();
```

#### doOnError操作符

![image.png](https://upload-images.jianshu.io/upload_images/1641428-263ba8f1b49d6960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注册一个出错回调

```
Observable.fromIterable(mItems)
    .doOnError(new Consumer<Throwable>() {
        @Override
        public void accept(Throwable throwable) throws Exception {
            //异常了
        }
    })
    .as(RxLifecycleUtil.bindLifecycle(this))
    .subscribe();
```

#### doTerminate操作符

doTerminate操作符注册一个动作，当它产生的Observable终止之前会被调用，无论是正 常还是异常终止。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d8164a788a1c0bbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.fromIterable(mItems)
        .doOnTerminate(new Action() {
            @Override
            public void run() throws Exception {
                //...准备结束了
            }
        })
        .as(RxLifecycleUtil.bindLifecycle(this))
        .subscribe();
```

#### doAfterTerminate操作符

doAfterTerminate操作符注册一个动作，当它产生的Observable终止之后会被调用，无论是正常还 是异常终止。

- 结束时调用，接口请求成功或失败后，隐藏弹窗

```
Observable.fromIterable(mItems)
        .doAfterTerminate(new Action() {
            @Override
            public void run() throws Exception {
            	   //已经结束
                hideLoading();
            }
        })
        .as(RxLifecycleUtil.bindLifecycle(this))
        .subscribe();
```

#### observeOn和subscribeOn操作符

![observeOn.png](https://upload-images.jianshu.io/upload_images/1641428-4027c20507d2afe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![subscribeOn.png](https://upload-images.jianshu.io/upload_images/1641428-f2abc198d3b7011b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ObserveOn：指定一个观察者在哪个调度器上观察这个Observable。可以说就是任务结束时进行回调的线程，而Android一般都是主线程。

SubscribeOn：指定Observable自身在哪个调度器上执行。可以说是耗时操作指定的线程，一般为IO线程或computation计算线程。

- Android的Handler的Scheduler一般都使用RxAnroid上提供的，怎么引用就不说了，看Github。例如

```
Observable.fromIterable(mItems)
		  //Observable执行在子线程，所以在子线程遍历
        .subscribeOn(Schedulers.io())
         //回调线程，在主线程
        .observeOn(AndroidSchedulers.mainThread())
        .as(RxLifecycleUtil.bindLifecycle(this))
        .subscribe();
```

#### timeout操作符

对原始Observable的一个镜像，如果过了一个指定的时长仍没有发射数据，它会发一个错误 通知。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-529c5f776bdca485.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一般用来定时检查，Observable指定时间内时候发送过事件，超过时间发送一个onError()，异常对象为TimeoutException。

- 例如封装WebSocket的Observable，指定时间内没有发出事件，发出onError()事件，再配合retry重试，尝试重新连接WebSocket。

```
Observable
        .create(new WebSocketOnSubscribe(url))
        //如果数据源指定之间内没有发出消息，会发送一个超时异常，配合retry吃掉这个异常后，重试
        .timeout(timeout, timeUnit)
        .retry()
        //将回调都放置到主线程回调，外部调用方直接观察，实现响应回调方法做UI处理
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
```

#### timestamp操作符

给Observable发射的数据项附加一个时间戳

![image.png](https://upload-images.jianshu.io/upload_images/1641428-88ede05e80066690.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如WebSocket实践中，后端要求我们定时多少秒发送一个WebSocket心跳消息。里面需要带一个时间戳，就可以使用timestamp操作符，包装数据为Timed对象。（内部其实就是map操作符，将数据包装在Timed对象，Timed对象中有个time字段为当前时间戳）

```
/**
* 发送心跳包
*/
public Observable<Boolean> sendHeartBeatMsg() {
return getRxWebSocket().heartBeat(getUrl(),
        AskTeacherConstant.CONSULTING_ROOM_PING_MSG_INTERVAL_TIME,
        TimeUnit.SECONDS, new HeartBeatGenerateCallback() {
            @Override
            public String onGenerateHeartBeatMsg(long timestamp) {
                return GsonUtil.toJson(new HeartBeatMsgRequestModel(WssCommandTypeEnum.HEART_BEAT.getCode(),
                        String.valueOf(timestamp / 1000)));
            }
        });
}

@Override
public Observable<Boolean> heartBeat(String url, int period, TimeUnit unit,HeartBeatGenerateCallback heartBeatGenerateCallback) {
	return Observable
	.interval(period, unit)
	//timestamp操作符，给每个事件加一个时间戳
	.timestamp()
	.retry()
	.flatMap(new Function<Timed<Long>, ObservableSource<Boolean>>() {
		    @Override
		    public ObservableSource<Boolean> apply(Timed<Long> timed) throws Exception {
		        long timestamp = timed.time();
				  String heartBeatMsg = heartBeatGenerateCallback.onGenerateHeartBeatMsg(timestamp);
		            Logger.d(TAG, "发送心跳消息: " + heartBeatMsg);
		            return send(url, heartBeatMsg);
		    }
		});
}
```

#### serialize操作符
强制一个Observable连续调用并保证行为正确。

Observable发射事件的线程是多个不同子线程（异步）进行发射，就可能造成事件混乱。可能onNext()、onError、onComplete顺序不是正确的，使用serialize操作符能使事件按同步顺序返回。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-03b9828b09ccb2ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("wally");
        emitter.onNext("wally");
        emitter.onComplete();
        emitter.onNext("wally");
    }
})
.serialize()
.subscribe();

//结果
//wally
//wally
//wally
//onComplete
```