#### 无侵入式获取全局Context

当我们在使用第三方库，或者自己封装库，如果需要需要用到Context时，一般做法就是将初始化方法暴露给调用方，让调用方在初始化类库时，传入Context。

```
publi class App extends Application {
    /**
     * 是否是Debug环境
     */
    public static final boolean IS_DEBUG = true;

    private static App mContext;

    @Override
    public void onCreate() {
    	super.onCreate();
    	//ARouter初始化
        if (IS_DEBUG) {
            ARouter.openLog();
            ARouter.openDebug();
        }
        ARouter.init(this);
    }
    
    public static Context getInstance() {
        return mContext;
    }
}
```

#### 解决方案

其实无侵入式获取Context的实现很简单，就是使用一个ContentProvider，ContentProvider的onCreate()方法调用时，调用getContext()即可获取到Context，再静态变量保存，后续直接获取即可。

#### Picasso的无侵入式获取Context

上述的原理，其实是从Picasso中借鉴的，一起来看一下吧。

- Picasso实例获取。重点在Double-Check单例中的PicassoProvider.context，调用PicassoProvider的context属性获取Context。

```
static volatile Picasso singleton = null;

public static Picasso get() {
  if (singleton == null) {
    synchronized (Picasso.class) {
      if (singleton == null) {
        //重点在这里，PicassoProvider.context获取Context
        if (PicassoProvider.context == null) {
          throw new IllegalStateException("context == null");
        }
        singleton = new Builder(PicassoProvider.context).build();
      }
    }
  }
  return singleton;
}
```

- PicassoProvider类

PicassoProvider其实是一个ContentProvider的子类，只要将PicassoProvider在AndroidManifest清单文件中注册，启动App时，Android框架会初始化这个PicassoProvider，onCreate()方法被调用时，getContext()保存Context即可。

```
public final class PicassoProvider extends ContentProvider {
  @SuppressLint("StaticFieldLeak") static Context context;

  @Override public boolean onCreate() {
    context = getContext();
    return true;
  }   
}
```

#### 代码时间

- ApplicationContextProvider，和Picasso一样，是ContentProvider的子类。在onCreate()中获取到Context，再保存到ContextProvider实例中。

```
public class ApplicationContextProvider extends ContentProvider {
    @SuppressLint("StaticFieldLeak")
    static Context mContext;

    @Override
    public boolean onCreate() {
        mContext = getContext();
        return false;
    }

    //...省略其他必须复写的方法（空实现即可）
}
```

- 在AndroidManifest.xml中注册ApplicationContextProvider

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.app.contextprovider">

    <application>
        <!-- 全局Context提供者 -->
        <provider
                android:name=".ApplicationContextProvider"
                android:authorities="${applicationId}.contextprovider"
                android:exported="false" />
    </application>
</manifest>
```

- ContextProvider，提供全局Context的单例类，提供get()方法获取单例实例，第一次构造时才从ApplicationContextProvider中获取Context来初始化自身。

```
public class ContextProvider {
    @SuppressLint("StaticFieldLeak")
    private static volatile ContextProvider instance;
    private Context mContext;

    private ContextProvider(Context context) {
        mContext = context;
    }

    /**
     * 获取实例
     */
    public static ContextProvider get() {
        if (instance == null) {
            synchronized (ContextProvider.class) {
                if (instance == null) {
                    Context context = ApplicationContextProvider.mContext;
                    if (context == null) {
                        throw new IllegalStateException("context == null");
                    }
                    instance = new ContextProvider(context);
                }
            }
        }
        return instance;
    }

    /**
     * 获取上下文
     */
    public Context getContext() {
        return mContext;
    }

    public Application getApplication() {
        return (Application) mContext.getApplicationContext();
    }
}
```

- 使用，例如使用全局Context发送广播

```
Intent intent = new Intent(action_update_user_info)
ContextProvider.get().getContext().sendBroadcast(intent);
```

#### leakcanary中的无侵入式初始化

leakcanary是Android非常出名的内存泄露检测库，由square公司出品。以前版本的需要我们手动调用install()初始化，最新版已经不需要了，原因就是也采用ContentProvider自动被Android框架调用的方式，获取到Context后进行install()。

AppWatcherInstaller，也是ContentProvider的子类。在onCreate()时，调用了InternalAppWatcher的install()方法进行安装。

```
//ContentProvider子类
internal class AppWatcherInstaller : ContentProvider() {
    override fun onCreate(): Boolean {
	    SharkLog.logger = DefaultCanaryLog()
	    val application = context!!.applicationContext as Application
	    //无侵入式安装
	    InternalAppWatcher.install(application)
	    return true
  }
  
  //省略其他代码...
}

internal object InternalAppWatcher {
  //省略其他代码...

  //安装
  fun install(application: Application) {
    SharkLog.d { "Installing AppWatcher" }
    checkMainThread()
    //安装过就不安装了，避免多进程重复初始化
    if (this::application.isInitialized) {
      return
    }
    InternalAppWatcher.application = application
	//安装
    val configProvider = { AppWatcher.config }
    ActivityDestroyWatcher.install(application, objectWatcher, configProvider)
    FragmentDestroyWatcher.install(application, objectWatcher, configProvider)
    onAppWatcherInstalled(application)
  }
  
  //省略其他代码...
}
```

#### AutoSize屏幕适配框架自动初始化

AutoSize是一个今日头条屏幕适配方案的第三方库，内部也有一个InitProvider，继承于ContentProvider，在onCreate()中调用AutoSizeConfig类的init()方法进行初始化，减少了配置。

```
public class InitProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        AutoSizeConfig.getInstance()
                .setLog(true)
                .init((Application) getContext().getApplicationContext())
                .setUseDeviceSize(false);
        return true;
    }
    
    //省略其他代码...
}
```

#### 总结

ContentProvider方法无侵入式初始化方案的优缺点：

- 优点：对于固定的初始化配置，可以使用ContextProvider方案减少调用方的配置，减少出错。

- 缺点：如果初始化非常耗时，无疑会拖慢App的启动，如果是耗时初始化，应该提供给调用方自行决定，例如将初始化推迟到主界面onCreate()时才调用初始化。