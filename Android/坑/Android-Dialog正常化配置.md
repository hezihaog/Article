#### Android-Dialog正常化配置

Android上的弹窗，一般使用Dialog来实现，如果需要自定义弹窗，也是继承Dialog来实现，但是往往有一些行为不正常的问题：

1. 弹窗背景默认是白色，如果弹窗是白色圆角的，因为默认白色背景在底部，则看不出圆角了（去掉默认背景，背景就是透明的了，那么弹窗的布局记得要加背景）。
2. 弹窗顶部默认有一个标题，往往我们不需要。
3. 弹窗的宽度，设置为MATCH_PARENT却不能铺满全屏。
4. 弹窗的左、右，默认距离屏幕有一定的padding，一般我们不需要。

#### 弹窗基类

一般我们会新建一个弹窗基类，进行初始化设置，解决上面的4个问题。

- styles.xml，新建一个BaseDialog主题，进行基础配置。

```
<style name="BaseDialog" parent="@android:style/Theme.Dialog">
    <!--  去掉弹窗的Window边框  -->
    <item name="android:windowFrame">@null</item>
    <!--  去掉弹窗的标题  -->
    <item name="android:windowNoTitle">true</item>
    <!--  设置弹窗的Window背景透明  -->
    <item name="android:windowBackground">@android:color/transparent</item>
    <!--  设置弹窗浮现在Activity之上  -->
    <item name="android:windowIsFloating">true</item>
    <!--  开启弹窗阴暗背景  -->
    <item name="android:backgroundDimEnabled">true</item>
    <!--  设置弹窗背景的阴暗程度  -->
    <item name="android:backgroundDimAmount">0.6</item>
    <!--  设置全屏  -->
    <item name="android:windowFullscreen">true</item>
</style>
```
- BaseDialog弹窗基类，主要是调用normalize()，修复弹窗背景和弹窗大小以及padding。同时声明了一个抽象方法onSetupDialogFrameSize()，方法传入设备的宽高，子类复写该方法，将需要的宽高放到数组中返回即可，如果需要弹窗宽度铺满屏幕，则返回屏幕宽度，如果需要包裹内容则返回ViewGroup.LayoutParams.WRAP_CONTENT。

```
public abstract class BaseDialog extends Dialog {
	public BaseDialog(@NonNull Context context) {
	        //基础主题，R.style.BaseDialog
	        super(context, R.style.BaseDialog);
	        init();
	    }
	
	    public BaseDialog(@NonNull Context context, int themeResId) {
	        //外部传入主题，推荐主题继承我们的基础弹窗主题，BaseDialog
	        super(context, themeResId);
	        init();
	    }
	
	    private void init() {
	        //让Dialog正常化
	        normalize();
	    }
	    
    /**
     * 正常化Dialog
     */
    private void normalize() {
        fixBackground();
        fixSize();
    }

    /**
     * 修复背景
     */
    private void fixBackground() {
        Window window = getWindow();
        if (window == null) {
            return;
        }
        //1、去掉白色的背景
        window.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
    }

    /**
     * 修复大小
     */
    private void fixSize() {
        Window window = getWindow();
        if (window == null) {
            return;
        }
        //获取需要显示的宽、高
        DisplayMetrics metrics = new DisplayMetrics();
        //3、当子类决定弹窗宽高
        getWindow().getWindowManager().getDefaultDisplay().getMetrics(metrics);
        int[] frameSize = onSetupDialogFrameSize(metrics.widthPixels, metrics.heightPixels);
        //4、去掉默认padding间距
        window.getDecorView().setPadding(0, 0, 0, 0);
        WindowManager.LayoutParams params = window.getAttributes();
        params.width = frameSize[0];
        params.height = frameSize[1];
        window.setAttributes(params);
    }
    
    /**
     * 子类需要复写该方法，返回需要的宽高
     */
    protected abstract int[] onSetupDialogFrameSize(int screenWidth, int screenHeight);
}
```

#### 基类使用例子

- 例如：分享弹窗，我们希望弹窗宽度是铺满屏幕，高度有多高就多高。那么我们在复写onSetupDialogFrameSize()方法中，返回屏幕宽度，高度包裹内容，即可。

```
public class ShareDialog extends BaseDialog {
    @Override
    protected int[] onSetupDialogFrameSize(int screenWidth, int screenHeight) {
        int[] size = new int[2];
        size[0] = screenWidth;
        size[1] = ViewGroup.LayoutParams.WRAP_CONTENT;
        return size;
    }
}
```

- 例如：底部弹窗，弹窗宽度为设备屏幕有80%，我们在onSetupDialogFrameSize()方法中将宽度乘以80%，再和高度一起返回即可。

```
public class UpgradeTipDialog extends BaseDialog {
    @Override
    public void onStart() {
        super.onStart();
        //设置弹窗在底部弹出
        Window window = getWindow();
        WindowManager.LayoutParams params = window.getAttributes();
        params.gravity = Gravity.BOTTOM;
        window.setAttributes(params);
    }

    @Override
    protected int[] onSetupDialogFrameSize(int screenWidth, int screenHeight) {
        int[] size = new int[2];
        size[0] = screenWidth * 0.08f;
        size[1] = screenHeight;
        return size;
    }
}
```

- 例如：升级弹窗，弹窗大小按照Layout资源文件中的大小，弹窗为全屏，onSetupDialogFrameSize()方法中返回设备的宽、高即可。

```
public class UpgradeTipDialog extends BaseDialog {
    @Override
    protected int[] onSetupDialogFrameSize(int screenWidth, int screenHeight) {
        int[] size = new int[2];
        size[0] = screenWidth;
        size[1] = screenHeight;
        return size;
    }
}
```