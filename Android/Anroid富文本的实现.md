#### Anroid富文本的实现

最近项目需要在Android端实现富文本编辑，提交Html标签到后端。查看内容则是后端接口传Html标签，支持重新编辑等。也是顺利实现了，特此记录一下~

#### 需求

- 支持设置字体大小
- 支持设置预览提示文字（Placeholder）
- 支持设置、取消设置，粗体
- 支持设置、取消设置，斜体
- 支持设置、取消设置，下划线
- 支持增加分隔线
- 支持插入图片（插入图片url）
- 支持撤销、取消撤销，上一步操作
- 支持清空所有内容
- 支持上传图片（这里是客户端上传，再将图片url给Web库插入Img标签）
- 支持回显（设置已有Html到编辑器中）
- 支持光标移动回显已设置的属性项
- 支持字体大小改变监听

#### 调研

富文本实现有以下3种方式可选，分别各有优缺点：

- 使用多种Android中的Layout，每种布局对应Html的一种格式（需要做一层映射）。总体来说比较复杂，也需要完全重新写，不能多端共用（前端、IOS），也比较花时间。

- WebView + JavaScript实现。现在Web端有很多成熟的JavaScript富文本编辑库。客户端只要做好和Web端的JS交互即可，总体实现代码在Web端。实现中，容易遇到Android的WebView的兼容性问题。例如图片上传选择无效，调用JS函数在4.4以下无法获得返回值等兼容性问题（还有其他奇葩问题。。我就遇到字符乱码的，但是IOS毛事都没有！）。

- EditText + SpanableString。Android中的SpanableString也可以实现富文本，只是说还是要自己写，项目时间赶呀，避免少踩坑还是用成熟的方案好。

最后采用了第二种方式：WebView + JavaScript。也反编译了某乎，某乎也是使用该方案！

其实除了WebView + JavaScript，还可以使用MarkDown语法做标记（没有选择的原因还是，主要实现在Android和IOS，2端代码不能共用，容易出差异性问题！）

#### JS交互梳理

- 首先，Web库的资源是放在assets文件夹中的。（HTML\CSS\JS等）
- 跳转页面后，加载assets文件夹中的页面.html，这里是index.html，由实现类中定义，基类只定义总体流程和提供基础调用JS的方法（也处理4.4以上、以下的兼容）。
- 像设置字体大小、粗体等都是调用JS函数，并获取返回值实现。如光标移动获取已设置的属性，是Web库load一个我们自定义的协议，客户端通过拦截加载的url，判断是否是我们协定的协议url来判断是否拦截（例如前缀判断），并获取其中的数据和参数（例如像Url带参数是以?号开始，每个键值对以=号分隔）。

#### 总体设计

因为当时没有前端大佬在，富文本的JS库都是之前IOS大佬改**simditor库**源码提供给客户端的（膜拜~），就暂时采用了这个库。但是毕竟不是做前端的，有bug不会改，要加功能不会加，所以很有可能以后换库或者有前端大佬相助做一套，应该将富文本的操作、监听、回显等操作抽象成接口，具体实现交给实现类！

- 定义操作接口、字体大小等类型枚举
- 因为需要上传图片，所以要设计上传参数Option类
- 同样因为要上传图片，肯定要做一定的压缩，所以也要有压缩参数option类
- 上传图片的回调接口
- 每种Web库的实现，其实因为抽象了接口，不依赖一定是JS+WebView实现，替换为别的实现也可以！
- 外部注册、更换具体实现的单例管理对象

1. 定义操作接口和字体大小枚举

```
public interface ILinghitRichEditor {
    /**
     * 依附WebView，在WebView创建时会调用
     */
    void attachWebView(WebView webView);

    /**
     * 分发onDestroy事件，需要外部调用，在Activity和Fragment的onDestroy()时调用，内部会将文件、图片上传等任务移除
     */
    void dispatchDestroy();

    //----------> 核心方法 <----------

    /**
     * 设置H标签的等级大小，这个字体大小是全局的，不能指定某个字体大或小
     *
     * @param sizeEnum 等级枚举
     */
    void setTitleSize(TitleSizeEnum sizeEnum);

    /**
     * 恢复到默认字体大小
     */
    void setDefaultTitleSize();

    /**
     * 获取当前设置的字体大小
     */
    TitleSizeEnum getCurrentTitleSize();

    /**
     * 设置窗体大小
     */
    void setFrameSize(int width, int height);

    /**
     * 设置预览文字
     */
    void setTextPlaceholder(String placeholder);

    /**
     * 设置或取消粗体
     */
    void toggleBold();

    /**
     * 设置或取消斜体
     */
    void toggleItalic();

    /**
     * 设置或取消下划线
     */
    void toggleUnderline();

    /**
     * 增加分割线
     */
    void addDivider();

    /**
     * 设置或取消删除线
     */
    void toggleDeleteLine();

    /**
     * 直接插入图片Url，这里插入图片，需要先上传到图片服务器后，再将图片的完整Url地址传过来，在WebView中加载
     */
    void insertImages(String... imgUrls);

    /**
     * 以路径形式批量插入图片
     *
     * @param uploadOption   上传配置
     * @param compressOption 压缩配置
     * @param uploadCallback 上传回调
     */
    void insertImages(
            LinghitRichEditorUploadOption uploadOption,
            LinghitRichEditorImageCompressOption compressOption,
            LinghitRichEditorImageCompressCallback uploadCallback);

    /**
     * 批量取消上传任务
     *
     * @param uploadTag 上传时，指定的Tag
     */
    void cancelUploadFileTasks(Object... uploadTag);

    /**
     * 批量取消压缩图片任务
     *
     * @param imageCompressTag 图片压缩时指定的Tag，如果是上传图片，则需要传
     */
    void cancelCompressImageTasks(String... imageCompressTag);

    /**
     * 撤销
     */
    void undo();

    /**
     * 取消撤销
     */
    void redo();

    /**
     * 清除所有内容
     */
    void clearAll();

    //----------> 一些基础方法 <----------

    /**
     * 设置要加载的Html，一般用于后台返回Html后，重新渲染
     *
     * @param contents 后台返回的Html
     */
    void setHtml(String contents);

    /**
     * 获取正在展示的Html
     */
    void getHtml(OnMethodResultCallback methodResultCallback);

    /**
     * 获取指定位置插入的图片的Url
     *
     * @param position 位置
     */
    void getInsertImgUrl(int position, OnMethodResultCallback callback);

    //----------> 回调接口 <----------

    interface AfterInitialLoadListener {
        /**
         * 加载完成的回调
         *
         * @param isReady 是否已经准备好了
         */
        void onAfterInitialLoad(boolean isReady);
    }

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

    /**
     * 光标移动时回调
     */
    interface OnCursorMoveListener {
        /**
         * 光标移动时回调
         *
         * @param isBold       是否为粗体
         * @param isItalic     是否为斜体
         * @param isUnderline  是否为下划线
         * @param isDeleteLine 是否为删除线
         * @param currentFont  当前Font
         */
        void onCursorMove(boolean isBold, boolean isItalic, boolean isUnderline, boolean isDeleteLine, String currentFont);
    }

    /**
     * 字体改变的监听
     */
    interface OnTitleSizeChangeListener {
        /**
         * 当TitleSize发生改变时回调
         *
         * @param beforeTitleSize    之前设置的字体大小
         * @param currentTitleSize   当前设置的字体大小
         * @param isDefaultTitleSize 是否是默认的字体大小
         */
        void onTitleSizeChange(TitleSizeEnum beforeTitleSize, TitleSizeEnum currentTitleSize, boolean isDefaultTitleSize);
    }

    /**
     * 设置WebView初始化加载的监听
     */
    void setOnInitialLoadListener(AfterInitialLoadListener listener);

    /**
     * 是否注册了初始化加载的监听
     *
     * @param listener 监听器
     */
    boolean isRegisterInitialLoadListener(AfterInitialLoadListener listener);

    /**
     * 设置光标移动的监听
     */
    void setOnCursorMoveListener(OnCursorMoveListener onCursorMoveListener);

    boolean isRegisterOnCursorMoveListener(OnCursorMoveListener onCursorMoveListener);

    /**
     * 设置字体大小改变的监听
     */
    void setOnTitleSizeChangeListener(OnTitleSizeChangeListener onTitleSizeChangeListener);
}
```

