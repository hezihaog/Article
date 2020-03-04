#### RxJava2订阅流程浅析

使用RxJava2也有7、8个月了，越来越迷上它，使用期间出现各种各样的问题，有些是理解错误，每次都是去看一文档、看一遍别人的博客，还是迷迷糊糊的，RxJava门槛相对来说还是高一点，就像一把双刃剑，用得好的人会用得很爽，用不好的人，就会觉得很难用。所以还是需要从源码层面上理解，才能更好的使用它。

#### 订阅流程简单使用

入门RxJava2时，最简单的流程就是如下，如果你用过，相信你也一定很熟悉：

Observable.create()创建一个被观察者，create()方法需要传入一个ObservableOnSubscribe对象，ObservableOnSubscribe对象为被订阅时的回调，回调传入一个emitter发射器，通过emitter发射器，可以对订阅者发送数据。

emitter可以发射onNext()事件，onError()事件和onComplete()事件，其中onError()和onComplete()是互斥的，只能出现一个，onNext()则是可以发射无限个。

订阅通过Observable.subscribe()方法进行订阅，可以传入多个重载的参数，这里介绍每种回调单独配置的重载方法，第一个Consumer为发送onNext()时回调，第二个Consumer<Throwable>为发送onError()回调，以及Action，它是onComplete()时回调，最后一个Consumer为取消订阅时回调。

subscribe()订阅方法返回一个Disposable类型对象，它相当于是取消订阅的一个凭证，通过它的dispose()方法，可以进行取消订阅，一般会在Activity的onDestroy()调用时调用注销订阅。

```
Disposable disposable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("a");
        emitter.onNext("b");
        emitter.onNext("c");
        //onError()和onComplete()是互斥的，只能出现一个!
        //emitter.onError(new RuntimeException("发生了异常"));
        emitter.onComplete();
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String data) throws Exception {
        Logger.d("收到数据 data: " + data);
    }
}, new Consumer<Throwable>() {
    @Override
    public void accept(Throwable throwable) throws Exception {
        Logger.d("发生异常: " + throwable.getMessage());
    }
}, new Action() {
    @Override
    public void run() throws Exception {
        Logger.d("发送完毕");
    }
}, new Consumer<Disposable>() {
    @Override
    public void accept(Disposable disposable) throws Exception {
        Logger.d("取消订阅");
    }
});
//取消订阅，一般会在Activity的onDestroy()调用时调用注销订阅
disposable.dispose();

//输出
收到数据 data: " + a
收到数据 data: " + b
收到数据 data: " + c
发送完毕
取消订阅
```

#### create()方法流程

创建可观察者是通过Observable.create()静态工厂方法创建的，我们点进去看一下。

create()方法一开始看起来有点复杂，我们一步步来：

1. ObjectHelper.requireNonNull(source, "source is null")，就是非空检查，如果传入的source对象为空，则抛出异常。
2. RxJavaPlugins.onAssembly(xxx)，是RxJava提供的Hook操作，我们一般不会设置，可以先忽略
3. new ObservableCreate<T>(source)，创建了一个ObservableCreate类，将传进来的source作为参数传入。（source对象是我们创建的ObservableOnSubscribe对象，被订阅时的回调对象）

```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    //重点是ObservableCreate类
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```

将非空检查和Hook操作去掉，则核心代码是这样的：

是不是一下子清爽了很多~那么我们继续跟进ObservableCreate类

```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    //重点是ObservableCreate类
    return new ObservableCreate<T>(source);
}
```

- ObservableCreate类结构

1. ObservableCreate类继承于Observable类。
2. 构造方法，将我们传入的source对象保存（再复写下：source对象是我们创建的ObservableOnSubscribe对象，被订阅时的回调对象）
3. 然后？就没了！ObservableCreate类的创建只保存了我们传进来的source对象！

```
public final class ObservableCreate<T> extends Observable<T> {
    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }
    
    //...省略其他方法
}
```

- 可观察者的被订阅时机

ObservableCreate的构造方法执行只保存了source对象，就直接返回出去了。那么ObservableCreate对象什么时候才被用到的呢？

答案是：subscribe()订阅方法！我们的例子中，使用了Observable.create()方法创建出ObservableCreate对象后，我们接连调用了subscribe()订阅方法。我们来看下它是神是鬼！

