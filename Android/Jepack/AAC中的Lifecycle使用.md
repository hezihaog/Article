#### AAC中的Lifecycle使用

AAC是Android Architecture Components的简称，是Google在谷歌IO大会上推出的。谈到Activity、Fragment的生命周期，大家肯定十分熟悉了。当然也会写过这样的代码，当一个组件需要在生命周期时做出相应的改变时，写一套生命周期的方法，给Activity、Fragment调用。一个还好，当有多少时，就会十分的臃肿。

例如我们在封装等待弹窗时，onCreate()时创建Dialog，onDestroy时销毁Dialog。

```
public class WaitLoadingController {
	public WaitLoadingController(@NonNull FragmentActivity activity) {
        mActivity = activity;
        mMainHandler = new Handler(Looper.getMainLooper());
    }

	 /**
     * 确保主线程执行
     */
    public void ensureMainThreadRun(Runnable runnable) {
        if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
            runnable.run();
        } else {
            mMainHandler.post(runnable);
        }
    }

	 public void onCreate() {
        ensureMainThreadRun(() -> {
            vLoadingDialog = new LoadingDialog(getActivity());
            vLoadingDialog.setCanceledOnTouchOutside(true);
            vLoadingDialog.setCancelable(true);
        });
    }

    public void onDestroy() {
        ensureMainThreadRun(() -> {
            hideWait();
            vLoadingDialog = null;
            mActivity = null;
        });
    }
    
    //...省略其他方法
}

public class HomeActivity extends AppCompatActivity {
	 private WaitLoadingController mWaitLoadingController;

    public void onCreate() {
        mWaitLoadingController = new WaitLoadingController(this);
        onDestroy. onCreate();
    }

    public void onDestroy() {
        super.onDestroy();
        mWaitLoadingController. onDestroy();
    }
}
```

#### 使用Lifecycle改造

先介绍一下接下来要接触的类和接口

- LifecycleOwner接口(Lifecycle持有者，Activity、Fragment已实现该接口)

- LifecycleObserver接口(Lifecycle的观察者，实现该接口，就能有监听Lifecycle的权利，所以一般是我们自定义的组件实现该接口)

- Lifecycle(生命周期对象，LifecycleOwner中持有Lifecycle对象，LifecycleOwner. getLifecycle()方法获得Lifecycle对象)
- Event，生命周期事件，@OnLifecycleEvent注解标识时传入，就能收到对应的生命周期事件回调。

```
public enum Event {
/**
 * Constant for onCreate event of the {@link LifecycleOwner}.
 */
ON_CREATE,
/**
 * Constant for onStart event of the {@link LifecycleOwner}.
 */
ON_START,
/**
 * Constant for onResume event of the {@link LifecycleOwner}.
 */
ON_RESUME,
/**
 * Constant for onPause event of the {@link LifecycleOwner}.
 */
ON_PAUSE,
/**
 * Constant for onStop event of the {@link LifecycleOwner}.
 */
ON_STOP,
/**
 * Constant for onDestroy event of the {@link LifecycleOwner}.
 */
ON_DESTROY,
/**
 * An {@link Event Event} constant that can be used to match all events.
 */
ON_ANY
}
```

#### 步骤

1. 自定义组件实现LifecycleObserver接口。
2. 获取LifecycleOwner，调用LifecycleOwner的getLifecycle()获取Lifecycle，调用addObserver()，传入自身，即可开始监听。
3. 提供内部的生命周期方法，使用@OnLifecycleEvent注解，注解中传入Event属性，接收相应的生命周期事件回调。

```
public class WaitLoadingController implements LifecycleObserver {
	public WaitLoadingController(@NonNull FragmentActivity activity, LifecycleOwner lifecycleOwner) {
        mActivity = activity;
        mMainHandler = new Handler(Looper.getMainLooper());
        lifecycleOwner.getLifecycle().addObserver(this);
    }
    
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    protected void onLifecycleCreate() {
    	 ensureMainThreadRun(() -> {
            vLoadingDialog = new LoadingDialog(getActivity());
            vLoadingDialog.setCanceledOnTouchOutside(true);
            vLoadingDialog.setCancelable(true);
        });
    }
    
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    protected void onLifecycleDestroy() {
        ensureMainThreadRun(() -> {
            hideWait();
            vLoadingDialog = null;
            mActivity = null;
        });
    }
    
    //...省略其他方法
}
```

#### 巧用Lifecycle，实现广播注册、注销自动化

说到广播，大家也不陌生，我们这里说的是动态广播，动态广播需要在生命周期onCreate()时注册，onDestory()时注销。然后一般广播接收器作为匿名的成员。

- 例如登录用户的资料更新，首页、前置页面都需要使用来显示UI，当最底层的Activity更新资料后，前置页面都需要更新，这时候我们一般使用广播通知来更新。

- 一般我们页面也不止一种广播事件监听，有时候会出现3，4个，那么onCreate()和onDestroy()就会非常多个注册和注销，BroadcastReceiver同样会成对增加。略显臃肿。

- 如果将注册、注销的操作自动化，广播回调使用接口回调处理。

```
//广播通用封装
public class AppBroadcastManager {
    private AppBroadcastManager() {
    }

    public static void register(BroadcastReceiver receiver, String... actions) {
        IntentFilter intentFilter = new IntentFilter();
        for (String action : actions) {
            intentFilter.addAction(action);
        }
        register(receiver, intentFilter);
    }

    public static void register(BroadcastReceiver receiver, IntentFilter filter) {
        try {
            ContextProvider.get().getContext()
                    .registerReceiver(receiver, filter);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void unregister(BroadcastReceiver receiver) {
        if (receiver == null) {
            return;
        }
        try {
            ContextProvider.get().getContext()
                    .unregisterReceiver(receiver);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void sendBroadcast(String action) {
        sendBroadcast(new Intent(action));
    }

    public static void sendBroadcast(String action, Intent intent) {
        intent.setAction(action);
        sendBroadcast(intent);
    }

    public static void sendBroadcast(Intent intent) {
        ContextProvider.get().getContext()
                .sendBroadcast(intent);
    }
}
```

