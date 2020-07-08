##### EventBus3源码分析

上篇，我们学习了EventBus3的使用方法，本篇一起来分析一下EventBus3的主要源码。

我们主要分析以下重点即可，其他的都是围绕这几个重点做补充：

1. EventBus实例创建
2. register注册流程
3. post事件发送流程
4. unregister解注册流程

#### EventBus实例创建

EventBus实例的创建方式有2种

- EventBus.getDefault()，获取默认配置的实例。使用的是很常见的Double Check和volatile保证单例。

```
static volatile EventBus defaultInstance;

/**
 * Double Check 方式单例
 */
public static EventBus getDefault() {
    EventBus instance = defaultInstance;
    if (instance == null) {
        synchronized (EventBus.class) {
            instance = EventBus.defaultInstance;
            if (instance == null) {
                instance = EventBus.defaultInstance = new EventBus();
            }
        }
    }
    return instance;
}
```

- 自定义配置，使用的是Builder建造者模式，可对EventBus的一些配置进行自定义，例如配置子线程回调订阅者方法的线程池执行器等。最后build()方法可按配置成EventBus实例，后续使用需要自己保存好实例，如果调用getDefault()获取的还是默认实例。如果希望让EventBus保存默认实例则可调用installDefaultEventBus()来保存实例为EventBus的默认实例。

```
/**
 * 自定义配置EventBus实例
 */
public static EventBusBuilder builder() {
    return new EventBusBuilder();
}

/**
 * EventBus实例自定义Builder配置
 */
public class EventBusBuilder {
	private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();
    boolean logSubscriberExceptions = true;
    boolean logNoSubscriberMessages = true;
    boolean sendSubscriberExceptionEvent = true;
    boolean sendNoSubscriberEvent = true;
    boolean throwSubscriberException;
    boolean eventInheritance = true;
    boolean ignoreGeneratedIndex;
    boolean strictMethodVerification;
    ExecutorService executorService = DEFAULT_EXECUTOR_SERVICE;
    List<Class<?>> skipMethodVerificationForClasses;
    List<SubscriberInfoIndex> subscriberInfoIndexes;
    Logger logger;
    MainThreadSupport mainThreadSupport;

    EventBusBuilder() {
    }
    
    //省略其他配置的set方法...
    
    /**
     * 配置子线程执行的线程池执行器
     */
    public EventBusBuilder executorService(ExecutorService executorService) {
        this.executorService = executorService;
        return this;
    }

    //省略其他配置的set方法...

	/**
	 * 将生成的实例安装到默认实例，后续则可以使用getDefault()获取回这个实例，注意只能安装一次
	 * 因此installDefaultEventBus()之前也不能调用getDefault()获取过实例，否则抛出异常
	 */
	public EventBus installDefaultEventBus() {
	    synchronized (EventBus.class) {
	        if (EventBus.defaultInstance != null) {
	            throw new EventBusException("Default instance already exists." +
	                    " It may be only set once before it's used the first time to ensure consistent behavior.");
	        }
	        EventBus.defaultInstance = build();
	        return EventBus.defaultInstance;
	    }
	}
	
	/**
	 * 按Builder配置生成EventBus实例
	 */
	public EventBus build() {
	    return new EventBus(this);
	}
}
```

#### 订阅者注册

订阅者的注册，通过调用register()方法，传入订阅者实例即可，一般我们为Activity、Fragment中使用。接下来分析注册流程。

- 获取订阅者的Class，通过subscriberMethodFinder.findSubscriberMethods()查找订阅者的所有订阅方法。
- 遍历每个订阅方法，为每个订阅方法进行注册。

```
/**
 * 订阅事件
 */
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    //查找订阅者所有的订阅方法
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        //遍历订阅的方法列表，对每个订阅方法都进行注册
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

来分析一下subscriberMethodFinder.findSubscriberMethods()。SubscriberMethodFinder类是一个专门查找订阅者方法的一个类。

- 反射查找订阅方法是一个很耗时的操作，所以EventBus也做了缓存METHOD_CACHE，搜索前先从缓存中找，找得到则马上使用。
- 接下来是2个分支，一个是apt编译器生成订阅信息索引的方法，一种是反射获取订阅者的订阅方法，ignoreGeneratedIndex标志位标识是否忽略apt生成的订阅信息索引，默认为false，所以会走else逻辑。
- findUsingInfo()方法，则是反射获取订阅者的订阅信息。
- 如果找不到订阅信息，证明订阅者没有声明订阅方法，抛出异常。找得到则将订阅信息缓存到METHOD_CACHE，然后再返回。

```
//订阅方法缓存
private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();

