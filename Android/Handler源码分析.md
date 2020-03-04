#### Handler源码分析

Handler对于Android开发者再熟悉不过了，也是面试题的常客了，所以了解Handler机制的源码就很有必要了，虽然Handler分析的文章已经有很多，但是自己总结一遍，印象才更深刻。

#### Handler简介

Handler机制，是Android中的一种消息传递机制，在开发中十分常用。由于Android从3.0开始不允许耗时操作在主线程中执行，必须在子线程中执行完后，将结果发送到主线程中更新UI。所以简单来讲Handler就是子线程和主线程通信的一种技术。

#### 常规使用

先是常规使用，Handler在主线程中创建，开启子线程处理耗时操作，再通过Handler发送消息到主线程，Handler的handleMessage()方法就会被回调，再更新UI。

```
public class MainActivity extends AppCompatActivity {
    private static final int WHAT_UPDATE_UI = 1000;

    private TextView vTextView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        vTextView = findViewById(R.id.text);
        Handler handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                //判断任务的种类，执行不同的操作
                switch (msg.what) {
                    case WHAT_UPDATE_UI:
                        vTextView.setText((String) msg.obj);
                        break;
                    default:
                        break;
                }
            }
        };
        
        //开启子线程，做耗时操作，结束后将新结果通过Handler发送到主线程更新UI
        Thread thread = new Thread() {
            @Override
            public void run() {
                super.run();
                
                //...处理耗时操作
                
                //使用Handler发送一个消息到主线程更新UI
                Message message = handler.obtainMessage();
                message.what = WHAT_UPDATE_UI;
                message.obj = "我是新文本";
                //发送新文本数据到主线程设置
                handler.sendMessage(message);
            }
        };
        thread.start();
    }
}
```

以及也很常用的，post()和postDelayed()。

```
//将任务推到队列中执行
Handler handler = new Handler();
handler.post(new Runnable() {
    @Override
    public void run() {
        //主线程更新操作...
    }
});

//将任务延时执行
handler.postDelayed(new Runnable() {
    @Override
    public void run() {
        //1秒后执行更新操作
    }
}, 1000);
```

#### 子线程中使用问题和解决方案

还有一种场景，就是子线程中创建Handler，让子线程成为轮训的线程，接收其他线程的消息，开发中并不多，但是特定场景会很有用，例如有一个一直执行的子线程，一直定时扫描着当前位置信息，到了指定范围，发送一个播放语音的消息的消息到主线程。

- 在子线程中，不能直接创建Handler实例，否则会抛出RuntimeException：Can't create handler inside thread ThreadName that has not called Looper.prepare()的异常。意思为：无法在没有调用Looper.prepare()的线程中创建Handler实例。

- 正确的做法是
	1. 先调用Looper.prepare()，准备Looper轮训器、MessageQueue消息队列。
	2. Looper.loop()，开始消息轮训。

- 主要原因是，Handler的创建需要使用到Looper轮训器，而轮训器是绑定在当前Thread线程的，而绑定操作在Looper.prepare()中，我们没有调用prepare，所以抛出异常。

```
Thread workThread = new Thread() {
    @Override
    public void run() {
        super.run();
        //准备Looper轮训器、MessageQueue消息队列
        Looper.prepare();
        Handler threadHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                //处理消息
                switch (msg.what) {
                    case WHAT_UPDATE_UI:
                        vTextView.setText((String) msg.obj);
                        break;
                    default:
                        break;
                }
            }
        };
        //开始消息轮训
        Looper.loop();
        //发送消息
        Message message = Message.obtain();
        message.what = WHAT_UPDATE_UI;
        message.obj = "子线程文本";
        threadHandler.sendMessage(message);
    }
};
```

#### Handler机制核心对象

- Handler，一般调用Api，sendMessage()、post()、postDelayed()来发送消息，子类覆写handleMessage()方法来处理消息。

- Looper，消息轮训器，它是线程唯一的，它不断在MessageQueue消息队列中获取消息，分发给对应的Handler。

- MessageQueue，消息队列，负责消息的存储，接收Handler发送的消息。

- Message，消息对象，有成员变量what标识消息类型，obj保存消息附带的数据，target保存发送该消息的Handler，并且有成员变量sPool维护下一个消息，形成链表复用消息。

#### Handler创建流程

接下来就是Handler源码分析了：

- Handler创建流程，一般我们会使用无参的构造方法，最终会走到Handelr的2个参数的构造方法，这个方法里面获取了当前线程的Looper轮训器，如果获取不到，则抛出异常，这个异常就是提示我们，需要调用Looper.prepare()准备Looper对象，并绑定到当前线程。

