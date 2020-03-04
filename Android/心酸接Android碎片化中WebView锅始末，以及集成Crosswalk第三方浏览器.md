#### 心酸接Android碎片化中WebView锅始末，以及集成Crosswalk第三方浏览器

工作的公司是物联网公司，大部分业务都在前端网页，而这个网页显示在客户的电视机上，每天工作人员过来上班都要手动输入网址很麻烦，所以公司决定让我开发一个App，内嵌一个WebView，加载前端网页，并加上开机自启动，安装在客户的安卓系统电视机上。需求如下：

1. 开机自启，打开App，显示前端网页。
2. 自动全屏。

心想这么简单的需求，一天我就写完了，可不知，连这么简单的需求，却踩到了Android碎片化的生态，WebView的兼容问题上。

#### 事情经过

Activity内嵌一个WebView控件，加载Url，很简单，也不和大家贴基础代码了，如果不知道的话，资源搜索一下，相信你没一会儿的的CV大法，就完成了功能。

在手机、平板（8.0、9.0）上安装、测试没问题，收工！第二天公司订购了一个安卓系统的电视机，我把Apk安装到电视机上（没有MicroUSB口，只能用U盘），点击图标，启动，结果只加载出了HTML基础结构，数据却没有显示。马上找来前端看看，他说Ajax请求没有请求成功，麻烦的是电视机没有控制台，不能调试，也不知道出什么错。（其实App里面也没有控制器，需要连接上电脑，开启Chrome来远程调试）。

当时心想那用电脑远程调试吧，结果发现电视机没有MicroUSB口，只有大口的USB口，所以需要2头都是大口的线，公司没有这种线...，搜了一下可以用adb命令，远程连接进行调试，结果实验的发现，敲完命令请求连接，竟然被拒绝！继续搜索一番无果，只能放弃。

心想这系统是多少呀，一看才知道是4.2的系统，奸商啊，这都9102年了，手机、平板都标配9.0了，这电视机还是4.2的系统？

Android的WebView基本每个版本都修改了一下，4.4.4才替换掉Webkit内核，采用Chromenium，但是版本却低得可怜，后面7.1开始才提得比较高，后面还提供了共享模式，就是只要你安装了Chrome浏览器，它的Chromenium内核比较高的话，App的WebView会使用这个高版本的内核。

而低版本就没办法了，所以兼容性问题自然很多了，也有可能是前端使用了新的Api导致不兼容，可是没得调试，也难搞，毕竟我测试8.0、9.0的系统是没问题的。

#### 尝试接入腾讯X5

以前也是做App开发的，WebView基本上也没遇到多少兼容性问题，毕竟现在机器更新换代太快，机子基本5.0以上。既然系统WebView不行，那就换一个比较热门的WebView吧，例如比较常见的就是腾讯的X5，QQ和微信都在用，质量有保证，接入也很简单，引入jar、so，替换包名，有一点X5做得很好，就是他的Api和系统的完全一样，所以只要替换对应的包名即可，无需更改使用方式，并且还贴心提供了sh、bat批处理来全局替换，可以说非常友好了。（接入方式官方文档有哈）

刷刷刷一下子就替换好了，手机、平板都测试没有问题，打包发到电视机，心想这下可没问题了吧！

结果让人心塞，竟然结果是一样的，X5竟然还是治不了这锅！

#### 系统浏览器打开

当时没想太多，觉得那么既然内嵌WebView不行，那就用Intent调起系统浏览器吧。

