#### ViewDragHelper实战，悬浮按钮拖动，可用于活动入口

在应用商店、京东、游戏中心，右下角都有一个悬浮的按钮，可能是可以拖动的，一般用于广告或活动。本篇来用ViewDragHelper来做悬浮按钮的拽托，并处理fling（惯性滑动）。

![悬浮按钮拽托.gif](https://upload-images.jianshu.io/upload_images/1641428-4651ccba3c1b704e.gif?imageMogr2/auto-orient/strip)

ViewDragHelper每个方法的分析，在上一篇博客[ViewDragHelper实战，实现滑动解锁](https://www.jianshu.com/p/81b329788511)中有分析，本篇则不再赘述了，推荐先看上篇，再回来看本篇，如果理解了上篇，本篇就很容易理解了。

#### 原理分析

1. 我们在ViewDragHelper中提供的回调中，处理悬浮按钮的移动边界，不允许超出父布局。
2. 松手回弹处理，在onViewReleased()方法中判断，松手时的坐标位于屏幕一半的左侧，还是右侧，决定回弹到哪一边，使用ViewDragHelper的settleCapturedViewAt()方法进行弹性移动。
3. fling操作处理，判断移动的距离是否小于固定值，并且速度小于指定速度，则当为fling操作，判断滑动方法是左右，还是上下，如果是左右，再惯性滑动到哪一侧。

主要复杂的地方在onViewReleased()，处理fling操作时代码比较多，如果不处理fling，只判断松手位置在屏幕一半的哪一边，代码量就只有3分之一。

#### 完整代码

- 约定id

由于我们要捕获子View，而布局中允许有多个子View，所以我们约定可拖动的按钮的id为float_button。

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="float_button" type="id"/>
</resources>
```

- 自定义View

重点都在FloatButtonLayout类中了，实现过程中发现，如果给悬浮按钮设置了OnClick点击事件，会导致无法拖动，估计是Down事件被悬浮按钮拦截了导致。为了处理这个问题，我在类中也判断了是否是点击操作，提供了回调设置，通过setCallback()，设置点击监听，代替原生onClick()点击监听即可。

```
public class FloatButtonLayout extends FrameLayout {
    /**
     * 可拽托按钮
     */
    private View mFloatButton;
    /**
     * 拽托帮助类
     */
    private ViewDragHelper mViewDragHelper;
    /**
     * 回调
     */
    private Callback mCallback;

    public FloatButtonLayout(@NonNull Context context) {
        this(context, null);
    }

    public FloatButtonLayout(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public FloatButtonLayout(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, AttributeSet attrs, int defStyleAttr) {
        mViewDragHelper = ViewDragHelper.create(this, 0.3f, new ViewDragHelper.Callback() {
            /**
             * 开始拽托时的X坐标
             */
            private int mDownX;
            /**
             * 开始拽托时的Y坐标
             */
            private int mDownY;
            /**
             * 开始拽托时的时间
             */
            private long mDownTime;

            @Override
            public boolean tryCaptureView(@NonNull View child, int pointerId) {
                return child == mFloatButton;
            }

            @Override
            public int clampViewPositionHorizontal(@NonNull View child, int left, int dx) {
                //限制左右移动的返回，不能超过父控件
                int leftBound = getPaddingStart();
                int rightBound = getMeasuredWidth() - getPaddingEnd() - child.getWidth();
                if (left < leftBound) {
                    return leftBound;
                }
                if (left > rightBound) {
                    return rightBound;
                }
                return left;
            }

            @Override
            public int clampViewPositionVertical(@NonNull View child, int top, int dy) {
                //限制上下移动的返回，不能超过父控件
                int topBound = getPaddingTop();
                int bottomBound = getMeasuredHeight() - getPaddingBottom() - child.getHeight();
                if (top < topBound) {
                    return topBound;
                }
                if (top > bottomBound) {
                    return bottomBound;
                }
                return top;
            }

            @Override
            public int getViewHorizontalDragRange(@NonNull View child) {
                return getMeasuredWidth() - getPaddingStart() - getPaddingEnd() - child.getWidth();
            }

            @Override
            public int getViewVerticalDragRange(@NonNull View child) {
                return getMeasuredHeight() - getPaddingTop() - getPaddingBottom() - child.getHeight();
            }

            @Override
            public void onViewCaptured(@NonNull View capturedChild, int activePointerId) {
                super.onViewCaptured(capturedChild, activePointerId);
                mDownX = capturedChild.getLeft();
                mDownY = capturedChild.getTop();
                mDownTime = System.currentTimeMillis();
            }

            @Override
            public void onViewReleased(@NonNull final View releasedChild, float xvel, float yvel) {
                super.onViewReleased(releasedChild, xvel, yvel);
                //松手回弹，判断如果松手位置，近左边还是右边，进行弹性滑动
                int fullWidth = getMeasuredWidth();
                final int halfWidth = fullWidth / 2;
                final int currentLeft = releasedChild.getLeft();
                final int currentTop = releasedChild.getTop();
                //滚动到左边
                final Runnable scrollToLeft = new Runnable() {
                    @Override
                    public void run() {
                        mViewDragHelper.settleCapturedViewAt(getPaddingStart(), currentTop);
                    }
                };
                //滚动到右边
                final Runnable scrollToRight = new Runnable() {
                    @Override
                    public void run() {
                        int endX = getMeasuredWidth() - getPaddingEnd() - releasedChild.getWidth();
                        mViewDragHelper.settleCapturedViewAt(endX, currentTop);
                    }
                };
                Runnable checkDirection = new Runnable() {
                    @Override
                    public void run() {
                        if (currentLeft < halfWidth) {
                            //在屏幕一半的左边，回弹回左边
                            scrollToLeft.run();
                        } else {
                            //在屏幕一半的右边，回弹回右边
                            scrollToRight.run();
                        }
                    }
                };
                //最小移动距离
                int minMoveDistance = fullWidth / 3;
                //计算移动距离
                int distanceX = currentLeft - mDownX;
                int distanceY = currentTop - mDownY;
                long upTime = System.currentTimeMillis();
                //间隔时间
                long intervalTime = upTime - mDownTime;
                float touched = getDistanceBetween2Points(new PointF(mDownX, mDownY), new PointF(currentLeft, currentTop));
                //处理点击事件，移动距离小于识别为移动的距离，并且时间小于400
                if (touched < mViewDragHelper.getTouchSlop() && intervalTime < 300) {
                    if (mCallback != null) {
                        mCallback.onClickFloatButton();
                    }
                    //因为判断为点击事件后，return就会让按钮不进行贴边回弹了，这里再添加处理，让可以贴边回弹
                    checkDirection.run();
                    return;
                }
                //判断上下滑还是左右滑
                if (Math.abs(distanceX) > Math.abs(distanceY)) {
                    //左右滑，滑动得少，并且速度很快，则为fling操作
                    if (Math.abs(distanceX) < minMoveDistance &&
                            Math.abs(xvel) > Math.abs(mViewDragHelper.getMinVelocity())) {
                        //距离相减为正数，则为往右滑
                        if (distanceX > 0) {
                            scrollToRight.run();
                        } else {
                            //否则为往左
                            scrollToLeft.run();
                        }
                    } else {
                        //不是fling操作，判断松手位置在屏幕左边还是右边
                        checkDirection.run();
                    }
                } else {
                    //上下滑，主要是判断在屏幕左还是屏幕右，不需要判断fling
                    checkDirection.run();
                }
                invalidate();
            }
        });
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mViewDragHelper.processTouchEvent(event);
        return true;
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mViewDragHelper != null && mViewDragHelper.continueSettling(true)) {
            invalidate();
        }
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mFloatButton = findViewById(R.id.float_button);
        if (mFloatButton == null) {
            throw new NullPointerException("必须要有一个可拽托按钮");
        }
    }

    /**
     * 获得两点之间的距离
     */
    public static float getDistanceBetween2Points(PointF p0, PointF p1) {
        return (float) Math.sqrt(Math.pow(p0.y - p1.y, 2) + Math.pow(p0.x - p1.x, 2));
    }

    public interface Callback {
        /**
         * 点击时回调
         */
        void onClickFloatButton();
    }

    public void setCallback(Callback callback) {
        mCallback = callback;
    }
}
```

#### 具体使用

- 布局中添加，包裹可拖动的悬浮按钮

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="可拖动移动按钮，点击按钮跳转活动页"
        android:textColor="@android:color/black"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.zh.android.floatbutton.weiget.FloatButtonLayout
        android:id="@+id/float_button_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:id="@id/float_button"
            android:layout_width="58dp"
            android:layout_height="58dp"
            android:src="@mipmap/ic_launcher" />
    </com.zh.android.floatbutton.weiget.FloatButtonLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

- Java代码，设置点击事件

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        FloatButtonLayout floatButton = findViewById(R.id.float_button_layout);
        //设置点击事件，跳转活动页面
        floatButton.setCallback(new FloatButtonLayout.Callback() {
            @Override
            public void onClickFloatButton() {
                startActivity(new Intent(MainActivity.this, NewYearActivity.class));
            }
        });
    }
}
```

#### 代码Github地址

[Github地址](https://github.com/hezihaog/FloatButton)