```
private BroadcastReceiver mReceiver = new BroadcastReceiver() {
	@Override
    public void onReceive(Context context, Intent intent) {
        User newUserInfo = (User) intent.getExtras().getSerializable(LingCommonConstant.Extra.USER_INFO);
                    if (newUserInfo != null) {
                    	//更新UserInfo
                        onUpdateUserInfo(newUserInfo);
                    }
    }
}

@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AppBroadcastManager.register(mReceiver, LingCommonConstant.Action.UPDATE_USER_INFO);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        AppBroadcastManager.unregister(mReceiver);
    }
```

#### 使用Lifecycle改造

1. BroadcastRegistry广播注册表。提供一个register方法注册，为每个广播接收器配备一个RegistryTableItem对象，他是LifecycleObserver的实现，在收到生命周期事件时，注册、注销广播接收器。

```
//广播注册表，提供LifecycleOwner和BroadcastReceiver即可自动注册广播接收器
public class BroadcastRegistry {
    private LifecycleOwner mLifecycleOwner;

    public BroadcastRegistry(LifecycleOwner lifecycleOwner) {
        mLifecycleOwner = lifecycleOwner;
    }

    public BroadcastRegistry register(BroadcastReceiver receiver, String... actions) {
        RegistryTableItem tableItem = new RegistryTableItem(receiver, actions);
        mLifecycleOwner.getLifecycle().addObserver(tableItem);
        return this;
    }

    /**
     * 广播注册表项，每一项负责一个广播的注册和注销
     */
    private static class RegistryTableItem implements LifecycleObserver {
        /**
         * 广播接收器
         */
        private BroadcastReceiver mReceiver;
        /**
         * 广播接收器监听的多个Action
         */
        private String[] mActions;

        RegistryTableItem(BroadcastReceiver receiver, String[] actions) {
            mReceiver = receiver;
            mActions = actions;
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
        protected void onLifecycleCreate() {
            AppBroadcastManager.register(mReceiver, mActions);
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
        protected void onLifecycleDestroy() {
            AppBroadcastManager.unregister(mReceiver);
        }
    }
}
```

2. UserInfoTower，具体业务广播的包装类，提供广播接收回调接口。 

```
public class UserInfoTower {
    private OnUpdateUserInfoCallback mCallback;

    public interface OnUpdateUserInfoCallback {
        /**
         * 更新时回调
         */
        void onUpdate(User user);
    }

    private UserInfoTower(LifecycleOwner lifecycleOwner, OnUpdateUserInfoCallback callback) {
        this.mCallback = callback;
        BroadcastRegistry registry = new BroadcastRegistry(lifecycleOwner);
        registry.register(new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                if (mCallback != null) {
                    User newUserInfo = (User) intent.getExtras().getSerializable(LingCommonConstant.Extra.USER_INFO);
                    if (newUserInfo != null) {
                        mCallback.onUpdate(newUserInfo);
                    }
                }
            }
        }, LingCommonConstant.Action.UPDATE_USER_INFO);
    }

    public static UserInfoTower create(LifecycleOwner lifecycleOwner, OnUpdateUserInfoCallback callback) {
        return new UserInfoTower(lifecycleOwner, callback);
    }

    public static void sendUpdateUserInfo(User user) {
        Intent args = new Intent();
        args.putExtra(LingCommonConstant.Extra.USER_INFO, user);
        AppBroadcastManager.sendBroadcast(LingCommonConstant.Action.UPDATE_USER_INFO, args);
    }
}
```

3. 更新资料后，发送广播

```
/**
 * 更新用户信息
 */
public Observable<HttpModel<Boolean>> updateUserInformation(
        Context context, String tag, String avatarUrl, String nickname, BirthdayInfoModel birthday,
        Integer sex, Integer workingStatus, Integer maritalStatus) {
    return LoginHttpManager.updateUserInformation(context, tag, avatarUrl, nickname, birthday, sex
            , workingStatus, maritalStatus)
            .doOnNext(new Consumer<HttpModel<Boolean>>() {
                @Override
                public void accept(HttpModel<Boolean> httpModel) throws Exception {
                    if (httpModel != null && httpModel.getData()) {
                        //...省略一些判断逻辑
                        UserInfoTower.sendUpdateUserInfo(mUser);
                    }
                }
            });
    }
```

4. 在Activity中，注册，实现更新逻辑。

```
UserInfoTower.create(this, new UserInfoTower.OnUpdateUserInfoCallback() {
    @Override
    public void onUpdate(User user) {
        int notFound = -1;
        int position = notFound;
        for (int i = 0; i < mAllItems.size(); i++) {
            Object itemModel = mAllItems.get(i);
            if (itemModel instanceof User) {
                position = i;
            }
        }
        //移除用户信息的Item，重新加入
        if (position != notFound) {
            mAllItems.remove(position);
            mRAdapter.notifyItemRemoved(position);
            mAllItems.add(position, user);
            mRAdapter.notifyItemInserted(position);
        }
    }
});
```

#### 总结

使用AAC中的Lifecycle，能让我们的生命周期绑定更加简单，组件无需提供public的生命周期方法，代码更加洁净。让我们的自定义组件更加注重内部逻辑。