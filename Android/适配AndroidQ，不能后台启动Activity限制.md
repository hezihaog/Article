#### 适配AndroidQ，不能后台启动Activity限制

在AndroidQ或例如Vivo、小米等第三方厂商ROM中，都对后台启动Activity做了限制，AndroidQ中并没有设计有权限申请来进行设置，而Vivo、小米则是在App权限设置中加入了后台启动Activity的权限。

![Vivo拦截应用后台启动Activity通知.png](https://upload-images.jianshu.io/upload_images/1641428-13748141990b6518.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

![Vivo后台弹出界面权限开关.png](https://upload-images.jianshu.io/upload_images/1641428-5b9b9d356e1c3f25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

![MIUI10后台弹出界面权限开关.png](https://upload-images.jianshu.io/upload_images/1641428-d6014ab986d91229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

而默认该项权限都是关闭的，并且因为没有Api可以调用申请。当App第一次在后台情况下跳转Activity时，系统会进行拦截，并弹出一条通知告诉用户，后续则不会重复提醒，而原生AndroidQ则是不提醒，直接拦截。

#### 为什么要后台时跳转，场景呢

- 场景一：启动页，展示品牌Logo和广告，5秒后跳转

有些小伙伴会问了，什么情况我们会后台进行跳转Activity呢？用户都不在操作界面，怎么会有跳转操作呢？其实有一种情况就足以解释了。

例如我们App一般都会设计一个SplashActivity或者一个WelcomeActivity来作为启动页，显示品牌Logo或者广告（一般都有），显示页面5秒后跳转到主页，只要设置用户点击了Home键回到桌面或者点击了其他App的通知(例如微信、QQ等)，就会跳转到通知来源的App，这是App还在计时，当过了5秒后，就会执行跳转操作，这时例如突然收到老板紧急微信，马上点击通知，去到微信中打字，就会突然被这个跳转的App拉回去，造成很不好的体验。（如果是我肯定很烦躁！）

相比iOS，跳转ViewController则不会拉回App。重新点击App时是已经跳转好的了。

- 场景二：电脑版登录，弹出界面确定登录

这种情况在多端登录时，可能会遇到，例如：在电脑端微信，请求登录微信，这时手机微信就会弹出一个Activity让我们点击确认，电脑端再进行登录。这时刚好撞到枪口上了，如果在AndroidQ中，界面就跳转失败了。

#### 找出厂商后台打开Activity的权限开关（不推荐）

和以前权限被关闭，让用户去设置中打开权限类似，例如用获取当前Activity应用，找出厂商的打开后台Activity的权限页面，其实这种方式工作量是最大的，要适配各大厂商的ROM，而且ROM又有不同的版本（例如MIUI9、MIUI10等），而且权限申请还有个理由请求获取，这个后台打开Activity的权限理由，也不好编，让用户觉得你要获取后台弹出的权限，十有八九都觉得你会干坏事，也自然不会允许了。

#### 解决方案，怎么适配

- 启动页延时5秒后跳转的场景

	一般Android端中进行定时、延时功能，一种是使用Timer定时器，一种是使用Handler循环调用来实现。
		
	1. 对于Timer，我们可以在启动页的onPause()中，将Timer停止，计算剩余时间，并在下一次的onResume()中重新启动一个Timer继续计时。
	2. 对于Handler，和上面Timer的处理方式一样，onPause()中removeCallbacksAndMessages(null)来去掉跳转的Message()或Runnable，再在下一次的onResume()中继续sendMessage()或postDelayed()继续计时。
	3. 之前有小伙伴提出过一种粗暴的手法，在跳转前，开启一个定时器，预估个2秒左右，判断当前Activity是否是本次跳转的Activity来判定跳转是否被拦截。对于该方案，我不是很推荐，因为预估的时间怎么评定比较不靠谱，不是每个页面跳转都是2秒，例如像淘宝、支付宝这种量级比较大的App，他们的跳转未必有那么快，尤其是冷启动的时候，往往会比较卡、比较久，低配置的机子会更加明显。

- 微信电脑端登录的场景

	其实对于微信的自动跳转，QQ则是更加优雅的方式，QQ的方案是发出一个通知，用户点击通知，回到QQ进行确认授权登录，而在跳转确定页面中，一种是在通知中，我们可以给PendingIntent设置跳转去Activity来进行跳转，而QQ则应该是考虑到了以下场景：
	
	1. 如果用户手滑，划走了通知，跳转确定需要再次在电脑端申请，再弹一次。
	2. 用户没有点击通知，而是直接点开QQ，却没有确定信息可以直接确认，因为需要点击通知才能跳转。

	那么QQ的做法是什么呢？我预估就是发起通知的同时，再栈顶的Activity跳转一个Fragment，或者使用一个View、Layout进行覆盖来显示。而通知的操作则是将应用的Activity栈拉回前台。
	
#### 官方适配方案

Google推荐方式是，类似QQ的友好提示，在应用在后台时，应该发起一个通知，用户点击后跳转。

1. 如果需要考虑到上面QQ考虑到的情况，发送通知，让用户点击通知将应用拉回前台，栈顶Activity跳转一个Fragment。

但是上面这种方式也有缺点：

1. 跳转Framgnet必须依赖Activity，如果应用被杀死了，所有Activity都回销毁回收，用户再点击通知时，拉起QQ后，再在最前的Activity进行跳转Fragment。对于一般流程，我们是先启动一个启动页再跳转到首页，很有可能出现在启动页中跳转了Fragment，然后在5秒后跳转到首页，恰巧跳转过去同时，启动页被销毁，这个Fragment顺带被销毁了。

2. 一般的路由框架，例如ARouter，Fragment跳转并不支持拦截，例如登录确认的页面，需要跳转前判断是否登录，未登录跳转到登录页面。当然一般不会，除非出现了Bug，就会可能在未登录时收到了确认登录的通知。例如某些极端情况，在电脑端进行申请登录，手机端刚好选择退出登录，如果推送消息或长连接稍微因为网络卡了一点，在退出登录之后收到了，就可能跳转去了确认的页面（我司的测试小姐姐、小哥哥们就是这么变态啊~他们真的会这么做的！）。作为严谨的程序员，我们还是需要加上登录判断，如果是旧页面就已经使用路由拦截器来验证，修改为Fragment就不能使用该功能了。

3. 如果是新模块这么写固然可以，但是如果是旧模块，已经是用跳转Activity的方式写了，改为Fragment会有些不现实，例如该界面有startActivity()的操作，耦合到了Activity的onActivityResult()，需要将原本的onActivityResult()挪到依附的Activity，改动太大了，并且依附到哪个Activity都是不确定的（只是栈顶就可以），代码放哪里都不合适，当时你可以将onActivityResult()的调用分发给所有依附的Fragment来解决，需要改动原有代码，总体来说不合适，我们需要通用方案，遵循开闭原则，对修改关闭，对拓展开放。

#### 解决方案

既然跳转Fragment依赖Activity，造成了耦合，我来讲一下我的解决方案吧~

首先跳转Activity会强行拉起我们的App，对用户造成了不好的影响，那么我们可以在跳转前，判断应用处于前台还是后台。

- 前台：直接跳转即可。
- 后台：则等到下一次用户回到App时继续跳转，与此同时，发送一个通知，让用户点击后将App拉回前台（并不是所有的跳转操作都需要发送通知，类似微信登录确认这种强调马上确认的场景才进行发送，不然用户频繁在跳转前将App后台了，就会频繁发出通知，一般来讲用户不会这样做，但是测试小姐姐、小哥哥会酱紫做呀！(大部分是我自己强迫症的原因)）。

对于前后台判断和监控有很多种方式，这里推荐我之前写的一篇文章[Android App前后台监控](https://www.jianshu.com/p/4628d5310dc6)，对于前后台判断、以及前、后台监听，都有现成的Api使用。

#### 实现步骤

1. 统一在跳转前判断前、后台情况。
2. 在前台直接跳转，再后台则注册一个回到前台时的监听回调，回调时再进行跳转。
3. 发送通知，用户点击通知，将App拉回前台，就会触发步骤二时注册的监听，从而继续跳转。

这样子的好处是，一是并没有强制改动为Fragment导致代码大改，而是在后台时暂停跳转，回到前台时继续跳转。

#### 示例代码

- 例如跳转到订单详情，我使用ARouter来跳转，再用kotlin来对ARouter的navigation()跳转方法做了一个拓展方法来统一跳转。

```
/**
 * 跳转到订单详情
 */
fun goWorkOrderDetail(
    activity: Activity,
    workOrderId: String,
    source: RouteSource = RouteSource.NORMAL,
    callback: NavigationCallback? = null
) {
    ARouter.getInstance()
        .build(ARouterUrl.WORK_ORDER_DETAIL)
        //传递订单Id
        .withString(WorkConstant.Args.WORK_ORDER_ID, workOrderId)
        .startNavigation(activity, source = source, callback = callback)
}
```

- RouteSource枚举，代表跳转时的来源，暂时定义了3种来源

	1. NORMAL，普通跳转，正常的Activity应用内跳转，会进行前、后台判断。
	2. NOTIFICATION，点击通知进行的跳转，不会判断前、后台判断。（不然，在后台时发送给用户的通知被点击了，又会因为在后台被拦截掉）
	3. SHORTCUT，8.0的Shortcut快捷方式来进行跳转，也不行前、后台判断。

```
enum class RouteSource(val code: Int) {
    /**
     * 普通跳转
     */
    NORMAL(1),
    /**
     * 通知栏跳转
     */
    NOTIFICATION(2),
    /**
     * 桌面快捷方式
     */
    SHORTCUT(3)
}
```

- startNavigation()拓展方法。

	1. continueNavigation()，正在发起跳转的方法，kotlin的方法支持内嵌方法，如果大家用Java来写，则可以将方法中的代码封装为Runnable即可。navigation()方法中，我统一包裹了一个NavigationCallback回调来包裹用户传入的跳转回调监听callback，目的是为了统一处理拦截的情况。
	2. 我使用AppMonitor.isAppBackground()方法判断当前是否在后台，并且传入的跳转来源RouteSource枚举是否为普通跳转，是则使用AppMonitor.register()方法来注册一个回到前台时的回调。被回调时，再调用continueNavigation()方法来继续完成跳转，记得取消注册，避免下一次切换又被回调。
	3. sendMoveAppForegroundMsg()，发送通知，大家按自己项目封装的来即可，由于我封装得比较多，不是本篇重点，所以只贴出通知被点击时调用的moveAppToForeground()方法。
	3. navNotification，是提供给发送通知时使用的标题和内容。
	4. 走到else分支，则代表是在前台，那么我们直接调用continueNavigation()，正常跳转即可。

```
/**
 * ARouter跳转Activity统一拓展
 * @param requestCode startActivityForResult时使用的requestCode
 * @param callback 跳转回调
 * @param source 跳转来源类型，默认为普通跳转，会进行后台、前台判断，如果为后台则到下一次回到前台时再跳转
 * @param navNotification 当跳转时不在前台时，是否发送一条通知让用户跳转回App，Pair的2个参数分别为title和content
 * 这个主要是为了兼容AndroidQ的各大第三方厂商定制的不让后台时跳转Activity的权限，一般用于像电脑登录微信时弹出界面使用
 */
@JvmOverloads
fun Postcard.startNavigation(
    activity: Activity,
    requestCode: Int = -1,
    callback: NavigationCallback? = null,
    source: RouteSource = RouteSource.NORMAL,
    navNotification: Pair<String, String>? = null
) {
    //这里兼容AndroidQ的限制，App不在后台进行跳转的情况
    AppMonitor.get().run {
        //正在发起跳转的方法
        fun continueNavigation() {
            navigation(activity, requestCode, object : NavigationCallback {
                override fun onFound(postcard: Postcard?) {
                    //路由目标被发现时调用
                    callback?.onFound(postcard)
                }

                override fun onArrival(postcard: Postcard?) {
                    //路由到达时调用
                    callback?.onArrival(postcard)
                }

                override fun onLost(postcard: Postcard?) {
                    //路由被丢失时调用
                    callback?.onLost(postcard)
                }

                override fun onInterrupt(postcard: Postcard?) {
                    //路由被拦截时调用，统一拦截处理，这里我贴一下我写的登录统一拦截处理
                    //未登录，拦截了，由于登录拦截器中，拦截时会给跳转参数中加一个标志位，如果判断到有标志位，就代表被拦截了，则跳转到登录页面
                    postcard?.run {
                        if (extras.getBoolean(ARouterUrl.IS_LOGIN_INTERCEPTOR)) {
                            ARouter.getInstance()
                                .build(ARouterUrl.LOGIN_LOGIN)
                                .navigation(activity)
                        }
                    }
                    callback?.onInterrupt(postcard)
                }
            })
        }
        //不在前台，订阅一个切换回前台的回调，回到前台时，再继续跳转
        if (isAppBackground && RouteSource.NORMAL == source) {
            register(object : AppMonitor.CallbackAdapter() {
                override fun onAppForeground() {
                    super.onAppForeground()
                    //回调一次后就取消注册，否则会重复回调
                    unRegister(this)
                    //回到前台再继续跳转
                    continueNavigation()
                }
            })
            //发送一条通知提醒用户点击，用户点击后再跳转
            if (navNotification != null) {
                sendMoveAppForegroundMsg()
            }
        } else {
            //在前台或者通知栏跳转，直接跳转
            continueNavigation()
        }
    }
}
```

- 通知被点击后，执行moveAppToForeground，将App从后台拉回到前台

```
/**
 * 将栈顶activity移到前台
 */
public void moveAppToForeground(Context context) {
    ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    List<ActivityManager.RunningTaskInfo> tasks = activityManager.getRunningTasks(100);
    for (ActivityManager.RunningTaskInfo task : tasks) {
        if (task.topActivity.getPackageName().equals(context.getPackageName())) {
            activityManager.moveTaskToFront(task.id, ActivityManager.MOVE_TASK_WITH_HOME);
        }
    }
}
```

#### 没有封装统一跳转，无侵入式拦截

如果没有封装类似startNavigation()，统一的跳转方法，而使用了路由框架。例如ARouter，我们可以建立一个拦截器。在拦截器中进行前、后台判断。

1. 在拦截器中判断前、后台情况，后台时，注册前台回调，记得取消注册，避免下一次切换又被回调，发送通知。下次回到前台时继续跳转。
2. 在前台，继续跳转。

```
@Interceptor(priority = 1)
class AppNavgationInterceptor : IInterceptor {
    override fun init(context: Context?) {
    }

    override fun process(postcard: Postcard?, callback: InterceptorCallback?) {
        AppMonitor.get().run {
            //在后台，拦截
            if (isAppBackground) {
                //获取RouteSource
                val routeSource = postcard?.extras?.getSerializable("route_source") as? RouteSource
                //获取要发送通知标题、内容
                val navNotificationPair = postcard?.extras?.getSerializable("nav_notification") as? Pair<String, String>
                if(routeSource == RouteSource.NORMAL) {
                    register(object :AppMonitor.CallbackAdapter() {
                        override fun onAppForeground() {
                            super.onAppForeground()
                            unRegister(this)
                            //回到前台后，继续跳转
                            callback?.onContinue(postcard)
                        }
                    })
                    //发送一条通知提醒用户点击，用户点击后再跳转
                    if (navNotificationPair != null) {
                        sendMoveAppForegroundMsg()
                    }
                } else {
                    //不是普通跳转类型，不进行拦截
                    callback?.onContinue(postcard)
                }
            } else {
                //在前台，放行，继续跳转
                callback?.onContinue(postcard)
            }
        }
    }
}
```

2. startNavigation()方法，则需要修改一下，主要是将跳转来源和通知标题、内容放到跳转参数，让拦截器获取。直接使用navigation()跳转即可。

```
@JvmOverloads
fun Postcard.startNavigation(
    activity: Activity,
    requestCode: Int = -1,
    callback: NavigationCallback? = null,
    source: RouteSource = RouteSource.NORMAL,
    navNotification: Pair<String, String>? = null
) {
	 //修改1：将跳转来源和通知标题、内容放到跳转参数，让拦截器获取
    withSerializable("route_source", source)
    if (navNotification != null) {
        withSerializable("nav_notification", navNotification)
    }
	navigation(activity, requestCode, object : NavigationCallback {
            //...省略其他复写方法
            override fun onInterrupt(postcard: Postcard?) {
            		//省略登录拦截处理
                callback?.onInterrupt(postcard)
            }
        })
}
```	