#### Java并发系列：线程池ThreadPoolExecutor参数自定义

#### 上篇回顾

上篇我们讲了，线程池的创建、提交任务到线程池、关闭线程池。基本使用我们已经没有多大问题了。上节我们创建线程池时，都是使用Executors这个工厂类的静态方法创建线程池，其实里面内置的参数并不一定适合我们的场景，这时候就需要自定义参数了，一起来学习一下吧~

- 上篇我们使用的Executors创建线程池

```
//创建一个线程池，容量大小为Integer.MAX_VALUE，所以是无限大
ExecutorService pool = Executors.newCachedThreadPool();

//创建容量为1的线程池，它的执行是串行的
ExecutorService pool = Executors.newSingleThreadExecutor();

//创建固定容量大小的线程池
ExecutorService pool = Executors.newFixedThreadPool(10);

//周期性任务线程池
ScheduledExecutorService pool = Executors.newScheduledThreadPool(10);
```

#### 线程池创建参数分析

在上面，我们使用Executors类，无论是newCachedThreadPool()，还是newSingleThreadExecutor()等等，都是创建了ThreadPoolExecutor的实例。

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

我们来看一下ThreadPoolExecutor类的构造方法。

```
public class ThreadPoolExecutor extends AbstractExecutorService {
	private static final RejectedExecutionHandler defaultHandler =
        new AbortPolicy();

	//...
	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
    
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
    
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
    
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
	//...
}
```

我们看到，ThreadPoolExecutor线程池的前3个构造方法缺省参数的，都是调用第四个构造方法。接下来我们来分析这几个构造参数的含义~

1. corePoolSize，核心线程数，这个数量，是线程池始终保持的线程数量，即使没有任务执行，也不会终止。默认线程池创建后，在没有任务来之前，是没有线程存在的，除非调用了prestartAllCoreThreads()或prestartCoreThread()这2个方法，这2个方法是用来预先创建线程的，那么就会根据这个**核心线程数**进行预创建线程，并放到线程池中。

2. maximumPoolSize，线程池最大线程数，他表示这个线程池最多能创建多少个线程。

3. keepAliveTime，空闲存活时间。即线程在没有执行任务时的存活时间，默认情况下，这个存活时间，只在线程池的线程数大于corePoolSize核心线程数时才生效，例如当前线程数5是大于核心线程数3的，那么多出的那2个线程，在空闲（没有执行任务）时，如果空闲时间大于**keepAliveTime**，就会被终止，直到线程池的线程数不超过corePoolSize核心线程数。但如果调用了allowCoreThreadTimeOut(true)允许核心线程池超时，即使线程池中的线程数不大于corePoolSize核心线程数，keepAliveTime空闲存活时间也会生效，直到线程池的线程数为0。

4. unit，keepAliveTime空闲存活时间的时间单位。

```
//天
TimeUnit.DAYS;
//小时
TimeUnit.HOURS;
//分钟
TimeUnit.MINUTES;
//秒
TimeUnit.SECONDS;
//毫秒
TimeUnit.MILLISECONDS;
//微秒
TimeUnit.MICROSECONDS;
//纳秒
TimeUnit.NANOSECONDS;
```

5. workQueue，BlockingQueue类型的阻塞队列接口，用于存储线程池中待执行的任务。一般为LinkedBlockingQueue或SynchronousQueue。

6. threadFactory，线程工厂，创建线程对象的工厂类。

7. handler，RejectedExecutionHandler接口，是当线程池拒绝加入任务时使用的任务拒绝处理器。拒绝处理器有以下4种。

```
//丢弃任务，并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.AbortPolicy
//忽略，什么都不发生，即抛弃任务就不管了
ThreadPoolExecutor.DiscardPolicy
//从队列中踢出最先进入队列（最后一个执行）的任务，然后尝试执行任务
ThreadPoolExecutor.DiscardOldestPolicy
//使用线程处理该任务
ThreadPoolExecutor.CallerRunsPolicy
```

#### ThreadPoolExecutor的7个参数理解

ThreadPoolExecutor的构造方法既有7个，怎么理解呢？