```
/**
 * 从本地浏览器中打开
 */
private fun goLocalBrowser(url: String) {
    url.localBrowserOpen(this) {
        //打开失败，没有安装浏览器
        Toast.makeText(applicationContext, it, Toast.LENGTH_SHORT).show()
    }
}

/**
 * String拓展方法，使用本地浏览器打开Url
 */
fun String.localBrowserOpen(context: Context, openFail: ((msg:String) -> Unit)?=null) {
    val intent = Intent().apply {
        action = Intent.ACTION_VIEW
        data = Uri.parse(this@localBrowserOpen)
        //避免有些机型上会出现浏览器拉不起来的情况，例如华为P20
        addCategory(Intent.CATEGORY_BROWSABLE)
        if (context !is Activity) {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        }
    }
    //先判断是否有浏览器可以响应，有才进行跳转
    val manager = context.packageManager
    val intentActivities = manager.queryIntentActivities(intent, 0)
    if (intentActivities.size > 0) {
        context.startActivity(intent)
    } else {
        openFail?.invoke("您没有安装浏览器，请安装一个浏览器")
    }
}
```

刷刷刷，就上面的代码就打开了，相信你也会！结果让我大跌眼镜，前端网页在系统的浏览器也是同样的情况，心想则浏览器是不是有问题啊！

然后我试了百度、新浪这种大公司的网站，百度基本没问题，只是有些样式不太对，新浪则是完全跑完进度条就没然后了！

这可是遇上了大锅了，这摆明这个系统的WebView是有问题的！后续我下载了UC、夸克、QQ浏览器、搜狗浏览器、Chrome、Firefox、Opera，国产浏览器全部阵亡，不是加载不出就是只加载出HTML骨架，而国外浏览器则只有Chrome和Opera能加载出来。

由于公司需求，开启自启后，如果是打开浏览器，需要自动全屏：

1. Chrome，没有全屏功能！（妈耶，竟然没有这个功能，难道国外的用户没有这种需求？）
2. Opera，有全屏功能，但是不会记录全屏状态，每次都要去点，而且没有得设置可以记录状态（国外的产品经理，你们这全屏功能做了一半，没做一半呀！）

最终只能选择Opera，除了客户要安装我们的App外，还要额外安装一个Opera浏览器，并且因为Intent调起系统浏览器，如果有多个浏览器，会弹窗让我们进行选择。而基本上每个电视盒子都会有一个自带浏览器，所以使用前还要配置一下，让客户选择并设置为默认浏览器，下次启动则不需要再选择。

#### 新的锅，接续接吧

后续，公司觉得买一个电视机太贵了，出不多2000块一台，所以决定买一个电视盒子，外接一个电视机显示屏，成本更低。没多久电视机就到货了，作为公司唯一的安卓开发，自然要自己去调试了。

这次我学聪明了，先看一下系统版本，是5.1的，嗯，很好，这次应该没啥问题了，毕竟5.0是Android一个分水岭，整个Android界面都重构了。

安装、打开一气呵成，点击图标，让我更加不明所以的问题发生了，一打开就闪退了！擦，这什么操作啊，马上我将异常信息打到Toast上显示：

```
android.view.InflateException: Binary XML file line #7: Error inflating class android.webkit.WebView
```

这错误更加让我匪夷所思，怀疑人生，我们知道，Xml中写的控件，LayoutInflater会进行反射创建，找不到对应的控件时，就会出现这种错误。马上百度搜索一下，果然有这种情况，而且也是5.1版本，找到了一种修复方法，马上CV尝试。在Manifest的Application节点下添加

```
<meta-data android:name="android.webkit.WebView.EnableSafeBrowsing"
                   android:value="true"/>
```

打包再安装，结果是一样的，依旧抛出InflateException异常，一时半会想不到原因，只能想到一种可能性，就是电视机的厂家，定制了Android ROM，将WebView控件删除了！随后问了一下客服，客服咨询技术支持后，说是有这个控件的...但是事实摆在眼前，其他手机、平板都没有问题，证明代码没问题。

后面让厂家发了一个7.1的ROM过来安装，结果就可以了！啊~这锅背的真的是心酸了。而且自带内嵌WebView的方式也正常了，真是喜大普奔啊！

#### 做B端也有痛苦

公司主要业务是对B端，意思就是对客户，而不是以前的App开发，是对用户，对C端。以前遇到一些用户的手机兼容性问题，基本是冷门手机、兼容、修复不了，就含糊过去了，但是对B端，客户就是上帝，客户就是公司资金来源，遇到问题还是要搞！

