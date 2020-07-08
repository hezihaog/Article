#### App启动页全屏处理

一般App都有一个启动页来作为品牌Logo或广告显示，如果要达到全屏的效果，有2种方式：

1. Activity全屏处理
2. Activity的状态栏透明化

#### 方案一：Activity全屏处理

- 在登录页的主题中设置为全屏，并且设置Window的背景为启动图片。

- 主题的父类是Theme.AppCompat.NoActionBar，所以也隐藏掉了ActionBar。

```
<!--欢迎页面theme-->
<style name="app_welcome_theme" parent="Theme.AppCompat.NoActionBar">
    <item name="android:windowBackground">@drawable/app_welcome_bg</item>
    <item name="android:windowFullscreen">true</item>
</style>
```

- 在启动页跳转到主页时，将全屏取消掉。

```
/**
 * 隐藏状态栏
 */
publi void hideStatusBar(Activity activity) {
    if (activity != null && !activity.isFinishing()) {
        WindowManager.LayoutParams attrs = activity.getWindow().getAttributes();
        attrs.flags |= WindowManager.LayoutParams.FLAG_FULLSCREEN;
        activity.getWindow().setAttributes(attrs);
    }
}

public class WelcomeActivity extends BaseActivity() {
	 //跳转到主页
	 private void goHome() {
        hideStatusBar(this);
        startActivity(this, HomeActivity.class);
        finish();
    }
}
```

- 全屏方案的缺点

全屏Activity切换到主页时为非全屏，状态栏会突然出现，造成界面的抖动。效果并不是很完美。

#### 方案二：Activity的状态栏透明化

先来讲一下代码设置透明的方式，实现是没问题的，也没有抖动，但是状态栏有从半透明到透明的效果，怀疑是代码设置的速度会比较慢的原因。

- 主题中，去掉全屏配置，只留一个Window背景为启动图

```
<!--欢迎页面theme-->
<style name="app_welcome_theme" parent="Theme.AppCompat.NoActionBar">
    <item name="android:windowBackground">@drawable/app_welcome_bg</item>
</style>
```

- Activity的onCreate()中，调用setStatusBarTranslucent()透明化状态栏

```
/**
 * 设置状态栏透明
 */
public void setStatusBarTranslucent(Activity activity) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        Window window = activity.getWindow();
        window.setNavigationBarColor(Color.BLACK);
        window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
        window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
        View decorView = window.getDecorView();
        int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
        decorView.setSystemUiVisibility(option);
        //透明着色
        window.setStatusBarColor(Color.TRANSPARENT);
    }
}

public class WelcomeActivity extends BaseActivity() {
	 public void onCreate(Bundle savedInstanceState) {
        //透明化状态栏
        setStatusBarTranslucent(this);
    }
}
```

#### 修复方案

- 既然代码设置透明比较慢，那么就从主题中设置，预先被处理，主要是以下2个属性：

	1. android:windowTranslucentStatus：透明掉状态栏，设置为true，状态栏有阴影，false则无阴影。
	2. android:statusBarColor：设置状态栏颜色为透明色。

```
<!--欢迎页面theme-->
<style name="app_welcome_theme" parent="Theme.AppCompat.NoActionBar">
    <item name="android:windowBackground">@drawable/app_welcome_bg</item>
	 <!-- 透明掉状态栏，设置为true，状态栏有阴影，false则无阴影 -->
    <item name="android:windowTranslucentStatus">false</item>
    <!-- 设置状态栏颜色为透明色 -->
    <item name="android:statusBarColor">@android:color/transparent</item>
</style>
```

- Activity则去掉在onCreate()中调用setStatusBarTranslucent()来透明的方法

```
public class WelcomeActivity extends BaseActivity() {
	 public void onCreate(Bundle savedInstanceState) {
		   //...
    }
}
```