- 举个栗子，有3位安卓工程师，核心员工，需要做一个项目，那么数量3就是**corePoolSize-核心线程数**。但是工期实在太赶，只有2个星期，3位工程师实在做不完，项目经理就委派HR大佬**threadFactory**招来了2位做外包的小伙伴，3+2=5，那么5就是**maximumPoolSize-最大线程数**。2个星期过后，项目顺利做完了，2位小伙伴就开始空闲了（基本没什么活干了），那么**keepAliveTime-空闲时间和unit空闲时间单位**就是他们的空闲时间，一到空闲时间，项目经理就将2位小伙伴“炒掉了”（请人还是要钱嘛）。过了没多久，App的第二版需求就出来了，一堆的需求蜂拥而至，项目经理要分辨哪些是紧急需求，哪些是不紧急需求，将需求整理放到一个**workQueue任务队列**中排队，3位小伙伴就开始工作啦~，临近第二期尾期了，突然产品经理提出，我要**加需求！**，这时候就需要**handler任务拒绝处理器**来决定了，是丢弃任务下期再做呢，还是3位小伙伴加加班搞定。

#### AbstractExecutorService、ExecutorService和Executor

```
//ThreadPoolExecutor继承于抽象类AbstractExecutorService
public class ThreadPoolExecutor extends AbstractExecutorService {
	//...
	
	//实现execute()执行任务方法
	public void execute(Runnable command) {
		//...
	}
	
	//...
}

//抽象类AbstractExecutorService实现了ExecutorService接口
public abstract class AbstractExecutorService implements ExecutorService {
	//...
	
	//对submit()系列方法的实现
	public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
    
    //对invokeAny()的实现
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
		  //...
    }
    
    //对invokeAll()的实现
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        	//...
        }
	
	//...
}

//ExecutorService接口继承了Executor接口
public interface ExecutorService extends Executor {
	//...

	//关闭线程池
	void shutdown();
	
	//提交有返回值的任务到线程池，submit有多个重载方法，这里不演示了
	<T> Future<T> submit(Callable<T> task);
	
	//执行任务集合中的任意一个有返回值的任务
	<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
     
    //执行任务集合中的所有任务   
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    throws InterruptedException;
    
    //...
}

//Executor接口中只定义了一个execute()方法，执行Runnable任务
public interface Executor {
    void execute(Runnable command);
}
```

- 先从最底开始，Executor接口定义了execute()方法，一个指定方法，证明Executor接口能执行一个无返回值的任务。

- ExecutorService接口，继承Executor，定义了例如shutdown()，submit()，invokeAny()，invokeAll()等方法。

- AbstractExecutorService抽象类，实现了ExecutorService接口，对submit()，invokeAny()，invokeAll()等在ExecutorService接口中申明的方法提供公共实现。

- 最后我们使用的ThreadPoolExecutor线程池类，继承于AbstractExecutorService抽象类，实现execute()等还未实现的方法，以及线程池的具体实现。

#### 如何合理设置线程池参数

#### 最简化公式

- CPU 密集型应用：线程池大小设置为 N + 1

- IO 密集型应用：线程池大小设置为 2N

公式的意义在于避免陷入极端情况。其中，计算密集型任务假设“等待时间/计算时间”等于0，IO密集型任务假设“等待时间/计算时间”等于1。

#### 为什么要有+1呢？

这是因为，就算是计算密集型任务，也可能存在缺页等问题（需要了解虚拟内存和物理内存的分配），产生“隐式”的IO。多一个额外的线程能确保CPU时钟周期不会被浪费，又不至于增加太多线程调度成本。

#### 严格公式

假设每个线程的“等待时间 / 计算时间”大小相等，显然，“计算时间 / (计算时间 + 等待时间)”也相等。对1个线程而言，只有计算时间占用了逻辑CPU，假设这个线程一直运行在同1个逻辑CPU上，显然，该逻辑CPU的CPU利用率即等于“计算时间 / (计算时间 + 等待时间)”。

对于多个线程的情况是一样的，则有公式：

```
逻辑CPU数 * CPU利用率 / 线程数 = 计算时间 / (计算时间 + 等待时间)
```
也等于：

```
线程数 = (1 + 等待时间/计算时间) * 逻辑CPU数 * CPU利用率
```

- “1 + 等待时间/计算时间” 只与任务本身有关。

- 逻辑CPU数可通过cat /proc/cpuinfo | grep -c processor得到。

- 标准的CPU利用率要通过实际监控得到，但在估算线程池大小时，应看做“期望得到的CPU利用率”，即可分配给该任务的CPU比例。如果只打算分配一半CPU给任务的话，就是0.5。

如果估算得到的线程数比较多，那么还要适当提高可分配的CPU比例，因为线程切换的成本随线程数增加而增加。如果竞争较激烈，则可以适当降低可分配的CPU比例，因为竞争通常也会导致线程阻塞，使CPU空闲。