如果是我们测试好，固定发货给固定客户倒好，毕竟都是没问题的，而有些客户已经有电视机了，自然不会再搞一台，遇到低版本的机子还是有上面的问题。

这不，今天来上班，就收到了老板电话，客户的电视机是小米电视，安装了我的App，内嵌WebView显示不出来，估计系统版本十有八九是7.1以下。毕竟现在很多电视机的系统还是5.1，没有到7.1。

#### 兼容还是要做的，Crosswalk来打救

没办法，还是有很多客户的电视机是7.1以下，兼容还是要做，只能另找方法，开始搜索WebView兼容处理的文章。

了解到有一个名叫Crosswalk的第三方浏览器，不过现在已经不维护了。它采用Chromenium内核，是独立内核到App，不使用系统的WebView内核。所以so会比较大，我接完后对比了一下，接入之前是3M，接入后是28M。

如果是对C端，接独立内核无疑加大安装包大小，基本不会采用这种方案，而采用X5，但是我这种对B端，不考虑Apk大小，而是能使用才是王道，毕竟现在出现很多不能使用的情况。

二话不说，接入Crosswalk走起~

#### Crosswalk接入

Crosswalk接入有3种方式：

1. Maven远程依赖（测试发现Maven依赖不上，估计不维护关闭了Maven私服吧）

```
//配置仓库
repositories {
    maven {
        url 'https://download.01.org/crosswalk/releases/crosswalk/android/maven2'
    }
}

//依赖
dependencies {
    implementation 'org.xwalk:xwalk_core_library:23.53.589.4'
}
```

