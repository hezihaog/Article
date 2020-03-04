#### AsyncTask源码解析

想起刚开始学Android开发的时候，AsyncTask是一个强大又难用的类，难用的原因：

1. 有3个泛型参数
2. 容易内存泄露
3. 输入参数和输出参数不一致就要新建一个AsyncTask子类，不好复用
4. 3.0之前是并行但是只有128个任务并发执行，由于AsyncTask是共享线程池，所以App如果有这么大数量发起异步任务时，会抛出RejectedExecutionException拒绝执行的异常。
5. 3.0之后任务是串行的，任务执行完一个再一个，以前不能自定义执行器避免128个任务溢出。

#### AsyncTask简单使用和讲解

新建类继承AsyncTask类，主要需要复写以下方法：

1. doInBackground(params)，在子线程运行任务时回调，返回任务结果

可选方法：

1. onPreExecute()，执行前回调，在创建AsyncTask的线程中回调，例如主线程中创建，则在主线程中回调，子线程中创建则在子线程中回调。

2. onPostExecute(result)，主线程中执行，任务执行完毕后回调。

3. onProgressUpdate(progress)，主线程中执行，获取任务进度，默认都不会回调，如果需要则调用AsyncTask中的publishProgress(progress)方法设置进度后回调。

4. onCancelled(result)，任务被取消时回调，为什么还有一个result任务结果对象呢？原因是如果调用cancel()停止AsyncTask任务时，如果任务已经执行完了，则会回调这个方法。

5. onCancelled()，同样是任务被取消时回调，但是没有result任务执行结果对象，这种是调用cancel()停止AsyncTask任务时，任务还未执行完，那么就会回调这个方法。

#### 简单使用

下载一张网络图片，并显示到ImageView，在没有现在的图片加载框架时，都是这么做的。

- 定义一个下载图片任务类，返回Bitmap

```
private static class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
    private OkHttpClient mClient;
    private Callback mCallback;

    /**
     * 回调接口
     */
    public interface Callback {
        /**
         * 执行前回调
         */
        void onStart();

        /**
         * 执行后回调
         *
         * @param bitmap 执行结果
         */
        void onFinish(Bitmap bitmap);
    }

    public DownloadImageTask(OkHttpClient client, Callback callback) {
        mClient = client;
        mCallback = callback;
    }

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
        //准备开始任务
        mCallback.onStart();
    }

    @Override
    protected Bitmap doInBackground(String... urls) {
        try {
            String url = urls[0];
            Request request = new Request.Builder()
                    .url(url)
                    .build();
            Call call = mClient.newCall(request);
            Response response = call.execute();
            //进度更新，这里不需要，所以只演示一些，如果是文件下载就会有进度
            //publishProgress();
            return BitmapFactory.decodeStream(response.body().byteStream());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    protected void onProgressUpdate(Void... values) {
        super.onProgressUpdate(values);
        //进度更新
    }

    @Override
    protected void onCancelled(Bitmap bitmap) {
        super.onCancelled(bitmap);
        //任务被取消
    }

    @Override
    protected void onCancelled() {
        super.onCancelled();
        //任务被取消
    }

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        super.onPostExecute(bitmap);
        //任务执行完毕
        mCallback.onFinish(bitmap);
    }
}
```

- 调用execute()开始下载，提供图片url和回调设置即可

```
vDownload.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        String url = "https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1565684573&di=82491d3ea2a4d5195d6f8bd90eba1953&src=http://image.coolapk.com/picture/2016/1210/459462_1481302685_5118.png.m.jpg";
        DownloadImageTask task = new DownloadImageTask(mClient, new DownloadImageTask.Callback() {
            @Override
            public void onStart() {
                vImage.setImageDrawable(null);
                //显示等待弹窗
                mLoadingDialog.setMessage("开始下载...");
                mLoadingDialog.show();
            }

            @Override
            public void onFinish(Bitmap bitmap) {
                //下载结束
                if (bitmap != null) {
                    vImage.setImageBitmap(bitmap);
                    Toast.makeText(MainActivity.this.getApplicationContext(), "下载成功", Toast.LENGTH_SHORT).show();
                    //隐藏等待弹窗
                    mLoadingDialog.dismiss();
                }
            }
        });
        task.execute(url);
    }
});
```

#### AsyncTask其他API

