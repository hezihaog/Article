#### Context中的装饰器模式

对于Android开发者来说，Context再熟悉不过，通过Context，我们可以启动一个Activity，启动一个Service服务，发送一个广播，注册、注销广播接收器，Context就像一个大总管，管理者4大组件。本篇就来聊聊Context中使用到的设计模式-装饰器模式。

#### Context类族

Context是一个抽象类，他的子类有Application、Activity、Service，以及ContextWrapper、ContextThemeWrapper。

- Application继承于ContextWrapper。
- Activity继承于ContextThemeWrapper，ContextThemeWrapper又继承于ContextWrapper。
- Service继承于ContextWrapper。

![Context类族类图.png](https://upload-images.jianshu.io/upload_images/1641428-7fd8cb08c7c17366.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ContextThemeWrapper为ContextWrapper提供了主题相关的能力，而Activity继承于ContextThemeWrapper，所以Activity就有了主题的能力。

#### ContextWrapper是什么

ContextWrapper继承于Context这个抽象类，它的构造方法传入了一个Context参数的变量base，如果传入null，则需要attachBaseContext()方法中传入。

内部复写了大量的Context抽象方法，例如startActivity()、sendBroadcast()、startService()，转调传入的base成员变量。

所以其实真正干活的是传入的Context对象base，ContextWrapper相当于一个装饰器。base对象则为被装饰器对象。

```
public class ContextWrapper extends Context {
    //被装饰的Context对象
    Context mBase;

    public ContextWrapper(Context base) {
        mBase = base;
    }

    protected void attachBaseContext(Context base) {
        if (mBase != null) {
            throw new IllegalStateException("Base context already set");
        }
        mBase = base;
    }
    
    //省略其他代码...
    
    @Override
    public SharedPreferences getSharedPreferences(String name, int mode) {
        return mBase.getSharedPreferences(name, mode);
    }
    
    @Override
    public void startActivity(Intent intent, Bundle options) {
        mBase.startActivity(intent, options);
    }
    
    @Override
    public void sendBroadcast(Intent intent) {
        mBase.sendBroadcast(intent);
    }
    
    @Override
    public Intent registerReceiver(
        BroadcastReceiver receiver, IntentFilter filter) {
        return mBase.registerReceiver(receiver, filter);
    }
    
    @Override
    public ComponentName startService(Intent service) {
        return mBase.startService(service);
    }
    
    @Override
    public boolean stopService(Intent name) {
        return mBase.stopService(name);
    }
    
    //省略其他代码...
}
```

#### 真正干活的ContextImpl

上面说到真正干活的Context通过构造或者attachBaseContext()方法传入。我们用Android Studio查找一个引用，发现Activity、Service、Application都调用了ContextWrapper构造方法，我们以Application为例来看下。

我们看到Application的构造方法调用了super(null)，Application也继承于ContextWrapper，所以这个super(null)调用的是ContextWrapper的带Context构造参数构造方法。

```
public class Application extends ContextWrapper implements ComponentCallbacks2 {
	public Application() {
	    super(null);
	}
}
```

Application构造传入了null，所以传入被装饰的Context实例只能在attachBaseContext(context)的调用处了，搜索一下，Application的attach(context)方法调用了attachBaseContext(context)。

```
final void attach(Context context) {
    attachBaseContext(context);
    mLoadedApk = ContextImpl.getImpl(context).mPackageInfo;
}
```

继续追踪，看下是谁调用了Application的attach(context)。Instrumentation类的newApplication()方法，通过反射Application的clazz，创建Application的实例，再调用了attach()方法传入了Context。Context实例是通过调用newApplication()传入的，来看看是谁调用了newApplication()方法。

```
public class Instrumentation {
    public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        Application app = getFactory(context.getPackageName())
                .instantiateApplication(cl, className);
        app.attach(context);
        return app;
    }
}
```

newApplication()方法，在LoadedApk类makeApplication()方法中调用，可以看到Context实例为ContextImpl，它继承了抽象类Context，所以被装饰的类实例为ContextImpl的实例。

```
public final class LoadedApk {
public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        //如果已经创建了，直接返回
        if (mApplication != null) {
            return mApplication;
        }

        Application app = null;

        String appClass = mApplicationInfo.className;
        if (forceDefaultAppClass || (appClass == null)) {
            appClass = "android.app.Application";
        }
        
        //...省略部分代码

        try {
            java.lang.ClassLoader cl = getClassLoader();
            if (!mPackageName.equals("android")) {
                Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER,
                        "initializeJavaContextClassLoader");
                initializeJavaContextClassLoader();
                Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
            }
            //创建被装饰者Context，实例为ContextImpl
            ContextImpl appContext = ContextImpl.createAppContext(mActivityThread, this);
            //Instrumentation的newApplication()方法在这里被调用
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);
            appContext.setOuterContext(app);
        } catch (Exception e) {
            //...省略部分代码
        }
        mActivityThread.mAllApplications.add(app);
        mApplication = app;
        return app;
    }
}

//ContextImpl继承抽象类Context
class ContextImpl extends Context {
    //创建Context实例，实例为ContextImpl
    static ContextImpl createAppContext(ActivityThread mainThread, LoadedApk packageInfo) {
        if (packageInfo == null) throw new IllegalArgumentException("packageInfo");
        ContextImpl context = new ContextImpl(null, mainThread, packageInfo, null, null, null, 0,
                null);
        context.setResources(packageInfo.getResources());
        return context;
    }
}
```

以上源码流程来看，被装饰的Context实例为ContextImpl，已经得到了我们的结果了，说个题外话，makeApplication()在什么实际被调用呢？

查看调用链，在ActivityThread中的performLaunchActivity方法中，调用了makeApplication()方法创建Application。

ActivityThread类，我们知道它是App进程中唯一的主线程。performLaunchActivity()方法则为启动Activity时调用生成Activity实例的方法。

```
public final class ActivityThread extends ClientTransactionHandler {
	//生成Activity实例
	private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
		//...省略其他代码
		
		Activity activity = null;
		//创建Activity实例
		activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
		//这里调用makeApplication()创建Application实例
		Application app = r.packageInfo.makeApplication(false, mInstrumentation);
		
		//...省略其他代码
		return activity
	}
}
```

#### ContextThemeWrapper-拓展主题的ContextWrapper

ContextThemeWrapper继承于ContextWrapper，带Context参数构造方法和attachBaseContext()则是原封不动的调用回ContextWrapper装饰器中转调ContextImpl的方法。

并提供了2个带Theme对象和themeResId的构造方法。以及提供了setTheme()、getThemeResId()、getTheme()等主题操作、获取的方法。

```
public class ContextThemeWrapper extends ContextWrapper {
    public ContextThemeWrapper() {
        super(null);
    }

    @Override
    protected void attachBaseContext(Context newBase) {
        super.attachBaseContext(newBase);
    }
    
    public ContextThemeWrapper(Context base, @StyleRes int themeResId) {
        super(base);
        mThemeResource = themeResId;
    }
    
    public ContextThemeWrapper(Context base, Resources.Theme theme) {
        super(base);
        mTheme = theme;
    }
    
    //------------- 下面就是拓展的主题相关的方法 -------------
    
    @Override
    public void setTheme(int resid) {
        if (mThemeResource != resid) {
            mThemeResource = resid;
            initializeTheme();
        }
    }

    /** @hide */
    @Override
    public int getThemeResId() {
        return mThemeResource;
    }

    @Override
    public Resources.Theme getTheme() {
        if (mTheme != null) {
            return mTheme;
        }

        mThemeResource = Resources.selectDefaultTheme(mThemeResource,
                getApplicationInfo().targetSdkVersion);
        initializeTheme();

        return mTheme;
    }
    
    //省略其他代码...
}
```

#### 总结

无论Context的继承者有多少，核心Api的实现都在ContextImpl中，诸如Activity、Service等继承ContextWrapper的类，如果增强Context中的Api则复写Context中的抽象方法，如果需要增加特殊中类的方法则新建即可（例如ContextThemeWrapper），这样做的好处是，如果后续需要增加一种更强大的组件，对原有组件可以进行包装拓展，装饰器模式则可以很好的组合新、老组件，只需要新组件继承ContextWrapper，再拓展自己特有的Api即可。

相比继承，装饰器可以多层叠加来组装，可拔插，而无需继承，比继承的具有更多的灵活性。