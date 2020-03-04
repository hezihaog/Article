#### OkHttp3拦截器通用参数配置

OkHttp3网络请求库相信大家都用起来了，通用请求参数配置每个App都会有，例如登录后，必传Token令牌，UserId用户Id等。

本篇记录一下自己封装的请求时带通用请求参数的配置，也是对Json上传的请求做通用参数配置时遇到的问题和解决方案。

我的项目使用的是对OkHttp3进行封装的**OkGo**，使用**Retrofit**也是一样适用的。

#### OkGo原始配置的问题

OkGo其实提供了一个通用请求参数的配置，但是这个配置只支持规定死的数据，例如在Application中初始化，参数中有一个platform的字段配置，例如安卓客户端是23，iOS为24，这种是没问题的。

但是例如上面说到的登录后，才得到的Token和UserId（我的项目是username，想不到后端的User表竟然不设置主键id），不可能在Application初始化时就初始化，毕竟第一次安装的时候都没有值，而OkGo的通用参数配置值一个HttpParams类，是存储到静态变量上的，如果需要登录后生效，每次登录操作成功后，都要重新设置一次，虽然可以实现，总是挺别扭的。

既然需要每次都添加通用参数，为何不在请求时进行拦截，添加上呢？直接用拦截器就可以实现了，所以就抛弃了OkGo提供的配置。

#### 封装思想

常用请求方式分为GET表单请求、POST表单请求、POST上传JSON请求，所以拦截器就对这3种进行适配，如果需要其他的请求方式，再做对应的拦截器即可。

3种请求方式，就要写3个拦截器，3个拦截器，我觉得有些冗余，其实1个就够了。拦截器就是责任链模式的应用，请求调用时，遍历拦截器链，每个拦截器都有机会处理。那么我们可以做一个统一的拦截器作为分发，内部做3个处理器再做责任链遍历处理。原理就是如此，撸起袖子就是干！

#### 使用

使用就一句代码即可！

```
val builder = OkHttpClient.Builder().apply {
	//配置这句即可
	RequestProcessor.getInstance().with(this)
}
//OkGo配置OkHttpClient，不是每个都需要，如果没有使用OkGO不需要加！
OkGo.getInstance().init(this)
    .setOkHttpClient(builder.build())
```

#### 类结构

类有5个，1个OkHttp拦截器类，1个请求处理者抽象接口，3个不同请求方式的自定义处理者。

- RequestProcessor，请求处理器，内含一个OkHttp拦截器，以及3个请求处理者。负责拦截请求，分发请求到注册的处理者。

- RequestProcessHandler，处理者抽象接口，有2个方法，isCanHandle(request)判断请求是否可以处理，process(request)处理方法。

- GetRequestHandler，GET表单请求方式的处理者，负责拦截GET表单请求，并且添加参数。

- FormPostRequestHandler，POST表单请求方式的处理者，负责拦截POST表单请求，并且添加参数。

- JsonPostRequestHandler，POST上传JSON请求方式的处理者，负责拦截POST上传JSON的请求，并对JSON添加公用参数。

#### 代码时间

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

	1. isCanHandle()方法，判断请求的方式是否是GET，是则返回true，拦截掉
	2. process()方法，处理本次的GET表单请求，分别处理Header和查询参数。Header添加参数，重新创建Headers.Builder，再将原始Header参数添加即可。
	3. 添加GET请求的参数，实际就是在Url上拼接上参数。OkHttp3提供了addEncodedQueryParameter()方法添加，key-value方式添加即可。
	3. 最后将新的header和url，重新设置。