标题文字大小枚举

```
public enum TitleSizeEnum {
    /**
     * 标题文字大小枚举
     */
    H1("h1"),
    H2("h2"),
    H3("h3"),
    H4("h4"),
    H5("h5"),
    /**
     * H0代表原始大小
     */
    H0("h0"),;

    private String mSize;

    TitleSizeEnum(String size) {
        mSize = size;
    }

    public String getSize() {
        return mSize;
    }

    @Override
    public String toString() {
        return mSize;
    }
}
```

操作的种类，粗体、斜体、删除线等

```
public enum  TypeEnum {
    /**
     * 操作的种类，粗体、斜体、删除线等
     */
    BOLD("bold"),
    ITALIC("italic"),
    UNDERLINE("underline"),
    DELETE_LINE("deleteline"),
    CURRENT_FONT("currentFont");

    private String mName;

    TypeEnum(String name) {
        this.mName = name;
    }

    public String getName() {
        return mName;
    }

    @Override
    public String toString() {
        return this.mName;
    }
}
```

2. 上传、压缩参数（为了参数的拓展性，使用了Builder构建者模式）

图片上传参数

```
public class LinghitRichEditorUploadOption {
    private String mUploadUrl;
    private Map<String, Object> mHeaderMap;
    private Map<String, Object> mParamsMap;
    private String mFileKey;
    private List<File> mFiles;
    private Object mTag;

    private LinghitRichEditorUploadOption(Builder builder) {
        this.mUploadUrl = builder.mUploadUrl;
        this.mHeaderMap = builder.mHeaderMap == null ? new HashMap<String, Object>() : builder.mHeaderMap;
        this.mParamsMap = builder.mParamsMap == null ? new HashMap<String, Object>() : builder.mParamsMap;
        this.mFileKey = builder.mFileKey;
        this.mFiles = builder.mFiles == null ? new ArrayList<File>() : builder.mFiles;
        this.mTag = builder.mTag;
    }

    //...省略get方法

    public static class Builder {
        /**
         * 上传地址Url
         */
        private String mUploadUrl;
        /**
         * 上传时需要带的Header参数，键值对，一般上传会带token验证
         */
        private Map<String, Object> mHeaderMap;
        /**
         * 上传时需要带的字段参数，键值对
         */
        private Map<String, Object> mParamsMap;
        /**
         * 文件上传Key，这里必须是统一的
         */
        private String mFileKey;
        /**
         * 需要上传的文件
         */
        private List<File> mFiles;
        /**
         * 本次上传任务的Tag，需要手动取消上传任务时需要使用Tag来标记
         */
        private Object mTag;

        private Builder() {
        }

        /**
         * 单个文件时，使用该重载
         */
        public static Builder newBuilder(String uploadUrl, String fileKey, File file) {
            ArrayList<File> files = new ArrayList<>();
            files.add(file);
            return new Builder(uploadUrl, fileKey, files);
        }

        /**
         * 同一个Key，多个File文件时，使用该重载
         */
        public static Builder newBuilder(String uploadUrl, String fileKey, File... files) {
            return new Builder(uploadUrl, fileKey, Arrays.asList(files));
        }

        /**
         * 同一个Key，多个File文件的Path
         */
        public static Builder newBuilder(String uploadUrl, String fileKey, String... filePaths) {
            ArrayList<File> files = new ArrayList<>();
            for (String filePath : filePaths) {
                files.add(new File(filePath));
            }
            return new Builder(uploadUrl, fileKey, files);
        }

        /**
         * 后续再追加File参数，也是可以的
         */
        public Builder addFile(String fileKey, File file) {
            this.mFileKey = fileKey;
            if (this.mFiles == null) {
                this.mFiles = new ArrayList<>();
            }
            mFiles.add(file);
            return this;
        }

        public Builder addFile(String fileKey, File... files) {
            this.mFileKey = fileKey;
            if (this.mFiles == null) {
                this.mFiles = new ArrayList<>();
            }
            mFiles.addAll(Arrays.asList(files));
            return this;
        }

        public Builder setTag(Object tag) {
            this.mTag = tag;
            return this;
        }

        public Builder setHeaderMap(Map<String, Object> headerMap) {
            this.mHeaderMap = headerMap;
            return this;
        }

        public Builder addHeader(String key, Object value) {
            if (this.mHeaderMap == null) {
                this.mHeaderMap = new HashMap<>();
            }
            mHeaderMap.put(key, value);
            return this;
        }

        public Builder addHeader(Map<String, Object> headerMap) {
            if (this.mHeaderMap == null) {
                this.mHeaderMap = new HashMap<>();
            }
            mHeaderMap.putAll(headerMap);
            return this;
        }

        public Builder setParamsMap(Map<String, Object> paramsMap) {
            this.mParamsMap = paramsMap;
            return this;
        }

        public Builder addParams(String key, Object value) {
            if (this.mParamsMap == null) {
                this.mParamsMap = new HashMap<>();
            }
            mParamsMap.put(key, value);
            return this;
        }

        public Builder addParams(Map<String, Object> paramsMap) {
            if (this.mParamsMap == null) {
                this.mParamsMap = new HashMap<>();
            }
            mParamsMap.putAll(paramsMap);
            return this;
        }

        public Builder setUploadUrl(String uploadUrl) {
            mUploadUrl = uploadUrl;
            return this;
        }

        public Builder setFilesMap(String key, List<File> files) {
            this.mFiles = files;
            return this;
        }

        /**
         * 必须的3个参数
         *
         * @param uploadUrl 服务器上传的Url
         * @param fileKey   文件key
         * @param files     文件键值对
         */
        public Builder(String uploadUrl, String fileKey, List<File> files) {
            this.mUploadUrl = uploadUrl;
            this.mFileKey = fileKey;
            this.mFiles = files;
        }

        public LinghitRichEditorUploadOption build() {
            return new LinghitRichEditorUploadOption(this);
        }
    }
}
```
压缩参数

