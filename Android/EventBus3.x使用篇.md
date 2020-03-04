#### EventBus3.x使用篇

EventBus是一个基于观察者模式的发布、订阅的框架。

- 一般我们在Android中使用它，对于对象难以接触的场景能充分解耦，而不用将对象一层层传入给调用方，例如Activity中，多个Fragment通信，多个组件间通信等。

- 以前我们组件间通信使用的是广播，广播是跨进程的，跨进程涉及IPC进程间通信会比较耗性能，而EventBus则是单进程间的，无需IPC，相比广播更轻量。

#### 添加依赖

```
implementation 'org.greenrobot:eventbus:3.1.1'
```

#### 添加混淆

```
-keepattributes *Annotation*
-keepclassmembers class * {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
 
# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```

#### 核心方法

- register(object)，接收者的注册方法。一般我们会在Activity、Fragment的onCreate()生命方法中调用。

- unregister(object)，接收者的解注销方法。一般我们会在Activity、Fragment的onDestroy()生命方法中调用。

- 使用@Subscribe注解标识接收方法。接收方法的参数必须是事件类对象。

- post(event)，发送一个JavaBean作为事件，为发送方调用。如果接收方在post()方法之后才注册，不能接收到事件，如果需要post()方法之后，接收方也能接收到事件，则需要使用postSticky(event)发送粘性事件。

#### 粘性事件方法

- postSticky(event)，发送粘性事件，能让接收方晚于发送方post()发送事件后也能接收到该粘性事件类型的**最后一个事件**。发送粘性事件，粘性事件会一直存在于内存中，下次再注册时会继续接收到该粘性事件，所以如果不想下次再接收到该粘性事件，需要调用removeStickyEvent(eventClass)方法将粘性事件移除。

- getStickyEvent(eventClass)，获取粘性事件。

- removeStickyEvent(eventClass)，移除粘性事件。

- removeStickyEvent(event)，移除粘性事件，如果你已经有event对象，需要移除，可以使用该重载方法。

#### 注册、解注册、订阅事件、发布事件

- 接收方：注册和解注册

```
@Override
public void onCreate() {
    super.onCreate();
    EventBus.getDefault().register(this);
}
 
@Override
public void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```

- 接收方：订阅，使用@Subscribe标记接收方法，接收方法的参数为事件类对象，接收方法必须为public

```
//定义登录事件
public class LoginEvent {
    private String uid;
    private String token;

    public LoginEvent(String uid, String token) {
        this.uid = uid;
        this.token = token;
    }

    //省略get、set方法
}

//订阅登录事件
@Subscribe()
public void onLogin(LoginEvent event) {
    String currentThreadName = Thread.currentThread().getName();
    System.out.println("当前线程名：" + currentThreadName);
    System.out.println("<收到了登录事件> uid:" + event.getUid() + "，token：" + event.getToken());
}
```

- 发送方：使用post()方法，发布事件

```
//创建事件
LoginEvent event = new LoginEvent("10086", "xxxxxxx1234");
//发送事件
EventBus.getDefault().post(event);
```

#### 订阅的多种线程模式

上面我们使用的@Subscribe()订阅事件，接收事件方法会在发送方的线程中调用，例如发送方在主线程，那么接收方就在主线程中接收到事件。如果发送方在子线程中发送事件，接收方就在子线程中接收。

如果我们在接口请求中发送事件（例如在子线程中），事件接收方法改变UI，那么肯定会收到不能在主线程外改变UI的异常。这时候我们就需要使用到线程模式的指定。

线程指定，我们只需要在接收方的@Subscribe()注解中，添加一个threadMode线程模式。线程模式有5种：

1. POSTING，默认指定线程模式就为POSTING，接收方法在发送方的线程中执行。

```
public class UpdateUnreadCountEvent {
    private int unreadCount;

    public UpdateUnreadCountEvent(int unreadCount) {
        this.unreadCount = unreadCount;
    }
    //...
}

/**
 * 子线程中发送事件，则在子线程中执行，主线程发送，则在主线程中执行。
 */
@Subscribe(threadMode = ThreadMode.POSTING)
public void onUpdateUnreadCount(UpdateUnreadCountEvent event) {
    String currentThreadName = Thread.currentThread().getName();
    System.out.println("当前线程名：" + currentThreadName);
    System.out.println("<收到了更新用户未读数事件> unreadCount：" + event.getUnreadCount());
}
```