```
class GetRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        return "GET" == originRequest.method()
    }

    override fun process(originRequest: Request): Request {
        val loginService = getLoginService()
        val token = loginService?.getToken() ?: ""
        val username = loginService?.getUsername() ?: ""
        val url = originRequest.url()
        //增加公共Header
        val newHeaders = Headers.Builder()
            //先添加原有Header
            .addAll(originRequest.headers())
            .add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
            .build()
        val newUrl = url.newBuilder().apply {
            run {
                //增加公共参数
                if (token.isNotBlank()) {
                    addEncodedQueryParameter(AppConstant.HttpParameter.TOKEN, token)
                    Logger.d("GET => 添加公共参数 -> ${AppConstant.HttpParameter.TOKEN} : $token")
                }
                if (username.isNotBlank()) {
                    addEncodedQueryParameter(AppConstant.HttpParameter.USERNAME, username)
                    Logger.d("GET => 添加公共参数 -> ${AppConstant.HttpParameter.USERNAME} : $username")
                }
            }
        }.build()
        val newBuilder = originRequest.newBuilder()
        return newBuilder
            .url(newUrl)
            .headers(newHeaders)
            .build()
    }
}
```

- FormPostRequestHandler，处理Post请求是表单提交，添加默认参数。

	1. isCanHandle()方法中判断请求的body对象是否是FormBody，否则不处理，process()方法中进行处理。
	1. Header添加方法和GET处理器中的一致，就不再赘述了。
	2. 添加参数，和GET不同，使用的是FormBody，它继承于RequestBody，创建FormBody，必须使用FormBody.Builder类，同样先添加回之前的参数，在使用builder.add()方法，key-value方式添加键值对参数，最后调用build()生成FormBody实例。
	3. 最后Header和参数重新配置到请求中。

```
class FormPostRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        return originRequest.body() is FormBody
    }

    override fun process(originRequest: Request): Request {
        val loginService = getLoginService()
        val token = loginService?.getToken() ?: ""
        val username = loginService?.getUsername() ?: ""
        val builder = FormBody.Builder()
        //增加公共Header
        val newHeaders = Headers.Builder()
            //先添加原有Header
            .addAll(originRequest.headers())
            .add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
            .build()
        val body = originRequest.body() as FormBody
        //将以前的参数添加
        for (i in 0 until body.size()) {
            builder.add(body.encodedName(i), body.encodedValue(i))
        }
        run {
            //增加公共参数
            if (token.isNotBlank()) {
                builder.add(AppConstant.HttpParameter.TOKEN, token)
                Logger.d("FORM POST => 添加公共参数 -> ${AppConstant.HttpParameter.TOKEN} : $token")
            }
            if (username.isNotBlank()) {
                builder.add(AppConstant.HttpParameter.USERNAME, username)
                Logger.d("FORM POST => 添加公共参数 -> ${AppConstant.HttpParameter.USERNAME} : $username")
            }
        }
        val newBuilder = originRequest.newBuilder()
        //构造新的请求体
        return newBuilder
            .headers(newHeaders)
            .post(builder.build())
            .build()
    }
}
```

#### POST请求上传JSON

上面只列举了GET、POST的表单请求处理，由于POST请求上传JSON并没有提供Api，需要分析问题和解决方案，所以单独拎出来。

#### 问题

上传JSON的方式，OkGo是这样子的：(上传Json的方法：upJson())

- genericGsonType()方式，是我对创建Gson的TypeToken的一个泛型实体的拓展方法。
- password.md5LoginPwd，是我对String类拓展的拓展属性，作用就是对字符串进行MD5。
- toJson()方法，是我对Map拓展的一个使用Gson对Map进行转换为Json的方法。
- ModelConvert，对OkGo的Converter转换器接口的实现类，作用是将Json数据转换为实体模型.

