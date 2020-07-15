#### WMRouter 使用篇

之前项目组件化，用的路由库都是ARouter，美团开源的路由库[WMRouter](https://github.com/meituan/WMRouter)，最近写了一个小demo来使用一下。

下面是官方定义

> WMRouter是一款Android路由框架，基于组件化的设计思路，有功能灵活、使用简单的特点。

#### WMRouter主要功能

- URI分发
- ServiceLoader 服务提供

URI分发功能可用于多工程之间的页面跳转、动态下发URI链接的跳转等场景，特点如下：

1. 支持多scheme、host、path
2. 支持URI正则匹配
3. 页面配置支持Java代码动态注册，或注解配置自动注册
4. 支持配置全局和局部拦截器，可在跳转前执行同步/异步操作，例如定位、登录等
5. 支持单次跳转特殊操作：Intent设置Extra/Flags、设置跳转动画、自定义StartActivity操作等
6. 支持页面Exported控制，特定页面不允许外部跳转
7. 支持配置全局和局部降级策略
8. 支持配置单次和全局跳转监听
9. 完全组件化设计，核心组件均可扩展、按需组合，实现灵活强大的功能

基于SPI (Service Provider Interfaces) 的设计思想，WMRouter提供了ServiceLoader模块，类似Java中的java.util.ServiceLoader，但功能更加完善。通过ServiceLoader可以在一个App的多个模块之间通过接口调用代码，实现模块解耦，便于实现组件化、模块间通信，以及和依赖注入类似的功能等。其特点如下：

1. 使用注解自动配置
2. 支持获取接口的所有实现，或根据Key获取特定实现
3. 支持获取Class或获取实例
4. 支持无参构造、Context构造，或自定义Factory、Provider构造
5. 支持单例管理
6. 支持方法调用

#### 添加依赖和Gradle插件

- 项目根build.gralde中，定义版本和Gradle插件地址

```
buildscript {
    ext.kotlin_version = '1.3.61'
    //1. 定义WMRouter版本
    ext.wmrouter_version = '1.2.0'

    repositories {
        google()
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        //2. 添加WMRouter插件
        classpath "com.sankuai.waimai.router:plugin:$wmrouter_version"
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

- WMRouter依赖、注解解释器

kotlin指定注解解释器使用kapt，java模块则使用annotationProcessor。

```
dependencies {
    //WMRouter依赖
    api "com.sankuai.waimai.router:router:$wmrouter_version"
    //注解解释器
    kapt "com.sankuai.waimai.router:compiler:$wmrouter_version"
}
```

- app可运行模块的build.gradle，引入WMRouter的Gradle插件，注意只有可运行模块需要，其他lib模块不能加！否则编译会报错（不像ARouter是需要app可运行模块和lib库模块都需要引入ARouter插件）

```
apply plugin: 'com.android.application'
//引入WMRouter插件，application插件模块才需要引入，其他module不需要
apply plugin: 'WMRouter'
```

#### 模块结构

1. app，壳工程
2. base，公共基础库
3. service，业务模块提供的服务提供接口
4. home，主页模块
5. location，定位模块
6. login，登录模块
7. mine，个人中心模块
8. setting，设置模块
9. shop，商品模块

#### 路由初始化

尽量早，一般在Application中初始化。通过设置根处理器，我们可以处理跳转失败、找不到目标路由时的情况，例如降级处理，跳转到降级页面。

```
class App : Application() {
    companion object {
        private lateinit var mInstance: App

        fun getContext(): Context {
            return mInstance.applicationContext
        }
    }

    override fun onCreate() {
        super.onCreate()
        mInstance = this
        initRouter()
    }

    /**
     * 初始化路由
     */
    private fun initRouter() {
        val enableLog = true
        //设置log
        val logger = DefaultLogger()
        Debugger.setLogger(logger)
        //log打印开关
        Debugger.setEnableLog(enableLog)
        //调试开关
        Debugger.setEnableDebug(enableLog)
        //根处理器
        val rootUriHandler = DefaultRootUriHandler(this)
        //设置全局跳转监听，可用于跳转失败时，Toast和埋点
        rootUriHandler.globalOnCompleteListener = object : OnCompleteListener {
            override fun onSuccess(request: UriRequest) {
                //跳转成功
            }

            override fun onError(request: UriRequest, resultCode: Int) {
                var text = request.getStringField(UriRequest.FIELD_ERROR_MSG, null) ?: ""
                if (text.isBlank()) {
                    text = when (resultCode) {
                        UriResult.CODE_NOT_FOUND -> "不支持的跳转链接"
                        UriResult.CODE_FORBIDDEN -> "没有权限"
                        else -> "跳转失败"
                    }
                }
                if (Debugger.isEnableDebug()) {
                    text += "($resultCode)"
                    text += "\n" + request.uri.toString()
                }
                ToastUtils.showLong(text)
            }
        }
        //开始初始化
        Router.init(rootUriHandler)
    }
}
```

#### Url跳转

路由库的一个特点，就是提供一个Url，即可跳转到目标的Activity页面。下面以登录页面跳转到主页的过程来演示。

- 定义Url，我的格式为，/模块名/页面名称

```
class RouterUrls private constructor() {
	companion object {
	     /**
         * 首页
         */
        const val HOME_HOME = "/home/home"
	}
}
```

- Activity使用@RouterUri注解标识，指定path属性为Url，path属性是一个数组，可以指定多个Url。

```
@RouterUri(path = [RouterUrls.HOME_HOME])
class HomeActivity : BaseActionBarActivity() {
}
```

- 使用Url，开始跳转

```
class LoginActivity : BaseActionBarActivity() {
	vLogin.setOnClickListener {
		//...省略验证代码
		//登录成功，跳转到主页
		Router.startUri(DefaultUriRequest(context, RouterUrls.HOME_HOME)
	}
}
```

- 传递参数，使用putExtra()来添加需要传递的数据。

```
Router.startUri(DefaultUriRequest(context, RouterUrls.HOME_HOME).apply {
    putExtra("username", username)
    putExtra("password", password)
    onComplete(object : OnCompleteListener {
        override fun onSuccess(request: UriRequest) {
            onSuccessBlock?.invoke()
        }

        override fun onError(request: UriRequest, resultCode: Int) {
            onFailBlock?.invoke()
        }
    })
})
```

- 获取路由跳转的跳转结果（成功、失败），通过onComplete()来设置回调。

```
Router.startUri(DefaultUriRequest(context, RouterUrls.HOME_HOME).apply {
    onComplete(object : OnCompleteListener {
        override fun onSuccess(request: UriRequest) {
            onSuccessBlock?.invoke()
        }

        override fun onError(request: UriRequest, resultCode: Int) {
            onFailBlock?.invoke()
        }
    })
})
```

#### 服务提供

服务提供，一般用于不同模块间提供API接口，其他模块只依赖到接口，实现只会放在具体的模块，获取服务时，WMRouter会返回具体的实现。一般模块接口会放在一个单独的module，让其他业务模块依赖，例如service模块，里面就存放模块的API接口。

下面以登录模块的API服务为例子：

- 定义模块接口，LoginService，命名一般为模块 + Service。

```
interface LoginService {
    /**
     * 是否已登录
     */
    fun isLogin(): Boolean

    /**
     * 获取用户信息
     */
    fun getUserInfo(): User

    /**
     * 跳转到登录
     * @param onSuccessBlock 跳转成功回调
     * @param onFailBlock 跳转失败回调
     */
    fun goLogin(
        context: Context,
        onSuccessBlock: (() -> Unit)? = null,
        onFailBlock: (() -> Unit)? = null
    )
}
```

- 定义服务唯一标识Url，我习惯模块的服务为：模块名 + s

```
class RouterUrls private constructor() {
    companion object {
    }
    
    //--------------------------- 登录 ---------------------------

    /**
     * 登录模块服务
     */
    const val LOGIN_SERVICE = "/logins"
}
```

- 定义服务实现，命名一般为服务接口 + impl，服务实现需要加@RouterService注解，有3个属性可用

	1. interfaces属性，实现的接口Class，获取服务时使用。
	2. key，就是上面RouterUrls中定义的Url，因为服务接口可以有多个实例，key就代表这个实例的唯一标识，获取服务时，就可以指定获取具体哪个实例。
	3. singleton，是否单例，尽量服务提供为单例保证唯一，默认为false。

```
@RouterService(
    interfaces = [LoginService::class],
    key = [RouterUrls.LOGIN_SERVICE],
    singleton = true
)
class LoginServiceImpl : LoginService {
    override fun isLogin(): Boolean {
        return LoginStorage.isLogin()
    }

    override fun getUserInfo(): User {
        val userInfo = LoginStorage.getUserInfo()
        return User(
            userInfo.first,
            userInfo.second
        )
    }

    override fun goLogin(
        context: Context,
        onSuccessBlock: (() -> Unit)?,
        onFailBlock: (() -> Unit)?
    ) {
        Router.startUri(DefaultUriRequest(context, RouterUrls.LOGIN_LOGIN).apply {
            onComplete(object : OnCompleteListener {
                override fun onSuccess(request: UriRequest) {
                    onSuccessBlock?.invoke()
                }

                override fun onError(request: UriRequest, resultCode: Int) {
                    onFailBlock?.invoke()
                }
            })
        })
    }
}
```

- 获取服务实例，使用接口class和url来获取。

```
val LoginService = Router.getService(LoginService::class.java, RouterUrls.LOGIN_SERVICE)
```

- 我们一般为了统一管理，会定义一个单独的类，来获取不同模块的方法。我定义的类为ModuleServiceManager。

```
class ModuleServiceManager private constructor() {
    companion object {
        /**
         * 获取登录模块服务
         */
        fun getLoginService(): LoginService {
            return Router.getService(LoginService::class.java, RouterUrls.LOGIN_SERVICE)
        }
    }
}
```

#### 拦截器Interceptor

跳转拦截器，WMRouter提供了接口UriInterceptor，我们自定义的拦截器，需要实现UriInterceptor接口，复写intercept()方法，进行拦截和放行处理。

例如经典拦截器案例，需要进行登录的页面，跳转前需要登录，未登录，跳转去登录页，登录了，则继续跳转。

intercept()方法回调时，会传入一个UriCallback，调用UriCallback的onNext()则代表放行，onComplete()则代表拦截。

- 定义登录拦截器：LoginInterceptor

LoginInterceptor的intercept()中，获取登录服务，调用isLogin()判断是否登录了，登录了调用callback.onNext()放行，未登录则拦截，随后注册一个登录回调，用户登录、成功、失败或取消，都回调回来，再进行调用callback.onNext()或callback.onComplete()。

```
class LoginInterceptor : UriInterceptor {
    override fun intercept(request: UriRequest, callback: UriCallback) {
        //获取登录服务
        val loginService = ModuleServiceManager.getLoginService()
        //登录了，放行
        if (loginService.isLogin()) {
            callback.onNext()
        } else {
            //------- 注意：需要注册一个回调，当登录成功、失败时回调，来决定是否拦截还是放行 -------
            ...
            //跳转去登录，登录成功、失败会回调上面的
            loginService.goLogin(request.context)
        }
    }
}
```

- 定义登录成功、失败或取消的回调

这个回调定义，最好在一个随处都可以获取，进行注册和解注册的地方，我选用在LoginService中定义，因为LoginService是单例，并且可以随处都可以调用ModuleServiceManager.getLoginService()来获取到实例。

```
interface LoginService {
    /**
     * 注册登录结果回调
     * @param observer 登录结果回调
     */
    fun registerLoginResultObserver(observer: OnLoginResultObserver)

    /**
     * 取消注册登录登录结果回调
     * @param observer 登录结果回调
     */
    fun unregisterLoginResultObserver(observer: OnLoginResultObserver)

    interface OnLoginResultObserver {
        /**
         * 登录成功回调
         */
        fun onLoginSuccess()

        /**
         * 登录取消回调
         */
        fun onLoginCancel()

        /**
         * 登录失败回调
         */
        fun onLoginFailure()
    }

    /**
     * 回调登录成功
     */
    fun notifyLoginSuccess()

    /**
     * 回调登录取消
     */
    fun notifyLoginCancel()

    /**
     * 回调登录失败
     */
    fun notifyLoginFailure()
}
```

- 拦截错误码，当我们调用callback.onComplete()终止分发流程时，会需要我们传一个int值的resultCode。我们需要自己定义code，一般自定义code为负数。

```
interface AppRouterUriResult {
    companion object {
        /**
         * 登录取消
         */
        const val CODE_LOGIN_CANCEL = -100

        /**
         * 登录失败
         */
        const val CODE_LOGIN_FAILURE = -101
    }
}
```

- 完善拦截器，我们在跳转去登录前，进行注册。

```
class LoginInterceptor : UriInterceptor {
    override fun intercept(request: UriRequest, callback: UriCallback) {
        val loginService = ModuleServiceManager.getLoginService()
        //登录了，放行
        if (loginService.isLogin()) {
            callback.onNext()
        } else {
            //------ 补充这里：注册登录成功、失败、取消的回调 ------
            loginService.run {
                registerLoginResultObserver(object : LoginService.OnLoginResultObserver {
                    override fun onLoginSuccess() {
                        //登录成功，放行，继续跳转
                        unregisterLoginResultObserver(this)
                        callback.onNext()
                    }

                    override fun onLoginCancel() {
                        //登录取消，拦截
                        unregisterLoginResultObserver(this)
                        callback.onComplete(AppRouterUriResult.CODE_LOGIN_CANCEL)
                    }

                    override fun onLoginFailure() {
                        //登录失败，拦截
                        unregisterLoginResultObserver(this)
                        callback.onComplete(AppRouterUriResult.CODE_LOGIN_FAILURE)
                    }
                })
            }
            //------ 补充这里：注册登录成功、失败、取消的回调 ------
            //跳转去登录
            loginService.goLogin(request.context)
        }
    }
}
```

- 登录页面，补充登录成功、失败、取消的回调调用

```
@RouterUri(path = [RouterUrls.LOGIN_LOGIN])
class LoginActivity : BaseActionBarActivity() {
    //...省略其他代码

    private fun bindView() {
        vLogin.setOnClickListener {
            val username = vUsername.text.toString().trim()
            if (username.isBlank()) {
                return@setOnClickListener
            }
            val password = vPassword.text.toString().trim()
            if (password.isBlank()) {
                return@setOnClickListener
            }
            //------------ 补充这里 ------------
            //登录成功
            try {
                LoginStorage.saveUserInfo(username, password)
                ModuleServiceManager.getLoginService().notifyLoginSuccess()
                //登录成功，跳转到主页
                ModuleServiceManager.getHomeService().goHome(this)
                finish()
            } catch (e: Exception) {
                e.printStackTrace()
                //出现异常，登录失败
                ModuleServiceManager.getLoginService().notifyLoginFailure()
            }
            //------------ 补充这里 ------------
        }
    }

    override fun onBackPressed() {
        super.onBackPressed()
        //------------ 补充这里 ------------
        //登录页面，按返回键，通知登录取消
        ModuleServiceManager.getLoginService().notifyLoginCancel()
        //------------ 补充这里 ------------
    }
}
```

#### 降级策略

当我们将一个不存在的url传给WMRouter时，会回调rootUriHandler根处理器的globalOnCompleteListener，我们可以在这里进行降级处理。

一般rootUriHandler，我们在Application初始化时，就已经指定了。当找不到Url时，会回调onError，此时我们判断resultCode进行处理即可。（这里我只是简单Toast一下，真实开发再进行页面跳转处理即可）

- WMRouter中已预定义的Code

```
public interface UriResult {

    /**
     * 跳转成功
     */
    int CODE_SUCCESS = 200;
    /**
     * 重定向到其他URI，会再次跳转
     */
    int CODE_REDIRECT = 301;
    /**
     * 请求错误，通常是Context或URI为空
     */
    int CODE_BAD_REQUEST = 400;
    /**
     * 权限问题，通常是外部跳转时Activity的exported=false
     */
    int CODE_FORBIDDEN = 403;
    /**
     * 找不到目标(Activity或UriHandler)
     */
    int CODE_NOT_FOUND = 404;
    /**
     * 发生其他错误
     */
    int CODE_ERROR = 500;
}
```

- 示例

```
/**
 * 初始化路由
 */
private fun initRouter() {
    ...
    //根处理器
    val rootUriHandler = DefaultRootUriHandler(this)
    //设置全局跳转监听，可用于跳转失败时，Toast和埋点
    rootUriHandler.globalOnCompleteListener = object : OnCompleteListener {
        override fun onSuccess(request: UriRequest) {
            //跳转成功
        }

        override fun onError(request: UriRequest, resultCode: Int) {
            var text = request.getStringField(UriRequest.FIELD_ERROR_MSG, null) ?: ""
            if (text.isBlank()) {
                text = when (resultCode) {
                    UriResult.CODE_NOT_FOUND -> "不支持的跳转链接"
                    UriResult.CODE_FORBIDDEN -> "没有权限"
                    else -> "跳转失败"
                }
            }
            if (Debugger.isEnableDebug()) {
                text += "($resultCode)"
                text += "\n" + request.uri.toString()
            }
            ToastUtils.showLong(text)
        }
    }
    //开始初始化
    Router.init(rootUriHandler)
}

//跳转无效Url
Router.startUri(getContext(), "/not_found")
```

#### UriHandler

经过上面的路由跳转、服务提供，应对基本使用，基本没问题了，但WMRouter的功能不止这些。下面介绍一下，WMRouter的特定之一UriHandler。