- cancel(boolean mayInterruptIfRunning)，取消任务，mayInterruptIfRunning代表强制取消，不等待任务执行完毕马上取消，相当于打断任务

- isCancelled()，是否被取消了，true为被取消了，false为没有被取消

- get()，直接获取执行结果，注意会阻塞线程

- get(long timeout, TimeUnit unit)，设定超时时间获取结果，如果超时还没有获取到结果，则抛出异常

- execute(parmas)，执行任务，传入参数

- executeOnExecutor(Executor exec, Params... params)，指定执行器去执行任务，3.0默认是串行执行器，需要并行则设置Async内部的并发执行器

- getStatus()，获取当前任务的执行状态

#### 一步步源码分析

接下来就是源码分析了，源码分析先从任务的执行方法execute()开始：

1. execute()方法，调用了executeOnExecutor()方法，传入了sDefaultExecutor默认执行器和任务参数。

	- 一开始先判断运行状态，RUNNING正在执行不允许再调用执行，FINISHED已经结束了也不允许执行，所以AsyncTask只允许执行一次
	
	- 调用onPreExecute()方法，一般我们在这个方法里面做Ui操作，但是这里是直接调用的，所以不会一定在主线程执行，而是在调用execute()方法的线程执行！所以如果在onPreExecute()方法做UI操作，最好判断当前是否是主线程再执行，如果不是则post到主线程执行
	
	- 设置参数给任务对象

	```
	mWorker.mParams = params;
	```

	- exec.execute(mFuture)，将任务交给执行器执行

```
/**
 * 执行任务
 *
 * @param params 执行参数
 */
@MainThread
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params);
}

/**
 * 提供执行器，执行任务
 *
 * @param exec   执行器，可以设置为内部的THREAD_POOL_EXECUTOR来实现并行，否则默认使用串行的执行器
 * @param params 任务参数
 */
@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec, Params... params) {
    //处理状态，RUNNING正在执行不允许再调用执行，FINISHED已经结束了也不允许执行，所以AsyncTask只允许执行一次
    if (mStatus != AsyncTask.Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
            default:
                break;
        }
    }
    //切换状态为运行中
    mStatus = AsyncTask.Status.RUNNING;
    //任务执行前，回调，onPreExecute是直接调用的，所以如果在子线程中调用AsyncTask的execute
    //onPreExecute的回调也是在子线程！
    onPreExecute();
    //配置任务参数
    mWorker.mParams = params;
    //将任务交给线程池执行
    exec.execute(mFuture);
    return this;
}
```