```
/**
 * 泛型实体拓展GsonTypeToken的type
 * @return Type
 */
inline fun <reified T> genericGsonType(): Type = object : TypeToken<T>() {}.type

/**
 * Map转Json
 */
fun HashMap<String, String>.toJson(): String {
    return GsonUtil.toJson(this)
}

//------------------- 登录模块 -------------------

/**
 * 登录
 * @param username 用户名
 * @param password 密码
 */
fun login(
    tag: String,
    username: String,
    password: String
): Observable<HttpModel<LoginModel>> {
    val type = genericGsonType<HttpModel<LoginModel>>()
    val request: PostRequest<HttpModel<LoginModel>> = OkGo.post(ApiUrl.LOGIN_LOGIN)
    return request.tag(tag)
        .upJson(LinkedHashMap<String, String>().apply {
            put("username", username)
            //密码md5处理，这里使用拓展方法，大家可以使用自己的方式
            put("password", password.md5LoginPwd)
        }.toJson())//toJson是我添加的拓展方法，就是用Gson.toJson(map)来转换成Json的
        .converter(ModelConvert(type))
        .adapt(ObservableBody())
}

//Json转换为模型转换器
public class ModelConvert<T> implements Converter<T> {
    private Type type;
    private Class<T> clazz;
    private Gson mGson;

    public ModelConvert() {
    }

    public ModelConvert(Type type) {
        this.type = type;
    }

    public ModelConvert(Class<T> clazz) {
        this.clazz = clazz;
    }

    public void setGson(Gson gson) {
        mGson = gson;
    }

    @Override
    public T convertResponse(Response response) throws Throwable {
        ResponseBody body = response.body();
        if (body == null) {
            return null;
        }
        T data;
        try {
            if (mGson == null) {
                mGson = GsonUtil.getGson();
            }
            JsonReader jsonReader = new JsonReader(body.charStream());
            if (type != null) {
                data = mGson.fromJson(jsonReader, type);
            } else if (clazz != null) {
                data = mGson.fromJson(jsonReader, clazz);
            } else {
                Type genType = getClass().getGenericSuperclass();
                Type type = ((ParameterizedType) genType).getActualTypeArguments()[0];
                data = mGson.fromJson(jsonReader, type);
            }
        } finally {
            response.close();
        }
        return data;
    }
}
```

#### 处理器中怎么添加参数

像上面的POST表单请求，Body类型为FormBody，提供了add()方法添加键值对，而Json上传的Body是RequestBody，那么来看一下这个RequestBody类。

- RequestBody是一个抽象类，提供了2个抽象方法，contentType()返回内容类型，writeTo()写出内容数据。

- 创建RequestBody有3个create()静态方法，以及最后一个对File文件上传的create()方法。

	1. create(MediaType contentType, String content)，直接写出String字符串。
	2. create(MediaType contentType, byte[] content)，写出byte数组，上面写出字符串的create()方法是转调这个方法实现的，转调下一个支持传入byte数组开始位置和byte大小的create()方法。
	3. create(MediaType contentType, byte[] content, int offset, int byteCount)，支持byte[]数组，指定开始位置、byte大小，这个方法被上面2个方法转调，复写的writeTo()方法，写出传入的byte数组。
	4. create(MediaType contentType, File file)，支持File文件的create()方法。

```
public abstract class RequestBody {
  //子类返回内容类型
  public abstract @Nullable MediaType contentType();

  //内容长度，子类可以复写返回确定的长度，如果不确定，传-1
  public long contentLength() throws IOException {
    return -1;
  }
  
  //支持String字符串写出，内部其实就是将字符串转换为byte数组
  public static RequestBody create(@Nullable MediaType contentType, String content) {
    Charset charset = Util.UTF_8;
    if (contentType != null) {
      charset = contentType.charset();
      if (charset == null) {
        charset = Util.UTF_8;
        contentType = MediaType.parse(contentType + "; charset=utf-8");
      }
    }
    byte[] bytes = content.getBytes(charset);
    return create(contentType, bytes);
  }
  
  //支持字节数组写出
  public static RequestBody create(final @Nullable MediaType contentType, final byte[] content) {
    return create(contentType, content, 0, content.length);
  }
  
  //上面2种create()方法的最终调用，重点是writeTo()方法，将字节数据通过传入的BufferedSink实例write()方法写出
  public static RequestBody create(final @Nullable MediaType contentType, final byte[] content,
      final int offset, final int byteCount) {
      if (content == null) throw new NullPointerException("content == null");
      Util.checkOffsetAndCount(content.length, offset, byteCount);
      return new RequestBody() {
      @Override public @Nullable MediaType contentType() {
        return contentType;
      }

      @Override public long contentLength() {
        return byteCount;
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        sink.write(content, offset, byteCount);
      }
    };
  }
  
  //针对File文件上传的重载
  public static RequestBody create(final @Nullable MediaType contentType, final File file) {
    if (file == null) throw new NullPointerException("file == null");

    return new RequestBody() {
      @Override public @Nullable MediaType contentType() {
        return contentType;
      }

      @Override public long contentLength() {
        return file.length();
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        Source source = null;
        try {
          source = Okio.source(file);
          sink.writeAll(source);
        } finally {
          Util.closeQuietly(source);
        }
      }
    };
  }
}
```

