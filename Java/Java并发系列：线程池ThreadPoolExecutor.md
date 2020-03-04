#### Java并发系列：线程池ThreadPoolExecutor基本使用

上一篇说到，线程的创建和销毁耗费的资源是很多的，我们应该使用线程池来代替显式创建线程，复用线程执行我们的任务，本篇我们就来学习一下线程池的基本使用吧~

#### 线程池的创建

既然要使用线程池，那么首先就需要创建线程池了。在JDK1.5以上版本，Java提供了Executors类，它其实可以说是一个工厂类，创建线程池需要比较多的一些参数，而Executors则提供了一批更加语义化的创建线程池的静态方法。

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

- Executors.newCachedThreadPool()，容量为无限大的线程池，它适合任务非常多，高并发的场景。例如EventBus，执行监听Event回调的任务，次数是未知的，就可以使用该策略的线程池。

- Executors.newSingleThreadExecutor()，容量为1的线程池，它执行任务是串行的，执行完一个再执行下一个任务。这种策略的线程池一般就是处理串行任务的。

- Executors.newFixedThreadPool(10)，固定容量的线程池，如果接收到任务时，线程池容量已到达设置容量大小，就会阻塞。

- Executors.newScheduledThreadPool(10)，周期性任务的线程池，可以延时执行或周期执行，可以作为定时器使用。

#### 提交任务到线程池

创建完线程池，接下来就是提交任务到线程池中执行了~执行方法常用有2种，execute()和submit()。

- execute(Runnable runnable)，执行一个runnable任务对象，所以它的执行是无返回值的。

- submit(Runnable runnable)，也是执行一个runnable任务对象，但是返回Future对象，Future对象可以通过get()方法获取返回值，但实际该重载方法传入null值，所以即使调用get()方法，获取到的也是null值。

- submit(Runnable runnable,T result)，和第二个重载方法一样，也是返回Future对象，其实第二个重载方法就是转调该方法，result对象传入null值。

- submit(Callable callable)，传入一个Callable对象，我们知道Callable对象是有返回值的，而Runnable是没有返回值的。

- invokeAny(Collection<? extends Callable<T>> tasks)，执行一个Callable任务集合，随机执行其中一个Callable任务，返回任务的结果，如果任务抛出异常，则Callable其他任务则会被取消。

- invokeAll(Collection<? extends Callable<T>> tasks)，执行一个Callable任务集合中的全部任务，返回List<Future<T>>。

#### 示例

- execute()，执行无返回值的Runnable。

```
/**
 * 提交Runnable任务
 */
private static void executeByRunnable() {
    ExecutorService pool = Executors.newCachedThreadPool();
    pool.execute(new Runnable() {
        @Override
        public void run() {
            System.out.println("我在execute()中执行任务");
        }
    });
}

//输出
我在execute()中执行任务
```

- submit()，执行有返回值的Callable。

```
/**
 * 提交Callable任务
 */
private static void submitByCallable() {
    try {
        ExecutorService pool = Executors.newCachedThreadPool();
        Future<String> future = pool.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果";
            }
        });
        String result = future.get();
        System.out.println("result: " + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

//输出
result: 我是结果
```

- invokeAny()，执行任务集合中任意一个任务。

```
/**
 * 随机执行任务集合中的任意一个任务
 */
private static void runRandomTask() {
    try {
        ExecutorService pool = Executors.newCachedThreadPool();
        ArrayList<Callable<String>> taskList = new ArrayList<>();
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果1";
            }
        });
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果2";
            }
        });
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果3";
            }
        });
        String result = pool.invokeAny(taskList);
        System.out.println("运行结果：" + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

//输出
运行结果：我是结果1
```

- invokeAll()，执行任务集合中的所有任务。

```
/**
 * 执行所有任务
 */
private static void runAllTask() {
    try {
        ExecutorService pool = Executors.newCachedThreadPool();
        ArrayList<Callable<String>> taskList = new ArrayList<>();
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果1";
            }
        });
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果2";
            }
        });
        taskList.add(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "我是结果3";
            }
        });
        List<Future<String>> futures = pool.invokeAll(taskList);
        for (Future<String> future : futures) {
            String result = future.get();
            System.out.println("获取结果：" + result);
        }
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

//输出
获取结果：我是结果1
获取结果：我是结果2
获取结果：我是结果3
```

- 相对于可以执行周期性任务的ScheduledExecutorService，则还有2个方法，分别是延时执行schedule()，周期性执行scheduleAtFixedRate()。

	- schedule()，延时执行。

	```
	/**
	 * 延迟执行
	 */
	private static void runTaskWithDelay(Runnable runnable) {
	    //延迟3秒后执行
	    ScheduledExecutorService pool = Executors.newScheduledThreadPool(10);
	    pool.schedule(runnable, 3, TimeUnit.SECONDS);
	}
	
	//调用
   runTaskWithDelay(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是线程池中的任务");
            System.out.println("我延迟3秒执行了...");
        }
    });
	
	//输出
	我是线程池中的任务
	我延迟3秒执行了...
	```

	- scheduleAtFixedRate()，周期性执行。

	```
	/**
	 * 周期执行任务
	 */
	private static void runTaskWithCycle(Runnable runnable) {
	    ScheduledExecutorService pool = Executors.newScheduledThreadPool(10);
	    //1秒执行一次
	    pool.scheduleAtFixedRate(runnable, 0, 1, TimeUnit.SECONDS);
	}
	
	//调用
    runTaskWithCycle(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是周期性的任务");
        }
    });
	
	//输出
	我是周期性的任务
	我是周期性的任务
	我是周期性的任务
	我是周期性的任务
	我是周期性的任务
	```

#### 关闭线程池

如果我们需要关闭线程池，有2种方法，shutdown()和shutdownNow()。

- shutdown()，并不会马上关闭线程池，而会等待任务都执行完毕后再关闭

```
/**
 * 关闭线程池，会等待任务执行完毕后再关闭
 */
private static void shutdownThreadPool() {
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(10);
    pool.schedule(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是执行结果...");
        }
    }, 1, TimeUnit.SECONDS);
    pool.shutdown();
}

//输出
我是执行结果...
```

- shutdownNow()则是强制关闭线程池，而不会等待任务执行完毕。

```
/**
 * 马上关闭线程池
 */
private static void shutdownNowThreadPool() {
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(10);
    pool.schedule(new Runnable() {
        @Override
        public void run() {
            System.out.println("我是执行结果...");
        }
    }, 1, TimeUnit.SECONDS);
    pool.shutdownNow();
}

//没有输出结果，因为任务被取消了
```

#### 总结

频繁使用线程必须使用线程池，避免线程的频繁创建和销毁带来的资源消耗。本篇，我们讲解了快捷创建线程池的Api。下一篇，我们再讲解线程池具体参数的自定义。