- 还有一个带Callback参数的构造方法，这个Callback是一个接口，使用该构造方法的意思是，单独提供一个消息处理的回调，callback参数不为空，则消息处理时不调用handleMessage()方法进行消息处理，而是调用这个设置的callback对象处理。

```
public Handler() {
    this(null, false);
}

public Handler(Callback callback) {
    this(callback, false);
}

public interface Callback {
    //消息处理方法
    public boolean handleMessage(Message msg);
}

public Handler(Callback callback, boolean async) {
    //..省略代码
    
    //获取当前线程的Looper
    mLooper = Looper.myLooper();
    //如果获取不到，则抛出异常
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
    //保存Looper轮训器中的消息队列
    mQueue = mLooper.mQueue;
    
    //...省略代码
}
```

- 既然Looper会绑定到当前线程，那么是怎么做的呢？来看下Looper.myLooper()方法。他从一个类型为ThreadLocal的sThreadLocal变量中获取，ThreadLocal是一个可以为每个线程都独立一份变量副本的类，可以先简单理解为有一个全局的Map，以线程对象为Key，获取到它自己那一份的变量，Looper既然使用了ThreadLocal，那么每个线程都有自己一份Looper对象。

```
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

- myLooper()方法是获取Looper对象，那么保存Looper对象呢？前面说到我们创建Handler，需要调用Looper.prepare()方法来创建Looper对象，那么来看下prepare()方法。

```
public static void prepare() {
    //quitAllowed为是否允许退出，默认为true，允许
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    //如果不为空，则抛出异常，所以prepare()只能调用一次
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //创建Looper实例，并将实例保存到sThreadLocal，这一步就是将当前线程和Looper对象绑定
    sThreadLocal.set(new Looper(quitAllowed));
}
```

- Looper实例创建了，我们来看下Looper的构造方法创建了什么。创建消息队列和保存当前线程对象。
保存Thread实例，主要是提供了isCurrentThread()和getThread()方法。isCurrentThread()判断当前线程是否是Looper绑定的线程。getThread()获取Looper绑定的线程。

```
private Looper(boolean quitAllowed) {
    //创建消息队列
    mQueue = new MessageQueue(quitAllowed);
    //保存当前线程对象
    mThread = Thread.currentThread();
}

//判断当前线程是否是Looper绑定的线程
public boolean isCurrentThread() {
    return Thread.currentThread() == mThread;
}

//获取Looper绑定的线程
public @NonNull Thread getThread() {
    return mThread;
}
```

- Looper的构造方法创建了MessageQueue，那么我们看一下MessageQueue的构造方法。主要是保存：是否允许退出的标识位和Natice层指针地址，MessageQueue暂告一段落，我们继续分析Handler的sendMessage()。

```
MessageQueue(boolean quitAllowed) {
    //保存是否允许退出的标识位
    mQuitAllowed = quitAllowed;
    //保存Natice层指针地址
    mPtr = nativeInit();
}
```

#### Handler消息发送、消息入队

- sendMessage()调用了一系列的重载方法，最终会调用到enqueueMessage()方法，enqueueMessage()方法中，将当前发送消息的Handler实例保存到Message对象的target字段中，后续回调消息处理会用到。最后调用消息队列queue的enqueueMessage()方法入队消息。

```
//马上执行的消息发送
public final boolean sendMessage(Message msg) {
    //调用了第二个sendMessageDelayed的重载方法
    //第二参数为延时时间，为0则代表不延时
    return sendMessageDelayed(msg, 0);
}

//支持延时执行的消息发送，第二参数为延时时间
public final boolean sendMessageDelayed(Message msg, long delayMillis) {
    //必要的越界处理
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    //调用sendMessageAtTime()，当前时间+延时时间=最终执行时间
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}

//指定执行时间的消息发送，第二参数为最终执行时间
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    //获取消息队列，如果获取不到抛出异常
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    //将消息入队
    return enqueueMessage(queue, msg, uptimeMillis);
}

//将消息入队到消息队列
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    //保存Handler实例到Message对象中，后续回调消息处理会用到
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    //消息入队
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

- enqueueMessage()方法，主要是排序好新消息的执行时间和队列中的消息执行时间在队列中的顺序。
消息发送、消息入队都处理了，那么还差一步就是循环了，前面说到Looper.loop()方法是开启轮训的方法，接下来就是分析它了

