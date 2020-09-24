#### OkHttp3拦截器通用参数配置

OkHttp3网络请求库相信大家都用起来了，通用请求参数配置每个App都会有，例如登录后，必传Token令牌，UserId用户Id等。

本篇记录一下自己封装的请求时带通用请求参数的配置，也是对Json上传的请求做通用参数配置时遇到的问题和解决方案。

#### 封装思想

常用请求方式分为GET表单请求、POST表单请求、POST上传JSON请求，所以拦截器就对这3种进行适配，如果需要其他的请求方式，再做对应的拦截器即可。

3种请求方式，就要写3个拦截器，3个拦截器，我觉得有些冗余，其实1个就够了。拦截器就是责任链模式的应用，请求调用时，遍历拦截器链，每个拦截器都有机会处理。那么我们可以做一个统一的拦截器作为分发，内部做3个处理器再做责任链遍历处理。原理就是如此，撸起袖子就是干！

#### 使用

使用就一句代码即可！

```
val builder = OkHttpClient.Builder()
//对你的OkHttpClient的Builder进行处理
val builder = builder.apply {
	//配置这句即可
	RequestProcessor.getInstance().with(this)
}
//OkGo配置OkHttpClient，如果你是用其他框架，将builder传框架即可
OkGo.getInstance().init(this)
    .setOkHttpClient(builder.build())
```

#### 类结构

类有5个，1个OkHttp拦截器类，1个请求处理者抽象接口

- RequestProcessor，请求处理器，内含一个OkHttp拦截器，以及3个请求处理者。负责拦截请求，分发请求到注册的处理者。

- RequestProcessHandler，处理者抽象接口，有2个方法，isCanHandle(request)判断请求是否可以处理，process(request)处理方法。

3个不同请求方式的自定义处理者

- GetRequestHandler，GET表单请求方式的处理者，负责拦截GET表单请求，并且添加参数。

- FormPostRequestHandler，POST表单请求方式的处理者，负责拦截POST表单请求，并且添加参数。

- JsonPostRequestHandler，POST上传JSON请求方式的处理者，负责拦截POST上传JSON的请求，并对JSON添加公用参数。

#### 代码实现

GET、POST的表单请求比较简单，OkHttp3都提供了获取方法和对应Builder类。遇到问题的是POST上传JSON添加公共参数的问题，官方并没有Api提供。

- RequestProcessor请求处理器。

	1. 单例，私有化构造方法，提供getInstance()方法获取单例。
	2. 提供with()方法，提供给外部传入OkHttpClient的Builder，我们添加完拦截器后返回
	3. 提供setEnable(enable)设置是否启动，isEnable()判断是否启用
	4. 提供registerProcessHandler()方法，支持外部注册请求处理者
	5. 提供unregisterProcessHandler()方法，支持外部解注册请求处理者
	6. 内部默认添加3种处理者

```
class RequestProcessor private constructor() {
    /**
     * 是否启用，默认启用
     */
    private var isEnable = true
    /**
     * 统一分发的拦截器
     */
    private var mDispatchInterceptor: Interceptor
    /**
     * 处理器链
     */
    private val mProcessHandlerChain by lazy {
        CopyOnWriteArrayList<RequestProcessHandler>()
    }

    companion object {
        private class SingleHolder {
            companion object {
                val INSTANCE: RequestProcessor = RequestProcessor()
            }
        }

        fun getInstance(): RequestProcessor {
            return SingleHolder.INSTANCE
        }
    }

    init {
        //注册多种请求处理器
        registerProcessHandler(GetRequestHandler())
        registerProcessHandler(FormPostRequestHandler())
        registerProcessHandler(JsonPostRequestHandler())
        //初始化分发拦截器
        this.mDispatchInterceptor = Interceptor { chain ->
            if (!isEnable) {
                chain.proceed(chain.request())
            } else {
                chain.proceed(processRequest(chain.request()))
            }
        }
    }

    /**
     * 处理请求
     */
    private fun processRequest(originRequest: Request?): Request {
        var outRequest: Request = originRequest!!
        //责任链分派给不同的请求处理器
        mProcessHandlerChain.forEach { handler ->
            //可以处理，就给对应的处理器处理
            if (handler.isCanHandle(originRequest)) {
                outRequest = handler.process(originRequest)
                return@forEach
            }
        }
        return outRequest
    }

    /**
     * 提供给外部传入OkHttpClient的Builder，我们添加完拦截器后返回
     */
    fun with(builder: OkHttpClient.Builder): OkHttpClient.Builder {
        return builder.addInterceptor(mDispatchInterceptor)
    }

    /**
     * 是否启用
     */
    fun isEnable(): Boolean {
        return this.isEnable
    }

    /**
     * 设置是否启动
     */
    fun setEnable(enable: Boolean) {
        this.isEnable = enable
    }

    /**
     * 注册请求处理器
     */
    fun registerProcessHandler(handler: RequestProcessHandler) {
        if (!mProcessHandlerChain.contains(handler)) {
            mProcessHandlerChain.add(handler)
        }
    }

    /**
     * 解注册请求处理器
     */
    fun unregisterProcessHandler(handler: RequestProcessHandler) {
        mProcessHandlerChain.remove(handler)
    }
}
```

- RequestProcessHandler，请求处理者抽象接口，定义2个抽象方法。