```
public class LinghitRichEditorImageCompressOption {
    private boolean isOrigin;
    private Bitmap.Config mBitmapConfig;
    private File mDiskDirectory;
    private int mCompressTaskNum;
    private Context mContext;
    private String mTag;

    public LinghitRichEditorImageCompressOption(Builder builder) {
        this.isOrigin = builder.isOrigin;
        this.mBitmapConfig = builder.mBitmapConfig == null ? Bitmap.Config.ARGB_8888 : builder.mBitmapConfig;
        this.mDiskDirectory = builder.mDiskDirectory == null ? LinghitRichEditorUtil.getAppCacheDirFile() : builder.mDiskDirectory;
        this.mCompressTaskNum = builder.mCompressTaskNum;
        this.mContext = builder.mContext == null ? LinghitRichEditorUtil.getApplicationContext() : builder.mContext;
        this.mTag = builder.mTag;
    }

    //...省略get方法

    public static class Builder {
        /**
         * 是否是原图，这里一般用于图片，是否上传原图
         * 如果为true，则不进行压缩，直接上传原文件
         */
        private boolean isOrigin;
        /**
         * Bitmap的色彩格式
         */
        private Bitmap.Config mBitmapConfig;
        /**
         * 压缩后的文件存放地址，一般都指定为App的Cache目录，如果需要放在别处，指定该参数
         */
        private File mDiskDirectory;
        /**
         * 可同时进行压缩的任务数量
         */
        private int mCompressTaskNum;
        /**
         * 这里最好传Activity，图片压缩会绑定Activity生命周期，如果在非Activity界面，无法传，则需要手动终止任务
         */
        private Context mContext;
        /**
         * 本次任务的Tag，需要手动取消压缩任务时，使用该Tag去取消
         */
        private String mTag;

        private Builder() {
        }

        public static Builder newBuilder() {
            return new Builder();
        }

        public Builder setIsOrigin(boolean isOrigin) {
            this.isOrigin = isOrigin;
            return this;
        }

        public Builder setBitmapConfig(Bitmap.Config config) {
            this.mBitmapConfig = config;
            return this;
        }

        public Builder setDiskDirectory(File diskDirectory) {
            this.mDiskDirectory = diskDirectory;
            return this;
        }

        public Builder setCompressTaskNum(int compressTaskNum) {
            mCompressTaskNum = compressTaskNum;
            return this;
        }

        public Builder setContext(Context context) {
            mContext = context;
            return this;
        }

        public Builder setTag(String tag) {
            mTag = tag;
            return this;
        }

        public LinghitRichEditorImageCompressOption build() {
            return new LinghitRichEditorImageCompressOption(this);
        }
    }
}
```

3. 图片上传和压缩的回调接口（2个都是异步操作，需要压缩完毕后再上传，所以接口回调嵌套在所难免，当时对RxJava并不熟悉，也不想引用太多的库，所以没有采用，如果后续需要更新，就会采用RxJava对异步串联进行重构）

图片上传回调接口

```
public interface LinghitRichEditorUploadCallback {
    /**
     * 当准备上传时回调
     *
     * @param file File对象
     * @return 需要返回对应的该次请求的Option对象
     */
    void onPrepareUpload(File file);

    /**
     * 开始上传时回调
     *
     * @param tag 任务的Tag
     */
    void onStartUpload(Object tag);

    /**
     * 上传成功的回调
     *
     * @param result 请求的结果
     */
    void onUploadSuccess(Object tag, String result);

    /**
     * 上传失败
     *
     * @param error 异常原因
     */
    void onUploadFail(Object tag, Throwable error);

    /**
     * 上传结束时回调，不管成功还是失败
     */
    void onUploadFinish(Object tag);

    /**
     * 上传进度更新时回调
     *
     * @param progress 进度对象
     * @param percent  当前进度百分比
     */
    void onUploadProgressUpdate(Object tag, Progress progress, float percent);
}
```

图片压缩回调接口

```
public interface LinghitRichEditorImageCompressCallback extends LinghitRichEditorUploadCallback {
    /**
     * 准备压缩时回调
     */
    void onPrepareCompress();

    /**
     * 压缩成功的回调
     *
     * @param tag 压缩任务的Tag
     */
    void onCompressSuccess(String tag);

    /**
     * 压缩失败
     */
    void onCompressFail(String tag);

    /**
     * 压缩结束的回调，成功、失败都会回调
     *
     * @param isSuccess 是否压缩成功
     */
    void onCompressFinish(String tag, boolean isSuccess);
}
```

4. 富文本实现基类（模板模式，限定流程和提供基础方法），由于实现类中需要注入WebView，实现类又是单例，所以为了确保不内存泄露，提供了destory()方法提供到外部调用，其实并不优雅，调用者容易忘记调用。好的做法是自动绑定生命周期，调用方无需知道也无需处理。当时也是不想引入太多的库，后续重构会加入Google提供的AAC组件中的Lifecycle组件管理生命周期。

	- 由于我们需要调用JS函数并获取返回值，在4.4以上提供了mWebView.evaluateJavascript("method", new ValueCallback)的方法来接收回调，但是4.4以下是没有Api提供获取返回值的！
	- 解决方案是：JS函数套一层alert()，alert就是弹窗，将返回值通过alert弹窗，再通过WebView的WebChromeClient，复写onJsAlert回调，拦截掉，返回值内容则是onJsAlert()回调函数的message参数回传。

