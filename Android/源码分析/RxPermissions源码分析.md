#### RxPermissions源码分析

说到安卓的权限申请，框架已经有很多了，自己也写过一个FastPermission，当时也参考了RxPermissions的代理权限结果给Fragment的设计，还加入了类似Glide的生命周期监听。

不过遗憾的是没有支持RxJava，虽然简单封装还是可以支持的，但是不支持批量的权限逐个申请再返回结果，只是一刀切批量申请完，一次性出结果。

RxPermissions有批量申请和逐个申请的Api，就寻思着究竟是怎么做的，那就顺着看一下源码吧，而且代码只有3个类，可以说是十分轻巧了。

#### 简单使用

分析源码前，先来看下基本使用，毕竟源码分析也是从使用的Api入手，不然直接看类会一团晕。

- 依赖

```
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}

dependencies {
    implementation 'com.github.tbruyelle:rxpermissions:0.10.2'
}
```

- request()，简单请求，只返回允许结果，允许为true，其他都是为false

```java
//创建RxPermissions实例
RxPermissions rxPermissions = new RxPermissions(this);
//申请牌照权限
rxPermissions
    .request(Manifest.permission.CAMERA)
    .subscribe(granted -> {
        if (granted) {
           //用户允许权限
        } else {
           //用户不允许权限
        }
    });
```

- rxPermissions.ensure(permission)，配合RxAndroid去请求，例如点击事件时请求权限，将点击事件转换为权限结果事件，需要用到compose操作符

```
RxView.clicks(findViewById(R.id.enableCamera))
	 //主要是这里，使用compose操作符，将数据源数据交给rxPermissions.ensure(permission)进行转换，转换结果为Boolean
    .compose(rxPermissions.ensure(Manifest.permission.CAMERA))
    .subscribe(granted -> {
        if (granted) {
           //用户允许权限
        } else {
           //用户不允许权限
        }
    });
```

- request()，传入多个权限一起申请

```
rxPermissions
	 //request传入多个需要申请的权限即可
    .request(Manifest.permission.CAMERA,
             Manifest.permission.READ_PHONE_STATE)
    .subscribe(granted -> {
        if (granted) {
           //用户允许所有权限
        } else {
           //用户不允许权限，这里不知道有多少个权限被拒绝，只要其中有一个被拒绝就会走到这里
        }
    });
```

- requestEach()，多个权限，逐个请求，并且返回Permission对象，而不是Boolean

```
rxPermissions
    .requestEach(Manifest.permission.CAMERA,
             Manifest.permission.READ_PHONE_STATE)
    .subscribe(permission -> {//逐个请求，多个权限会回调多次，例如这里请求了2个权限，则会回调2次
        if (permission.granted) {
           //权限被允许
        } else if (permission.shouldShowRequestPermissionRationale) {
           //权限被勾选不再提示
        } else {
           //权限被拒绝，这里我们可以弹窗提示用户等
        }
    });
```

- rxPermissions.requestEachCombined()，多个权限申请，组合为一个Permission对象，而不是一个权限对应一个

```
rxPermissions.requestEachCombined(Manifest.permission.CAMERA, Manifest.permission.READ_PHONE_STATE)
    .subscribe(permission -> {//这里只会回调一次
        if (permission.granted) {
           //所有权限都被允许了
        } else if (permission.shouldShowRequestPermissionRationale)
           //只要有一个权限勾选了不再提示，就会走到这里
        } else {
           //只要有一个权限被拒绝，就会走到这里
        }
    });
```

- 如果是配合RxAndroid，又是多个权限对应一个Permission对象，那么可以使用compse()连接rxPermissions.ensureEachCombined()转换

```
RxView.clicks(findViewById(R.id.enableCamera))
	 //主要是这里，使用compose操作符，将数据源数据交给rxPermissions.ensure(permission)进行转换，转换结果为Boolean
    .compose(rxPermissions.ensureEachCombined(Manifest.permission.CAMERA, Manifest.permission.READ_PHONE_STATE))
    .subscribe(permission -> {//这里只会回调一次
        if (permission.granted) {
           //所有权限都被允许了
        } else if (permission.shouldShowRequestPermissionRationale)
           //只要有一个权限勾选了不再提示，就会走到这里
        } else {
           //只要有一个权限被拒绝，就会走到这里
        }
    });
```

主要Api总结：