2. MAIN，指定接收方法在主线程中调用，如果当前就处于主线程会直接执行，不会再使用Handler添加到任务队列中。

```
public class LoginEvent {
    private String uid;
    private String token;

    public LoginEvent(String uid, String token) {
        this.uid = uid;
        this.token = token;
    }
    //...
}

/**
 * 必定在主线程中执行
 */
@Subscribe(threadMode = ThreadMode.MAIN)
public void onLogin(LoginEvent event) {
    String currentThreadName = Thread.currentThread().getName();
    System.out.println("当前线程名：" + currentThreadName);
    System.out.println("<收到了登录事件> uid:" + event.getUid() + "，token：" + event.getToken());
}
```

3. MAIN_ORDERED，接收方法也是在主线程中执行，但是和MAIN模式有区别，就算当前就在主线程中，也会使用Handler添加到任务队列中，再执行。

```
public class ExitAppEvent {
}

/**
 * 每次发送都使用Handler去post一个任务到队列中，再执行
 */
@Subscribe(threadMode = ThreadMode.MAIN_ORDERED)
public void onExitApp(ExitAppEvent event) {
    String currentThreadName = Thread.currentThread().getName();
    System.out.println("当前线程名：" + currentThreadName);
    System.out.println("<收到了退出App事件>");
}
```

4. BACKGROUND，指定接收方法在线程池的子线程中执行，如果当前就处于子线程中，则直接执行，而不会再新建一个线程去执行。

```
public class ObtainUserInfoEvent {
    private String uid;

    public ObtainUserInfoEvent(String uid) {
        this.uid = uid;
    }
    //...
    
    /**
     * 子线程执行，如果已经在子线程了，直接执行
     */
    @Subscribe(threadMode = ThreadMode.BACKGROUND)
    public void onObtainUserInfo(ObtainUserInfoEvent event) {
        String currentThreadName = Thread.currentThread().getName();
        System.out.println("当前线程名：" + currentThreadName);
        System.out.println("<收到了更新用户信息事件> uid：" + event.getUid());
    }
}
```

5. ASYNC，同样是线程池的子线程中执行，但和BACKGROUND模式有区别，如果当前就处于子线程中，也会新建一个线程去执行。

```
public class LogoutEvent {
    private String uid;

    public LogoutEvent(String uid) {
        this.uid = uid;
    }
    //...
    
    /**
     * 子线程执行，不管当前是不是已经在子线程，都新开一个线程去执行
     */
    @Subscribe(threadMode = ThreadMode.ASYNC)
    public void onLogout(LogoutEvent event) {
        String currentThreadName = Thread.currentThread().getName();
        System.out.println("当前线程名：" + currentThreadName);
        System.out.println("<收到了登出事件> uid：" + event.getUid());
    }
}
```

#### 发送粘性事件和订阅粘性事件

如果事件发送比订阅早，但需要接收方订阅时依旧能接收到事件的最后一个，那么就需要使用到粘性事件，注意：粘性事件会一直存在于内存中，下一次再订阅依旧会受到事件接收的调用，如果不想则需要移除粘性事件。

1. 事件的注册和解注册和普通事件订阅是一致的，所以就不再贴出了。
2. 粘性事件的接收，需要将@Subscribe()注解添加一个sticky属性，指定为true，该属性默认为false。

```
public class LoginEvent {
    private String uid;
    private String token;

    public LoginEvent(String uid, String token) {
        this.uid = uid;
        this.token = token;
    }
    //...
}

@Subscribe(sticky = true, threadMode = ThreadMode.MAIN)
public void onLogin(MessageEvent event) {
    //UI操作...
}
```

3. 发送粘性事件，post(event)方法改为postSticky(event)即可。

```
LoginEvent event = new LoginEvent("10086", "xxxxxxx1234");
EventBus.getDefault().postSticky(event);
```

4. 获取粘性事件，一般用于不订阅，直接获取事件使用事件数据。或者调用removeStickyEvent移除事件（一般是收到事件后，马上移除）。

```
LoginEvent event = EventBus.getDefault().getStickyEvent(LoginEvent.class);
String uid = event.getUid();
//...其他处理
//或者取消事件
EventBus.getDefault().removeStickyEvent(event);
```

5. 移除粘性事件，分为事件对象移除或使用事件类型移除。

```
//使用事件对象移除
EventBus.getDefault().removeStickyEvent(event);
//使用事件类型移除
EventBus.getDefault().removeStickyEvent(LoginEvent.class);
```

#### 事件接收优先级和拦截事件传播