```
public abstract class BaseLinghitRichEditor implements ILinghitRichEditor {
    /**
     * 依赖的WebView
     */
    private WebView mWebView;
    /**
     * 是否已经准备好了
     */
    private boolean isReady = false;
    /**
     * 兼容4.4调用Js方法无返回值使用的Map映射
     */
    private HashMap<String, OnMethodResultCallback> mCompatKitkatJsMethodCallbackMap;
    private AfterInitialLoadListener mInitialLoadListener;
    private OnCursorMoveListener mOnCursorMoveListener;
    /**
     * 存放上传任务的Tag
     */
    private ArrayList<Object> mUploadTaskTags;
    /**
     * 存放图片压缩的Tag
     */
    private ArrayList<String> mImageCompressTags;
    /**
     * 当前字体设置的字体大小
     */
    private TitleSizeEnum mCurrentTitleSizeEnum = TitleSizeEnum.H0;
    private OnTitleSizeChangeListener mOnTitleSizeChangeListener;

    @Override
    @SuppressLint("SetJavaScriptEnabled")
    public void attachWebView(WebView webView) {
        this.mWebView = webView;
        attachActivity((Activity) webView.getContext());
        if (mCompatKitkatJsMethodCallbackMap == null) {
            this.mCompatKitkatJsMethodCallbackMap = new HashMap<>(16);
        }
        if (mUploadTaskTags == null) {
            mUploadTaskTags = new ArrayList<>();
        }
        if (mImageCompressTags == null) {
            mImageCompressTags = new ArrayList<>();
        }
        //对WebView进行一些设置
        mWebView.setVerticalScrollBarEnabled(false);
        mWebView.setHorizontalScrollBarEnabled(false);
        WebSettings settings = mWebView.getSettings();
        //支持JS通信
        settings.setJavaScriptEnabled(true);
        //设置编码格式
        settings.setDefaultTextEncodingName("utf-8");
        //通知子类复写，子类的设置大于父类的
        onInitWebViewWebSettings(settings);
        mWebView.setWebChromeClient(onCreateWebChromeClient());
        mWebView.setWebViewClient(onCreateWebViewClient());
        //加载index.html
        mWebView.loadUrl(getLoadHtmlPath());
        LinghitRichEditorAgent.Callback callback = LinghitRichEditorAgent
                .getInstance()
                .getCallback();
        //允许Chrome远程调试
        if (callback != null) {
            boolean isCanDebugWebView = callback.isCanDebugWebView();
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
                WebView.setWebContentsDebuggingEnabled(isCanDebugWebView);
            }
        }
    }

    /**
     * 依附到Activity时回调
     */
    protected void attachActivity(Activity activity) {
    }

    @Override
    public void dispatchDestroy() {
        //取消任务
        if (mUploadTaskTags != null) {
            for (Object tag : mUploadTaskTags) {
                MMCHttp.getInstance().cancelTag(tag);
            }
        }
        if (mImageCompressTags != null) {
            for (String tag : mImageCompressTags) {
                Flora.cancel(tag);
            }
        }
        //销毁WebView
        if (mWebView != null) {
            mWebView.destroy();
            ViewParent parentLayout = mWebView.getParent();
            if (parentLayout instanceof ViewGroup) {
                ((ViewGroup) parentLayout).removeView(mWebView);
            }
            mWebView = null;
        }
    }

    /**
     * 设置字体大小后，需要再setTitleSize()或者setDefaultTitleSize()后调用
     *
     * @param isSuccess 是否设置成功
     * @param sizeEnum  设置成功的字体大小枚举
     */
    protected void setTitleSizeAfter(boolean isSuccess, TitleSizeEnum sizeEnum) {
        //这里去保存字体大小
        if (isSuccess) {
            //记录下之前的
            TitleSizeEnum beforeTitleSizeEnum = mCurrentTitleSizeEnum;
            //更新当前的
            this.mCurrentTitleSizeEnum = sizeEnum;
            if (mOnTitleSizeChangeListener != null) {
                mOnTitleSizeChangeListener.onTitleSizeChange(
                        beforeTitleSizeEnum
                        , mCurrentTitleSizeEnum,
                        mCurrentTitleSizeEnum == TitleSizeEnum.H0);
            }
        }
    }

    @Override
    public TitleSizeEnum getCurrentTitleSize() {
        return this.mCurrentTitleSizeEnum;
    }

    @Override
    public void insertImages(final LinghitRichEditorUploadOption uploadOption,
                             LinghitRichEditorImageCompressOption compressOption,
                             final LinghitRichEditorImageCompressCallback uploadCallback) {
        //上传图片，再JS调用，在WebView中显示
        if (uploadOption != null) {
            final String fileKey = uploadOption.getFileKey();
            List<File> files = uploadOption.getFiles();
            //这里压缩图片，选择原图则所有都是原图，不会个别原图，个别压缩
            if (compressOption == null) {
                //默认压缩参数
                compressOption = LinghitRichEditorImageCompressOption
                        .Builder
                        .newBuilder()
                        .setBitmapConfig(Bitmap.Config.ARGB_4444)
                        //默认，多少张图片就多少个线程进行压缩
                        .setCompressTaskNum(files.size())
                        .setIsOrigin(false)
                        .build();
            }
            if (!compressOption.isOrigin()) {
                //如果是需要压缩的，进行压缩处理
                uploadCallback.onPrepareCompress();
                final String tag = compressOption.getTag();
                Context context = compressOption.getContext();
                CompressTaskBuilder compressTaskBuilder;
                if (context instanceof Activity) {
                    compressTaskBuilder = Flora.with((Activity) context);
                } else {
                    compressTaskBuilder = Flora.with(tag);
                    mImageCompressTags.add(tag);
                }
                compressTaskBuilder
                        .bitmapConfig(compressOption.getBitmapConfig())
                        .compressTaskNum(compressOption.getCompressTaskNum())
                        .diskDirectory(compressOption.getDiskDirectory())
                        .load(files)
                        .compress(new Callback<List<String>>() {
                            @Override
                            public void callback(List<String> paths) {
                                uploadCallback.onCompressSuccess(tag);
                                uploadCallback.onCompressFinish(tag, true);
                                //压缩成功后，再上传
                                ArrayList<File> files = new ArrayList<>();
                                for (String path : paths) {
                                    File imageFile = new File(path);
                                    files.add(imageFile);
                                }
                                uploadFiles(uploadOption, fileKey, files, uploadCallback);
                            }

                            @Override
                            public void noData() {
                                //压缩失败
                                uploadCallback.onCompressFail(tag);
                                uploadCallback.onCompressFinish(tag, false);
                            }
                        });
            } else {
                //原图上传
                List<File> images = uploadOption.getFiles();
                uploadFiles(uploadOption, fileKey, images, uploadCallback);
            }
        } else {
            throw new NullPointerException("LinghitRichEditorUploadOption 不能为空！");
        }
    }

    @Override
    public void cancelUploadFileTasks(Object... uploadTag) {
        //取消上传任务
        if (uploadTag != null) {
            for (Object tag : uploadTag) {
                MMCHttp.getInstance().cancelTag(tag);
            }
        }
    }

    @Override
    public void cancelCompressImageTasks(String... imageCompressTag) {
        //取消图片压缩任务
        if (imageCompressTag != null) {
            for (String tag : imageCompressTag) {
                Flora.cancel(tag);
            }
        }
    }

    /**
     * 批量上传文件
     *
     * @param files          文件对象数组
     * @param uploadCallback 回调对象
     */
    private void uploadFiles(LinghitRichEditorUploadOption uploadOption,
                             String fileKey,
                             List<File> files,
                             final LinghitRichEditorUploadCallback uploadCallback) {
        if (uploadCallback == null) {
            throw new NullPointerException("LinghitRichEditorImageCompressCallback不能为空！");
        }
        for (File file : files) {
            uploadCallback.onPrepareUpload(file);
            final Object tag = uploadOption.getTag();
            if (tag != null) {
                mUploadTaskTags.add(tag);
            }
            String uploadUrl = uploadOption.getUploadUrl();
            Map<String, Object> headerMap = uploadOption.getHeaderMap();
            //组装Header
            HttpHeaders httpHeaders = new HttpHeaders();
            for (Map.Entry<String, Object> entry : headerMap.entrySet()) {
                httpHeaders.put(entry.getKey(), String.valueOf(entry.getValue()));
            }
            //组装参数
            Map<String, Object> paramsMap = uploadOption.getParamsMap();
            HttpParams httpParams = new HttpParams();
            for (Map.Entry<String, Object> entry : paramsMap.entrySet()) {
                httpParams.put(entry.getKey(), String.valueOf(entry.getValue()));
            }
            //开始上传图片
            PostRequest<String> uploadRequest = MMCHttp.<String>post(uploadUrl)
                    .tag(tag)
                    .headers(httpHeaders)
                    .params(httpParams);
            ArrayList<File> fileList = new ArrayList<>();
            fileList.add(file);
            uploadRequest.addFileParams(fileKey, fileList);
            uploadRequest.execute(new StringCallback() {
                @Override
                public void onStart(Request<String, ? extends Request> request) {
                    uploadCallback.onStartUpload(request.getTag());
                }

                @Override
                public void onSuccess(Response<String> response) {
                    uploadCallback.onUploadSuccess(tag, response.body());
                }

                @Override
                public void onError(Response<String> response) {
                    uploadCallback.onUploadFail(tag, response.getException());
                }

                @Override
                public void onFinish() {
                    uploadCallback.onUploadFinish(tag);
                }

                @Override
                public void uploadProgress(Progress progress) {
                    uploadCallback.onUploadProgressUpdate(tag, progress, progress.fraction * 100);
                }
            });
        }
    }

    @Override
    public void getInsertImgUrl(int position, final OnMethodResultCallback callback) {
        exec("document.getElementsByTagName('img')[" + position + "].getAttribute('src')"
                , new OnMethodResultCallback() {
                    @Override
                    public void onMethodResult(String result) {
                        if (callback != null) {
                            handlerMethodResult(callback, result);
                        }
                    }
                });
    }

    /**
     * 获取需要加载的Html页面路径，一般为asset文件下Html文件
     * 一般格式为：file:///android_asset/xxx.html
     */
    protected abstract String getLoadHtmlPath();

    /**
     * 需要创建WebViewClient时回调，EditorWebViewClient已经写了提供拦截的代码，一般不需要重写，如果有需求再重写
     */
    protected WebViewClient onCreateWebViewClient() {
        return new WebViewClient() {

            @Override
            public void onPageFinished(WebView view, String url) {
                mWebViewClientDelegate.setHostWebViewClient(this);
                mWebViewClientDelegate.onPageFinished(view, url);
            }

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                mWebViewClientDelegate.setHostWebViewClient(this);
                return mWebViewClientDelegate.shouldOverrideUrlLoading(view, url);
            }
        };
    }

    /**
     * 需要创建WebChromeClient时回调
     */
    protected WebChromeClient onCreateWebChromeClient() {
        return new WebChromeClient() {
            @Override
            public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
                super.onJsAlert(view, url, message, result);
                return mWebChromeClientDelegate.onJsAlert(view, url, message, result);
            }
        };
    }

    /**
     * 当拦截Url加载时回调，一般需要实现类去做前端传递的数据拦截
     *
     * @return 是否拦截了
     */
    protected abstract boolean onInterceptUrlLoading(WebView webView, String url);

    /**
     * 当设置WebSettings时回调，子类复写该方法进行更多设置，可以不处理，需要则复写
     */
    protected void onInitWebViewWebSettings(WebSettings webSettings) {
    }

    protected void exec(final String trigger) {
        exec(trigger, null);
    }

    /**
     * 执行调用JS代码
     *
     * @param trigger 调用js的格式代码
     */
    protected void exec(final String trigger, final OnMethodResultCallback resultCallback) {
        if (isReady) {
            load(trigger, resultCallback);
        } else {
            mWebView.postDelayed(new Runnable() {
                @Override
                public void run() {
                    exec(trigger, resultCallback);
                }
            }, 100);
        }
    }

    /**
     * 调用Js方法，兼容4.4一下和4.4以上
     *
     * @param trigger        调用js的格式代码
     * @param resultCallback 函数返回值监听
     */
    private void load(String trigger, final OnMethodResultCallback resultCallback) {
        //4.4以上直接调用
        if (isOvertopKitkat()) {
            mWebView.evaluateJavascript(trigger, new ValueCallback<String>() {
                @Override
                public void onReceiveValue(String value) {
                    handlerMethodResult(resultCallback, value);
                }
            });
        } else {
            mCompatKitkatJsMethodCallbackMap.put(getLoadHtmlPath(), resultCallback);
            mWebView.loadUrl(trigger);
        }
    }

    /**
     * 版本号是否大于等于4.4
     */
    protected boolean isOvertopKitkat() {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT;
    }

    //----------> 设置给WebView的回调等 <----------
    /**
     * 将2个Client的方法抽到代理对象中
     */
    protected RichEditorWebChromeClientDelegate mWebChromeClientDelegate = new RichEditorWebChromeClientDelegate();
    protected EditorWebViewClientDelegate mWebViewClientDelegate = new EditorWebViewClientDelegate();

    /**
     * WebChromeClient里的方法代理，不强制继承WebChromeClient
     */
    public class RichEditorWebChromeClientDelegate {
        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
            //回调4.4一下的Js返回值处理
            OnMethodResultCallback callback = mCompatKitkatJsMethodCallbackMap.get(url);
            if (callback != null) {
                handlerMethodResult(callback, message);
            }
            result.confirm();
            return true;
        }
    }

    /**
     * WebViewClient方法代理
     */
    public class EditorWebViewClientDelegate {
        private WebViewClient mHostWebViewClient;

        public void setHostWebViewClient(WebViewClient webViewClient) {
            this.mHostWebViewClient = webViewClient;
        }

        public void onPageFinished(WebView view, String url) {
            isReady = url.equalsIgnoreCase(getLoadHtmlPath());
            if (mInitialLoadListener != null) {
                mInitialLoadListener.onAfterInitialLoad(isReady);
            }
        }

        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            //如果是JS那边的回调，拦截掉，接口回调到外层
            boolean isIntercept = onInterceptUrlLoading(view, url);
            if (isIntercept) {
                return true;
            } else {
                return mHostWebViewClient.shouldOverrideUrlLoading(view, url);
            }
        }
    }

    /**
     * 处理JS返回值，因为有一些字符、转义符可能会存在，心累~还是前端回调安卓方法少坑
     *
     * @param callback 回调
     * @param value    JS返回值
     */
    private void handlerMethodResult(OnMethodResultCallback callback, String value) {
        if (TextUtils.isEmpty(value) || "\"\"".equals(value)) {
            value = "";
        } else {
            //如果前后有双引号，裁剪掉前后的双引号
            if (value.startsWith("\"") && value.endsWith("\"")) {
                value = value.substring(1, value.length() - 1);
            }
        }
        //该回调可能会将<符号转为Unicode编码，需要转换
        value = CharsetUtil.fixUnicodeStr(value);
        //去掉转义符
        value = value.replaceAll("\\\\", "");
        //JS回调，如果是null，java层会为"null"，我们替换为空字符串
        if ("null".equals(value)) {
            value = "";
        }
        if (callback != null) {
            callback.onMethodResult(value);
        }
    }

    @Override
    public void setOnInitialLoadListener(AfterInitialLoadListener initialLoadListener) {
        this.mInitialLoadListener = initialLoadListener;
    }

    @Override
    public boolean isRegisterInitialLoadListener(AfterInitialLoadListener initialLoadListener) {
        return this.mInitialLoadListener == initialLoadListener;
    }

    @Override
    public void setOnCursorMoveListener(OnCursorMoveListener onCursorMoveListener) {
        this.mOnCursorMoveListener = onCursorMoveListener;
    }

    @Override
    public boolean isRegisterOnCursorMoveListener(OnCursorMoveListener onCursorMoveListener) {
        return this.mOnCursorMoveListener == onCursorMoveListener;
    }

    @Override
    public void setOnTitleSizeChangeListener(OnTitleSizeChangeListener onTitleSizeChangeListener) {
        this.mOnTitleSizeChangeListener = onTitleSizeChangeListener;
    }

    @Override
    public boolean isRegisterOnTitleSizeChangeListener(OnTitleSizeChangeListener onTitleSizeChangeListener) {
        return this.mOnTitleSizeChangeListener == onTitleSizeChangeListener;
    }

    public AfterInitialLoadListener getInitialLoadListener() {
        return mInitialLoadListener;
    }

    public OnCursorMoveListener getOnCursorMoveListener() {
        return mOnCursorMoveListener;
    }
}
```