Api名 | 解释
:-: | :-: |
request() | 传入单个、多个权限，批量申请，返回结果为Boolean |
rxPermissions.ensure(permission) | 通过compose()操作符连接，效果和request一致 | 
requestEach(permissions)| 逐个申请权限，返回值为Permission，可以根据Permission的字段做更详细的判断 |
requestEachCombined(permission) | 批量申请权限，将结果组合到一个Permission对象，而不是一个权限对应一个权限，所有权限都允许才允许，只要有一个权限拒绝就为拒绝，只要有一个权限勾选了不再提醒则不再提醒 |

#### 类库结构

涉及的源码类：

1. Permission，权限模型实体，包含了权限名和申请结果等信息
2. RxPermissions，主要API入口和权限处理
3. RxPermissionsFragment，权限请求代理Fragment，接收onRequestPermissionsResult回调，所以不用耦合到请求者的Activity

#### 源码分析

1. RxPermissions的构造方法

	- 我们从RxPermissions的构造方法开始，发现调用getLazySingleton()方法获取Lazy对象，n内部缓存了RxPermissionsFragment代理Fragment的实例。2种构造方法都是获取他们的FragmentManager进行插入一个空布局（隐形）的Fragment。

	- 先使用Tag查找代理Fragment，如果没有找到就创建并插入到Activity，并缓存代理Fragment的实例，后面使用则直接获取缓存的实例即可，不用每次都使用Tag进行find查找。

	```
	/**
     * 权限申请代理Fragment懒加载和缓存
     */
    @VisibleForTesting
    Lazy<RxPermissionsFragment> mRxPermissionsFragment;

    /**
     * 以Activity，构造实例
     */
    public RxPermissions(@NonNull final FragmentActivity activity) {
        mRxPermissionsFragment = getLazySingleton(activity.getSupportFragmentManager());
    }

    /**
     * 以Fragment，构造实例
     */
    public RxPermissions(@NonNull final Fragment fragment) {
        mRxPermissionsFragment = getLazySingleton(fragment.getChildFragmentManager());
    }
    
    /**
     * 懒加载接口
     */
    @FunctionalInterface
    public interface Lazy<V> {
        /**
         * 获取懒加载的对象
         *
         * @return 缓存的对象
         */
        V get();
    }
    
    /**
     * 获取懒加载实例
     *
     * @param fragmentManager Fragment管理器
     */
    @NonNull
    private Lazy<RxPermissionsFragment> getLazySingleton(@NonNull final FragmentManager fragmentManager) {
        return new Lazy<RxPermissionsFragment>() {
            private RxPermissionsFragment rxPermissionsFragment;

            @Override
            public synchronized RxPermissionsFragment get() {
                //缓存实例，下次使用直接获取
                if (rxPermissionsFragment == null) {
                    rxPermissionsFragment = getRxPermissionsFragment(fragmentManager);
                }
                return rxPermissionsFragment;
            }
        };
    }

    /**
     * 获取代理Fragment
     *
     * @param fragmentManager Fragment管理器
     */
    private RxPermissionsFragment getRxPermissionsFragment(@NonNull final FragmentManager fragmentManager) {
        //查找Fragment实例
        RxPermissionsFragment rxPermissionsFragment = findRxPermissionsFragment(fragmentManager);
        boolean isNewInstance = rxPermissionsFragment == null;
        //没有找到则创建，再添加
        if (isNewInstance) {
            rxPermissionsFragment = new RxPermissionsFragment();
            fragmentManager
                    .beginTransaction()
                    .add(rxPermissionsFragment, TAG)
                    .commitNow();
        }
        return rxPermissionsFragment;
    }

    /**
     * 查找权限代理Fragment
     *
     * @param fragmentManager Fragment管理器
     */
    private RxPermissionsFragment findRxPermissionsFragment(@NonNull final FragmentManager fragmentManager) {
        return (RxPermissionsFragment) fragmentManager.findFragmentByTag(TAG);
    }
	```

