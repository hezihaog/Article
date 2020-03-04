#### Android 仿vivo的SeekBar样式

本篇其实是有点水的，目的是为了记录一下SeekBar的样式自定义步骤，SeekBar使用并不是特别多，但一需要用的时候就东找西找，实在不好，也耗时间，刚好看到vivo系统设备里面的SeekBar样式，就想借此做一下，再做一下记录。

![vivo设置页面.jpeg](https://upload-images.jianshu.io/upload_images/1641428-82dae6213cbe3b04.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 最终效果

先来看下最终的效果，基本和vivo的一致了。

![Android 仿vivo的SeekBar样式.gif](https://upload-images.jianshu.io/upload_images/1641428-c544dde6018e6ef6.gif?imageMogr2/auto-orient/strip)

#### 需要自定义的部分

从上图可以看出，有3个地方需要自定义：

1. 已有进度颜色 - 蓝色
2. 未有的进度颜色 - 灰色
3. 圆形滑块

那么就开始吧，步骤不是很多

#### 进度颜色

已有进度和未有进度的定义是在同一个xml文件里定义的。我们在res-drawable文件夹下，新建一个xml文件，类型是layer-list。

 - 背景颜色为灰色，新建一个item，id是@android:id/background，这个是固定死的，不能改。
 - 进度颜色为蓝色，同样新建一个item，id是@android:id/progress，也是不可以改的。
 - item内嵌一个shap，形状是矩形圆角，圆角半径是2dp。

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 背景 -->
    <item android:id="@android:id/background">
        <shape android:shape="rectangle">
            <solid android:color="#ACACAC" />
            <corners android:radius="2dp" />
        </shape>
    </item>

    <!-- 进度 -->
    <item android:id="@android:id/progress">
        <clip>
            <shape android:shape="rectangle">
                <solid android:color="#2382F7" />
                <corners android:radius="2dp" />
            </shape>
        </clip>
    </item>
</layer-list>
```

#### 圆形滑块

圆形滑块，同样在res-drawable文件夹下，新建一个xml文件，类型是shape，形状为oval椭圆，后面会定义size，设置为圆形。

- 滑块的填充颜色为白色，使用solid标签定义。
- 滑块的边有一圈灰色，使用stroke标签定义，颜色灰色。
- 定义滑块的大小，使用size标签定义，大小为26dp。

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#FFFFFF" />
    <stroke
        android:width="1dp"
        android:color="#BBBBBB" />
    <size
        android:width="26dp"
        android:height="26dp" />
</shape>
```

#### 背景和滑块配置

- 进度、背景使用progressDrawable属性配置。
- 滑块，使用thumb属性配置。如果需要去掉滑块，则设置为android:thumb="@null"即可。

注意：

maxHeight和minHeight必须要设置，否则滑块的高度会不能超过SeekBar的高度，少配置一个都不行！

```
<SeekBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        android:layout_marginEnd="20dp"
        android:max="100"
        android:maxHeight="2dp"
        android:minHeight="2dp"
        android:progress="0"
        android:progressDrawable="@drawable/app_vivo_seek_bar_bg"
        android:thumb="@drawable/app_vivo_seek_bar_thumb"
        tools:progress="40" />
```

#### 处理滑块被截断的问题

5.0以下，使用上面的定义是没有问题的，但是在5.0却出现滑块和背景之间有截断的现象，处理这个问题，我们需要加以下属性来处理。

```
android:thumbOffset="0dp"
```

#### 左右两边有一定距离的padding

默认2边会有一定距离的padding，我们都配置为0dp即可。

```
android:paddingStart="0dp"
android:paddingLeft="0dp"
android:paddingEnd="0dp"
android:paddingRight="0dp"
```

#### 去掉拖拉滑块时，默认阴影效果

默认拽托滑块时，会有一圆阴影浮现在滑块上，如果需要去掉这个效果，需要加一个属性。

```
android:duplicateParentState="true"
```

#### 最终配置

```
<SeekBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        android:layout_marginEnd="20dp"
        android:duplicateParentState="true"
        android:max="100"
        android:maxHeight="2dp"
        android:minHeight="2dp"
        android:paddingStart="0dp"
        android:paddingLeft="0dp"
        android:paddingEnd="0dp"
        android:paddingRight="0dp"
        android:progress="0"
        android:progressDrawable="@drawable/app_vivo_seek_bar_bg"
        android:thumb="@drawable/app_vivo_seek_bar_thumb"
        android:thumbOffset="0dp"
        tools:progress="40" />
```