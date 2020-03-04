#### RxJava日常使用总结（九）连接操作

本篇介绍的是RxJava的连接操作。说之前，先来普及一下，Observable的种类。纳尼？Observable还有种类？一直没用过其他种类的Observable呀？

#### Observable的种类

- Cool Observable（冷的被观察者）其实捏，我们平时用的Observable都是Cool Observable，有什么特点呢？

	1. Cool Observable必须等到有Observer去subscribe（订阅）才发送数据。
	2. Cool Observable对每个观察者都是独立的一个发送通道，即使他们的数据是一样的。即使这个Observable已经发送了很多事件，再来一个Observer进行subscribe，所有的数据都会重放给这个新来的Observer。（类似Android中的粘性广播）

- Hot Observable（热的被观察者），相比Cool Observable，又有什么区别呢？
	1. Hot Observable，发送数据需要一个Connect操作符才开始发送事件，所以它更灵活，可以手动控制什么时候开始发送。如果不调用，则一直都不会发送（就算有再多的Observer订阅），还有就是Hot Observable就算没有观察者也可以发送数据，而Cool Observable需要等待到观察者订阅才发送。
	2. Hot Observable对每个观察者发送的事件是“共享”的，所以如果Hot Observable已经发送了很多事件，再来一个Observer进行subscribe，这个Observer会收不到之前发送的事件。（类似Android中的普通广播）

- Hot Observable也称作为可连接的Observable。Cool Observable就称作为普通的Observable

#### publish和connect操作符

Publish和Connect一般一起使用，所以就一起介绍了

Publish：将普通的Observable转换为可连接的Observable。
Connect：让一个可连接的Observable开始发射数据给订阅者。Observable调用Connect后，返回一个Disposable，使用Disposable可让Observable终止。（不调用就不用终止，就算所有观察者都取消注册，它依然不会终止，除非手动调用）

![image.png](https://upload-images.jianshu.io/upload_images/1641428-8582b77ce9db7364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
ConnectableObservable<Long> observable = Observable.interval(1, TimeUnit.SECONDS)
            //转换为可连接的Observable
            .publish();
    observable.subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long value) throws Exception {
            System.out.println("value -> " + value);
        }
    });
    //开始发送
    Disposable disposable = observable.connect();
    observable
            //延迟2秒钟注册
            .delaySubscription(2, TimeUnit.SECONDS)
            .subscribe(new Consumer<Long>() {
                @Override
                public void accept(Long value) throws Exception {
                    //收不到2秒之前发送的数据，收到的数据只有2秒后的数据
                    System.out.println("value -> " + value);
                }
            });
    new Timer().schedule(new TimerTask() {
        @Override
        public void run() {
            //6s之后停止发射数据
            disposable.dispose();
        }
    },6000);
```

#### refCount操作符

让一个可连接的Observable行为像普通的Observable

大白话讲：有些时候，我们想还是按照Cool Observable去当有Observer订阅就发射数据，但又想拥有Hot Observable的行为特性时，就可以使用RefCount操作符。RefCount相当于Publish操作符的逆向，转换后，这个Observable再被订阅时自动开始发射数据，但是和Cool Observable不同的是，依然还是像Hot Observable一样，如果已经开始发送了，比较后来的Observer再来订阅，依然不会收到之前的数据！还有，就是终止的操作则变为所有观察者都取消订阅了，就终止。（Hot Observable需要手动调用Connect返回的disposable去调用dispose()才能终止，相当于开始发送和终止被得自动化了）

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d226007efbe60726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS)
        //转换为可连接的Observable
        .publish()
        //再"逆向"转换为普通的Observable
        .refCount();
observable.subscribe(new Consumer<Long>() {
    @Override
    public void accept(Long value) throws Exception {
        System.out.println("value -> " + value);
    }
});
observable
        //延迟2秒钟注册
        .delaySubscription(2, TimeUnit.SECONDS)
        .subscribe(new Consumer<Long>() {
            @Override
            public void accept(Long value) throws Exception {
                //收不到2秒之前发送的数据，收到的数据只有2秒后的数据
                System.out.println("value -> " + value);
            }
        });
```