1. 事件的传播，还可以指定优先级，优先级越高，越早执行。

2. 并且如果事件是在POSTING线程模式中的，可以打断事件的传播。

```
//指定优先级，默认优先级为0，比onEventB早接收
@Subscribe(priority = 1, threadMode = ThreadMode.POSTING);
public void onEventA(MessageEvent event) {
	System.out.println("A：收到登录事件");
	//打断事件传播，仅限于POSTING线程模式
	EventBus.getDefault().cancelEventDelivery(event);
}

@Subscribe(threadMode = ThreadMode. POSTING);
public void onEventB(MessageEvent event) {
	System.out.println("B：收到登录事件");
}
```

#### EventBus配置和替换默认实例

上面我们获取EventBus实例都是使用EventBus.getDefault()方法获得，调用该方法获取的都是默认配置生成的EventBus，那么如果我们想自己配置需要怎么做的呢？EventBus给我们提供了EventBus.builder类进行配置，最后使用installDefaultEventBus()来替换默认实例。

```
EventBus eventBus = EventBus.builder()
    .logSubscriberExceptions(false)//默认为ture，是否记录，调用接收者响应事件的方法出现异常时的异常日志
    .logNoSubscriberMessages(false) //默认为ture，是否记录，发送者发布事件时没有接收者的日志 
    .sendNoSubscriberEvent(false) //默认为ture，当发送者发布事件时没有接收者时，是否将事件转化为 post一个NoSubscriberEvent事件
    .sendSubscriberExceptionEvent(false) // 默认为ture，调用接收者响应事件的方法出现异常时，是否 post一个SubscriberExceptionEvent事件
    .throwSubscriberException(BuildConfig.DEBUG) //默认为 false，调用接收者响应事件的方法出现异常时是否抛出EventBusException
    .eventInheritance(false) // 默认为ture，若发送者发布的事件是接收者的订阅事件的子类，是否将事件传递给接收者处理
    .ignoreGeneratedIndex(true) //默认为false，是否忽略Index索引
    .strictMethodVerification(true) //默认为false,是否严格认证 @Subscribe注解标注的接收者响应事件方法,如果 方法是0个或多于1个参数,或者是 非 public, 抽象的,静态的,会抛出EventBusException
    .installDefaultEventBus();//将生成的配置实例设置到Default单例，后续getDefault()获取到的都是该配置的EventBus实例
```

#### 添加索引替代反射

默认EventBus调用接收者的接收方法是使用反射调用，查找订阅方法也是使用反射，对于性能有一定影响，Android上使用，我们使用提供的annotationProcessor，编译时查找接收者的接收方法和接收者信息，生成索引，那么运行时则直接使用索引而避免了反射，效率更高。

- 增加annotationProcessor配置

```
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'me.wally.eventbus.sample.AppEventBusIndex' ]
            }
        }
    }
}

dependencies {
    implementation 'org.greenrobot:eventbus:3.1.1'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
```

- 给EventBus配置索引，调用addIndex添加生成的索引类AppEventBusIndex，如果其他Lib库也使用到，则也需要将索引类配置，通过installDefaultEventBus替换getDefault()的单例

```
EventBus.builder()
.addIndex(new AppEventBusIndex())
//其他类库的索引
.addIndex(new LibEventBusIndex())
.installDefaultEventBus();
EventBus eventBus = EventBus.getDefault();
//...后续使用
```

#### EventBus的优缺点

优点：

1. EventBus能代替广播为我们提供单进程的组件间通信，比广播，如果涉及多进程通信，还是使用广播方便。

2. Activity内多Fragment，避免通过Activity作为中介，查找Fragment耦合Activity。使用EventBus解耦。

缺点：

1. 组件化项目，如果多个Module之间通信，需要将Event类下沉到Base库，容易造成Event类膨胀。

2. 容易出现滥用，导致调用流程太松散，不好追踪调用流程。

#### 总结

EventBus不应该滥用，否则出现调用流程不清晰的问题，适当区域使用可以解耦。

例如详情页面点赞，外层列表页需要更新点赞状态和点赞数量等，如果使用startActivityForResult()跳转结束时返回点赞状态和点赞数量，多个界面都需要同步这些数据时，会产生耦合Activity。

如果还需要使用数据库记录数据，将数据库代码写在Activity，另外一个界面也需要时，容易代码臃肿。

此时使用EventBus能很简单的同步状态和数据，数据库操作也能单独一个类接收事件消息进行操作即可。