2. request()请求方法

	- RxPermissions实例构造完毕，我们来看下最基础的请求权限方法，request()。
		1. 以一个对象TRIGGER发起一个数据源，使用compose()操作符连接ensure()方法，传入申请的权限

	```
	/**
     * 直接发起申请权限，批量申请权限
     *
     * @param permissions 申请的权限列表
     */
    @SuppressWarnings({"WeakerAccess", "unused"})
    public Observable<Boolean> request(final String... permissions) {
        return Observable.just(TRIGGER).compose(ensure(permissions));
    }
	```
	
	- 那么我来看下ensure()方法究竟做了什么。
		1. 创建了ObservableTransformer对象，Transformer是提供给compose()操作符进行链接，这样可以复用数据模型的转换，而不需要打断Rx的链式调用（平常我们封装都会将对象包裹一层再调用，这样就是断开Rx的链式调用）
		2. 调用了request()方法，传入上一个数据源发送的对象，就是TRIGGER对象，和申请的权限数组，再使用buffer操作符，将发送数据按申请权限的总数为一次数据发射的内容，所以数据模型从Observable<Permission>转换为了Observable<List<Permission>>。
		3. 最后使用flatMap铺平结果集，将数据结果类型转换为Observable<Boolean>，来看下里面做了什么，1）先判断权限结果是否为空，为空则使用Observable.empty()，发射完成的信息给订阅者。2）遍历Permissions数组，所有申请结果为允许才返回true，否则有一个为拒绝，则返回false。

	```
	/**
     * 批量申请权限Transformer，可以使用compose操作符连接，全部都授权了才返回true，否则为false，只会通知订阅者一次
     *
     * @param permissions 需要申请的权限
     */
    @SuppressWarnings("WeakerAccess")
    public <T> ObservableTransformer<T, Boolean> ensure(final String... permissions) {
        return new ObservableTransformer<T, Boolean>() {
            @Override
            public ObservableSource<Boolean> apply(Observable<T> o) {
                //申请权限
                return request(o, permissions)
                        //一次性申请，buffer指定一次发射的数量为权限列表数量，所以是一次性申请权限
                        .buffer(permissions.length)
                        .flatMap(new Function<List<Permission>, ObservableSource<Boolean>>() {
                            @Override
                            public ObservableSource<Boolean> apply(List<Permission> permissions) {
                                //申请的权限为空，直接通知订阅者的onComplete
                                if (permissions.isEmpty()) {
                                    return Observable.empty();
                                }
                                //所有权限都允许了，才返回true
                                for (Permission permission : permissions) {
                                    //是要有一个没有授权，则返回false
                                    if (!permission.granted) {
                                        return Observable.just(false);
                                    }
                                }
                                //所有都允许了
                                return Observable.just(true);
                            }
                        });
            }
        };
    }
	```
	
	- 上面ensure()方法主要是靠request(o, permissions)方法去申请权限，我们来看看
		1. 首先判断如果传入的权限列表为空则抛出异常。
		2. 调用oneOf(trigger, pending(permissions))方法，再使用flatMap()，将原来的结果类型Observable<Permission>接转换为结果类型Observable<Permission>。
		3. oneOf()方法中使用了merge()操作符，将pending()方法中的数据源和上面Observable.just(TRIGGER)数据源合并发射。
		4. pending()方法是判断权限是否都包含在了正在请求的权限列表中。其实oneOf()方法我觉得有点多余，为什么不直接Observable.just(TRIGGER)发射呢？

	```
	/**
     * 申请权限request中转
     *
     * @param trigger     原始数据源
     * @param permissions 申请的权限
     */
    private Observable<Permission> request(final Observable<?> trigger, final String... permissions) {
        if (permissions == null || permissions.length == 0) {
            throw new IllegalArgumentException("RxPermissions.request/requestEach requires at least one input permission");
        }
        //数据源一一匹配，确保是成对存在
        return oneOf(trigger, pending(permissions))
                .flatMap(new Function<Object, Observable<Permission>>() {
                    @Override
                    public Observable<Permission> apply(Object o) {
                        //真正申请权限的实现，每个权限都一个个去申请
                        return requestImplementation(permissions);
                    }
                });
    }
    
    /**
     * 数据源一一匹配，确保是成对存在，但是一般都是存在，这里判断是不是多余
     */
    private Observable<?> oneOf(Observable<?> trigger, Observable<?> pending) {
        if (trigger == null) {
            return Observable.just(TRIGGER);
        }
        return Observable.merge(trigger, pending);
    }
    
    /**
     * 过滤掉权限和结果数据源不匹配的情况
     *
     * @param permissions 申请的权限
     */
    private Observable<?> pending(final String... permissions) {
        for (String permission : permissions) {
            if (!mRxPermissionsFragment.get().containsByPermission(permission)) {
                return Observable.empty();
            }
        }
        return Observable.just(TRIGGER);
    }
	```
	
	- request(trigger, permissions)主要是调用了requestImplementation()方法，看方法名是申请权限的真正实现，来看看吧

		1. 首先创建了list集合和unrequestedPermissions集合，list集合的类型是List<Observable<Permission>>，他是将权限包装为一个可观察对象（数据源）Observable<Permission>，它是做了一个for循环，做了3种分类，1）已允许的权限，2）之前就被拒绝的权限，3）待申请的权限。
		2. 先从代理Fragment以权限名字符串获取一个PublishSubject，PublishSubject是Subject的子类，Subject类既可以作为Observable可观察者使用，也可以作为Observer观察者使用。如果获取不到，则新建一个PublishSubject，并保存到代理Fragment中。并将权限名，保存到unrequestedPermissions集合
		3. 判断unrequestedPermissions集合是否为空，不为空则代码有权限需要请求，调用requestPermissionsFromFragment()方法进行请求。
		4. 调用Observable.fromIterable()操作符，将上面保存的权限申请结果可观察者list集合中的数据源通过Observable.fromIterable()发送，再使用Observable.concat()操作符顺序并合并发送的数据。

	```
	/**
     * 真正申请权限
     *
     * @param permissions 申请的权限
     */
    @TargetApi(Build.VERSION_CODES.M)
    private Observable<Permission> requestImplementation(final String... permissions) {
        //权限申请前的结果
        List<Observable<Permission>> list = new ArrayList<>(permissions.length);
        //待申请的权限列表
        List<String> unrequestedPermissions = new ArrayList<>();
        //为每个权限创建一个数据源
        for (String permission : permissions) {
            mRxPermissionsFragment.get().log("Requesting permission " + permission);
            //加入已经被允许的权限数据源
            if (isGranted(permission)) {
                list.add(Observable.just(new Permission(permission, true, false)));
                continue;
            }
            //加入被拒绝的权限的数据源
            if (isRevoked(permission)) {
                list.add(Observable.just(new Permission(permission, false, false)));
                continue;
            }
            //获取权限申请存根，这种是为了避免快速请求多次，存入了多个结果数据源回调
            PublishSubject<Permission> subject = mRxPermissionsFragment.get().getSubjectByPermission(permission);
            //不存在则创建一个，并保存到代理Fragment
            if (subject == null) {
                //需要申请，添加到待申请的权限
                unrequestedPermissions.add(permission);
                subject = PublishSubject.create();
                mRxPermissionsFragment.get().setSubjectForPermission(permission, subject);
            }
            //加入待进行申请的权限数据源
            list.add(subject);
        }
        //如果存在需要申请的权限，则申请权限
        if (!unrequestedPermissions.isEmpty()) {
            //集合转为数组
            String[] unrequestedPermissionsArray = unrequestedPermissions.toArray(new String[unrequestedPermissions.size()]);
            //调用代理Fragment去申请权限
            requestPermissionsFromFragment(unrequestedPermissionsArray);
        }
        //发射允许和被拒绝的权限，concat顺序发送权限结果数据源，将他们的结果按顺序铺平发送
        return Observable.concat(Observable.fromIterable(list));
    }
    
    /**
     * 判断权限是否允许
     *
     * @param permission 目标权限
     */
    @SuppressWarnings("WeakerAccess")
    public boolean isGranted(String permission) {
        return !isMarshmallow() || mRxPermissionsFragment.get().isGranted(permission);
    }

    /**
     * 权限是否拒绝
     *
     * @param permission 目标权限
     */
    @SuppressWarnings("WeakerAccess")
    public boolean isRevoked(String permission) {
        return isMarshmallow() && mRxPermissionsFragment.get().isRevoked(permission);
    }

    /**
     * 判断当前运行的系统是否大于6.0
     */
    boolean isMarshmallow() {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.M;
    }
    
    @SuppressWarnings("WeakerAccess")
    public Observable<Boolean> shouldShowRequestPermissionRationale(final Activity activity, final String... permissions) {
        //如果当前运行的系统不是6.0，则不管，所以兼容不了国产6.0一下的ROM
        if (!isMarshmallow()) {
            return Observable.just(false);
        }
        return Observable.just(shouldShowRequestPermissionRationaleImplementation(activity, permissions));
    }

    /**
     * 判断是否权限是否被用户勾选了不再提示
     */
    @TargetApi(Build.VERSION_CODES.M)
    private boolean shouldShowRequestPermissionRationaleImplementation(final Activity activity, final String... permissions) {
        for (String permission : permissions) {
            //没有允许，如果用户勾选了不再提示shouldShowRequestPermissionRationale会返回false
            if (!isGranted(permission) && !activity.shouldShowRequestPermissionRationale(permission)) {
                return false;
            }
        }
        return true;
    }
	```
	
	- requestImplementation(permissions)方法，调用了requestPermissionsFromFragment()方法，让请求交给了RxPermissionsFragment代理Fragment。
		1. RxPermissionsFragment代理Fragment的requestPermissions()方法被调用，则调用了requestPermissions()方法申请权限（系统API）。

	```
	 /**
     * 权限申请请求码
     */
    private static final int PERMISSIONS_REQUEST_CODE = 42;
	
	/**
     * 调用申请权限的代理Fragment申请权限
     *
     * @param permissions 目标权限列表
     */
    @TargetApi(Build.VERSION_CODES.M)
    void requestPermissionsFromFragment(String[] permissions) {
        mRxPermissionsFragment.get().log("requestPermissionsFromFragment " + TextUtils.join(", ", permissions));
        mRxPermissionsFragment.get().requestPermissions(permissions);
    }
    
    /**
     * 开始申请权限
     *
     * @param permissions 需要申请的权限列表
     */
    @TargetApi(Build.VERSION_CODES.M)
    void requestPermissions(@NonNull String[] permissions) {
        //调用系统的申请权限API
        requestPermissions(permissions, PERMISSIONS_REQUEST_CODE);
    }
	```
	
