#### Android 多渠道打包总结

#### 为什么要多渠道

App一般会上传多个应用商店，例如应用宝、小米、华为、OPPO等。

1. 现在的手机都自带应用商店，用户一般会在自带应用商店上搜索。

2. 产品、运营可统计使用不同手机品牌的用户的消费情况，App日活、月活、UV、PV等。

#### 传统方式打包的痛点

如果App不是很出名，一般都需要自己打包上传，除非像微信这种，商店自动抓包进行上传。曾经参加过一个技术沙龙，微信的工程师说到，他们的热更新技术，如果使用了360加固进行，就是失效，但是360应用商店必须加固后才能上传，所以他们就不上传了，最后360应用商店也还是抓包上传了（可见有实力就是不一样）。

不同的渠道，可能App名字不同，App图标Icon不同，如果手动替换的话，体力劳动特别累人，笔者就曾经做过一个app，需要给不同的景区生成app，每个景区的图标和app名字都不一样，传统方式就是打包完一个，手动替换再进行打包，10个还好，慢慢后面景区越来越多，上升了50个，工作量可想而知，而且还容易错，每个都需要手动检查一遍（曾经花2个小时，对着屏幕一个个替换，眼睛都花了，自然容易出错！），这样进行几次后，意识到这种重复劳动不应该由我们来做，应该交给机器呀！

#### Gradle配置多渠道打包

后面我们搜索了一下资料，决定使用Gradle配置的方式进行打包，但他也有优缺点。

- 优点：就是每个包都只需要配置好要替换的文件和占位，即可开始打包。

- 缺点：每次打包一个渠道包，都需要编译一次，项目非常大的时候，可谓非常耗时！

#### Manifest占位，动态替换meta标签值，动态标识渠道

#### 实现步骤

1. 配置App模块的build.gradle文件。

	- 使用productFlavors，添加2个渠道，例如360和应用宝。
	- 注意如果是3.0版本的Gradle，必须要添加上flavorDimensions纬度。
	- 使用manifestPlaceholders，增加清单文件占位，格式：[占位名:值, 占位名:值]。

```
apply plugin: 'com.android.application'

android {
    //省略其他配置...

    //3.0版本Gradle开始必须添加纬度
    flavorDimensions "default"
    
    //多渠道打包配置
    productFlavors {
        //渠道包
        app_360 {
            manifestPlaceholders = [channel: "app_360"]
        }
        app_qq {
            manifestPlaceholders = [channel: "app_qq"]
        }
    }
}
```

2. 清单文件增加占位标识
	- 例如我们使用友盟进行渠道记录，增加一个meta_data标签，标签名为UMENG_CHANNEL，值是占位符标识${channel}。

**注意这个标识要和第一步的build.gradle文件中的productFlavors配置一致**

- 例如build.gradle。

```
//占位名：channel，值：app_360
manifestPlaceholders = [app_name: "360渠道包", channel: "app_360"]
```

- meta标签。

```
<!-- 占位名：channel，不包括${} -->
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="${channel}" />
```

- 完整配置

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.zh.makechannelpackage">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!-- 省略其他配置... -->

        <!-- 多渠道配置 -->
        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${channel}" />
    </application>
</manifest>
```

3. 在Java代码层读取meta标签，获取其值即可。

```
//渠道工具类
public class ChannelUtil {
    /**
     * 获取app包内的渠道标识
     */
    public static String getChannel(Context context) {
        try {
            PackageManager manager = context.getPackageManager();
            ApplicationInfo appInfo = manager.getApplicationInfo(context.getPackageName(), PackageManager.GET_META_DATA);
            return appInfo.metaData.getString("UMENG_CHANNEL");
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
            return "";
        }
    }
}

//调用
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String channel = ChannelUtil.getChannel(getApplicationContext());
        Toast.makeText(getApplicationContext(), "当前渠道：" + channel, Toast.LENGTH_SHORT).show();
    }
}
```

#### 给不同的编译环境配置清单Key

除了productFlavors中配置manifestPlaceholders生成渠道包外，在编译环境buildTypes下也可以使用，例如不同编译环境下，高德地图的key不同，就可以在buildTypes中使用。

1. 修改build.gradle配置。

```
buildTypes {
    debug {
        manifestPlaceholders = [amap_key: "amp_debug_xxxxxkey"]
        //省略其他配置...
    }

    release {
        manifestPlaceholders = [amap_key: "amp_release_xxxxxkey"]
        //省略其他配置...
    }
}
```

2. 清单文件中添加高德Key配置。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.zh.makechannelpackage">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- 配置高德key -->
        <meta-data
            android:name="com.amap.api.v2.apikey"
            android:value="${amap_key}" />
    </application>
</manifest>
```

#### Lib模块共享，App接入问题

一般我们都会使用多个Lib进行分模块开发，例如支付模块的Pay库中的微信回调Activity，7.0获取文件需要使用FileProvider等，都需要制定包名，但是我们的lib需要提供给其他App进行接入，清单文件的配置需要接入方配置，如果配置比较多，而且后续版本需要更换配置，都需要接入方重新配置会比较麻烦，那么可不可以将配置留在Lib库中，通过动态配置的方式动态配置呢。

