#### ViewDragHelper实战，实现QQ侧滑菜单

## 前言

前面使用ViewDragHelper实现过[滑动解锁](https://www.jianshu.com/p/81b329788511)、[浮动按钮](https://www.jianshu.com/p/2e4316740fc2)，可以说ViewDragHelper真是一个神器，实现拽托、移动，都只需要很少的代码就可以实现，手势相关代码都在ViewDragHelper内部帮我们封装好了，本篇再次使用ViewDragHelper来实现QQ的侧滑菜单。

说到这个侧边栏，一开始学习安卓的时候，看的是一脸懵逼，一直都想实现它，但因当时水平不过，不足以有能力去实现，如今做安卓也有些年头了，终于也用ViewDragHelper实现了这个功能。想想还是有些兴奋呢~

当然本篇的代码，只能实现基本的左滑菜单，并且可能会有一些手势冲突没有处理，但是主线依旧是左侧滑，其他处理都是锦上添花~如果实际开发中要用，最好还是使用已经造好的轮子。（轮子虽好，但始终不是自己的，有空自己动手，才能理解到精髓喔）

## 成品展示

![ViewDragHelper实战，实现QQ侧滑菜单.gif](https://upload-images.jianshu.io/upload_images/1641428-fba41ec655c37710.gif?imageMogr2/auto-orient/strip)

## 分析

#### 布局分析

侧滑菜单，一共分为3个部分：

- 页面整体，包含**菜单部分**和**内容部分**

- 菜单部分，和内容布局同一个层级，被SlidingMenu控件包裹。

- 内容部分，同上，和菜单部分同一个层级，被SlidingMenu控件包裹。不难发现我们的效果中，打开菜单时，内容区域会有一个黑色遮罩，根据滑动的距离从透明到半透明，所以内容部分还有一个遮罩。

#### 行为分析

 - 打开菜单

	1. 菜单从左向右拉出，菜单跟随内容一起向右移动。
	2. 内容从左侧拉到完全出现，不能继续拉，内容也不能继续拉。

- 关闭菜单

	1. 内容从右向左拉回，菜单跟随内容一起向左移动。
	2. 内容从右向左移动，完全移出屏幕时，内容不能继续向左拉。

- 其他细节

	1. 拽托菜单和内容都可以拉动整体菜单的移动。
	2. 遮罩根据拽托的位置，形成一个比值，让遮罩的透明度改变。
		- 菜单打开时，比值为1，菜单关闭时，比值为0。
		- 从关闭到打开，比值从0开始，逐渐趋向1。

#### 逐步实现

- 创建侧滑菜单布局 SlidingMenu

	1. 复写onFinishInflate()，获取子控件和布局子控件，必须只有菜单和内容这2个子控件。
	2. 复写onLayout()，布局菜单和内容。
		- 菜单布局，left值为负的菜单宽度，right值为父布局的left，所以一开始菜单都默认隐藏在屏幕的左侧，是看不到的。
		- 内容区域，所有值都和父布局一样即可，就是铺满父布局。

```
public class SlidingMenu extends FrameLayout {
    /**
     * 菜单View
     */
    private View vMenuView;
    /**
     * 内容View
     */
    private View vContentView;

    public SlidingMenu(@NonNull Context context) {
        this(context, null);
    }

    public SlidingMenu(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SlidingMenu(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }
    
    private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        //初始化...
    }
    
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        //获取所有子View，确保只有菜单和内容这2个控件
        int childCount = getChildCount();
        if (childCount != 2) {
            throw new IllegalStateException("侧滑菜单内只能有2个子View，分别是菜单和内容");
        }
        vMenuView = getChildAt(0);
        vContentView = getChildAt(1);
    }
    
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        //布局菜单和内容，菜单在最左边，普通状态是看不到的
        vMenuView.layout(-vMenuView.getMeasuredWidth(), top, left, bottom);
        //内容View铺满整个父控件
        vContentView.layout(left, top, right, bottom);
    }
}
```

- 新建菜单状态回调接口，提供以下回调方法：

	- onMenuOpen()，当菜单开启时回调。
	- onSliding(float fraction)，正在滑动时回调，回传fraction值，为滑动比值。
	- onMenuClose()，当菜单关闭时回调。

```
/**
 * 菜单状态改变监听
 */
private OnMenuStateChangeListener mMenuStateChangeListener;

/**
 * 菜单状态改变监听
 */
public interface OnMenuStateChangeListener {
    /**
     * 当菜单开启时回调
     */
    void onMenuOpen();

    /**
     * 正在滑动时回调
     *
     * @param fraction 滑动百分比值
     */
    void onSliding(float fraction);

    /**
     * 当菜单关闭时回调
     */
    void onMenuClose();
}

/**
 * 设置状态改变监听
 *
 * @param menuStateChangeListener 监听器
 */
public void setOnMenuStateChangeListener(OnMenuStateChangeListener menuStateChangeListener) {
    mMenuStateChangeListener = menuStateChangeListener;
}
```

- 创建ViewDragHelper，将事件委托给ViewDragHelper处理
	- 复写onInterceptTouchEvent()，事件委托给ViewDragHelper。
	- 复写onTouchEvent()，事件委托给ViewDragHelper。
	- 复写computeScroll()，因为ViewDragHelper的滚动是依靠Scroller的，所以需要将Scroller的相关处理交给ViewDragHelper。

```
/**
 * 拽托帮助类
 */
private ViewDragHelper mViewDragHelper;

/**
 * 初始化
 */
private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
    	//...先省略，下面会说
    }
}

@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    //将onInterceptTouchEvent委托给ViewDragHelper
    return mViewDragHelper.shouldInterceptTouchEvent(ev);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    //将onTouchEvent委托给ViewDragHelper
    mViewDragHelper.processTouchEvent(event);
    return true;
}

@Override
public void computeScroll() {
    super.computeScroll();
    //判断是否移动到头了，未到头则继续
    if (mViewDragHelper != null) {
        if (mViewDragHelper.continueSettling(true)) {
            invalidate();
        }
    }
}
```

- 复写ViewDragHelper，处理横向侧滑

	1. 复写tryCaptureView()，确定菜单和内容可以拽托
	2. 复写getViewHorizontalDragRange()，返回拽托范围，返回非0值即可，某些情况需要该值来确定是否可以拽托。
	3. 复写clampViewPositionHorizontal()，处理横向拽托，由于菜单和内容都可以拽托，拽托这2部分都会回调，但方法传入的child为菜单或内容。所以需要判断拽托的控件来处理。
		- 拽托的是菜单，处理如下：
			- 左边界left不能超过菜单宽度，因坐标系，屏幕左侧为负值，所以不能小于负的菜单宽度。
			- 左边界left不能超过0，因为只能完全显示菜单后，就不能继续向右拽托了。
			- 其他情况为允许值，直接返回传入的left值即可。

		- 拽托的是内容，处理如下：
			- 左边界left不能超过0，因为内容不能拽托出屏幕左侧。
			- 左边界left不能超过菜单宽度，因为菜单完全显示后，内容就不能继续向右拽托了。
			- 其他情况为允许值，直接返回传入的left值即可。

```
//...省略其他代码

/**
 * 初始化
 */
private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
        @Override
        public boolean tryCaptureView(@NonNull View child, int pointerId) {
            //菜单和内容都可以拽托
            return child == vMenuView || child == vContentView;
        }

        @Override
        public int getViewHorizontalDragRange(@NonNull View child) {
            //拽托范围，返回非0值即可，某些情况需要该值来确定是否可以拽托
            return vContentView.getWidth();
        }

        @Override
        public int clampViewPositionHorizontal(@NonNull View child, int left, int dx) {
            //处理横向拽托，获取菜单宽度
            int menuWidth = vMenuView.getWidth();
            //因为拽托菜单和内容传入的对象不同，我们需要处理2种情况
            if (child == vMenuView) {
                //拽托的是菜单
                if (left < -menuWidth) {
                    //左边距离，最多只能完全隐藏于屏幕最左侧
                    return -menuWidth;
                } else if (left > 0) {
                    //左边距离，最多能完全出现在屏幕
                    return 0;
                } else {
                    return left;
                }
            } else if (child == vContentView) {
                //拽托的是内容区域，不能移动超出最左边的屏幕
                if (left < 0) {
                    return 0;
                } else if (left > menuWidth) {
                    //最多不能超过菜单的宽度
                    return menuWidth;
                } else {
                    return left;
                }
            }
            return 0;
        }
    });
}

//...省略其他代码
```

- 处理拽托时，联动处理

如果只复写clampViewPositionHorizontal()，只能拽托菜单或者内容的单独移动，它们并不是联动的，我们需要拽托菜单时，内容随着一起移动。相应的，拽托内容时，菜单也会随着一起移动。

1. 复写onViewPositionChanged()，当拽托菜单或内容时，回调移动位置等相关信息。
2. 同样需要判断child对象是菜单还是内容（）。
	- 拽托对象为菜单时，手动调用内容View的layout()方法，让内容View移动。
	- 拽托对象为内容时，手动调用菜单View的layout()方法，让内容View移动。
3. 处理开、关状态处理以及回调监听。
4. 定义菜单打开、关闭方法
	- openMenu()，打开菜单。
	- closeMenu()，关闭菜单。

```
//...省略其他代码

/**
 * 初始化
 */
private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
		mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public void onViewPositionChanged(@NonNull View changedView, int left, int top, int dx, int dy) {
                super.onViewPositionChanged(changedView, left, top, dx, dy);
                //处理联动，拽托菜单布局，让内容布局跟着动
                if (changedView == vMenuView) {
                    int newLeft = vContentView.getLeft() + dx;
                    int right = newLeft + vContentView.getWidth();
                    vContentView.layout(newLeft, top, right, getBottom());
                } else if (changedView == vContentView) {
                    //拽托内容布局，让菜单布局跟着动
                    int newLeft = vMenuView.getLeft() + dx;
                    vMenuView.layout(newLeft, top, left, getBottom());
                }
                if (mMenuStateChangeListener != null) {
                    float fraction = (vContentView.getLeft() * 1f) / vMenuView.getWidth();
                    mMenuStateChangeListener.onSliding(fraction);
                }
                //处理开、关状态，由于该方法会不断被回调，所以需要加上状态值，保证只回调一次给监听器
                if ((vMenuView.getLeft() == -vMenuView.getWidth()) && isOpenMenu) {
                    //关
                    isOpenMenu = false;
                    if (mMenuStateChangeListener != null) {
                        mMenuStateChangeListener.onMenuClose();
                    }
                } else if (vMenuView.getLeft() == 0 && !isOpenMenu) {
                    //开
                    isOpenMenu = true;
                    if (mMenuStateChangeListener != null) {
                        mMenuStateChangeListener.onMenuOpen();
                    }
                }
            }
    });
    
        /**
     * 打开菜单
     */
    public void openMenu() {
        mViewDragHelper.smoothSlideViewTo(vMenuView, 0, vMenuView.getTop());
        ViewCompat.postInvalidateOnAnimation(SlidingMenu.this);
    }

    /**
     * 关闭菜单
     */
    public void closeMenu() {
        mViewDragHelper.smoothSlideViewTo(vMenuView, -vMenuView.getWidth(), vMenuView.getTop());
        ViewCompat.postInvalidateOnAnimation(SlidingMenu.this);
    }
}

//...省略其他代码
```

- 处理松手回弹和fling操作

	- 首先，先处理fling操作，当向左快速惯性滑动时，xvel值小于0，关闭菜单。如果是向右，则xvel值大于300则打开菜单。
	- 如果不是fling操作，则处理为松手回台，判断菜单打开时的left时，如果小于菜单宽度的一半，则为关闭，否则为打开。

```
//...省略其他代码

/**
 * 初始化
 */
private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
		mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public void onViewReleased(@NonNull View releasedChild, float xvel, float yvel) {
                super.onViewReleased(releasedChild, xvel, yvel);
                //fling操作
                if (xvel < 0) {
                    //向左
                    closeMenu();
                    return;
                } else if (xvel > 300) {
                    //向右
                    openMenu();
                    return;
                }
                //松手回弹
                float halfMenuWidth = vMenuView.getWidth() / 2f;
                //如果菜单打开的范围小于菜单的一半，则当为关
                if (vMenuView.getLeft() < -halfMenuWidth) {
                    //关
                    closeMenu();
                } else {
                    //开
                    openMenu();
                }
            }
    });
}

//...省略其他代码
```

#### 完整代码

```
public class SlidingMenu extends FrameLayout {
    /**
     * 菜单View
     */
    private View vMenuView;
    /**
     * 内容View
     */
    private View vContentView;
    /**
     * 拽托帮助类
     */
    private ViewDragHelper mViewDragHelper;
    /**
     * 菜单状态改变监听
     */
    private OnMenuStateChangeListener mMenuStateChangeListener;
    /**
     * 菜单是否开启
     */
    private boolean isOpenMenu;

    public SlidingMenu(@NonNull Context context) {
        this(context, null);
    }

    public SlidingMenu(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SlidingMenu(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    /**
     * 初始化
     */
    private void init(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public boolean tryCaptureView(@NonNull View child, int pointerId) {
                //菜单和内容都可以拽托
                return child == vMenuView || child == vContentView;
            }

            @Override
            public int getViewHorizontalDragRange(@NonNull View child) {
                //拽托范围，返回非0值即可，某些情况需要该值来确定是否可以拽托
                return vContentView.getWidth();
            }

            @Override
            public int clampViewPositionHorizontal(@NonNull View child, int left, int dx) {
                //处理横向拽托，获取菜单宽度
                int menuWidth = vMenuView.getWidth();
                //因为拽托菜单和内容传入的对象不同，我们需要处理2种情况
                if (child == vMenuView) {
                    //拽托的是菜单
                    if (left < -menuWidth) {
                        //左边距离，最多只能完全隐藏于屏幕最左侧
                        return -menuWidth;
                    } else if (left > 0) {
                        //左边距离，最多能完全出现在屏幕
                        return 0;
                    } else {
                        return left;
                    }
                } else if (child == vContentView) {
                    //拽托的是内容区域，不能移动超出最左边的屏幕
                    if (left < 0) {
                        return 0;
                    } else if (left > menuWidth) {
                        //最多不能超过菜单的宽度
                        return menuWidth;
                    } else {
                        return left;
                    }
                }
                return 0;
            }

            @Override
            public void onViewPositionChanged(@NonNull View changedView, int left, int top, int dx, int dy) {
                super.onViewPositionChanged(changedView, left, top, dx, dy);
                //处理联动，拽托菜单布局，让内容布局跟着动
                if (changedView == vMenuView) {
                    int newLeft = vContentView.getLeft() + dx;
                    int right = newLeft + vContentView.getWidth();
                    vContentView.layout(newLeft, top, right, getBottom());
                } else if (changedView == vContentView) {
                    //拽托内容布局，让菜单布局跟着动
                    int newLeft = vMenuView.getLeft() + dx;
                    vMenuView.layout(newLeft, top, left, getBottom());
                }
                if (mMenuStateChangeListener != null) {
                    float fraction = (vContentView.getLeft() * 1f) / vMenuView.getWidth();
                    mMenuStateChangeListener.onSliding(fraction);
                }
                //处理开、关状态，由于该方法会不断被回调，所以需要加上状态值，保证只回调一次给监听器
                if ((vMenuView.getLeft() == -vMenuView.getWidth()) && isOpenMenu) {
                    //关
                    isOpenMenu = false;
                    if (mMenuStateChangeListener != null) {
                        mMenuStateChangeListener.onMenuClose();
                    }
                } else if (vMenuView.getLeft() == 0 && !isOpenMenu) {
                    //开
                    isOpenMenu = true;
                    if (mMenuStateChangeListener != null) {
                        mMenuStateChangeListener.onMenuOpen();
                    }
                }
            }

            @Override
            public void onViewReleased(@NonNull View releasedChild, float xvel, float yvel) {
                super.onViewReleased(releasedChild, xvel, yvel);
                //fling操作
                if (xvel < 0) {
                    //向左
                    closeMenu();
                    return;
                } else if (xvel > 300) {
                    //向右
                    openMenu();
                    return;
                }
                //松手回弹
                float halfMenuWidth = vMenuView.getWidth() / 2f;
                //如果菜单打开的范围小于菜单的一半，则当为关
                if (vMenuView.getLeft() < -halfMenuWidth) {
                    //关
                    closeMenu();
                } else {
                    //开
                    openMenu();
                }
            }
        });
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        //获取所有子View，确保只有菜单和内容这2个控件
        int childCount = getChildCount();
        if (childCount != 2) {
            throw new IllegalStateException("侧滑菜单内只能有2个子View，分别是菜单和内容");
        }
        vMenuView = getChildAt(0);
        vContentView = getChildAt(1);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        //布局菜单和内容，菜单在最左边，普通状态是看不到的
        vMenuView.layout(-vMenuView.getMeasuredWidth(), top, left, bottom);
        //内容View铺满整个父控件
        vContentView.layout(left, top, right, bottom);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        //将onInterceptTouchEvent委托给ViewDragHelper
        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //将onTouchEvent委托给ViewDragHelper
        mViewDragHelper.processTouchEvent(event);
        return true;
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        //判断是否移动到头了，未到头则继续
        if (mViewDragHelper != null) {
            if (mViewDragHelper.continueSettling(true)) {
                invalidate();
            }
        }
    }

    /**
     * 打开菜单
     */
    public void openMenu() {
        mViewDragHelper.smoothSlideViewTo(vMenuView, 0, vMenuView.getTop());
        ViewCompat.postInvalidateOnAnimation(SlidingMenu.this);
    }

    /**
     * 关闭菜单
     */
    public void closeMenu() {
        mViewDragHelper.smoothSlideViewTo(vMenuView, -vMenuView.getWidth(), vMenuView.getTop());
        ViewCompat.postInvalidateOnAnimation(SlidingMenu.this);
    }

    /**
     * 菜单状态改变监听
     */
    public interface OnMenuStateChangeListener {
        /**
         * 当菜单开启时回调
         */
        void onMenuOpen();

        /**
         * 正在滑动时回调
         *
         * @param fraction 滑动比值
         */
        void onSliding(float fraction);

        /**
         * 当菜单关闭时回调
         */
        void onMenuClose();
    }

    /**
     * 设置状态改变监听
     *
     * @param menuStateChangeListener 监听器
     */
    public void setOnMenuStateChangeListener(OnMenuStateChangeListener menuStateChangeListener) {
        mMenuStateChangeListener = menuStateChangeListener;
    }
}
```

## 基本使用

### xml布局

- 总布局

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.zh.android.slidingmenu.sample.SlidingMenu
        android:id="@+id/sliding_menu"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <include layout="@layout/layout_menu" />

        <include layout="@layout/layout_content" />
    </com.zh.android.slidingmenu.sample.SlidingMenu>
</FrameLayout>
```

- 菜单布局

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/menu_list"
    android:layout_width="280dp"
    android:layout_height="match_parent" />
```

- 内容布局

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/tool_bar"
            android:layout_width="match_parent"
            android:layout_height="?actionBarSize"
            android:background="@color/colorPrimary"
            app:title="@string/app_name"
            app:titleTextColor="@android:color/white" />

        <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/content_list"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />
    </LinearLayout>

    <View
        android:id="@+id/content_bg"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:alpha="0"
        android:background="#8C000000"
        tools:alpha="1" />
</FrameLayout>
```

### Java代码

```
public class MainActivity extends AppCompatActivity {
    /**
     * 侧滑菜单
     */
    private SlidingMenu vSlidingMenu;

    /**
     * 透明度估值器
     */
    private FloatEvaluator mAlphaEvaluator;
    
        @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findView();
        bindView();
    }

    private void findView() {
        ...省略其他控件
        
        vSlidingMenu = findViewById(R.id.sliding_menu);
        vContentBg = findViewById(R.id.content_bg);
        
        ...省略其他控件
    }
    
    private void bindView() {
        //创建估值器
        mAlphaEvaluator = new FloatEvaluator();
        //------------ 重点：设置侧滑菜单的状态切换监听 ------------
        vSlidingMenu.setOnMenuStateChangeListener(new SlidingMenu.OnMenuStateChangeListener() {
            @Override
            public void onMenuOpen() {
                Log.d(TAG, "菜单打开");
                //让黑色遮罩，禁用触摸
                vContentBg.setClickable(true);
            }

            @Override
            public void onSliding(float fraction) {
                Log.d(TAG, "菜单拽托中，百分比：" + fraction);
                //设定最小、最大透明度值
                float startValue = 0;
                float endValue = 0.55f;
                //估值当前的透明度值，并设置
                Float value = mAlphaEvaluator.evaluate(fraction, startValue, endValue);
                vContentBg.setAlpha(value);
            }

            @Override
            public void onMenuClose() {
                Log.d(TAG, "菜单关闭");
                //让黑色遮罩，恢复触摸
                vContentBg.setClickable(false);
            }
        });
        //------------ 重点：设置侧滑菜单的状态切换监听 ------------
    }
}
```

#### 总结

项目代码，我提交到了github上，有需要的同学可以自行clone：[Github地址](https://github.com/hezihaog/SlidingMenu)

使用ViewDragHelper这个神器，无论是做拽托、移动都很方便，本篇的难点其实在于坐标计算，尤其在onViewPositionChanged()方法中拽托联动菜单和内容这2个部分，计算2个控件的4个点会比较费脑子外，其他计算倒还好。