/**
 * 查找订阅者的订阅方法，将订阅信息封装在SubscriberMethod类中，多个方法，所以返回值为一个List集合
 *
 * @param subscriberClass 订阅者Class
 */
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    //先从缓存中查找，有则直接使用
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }
    //是否忽略apt生成的索引，ignoreGeneratedIndex默认为false
    if (ignoreGeneratedIndex) {
        subscriberMethods = findUsingReflection(subscriberClass);
    } else {
        //反射获取所有订阅的方法
        subscriberMethods = findUsingInfo(subscriberClass);
    }
    //没有任何一个订阅方法，抛出异常
    if (subscriberMethods.isEmpty()) {
        throw new EventBusException("Subscriber " + subscriberClass
                + " and its super classes have no public methods with the @Subscribe annotation");
    } else {
        //获取到所有订阅方法后，保存到缓存中，然后返回
        METHOD_CACHE.put(subscriberClass, subscriberMethods);
        return subscriberMethods;
    }
}
```

findSubscriberMethods()的主要逻辑都在findUsingInfo()上，继续追踪

- 第一步，调用prepareFindState()方法，获取当前线程的查找状态并初始化
- 第二步，判断findState.subscriberInfo字段是否为空，为空调用findUsingReflectionInSingleClass()，这个方法才是反射获取订阅者的订阅信息
- 第三步，配置父类Class信息，会自动忽略系统类来提高性能
- 最后，getMethodsAndRelease()，转移findState上保存的List<SubscriberMethod>，并对FindState中间对象回收

```
/**
 * 反射查找订阅者所有的订阅方法
 *
 * @param subscriberClass 订阅者Class
 */
private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
    //获取当前线程的查找状态
    FindState findState = prepareFindState();
    //初始化一些值
    findState.initForSubscriber(subscriberClass);
    while (findState.clazz != null) {
        //获取订阅信息
        findState.subscriberInfo = getSubscriberInfo(findState);
        //初始化时，findState.subscriberInfo为null
        if (findState.subscriberInfo != null) {
            SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
            for (SubscriberMethod subscriberMethod : array) {
                if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                    findState.subscriberMethods.add(subscriberMethod);
                }
            }
        } else {
            //反射获取订阅者的所有订阅方法
            findUsingReflectionInSingleClass(findState);
        }
        //配置父类Class信息，会自动忽略系统类来提高性能
        findState.moveToSuperclass();
    }
    //转移findState上保存的List<SubscriberMethod>，并对FindState中间对象回收
    return getMethodsAndRelease(findState);
}
```

分析prepareFindState()方法

- 很明显，FindState是复用对象的，作用就是避免频繁创建FindState类。
- 先从缓存池中查找FindState可用的对象，可用则返回。
- 不可用，则新建一个FindState类实例。

```
//对象池大小
private static final int POOL_SIZE = 4;
//FindState对象池
private static final FindState[] FIND_STATE_POOL = new FindState[POOL_SIZE];

/**
 * 获取一个查找状态
 */
private FindState prepareFindState() {
    synchronized (FIND_STATE_POOL) {
        //享元模式，从对象池中找，避免频繁创建FindState类
        for (int i = 0; i < POOL_SIZE; i++) {
            FindState state = FIND_STATE_POOL[i];
            if (state != null) {
                FIND_STATE_POOL[i] = null;
                return state;
            }
        }
    }
    //没有缓存对象可用，则创建一个
    return new FindState();
}
```

findState.initForSubscriber(subscriberClass)，则是初始化findState的初始化一些值

```
/**
 * 查找状态实体类
 */
static class FindState {
    Class<?> subscriberClass;
    boolean skipSuperClasses;
    SubscriberInfo subscriberInfo;

	//初始化findState的初始化一些值
	void initForSubscriber(Class<?> subscriberClass) {
	    this.subscriberClass = clazz = subscriberClass;
	    skipSuperClasses = false;
	    subscriberInfo = null;
	}
}
```

findState经过initForSubscriber()后，subscriberInfo字段被置为null，所以只能走else逻辑，继续分析findUsingReflectionInSingleClass()

- findUsingReflectionInSingleClass()方法，主要是使用反射来获取订阅者的Subscribe注解的方法，然后将方法的Method对象（有这个对象就能调用invoked调用订阅者方法），事件Class，配置的threadMode，priorty优先级，是否粘性sticky，封装到SubscriberMethod对象中，最后所有的订阅方法都在SubscriberMethod对象都收集到了findState.subscriberMethods集合中。这样订阅者的订阅信息都收集到了。

```
/**
 * 反射获取订阅者的所有订阅方法
 *
 * @param findState 查找状态
 */