#### OkGo的upJson是怎么实现的

- upJson()方法，在BodyRequest中。将json保存到成员变量content后，强制指定mediaType类型为HttpParams.MEDIA_TYPE_JSON，那么我们看下content变量什么时候使用

- generateRequestBody()方法，顾名思义，是生成请求体的，为复写的方法，这个方法中判断不同字段来确定请求不同的类型，我们是上传字符串的类型，所以走到了RequestBody.create(mediaType, content)，方法调用就是我们上面提到的RequestBody的create()静态方法。

```
public abstract class BodyRequest<T, R extends BodyRequest> extends Request<T, R> implements HasBody<R> {
    private static final long serialVersionUID = -6459175248476927501L;

    protected transient MediaType mediaType;        //上传的MIME类型
    protected String content;                       //上传的文本内容

    /** 注意使用该方法上传字符串会清空实体中其他所有的参数，头信息不清除 */
    @SuppressWarnings("unchecked")
    @Override
    public R upJson(String json) {
        this.content = json;
        this.mediaType = HttpParams.MEDIA_TYPE_JSON;
        return (R) this;
    }
    
    //子类复写生成RequestBody
    @Override
    public RequestBody generateRequestBody() {
        if (isSpliceUrl) url = HttpUtils.createUrlFromParams(baseUrl, params.urlParamsMap);

        if (requestBody != null) return requestBody;                                                //自定义的请求体
        //我们走到这个判断！
        if (content != null && mediaType != null) return RequestBody.create(mediaType, content);    //上传字符串数据
        if (bs != null && mediaType != null) return RequestBody.create(mediaType, bs);              //上传字节数组
        if (file != null && mediaType != null) return RequestBody.create(mediaType, file);          //上传一个文件
        return HttpUtils.generateMultipartRequestBody(params, isMultipart);
    }
}
```

- 题外话，我们看一下generateRequestBody()方法什么时候被调用，确保请求时被调用，generateRequestBody()方法为Request类的抽象方法，而BodyRequest类继承于Request，所以必须复写这方法。在getRawCall()方法中调用了generateRequestBody()方法，在execute()方法中调用execute()开始执行。而这个call对象则是OkHttp3提供给我们的Call接口，他的execute()方法为同步调用。

- 这下我们就放心了，BodyRequest类上的generateRequestBody()在请求时会被调用，所以上传Json的RequestBody就是RequestBody，而不是Post请求表单是FormBody，也没有单独提供子类实现。

- 虽然execute()方法是同步调用时使用，而配合RxJava是另外一个adpt()方法进行适配，不过都会调用到generateRequestBody()方法生成RequestBody对象。

```
public abstract class Request<T, R extends Request> implements Serializable {
    /** 根据不同的请求方式和参数，生成不同的RequestBody */
    protected abstract RequestBody generateRequestBody();
    
    /** 获取okhttp的同步call对象 */
    public okhttp3.Call getRawCall() {
        //构建请求体，返回call对象
        RequestBody requestBody = generateRequestBody();
        if (requestBody != null) {
            ProgressRequestBody<T> progressRequestBody = new ProgressRequestBody<>(requestBody, callback);
            progressRequestBody.setInterceptor(uploadInterceptor);
            mRequest = generateRequest(progressRequestBody);
        } else {
            mRequest = generateRequest(null);
        }
        if (client == null) client = OkGo.getInstance().getOkHttpClient();
        return client.newCall(mRequest);
    }
    
    /** 普通调用，阻塞方法，同步请求执行 */
    public Response execute() throws IOException {
        return getRawCall().execute();
    }
}
```

#### 问题重点

- 既然创建Json生成RequestBody就为RequestBody，所以重点在writeTo()方法，我们需要做一下几步：

1. 只要将sink对象捕获到，调用write()方法之前将content对象先转换为Json。
2. 添加通用字段属性后，再重新转换会字符串。
3. 再将字符串转换回byte数组，并重新定义byteCount即可！