```
//入队消息，第二参数为执行的时间
boolean enqueueMessage(Message msg, long when) {
    //...省略代码

    //同步锁保证线程安全
    synchronized (this) {
        //正在退出，不执行    
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        //如果消息链表头为空，获取入队的消息是马上执行的，或者执行时间比当前链表头消息执行得早，则将新消息设置为链表头，并将原来的链表头消息保存到新消息的next字段
        //马上唤醒队列
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            //死循环，判断新消息的执行时间是否比队列中的所有消息的执行时间早，是的话，将新消息作为消息链表头
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        //唤醒队列
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

#### Loop轮训消息队列、消息分发

- Loop.loop()方法，我省略了无关代码，主要是获取消息队列，开启一个死循环，不断调用队列的next()方法获取下一个消息，获取到后调用消息的target字段的dispatchMessage()方法处理Message，前面说到target字段就是发送消息的Handler实例。

```
public static void loop() {
    //获取当前线程绑定的Looper对象，获取不到抛出异常
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    //获取消息队列
    final MessageQueue queue = me.mQueue;

    //...省略代码

    //开启一个死循环
    for (;;) {
        //从消息队列中获取下一个消息，该方法会阻塞线程
        Message msg = queue.next(); // might block
        //获取不到消息，不执行（注释意思是获取不到，则证明消息队列正在退出）
        if (msg == null) {
            //No message indicates that the message queue is quitting.
            return;
        }
        //...省略代码
        try {
            //分发消息回发送这个消息的Handler
            msg.target.dispatchMessage(msg);
        } finally {
            //...省略代码
        }
        //...省略代码
        //回收消息进行复用
        msg.recycleUnchecked();
    }
}
```

- 调用MessageQueue的next()方法将消息出队

```
public final class MessageQueue {
	//省略代码...

	Message next() {
	    final long ptr = mPtr;
	    //如果队列已经停止了，返回null
	    if (ptr == 0) {
	        return null;
	    }
	    //开启死循环
	    for (;;) {
	        synchronized (this) {
	            //获取当前时间
	            final long now = SystemClock.uptimeMillis();
	            Message prevMsg = null;
	            Message msg = mMessages;
	            if (msg != null && msg.target == null) {
	                 //msg == target 的情况只能是屏障消息，即调用postSyncBarrier()方法
	                //如果存在屏障，停止同步消息，异步消息还可以执行
	                do {
	                    prevMsg = msg;
	                    msg = msg.next;
	                } while (msg != null && !msg.isAsynchronous());  //找出异步消息，如果有的话
	            }
	            if (msg != null) {
	                if (now < msg.when) {
	                    //当前消息还没准备好(时间没到)
	                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
	                } else {
	                    // 消息已准备，可以取出
	                    if (prevMsg != null) {
	                        //有屏障，prevMsg 为异步消息 msg 的前一节点，相当于拿出 msg ,链接前后节点
	                        prevMsg.next = msg.next;
	                    } else {
	                        //没有屏障,msg 即头节点，将 mMessages 设为新的头结点
	                        mMessages = msg.next;
	                    }
	                    msg.next = null;  //断开即将执行的 msg
	                    msg.markInUse(); //标记为使用状态
	                    return msg;  //返回取出的消息，交给Looper处理
	                }
	            }
	            //队列已经退出
	            if (mQuitting) {
	                dispose();
	                //返回null后Looper.loop()方法也会结束循环
	                return null;
	            }
	    }
	}
	
	//省略代码...
}
```

- 经过消息队列的消息分发，回到Handler的dispatchMessage()方法，这个方法将Message对象分发到不同的判断分支。

```
public void dispatchMessage(Message msg) {
    //msg.callback不为空，则调用handleCallback()处理消息
    //这种分支对应post()、postDelayed()的发送方式
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        //Handler构造时指定的Callback不为空，则调用mCallback的handleMessage()处理
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //都没有，则调用我们重写的handleMessage
        handleMessage(msg);
    }
}

//post()系列方法发送消息的处理方法
private static void handleCallback(Message message) {
    //直接调用Message对象保存的callback字段的run()方法执行
    message.callback.run();
}