private void findUsingReflectionInSingleClass(FindState findState) {
    Method[] methods;
    try {
        //使用getDeclaredMethods，反射获取所有方法，不会获取到父类中的方法，避免查找耗时，尤其是Activity，一般我们都是在子类上使用
        methods = findState.clazz.getDeclaredMethods();
    } catch (Throwable th) {
        // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
        methods = findState.clazz.getMethods();
        findState.skipSuperClasses = true;
    }
    //遍历订阅者的方法
    for (Method method : methods) {
        //获取修饰符
        int modifiers = method.getModifiers();
        //判断方法的修饰符是否为Public公开，并且不是static静态方法，
        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            //获取方法参数
            Class<?>[] parameterTypes = method.getParameterTypes();
            //限定订阅方法的方法参数为1个，就是event事件类
            if (parameterTypes.length == 1) {
                //判断方法是否加了@Subscribe注解，必须加了才处理
                Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                if (subscribeAnnotation != null) {
                    //获取第一个参数，就是事件event
                    Class<?> eventType = parameterTypes[0];
                    //检查是否已经添加过了，没有添加过才继续
                    if (findState.checkAdd(method, eventType)) {
                        //获取@Subscribe注解上的threadMode参数，就是事件回调的线程策略
                        ThreadMode threadMode = subscribeAnnotation.threadMode();
                        //创建SubscriberMethod对象，代表每个订阅方法的信息
                        //将@Subscribe注解上定义的事件类型、线程回调策略、回调优先级、是否粘性等字段保存到SubscriberMethod类中
                        //再将SubscriberMethod对象保存到findState.subscriberMethods订阅的方法列表中
                        findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                    }
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException("@Subscribe method " + methodName +
                        "must have exactly 1 parameter but has " + parameterTypes.length);
            }
        } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
            String methodName = method.getDeclaringClass().getName() + "." + method.getName();
            throw new EventBusException(methodName +
                    " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
        }
    }
}
```

回到SubscriberMethodFinder的findUsingInfo()方法，调用完findUsingReflectionInSingleClass()收集订阅者订阅信息后，再调用了moveToSuperclass()方法，其实就是配置父类Class信息，会自动忽略系统类来提高性能。

```
/**
 * 配置父类Class信息，会自动忽略系统类来提高性能
 */
void moveToSuperclass() {
    if (skipSuperClasses) {
        clazz = null;
    } else {
        clazz = clazz.getSuperclass();
        String clazzName = clazz.getName();
        //跳过系统的类（肯定不会有EventBus的东西），来提高性能
        if (clazzName.startsWith("java.") || clazzName.startsWith("javax.") || clazzName.startsWith("android.")) {
            clazz = null;
        }
    }
}
```

调用getMethodsAndRelease()，将findState中的subscriberMethods订阅信息集合转到一个ArrayList中，调用findState的recycle()重置对象字段，再将实例放到FIND_STATE_POOL对象池中复用。

```
/**
 * 将FindState上保存的订阅方法保存到一个List集合，并将FindState回收，将FindState类放到对象池中复用
 *
 * @param findState 保存了订阅信息的中间类
 */
private List<SubscriberMethod> getMethodsAndRelease(FindState findState) {
    //将订阅的方法信息保存到一个List集合
    List<SubscriberMethod> subscriberMethods = new ArrayList<>(findState.subscriberMethods);
    //回收，就是重置字段
    findState.recycle();
    synchronized (FIND_STATE_POOL) {
        for (int i = 0; i < POOL_SIZE; i++) {
            //保存到对象池中复用
            if (FIND_STATE_POOL[i] == null) {
                FIND_STATE_POOL[i] = findState;
                break;
            }
        }
    }
    //返回这个List集合
    return subscriberMethods;
}

/**
 * 查找状态实体类
 */
static class FindState {
	//...忽略其他字段和方法

	/**
     * 回收操作，重置字段
     */
    void recycle() {
        subscriberMethods.clear();
        anyMethodByEventType.clear();
        subscriberClassByMethodKey.clear();
        methodKeyBuilder.setLength(0);
        subscriberClass = null;
        clazz = null;
        skipSuperClasses = false;
        subscriberInfo = null;
    }
}
```

调用findSubscriberMethods()方法获取到订阅者的订阅信息列表后，我们回到register()方法，将订阅信息列表遍历，调用subscribe()方法，传入订阅对象和订阅信息。

- subscribe()方法的作用是：将订阅信息和订阅关系分拆到3个Map中，然后订阅者优先级排序，最后如果有粘性方法订阅，查找粘性事件的Map，回调订阅者方法。

先来了解下这3个Map是什么（很重要，后续事件回调都需要用到它们）：

- subscriptionsByEventType，事件类型和订阅它的订阅者的订阅信息（包含订阅者对象和订阅方法），一对多关系，一个事件类型，对应多个订阅者信息。

- typesBySubscriber，订阅者和它所订阅的事件类型映射，一对多关系，一个订阅者对应多个事件

- stickyEvents，粘性事件映射表，一对一关系，一个事件对应一个最近发送的事件对象

```
/**
 * 事件类型和订阅它的订阅者的订阅信息（包含订阅者对象和订阅方法），一对多关系，一个事件类型，对应多个订阅者信息
 */
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
/**
 * 订阅者和它所订阅的事件类型映射，一对多关系，一个订阅者对应多个事件
 */
