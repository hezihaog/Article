#### Android WebView和Js交互

现在纯原生Android开发越来越少了，现在一般App都会混合开发，其他混合的技术先不说，最常用就是WebView加载H5页面，再App客户端和Web端交互，提供一些用户信息、客户端Api等，本篇介绍WebView调用Js，Js调用Android方法的知识。

#### 本文使用的Html文件

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>WebView</title>
    <style type="text/css">
        body {
            background: #caff0c;
        }

        .btn {
            line-height: 40px;
            margin: 10px;
            background: #cccccc;
        }

        #input {
            margin-left: 5px;
        }
    </style>
</head>
<body>
<h2>WebView</h2>
<div><span>请输入要传递的数据</span><input type="text" id="input"></div>
<div id="btn"><span class="btn">点我</span></div>
<div><a href="info://info?name=wally&age=18">点我跳转链接</a></div>

<script type="text/javascript">
	//5.按钮点击，调用客户端方法
    var btnEle = document.getElementById("btn");
    var inputEle = document.getElementById("input");
    btnEle.addEventListener("click", function () {
        var str = inputEle.value;
        if (window.androidJs) {
            window.androidJs.setValue(str);
        } else {
            alert("window androidJs is not found！")
        }
    })

    //提供给客户端调用的有参Js方法，设置输入文字
    var remote = function (str) {
        inputEle.value = str;
    }

    //带返回值的Js方法，给客户端获得输入的文字
    var remoteWithValue = function () {
        return inputEle.value;
    }
</script>
</body>
</html>
```

#### JS调用Android方法

- （方式一）

	1. 允许WebView加载Js代码
	2. 编写一个提供给Web的Js接口类（方法要加上@JavascriptInterface注解）
	3. 给WebView添加Js接口（将接口对象挂载到Web的window下）
	4. Web端调用window下的接口类对象中的Android方法

Android客户端，设置允许加载Js代码，提供Js接口类，注入到Web端的Window中。

```
/**
 * 加载的Html位置
 */
private static final String SETUP_HTML = "file:///android_asset/index.html";

//1.允许WebView加载Js代码
mWebView.getSettings().setJavaScriptEnabled(true);

/**
 * 2.编写一个提供给Web的Js接口类
 */
public class JsInterface {
    private static final String TAG = JsInterface.class.getSimpleName();
    private Callback mCallback;

    public JsInterface(Callback callback) {
        mCallback = callback;
    }

    @JavascriptInterface
    public void setValue(String value) {
        if (TextUtils.isEmpty(value)) {
            return;
        }
        Log.d(TAG, "value: " + value);
        if (mCallback != null) {
            mCallback.onSetValue(value);
        }
    }

    public interface Callback {
        void onSetValue(String value);
    }
}

//3.给WebView添加Js接口
mWebView.addJavascriptInterface(new JsInterface(new JsInterface.Callback() {
    @Override
    public void onSetValue(final String value) {
        mMainHandler.post(new Runnable() {
            @Override
            public void run() {
                mResult.setText("value: " + value);
            }
        });
    }
}), "androidJs");
        
//4.加载Html
mWebView.loadUrl(SETUP_HTML);
```

Html中按钮点击时，Js调用客户端方法

```
//5.按钮点击，调用客户端方法
var btnEle = document.getElementById("btn");
var inputEle = document.getElementById("input");
btnEle.addEventListener("click", function () {
    var str = inputEle.value;
    //判断是否在客户端环境
    if (window.androidJs) {
        window.androidJs.setValue(str);
    } else {
        alert("window androidJs is not found！")
    }
})
```

- （方式二）
	1. 和客户端协商协议，以该协议加载Url，客户端WebView.setWebViewClient()，复写shouldOverrideUrlLoading()进行拦截，获取行为和参数。

```
//和前端协商的协议前缀
private static final String CALLBACK_SCHEME = "info://info";

mWebView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        //是否拦截Url
        boolean isIntercept = url.startsWith(CALLBACK_SCHEME);
        //处理拦截特定前缀Url
        handleUrlIntercept(url);
        if (isIntercept) {
            return true;
        } else {
            return super.shouldOverrideUrlLoading(view, url);
        }
    }
});
        
/**
 * 处理拦截特定前缀Url
 */