1. 例如微信支付的回调Activity配置，通过applicationId占位，可以动态获取到接入方的包名，只要按照约定，在包名下建立wxapi包下，建立WXEntryActivity文件即可。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.zh.makechannelpackage">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- 动态配置微信回调Activity -->
        <activity
            android:name="${applicationId}.wxapi.WXEntryActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:exported="true"
            android:theme="@android:style/Theme.Translucent.NoTitleBar" />
            
        <!-- 省略其他配置... -->
    </application>
</manifest>
```

2. 再例如图片选择库，使用到了FileProvider，我们不希望接入方配置，则也可以使用applicationId进行占位。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.zh.makechannelpackage">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- FileProvider配置 -->
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        <!-- 省略其他配置... -->
    </application>
</manifest>
```

#### 动态指定App名称

经过前面的配置已经足够配置出不同渠道标识的渠道包了，我们还可以继续拓展下，例如不同渠道的App名字不同。（一般因为有的应用商店会以App名称而被拒...）

1. 修改build.gradle文件。

manifestPlaceholders中添加一个标识，**app_name**，值为对应的名称。例如360渠道为360渠道包，应用宝渠道为应用宝渠道。

```
//3.0Gradle开始必须添加纬度
flavorDimensions "default"
//多渠道打包配置
productFlavors {
        //渠道包
        app_360 {
            manifestPlaceholders = [app_name: "360渠道包", channel: "app_360"]
        }
        app_qq {
            manifestPlaceholders = [app_name: "应用宝渠道包", channel: "app_qq"]
        }
}
```

#### 动态指定App包名

例如我们debug环境下，希望测试包和正式包能够共存，那么共存就需要包名不同，所以可以对包名进行替换或者加后缀。

1. 修改build.gradle文件。
	- applicationIdSuffix，给包名添加后缀。例如测试版加上.internal后缀。
	- applicationId，整个替换包名。（一般不会这么干，只是提一下）

```
//3.0Gradle开始必须添加纬度
flavorDimensions "default"
//多渠道打包配置
productFlavors {
    //开发版
    developer {
        //测试版加包名后缀，方便和正式版共存
        applicationIdSuffix ".internal"
        manifestPlaceholders = [app_name: "开发版", channel : "app_internal"]
    }
    production {
        //也可以完全替换包名
        applicationId "me.zh.demo"
        manifestPlaceholders = [app_name: "正式版",
                                channel: "app_internal"]
    }
}
```

#### 动态生成变量

开发中，Log打印是必不可少的，但是我们在正式环境是需要将Log打印去掉的，常规做法就是打印函数前加一个LogEnable的变量，将打印调用去掉。而我们一般会在常量类中添加开关变量。

```
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        Logger.setDelegate(new L());
        Logger.setLogPrintEnable(true);
    }
}
```

- 问题：而每次打包之前都需要将开关变量设置为false，很容易忘。

- 期望：我们希望不同buildType下的编译环境，打印Log是不一样的，例如buildType为debug时，Log开关为开，buildType为release时，Log开关为关。

1. Gradle为我们提供了buildConfigField，用于动态生成变量在BuildConfig，那么我们给不同的buildType进行添加即可。

```
//编译类型
buildTypes {
    debug {
        //打印开关变量
        buildConfigField("boolean", "LOG_ENABLE", "true")
        //省略其他配置...
    }

    release {
        buildConfigField("boolean", "LOG_ENABLE", "false")
        //省略其他配置...
    }
}
```

2. 在Java代码中，设置BuildConfig中生成的变量即可。

```
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        Logger.setDelegate(new L());
        Logger.setLogPrintEnable(BuildConfig.LOG_ENABLE);
    }
}
```

#### 不同渠道替换资源文件

能替换包名和App名字了，原以为完毕，结果运营说，360商店我们要出首发！图标和启动图要加上360的标识！这时候就需要用到同结构、同名文件夹来替换了。

1. 在和main文件夹下，建立同名渠道的文件夹，例如**app_360**渠道，建立的文件夹名为**app_360**。

2. assets文件夹、res文件夹、以及AndroidManifest.xml清单文件结构都要和main中的一致。

3. AppIcon同名即可替换。清单文件同名即可，内部特定不同的内容即可。

![多渠道配置1.png](https://upload-images.jianshu.io/upload_images/1641428-b6189389b3ebad86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![多渠道配置2.png](https://upload-images.jianshu.io/upload_images/1641428-d03a63b548688fe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 关于Java代码多渠道

上面说到资源文件，建立同目录进行替换，那么Java代码可以同级目录下，同名Java文件替换吗？很遗憾的是不可以，需要不同渠道，进行不同的逻辑时，只能通过刚才ChannelUtil获取渠道信息，进行逻辑判断了。

#### 总结

使用Gradle来进行配置，大大减少了我们的工作量，妈妈再也不用担心我打渠道包需要2小时啦，至于gradle每构建一个渠道包都编译一次问题，可以单独给assets设置一个配置文件，后续使用zip修改配置文件，达到只打一个包，其他包都使用打包工具进行打包，时间可以节省非常多。