```
//将字节数据通过传入的BufferedSink实例write()方法写出
public static RequestBody create(final @Nullable MediaType contentType, final byte[] content,
	  final int offset, final int byteCount) {
	  if (content == null) throw new NullPointerException("content == null");
	  Util.checkOffsetAndCount(content.length, offset, byteCount);
	  return new RequestBody() {
	  @Override public @Nullable MediaType contentType() {
	    return contentType;
	  }
	
	  @Override public long contentLength() {
	    return byteCount;
	  }
	
	  @Override public void writeTo(BufferedSink sink) throws IOException {
	    sink.write(content, offset, byteCount);
	  }
	};
}
```

1. 对于第一步问题，既然我们的create()方法调用了BufferedSink类的writeTo()方法，那么可以继承BufferedSink，复写writeTo()方法，对writeTo()方法进行增强，将公共参数添加即可！
2. 对于第二、三步，content字段在writeTo()方法都传入了，直接操作即可。

- 事情如果这么简单的话，也不会重点讲了~当我们继承BufferedSink时，发现BufferedSink是一个接口，而且定义的方法有10多个，我们会想到使用它的实现类，它的实现类有2个：

	1. Buffer，看样子是BufferedSink的唯一实现类。
	2. RealBufferedSink，构造方法传入了Sink接口作为参数对象，所有抽象方法都转调了这个对象，看样子是装饰器模式，不过每个方法后都调用了emitCompleteSegments()方法作为返回值，而没有用到传入的sink对象对应方法的返回值。让人有点不放心，而且这个类的源码并没有注释。

选用哪个？做决定最简单就是debug断点看一下，传入的对象是什么类型。我断点看过了，事实证明，是Buffer类的实例。

那么我们去继承Buffer类，重写writeTo()方法吧，添加参数，返回，完事！结果马上打脸了，你会发现继承会报错，仔细一看Buffer类是final的，就是说不能被继承！

```
public final class Buffer implements BufferedSource, BufferedSink, Cloneable, ByteChannel {
	//...省略代码
}
```

#### 解决思路

不能继承，好像就不能复写writeTo()方法了，一时半会卡住了，不过想了一会还是有办法的，既然不能使用继承，我们还有大杀器-组合！

我们可以模仿RealBufferedSink类，构造方法注入BufferedSink接口作为参数，自己做一遍装饰者模式，在装饰器中复写writeTo()方法，即可接触到writeTo()，进而做增强！

- BufferedSinkDecorator，基础装饰器，职责就是复写BufferedSink接口中的所有抽象方法。让子类装饰器直接复写目标装饰器即可。

```
/**
 * BufferedSink装饰器基类，将传入的真实实现全部转调一遍，子类复写对应方法即可
 */
open class BufferedSinkDecorator(private val wrapperBuffer: BufferedSink) : BufferedSink {
    //...省略其他方法，都是转调wrapperBuffer对象的对应方法

    override fun write(source: ByteArray, offset: Int, byteCount: Int): BufferedSink {
        return wrapperBuffer.write(source, offset, byteCount)
    }
    
    //...省略其他方法，都是转调wrapperBuffer对象的对应方法
}
```

- AppendBufferedSinkDecorator，具体的装饰器。复写write()方法，将方法传入的byte数组类型的source参数、byteCount字节大小进行重新定义，offset不需要处理，Buffer的源码中都是传0。

	1. 先创建String实例，将byte数组数据source传入，即可构造具有byte数组数据的字符串，这个就是Json数据。
	2. 创建JSONObject实例，传入json字符串，即可构造出Json结构的JSONObject，至于为什么使用JSONObject，而不是JSONArray，这是因为和后端定义的json结构都是以"{}"对象包裹，一般都是这样。
	3. 有了JSONObject，就可以进行put()操作，添加我们的公共参数即可。
	4. 最后将JSONObject进行toString()操作，即可重新生成json字符串，再将json字符串转为byte数组，重新获取byteCount即可。