private void handleUrlIntercept(String text) {
    HashMap<String, String> paramsMap = new HashMap<>(5);
    String result = text.replaceFirst(CALLBACK_SCHEME.concat("\\?"), "");
    String[] paramsGroup = result.split("&");
    //按组拆分bold=false
    for (String groupStr : paramsGroup) {
        String[] paramsKeyValue = groupStr.split("=");
        //参数和值拆分
        String paramsName = paramsKeyValue[0];
        String paramsValue = paramsKeyValue[1];
        paramsMap.put(paramsName, paramsValue);
    }
    final String name = paramsMap.get("name");
    final String age = paramsMap.get("age");
    toast("拦截Url，获取参数 -> " + "name:" + name + " age:" + age);
}
```

Web端，超链接加载自定义协议，客户端进行拦截

```
//点击超链接，加载自定义协议，客户端进行拦截
<div><a href="info://info?name=wally&age=18">点我跳转链接</a></div>
```

- （方式三）
	1. 使用prompt，发送数据，客户端webView.setWebChromeClient，复写onJsPrompt进行拦截，获取数据。

```
mWebView.setWebChromeClient(new WebChromeClient() {
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        //拦截掉，回调4.4一下的Js返回值处理
        OnMethodResultCallback callback = mCompatKitkatJsMethodCallbackMap.get(url);
        if (callback != null) {
            callback.onMethodResult(message);
            mCompatKitkatJsMethodCallbackMap.remove(url);
            //拦截后必须取消掉，不发送到客户端WebView，不调用也不行，会阻塞等待结果
            result.cancel();
            return true;
        } else {
            return super.onJsPrompt(view, url, message, defaultValue, result);
        }
    }
});
```
	
#### Android调用Js方法

```
/**
 * 调用JS方法获取返回值的监听
 */
interface OnMethodResultCallback {
    /**
     * 返回值返回时回调
     *
     * @param result 函数返回值
     */
    void onMethodResult(String result);
}
```

- 调用无返回值Js方法
	1. Web提供Js方法
	2. Android客户端调用WebView.loadUrl(“javascript:methodName(params)”);，调用Js方法。

客户端，设置发送文字调用Js方法，设置输入内容

```
//调用无返回值的Js方法
mSend.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        String str = mInput.getText().toString();
        mWebView.loadUrl("javascript:if(window.remote){window.remote('" + str + "')}");
    }
});
```

Web端提供的Js方法，设置输入内容

```
//提供给客户端调用，设置输入文字
var remote = function (str) {
    inputEle.value = str;
}
```

- 调用有参Js方法
	1. 4.4以上，调用webView.evaluateJavascript()，提供一个ValueCallback获取返回值。
	2. 4.4一下，没有获取返回值的Api，只能使用alert，prompt，confirm。在mWebView.setWebChromeClient()中复写onJsAlert()、onJsConfirm()、onJsPrompt()拦截。

```
//调动有返回值的Js方法
mCallJsParamsMethod.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        final OnMethodResultCallback onCall = new OnMethodResultCallback() {
            @Override
            public void onMethodResult(String result) {
                toast("收到调用的Js方法返回值: " + result);
            }
        };
        if (isOvertopKitkat()) {
            //4.4以上可以直接用
            mWebView.evaluateJavascript("javascript:remoteWithValue();", new ValueCallback<String>() {
                @Override
                public void onReceiveValue(String value) {
                    onCall.onMethodResult(value);
                }
            });
        } else {
            mCompatKitkatJsMethodCallbackMap.put(SETUP_HTML, onCall);
            //4.4以下，使用prompt将信息带出去，受到信息再拦截掉
            mWebView.loadUrl("javascript:prompt(remoteWithValue());");
        }
    }
});

mWebView.setWebChromeClient(new WebChromeClient() {
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        //拦截掉，回调4.4一下的Js返回值处理
        OnMethodResultCallback callback = mCompatKitkatJsMethodCallbackMap.get(url);
        if (callback != null) {
            callback.onMethodResult(message);
            mCompatKitkatJsMethodCallbackMap.remove(url);
            //拦截后必须取消掉，不发送到客户端WebView，不调用也不行，会阻塞等待结果
            result.cancel();
            return true;
        } else {
            return super.onJsPrompt(view, url, message, defaultValue, result);
        }
    }
});

/**
 * 版本号是否大于等于4.4
 */
protected boolean isOvertopKitkat() {
    return Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT;
}
```