2. arr文件依赖，官方地址下载arr，依赖（我采用这种方式）
[下载地址]([https://download.01.org/crosswalk/releases/crosswalk/android/](https://download.01.org/crosswalk/releases/crosswalk/android/)
)

版本说明：

- stable（稳定版，推荐）
- beta（测试版）
- canary（金丝雀版）

![crosswalk各版本下载.png](https://upload-images.jianshu.io/upload_images/1641428-01616d09495ae223.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选择下载最新版本.png](https://upload-images.jianshu.io/upload_images/1641428-3a13265ec163e200.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选择下载arr文件.png](https://upload-images.jianshu.io/upload_images/1641428-4af640c07b1479ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当前最新版本为crosswalk-23.53.589.4，我们将arr文件导入到lib目录下，再在App或模块Gradle文件下，添加依赖

```
repositories {
    flatDir {
        dirs 'libs'
    }
}
```

```
dependencies {
    implementation(name: 'crosswalk-23.53.589.4', ext: 'aar')
}
```

3. 下载源码，导入module（我觉得没必要，每多一个module依赖，就会导致编译减慢，arr则不会，并且库代码不变的，使用arr更好）

#### 开始配置

Crosswalk的Api，有一些和系统的不一样，所以替换其他要花点时间，像X5那样都是同名方法的话就方便多了，出问题了再换回去也方便。

- 清单中，添加权限

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
```

- 替换WebView为XWalkView

```
<org.xwalk.core.XWalkView
        android:id="@+id/web_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

- 初始化XWalkView，XWalkView的初始化和传统的WebView有点区别，他需要继承XWalkActivity，在onXWalkReady()回调时，再进行对XWalkView进行配置和操作。但是XWalkActivity继承于Activity，而我是将WebView抽取到Fragment的，所以我直接将XWalkActivity里面的代码拷贝出来使用做一个基类，就可以了！

```
public abstract class BaseXWalkActivity extends BaseActivity {
    private XWalkActivityDelegate mActivityDelegate;

    /**
     * XWalkView初始化完成，在这个回调用配置XWalkView
     */
    protected abstract void onXWalkReady();

    public boolean isXWalkReady() {
        return this.mActivityDelegate.isXWalkReady();
    }

    public boolean isSharedMode() {
        return this.mActivityDelegate.isSharedMode();
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Runnable cancelCommand = new Runnable() {
            @Override
            public void run() {
                BaseXWalkActivity.this.finish();
            }
        };
        Runnable completeCommand = new Runnable() {
            @Override
            public void run() {
                BaseXWalkActivity.this.onXWalkReady();
            }
        };
        this.mActivityDelegate = new XWalkActivityDelegate(this, cancelCommand, completeCommand);
    }

    @Override
    protected void onResume() {
        super.onResume();
        this.mActivityDelegate.onResume();
    }
}
```

- 继承BaseXWalkActivity，通知Fragment配置XWalkView

```
class InnerWebActivity : BaseXWalkActivity() {
    private lateinit var mWebBrowseFragment: AppWebBrowseFragment
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //添加Fragment，每个项目的封装方式不一样，代码我就不贴了，不是重点...
        mWebBrowseFragment = ...
    }

    override fun onXWalkReady() {
        //通过Fragment初始化，配置XWalkView
        mWebBrowseFragment.onWebViewEnvReady()
    }
}
```

- Fragment中，配置XWalkView

```
public class AppWebBrowseFragment extends BaseFragment {
    private XWalkView vWebView;
    
    @Nullable
    @Override
    public View onCreateView(@NotNull LayoutInflater inflater, Nullable ViewGroup container, Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.base_app_web_browse_fragment, container, false);
    }

    @Override
    public void onViewCreated(@NotNull View view, @org.jetbrains.annotations.Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        vWebView = view.findViewById(R.id.web_view);
    }
    
        /**
     * 提供给外部，外部环境初始化完毕后，才回调，再开始WebView的初始化
     */
    public void onWebViewEnvReady() {
        setupWebView();
    }

    @SuppressLint("SetJavaScriptEnabled")
    private void setupWebView() {
        //开启默认动画
        XWalkPreferences.setValue(XWalkPreferences.ANIMATABLE_XWALK_VIEW, true);
        XWalkSettings settings = vWebView.getSettings();
        settings.setLoadWithOverviewMode(false);
        //支持JS
        settings.setJavaScriptEnabled(true);
        //支持通过JS打开新窗口
        settings.setJavaScriptCanOpenWindowsAutomatically(true);
        //将图片调整到合适WebView的大小
        settings.setUseWideViewPort(true);
        //缩放至屏幕的大小
        settings.setLoadWithOverviewMode(true);
        //支持自动加载图片
        settings.setLoadsImagesAutomatically(true);
        //支持多窗口
        settings.supportMultipleWindows();
        //支持缩放
        settings.setSupportZoom(true);
        //支持双击缩放、缩小
        settings.setBuiltInZoomControls(true);
        //设置允许访问见
        settings.setAllowFileAccess(true);
        //设置支持访问内容
        settings.setAllowContentAccess(true);
        //开启DOM存储
        settings.setDomStorageEnabled(true);
        //不进行缓存
        settings.setCacheMode(WebSettings.LOAD_NO_CACHE);
        //加载Url
        vWebView.loadUrl(mWebIntentParams.getUrl());
    }
}
```

- 配置类似系统WebView的WebChromeClient，配置XWalkView的XWalkUIClient

```
vWebView.setUIClient(new XWalkUIClient(vWebView) {
    @Override
    public void onReceivedTitle(XWalkView view, String title) {
        super.onReceivedTitle(view, title);
        //获取到网页的Title
    }
});
```

- 配置类似系统WebView的WebViewClient，配置XWalkView的XWalkResourceClient

```
vWebView.setResourceClient(new XWalkResourceClient(vWebView) {
    @Override
    public void onProgressChanged(XWalkView view, int progressInPercent) {
        super.onProgressChanged(view, progressInPercent);
        //处理加载进度...
    }

    @Override
    public void onLoadStarted(XWalkView view, String url) {
        super.onLoadStarted(view, url);
        //开始加载
    }

    @Override
    public void onLoadFinished(XWalkView view, String url) {
        super.onLoadFinished(view, url);
        //加载结束
    }

    @Override
    public void onReceivedLoadError(XWalkView view, int errorCode, String description, String failingUrl) {
        super.onReceivedLoadError(view, errorCode, description, failingUrl);
        //加载发生错误
    }
});
```

- 判断是否可以返回上一页

```
public boolean canGoBack() {
    return vWebView.getNavigationHistory().canGoBack();
}
```

- 返回上一页

```
public void goBack() {
    vWebView.getNavigationHistory().navigate(XWalkNavigationHistory.Direction.BACKWARD, 1);
}
```

- 重新加载当前Url

```
public void reloadUrl() {
    vWebView.reload(0);
}
```

- 获取当前加载的Url

```
public String getLoadUrl() {
    return vWebView.getUrl();
}
```

- 将生命周期回调XWalkView。但停止维护的时候为Android6.0，后面7.0的SDkApi更改了，现在调用会抛出TextureView doesn't support displaying a background drawable的异常，所以我没有调用，只回调了onDestroy()生命周期。

```
@Override
public void onResume() {
    super.onResume();
    if (vWebView != null) {
        vWebView.resumeTimers();
        vWebView.onShow();
    }
}

@Override
public void onPause() {
    super.onPause();
    if (vWebView != null) {
        vWebView.pauseTimers();
        vWebView.onHide();
    }
}

@Override
public void onDestroy() {
    super.onDestroy();
    XWalkPreferences.setValue(XWalkPreferences.ANIMATABLE_XWALK_VIEW, false);
}
```

- 回调onActivityResult()给XWalkView。

```
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (vWebView != null) {
        vWebView.onActivityResult(requestCode, resultCode, data);
    }
}
```

- 回调onNewIntent()给XWalkView，但Fragment没有onNewIntent()方法，所以需要在Activity中通知Fragment。

```
class InnerWebActivity : BaseXWalkActivity() {
    private lateinit var mWebBrowseFragment: AppWebBrowseFragment
    
    //...省略其他代码
    
    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        mWebBrowseFragment.onNewIntent(intent)
    }
}

public class AppWebBrowseFragment extends BaseFragment {
    //...省略其他代码
    
    public void onNewIntent(Intent intent) {
        if (vWebView != null) {
            vWebView.onNewIntent(intent);
        }
    }
}
```

#### Crosswalk也有坑

- 进入后台，再从桌面点击图标启动，抛出异常

原因是Crosswalk从Android6.0后便停止维护，显示WebView内容是使用TextureView，但库的内部调用了TextureView的setBackgroundDrawable()，该方法从7.0开始则不能使用了，7.0后调用则会抛出UnsupportedOperationException异常，信息为TextureView doesn't support displaying a background drawable。

出现这种问题，要么clone下来Crosswalk的源码去改，要么规避这个问题，由于我的App目前都是跑在广告机，一直在前台，就简单粗暴的try-catch了。

```
@Override
public View onCreateView(@NotNull LayoutInflater inflater, @org.jetbrains.annotations.Nullable ViewGroup container, @org.jetbrains.annotations.Nullable Bundle savedInstanceState) {
    View view = null;
    try {
        view = super.onCreateView(inflater, container, savedInstanceState);
    } catch (Throwable e) {
        e.printStackTrace();
    }
    //这里处理XWalk独立内核里面的Bug，App进入后台，从桌面图标点击时，会因为7.0-API改动原因，TextureView的setBackgroundDrawable()会抛出异常
    if (view == null) {
        //重启App
        AppUtils.relaunchApp(true);
    }
    return view;
}
```

- 同时集成腾讯X5内核和Crosswalk会导致冲突

由于上面Crosswalk不维护的问题，所以集成腾讯X5，在7.0以上使用X5的WebView，以下则使用Crosswalk的WebView。一切都很美好，结果一切具备，只欠启动，结果就出问题了！

- 同时集成，选用X5，结果内部反射创建WebView，一直失败，导致一直都降级为原生WebView。

```
2019-12-23 09:42:51.540 25557-25557/com.tongwei.bootrunsample.internal E/WebCoreProxy: createSDKWebview failure - th: java.lang.reflect.InvocationTargetException; msg: null; cause: java.lang.NullPointerException: Attempt to invoke interface method 'com.tencent.tbs.core.webkit.WebViewProvider com.tencent.tbs.core.webkit.WebViewFactoryProvider.createWebView(com.tencent.tbs.core.webkit.WebView, com.tencent.tbs.core.webkit.WebView$PrivateAccess)' on a null object reference
        at com.tencent.tbs.core.webkit.WebView.ensureProviderCreated(Unknown Source:16)
        at com.tencent.tbs.core.webkit.WebView.setOverScrollMode(Unknown Source:3)
        at android.view.View.<init>(View.java:4592)
        at android.view.View.<init>(View.java:4724)
        at android.view.ViewGroup.<init>(ViewGroup.java:597)
        at android.widget.AbsoluteLayout.<init>(AbsoluteLayout.java:55)
        at android.widget.AbsoluteLayout.<init>(AbsoluteLayout.java:51)
        at com.tencent.tbs.core.webkit.AbsoluteLayout.<init>(Unknown Source:0)
        at com.tencent.tbs.core.webkit.WebView.<init>(Unknown Source:0)
        at com.tencent.tbs.core.webkit.WebView.<init>(Unknown Source:7)
        at com.tencent.tbs.core.webkit.WebView.<init>(Unknown Source:1)
        at com.tencent.tbs.core.webkit.tencent.TencentWebViewProxy$InnerWebView.<init>(Unknown Source:3)
        at com.tencent.tbs.core.webkit.tencent.TencentWebViewProxy.<init>(Unknown Source:114)
        at com.tencent.tbs.core.webkit.adapter.X5WebViewAdapter.<init>(Unknown Source:10)
        at java.lang.reflect.Constructor.newInstance0(Native Method)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:334)
        at com.tencent.tbs.tbsshell.WebCoreProxy.createSDKWebViewInstance(Unknown Source:40)
        at com.tencent.tbs.tbsshell.partner.webaccelerator.WebViewPool.acquireWebView(Unknown Source:40)
        at com.tencent.tbs.tbsshell.partner.WebCoreReflectionPartnerImpl.createSDKWebview(Unknown Source:30)
        at com.tencent.tbs.tbsshell.WebCoreProxy.createSDKWebview(Unknown Source:4)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.tencent.smtt.export.external.DexLoader.invokeStaticMethod(DexLoader.java:281)
        at com.tencent.smtt.sdk.u.a(X5CoreWizard.java:108)
        at com.tencent.smtt.sdk.QbSdk$3.handleMessage(QbSdk.java:1809)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:192)
        at android.app.ActivityThread.main(ActivityThread.java:6866)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:550)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:817)
```

- 同时集成，选用Crosswalk，导致Crosswalk初始化时抛出异常，说调用初始化是不在主线程，但我的调用是在Activity的onCreate()调用，必定在主线程，就算使用单独的Handler进行post调用，也是抛出该异常。

```
2019-12-23 09:42:52.435 25557-25557/com.tongwei.bootrunsample.internal E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.tongwei.bootrunsample.internal, PID: 25557
    java.lang.RuntimeException: java.lang.RuntimeException: Did not yet override the UI thread
        at org.xwalk.core.ReflectMethod.invoke(ReflectMethod.java:67)
        at org.xwalk.core.XWalkCoreWrapper.initXWalkView(XWalkCoreWrapper.java:241)
        at org.xwalk.core.XWalkCoreWrapper.dockXWalkCore(XWalkCoreWrapper.java:202)
        at org.xwalk.core.XWalkLibraryLoader$ActivateTask.onPostExecute(XWalkLibraryLoader.java:341)
        at org.xwalk.core.XWalkLibraryLoader$ActivateTask.onPostExecute(XWalkLibraryLoader.java:317)
        at android.os.AsyncTask.finish(AsyncTask.java:695)
        at android.os.AsyncTask.-wrap1(Unknown Source:0)
        at android.os.AsyncTask$InternalHandler.handleMessage(AsyncTask.java:712)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:192)
        at android.app.ActivityThread.main(ActivityThread.java:6866)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:550)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:817)
     Caused by: java.lang.RuntimeException: Did not yet override the UI thread
        at org.chromium.base.ThreadUtils.getUiThreadHandler(ThreadUtils.java:51)
        at org.chromium.base.ThreadUtils.runningOnUiThread(ThreadUtils.java:199)
        at org.chromium.base.ThreadUtils.runOnUiThreadBlocking(ThreadUtils.java:66)
        at org.xwalk.core.internal.XWalkViewDelegate.startBrowserProcess(XWalkViewDelegate.java:195)
        at org.xwalk.core.internal.XWalkViewDelegate.init(XWalkViewDelegate.java:138)
        at java.lang.reflect.Method.invoke(Native Method)
        at org.xwalk.core.ReflectMethod.invoke(ReflectMethod.java:61)
        at org.xwalk.core.XWalkCoreWrapper.initXWalkView(XWalkCoreWrapper.java:241) 
        at org.xwalk.core.XWalkCoreWrapper.dockXWalkCore(XWalkCoreWrapper.java:202) 
        at org.xwalk.core.XWalkLibraryLoader$ActivateTask.onPostExecute(XWalkLibraryLoader.java:341) 
        at org.xwalk.core.XWalkLibraryLoader$ActivateTask.onPostExecute(XWalkLibraryLoader.java:317) 
        at android.os.AsyncTask.finish(AsyncTask.java:695) 
        at android.os.AsyncTask.-wrap1(Unknown Source:0) 
        at android.os.AsyncTask$InternalHandler.handleMessage(AsyncTask.java:712) 
        at android.os.Handler.dispatchMessage(Handler.java:106) 
        at android.os.Looper.loop(Looper.java:192) 
        at android.app.ActivityThread.main(ActivityThread.java:6866) 
        at java.lang.reflect.Method.invoke(Native Method) 
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:550) 
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:817) 
```

原因和解决方案

- 原因：X5和Crosswalk有依赖同样的类库，类名是一样的，导致冲突。

- 解决方案：在初始化X5前，设置TBS_SETTINGS_USE_PRIVATE_CLASSLOADER字段为true，让X5使用独立的ClassLoader初始化。

```
//由于crosswalk及quic与x5存在相同的包名，会导致内核默认加载app classLoader中的类，导致内核加载失败或者出现crash
//使用以下配置，可让X5使用独立ClassLoader进行加载
HashMap<String, Object> map = new HashMap<>(16);
map.put(TbsCoreSettings.TBS_SETTINGS_USE_PRIVATE_CLASSLOADER, true);
QbSdk.initTbsSettings(map);
//初始化环境
QbSdk.initX5Environment(application, new QbSdk.PreInitCallback() {
    @Override
    public void onCoreInitFinished() {
    }

    @Override
    public void onViewInitFinished(boolean isInitSuccess) {
        if (isInitSuccess) {
            //初始化成功...
        } else {
            //失败...
        }
    }
});
```

#### 测试和总结

首先为了确认问题，我将我的Nexus5下了2个ROM，分别是5.1和6.0，刷机后，安装未接入Crosswalk的apk，发现5.1和6.0都同样加载不出数据，所以其实也不完全是电视机系统的锅，的确存在兼容性问题。再安装接入Crosswalk的apk，加载完成。

原本使用WebView显示前端网页是一件很简单的事，却出了这么多锅，安卓的碎片化的确严重，导致很多官方Api在不同的厂商ROM下，各有各的问题，例如权限申请，透明状态栏等，都出了很多兼容库，却没有一个统一的库处理。 