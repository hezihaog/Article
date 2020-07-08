#### Java并发系列：Thread、Runnable、Callable、Future和FutureTask

在Java的世界里，异步操作一般使用Thread，本篇来讲讲Thread的有返回值和无返回值的多线程Api。

#### 无返回值的Thread创建

1. 继承Thread，复写run方法，多线程执行时回调Thread的run()方法进行执行任务。

```
/**
 * 使用Thread，复写run方法进行任务执行
 */
private static void runThread() {
    Thread thread = new Thread() {
        @Override
        public void run() {
            super.run();
            System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
        }
    };
    thread.start();
}

//输出
我在Thread-0中执行...
```

2. 使用Runnable，和Thread组合执行。Thread还有一个有参构造传入Runnable，任务执行则在Runnable的run()方法中执行。

```
/**
 * 使用Runnable构建Thread对象
 */
private static void runThreadByRunnable() {
    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是Runnable中的任务");
            System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
        }
    });
    thread.start();
}

//输出
我是Runnable中的任务
我在Thread-0中执行...
```

3.需要注意的是，Thread启动线程不能调用run()，必须调用start()，否则执行任务的线程为当前线程，而不是新起一个线程去执行。

```
private static void runThreadByRunnable() {
    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是Runnable中的任务");
            System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
        }
    });
    //错误示范，Thread启动线程不能调用run()，必须调用start()，否则执行任务的线程为当前线程，而不是新起一个线程去执行。
    thread.run();
}

//输出
我是Runnable中的任务
我在main中执行...
```

#### 有返回值Thread

使用Runnable和Thread，是没有返回值的，当我们需要获取到结果后，继续执行时，就有点尴尬了~而在JDK1.5时，提供了**Callable、Future和FutureTask**，作为多线程Api的补充。一起来看下吧~

1. 首先，Callable，是一个带返回值的接口，对比Runnable来对比。

```
//有返回值，Callable
public interface Callable<V> {
    V call() throws Exception;
}

//无返回值，Runnable
public interface Runnable {
    public abstract void run();
}
```

2. 而Future，为有结果操作对象的操作接口。

```
public interface Future<V> {
	//取消执行
    boolean cancel(boolean mayInterruptIfRunning);

	//是否已取消
    boolean isCancelled();

	//是否已结束
    boolean isDone();

	//获取返回值，阻塞操作，如果还未执行完，则会以一个死循环进行循环检查，直到生成了结果，结束阻塞并返回结果
    V get() throws InterruptedException, ExecutionException;

	//get重载，设置一个超时时间，如果指定时间后还未返回结果，抛出TimeoutException异常
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

3. FutureTask，既然Callable是返回值任务回调接口，Future为操作接口，感觉还少点东西，啥呢？缺一个Future的实现类，不然操作的状态谁来维护呢？(cancel、done)。

```
//FutureTask实现了RunnableFuture接口，来瞅瞅~
public class FutureTask<V> implements RunnableFuture<V> {
}

//RunnableFuture接口，继承自Runnable和Future
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

//创建FutureTask。要在子线程执行，就需要Thread环境，而Thread类的构造方法只能传一个Runnable，刚好RunnableFuture实现了Runnable，所以Thread的构造就能传入FutureTask对象。
Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
        System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
        return "我是结果";
    }
};
FutureTask<String> futureTask = new FutureTask<>(callable);
//将任务交给线程执行
Thread thread = new Thread(futureTask);
thread.start();
```

4. 获取返回值结果，因为FutureTask实现了Future接口，所以它有get()方法。我们调用get方法获取结果。

```
//...省略上面代码
FutureTask<String> futureTask = new FutureTask<>(callable);
//将任务交给线程执行
Thread thread = new Thread(futureTask);
thread.start();
String result = futureTask.get();
//获取结果
System.out.println("我是FutureTask+Callable中执行的任务，结果为: " + result);
```

5. 最终代码

```
/**
 * 带返回值的任务
 */
private static void runThreadByCallable() {
    Callable<String> callable = new Callable<String>() {
        @Override
        public String call() throws Exception {
            System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
            return "我是结果";
        }
    };
    FutureTask<String> futureTask = new FutureTask<>(callable);
    try {
        //将任务交给线程执行
        Thread thread = new Thread(futureTask);
        thread.start();
        //futureTask.cancel(true);
        //如果取消或结束了，就不往下走了
        if (futureTask.isCancelled()) {
            System.out.println("任务被取消了");
            return;
        }
        if (futureTask.isDone()) {
            System.out.println("任务完成了");
            return;
        }
        String result = futureTask.get();
        System.out.println("我是FutureTask+Callable中执行的任务，结果为: " + result);
        if (futureTask.isDone()) {
            System.out.println("我完成任务啦...");
        }
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

//输出
我在Thread-0中执行...
我是FutureTask+Callable中执行的任务，结果为: 我是结果
我完成任务啦...
```

#### 使用线程池

频繁创建线程和销毁线程，十分消耗资源。那么有没有复用线程，执行任务的Api呢？有，那就是线程池~线程池这里不细讲哈~

- 不带返回值的结果。在线程池中执行~

```
/**
 * 不带返回值的任务在线程池中执行
 */
private static void runThreadByExecutorService(Runnable runnable) {
    ExecutorService executor = Executors.newCachedThreadPool();
    executor.execute(runnable);
}

//输出
我在pool-1-thread-1中执行...
```

- 带返回值的结果，这里使用Executors创建一个线程池，通过executor.submit()提交Callable，返回一个Future接口对象。哎？怎么没有FutureTask出现？我们来看submit()方法。原来submit()将Callable包装为RunnableFuture，再返回啦~所以不需要new FutureTask()喔~而返回的Future接口，就是RunnableFuture。

```
//提交任务
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}

//将Callable包装为RunnableFuture
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
```

- 最终代码~

```
/**
 * 带返回值任务在线程池中执行
 */
private static void runThreadByExecutorService() {
	//创建线程池
    ExecutorService executor = Executors.newCachedThreadPool();
    //提交任务
    Future<String> future = executor.submit(new Callable<String>() {
        @Override
        public String call() throws Exception {
            System.out.println("我在" + Thread.currentThread().getName() + "中执行...");
            return "我是结果";
        }
    });
    executor.shutdown();
    try {
    	 //获取结果
        String result = future.get();
        System.out.println("带返回值的任务-结果为: " + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

//输出
我在pool-1-thread-1中执行...
带返回值的任务-结果为: 我是结果
```

#### 总结

1. 需要在子线程中执行任务，并返回返回值时，最好使用JDK提供的Callable、Future和FutureTask来实现~
2. 频繁使用线程，需要使用线程池。