2. 看完executeOnExecutor()方法，会对mWorker和mFuture这个2个对象有点疑惑，他们究竟是什么类的对象，我们去看下

	- 看到AsyncTask有个成员变量mWorker，类型是WorkerRunnable

	```
	 /**
     * 任务实现
     */
    private final WorkerRunnable<Params, Result> mWorker;
	```
	
	- WorkerRunnable类实现了Callable接口，所以他是一个可有返回值的任务，有一个mParams成员变量保存任务的参数。

	```
	/**
     * 任务
     *
     * @param <Params> 参数类型
     * @param <Result> 结果类型
     */
    private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        /**
         * 任务参数
         */
        Params[] mParams;
    }
	```
	
	- 接下来看下WorkerRunnable的创建时机，是在构造方法中创建的，AsyncTask的构造方法有2个重载方法，无参构造不传入Looper，还有个支持传入Handler参数的构造方法，这2个重载最终都是调用了具有Looper参数的构造方法。这个构造方法会判断传入的Looper是否为空或者是否为主线程的Looper，如果这2个条件成立则调用getMainHandler()创建主线程的Handler，否则使用指定Looper的Handler。

	```
	//-------------------- AsyncTask构造方法 start --------------------

    /**
     * 空参构造，取主线程的Handler回调
     */
    public AsyncTask() {
        this((Looper) null);
    }

    /**
     * 指定回调的Handler
     */
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * 可指定消息轮训器
     *
     * @param callbackLooper 轮训器
     */
    public AsyncTask(@Nullable Looper callbackLooper) {
        //没指定，或者指定为主线程的轮训器，则构造主线程的Handler，否则使用指定的轮训器创建
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
                ? getMainHandler()
                : new Handler(callbackLooper);
        //任务的具体执行体，WorkerRunnable实现了Callable接口，就是说可以返回结果的任务
        mWorker = new WorkerRunnable<Params, Result>() {
            @Override
            public Result call() throws Exception {
                //标识任务为已经执行了
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    //设置线程优先级为后台线程，多个线程并发时，无关紧要的线程分配的CPU时间将会减少，有利于主线程的处理
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //执行任务，并获取执行结果
                    result = doInBackground(mParams);
                    //将进程中未执行的命令，一并送往CPU处理
                    Binder.flushPendingCommands();
                } catch (Throwable throwable) {
                    //抛出了异常，设置任务已经被取消
                    mCancelled.set(true);
                    throw throwable;
                } finally {
                    //发送结果
                    postResult(result);
                }
                //返回结果
                return result;
            }
        };
        //将WorkerRunnable作为FutureTask任务去执行
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                //任务结束，任务结束了，call还没有被调用
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    //被取消时会抛出异常
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    //执行发生异常异常
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    //取消失败
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    //-------------------- AsyncTask构造方法 end --------------------
    
    /**
     * 获取主线程Handler
     */
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }

    private Handler getHandler() {
        return mHandler;
    }
	```
	
	- 接下来看WorkerRunnable的构造，WorkerRunnable其实就是任务的具体执行。

		- mTaskInvoked是一个原子性的布尔值，将她它设置为true，代表已经执行了
		- 配置线程的优先级为后台线程，多个线程并发时，无关紧要的线程分配的CPU时间将会减少，有利于主线程的处理
		- 调用doInBackground()，回调我们执行耗时操作
		- 调用Binder.flushPendingCommands()，这个方法我也不太了解，只搜索到说设置线程优先级为后台线程，多个线程并发时，无关紧要的线程分配的CPU时间将会减少，有利于主线程的处理。属于一种优化手段
		- 对任务执行try-catach，发生异常时，将任务标记为取消，mCancelled.set(true);
		- finally块进行postResult(result)，发送结果通知主线程
		- 最后返回结果result

	```
	 /**
     * 原子性布尔标记是否取消
     */
    private final AtomicBoolean mCancelled = new AtomicBoolean();
	
	 /**
     * 原子性布尔标记，任务是否执行了
     */
    private final AtomicBoolean mTaskInvoked = new AtomicBoolean();
	
	//任务的具体执行体，WorkerRunnable实现了Callable接口，就是说可以返回结果的任务
    mWorker = new WorkerRunnable<Params, Result>() {
        @Override
        public Result call() throws Exception {
            //标识任务为已经执行了
            mTaskInvoked.set(true);
            Result result = null;
            try {
                //设置线程优先级为后台线程，多个线程并发时，无关紧要的线程分配的CPU时间将会减少，有利于主线程的处理
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //执行任务，并获取执行结果
                result = doInBackground(mParams);
                //将进程中未执行的命令，一并送往CPU处理
                Binder.flushPendingCommands();
            } catch (Throwable throwable) {
                //抛出了异常，设置任务已经被取消
                mCancelled.set(true);
                throw throwable;
            } finally {
                //发送结果
                postResult(result);
            }
            //返回结果
            return result;
        }
    };
	```
	
	- 接下来我们看postResult(result)方法，方法意思就是发送执行结果，这里看到我们熟悉的Handler，调用getHandler()，获取一个Handler发送一个MESSAGE_POST_RESULT类型的消息，将结果包装为AsyncTaskResult，发送到主线程

	- AsyncTaskResult，实际就是一个包裹任务结果和任务实例的类

	```
	/**
     * 发送任务执行结果到主线程进行回调
     */
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<>(this, result));
        message.sendToTarget();
        return result;
    }
    
    /**
     * 获取主线程Handler
     */
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }

    private Handler getHandler() {
        return mHandler;
    }
    
    /**
     * 任务执行结果包裹类，包装结果作为Handler发送的数据
     *
     * @param <Data> 任务结果类型
     */
    @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        /**
         * @param task 任务对象
         * @param data 执行结果
         */
        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
	```
	
	- 既然看到了发送结果给主线程的Handler，那么就有Handler定义，Handler使用getMainHandler()方法调用获取到的，创建了一个InternalHandler类，一个内部的Handler类，不难猜到刚才MESSAGE_POST_RESULT标志的消息处理肯定在这个Handler中处理

	- 果然没猜错，InternalHandler中复写了handleMessage()方法，处理MESSAGE_POST_RESULT类型的消息并且还有MESSAGE_POST_PROGRESS任务进度类型的消息

	- 刚才AsyncTaskResult的结果，包裹了AsyncTask的实例，所以在这里回调AsyncTask的onProgressUpdate()方法，就是相当于在主线程回调

	- 说回MESSAGE_POST_RESULT类型的消息处理，调用了AsyncTask实例的finish()方法，传入了任务结果result

	```
	/**
     * 内部Handler，负责从子线程中发送消息会主线程进行方法回调
     */
    private static class InternalHandler extends Handler {
        InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                //主线程通知，任务结果，获取到结果
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    //更新进度
                    result.mTask.onProgressUpdate(result.mData);
                    break;
                default:
                    break;
            }
        }
    }
	```
	
	- 继续跟踪，finish()方法，finish()方法就是结束任务的方法，首先isCancelled()判断任务是否被取消，如果取消了，回调onCancelled(result)，如果没有被取消则回调onPostExecute通知结束

	- 设置任务状态为结束，mStatus = AsyncTask.Status.FINISHED

	```
	/**
     * 结束任务
     *
     * @param result 结果
     */
    private void finish(Result result) {
        //结束任务时，如果被标记取消，则回调取消
        if (isCancelled()) {
            onCancelled(result);
        } else {
            //获取到结果了，并且没有取消，则回调任务结束
            onPostExecute(result);
        }
        //设置状态为结束
        mStatus = AsyncTask.Status.FINISHED;
    }
	```
	
	- 到这里，我们已经将mWork的流程看完了，接下来回调我们mWork创建完毕后的流程

	- 可以看到mWorker作为FutureTask的构造参数传入，就是说将任务交给FutureTask去执行

	- 然后复写了FutureTask的done()方法，就是任务结束后做逻辑，done中主要是调用了postResultIfNotInvoked()方法

	```
	//将WorkerRunnable作为FutureTask任务去执行
    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            //任务结束，任务结束了，call还没有被调用
            try {
                postResultIfNotInvoked(get());
            } catch (InterruptedException e) {
                //被取消时会抛出异常
                android.util.Log.w(LOG_TAG, e);
            } catch (ExecutionException e) {
                //执行发生异常异常
                throw new RuntimeException("An error occurred while executing doInBackground()",
                        e.getCause());
            } catch (CancellationException e) {
                //取消失败
                postResultIfNotInvoked(null);
            }
        }
    };
	```
	
	- postResultIfNotInvoked主要是判断mTaskInvoked原子性布尔值，如果没有设置为true，则调用postResult再将结果发送给主线程

	- 这里有点奇怪，这个方法调用在FutureTask的done方法，而mTaskInvoked标志在call()回调时就会设置为true，所以后面的if判断一直都是进不去的，感觉这个方法是多余的，除非call方法执行还没到mTaskInvoked设置成功之前就抛出了异常，这个if判断才成立

	```
	/**
     * 发送任务执行结果，如果没有发送过则发送
     */
    private void postResultIfNotInvoked(Result result) {
        //这里有点奇怪，这个方法调用在FutureTask的done方法，而mTaskInvoked标志在call()回调时就会设置为true
        //所以后面的if判断一直都是进不去的，感觉这个方法是多余的，除非call方法执行还没到mTaskInvoked设置成功之前就抛出了异常
        //这个if判断才成立
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
	```
	
	- FutureTask的创建看完了，无非就是对mWork的包装，主要逻辑还是在mWork的call()方法中。那么FutureTask什么时候被执行呢？回到executeOnExecutor()方法，FutureTask的mFuture被放入了exec执行器去执行

	```
	/**
     * 提供执行器，执行任务
     *
     * @param exec 执行器，可以设置为内部的THREAD_POOL_EXECUTOR来实现并行，否则默认使用串行的执行器
     * @param params 任务参数
     */
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec, Params... params) {
        //...省略代码
        //将任务交给线程池执行
        exec.execute(mFuture);
        return this;
    }
	```
	
	- 那么来看下一下exec是哪里来的，一般我都是调用execute(exec, params)方法开始任务执行，而execute(params)方法传入的exec是sDefaultExecutor

	- sDefaultExecutor成员变量是SERIAL_EXECUTOR

	- SERIAL_EXECUTOR是SerialExecutor的实例

	```
	 /**
     * 默认线程池，默认为串行
     */
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    
	 /**
     * 串行线程池
     */
    private static final Executor SERIAL_EXECUTOR = new SerialExecutor();
	```

	- SerialExecutor实现了Executor接口，所以它也是一个执行器，内部有一个mTasks，类型是ArrayDeque，所以它是一个队列，并且不是固定容量的的队列，是无限扩容的，所以任务即使任务再多，也不会有超出128容量的限制

	- 其实SerialExecutor是对并行执行器THREAD_POOL_EXECUTOR的代理，它复写了execute，将添加的任务都添加到自己的任务队列，在每个任务执行完后，再执行下一个任务，所以就有了串行的效果，任务执行还是原来的THREAD_POOL_EXECUTOR，SerialExecutor起到了重置执行顺序的问题

	- scheduleNext()为执行队头的任务，第一次执行mActive为null，所以会马上执行一个任务，然后就开始串行执行，不断执行完一个任务后继续执行下一个任务

	```
	 /**
     * 顺序执行器，将线程池的执行包裹，所以即使线程池是有容量的线程池，经过SerialExecutor包裹，都是串行执行，相当于单线程执行
     */
    private static class SerialExecutor implements Executor {
        /**
         * 任务队列，让任务串行
         */
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<>();
        /**
         * 当前执行的任务
         */
        Runnable mActive;

        @Override
        public synchronized void execute(@NonNull final Runnable runnable) {
            //创建一个代理任务，包裹真正的任务，再将代理任务进队
            mTasks.offer(new Runnable() {
                @Override
                public void run() {
                    try {
                        runnable.run();
                    } finally {
                        //执行完一个任务后，自动执行下一个任务
                        scheduleNext();
                    }
                }
            });
            //第一次执行，马上执行任务
            if (mActive == null) {
                scheduleNext();
            }
        }

        /**
         * 执行下一个任务
         */
        protected synchronized void scheduleNext() {
            //获取下一个任务，如果有则执行
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
	```
	
	- THREAD_POOL_EXECUTOR说到是并行执行器，那么来看看

		1. THREAD_POOL_EXECUTOR的创建主要是在static代码块，并且THREAD_POOL_EXECUTOR是static修饰，所以属于类，所欲App中所有的AsyncTask都是共享线程池的

		2. Executor的具体参数通过构造方法传入，具体是如下
			- CORE_POOL_SIZE，CPU核心数，取当前的CPU的核心数
			- MAXIMUM_POOL_SIZE，线程池核心线程数，2-4之间,但是取决于CPU核数
			- MAXIMUM_POOL_SIZE，线程池最大线程数，CPU核数*2+1
			- KEEP_ALIVE_SECONDS，线程池空闲时，线程存活时间30s

		3. Executor设置allowCoreThreadTimeOut(true)，允许核心线程在空闲时允许销毁线程

	```
	
	/**
     * 获得当前CPU的核心数
     */
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    /**
     * 设置线程池的核心线程数2-4之间,但是取决于CPU核数
     */
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    /**
     * 设置线程池的最大线程数为 CPU核数*2+1
     */
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    /**
     * 设置线程池空闲线程存活时间30s
     */
    private static final int KEEP_ALIVE_SECONDS = 30;
	
	 /**
     * 任务线程池，static修饰，所以是类共享，所以多个AsyncTask会共享一个线程池
     */
    public static final Executor THREAD_POOL_EXECUTOR;
    
    static {
        //定义线程池，使用sPoolWorkQueue作为队列，如果超出任务执行数量，则抛出RejectedExecutionException
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        //允许线程池在没有任务时销毁线程
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }
	```
	