```
/**
 * 装饰增加参数的BufferedSink装饰器
 * @param wrapperBuffer 包裹的BufferedSink实例
 * @param appendParameter 需要增加的数据
 */
class AppendBufferedSinkDecorator(
    wrapperBuffer: BufferedSink,
    private val appendParameter: Map<String, String>
) : BufferedSinkDecorator(wrapperBuffer) {
    //在write方法上动刀，将传进来的ByteArray，就是Json内容的字节数组，先解析，再加上我们的公共参数，再重新写入
    override fun write(source: ByteArray, offset: Int, byteCount: Int): BufferedSink {
        var newSource: ByteArray
        var newByteCount: Int
        try {
            //候选Json，可能是，也可能不是，如果转为JSONObject时抛出异常，则不是json
            var candidateJson = String(source)
            val jsonObject = JSONObject(candidateJson).apply {
                //添加公共参数
                appendParameter.forEach { entry ->
                    put(entry.key, entry.value)
                }
            }
            candidateJson = jsonObject.toString()
            //重置内容和长度
            newSource = candidateJson.toByteArray()
            newByteCount = newSource.size
        } catch (e: Exception) {
            e.printStackTrace()
            //不是json，忽略
            newSource = source
            newByteCount = byteCount
        }
        return super.write(newSource, offset, newByteCount)
    }
}
```

- AppendRequestBody。由于RequestBody的create()方法，已经写死了sink来调用writeTo()，而我们要将AppendBufferedSinkDecorator包裹原有的sink对象，所以RequestBody也要做一个装饰作用，重写writeTo()方法，创建AppendBufferedSinkDecorator实例，包裹原有sink对象。对原有的sink对象的writeTo()方法调用前，做添加公共参数的处理。

```
class AppendRequestBody(private val wrapperBody: RequestBody) : RequestBody() {
    override fun contentType(): MediaType? {
        return wrapperBody.contentType()
    }

    override fun writeTo(sink: BufferedSink) {
        val loginService = getLoginService()
        val token = loginService?.getToken() ?: ""
        val username = loginService?.getUsername() ?: ""
        val wrapperSink = AppendBufferedSinkDecorator(sink, LinkedHashMap<String, String>().apply {
            //增加公共参数
            if (token.isNotBlank()) {
                put(AppConstant.HttpParameter.TOKEN, token)
                Logger.d("JSON POST => 添加公共参数 -> ${AppConstant.HttpParameter.TOKEN} : $token")
            }
            if (username.isNotBlank()) {
                put(AppConstant.HttpParameter.USERNAME, username)
                Logger.d("JSON POST => 添加公共参数 -> ${AppConstant.HttpParameter.USERNAME} : $username")
            }
        })
        wrapperBody.writeTo(wrapperSink)
    }
}
```

- JsonPostRequestHandler，最后将2个装饰类在JsonPostRequestHandler类的process()方法中，进行请求的body进行包装，返回我们定义的AppendRequestBody，最后将生成的header和body重新配置即可。

```
class JsonPostRequestHandler : RequestProcessHandler {
    override fun isCanHandle(originRequest: Request): Boolean {
        return originRequest.body() is RequestBody
    }

    override fun process(originRequest: Request): Request {
        //增加公共Header
        val newHeaders = Headers.Builder()
            //先添加原有Header
            .addAll(originRequest.headers())
            .add(AppConstant.HttpParameter.PLATFORM, ApiUrl.PLATFORM)
            .build()
        //将原始Body包裹返回
        val newBody = AppendRequestBody(originRequest.body()!!)
        val newBuilder = originRequest.newBuilder()
        //构造新的请求体
        return newBuilder
            .headers(newHeaders)
            .post(newBody)
            .build()
    }
    
    //省略上面的AppendRequestBody、AppendBufferedSinkDecorator、BufferedSinkDecorator
}
```

#### 总结

本次对Json上传请求的RequestBody进行装饰器模式增强来达到公共参数的添加，统一了公共参数的添加，之前对于Json上传类型的请求，由于没有Api提供，所以都现在调用处添加拓展方法进行公共参数添加，代码冗余，而且容易漏。

今天终于强迫症忍不住要想办法将公共参数添加抽取到拦截器中，虽然遇到了一些小困难，不过还是通过装饰器来巧妙化解，今天又是有进步的一天，耶！