```
interface RequestProcessHandler {
    /**
     * 是否可以处理
     * @return 返回true代表可以处理，返回false代表不可以处理
     */
    fun isCanHandle(originRequest: Request): Boolean

    /**
     * 处理
     * @param originRequest 原始的请求
     * @return 处理过后的请求
     */
    fun process(originRequest: Request): Request
}
```

- GetRequestHandler，GET表单请求者。

	1. isCanHandle()方法，判断请求的方式是否是GET，否则不处理
	2. process()方法，处理本次的GET表单请求，处理Header。Header添加参数，重新创建Headers.Builder，再将原始Header参数添加即可。
	3. 最后将新的header，重新设置。

```
class GetRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        return "GET" == originRequest.method()
    }

    override fun process(originRequest: Request): Request {
        val loginService = getLoginService()
        val token = loginService?.getToken() ?: ""
        val userId = loginService?.getUserId() ?: ""
        //原始Header
        val originHeaders = originRequest.headers()
        //增加公共Header
        val newHeaders = Headers.Builder()
            //先添加原有Header
            .addAll(originHeaders).apply {
                //平台标识
                add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
                //增加公共参数
                if (token.isNotBlank()) {
                    //调用方没有加相同的公共参数时，才添加，避免在切换的业务场景时覆盖
                    val key = AppConstant.HttpParameter.TOKEN
                    if (originHeaders.get(key).isNullOrBlank()) {
                        add(key, token)
                        LogUtils.d("JSON POST => 添加公共参数 -> $key : $token")
                    }
                }
                if (userId.isNotBlank()) {
                    val key = AppConstant.HttpParameter.USER_ID
                    if (originHeaders.get(key).isNullOrBlank()) {
                        add(key, userId)
                        LogUtils.d("JSON POST => 添加公共参数 -> $key : $userId")
                    }
                }
            }
            .build()
        return originRequest.newBuilder()
            //设置Header
            .headers(newHeaders)
            .build()
    }
}
```

- FormPostRequestHandler，处理Post请求是表单提交

	1. isCanHandle()方法中判断请求的body对象是否是FormBody，否则不处理，如果是，则在process()方法中进行处理。
	3. 设置公共参数到Header，最后重新配置Header到请求中。

```
class FormPostRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        val body = originRequest.body()
        if (body is FormBody) {
            return true
        } else if (body is ProgressRequestBody<*>) {
            //这里是处理OkGo对FormBody的包装，如果你不是用OkGo框架，则不需要这句判断
            return try {
                //反射获取包装的RequestBody
                val newBody: RequestBody = Reflect.on(body).field("requestBody").get()
                newBody is FormBody
            } catch (e: Exception) {
                false
            }
        }
        return false
    }

    override fun process(originRequest: Request): Request {
        //公共参数
        val loginService = getLoginService()
        val token = loginService?.getToken() ?: ""
        val userId = loginService?.getUserId() ?: ""
        //原始Header
        val originHeaders = originRequest.headers()
        //增加公共Header
        val newHeaders = Headers.Builder()
            //先添加原有Header
            .addAll(originHeaders).apply {
                //平台标识
                add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
                //增加公共参数
                if (token.isNotBlank()) {
                    val key = AppConstant.HttpParameter.TOKEN
                    if (originHeaders.get(key).isNullOrBlank()) {
                        add(key, token)
                        LogUtils.d("JSON POST => 添加公共参数 -> $key : $token")
                    }
                }
                if (userId.isNotBlank()) {
                    val key = AppConstant.HttpParameter.USER_ID
                    if (originHeaders.get(key).isNullOrBlank()) {
                        add(key, userId)
                        LogUtils.d("JSON POST => 添加公共参数 -> $key : $userId")
                    }
                }
            }
            .build()
        return originRequest.newBuilder()
            //设置Header
            .headers(newHeaders)
            .build()
    }
}
```

- JsonPostRequestHandler，处理Json上传请求

	1. isCanHandle()方法中判断请求的body对象是否是RequestBody，否则不处理，如果是，则在process()方法中进行处理。
	3. 设置公共参数到Header，最后重新配置Header到请求中。

```
class JsonPostRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        return originRequest.body() is RequestBody
    }

    override fun process(originRequest: Request): Request {
        //原始Header
        val originHeaders = originRequest.headers()
        val newHeaders = originRequest.headers().newBuilder().apply {
            //增加公共参数
            val loginService = getLoginService()
            val token = loginService?.getToken() ?: ""
            val userId = loginService?.getUserId() ?: ""
            //平台标识
            add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
            //Token令牌
            if (token.isNotBlank()) {
                val key = AppConstant.HttpParameter.TOKEN
                if (originHeaders.get(key).isNullOrBlank()) {
                    add(key, token)
                    LogUtils.d("JSON POST => 添加公共参数 -> $key : $token")
                }
            }
            //用户Id
            if (userId.isNotBlank()) {
                val key = AppConstant.HttpParameter.USER_ID
                if (originHeaders.get(key).isNullOrBlank()) {
                    add(key, userId)
                    LogUtils.d("JSON POST => 添加公共参数 -> $key : $userId")
                }
            }
        }
            .build()
        return originRequest.newBuilder()
            //设置Header
            .headers(newHeaders)
            .build()
    }
}
```