//一般我们都需要复写该方法进行消息处理
public void handleMessage(Message msg) {
}
```

- handleCallback()调用了Message对象保存的callback字段的run()方法，那么来看下callback对象的保存时机。post()和postDelayed()这2个方法都调用了getPostMessage()方法获取Message对象，真相就在getPostMessage()方法中，将Runnable对象保存到了获取到的Message对象的callback字段。

```
//发送一个Runnable消息，马上执行
public final boolean post(Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

//发送一个Runnable消息，延时执行
public final boolean postDelayed(Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}

//获取post系列方法的Message种类
private static Message getPostMessage(Runnable r) {
    //复用一个消息，和普通sendMessage()系列方法一样
    Message m = Message.obtain();
    //重点在这里，将Runnable任务保存到message对象的callback，后续分发消息时会调用这个callback字段
    m.callback = r;
    return m;
}
```

##### Message消息的复用

一般我们获取Message会调用Handler的obtainMessage()方法，这个方法是获取一个复用的Message对象，内部采用享元模式复用Message对象，在Android中，View绘制，Activity生命周期，都是使用Handler发送Message实现，如果每次都new一个消息对象，肯定是十分消耗内存的，也容易产生GC垃圾回收导致卡顿。

```
//获取一个复用的Message消息对象
public final Message obtainMessage() {
    //调用Handler参数的obtain()方法获取消息
    return Message.obtain(this);
}

//获取绑定Handler的Message消息对象
public static Message obtain(Handler h) {
    //调用无参obtain()获取消息
    Message m = obtain();
    m.target = h;
    return m;
}
```

- Message消息类

```
public final class Message implements Parcelable {
    //消息类型
    public int what;

    //消息附带的数据
    public Object obj;
    
    //同步锁对象
    public static final Object sPoolSync = new Object();
    
    //对象池，链头
    private static Message sPool;
    
    //链表，下一个消息对象
    Message next;
    
    //正在使用的标志
    static final int FLAG_IN_USE = 1 << 0;

    /** If set message is asynchronous */
    static final int FLAG_ASYNCHRONOUS = 1 << 1;

    /** Flags to clear in the copyFrom method */
    static final int FLAGS_TO_CLEAR_ON_COPY_FROM = FLAG_IN_USE;
    
    //对象池最大大小，只缓存50个Message对象
    private static final int MAX_POOL_SIZE = 50;
    
    //从缓存池中获取
    public static Message obtain() {
        synchronized (sPoolSync) {
            //对象池有数据，直接复用        
            if (sPool != null) {
                //获取表头消息
                Message m = sPool;
                //将表头的下一个消息替换到表头
                sPool = m.next;
                //断开原始表头的next字段
                m.next = null;
                //清除掉是否正在使用的标志
                m.flags = 0; // clear in-use flag
                //返回消息对象，减小对象池大小
                sPoolSize--;
                return m;
            }
        }
        //没有消息对象可以复用，创建一个
        return new Message();
    }
}

//重置对象字段，并将对象放回对象池
void recycleUnchecked() {
    // Mark the message as in use while it remains in the recycled object pool.
    // Clear out all other details.
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;
    synchronized (sPoolSync) {
        //池子没有满，则放到池中    
        if (sPoolSize < MAX_POOL_SIZE) {
            //将表头放到表尾
            next = sPool;
            //再将消息放到表头
            sPool = this;
            //池子放入了消息，所以对象池大小自增
            sPoolSize++;
        }
    }
}
```

#### 主线程为什么不需要调用Looper.prepare()和Looper.loop()

我们平常在主线程使用Handler时，并没有调用过Looper.prepare()和Looper.loop()这2个方法，为什么创建Handler时不会抛出异常呢？

原因就是创建Handler时，调用Looper.myLooper()获取主线程绑定的Looper不为空，所以没有抛出异常。经过Looper类中查找发现，除了Looper.prepare()之外，还有一个prepareMainLooper()的方法。

prepareMainLooper()方法的注释，意思大概就是，创建主线程的Looper对象，该方法由Android框架在主线程自动调用，我们不应该主动调用该方法。

```
/**
 * Initialize the current thread as a looper, marking it as an
 * application's main looper. The main looper for your application
 * is created by the Android environment, so you should never need
 * to call this function yourself.  See also: {@link #prepare()}
 */
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

那么什么时候会调用prepareMainLooper()方法呢，AndroidStudio点击方法查找调用链，我们发现在ActivityThread中有调用。ActivityThread是Android程序的主线程，main方法则是启动的方法，我们看到先是调用了Looper.prepareMainLooper()，初始化主线程的Looper。再调用了Looper.loop()开启主线程轮训。

```
public final class ActivityThread extends ClientTransactionHandler {
	//main方法，App进程的主入口
	public static void main(String[] args) {
	    //省略代码...
	
	    //初始化主线程的Looper
	    Looper.prepareMainLooper();
	
	    ActivityThread thread = new ActivityThread();
	    thread.attach(false, startSeq);
	    
	    //省略代码...
	
	    if (sMainThreadHandler == null) {
	        sMainThreadHandler = thread.getHandler();
	    }
	    
	    //开启主线程轮训
	    Looper.loop();
	
	    throw new RuntimeException("Main thread loop unexpectedly exited");
	}
}
```

#### 总体流程

- Handler创建，获取当前线程绑定的Looper对象，我们需要调用Looper.perare()方法创建Looper对象和消息队列MessageQueue，再调用Looper.loop()方法开始轮训，轮训实际是开启了一个死循环，不断从消息队列中获取消息，并调用消息保存的Handler实例的消息处理方法dispatchMessage()进行消息分发。发送消息则是将消息入队到消息队列中，轮训器轮训到新的消息后，又开始消息分发，Handler、Looper、MessageQueue形成一个三角环形链路，不断处理我们发送的消息。