5. 提供一个默认实现，由于将很多都抽取到了BaseLinghitRichEditor基类，实现类的代码就非常少，而且代码也更加注重实现之间的差异。

```
public class LinghitRichEditorImpl extends BaseLinghitRichEditor {
    /**
     * 加载的Html位置
     */
    private static final String SETUP_HTML = "file:///android_asset/index.html";
    /**
     * 前端库回调的前缀
     */
    private static final String CALLBACK_SCHEME = "status://status";

    @Override
    public void setTitleSize(TitleSizeEnum sizeEnum) {
        String fontSize = sizeEnum.getSize();
        exec("javascript:titleSize('" + fontSize + "');");
        setTitleSizeAfter(true, sizeEnum);
    }

    @Override
    public void setDefaultTitleSize() {
        TitleSizeEnum sizeEnum = TitleSizeEnum.H0;
        setTitleSize(sizeEnum);
    }

    @Override
    public void setFrameSize(int width, int height) {
        exec("javascript:size('" + width + "'" + ",'" + height + "');");
    }

    @Override
    public void setTextPlaceholder(String placeholder) {
        if (TextUtils.isEmpty(placeholder)) {
            placeholder = "";
        }
        exec("javascript:placeholder('" + placeholder + "');");
    }

    @Override
    public void toggleBold() {
        exec("javascript:bold();");
    }

    @Override
    public void toggleItalic() {
        exec("javascript:italic();");
    }

    @Override
    public void toggleUnderline() {
        exec("javascript:underline();");
    }

    @Override
    public void addDivider() {
        exec("javascript:hr();");
    }

    @Override
    public void toggleDeleteLine() {
        exec("javascript:deleteline();");
    }

    @Override
    public void insertImages(String... imgUrls) {
        for (String imgUrl : imgUrls) {
            exec("javascript:insertImage('" + imgUrl + "');");
        }
    }

    @Override
    public void undo() {
        exec("javascript:undo();");
    }

    @Override
    public void redo() {
        exec("javascript:redo();");
    }

    @Override
    public void clearAll() {
        setHtml("");
    }

    //----------> 一些基础方法 <----------

    @Override
    public void setHtml(String contents) {
        if (contents == null) {
            contents = "";
        }
        exec("javascript:setValue('" + contents + "');");
    }

    @Override
    public void getHtml(final ILinghitRichEditor.OnMethodResultCallback methodResultCallback) {
        String trigger;
        if (isOvertopKitkat()) {
            trigger = "javascript:getValue();";
        } else {
            trigger = "javascript:alert(getValue());";
        }
        exec(trigger, methodResultCallback);
    }

    @Override
    protected String getLoadHtmlPath() {
        return SETUP_HTML;
    }

    @Override
    protected boolean onInterceptUrlLoading(WebView webView, String url) {
        //移动光标时网页会调用
        if (TextUtils.indexOf(url, CALLBACK_SCHEME) == 0) {
            callMoveCallback(url);
            return true;
        } else {
            return false;
        }
    }

    /**
     * 移动光标时，JS回调客户端
     *
     * @param text 网页回传的数据
     */
    private void callMoveCallback(String text) {
        //拆分参数，原始格式为：
        //status://status?bold=false&italic=false&underline=false&deleteline=false&currentFont=text
        //需替换前缀：status://status?
        HashMap<String, String> paramsMap = new HashMap<>(5);
        //注意：?号这里需要转移！
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
        //分别是：粗体、斜体、下划线、删除线、当前大小
        boolean isBold = Boolean.valueOf(paramsMap.get(TypeEnum.BOLD.getName()));
        boolean isItalic = Boolean.valueOf(paramsMap.get(TypeEnum.ITALIC.getName()));
        boolean isUnderline = Boolean.valueOf(paramsMap.get(TypeEnum.UNDERLINE.getName()));
        boolean isDeleteLine = Boolean.valueOf(paramsMap.get(TypeEnum.DELETE_LINE.getName()));
        String currentFont = paramsMap.get(TypeEnum.CURRENT_FONT.getName());
        if (getOnCursorMoveListener() != null) {
            getOnCursorMoveListener().onCursorMove(isBold, isItalic, isUnderline, isDeleteLine, currentFont);
        }
    }
}
```