3. RxPermissionsFragment代理Fragment分析

	- RxPermissionsFragment实际就是一个没有布局的Fragment(透明)，使用它的原因是通过它去申请权限，权限申请结果会回调到这个代理Fragment的onRequestPermissionsResult()，而不是调用到调用方的Activity，这样就可以将申请逻辑解耦出来，调用方只需要关注申请结果即可。

	- 先来看RxPermissionsFragment的构造，首先提供了无参构造，因为Activity恢复Fragment实例时是使用了反射构造，所以必须提供一个无参构造。
	- 复写了onCreate()生命周期回调方法，调用了setRetainInstance(true)方法，这个方法的作用是保持实例，避免屏幕旋转时，重建Fragment实例。

	```
	/**
     * 权限申请请求码
     */
    private static final int PERMISSIONS_REQUEST_CODE = 42;

    /**
     * 当前请求的权限结果数据源
     */
    private Map<String, PublishSubject<Permission>> mSubjects = new HashMap<>();
    /**
     * 是否打印Log
     */
    private boolean mLogging;
	
	public RxPermissionsFragment() {
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //保持实例，避免屏幕旋转时，重建Fragment实例
        setRetainInstance(true);
    }
	```
	
	- 权限代理Fragent主要的代码是在申请权限结果的回调

		1. 首先判断，权限请求的请求码，不是自己的请求码忽略掉，不处理。
		2. 组装一个shouldShowRequestPermissionRationale数组，主要是收集权限是否被勾选了不再提示。
		3. 调用了onRequestPermissionsResult()方法，进行处理结果。

	```
	/**
     * 权限申请回调
     *
     * @param requestCode  请求码
     * @param permissions  申请权限列表
     * @param grantResults 申请结果
     */
    @Override
    @TargetApi(Build.VERSION_CODES.M)
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        //忽略不是自己请求的权限回调
        if (requestCode != PERMISSIONS_REQUEST_CODE) {
            return;
        }
        //检查是否需要显示原理
        boolean[] shouldShowRequestPermissionRationale = new boolean[permissions.length];
        for (int i = 0; i < permissions.length; i++) {
            shouldShowRequestPermissionRationale[i] = shouldShowRequestPermissionRationale(permissions[i]);
        }
        //开始处理权限请求结果
        onRequestPermissionsResult(permissions, grantResults, shouldShowRequestPermissionRationale);
    }
	```
	
	- 权限结果处理，主要是onRequestPermissionsResult()方法中

		1. 遍历权限数组，以权限名从之前的mSubjects映射Map中找回PublishSubject，PublishSubject在这里其实相当于被外部订阅的可观察者，使用它发送权限结果即可通知到外部订阅的观察者。
		2. 判断subject是否为空，一般不会为null，如果为null，则抛出异常。
		3. 从mSubjects中移除缓存的PublishSubject，避免下次再次请求同样的权限时，出现问题。
		4. 判断权限是否被允许，将结果通过PublishSubject发送给外部的订阅者。

	```
	/**
     * 权限结果处理
     *
     * @param permissions                          权限列表
     * @param grantResults                         申请结果
     * @param shouldShowRequestPermissionRationale 是否被用户勾选了不再提示
     */
    void onRequestPermissionsResult(String[] permissions, int[] grantResults, boolean[] shouldShowRequestPermissionRationale) {
        for (int i = 0, size = permissions.length; i < size; i++) {
            log("onRequestPermissionsResult  " + permissions[i]);
            //用回权限映射找回数据源存根
            PublishSubject<Permission> subject = mSubjects.get(permissions[i]);
            if (subject == null) {
                //一般不会找不到，如果找不到则抛异常
                Log.e(RxPermissions.TAG, "RxPermissions.onRequestPermissionsResult invoked but didn't find the corresponding permission request.");
                return;
            }
            //移除存根
            mSubjects.remove(permissions[i]);
            //判断是否被允许了
            boolean granted = grantResults[i] == PackageManager.PERMISSION_GRANTED;
            //将结果发送回订阅者
            subject.onNext(new Permission(permissions[i], granted, shouldShowRequestPermissionRationale[i]));
            subject.onComplete();
        }
    }
	```
	
	- 代理Fragment中mSubjects映射Map中的一些方法。

		1. getSubjectByPermission()，以权限名查找PublishSubject可观察数据源。
		2. containsByPermission()，判断指定权限名的PublishSubject是否申请了（申请前会保存进来）
		3. setSubjectForPermission()，保存创建的PublishSubject和权限名进Map集合。

	```
	/**
     * 以权限名找回权限数据源存根
     *
     * @param permission 目标权限
     */
    public PublishSubject<Permission> getSubjectByPermission(@NonNull String permission) {
        return mSubjects.get(permission);
    }

    /**
     * 判断权限是否正在申请
     *
     * @param permission 目标权限
     */
    public boolean containsByPermission(@NonNull String permission) {
        return mSubjects.containsKey(permission);
    }

    /**
     * 保存权限申请存根
     *
     * @param permission 权限名
     * @param subject    权限申请存根
     */
    public void setSubjectForPermission(@NonNull String permission, @NonNull PublishSubject<Permission> subject) {
        mSubjects.put(permission, subject);
    }
	```
	
	- 以及一些权限检查允许还是拒绝等辅助方法

	```
	/**
     * 判断权限是否被允许
     *
     * @param permission 权限
     */
    @TargetApi(Build.VERSION_CODES.M)
    boolean isGranted(String permission) {
        final FragmentActivity fragmentActivity = getActivity();
        if (fragmentActivity == null) {
            throw new IllegalStateException("This fragment must be attached to an activity.");
        }
        return fragmentActivity.checkSelfPermission(permission) == PackageManager.PERMISSION_GRANTED;
    }

    /**
     * 权限被撤销
     *
     * @param permission 权限
     */
    @TargetApi(Build.VERSION_CODES.M)
    boolean isRevoked(String permission) {
        final FragmentActivity fragmentActivity = getActivity();
        if (fragmentActivity == null) {
            throw new IllegalStateException("This fragment must be attached to an activity.");
        }
        return fragmentActivity.getPackageManager().isPermissionRevokedByPolicy(permission, getActivity().getPackageName());
    }
    
    /**
     * 设置Log开关
     *
     * @param logging 是否允许打印Log
     */
    public void setLogging(boolean logging) {
        mLogging = logging;
    }
    
    /**
     * Log打印
     *
     * @param message 需要打印的信息
     */
    void log(String message) {
        if (mLogging) {
            Log.d(RxPermissions.TAG, message);
        }
    }
	```