#### 解决串行问题

由于3.0后的AsyncTask默认的执行器是SerialExecutor，包装了并发执行器，而如果我们需要切换为并发执行器，则可以调用setDefaultExecutor(AsyncTask.THREAD_POOL_EXECUTOR)

```
//设置默认线程执行器为并发执行器，128容量队列
setDefaultExecutor(AsyncTask.THREAD_POOL_EXECUTOR)
```
	
如果只想某个AsyncTask使用串行，其他使用并行，则可以调用executeOnExecutor()来执行执行器执行

```
executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, params);
```

#### 解决AsyncTask内存泄露

AsyncTask内存泄露主要是因为AsyncTask的创建我们是创建了匿名内部类或者非静态的内部类，在Java中匿名内部类是隐式传入外部类的引用的，而AsyncTask一般是在Activity中使用，所以隐式持有了Activity的引用，如果AsyncTask执行太耗时，在Activity销毁时还在执行就会发生内存泄露，导致Activity无法回收，解决方案：

1. 将AsyncTask作为静态内部类使用，静态内部类不会隐式传入外部类的引用，那么如果AsyncTask中需要使用到外部类，例如Activity时，我们可以将Activity使用弱引用包装，保证Activity的回收

2.Activity的onDestory()被调用时，手动将AsyncTask调用cancel()取消任务，并且置空AsyncTask的引用

