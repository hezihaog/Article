#### 设计模式-Builder模式（构建者模式）

## 为什么要用Builder模式？什么时候用？

- Builder模式一般用于构造一个复杂对象时使用，可以屏蔽构造的细节，Builder模式构造对象和多个参数分开，达到解耦的效果，并且能自由拓展。

#### 怎样实现Builder模式

- 私有构造方法，构造只能由内部的Builder内部类的build方法创建。
- 内部类拷贝目标类的参数，提供set方法，并且每个set方法都返回builder实例自身，链式调用。
- 目标类提供一个私有支持Builder参数构造的构造方法（拷贝属性值到自身），给Builder的builder方法调用，传入自身。
- 构建时的处理，在builder中处理，不一定全是拷贝属性，例如一些默认值的处理、值越界的处理等。

#### Android中Builder模式的实现

- 对话框创建，AlertDialog
- 通知创建，Notification Builder

## 实例

#### Recorder录音配置Option

- 录音的前的一些配置，例如输出路径、文件前缀、文件后缀等属性。

- 使用

```
new RecorderOption.Builder().setOutputDirPath("xxx/yyy/zzz").setFilePrefix("voice").setFileSuffix(".aac").build()
```

- 实现

```
package com.hule.dashi.mediaplayer;

import android.text.TextUtils;

/**
 * <b>Package:</b> com.hule.dashi.mediaplayer <br>
 * <b>Create Date:</b> 2019/2/19  4:04 PM <br>
 * <b>@author:</b> zihe <br>
 * <b>Description:</b> 录音配置 <br>
 */
public class RecorderOption {
    private String outputDirPath;
    private String filePrefix;
    private String fileSuffix;

    private RecorderOption() {
    }

    public RecorderOption(Builder builder) {
        this.outputDirPath = builder.outputDirPath;
        this.filePrefix = TextUtils.isEmpty(builder.filePrefix) ? "voice_" : builder.filePrefix;
        this.fileSuffix = TextUtils.isEmpty(builder.fileSuffix) ? ".aac" : builder.fileSuffix;
    }

    public String getOutputDirPath() {
        return outputDirPath;
    }

    public String getFilePrefix() {
        return filePrefix;
    }

    public String getFileSuffix() {
        return fileSuffix;
    }

    public static class Builder {
        /**
         * 输出目录路径
         */
        private String outputDirPath;
        /**
         * 文件前缀
         */
        private String filePrefix;
        /**
         * 文件后缀
         */
        private String fileSuffix;

        public Builder setOutputDirPath(String outputDirPath) {
            this.outputDirPath = outputDirPath;
            return this;
        }

        public Builder setFilePrefix(String filePrefix) {
            this.filePrefix = filePrefix;
            return this;
        }

        public Builder setFileSuffix(String fileSuffix) {
            this.fileSuffix = fileSuffix;
            return this;
        }

        public RecorderOption build() {
            return new RecorderOption(this);
        }
    }
}
```

#### WebSocket配置

```
public class RxWebSocketBuilder {
    /**
     * 是否打印Log
     */
    boolean mIsPrintLog;
    /**
     * Log代理对象
     */
    Logger.LogDelegate mLogDelegate;
    /**
     * 支持外部传入OkHttpClient
     */
    OkHttpClient mClient;
    /**
     * 支持SSL
     */
    SSLSocketFactory mSslSocketFactory;
    X509TrustManager mTrustManager;
    /**
     * 重连间隔时间
     */
    long mReconnectInterval;
    /**
     * 重连间隔时间的单位
     */
    TimeUnit mReconnectIntervalTimeUnit;

    public RxWebSocketBuilder isPrintLog(boolean isPrintLog) {
        this.mIsPrintLog = isPrintLog;
        return this;
    }

    public RxWebSocketBuilder logger(Logger.LogDelegate logDelegate) {
        Logger.setDelegate(logDelegate);
        return this;
    }

    public RxWebSocketBuilder client(OkHttpClient client) {
        this.mClient = client;
        return this;
    }

    public RxWebSocketBuilder sslSocketFactory(SSLSocketFactory sslSocketFactory, X509TrustManager trustManager) {
        this.mSslSocketFactory = sslSocketFactory;
        this.mTrustManager = trustManager;
        return this;
    }

    public RxWebSocketBuilder reconnectInterval(long reconnectInterval, TimeUnit reconnectIntervalTimeUnit) {
        this.mReconnectInterval = reconnectInterval;
        this.mReconnectIntervalTimeUnit = reconnectIntervalTimeUnit;
        return this;
    }

    public RxWebSocket build() {
        return new RxWebSocket(this);
    }
}
```

#### 图片压缩配置

- 使用

```
LinghitRichEditorImageCompressOption compressOption = LinghitRichEditorImageCompressOption
                        .Builder
                        .newBuilder()
                        .setBitmapConfig(Bitmap.Config.ARGB_4444)
                        //默认，多少张图片就多少个线程进行压缩
                        .setCompressTaskNum(files.size())
                        .setIsOrigin(false)
                        .build();
```

- 实现

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

    public boolean isOrigin() {
        return isOrigin;
    }

    public Bitmap.Config getBitmapConfig() {
        return mBitmapConfig;
    }

    public File getDiskDirectory() {
        return mDiskDirectory;
    }

    public int getCompressTaskNum() {
        return mCompressTaskNum;
    }

    public Context getContext() {
        return mContext;
    }

    public String getTag() {
        return mTag;
    }

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