#### share操作符

因为publish和refCount太常用了，RxJava就提供了share操作符，内部其实就是调用了这2个操作符。所以平时一般用share比较多。

```
public final Observable<T> share() {
    return publish().refCount();
}
```

#### Hot Observable的使用场景

当发送的事件是“共享”的，并且需要所有观察者都取消订阅时才终止Observable的时候。

例如WebSocket聊天，像微信、QQ聊天中，有文本消息、图片消息、语音消息、系统消息等等~。

每种消息类型的处理，都可以比作为一个订阅者（一个订阅者只处理他绑定的那种消息），每个订阅者订阅的时机可能会不一致，可能在指定条件下才触发。（当然现在，暂时我都是一次性订阅），就当有可能会不同时机订阅，如果是Cool Observable，例如文本消息被订阅了，数据源Observable已经发送了很多消息了，指定时机才订阅了系统消息观察，之前所有的文本消息都会批量重放给这个系统消息观察者，而他又不需要处理这种类型事件，那么简直是浪费了这些事件，浪费了不必要的资源。

再比如，现在可能只有一个聊天界面存在了这些订阅者订阅，但是消息可不是只有一个界面会收到消息，外层的消息列表也是可以接收到消息的进行显示（你见过微信退出聊天界面就收不到消息的？）。甚至，App被杀了，所有的Activity都已经销毁，剩下Service在运行，消息也是在监听着（微信被杀了，就不能收到消息了？）

最后也是消息处理分层的问题，消息接收，无网需要查看聊天记录，那么肯定就要将消息记录插入到数据库，那么插入的这些处理操作，肯定不会和界面的消息观察放在一起（界面一退出就取消观察了，数据库插入记录的操作就不能执行了，这肯定不行）。所以做数据库插入记录的操作肯定是单独的一个消息观察者，那么在Cool Observable的情况下，旧的记录就会被重放，那这个记录就肯定会乱套了。

```
Observable
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
```

#### replay操作符

上面说到Hot Observable发射的事件，如果订阅者在Observable发射数据后，已发送了多个数据后再来订阅时，前面的数据是不会收到的。那么有没有办法能收到呢？replay操作符就能派上用场了。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-e7f24923da7ed3a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- replay()，没有参数，接收他订阅前的所有的事件，这时候就和Cool Observable是一样的了。

- replay(int bufferSize)，缓存事件的个数。例如设置为1，则后订阅的观察者能收到它订阅之前的一个事件。

```
Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS)
            //转换为可连接的Observable
            .publish()
            //再"逆向"转换为普通的Observable
            .refCount();
    observable.subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long value) throws Exception {
            System.out.println("value -> " + value);
        }
    });
    observable
            //缓存订阅前，一个1个事件
            .replay(1)
            //延迟2秒钟注册
            .delaySubscription(2, TimeUnit.SECONDS)
            .subscribe(new Consumer<Long>() {
                @Override
                public void accept(Long value) throws Exception {
                    //收不到2秒之前发送的数据，收到的数据只有2秒后的数据
                    System.out.println("value -> " + value);
                }
            });
```

- replay(long time, TimeUnit unit)，缓存指定时间内的事件，例如缓存3秒之前的事件，那么后订阅的观察者订阅时，它会收到订阅前3秒前发送的事件。

```
Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS)
            //转换为可连接的Observable
            .publish()
            //再"逆向"转换为普通的Observable
            .refCount();
    observable.subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long value) throws Exception {
            System.out.println("value -> " + value);
        }
    });
    observable
            //缓存订阅前，3秒内的事件
            .replay(3, TimeUnit.SECONDS)
            //延迟2秒钟注册
            .delaySubscription(2, TimeUnit.SECONDS)
            .subscribe(new Consumer<Long>() {
                @Override
                public void accept(Long value) throws Exception {
                    //收不到2秒之前发送的数据，收到的数据只有2秒后的数据
                    System.out.println("value -> " + value);
                }
            });
```