4. 3种权限请求方法和数据源转换方法分析

	1. 上面分析的权限请求入口是request(permissions)，是一个返回值是Observable<Boolean>的请求方法，为批量申请权限，所有权限都允许才返回true，否则为false。上面已经分析过了，就不再分析了。

	```
	/**
     * 批量申请权限Transformer，可以使用compose操作符连接，全部都授权了才返回true，否则为false，只会通知订阅者一次
     *
     * @param permissions 需要申请的权限
     */
    @SuppressWarnings("WeakerAccess")
    public <T> ObservableTransformer<T, Boolean> ensure(final String... permissions) {
        return new ObservableTransformer<T, Boolean>() {
            @Override
            public ObservableSource<Boolean> apply(Observable<T> o) {
                //申请权限
                return request(o, permissions)
                        //一次性申请，buffer指定一次发射的数量为权限列表数量，所以是一次性申请权限
                        .buffer(permissions.length)
                        .flatMap(new Function<List<Permission>, ObservableSource<Boolean>>() {
                            @Override
                            public ObservableSource<Boolean> apply(List<Permission> permissions) {
                                //申请的权限为空，直接通知订阅者的onComplete
                                if (permissions.isEmpty()) {
                                    return Observable.empty();
                                }
                                //所有权限都允许了，才返回true
                                for (Permission permission : permissions) {
                                    //是要有一个没有授权，则返回false
                                    if (!permission.granted) {
                                        return Observable.just(false);
                                    }
                                }
                                //所有都允许了
                                return Observable.just(true);
                            }
                        });
            }
        };
    }
	```
	
	2. ensure(permissions)，提供一个ObservableTransformer，主要是提供给已经有Observable，但需要链式链接而提供的，一般通过compose()操作符连接。上面的request()方法也是手动发射一个Object对象发起一个Observable，再通过compose()操作符连接。request(permission)的主要逻辑都在这里，request(permissions)方法只是起到一起快速发起一个Observable的作用。
	3. requestEach(permissions)，和request(permissions)不同，批量申请权限，但是不全部判断权限是否全部允许才回调，而是每个权限的申请结果都回调外部的订阅者，返回结果为Observable<Permission>，我们可以对结果Permission对象获取更加精细的字段进行处理。
	4. ensureEach(permissions)，和ensure(permissions)类似，也是快速发起一个Observable的作用，和request(permissions)一样，requestEach(permissions)的主要逻辑都在ensureEach(permissions)。ensureEach()其实和ensure(permissions)的区别就是没有使用一个for循环统一判断权限Permission的granted属性为true才返回结果。这里回调订阅者是多次的。

	```
	/**
     * 申请权限Transformer，可以使用compose操作符连接，每个申请一次，所以会调用订阅者多次
     *
     * @param permissions 申请的权限列表
     */
    @SuppressWarnings("WeakerAccess")
    public <T> ObservableTransformer<T, Permission> ensureEach(final String... permissions) {
        return new ObservableTransformer<T, Permission>() {
            @Override
            public ObservableSource<Permission> apply(Observable<T> o) {
                return request(o, permissions);
            }
        };
    }
	```
	
	5. requestEachCombined(permissions)，和requestEach(permissions)类似，但是它将每个权限的请求结果都封装到一个Permission对象中。

	```
	/**
     * 直接发起申请权限，也是批量申请，但是返回结果不是Boolean而是Permission
     *
     * @param permissions 申请的权限列表
     */
    public Observable<Permission> requestEachCombined(final String... permissions) {
        return Observable.just(TRIGGER).compose(ensureEachCombined(permissions));
    }
	```
	
	6. requestEachCombined(permissions)，同样也是快速发起Observable的一个方法，具体实现都在ensureEachCombined(permissions)中。

	```
	/**
     * 和ensure类似，都是批量申请权限，但是返回结果不是Boolean，而是Permission对象
     *
     * @param permissions 申请的权限列表
     */
    public <T> ObservableTransformer<T, Permission> ensureEachCombined(final String... permissions) {
        return new ObservableTransformer<T, Permission>() {
            @Override
            public ObservableSource<Permission> apply(Observable<T> o) {
                //申请权限
                return request(o, permissions)
                        //buffer，一次性批量申请所有权限
                        .buffer(permissions.length)
                        .flatMap(new Function<List<Permission>, ObservableSource<Permission>>() {
                            @Override
                            public ObservableSource<Permission> apply(List<Permission> permissions) {
                                if (permissions.isEmpty()) {
                                    return Observable.empty();
                                }
                                //将结果直接发送
                                return Observable.just(new Permission(permissions));
                            }
                        });
            }
        };
    }
	```
	