6. 外部管理类，提供注册、替换实现

```
public class LinghitRichEditorAgent {
    private ILinghitRichEditor mEditor;
    private Context mApplicationContext;
    private Callback mCallback;

    private LinghitRichEditorAgent() {
    }

    private static final class SingleHolder {
        private static final LinghitRichEditorAgent INSTANCE = new LinghitRichEditorAgent();
    }

    public static LinghitRichEditorAgent getInstance() {
        return SingleHolder.INSTANCE;
    }

    /**
     * 提供给外部使用的注册
     */
    public void register(Context context, Callback callback) {
        register(context, null, callback);
    }

    /**
     * 如果App依赖方需要自定义前端库实现对象，则调用该方法
     *
     * @param editor 前端库实现对象
     */
    public void register(Context context, ILinghitRichEditor editor, Callback callback) {
        this.mApplicationContext = context.getApplicationContext();
        this.mCallback = callback;
        if (editor == null) {
            editor = new LinghitRichEditorImpl();
        }
        this.mEditor = editor;
    }

    /**
     * 更换实现
     */
    public void replaceImpl(ILinghitRichEditor editor) {
        this.mEditor = editor;
    }

    public Context getApplicationContext() {
        return mApplicationContext;
    }

    public ILinghitRichEditor getEditor() {
        return mEditor;
    }

    public Callback getCallback() {
        return mCallback;
    }

    public interface Callback {
        /**
         * 是否可以Debug设备的WebView
         */
        boolean isCanDebugWebView();
    }
}
```