private final Map<Object, List<Class<?>>> typesBySubscriber;
/**
 * 粘性事件映射表，一对一关系，一个事件对应一个最近发送的事件对象
 */
private final Map<Class<?>, Object> stickyEvents;

/**
 * 将订阅者和订阅方法执行绑定
 *
 * @param subscriber       订阅者
 * @param subscriberMethod 订阅方法信息对象
 */
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    //获取事件类型
    Class<?> eventType = subscriberMethod.eventType;
    //新建Subscription订阅信息类
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    //从subscriptionsByEventType中查找订阅信息列表
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    //没有，则创建一个，并放到subscriptionsByEventType中
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList<>();
        subscriptionsByEventType.put(eventType, subscriptions);
    } else {
        //已经调用了register方法订阅过了，不能重复订阅
        if (subscriptions.contains(newSubscription)) {
            throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                    + eventType);
        }
    }

    //优先级排序，优先级越高，越在List列表中靠前
    int size = subscriptions.size();
    for (int i = 0; i <= size; i++) {
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }
    //从typesBySubscriber中用订阅者获取它订阅的事件类型列表
    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
    //还没有注册过，所以时间类型列表subscribedEvents为空，为空则创建一个
    if (subscribedEvents == null) {
        subscribedEvents = new ArrayList<>();
        //创建完毕，再保存到Map
        typesBySubscriber.put(subscriber, subscribedEvents);
    }
    //将事件类型添加到事件类型列表中
    subscribedEvents.add(eventType);

    //判断订阅方法是否是粘性的
    if (subscriberMethod.sticky) {
        //判断是否需要发送子类事件时也发送父类事件，默认为true，如果事件POJO不会继承，建议设置为false来提高性能
        if (eventInheritance) {
            // Existing sticky events of all subclasses of eventType have to be considered.
            // Note: Iterating over all events may be inefficient with lots of sticky events,
            // thus data structure should be changed to allow a more efficient lookup
            // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                if (eventType.isAssignableFrom(candidateEventType)) {
                    Object stickyEvent = entry.getValue();
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            //粘性事件意思是订阅时，如果之前有发送过粘性事件则马上回调订阅方法
            //从粘性事件列表以事件类型中获取粘性事件POJO实例（所有粘性事件都会保存最近一份到内存）
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```

判断到存在粘性订阅方法时，调用checkPostStickyEventToSubscription()方法，检查粘性事件是否存在，存在则调用postToSubscription发送事件，回调订阅者的订阅方法。postToSubscription()方法我们在下面post()事件时分析。

```
/**
 * 检查粘性事件对象，以及将粘性事件发送到订阅者
 *
 * @param newSubscription 订阅信息
 * @param stickyEvent     粘性事件
 */
private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
    //没有发送过这个类型的粘性事件，那么就不做任何操作
    if (stickyEvent != null) {
        // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
        // --> Strange corner case, which we don't take care of here.
        //不为空，那么将这个粘性事件发送给订阅者
        postToSubscription(newSubscription, stickyEvent, isMainThread());
    }
}
```

register流程我们分析完了，工作原理主要是使用反射获取订阅者使用Subscribe注册标记的订阅方法，里面的耗时操作都加上了享元模式-对象池缓存或者是Map缓存，避免重复查找。

获取完订阅者的订阅信息列表后，将订阅信息和订阅关系分拆到很关键的3个Map中，最后判断是否有粘性事件订阅方法，再判断粘性事件是否存在，存在则发送事件，回调订阅者的订阅方法。

#### 发送事件

发送事件使用post()方法，发送粘性事件使用，下面来一起分析一下吧。

```
/**
 * 发送事件
 */
public void post(Object event) {
    //获取当前线程的发送状态
    PostingThreadState postingState = currentPostingThreadState.get();
    //获取发送状态的队列
    List<Object> eventQueue = postingState.eventQueue;
    //事件入队
    eventQueue.add(event);
    //如果没有在发送，则马上发送
    if (!postingState.isPosting) {
        //对postingState做一些配置
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            //队列不为空，则发送事件
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```

第一步，通过currentPostingThreadState.get()，获取当前线程的发送状态。currentPostingThreadState是一个ThreadLocal对象，所以每个线程都有一个PostingThreadState对象。

- 将事件放入postingState的eventQueue字段，eventQueue是事件队列。
- 判断postingState.isPosting字段，如果没有在发送，则马上发送。
- 通过一个while循环，调用postSingleEvent()，发送队列中的事件。
- 发送接触后，finally块再将必要字段重置。

```
/**
 * ThreadLocal类型，保存当前线程的发送状态类，每个线程都有一份PostingThreadState
 */
private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
    @Override
    protected PostingThreadState initialValue() {
        return new PostingThreadState();
    }
};

/**
 * 发送线程的状态
 */
final static class PostingThreadState {
    /**
     * 事件队列
     */
    final List<Object> eventQueue = new ArrayList<>();
    /**
     * 是否正在发送中
     */
    boolean isPosting;
    /**
     * 是否主线程
     */
    boolean isMainThread;
    Subscription subscription;
    Object event;
    boolean canceled;
}
```

postSingleEvent()方法，发送单个事件。

- 判断eventInheritance标志位，代表是否发送子类事件时也发送父类事件，然后调用postSingleEventForEventType()方法发送事件。
- postSingleEventForEventType()方法返回一个Boolean值，代表这个事件是否有订阅者订阅，如果没有，则发送一个NoSubscriberEvent事件。

```
/**
 * 发送一个事件
 *
 * @param event        事件对象
 * @param postingState 发送状态
 */
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    //判断是否发送子类事件时也发送父类事件
    if (eventInheritance) {
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            //发送事件
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        //发送事件
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    //处理没有订阅者的情况
    if (!subscriptionFound) {
        if (logNoSubscriberMessages) {
            logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
        }
        //没有订阅者，发送一个NoSubscriberEvent事件
        if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                eventClass != SubscriberExceptionEvent.class) {
            post(new NoSubscriberEvent(this, event));
        }
    }
}
```

分析postSingleEventForEventType()

- subscriptionsByEventType：事件类型和订阅它的订阅者的订阅信息（包含订阅者对象和订阅方法），一对多关系，一个事件类型，对应多个订阅者信息

- 从subscriptionsByEventType映射Map中，使用事件类型获取所有的订阅者信息，并赋值到subscriptions集合
- 判断subscriptions集合是否为空，为空则代表没有订阅者，否则则有订阅者
- 有订阅者，遍历subscriptions集合，调用postToSubscription()方法，为每个订阅者发送事件

```
/**
 * 按事件类型，发送单个事件
 *
 * @param event        事件对象
 * @param postingState 发送状态
 * @param eventClass   事件类型Class
 * @return 是否有订阅者订阅者这个事件
 */
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        //回查subscriptionsByEventType，获取要发送的这个事件的所有订阅者信息
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    //有订阅者订阅，则处理
    if (subscriptions != null && !subscriptions.isEmpty()) {
        //遍历订阅者，将事件发送给他们
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            //是否发送失败
            boolean aborted = false;
            try {
                //发送事件
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                //重置状态
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    //没有订阅者订阅
    return false;
}
```

看来postToSubscription是最终的发送方法，继续分析

- 从subscription订阅信息中，查找出订阅方法回调模式，进行分类，将事件代理给不同的Poster。

- invokeSubscriber()则是调用subscription订阅信息中的method对象的invoke()方法调用。

模式简介：

- POSTING模式，直接在当前线程反射调用订阅者的订阅方法
- MAIN模式，保证在主线程回调订阅者的订阅方法
- MAIN_ORDERED模式，保证主线程中回调订阅者的订阅方法，但是每次都是用Handler去post一个消息
- BACKGROUND模式，保证在子线程回调，如果当前已经在子线程，直接调用订阅者的订阅方法
- ASYNC模式，无论在哪个线程都让asyncPoster执行任务，所以就算已经在线程中了，也新开一个线程执行

```
/**
 * 发送事件到订阅者
 *
 * @param subscription 订阅信息
 * @param event        事件类型
 * @param isMainThread 当时是否在主线程
 */
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    //分类事件回调线程模式
    switch (subscription.subscriberMethod.threadMode) {
        //POSTING模式，直接在当前线程反射调用订阅者的订阅方法
        case POSTING:
            invokeSubscriber(subscription, event);
            break;
        case MAIN:
            //MAIN模式，保证在主线程回调订阅者的订阅方法
            //判断当前是否为主线程，直接调用即可
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                //当前不在主线程，将任务交给mainThreadPoster主线程发送器使用Handler发送
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case MAIN_ORDERED:
            //MAIN_ORDERED模式，保证主线程中回调订阅者的订阅方法，但是每次都是用Handler去post一个消息
            //所以即使当前已经是主线程了，也依然post一个消息给Handler排队执行
            if (mainThreadPoster != null) {
                mainThreadPoster.enqueue(subscription, event);
            } else {
                //不在安卓上使用EventBus，直接调用订阅者订阅方法
                // temporary: technically not correct as poster not decoupled from subscriber
                invokeSubscriber(subscription, event);
            }
            break;
        case BACKGROUND:
            //BACKGROUND模式，保证在子线程回调，如果当前已经在子线程，直接调用订阅者的订阅方法
            if (isMainThread) {
                //不在子线程，将任务交给backgroundPoster
                backgroundPoster.enqueue(subscription, event);
            } else {
                //已经在子线程了，直接调用
                invokeSubscriber(subscription, event);
            }
            break;
        case ASYNC:
            //ASYNC模式，无论在哪个线程都让asyncPoster执行任务，所以就算已经在线程中了，也新开一个线程执行
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
    }
}

/**
 * 发射调用订阅者的订阅方法
 *
 * @param subscription 订阅信息
 * @param event        事件对象
 */
void invokeSubscriber(Subscription subscription, Object event) {
    try {
        //拿到订阅信息的method对象，invoke()反射调用
        subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
    } catch (InvocationTargetException e) {
        handleSubscriberException(subscription, event, e.getCause());
    } catch (IllegalAccessException e) {
        throw new IllegalStateException("Unexpected exception", e);
    }
}
```

接下来，我们分析每种Poster：

- Poster接口，多种Poster，其实是采用了策略模式，抽取了接口Poster，不同策略的实现自己的enqueue()方法即可。

```
/**
 * 事件发送器
 */
interface Poster {
    /**
     * 入队一个事件发送任务
     *
     * @param subscription 订阅信息
     * @param event        事件对象
     */
    void enqueue(Subscription subscription, Object event);
}
```

- PendingPost消息，相当于Handler的Messsage对象，PendingPost类则是将每次要发送的事件和下一个事件用next字段保存，内部有一个static的pendingPostPool，其实是PendingPost的对象池，obtainPendingPost()从对象池中获取一个消息，releasePendingPost()则是将消息放回对象池中。

```
/**
 * 封装等待发送事件Api的类，相当于一个消息，类似Handler的Message对象，内部有对象池和事件
 */
final class PendingPost {
    /**
     * 队列
     */
    private final static List<PendingPost> pendingPostPool = new ArrayList<PendingPost>();
    /**
     * 事件
     */
    Object event;
    /**
     * 订阅者信息
     */
    Subscription subscription;
    /**
     * 下一个要发送的信息
     */
    PendingPost next;

    private PendingPost(Object event, Subscription subscription) {
        this.event = event;
        this.subscription = subscription;
    }

    /**
     * 从池子中获取一个消息
     *
     * @param subscription 订阅信息
     * @param event        事件
     */
    static PendingPost obtainPendingPost(Subscription subscription, Object event) {
        synchronized (pendingPostPool) {
            int size = pendingPostPool.size();
            if (size > 0) {
                PendingPost pendingPost = pendingPostPool.remove(size - 1);
                //对字段赋值
                pendingPost.event = event;
                pendingPost.subscription = subscription;
                pendingPost.next = null;
                return pendingPost;
            }
        }
        return new PendingPost(event, subscription);
    }

    /**
     * 回收
     *
     * @param pendingPost 包裹了订阅者信息的消息
     */
    static void releasePendingPost(PendingPost pendingPost) {
        //重置字段
        pendingPost.event = null;
        pendingPost.subscription = null;
        pendingPost.next = null;
        //将对象放回对象池
        synchronized (pendingPostPool) {
            //这里对池子大小做限制，不然会不断让池容量增长
            if (pendingPostPool.size() < 10000) {
                pendingPostPool.add(pendingPost);
            }
        }
    }
}
```

- PendingPostQueue，Poster发送事件使用的队列，维护了2个消息，队头消息和队尾消息，提供enqueue()方法消息入队，poll()方法获取下一个消息，这2个方法中，维护消息之间的链表关系。

```
/**
 * 发送事件队列，维护一个消息对象链表
 */
final class PendingPostQueue {
    /**
     * 队头消息
     */
    private PendingPost head;
    /**
     * 队尾
     */
    private PendingPost tail;

    /**
     * 入队
     *
     * @param pendingPost 下一个消息
     */
    synchronized void enqueue(PendingPost pendingPost) {
        if (pendingPost == null) {
            throw new NullPointerException("null cannot be enqueued");
        }
        //将事件绑定在，当前最后一个事件对象的next
        if (tail != null) {
            tail.next = pendingPost;
            tail = pendingPost;
        } else if (head == null) {
            //第一次，队头为空，赋值
            head = tail = pendingPost;
        } else {
            throw new IllegalStateException("Head present, but no tail");
        }
        notifyAll();
    }

    /**
     * 获取下一个消息
     */
    synchronized PendingPost poll() {
        PendingPost pendingPost = head;
        if (head != null) {
            head = head.next;
            if (head == null) {
                tail = null;
            }
        }
        return pendingPost;
    }

    synchronized PendingPost poll(int maxMillisToWait) throws InterruptedException {
        if (head == null) {
            wait(maxMillisToWait);
        }
        return poll();
    }
}
```

1. Android主线程Poster：HandlerPoster
2. 子线程串行Poster：BackgroundPoster
3. 子线程并发Poster：AsyncPoster

- Android主线程Poster：HandlerPoster，吱声继承Handler，实现Poster接口，enqueue()方法首先调用obtainPendingPost()获取一个消息，然后入队队列，判断如果没有执行则开始，然后就是用Handler的sendMessage()发消息，Handler发送消息，会主线程回调handleMessage()方法，用while循环，不断从队列中拿取消息，然后使用消息内的订阅信息，invokeSubscriber()调用订阅者的订阅方法，这样就实现了主线程回调。

```
/**
 * Android主线程事件发送器，继承Handler，实现Poster接口
 */
public class HandlerPoster extends Handler implements Poster {
    /**
     * 发送队列
     */
    private final PendingPostQueue queue;
    private final int maxMillisInsideHandleMessage;
    private final EventBus eventBus;
    /**
     * 是否正在运行，没有限制
     */
    private boolean handlerActive;

    protected HandlerPoster(EventBus eventBus, Looper looper, int maxMillisInsideHandleMessage) {
        super(looper);
        this.eventBus = eventBus;
        this.maxMillisInsideHandleMessage = maxMillisInsideHandleMessage;
        queue = new PendingPostQueue();
    }

    /**
     * 入队一个事件
     *
     * @param subscription 订阅信息
     * @param event        事件对象
     */
    @Override
    public void enqueue(Subscription subscription, Object event) {
        //获取一个等待发送的消息
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        synchronized (this) {
            //消息入队
            queue.enqueue(pendingPost);
            //没有正在执行，那么执行
            if (!handlerActive) {
                handlerActive = true;
                //使用Handler发消息
                if (!sendMessage(obtainMessage())) {
                    throw new EventBusException("Could not send handler message");
                }
            }
        }
    }

    @Override
    public void handleMessage(Message msg) {
        boolean rescheduled = false;
        try {
            long started = SystemClock.uptimeMillis();
            //死循环，从队列中获取下一个消息
            while (true) {
                PendingPost pendingPost = queue.poll();
                if (pendingPost == null) {
                    synchronized (this) {
                        //为空再检查一遍，因为在并发情况，使用synchronized同步
                        pendingPost = queue.poll();
                        //真的是获取不到了，那么就是没有
                        if (pendingPost == null) {
                            handlerActive = false;
                            return;
                        }
                    }
                }
                //反射调用订阅者的订阅方法，这里在Handler中回调，所以在主线程
                eventBus.invokeSubscriber(pendingPost);
                long timeInMethod = SystemClock.uptimeMillis() - started;
                //继续循环发送消息
                if (timeInMethod >= maxMillisInsideHandleMessage) {
                    if (!sendMessage(obtainMessage())) {
                        throw new EventBusException("Could not send handler message");
                    }
                    rescheduled = true;
                    return;
                }
            }
        } finally {
            handlerActive = rescheduled;
        }
    }
}
```

- 子线程串行Poster：BackgroundPoster，实现了Runnable和Poster接口，enqueue()方法中，也是先obtainPendingPost()获取一个消息，然后消息入队，判断没有执行，则马上执行。通过eventBus.getExecutorService()获取配置的线程池执行器，调用执行器的execute()，将自己传入，能传入是因为BackgroundPoster实现了Runnable接口，那么自然run()肯定被复写了。run()方法，也是启动一个while循环，从队列中获取下一个消息，由于run()方法回调已经在子线程中执行了，所以invokeSubscriber()调用回订阅者的订阅方法，自然在子线程执行。BackgroundPoster是单线程执行的，它是怎么做到的呢？其实是因为executorRunning这个标志位，任务开始前置为true，任务结束才置为false，在enqueue()入队方法时就返回executorRunning是否为false，为false则只添加到队列中，这样自然成为了单线程执行了。其实还使用了synchronized来保证不同线程去enqueue()入队时，在子线程中线程不安全的问题。

```
/**
 * 子线程串行回调事件订阅的发送器，实现了Runnable和Poster接口
 */
final class BackgroundPoster implements Runnable, Poster {
    /**
     * 发送队列
     */
    private final PendingPostQueue queue;
    private final EventBus eventBus;

    /**
     * 这个变量为了让线程池只能单线程执行
     */
    private volatile boolean executorRunning;

    BackgroundPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }

    @Override
    public void enqueue(Subscription subscription, Object event) {
        //获取一个消息，并将任务重新初始化
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        //加synchronized保证单线程执行
        synchronized (this) {
            //任务入队
            queue.enqueue(pendingPost);
            //如果没有执行，马上执行
            if (!executorRunning) {
                executorRunning = true;
                //获取配置的线程池执行器进行执行，将任务包裹到自身去执行
                eventBus.getExecutorService().execute(this);
            }
        }
    }

    @Override
    public void run() {
        try {
            try {
                //一直死循环执行
                while (true) {
                    //获取下一个消息，并设定1秒阻塞
                    PendingPost pendingPost = queue.poll(1000);
                    if (pendingPost == null) {
                        //加synchronized保证单线程执行，这里也加是因为要保证executorRunning的值不出错
                        synchronized (this) {
                            //同样要双重检查
                            pendingPost = queue.poll();
                            if (pendingPost == null) {
                                //没有事件了，跳出死循环
                                executorRunning = false;
                                return;
                            }
                        }
                    }
                    //调用订阅者
                    eventBus.invokeSubscriber(pendingPost);
                }
            } catch (InterruptedException e) {
                eventBus.getLogger().log(Level.WARNING, Thread.currentThread().getName() + " was interruppted", e);
            }
        } finally {
            //所有发送任务执行完毕，标志位置为false，下次再入队再继续执行
            executorRunning = false;
        }
    }
}
```

- 子线程并发Poster：AsyncPoster，实现了Runnable,、Poster接口，和BackgroundPoster类似，但是不是串行而是并行，enqueue()方法获取一个消息，将消息入队，然后eventBus.getExecutorService()获取到线程池执行器后，马上execute()传入自身，那么也一样，run()方法肯定被重写，run()方法中，获取下一个消息，马上调用invokeSubscriber()回调订阅者的订阅方法，因为run()回调是在子线程，所以回调订阅者时为子线程调用。

```
/**
 * 子线程并发执行器，实现了Runnable,、Poster接口
 */
