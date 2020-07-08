#### Android-skin-support 换肤框架使用

前段时间给App接入了换肤功能，使用到了[Android-skin-support](https://github.com/ximsfei/Android-skin-support)这个换肤框架，所以写这篇文章记录一下。

![换肤效果.png](https://upload-images.jianshu.io/upload_images/1641428-22e98665b7d3b61f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Android-skin-support集成

- 依旧使用v7的support，使用以下依赖

```
implementation 'skin.support:skin-support:3.1.4'                   // skin-support 基础控件支持
implementation 'skin.support:skin-support-design:3.1.4'            // skin-support-design material design 控件支持[可选]
implementation 'skin.support:skin-support-cardview:3.1.4'          // skin-support-cardview CardView 控件支持[可选]
implementation 'skin.support:skin-support-constraint-layout:3.1.4' // skin-support-constraint-layout ConstraintLayout 控件支持[可选]
```

- 已迁移到AndroidX，则使用以下依赖（注：和上面的support对比，就是版本号升级到了4.04，support是3.1.4喔）

```
implementation 'skin.support:skin-support:4.0.4'                   // skin-support
implementation 'skin.support:skin-support-appcompat:4.0.4'         // skin-support 基础控件支持
implementation 'skin.support:skin-support-design:4.0.4'            // skin-support-design material design 控件支持[可选]
implementation 'skin.support:skin-support-cardview:4.0.4'          // skin-support-cardview CardView 控件支持[可选]
implementation 'skin.support:skin-support-constraint-layout:4.0.4' // skin-support-constraint-layout ConstraintLayout 控件支持[可选]
```

- Activity基类继承SkinCompatActivity。继承的确不太友好，只有继承了SkinCompatActivity，换肤时，才会遍历View树上的控件进行换肤。

```
public class BaseActivity extends SkinCompatActivity {
}
```

#### Android-skin-support初始化以及简单封装使用

- 我对换肤框架初始化和换肤方法封装了一下，提供了一个Api接口以及一个实现类

```
//Api接口
interface SkinApi {
    /**
     * 初始化
     */
    fun init(application: Application)

    /**
     * 从Assets中加载皮肤apk
     * @param targetSkinName 目标皮肤名称
     * @param startBlock 开始换肤时回调
     * @param successBlock 换肤成功时回调
     * @param failedBlock 换肤失败时回调
     */
    fun loadSkinFromAssets(
        targetSkinName: String,
        startBlock: ((preSkinName: String) -> Unit)? = null,
        successBlock: ((preSkinName: String, applySkinName: String) -> Unit)? = null,
        failedBlock: ((errMsg: String) -> Unit)? = null
    )
}

//实现类
object SkinProxy : SkinApi {
    override fun init(application: Application) {
        SkinMaterialManager.init(application)
        //基础控件换肤初始化
        SkinCompatManager.withoutActivity(application)
            //material design 控件换肤初始化[可选]
            .addInflater(SkinMaterialViewInflater())
            //ConstraintLayout 控件换肤初始化[可选]
            .addInflater(SkinConstraintViewInflater())
            //CardView v7 控件换肤初始化[可选]
            .addInflater(SkinCardViewInflater())
            //关闭状态栏换肤，默认打开[可选]
            //.setSkinStatusBarColorEnable(false)
            //关闭windowBackground换肤，默认打开[可选]
            //.setSkinWindowBackgroundEnable(false)
            .loadSkinFromAssets(
                SkinStorage.getApplyAppSkinName()
            )
    }

    override fun loadSkinFromAssets(
        targetSkinName: String,
        startBlock: ((preSkinName: String) -> Unit)?,
        successBlock: ((preSkinName: String, applySkinName: String) -> Unit)?,
        failedBlock: ((errMsg: String) -> Unit)?
    ) {
        //获取应用前的皮肤名称，如果重复应用，不继续
        if (SkinStorage.getApplyAppSkinName() == targetSkinName) {
            return
        }
        SkinCompatManager.getInstance()
            .loadSkinFromAssets(targetSkinName, startBlock, { preSkinName, newApplySkinName ->
                //保存记录到本地
                SkinStorage.saveApplySkinName(newApplySkinName)
                successBlock?.invoke(preSkinName, newApplySkinName)
            }, failedBlock)
    }
}
```

- loadSkinFromAssets()方法，是从assets文件夹中加载皮肤包的方法，是我对库中SkinCompatManager添加的拓展方法，目的是添加换肤开始、成功、失败的回调。以及提供换肤前应用的皮肤名称。

```
/**
 * 插件化方式加载皮肤：从Assets文件夹中加载
 * @param targetSkinName 本次要应用的皮肤
 * @param startBlock 开始应用时回调
 * @param successBlock 应用成功时回调
 * @param failedBlock 应用失败时回调
 */
@JvmOverloads
fun SkinCompatManager.loadSkinFromAssets(
    targetSkinName: String,
    startBlock: ((preSkinName: String) -> Unit)? = null,
    successBlock: ((preSkinName: String, newApplySkinName: String) -> Unit)? = null,
    failedBlock: ((errMsg: String) -> Unit)? = null
) {
    //从sp中，获取应用前的皮肤名称
    val preSkinName = SkinStorage.getApplyAppSkinName()
    loadSkin(
        targetSkinName,
        object : SkinCompatManager.SkinLoaderListener {
            override fun onStart() {
                startBlock?.invoke(preSkinName)
            }

            override fun onSuccess() {
                successBlock?.invoke(preSkinName, targetSkinName)
            }

            override fun onFailed(errMsg: String?) {
                failedBlock?.invoke(errMsg ?: "")
            }
        },
        SkinCompatManager.SKIN_LOADER_STRATEGY_ASSETS
    )
}
```

- 在Application中调用初始化换肤框架

```
SkinProxy.init(it.applicationContext as Application)
```

- 切换皮肤，皮肤包打包见下面的 **打包皮肤包**

```
//目标皮肤包名称
val targetSkin = "purple.skin";
//开始切换皮肤
SkinProxy.loadSkinFromAssets(
    targetSkin, { preSkinName ->
        logd("开始换肤，当前应用的皮肤为: $preSkinName")
    }, { preSkinName, newApplySkinName ->
        logd("换肤成功，旧皮肤为: ${preSkinName}，新皮肤为: $newApplySkinName")
    }, { errMsg ->
        logd("换肤失败，errMsg: $errMsg")
    }
)
```

#### 打包皮肤包

打包皮肤包，库文档并没有细说，但其实很简单，新建一个Application的Module模块（可运行模块），在res资源目录下放置同名资源，打包为apk包，放到宿主的assets文件夹下的skins文件夹下。

主题可以有很多，我们不必每个皮肤包都新建一个Module，我们只需要使用Gradle做多渠道打包即可。

注意，皮肤包的包名不能和宿主包名一致，所以使用applicationIdSuffix，给每个皮肤包都加一个后缀即可。

- build.gradle文件配置多渠道打包

```
flavorDimensions "default"
//多种皮肤包，多渠道打包配置
productFlavors {
    //原始颜色
    "default" {
        applicationIdSuffix ".default"
    }
    //紫色
    purple {
        applicationIdSuffix ".purple"
    }
    //蓝色
    blue {
        applicationIdSuffix ".blue"
    }
}
```

- 例如：我的需求只有将主题色替换掉即可，所以在不同的多渠道文件夹下，放置不同主题的colors.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="base_color_primary">#673AB7</color>
    <color name="base_color_primary2">#7C4DFF</color>
    <color name="base_color_primary_dark">#512DA8</color>
    <color name="base_color_accent">#7C4DFF</color>
</resources>
```

![多渠道打包皮肤包.png](https://upload-images.jianshu.io/upload_images/1641428-44a0c7880a826c83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

- 打包皮肤包apk，将apk重命名，注意将apk后缀改为skin后缀，然后将皮肤包放置到宿主的assets文件夹下的skins文件夹下。

![放置皮肤包到宿主assets文件夹下.png](https://upload-images.jianshu.io/upload_images/1641428-a365f7b9e8a7b54e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 自定义控件支持换肤

换肤框架，提供了v7、design库的控件的换肤版本，但是我们自己的自定义控件则自己去适配了，下面我给出自定义顶部栏、TabLayout的换肤适配。（还有其他适配方案，可以看官方Github文档），基本步骤如下：

1. 继承需要换肤的控件，实现SkinCompatSupportable接口
2. 在控件的构造方法中，获取需要换肤的自定义属性，获取当前应用的皮肤资源，进行换肤（回显之前的换肤设置）
3. 在SkinCompatSupportable接口的applySkin回调中，再处理应用运行中换肤的处理（启动时不会回调，手动切换皮肤时回调）。

- 自定义顶部栏，其实使用的是QMUI的TopBar，有兴趣的小伙伴可以去看下。

- 自定义属性

```
<!--************ TopBar自定义属性 ***********-->
<declare-styleable name="TopBar">
    <!-- 省略其他自定义属性... -->
    
    <attr name="topbar_bg_color" format="color"/>
    
    <!-- 省略其他自定义属性... -->
</declare-styleable>
```

- 换肤兼容

```
//定义支持换肤控件的TopBar
public class SkinCompatTopBar extends TopBar implements SkinCompatSupportable {
    private int mTopBarBgColorResId;

    public SkinCompatTopBar(Context context, AttributeSet attrs) {
        super(context, attrs);
        //步骤一：获取需要支持换肤的自定义属性，例如这里顶部栏的背景颜色
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TopBar, 0, 0);
        mTopBarBgColorResId = array.getResourceId(R.styleable.TopBar_topbar_bg_color, SkinCompatHelper.INVALID_ID);
        array.recycle();
        //2。应用之前使用的换肤（回显）
        applyTopBarBackgroundColor();
    }

    //换肤处理
    private void applyTopBarBackgroundColor() {
        mTopBarBgColorResId = SkinCompatHelper.checkResourceId(mTopBarBgColorResId);
        if (mTopBarBgColorResId != SkinCompatHelper.INVALID_ID) {
            int color = SkinCompatResources.getColor(getContext(), mTopBarBgColorResId);
            setTopBarBackgroundColor(color);
        }
    }

    @Override
    public void applySkin() {
        //3、应用运行间，手动切换换肤回调，再次进行换肤操作
        applyTopBarBackgroundColor();
    }
}
```

- TabLayout，自定义SkinTabLayout继承于SkinMaterialTabLayout，而SkinMaterialTabLayout继承design包的TabLayout，但是换肤库中的SkinMaterialTabLayout并没有处理TabLayout的背景换肤处理，不太明白为什么其他属性都处理了，这个那么常用的属性不处理。

```
public class SkinTabLayout extends SkinMaterialTabLayout {
    private SkinCompatBackgroundHelper mSkinCompatBackgroundHelper;

    public SkinTabLayout(Context context) {
        this(context, null);
    }

    public SkinTabLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SkinTabLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        //步骤一：库里其实提供了android:backgroundColor背景颜色属性处理SkinCompatBackgroundHelper类，我们让它去对TabLayout进行背景颜色换肤即可，不需要自己写
        mSkinCompatBackgroundHelper = new SkinCompatBackgroundHelper(this);
        mSkinCompatBackgroundHelper.loadFromAttributes(attrs, defStyleAttr);
        //步骤二：马上处理换肤
        mSkinCompatBackgroundHelper.applySkin();
    }

    @Override
    public void applySkin() {
        super.applySkin();
        //步骤三：应用运行间，手动切换换肤回调，再次进行换肤操作
        if (mSkinCompatBackgroundHelper != null) {
            mSkinCompatBackgroundHelper.applySkin();
        }
    }
}
```