7. 为了显示可替换实现，例子里引入了一个开源库richeditor-android（将库中的assets文件夹的文件拷贝到项目中即可）。提供一个JPRichEditorImpl。

```
public class JPRichEditorImpl extends BaseLinghitRichEditor {
    private static final String SETUP_HTML = "file:///android_asset/editor.html";
    private static final String CALLBACK_SCHEME = "re-callback://";
    private static final String STATE_SCHEME = "re-state://";

    public enum Type {
        BOLD,
        ITALIC,
        SUBSCRIPT,
        SUPERSCRIPT,
        STRIKETHROUGH,
        UNDERLINE,
        H1,
        H2,
        H3,
        H4,
        H5,
        H6,
        ORDEREDLIST,
        UNORDEREDLIST,
        JUSTIFYCENTER,
        JUSTIFYFULL,
        JUSTUFYLEFT,
        JUSTIFYRIGHT
    }

    private String mContents;

    private void callback(String text) {
        mContents = text.replaceFirst(CALLBACK_SCHEME, "");
    }

    @Override
    protected String getLoadHtmlPath() {
        return SETUP_HTML;
    }

    @Override
    protected boolean onInterceptUrlLoading(WebView webView, String url) {
        //JP库是文字改变时就回调，在url后面加上返回值
        // 不同于simditor库，是通过js方法返回值，JS方法返回值，安卓回调函数中字符被编码过
        String decode;
        try {
            decode = URLDecoder.decode(url, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            // No handling
            return false;
        }
        if (TextUtils.indexOf(decode, CALLBACK_SCHEME) == 0) {
            callback(decode);
            return true;
        } else if (TextUtils.indexOf(decode, STATE_SCHEME) == 0) {
            //因为该开源库做了我们不需要的功能回调，也要拦截掉
            return true;
        }
        return false;
    }

    @Override
    public void setTitleSize(TitleSizeEnum sizeEnum) {
        //因为这个库的H标签参数不一样，这里需要做一个映射
        int heading;
        switch (sizeEnum) {
            case H0:
                //默认大小
                heading = 1;
                break;
            case H1:
                heading = 2;
                break;
            case H2:
                heading = 3;
                break;
            case H3:
                heading = 4;
                break;
            case H4:
                heading = 5;
                break;
            case H5:
                heading = 6;
                break;
            default:
                heading = 1;
                break;
        }
        exec("javascript:RE.setHeading('" + heading + "');");
    }

    @Override
    public void setDefaultTitleSize() {
        setTitleSize(TitleSizeEnum.H0);
    }

    @Override
    public void setFrameSize(int width, int height) {
        exec("javascript:RE.setWidth('" + width + "px');");
        exec("javascript:RE.setHeight('" + height + "px');");
    }

    @Override
    public void setTextPlaceholder(String placeholder) {
        exec("javascript:RE.setPlaceholder('" + placeholder + "');");
    }

    @Override
    public void toggleBold() {
        exec("javascript:RE.setBold();");
    }

    @Override
    public void toggleItalic() {
        exec("javascript:RE.setItalic();");
    }

    @Override
    public void toggleUnderline() {
        exec("javascript:RE.setUnderline();");
    }

    @Override
    public void addDivider() {
        //该库没有提供，空实现
    }

    @Override
    public void toggleDeleteLine() {
        exec("javascript:RE.setStrikeThrough();");
    }

    @Override
    public void insertImages(String... imgUrls) {
        //标题
        String alt = "";
        for (String url : imgUrls) {
            exec("javascript:RE.prepareInsert();");
            exec("javascript:RE.insertImage('" + url + "', '" + alt + "');");
        }
    }

    @Override
    public void undo() {
        exec("javascript:RE.undo();");
    }

    @Override
    public void redo() {
        exec("javascript:RE.redo();");
    }

    @Override
    public void clearAll() {
        setHtml("");
    }

    @Override
    public void setHtml(String contents) {
        if (contents == null) {
            contents = "";
        }
        try {
            exec("javascript:RE.setHtml('" + URLEncoder.encode(contents, "UTF-8") + "');");
        } catch (UnsupportedEncodingException e) {
            // No handling
        }
        mContents = contents;
    }

    @Override
    public void getHtml(OnMethodResultCallback methodResultCallback) {
        methodResultCallback.onMethodResult(mContents);
    }
}
```

8. 由于要使用WebView，所以提供了一个基础注入WebView到Editor实现类的WebView，如果有特殊需求，继承即可，不能继承，拷贝调用代码即可。

```
public class LinghitRichEditorWebView extends WebView {
    private ILinghitRichEditor mEditor;

    public LinghitRichEditorWebView(Context context) {
        this(context, null);
    }

    public LinghitRichEditorWebView(Context context, AttributeSet attrs) {
        this(context, attrs, android.R.attr.webViewStyle);
    }

    public LinghitRichEditorWebView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mEditor = LinghitRichEditorAgent.getInstance().getEditor();
        mEditor.attachWebView(this);
    }

    /**
     * 获取Editor对象，即可调用前端库的功能
     */
    public ILinghitRichEditor getEditor() {
        return mEditor;
    }

    /**
     * 外部使用需要调用该方法通知Activity要销毁，内部做取消任务监听等操作
     */
    public void onActivityDestroy() {
        if (mEditor != null) {
            mEditor.dispatchDestroy();
        }
    }
}
```

9. 在Appplication中注册

```
@Override
public void onCreate() {
    super.onCreate();
    //回调对象，内部需要外部决定一些设置时会回调
    LinghitRichEditorAgent.Callback callback = new LinghitRichEditorAgent.Callback() {
        @Override
        public boolean isCanDebugWebView() {
            return BuildConfig.DEBUG;
        }
    };
    //注册富文本，可指定具体实现
    LinghitRichEditorAgent
            .getInstance()
            .register(this, new JPRichEditorImpl(), callback);
    //如果只是用本库的默认实现，不用指定
    LinghitRichEditorAgent
            .getInstance()
            .register(this, callback);
}
```

10. Activity、Fragment中使用

- 基本使用