#### 解决AsyncTask并发线程池，阻塞队列超出128溢出抛出异常

AsyncTask3.0后提供了setDefaultExecutor(Executor exec)配置线程池执行器，如果想某次任务执行是指定线程池执行器则可以调用executeOnExecutor(Executor exec, Params... params)执行
	
#### AsyncTask执行流程

1. 类构造，static代码块先执行，先创建并行线程池，创建串行执行器代理并行执行器，并将串行执行器设置为默认执行器
2. 类构造方法，创建WorkerRunnable具体任务和FutureTask任务执行
3. 调用execute(params)，将任务提交给配置的默认执行器，并回调onPreExecute()
4. 执行器添加任务到队列，开始执行任务，任务执行完后执行下一个任务
5. 任务执行，回调doInBackground，任务执行完毕使用Handler发送消息到主线程，再回调onPostExecute
6. 如果doInBackground中调用了publishProgress(progress)更新进度，则使用Handler发送消息，在主线程中回调onProgressUpdate(progress)进度更新

#### 全部源码和注释

```
public abstract class AsyncTask<Params, Progress, Result> {
    private static final String LOG_TAG = "AsyncTask";

    /**
     * 获得当前CPU的核心数
     */
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    /**
     * 设置线程池的核心线程数2-4之间,但是取决于CPU核数
     */
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    /**
     * 设置线程池的最大线程数为 CPU核数*2+1
     */
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    /**
     * 设置线程池空闲线程存活时间30s
     */
    private static final int KEEP_ALIVE_SECONDS = 30;

    /**
     * 线程工厂，统一创建线程并配置
     */
    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        @Override
        public Thread newThread(@NonNull Runnable runnable) {
            //给每个生成的线程命名
            return new Thread(runnable, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    /**
     * 任务队列，最大容量为128，就是最多支持128个任务并发
     */
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * 任务线程池，static修饰，所以是类共享，所以多个AsyncTask会共享一个线程池
     */
    public static final Executor THREAD_POOL_EXECUTOR;

    static {
        //定义线程池，使用sPoolWorkQueue作为队列，如果超出任务执行数量，则抛出RejectedExecutionException
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        //允许线程池在没有任务时销毁线程
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }

    /**
     * 串行线程池
     */
    private static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    /**
     * Handler的消息类型Code，发送结果的消息类型
     */
    private static final int MESSAGE_POST_RESULT = 0x1;
    /**
     * Handler的消息类型Code，更新进度的消息类型
     */
    private static final int MESSAGE_POST_PROGRESS = 0x2;

    /**
     * 默认线程池，默认为串行
     */
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    /**
     * 主线程Handler
     */
    private static InternalHandler sHandler;

    /**
     * 任务实现
     */
    private final WorkerRunnable<Params, Result> mWorker;
    /**
     * 任务
     */
    private final FutureTask<Result> mFuture;

    /**
     * 当前的任务状态
     */
    private volatile AsyncTask.Status mStatus = AsyncTask.Status.PENDING;

    /**
     * 是否取消
     */
    private final AtomicBoolean mCancelled = new AtomicBoolean();
    /**
     * 任务是否执行了
     */
    private final AtomicBoolean mTaskInvoked = new AtomicBoolean();

    /**
     * 回调Handler
     */
    private final Handler mHandler;

    /**
     * 顺序执行器，将线程池的执行包裹，所以即使线程池是有容量的线程池，经过SerialExecutor包裹，都是串行执行，相当于单线程执行
     */
    private static class SerialExecutor implements Executor {
        /**
         * 任务队列，让任务串行
         */
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<>();
        /**
         * 当前执行的任务
         */
        Runnable mActive;

        @Override
        public synchronized void execute(@NonNull final Runnable runnable) {
            //创建一个代理任务，包裹真正的任务，再将代理任务进队
            mTasks.offer(new Runnable() {
                @Override
                public void run() {
                    try {
                        runnable.run();
                    } finally {
                        //执行完一个任务后，自动执行下一个任务
                        scheduleNext();
                    }
                }
            });
            //第一次执行，马上执行任务
            if (mActive == null) {
                scheduleNext();
            }
        }

        /**
         * 执行下一个任务
         */
        protected synchronized void scheduleNext() {
            //获取下一个任务，如果有则执行
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

    /**
     * 运行状态
     */
    public enum Status {
        /**
         * 等待
         */
        PENDING,

        /**
         * 运行
         */
        RUNNING,

        /**
         * 结束
         */
        FINISHED,
    }

    /**
     * 获取主线程Handler
     */
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }

    private Handler getHandler() {
        return mHandler;
    }

    /**
     * 外部配置任务执行器，可外部定制
     *
     * @param exec 执行器
     */
    public static void setDefaultExecutor(Executor exec) {
        sDefaultExecutor = exec;
    }

    //-------------------- AsyncTask构造方法 --------------------

    /**
     * 空参构造，取主线程的Handler回调
     */
    public AsyncTask() {
        this((Looper) null);
    }

    /**
     * 指定回调的Handler
     */
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * 可指定消息轮训器
     *
     * @param callbackLooper 轮训器
     */
    public AsyncTask(@Nullable Looper callbackLooper) {
        //没指定，或者指定为主线程的轮训器，则构造主线程的Handler，否则使用指定的轮训器创建
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
                ? getMainHandler()
                : new Handler(callbackLooper);
        //任务的具体执行体，WorkerRunnable实现了Callable接口，就是说可以返回结果的任务
        mWorker = new WorkerRunnable<Params, Result>() {
            @Override
            public Result call() throws Exception {
                //标识任务为已经执行了
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    //设置线程优先级为后台线程，多个线程并发时，无关紧要的线程分配的CPU时间将会减少，有利于主线程的处理
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //执行任务，并获取执行结果
                    result = doInBackground(mParams);
                    //将进程中未执行的命令，一并送往CPU处理
                    Binder.flushPendingCommands();
                } catch (Throwable throwable) {
                    //抛出了异常，设置任务已经被取消
                    mCancelled.set(true);
                    throw throwable;
                } finally {
                    //发送结果
                    postResult(result);
                }
                //返回结果
                return result;
            }
        };
        //将WorkerRunnable作为FutureTask任务去执行
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                //任务结束，任务结束了，call还没有被调用
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    //被取消时会抛出异常
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    //执行发生异常异常
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    //取消失败
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    //-------------------- AsyncTask构造方法 --------------------

    /**
     * 发送任务执行结果，如果没有发送过则发送
     */
    private void postResultIfNotInvoked(Result result) {
        //这里有点奇怪，这个方法调用在FutureTask的done方法，而mTaskInvoked标志在call()回调时就会设置为true
        //所以后面的if判断一直都是进不去的，感觉这个方法是多余的，除非call方法执行还没到mTaskInvoked设置成功之前就抛出了异常
        //这个if判断才成立
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }

    /**
     * 发送任务执行结果到主线程进行回调
     */
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<>(this, result));
        message.sendToTarget();
        return result;
    }

    /**
     * 获取任务运行状态
     */
    public final AsyncTask.Status getStatus() {
        return mStatus;
    }

    /**
     * 执行前回调
     */
    @MainThread
    protected void onPreExecute() {
    }

    /**
     * 任务执行回调（子线程执行）
     *
     * @param params 任务参数
     * @return 任务结果
     */
    @WorkerThread
    protected abstract Result doInBackground(Params... params);

    /**
     * 执行后回调
     *
     * @param result 执行结果
     */
    @SuppressWarnings({"UnusedDeclaration"})
    @MainThread
    protected void onPostExecute(Result result) {
    }

    /**
     * 进度更新
     *
     * @param values 进度
     */
    @SuppressWarnings({"UnusedDeclaration"})
    @MainThread
    protected void onProgressUpdate(Progress... values) {
    }

    /**
     * 任务已经执行完毕了，却发现被取消，则回调，并带上结果
     *
     * @param result 结果
     */
    @SuppressWarnings({"UnusedParameters"})
    @MainThread
    protected void onCancelled(Result result) {
        onCancelled();
    }

    /**
     * 任务未执行完，被取消，则回调
     */
    @MainThread
    protected void onCancelled() {
    }

    /**
     * 是否被取消了
     */
    public final boolean isCancelled() {
        return mCancelled.get();
    }

    /**
     * 取消任务
     *
     * @param mayInterruptIfRunning 是否强制取消，不等待执行完毕后再取消（马上打断）
     */
    public final boolean cancel(boolean mayInterruptIfRunning) {
        //设置任务取消的标志
        mCancelled.set(true);
        return mFuture.cancel(mayInterruptIfRunning);
    }

    /**
     * 获取执行结果
     *
     * @return 执行结果
     */
    public final Result get() throws InterruptedException, ExecutionException {
        return mFuture.get();
    }

    /**
     * 获取结果，并设定超时，如果超时还没获取到结果，则抛出异常
     *
     * @param timeout 超时时间
     * @param unit    超时时间单位
     */
    public final Result get(long timeout, TimeUnit unit) throws InterruptedException,
            ExecutionException, TimeoutException {
        return mFuture.get(timeout, unit);
    }

    /**
     * 执行任务
     *
     * @param params 执行参数
     */
    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

    /**
     * 提供执行器，执行任务
     *
     * @param exec   执行器，可以设置为内部的THREAD_POOL_EXECUTOR来实现并行，否则默认使用串行的执行器
     * @param params 任务参数
     */
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec, Params... params) {
        //处理状态，RUNNING正在执行不允许再调用执行，FINISHED已经结束了也不允许执行，所以AsyncTask只允许执行一次
        if (mStatus != AsyncTask.Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
                default:
                    break;
            }
        }
        //切换状态为运行中
        mStatus = AsyncTask.Status.RUNNING;
        //任务执行前，回调，onPreExecute是直接调用的，所以如果在子线程中调用AsyncTask的execute
        //onPreExecute的回调也是在子线程！
        onPreExecute();
        //配置任务参数
        mWorker.mParams = params;
        //将任务交给线程池执行
        exec.execute(mFuture);
        return this;
    }

    /**
     * 执行Runnable
     */
    @MainThread
    public static void execute(Runnable runnable) {
        sDefaultExecutor.execute(runnable);
    }

    /**
     * 更新进度，需要手动调用
     *
     * @param values 进度
     */
    @WorkerThread
    protected final void publishProgress(Progress... values) {
        //没有取消才更新
        if (!isCancelled()) {
            getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<>(this, values)).sendToTarget();
        }
    }

    /**
     * 结束任务
     *
     * @param result 结果
     */
    private void finish(Result result) {
        //结束任务时，如果被标记取消，则回调取消
        if (isCancelled()) {
            onCancelled(result);
        } else {
            //获取到结果了，并且没有取消，则回调任务结束
            onPostExecute(result);
        }
        //设置状态为结束
        mStatus = AsyncTask.Status.FINISHED;
    }

    /**
     * 内部Handler，负责从子线程中发送消息会主线程进行方法回调
     */
    private static class InternalHandler extends Handler {
        InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                //主线程通知，任务结果，获取到结果
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    //更新进度
                    result.mTask.onProgressUpdate(result.mData);
                    break;
                default:
                    break;
            }
        }
    }

    /**
     * 任务
     *
     * @param <Params> 参数类型
     * @param <Result> 结果类型
     */
    private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        /**
         * 任务参数
         */
        Params[] mParams;
    }

    /**
     * 任务执行结果包裹类，包装结果作为Handler发送的数据
     *
     * @param <Data> 任务结果类型
     */
    @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        /**
         * @param task 任务对象
         * @param data 执行结果
         */
        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
}
```

#### 总结

这次分析AsyncTask收获挺大的，以前看不懂主要是不了解Java并发包中的Callable、FutureTask和Executor线程池执行器，还有阻塞队列BlockingQueue。执行器的串行还使用了代理模式进行代理，将并行改为串行。