1. 首先，我们是4个参数的subscribe()重载方法，最终会调用只有一个参数的Observer的subscribe(Observer)方法。
2. subscribe()方法，第一步就是对传入的参数都进行非空判断，如果为空，则抛出异常。
3. 对传进来的onNext、onError、onComplete、onSubscribe组装为一个Observer对象
4. 调用subscribe()，传入Observer对象。

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
        Action onComplete, Consumer<? super Disposable> onSubscribe) {
    //1、一堆参数的非空判断！
    ObjectHelper.requireNonNull(onNext, "onNext is null");
    ObjectHelper.requireNonNull(onError, "onError is null");
    ObjectHelper.requireNonNull(onComplete, "onComplete is null");
    ObjectHelper.requireNonNull(onSubscribe, "onSubscribe is null");

    //2、对传进来的onNext、onError、onComplete、onSubscribe组装为一个Observer对象
    LambdaObserver<T> ls = new LambdaObserver<T>(onNext, onError, onComplete, onSubscribe);

    //3、调用subscribe()，传入Observer对象
    subscribe(ls);
    return ls;
}
```

- 最终subscribe(observer)方法

1. 非空判断，对传入的observer对象，必须observer订阅者对象不为空。
2. Hook操作，提供被注册的钩子给RxJavaPlugins，没有处理，则返回的还是原有的observer。
3. 非空判断，RxJavaPlugins.onSubscribe()处理后的observer必须不为空。
4. 重点：调用subscribeActual()方法！
5. try-catch，对抛出的异常交给RxJavaPlugins处理，我们通过RxJavaPlugins可以设置一个全局异常处理。

最终逻辑落在了subscribeActual()方法，方法名翻译过来是实际的订阅，十有八九是这个方法做订阅操作了。

```
@SchedulerSupport(SchedulerSupport.NONE)
@Override
public final void subscribe(Observer<? super T> observer) {
    //非空判断，对传入的observer对象，必须observer订阅者对象不为空
    ObjectHelper.requireNonNull(observer, "observer is null");
    try {
        //Hook操作，提供被注册的钩子给RxJavaPlugins，没有处理，则返回的还是原有的observer
        observer = RxJavaPlugins.onSubscribe(this, observer);
        //非空判断，RxJavaPlugins.onSubscribe()处理后的observer必须不为空
        ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null Observer. Please change the handler provided to RxJavaPlugins.setOnObservableSubscribe for invalid null returns. Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");
        //重点：调用subscribeActual()方法！
        subscribeActual(observer);
    } catch (NullPointerException e) {
        //捕获抛出的空指针，再抛出
        throw e;
    } catch (Throwable e) {
        //将所有抛出的异常，交给RxJavaPlugins处理，提一下，我们通过RxJavaPlugins可以设置RxJava中抛出的所有异常！
        Exceptions.throwIfFatal(e);
        RxJavaPlugins.onError(e);
        NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
        npe.initCause(e);
        throw npe;
    }
}
```

- subscribeActual()方法分析

1. 纳尼，subscribeActual()方法是个抽象方法，具体实现在哪呢？

先来理一下，一开始Observable.create()创建了ObservableCreate类对象并返回，我们再调用了subscribe()方法，subscribe()方法内部再调用了subscribeActual()方法，而且它是抽象方法，那么subscribeActual()方法，肯定是被调用对象ObservableCreate这个对象里!

```
protected abstract void subscribeActual(Observer<? super T> observer);
```

- ObservableCreate类中的subscribeActual()方法

我们回到ObservableCreate类中，subscribeActual()方法的确被复写了

1. 创建发射器，将订阅者传给发射器
2. 调用observer订阅者的onSubscribe，通知订阅者，开始订阅
3. 调用source对象的subscribe()方法，通知可观察者被订阅了（再复习一下，source对象为我们调用create()对象传入的ObservableOnSubscribe回调对象），我们会在这里调用发射器的onNext()发射数据
4. try-catch处理，如果订阅过程中，发生异常，调用订阅者的onError()，通知订阅过程发生异常

subscribeActual()方法很简短，主要是通知可观察者被订阅和订阅者开始订阅，异常处理等。

下一步的重点在source.subscribe(parent)中，parent为CreateEmitter类对象，它包裹了被订阅者，我们在subscribe()方法中，就是Observable.create()中提供的回调对象的subscribe()方法中，调用了发射器emitter的onNext()发射数据给订阅者，所以下一步的逻辑应该是在CreateEmitter发射器类中。

```
@Override
protected void subscribeActual(Observer<? super T> observer) {
    //创建发射器，将订阅者传给发射器
    CreateEmitter<T> parent = new CreateEmitter<T>(observer);
    //调用observer订阅者的onSubscribe，通知订阅者，开始订阅
    observer.onSubscribe(parent);
    try {
        //再复习一下，source对象为我们调用create()对象传入的ObservableOnSubscribe回调对象
        //重点：调用source对象的subscribe()方法，通知可观察者被订阅了。我们会在这里调用发射器的onNext()发射数据
        source.subscribe(parent);
    } catch (Throwable ex) {
        Exceptions.throwIfFatal(ex);
        //调用订阅者的onError()，通知订阅过程发生异常
        parent.onError(ex);
    }
}
```

- CreateEmitter发射器

1. CreateEmitter发射器，实现了ObservableEmitter发射器接口和Disposable切断订阅接口
2. 构造方法，就是保存了observer订阅者。
3. onNext()，先空检查，如果为null，会抛出异常，证明RxJava2后不能发射null了。没有解除订阅，则调用observer订阅者的onNext()。
4. onError()事件，try-catch处理，如果事件中发生异常则交给RxJavaPlugins处理。
5. tryOnError()，上面onError()事件的，try-catch处理，如果为空或订阅者的onError()方法发生异常，返回true，否则返回false。
6. onComplete()完成，调用订阅者的observer.onComplete()
7. setCancellable()，提供一个Cancellable回调对象，当被dispose()时回调，可以在dispose()时做解除订阅时需要做的事情
8. dispose()，切断订阅。
9. isDisposed()，判断是否已经切断了订阅。

到这里，RxJava的订阅流程就接触了，下面来总结一下。

```
//发射器，实现了ObservableEmitter发射器接口和Disposable切断接口
static final class CreateEmitter<T>
extends AtomicReference<Disposable>
implements ObservableEmitter<T>, Disposable {
    private static final long serialVersionUID = -3434801548987643227L;

    //订阅者
    final Observer<? super T> observer;

    //构造方法，就是保存了observer订阅者
    CreateEmitter(Observer<? super T> observer) {
        this.observer = observer;
    }

    @Override
    public void onNext(T t) {
        //先空检查，如果为null，会抛出异常，证明RxJava2后不能发射null了
        if (t == null) {
            onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
            return;
        }
        //没有解除订阅，则调用observer订阅者的onNext()
        if (!isDisposed()) {
            observer.onNext(t);
        }
    }

    @Override
    public void onError(Throwable t) {
        //onError()事件，try-catch处理，如果事件中发生异常则交给RxJavaPlugins处理
        if (!tryOnError(t)) {
            RxJavaPlugins.onError(t);
        }
    }

    @Override
    public boolean tryOnError(Throwable t) {
        //上面onError()事件的，try-catch处理，如果为空或订阅者的onError()方法发生异常，返回true，否则返回false
        if (t == null) {
            t = new NullPointerException("onError called with null. Null values are generally not allowed in 2.x operators and sources.");
        }
        if (!isDisposed()) {
            try {
                observer.onError(t);
            } finally {
                dispose();
            }
            return true;
        }
        return false;
    }

    @Override
    public void onComplete() {
        //onComplete()完成，调用订阅者的observer.onComplete()
        if (!isDisposed()) {
            try {
                observer.onComplete();
            } finally {
                dispose();
            }
        }
    }

    @Override
    public void setDisposable(Disposable d) {
        DisposableHelper.set(this, d);
    }

    @Override
    public void setCancellable(Cancellable c) {
        //提供一个Cancellable回调对象，当被dispose()时回调，可以在dispose()时做解除订阅时需要做的事情
        setDisposable(new CancellableDisposable(c));
    }

    @Override
    public ObservableEmitter<T> serialize() {
        //序列化发射的数据，使用一个SerializedEmitter类来包裹CreateEmitter，装饰者模式，如果你发射的数据在不同线程中，可以调用serialize()来让发射的数据按发射顺序发射，不是本篇重点，就不分析SerializedEmitter类
        return new SerializedEmitter<T>(this);
    }

    @Override
    public void dispose() {
        //切断订阅
        DisposableHelper.dispose(this);
    }

    @Override
    public boolean isDisposed() {
        //判断是否已经切断了订阅
        return DisposableHelper.isDisposed(get());
    }

    @Override
    public String toString() {
        return String.format("%s{%s}", getClass().getSimpleName(), super.toString());
    }
}
```

#### 流程总结

RxJava2的总体流程：

RxJava2的订阅流程还算清晰，主要是要清楚几个切入点：

![RxJava2订阅流程.png](https://upload-images.jianshu.io/upload_images/1641428-4886fe4578832c0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建被观察者时机
- 订阅时机
- 调用发射器发射数据时机