```
private void findView(View view) {
		 //粗体
        mBoldCheckBox = view.findViewById(R.id.bold_check_box);
        //斜体
        mItalicCheckBox = view.findViewById(R.id.italic_check_box);
        //删除线
        mDeleteLineCheckBox = view.findViewById(R.id.delete_line_check_box);
        mHRadioGroup = view.findViewById(R.id.h_radio_group);
    }

    private void bindView() {
        //设置默认的提示文字
        mEditorWebView.getEditor().setTextPlaceholder("请输入您的观点：");
    	 //设置粗体切换
        mBoldCheckBox.setOnCheckChangeListener(isChecked -> mEditWebView.getEditor().toggleBold());
        //设置斜体切换
        mItalicCheckBox.setOnCheckChangeListener(isChecked -> mEditWebView.getEditor().toggleItalic());
        //设置删除线切换
        mDeleteLineCheckBox.setOnCheckChangeListener(isChecked -> mEditWebView.getEditor().toggleDeleteLine());
        //设置字体大小切换
        mHRadioGroup.setOnCheckedChangeListener((group, checkedButton, uncheckButton) -> {
            if (checkedButton.getId() == R.id.h1_radio_btn) {
                mEditWebView.getEditor().setTitleSize(TitleSizeEnum.H1);
            } else if (checkedButton.getId() == R.id.h2_radio_btn) {
                mEditWebView.getEditor().setTitleSize(TitleSizeEnum.H2);
            } else if (checkedButton.getId() == R.id.h3_radio_btn) {
                mEditWebView.getEditor().setTitleSize(TitleSizeEnum.H3);
            } else if (checkedButton.getId() == R.id.h4_radio_btn) {
                mEditWebView.getEditor().setTitleSize(TitleSizeEnum.H4);
            } else if (checkedButton.getId() == R.id.h5_radio_btn) {
                mEditWebView.getEditor().setTitleSize(TitleSizeEnum.H5);
            }
        });
        //注册移动光标重新渲染状态回调
        mEditWebView.getEditor().setOnCursorMoveListener((isBold, isItalic, isUnderline, isDeleteLine, currentFont) -> {
            String builder = ("是粗体：" + isBold) +
                    "是斜体：" + isItalic +
                    "是下划线：" + isUnderline +
                    "是删除线：" + isDeleteLine +
                    "是当前Font：" + currentFont;
            L.i(builder);
            mBoldCheckBox.renderCheckStatus(isBold);
            mItalicCheckBox.renderCheckStatus(isItalic);
            mDeleteLineCheckBox.renderCheckStatus(isDeleteLine);
        });
        //回显Html
        mEditorWebView.getEditor().setHtml(content);
        //分割线
        addDividerIv.setOnClickListener(v -> {
            //增加分隔线
            mEditorWebView.getEditor().addDivider();
        });
    }
```

- 图片上传

```
//选择图片上传
final TakePhotoDelegateFragment.SimpleOnTakePhotoCallback callback = new TakePhotoDelegateFragment.SimpleOnTakePhotoCallback() {
            @Override
            public void onTakePhoto(ArrayList<String> imgPaths) {
                super.onTakePhoto(imgPaths);
                //上传多图
                startUploadMultipleImage(imgPaths);
            }
        };

/**
 * 批量上传图片
 */
private void startUploadMultipleImage(List<String> imgPaths) {
    for (int i = 0; i < imgPaths.size(); i++) {
        File file = new File(imgPaths.get(i));
        String filePath = file.getAbsolutePath();
        String fileName = file.getName();
        //文件上传任务的Tag，需要取消任务时使用，一般会添加
        String uploadTag = String.valueOf(System.currentTimeMillis());
        //这里找的第三方接口，url和参数换成你们要的
        String url = ApiUrl.TOPIC_UPLOAD_IMAGE_FILE;
        LinkedHashMap<String, String> headerInMap = HttpConfig.getHeaderInMap(getActivity(), null, url, HttpMethod.POST.toString());
        LinkedHashMap<String, String> paramsInMap = HttpConfig.getParamsInMap();
        LinkedHashMap<String, Object> headers = new LinkedHashMap<>(headerInMap);
        LinkedHashMap<String, Object> params = new LinkedHashMap<>(paramsInMap);
        LinghitRichEditorUploadOption uploadOption = LinghitRichEditorUploadOption.Builder.newBuilder(
                url,
                TopicConstant.Key.KEY_UPLOAD_FILE, filePath)
                .addHeader(headers)
                .addParams(params)
                .addParams(TopicConstant.Key.PARAMS_UPLOAD_FILE_NAME, fileName)
                .setTag(uploadTag)
                .build();
        //压缩参数
        LinghitRichEditorImageCompressOption compressOption = LinghitRichEditorImageCompressOption
                .Builder
                .newBuilder()
                //这里推荐传，会根据生命周期进行终止压缩任务
                .setContext(getActivity())
                //是否原图，一般都需要压缩，如果不需要，传true
                .setIsOrigin(false)
                //同时进行的压缩的个数，一般设置为图片数量
                .setCompressTaskNum(imgPaths.size())
                .setBitmapConfig(Bitmap.Config.ARGB_8888)
                //图片压缩的Tag，如果需要手动取消上传，就需要指定，一般都会指定
                .setTag(uploadTag)
                .build();
        //上传到服务器，拿取图片Url地址
        LinghitRichEditorImageCompressCallback callback = new LinghitRichEditorImageCompressCallback() {

            @Override
            public void onPrepareCompress() {
                //准备压缩
            }

            @Override
            public void onCompressSuccess(String tag) {
                //压缩成功
            }

            @Override
            public void onCompressFail(String tag) {
                //压缩失败
            }

            @Override
            public void onCompressFinish(String tag, boolean isSuccess) {
            }

            @Override
            public void onPrepareUpload(File file) {
            }

            @Override
            public void onStartUpload(Object tag) {
            }

            @Override
            public void onUploadSuccess(Object tag, String result) {
                TypeToken<HttpModel<UploadImageModel>> typeToken = new TypeToken<HttpModel<UploadImageModel>>() {
                };
                HttpModel<UploadImageModel> uploadModel = GsonUtil.parseJson(result, typeToken);
                if (uploadModel != null) {
                    String imgUrl = uploadModel.getData().getImageUrl();
                    mEditorWebView.getEditor().insertImages(imgUrl);
                    ToastUtil.showMsg(getActivity(), getString(R.string.topic_upload_success_tip));
                    L.i("上传成功");
                } else {
                    ToastUtil.showMsg(getActivity(), getString(R.string.topic_upload_fail_tip));
                }
            }

            @Override
            public void onUploadFail(Object tag, Throwable error) {
                ToastUtil.showMsg(getActivity(), getString(R.string.topic_upload_fail_tip));
            }

            @Override
            public void onUploadFinish(Object tag) {
            }

            @Override
            public void onUploadProgressUpdate(Object tag, Progress progress, float percent) {
                L.i("当前上传进度：" + percent + "%");
            }
        };
        mEditorWebView.getEditor().insertImages(uploadOption, compressOption, callback);
    }
}
```

#### 总结

- 优点
	1. 实现可替换
	2. 抽象操作接口，面向接口编程

- 不足
	1. 异步操作回调串联，需要更优雅需要使用RxJava避免回调地域。
	2. 生命周期没有自动绑定，应该使用AAC的Lifecycle改造。