5. Permission类分析

	1. Permission类有3个构造方法
		- 第一个构造方法，使用权限名称和是否允许权限创建。
		- 第二个构造方法，在第一个构造方法的基础上，增加了一个是否勾选不再提供的复选框。
		- 第三个构造方法，传入一个权限列表，通过了combineName(permissions)、combineGranted(permissions)、combineShouldShowRequestPermissionRationale(permissions)，决定3个字段。

	```
	public class Permission {
		/**
	     * 权限名
	     */
	    public final String name;
	    /**
	     * 是否允许
	     */
	    public final boolean granted;
	    /**
	     * 是否被用户勾选了不再提示
	     */
	    public final boolean shouldShowRequestPermissionRationale;
	
	    public Permission(String name, boolean granted) {
	        this(name, granted, false);
	    }
	
	    public Permission(String name, boolean granted, boolean shouldShowRequestPermissionRationale) {
	        this.name = name;
	        this.granted = granted;
	        this.shouldShowRequestPermissionRationale = shouldShowRequestPermissionRationale;
	    }
	
	    public Permission(List<Permission> permissions) {
	        name = combineName(permissions);
	        granted = combineGranted(permissions);
	        shouldShowRequestPermissionRationale = combineShouldShowRequestPermissionRationale(permissions);
	    }
	}
	```
	
	2. 3个字段，前2个构造方法是针对不组合权限请求的情况，第三个构造则是组合权限申请的情况，下面来分析使用的3个方法。

		1. combineName(permissions)，实际就是将每个权限名称，用StringBuilder拼接起来，中间使用逗号分隔。再赋值给name字段。

	```
	/**
     * 所有权限，用逗号分开组合为一个字符串
     *
     * @param permissions 权限组
     */
    private String combineName(List<Permission> permissions) {
        return Observable.fromIterable(permissions)
                .map(new Function<Permission, String>() {
                    @Override
                    public String apply(Permission permission) throws Exception {
                        return permission.name;
                    }
                }).collectInto(new StringBuilder(), new BiConsumer<StringBuilder, String>() {
                    @Override
                    public void accept(StringBuilder s, String s2) throws Exception {
                        if (s.length() == 0) {
                            s.append(s2);
                        } else {
                            s.append(", ").append(s2);
                        }
                    }
                }).blockingGet().toString();
    }
	```
	
	2. combineGranted(permissions)，将多个权限的允许结果都判断一遍，所有权限都允许了，granted字段才为true，否则为false。

	```
    /**
     * 判断所有权限是否都允许了
     *
     * @param permissions 权限列表
     * @return true则代表所有权限都允许，false代表权限列表中有一项不允许
     */
    private Boolean combineGranted(List<Permission> permissions) {
        return Observable.fromIterable(permissions)
                .all(new Predicate<Permission>() {
                    @Override
                    public boolean test(Permission permission) throws Exception {
                        return permission.granted;
                    }
                }).blockingGet();
    }
	```
	
	3. combineShouldShowRequestPermissionRationale(permissions)，判断权限是否有一项被勾选了不再提示，其中一项被勾选了不再提示，则shouldShowRequestPermissionRationale字段为true，所有都没勾选，才为false

	```
	/**
     * 判断权限列表中是否有一项被勾选了不再提示
     *
     * @param permissions 权限列表
     * @return true则表示有一项被勾选了不再提示，false则没有一项被勾选了不再提示
     */
    private Boolean combineShouldShowRequestPermissionRationale(List<Permission> permissions) {
        return Observable.fromIterable(permissions)
                .any(new Predicate<Permission>() {
                    @Override
                    public boolean test(Permission permission) throws Exception {
                        return permission.shouldShowRequestPermissionRationale;
                    }
                }).blockingGet();
    }
	```
	
#### 总结

这次分析RxPermissions，也是对之前的权限管理作为补充，之前之所以没有使用，原因是当时并不熟悉RxJava2，对库中大量的Rx操作符十分疑惑，所以才写了一个FastPermission库，同样也借鉴了使用隐形的Fragment作为申请权限的代理，将申请权限和发起方解耦。

RxPermissions的3种请求方法，也对之前实现FastPermission时做了补充，之前的权限都是一刀切，只要有一个权限拒绝了，回调则为拒绝，没有做到逐个申请的回调。