class AsyncPoster implements Runnable, Poster {
    /**
     * 消息队列
     */
    private final PendingPostQueue queue;
    private final EventBus eventBus;

    AsyncPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }

    @Override
    public void enqueue(Subscription subscription, Object event) {
        //入队，获取一个缓存的PendingPost消息对象，重新初始化
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        //将消息入队
        queue.enqueue(pendingPost);
        //获取执行器执行
        eventBus.getExecutorService().execute(this);
    }

    @Override
    public void run() {
        //获取一个PendingPost消息
        PendingPost pendingPost = queue.poll();
        if (pendingPost == null) {
            throw new IllegalStateException("No pending post available");
        }
        //反射调用订阅者的订阅方法
        eventBus.invokeSubscriber(pendingPost);
    }
}
```

#### 订阅者解注册

- typesBySubscriber：订阅者和它所订阅的事件类型映射，一对多关系，一个订阅者对应多个事件

解注册，使用typesBySubscriber获取订阅者订阅的所有事件类型，判断订阅列表是否为空，不为空，则for循环订阅的事件列表，调用unsubscribeByEventType()，再将订阅者从typesBySubscriber中移除。

```
/**
 * 注销事件注册
 *
 * @param subscriber 订阅者
 */
public synchronized void unregister(Object subscriber) {
    //获取订阅者订阅的所有事件类型
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    //没有订阅过，忽略
    if (subscribedTypes != null) {
        //遍历订阅的事件类型，取消注册
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        //从订阅者列表中移除
        typesBySubscriber.remove(subscriber);
    } else {
        logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
    }
}
```

- 获取事件类型的所有订阅者列表
- 有订阅者，则遍历订阅者列表，找出需要解注册的订阅者，将订阅信息active标志设置为false，表示取消，再将订阅者从订阅列表中移除。

```
/**
 * 使用事件类型，注销订阅
 *
 * @param subscriber 订阅者
 * @param eventType  事件类型
 */
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    //获取事件类型的所有订阅者的订阅定系
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    //没有订阅者忽略
    if (subscriptions != null) {
        //遍历订阅者的订阅信息列表
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            //找到本次要取消订阅的订阅者信息
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                //设置为取消订阅
                subscription.active = false;
                //从列表中移除
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}
```

#### 发送粘性事件

发送粘性事件，使用的是postSticky()方法，先将事件放到stickyEvents这个Map中，保存事件类型和最近发送的事件对象。然后调用post()发送事件，流程和上面的Post发送事件流程一样。

- stickyEvents：粘性事件映射表，一对一关系，一个事件对应一个最近发送的事件对象

```
/**
 * 发送粘性事件
 *
 * @param event 事件
 */
public void postSticky(Object event) {
    //保存事件到粘性事件映射表
    synchronized (stickyEvents) {
        stickyEvents.put(event.getClass(), event);
    }
    //马上发送事件，和普通事件一样，粘性事件的特点是register()订阅时，马上检查是否有存在的粘性事件，有则马上回调
    post(event);
}
```

#### 总结

EventBus中可见到运用了好几种设计模式：

- 单例模式：Double Check保存EventBus实例
- 建造者模式：EventBusBuilder自定义EventBus配置
- 享元模式：对象池复用FindState对象，减少频繁创建
- 策略模式：不同的线程有不同的Poster对象发送事件

熟悉设计模式，更容易理解开源框架的逻辑，可以说设计模式是框架中常用的套路了~

EventBus中的subscriptionsByEventType、typesBySubscriber，这2个Map，设计得很巧妙，在注册、反注册、发送事件等3个重要流程中都起到了很重要的作用，subscriptionsByEventType记录了订阅者和订阅方法信息保存，在Post发送时，使用subscriptionsByEventType反查出订阅对应的订阅方法，利用不同的Poster在不同线程中调用。typesBySubscriber则是保存订阅者和订阅的事件类型的关系，在注册和反注册中起到注册表的作用。