#### Android App前后台监控

说到App的前后台监控，有什么使用场景呢？例如：

1. IM模块，收到消息时，需要判断当前App是否在前台，如果在前台则震动一下提醒用户，如果在后台则发送一条通知提醒用户。

2. 收到Push推送，需要判断App是否存活，如果存活则直接跳转到目标界面，如果不存活则先启动App，再跳转到目标页面。

#### 有没有简便的Api

- iOS上的AppDelegate

	1. **applicationDidEnterBackground**方法回调提醒App处于后台。
	
	2. **applicationWillEnterForeground**方法回调提醒App处于前台。

- Android的Application
	1. 很遗憾，没有！

既然没有，有没有别的法子？

#### 解决方案

其实获取方法，有好几种：

1. 通过ActivityManager.getRunningTasks()获取当前运行的task列表，需要遍历，而且需要申请android.permission.GET_TASKS权限，而且5.0后被废弃，5.0以上版本失效。

2. 5.0后，ActivityManager提供了getRunningAppProcesses()方法获取正在运行的应用程序，同样需要遍历，但不需要申请权限，但据说8.0后失效了。

3. Application注册ActivityLifecycleCallbacks，获取App上的所有Activity生命周期改变的回调，对onActivityStarted和onActivityStopped做计数即可，如果计数为0代表为后台，大于0则为前台，需要Api14以上，就是4.0以上可使用。一般现在兼容都在4.4，所以基本没有兼容性问题，并且不需要申请权限。

#### 方案选择

基于申请权限和不需要遍历的性能消耗，我选择了方案3。

新建一个单例类AppMonitor，在Application中初始化，并注册ActivityLifecycleCallbacks回调，我们再对生命周期回调进行计数，并且提供我们自定义的回调即可。

1. 提供initialize方法初始化AppMonitor。

2. 提供Callback接口
	- 提供onAppForeground回调，表示App切换到前台。
	- 提供onAppBackground回调，表示App切换到后台。
	- 提供onAppUIDestroyed回调，表示App所有界面都销毁了，可以认为就是App被“杀死”了。

```
public class AppMonitor {
    /**
     * 注册了的监听器
     */
    private List<Callback> mCallbacks;
    /**
     * 是否初始化了
     */
    private boolean isInited;
    /**
     * 活跃Activity的数量
     */
    private int mActiveActivityCount = 0;
    /**
     * 存活的Activity数量
     */
    private int mAliveActivityCount = 0;

    public static AppMonitor get() {
        return SingleHolder.INSTANCE;
    }

    private static final class SingleHolder {
        private static final AppMonitor INSTANCE = new AppMonitor();
    }

    public void initialize(Context context) {
        if (isInited) {
            return;
        }
        mCallbacks = new CopyOnWriteArrayList<>();
        registerLifecycle(context);
        isInited = true;
    }

    /**
     * 注册生命周期
     */
    private void registerLifecycle(Context context) {
        Application application = (Application) context.getApplicationContext();
        application.registerActivityLifecycleCallbacks(new Application.ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                mAliveActivityCount++;
            }

            @Override
            public void onActivityStarted(Activity activity) {
                mActiveActivityCount++;
                notifyChange();
            }

            @Override
            public void onActivityResumed(Activity activity) {
            }

            @Override
            public void onActivityPaused(Activity activity) {
            }

            @Override
            public void onActivityStopped(Activity activity) {
                mActiveActivityCount--;
                notifyChange();
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                mAliveActivityCount--;
                notifyAppAliveChange();
            }
        });
    }

    /**
     * 判断App是否活着
     */
    public boolean isAppAlive() {
        return mAliveActivityCount > 0;
    }
    
    /**
     * 判断App是否在前台
     */
    public boolean isAppForeground() {
        return mActiveActivityCount > 0;
    }

    /**
     * 判断App是否在后台
     */
    public boolean isAppBackground() {
        return mActiveActivityCount <= 0;
    }

    /**
     * 通知监听者
     */
    private void notifyChange() {
        if (mActiveActivityCount > 0) {
            for (Callback callback : mCallbacks) {
                callback.onAppForeground();
            }
        } else {
            for (Callback callback : mCallbacks) {
                callback.onAppBackground();
            }
        }
    }

    /**
     * 通知监听者界面销毁
     */
    private void notifyAppAliveChange() {
        if (mAliveActivityCount == 0) {
            for (Callback callback : mCallbacks) {
                callback.onAppUIDestroyed();
            }
        }
    }

    public interface Callback {
        /**
         * 当App切换到前台时回调
         */
        void onAppForeground();

        /**
         * App切换到后台时回调
         */
        void onAppBackground();

        /**
         * App所有界面都销毁了
         */
        void onAppUIDestroyed();
    }

    public static class CallbackAdapter implements Callback {

        @Override
        public void onAppForeground() {
        }

        @Override
        public void onAppBackground() {
        }

        @Override
        public void onAppUIDestroyed() {
        }
    }

    /**
     * 注册回调
     */
    public void register(Callback callback) {
        if (mCallbacks.contains(callback)) {
            return;
        }
        mCallbacks.add(callback);
    }

    /**
     * 注销回调
     */
    public void unRegister(Callback callback) {
        if (!mCallbacks.contains(callback)) {
            return;
        }
        mListener.remove(callback);
    }
}
```

#### 使用

- 初始化

```
public class MyApplication extends Application {
@Override
    public void onCreate() {
        super.onCreate();
        //前后台监听初始化
        AppMonitor.get().initialize(this);
    }
}
```

- 注册监听和注销监听

```
AppMonitor.Callback callback = new AppMonitor.Callback() {
    @Override
    public void onAppForeground() {
        //App切换到前台
    }

    @Override
    public void onAppBackground() {
        //App切换到后台
    }

    @Override
    public void onAppUIDestroyed() {
        //App被杀死了
    }
};
//注册回调
AppMonitor.get().register(callback);
//注销回调
AppMonitor.get().unRegister(callback);
```

- 判断App是否在前台

```
boolean isAppForeground = AppMonitor.get().isAppForeground();
```

- 判断App是否在后台

```
boolean isAppBackground = AppMonitor.get().isAppBackground();
```

- 判断App是否存活（所有页面都销毁了）

```
boolean isAlive = AppMonitor.get().isAppAlive();
```

#### 总结

虽然Android中没有简洁明了的Api，获取前后台状态和回调，但是好在我们可以自己动手丰衣足食，满足需求。