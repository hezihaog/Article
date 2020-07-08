#### 反射设置ViewPager的Scroller，改变切换速率

当我们调用ViewPager的setCurrentItem(item)时，切换Item会有点生硬，一般做Banner轮播图的时候也会使用ViewPager，而ViewPager默认的滚动速度太生硬，我们需要做到vivo商店的Banner的速度，先快，后慢。

![vivo商店ViewPager效果.gif](https://upload-images.jianshu.io/upload_images/1641428-53baa49b3ab5768e.gif?imageMogr2/auto-orient/strip)

查看ViewPager源码，发现ViewPager滚动页面是使用Scroller，而Scroller的速度是通过设置Interpolator插值器去实现的。

#### ViewPager源码

切换到指定位置的页面，我们会使用setCurrentItem()，这个方法，内部最终会调用这个smoothScrollTo()方法，内部会调用Scroller进行滚动，并且ViewPager复写了computeScroll()来持续滚动。

```
public class ViewPager extends ViewGroup {
	private Scroller mScroller;
	
	private static final Interpolator sInterpolator = new Interpolator() {
        @Override
        public float getInterpolation(float t) {
            t -= 1.0f;
            return t * t * t * t * t + 1.0f;
        }
    };
    
    public ViewPager(@NonNull Context context) {
        super(context);
        initViewPager();
    }

    public ViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initViewPager();
    }
    
    //初始化ViewPager
    void initViewPager() {
        final Context context = getContext();
        mScroller = new Scroller(context, sInterpolator);
    }
    
    //我们一般调用setCurrentItem()切换页面
    public void setCurrentItem(int item) {
        mPopulatePending = false;
        //调用这个方法实现
        setCurrentItemInternal(item, !mFirstLayout, false);
    }
    
    void setCurrentItemInternal(int item, boolean smoothScroll, boolean always) {
        //调用另外一个重载方法
        setCurrentItemInternal(item, smoothScroll, always, 0);
    }
    
    void setCurrentItemInternal(int item, boolean smoothScroll, boolean always, int velocity) {
        ...省略其他预先处理代码
        //调用滚动到指定条目
        scrollToItem(item, smoothScroll, velocity, dispatchSelected);
    }
    
    private void scrollToItem(int item, boolean smoothScroll, int velocity,
            boolean dispatchSelected) {
        ...省略其他预先处理代码
        //调用缓慢滚动
        smoothScrollTo(destX, 0, velocity);
    }
    
    void smoothScrollTo(int x, int y, int velocity) {
        //开始滚动，还需要复写computeScroll来持续滚动
        mScroller.startScroll(sx, sy, dx, dy, duration);
        ViewCompat.postInvalidateOnAnimation(this);
    }
    
    @Override
    public void computeScroll() {
        mIsScrollStarted = true;
        //判断是否滚动到了终点
        if (!mScroller.isFinished() && mScroller.computeScrollOffset()) {
            //没有滚动完，继续滚动
            int oldX = getScrollX();
            int oldY = getScrollY();
            int x = mScroller.getCurrX();
            int y = mScroller.getCurrY();

            if (oldX != x || oldY != y) {
                scrollTo(x, y);
                if (!pageScrolled(x)) {
                    mScroller.abortAnimation();
                    scrollTo(0, y);
                }
            }

            //通知重绘，继续滚动
            ViewCompat.postInvalidateOnAnimation(this);
            return;
        }

        //滚动结束
        completeScroll(true);
    }
}
```

#### 设置Scroller和Interpolator

查找ViewPager的方法，发现并没有类似于setScroller()，setScrollerInterpolator()，这种公开的API。

没有公共API可以使用，那么我们只能使用反射，进行Hook，我们只要ViewPager有个私有属性mScroller，就是Scroller的实例，只要我们反射替换掉mScroller变量，设置为我们有特定Interpolator的Scroller，就可以达到不同速度的切换效果。

- 准备Scroller

FixedSpeedScroller这个类，我是从系统某个类的内部类中拷贝出来了，具体哪里也不太记得了，他比较符合我们的需求，是一个先快后慢的效果。Scroller的切换速度取之于Interpolator，所以我们还有一个ViscousFluidInterpolator。

```
public static class FixedSpeedScroller extends Scroller {
    private int mDuration = 1500;

    public FixedSpeedScroller(Context context) {
        super(context);
    }

    public FixedSpeedScroller(Context context, Interpolator interpolator) {
        super(context, interpolator);
    }

    @Override
    public void startScroll(int startX, int startY, int dx, int dy, int duration) {
        super.startScroll(startX, startY, dx, dy, mDuration);
    }

    @Override
    public void startScroll(int startX, int startY, int dx, int dy) {
        super.startScroll(startX, startY, dx, dy, mDuration);
    }

    public int getmDuration() {
        return mDuration;
    }

    public void setmDuration(int time) {
        mDuration = time;
    }
}
```

- 快进慢出Interpolator，ViscousFluidInterpolator

```
public class ViscousFluidInterpolator implements Interpolator {
    /**
     * Controls the viscous fluid effect (how much of it).
     */
    private static final float VISCOUS_FLUID_SCALE = 8.0f;

    private static final float VISCOUS_FLUID_NORMALIZE;
    private static final float VISCOUS_FLUID_OFFSET;

    static {

        // must be set to 1.0 (used in viscousFluid())
        VISCOUS_FLUID_NORMALIZE = 1.0f / viscousFluid(1.0f);
        // account for very small floating-point error
        VISCOUS_FLUID_OFFSET = 1.0f - VISCOUS_FLUID_NORMALIZE * viscousFluid(1.0f);
    }

    private static float viscousFluid(float x) {
        x *= VISCOUS_FLUID_SCALE;
        if (x < 1.0f) {
            x -= (1.0f - (float) Math.exp(-x));
        } else {
            float start = 0.36787944117f;   // 1/e == exp(-1)
            x = 1.0f - (float) Math.exp(1.0f - x);
            x = start + x * (1.0f - start);
        }
        return x;
    }

    @Override
    public float getInterpolation(float input) {
        final float interpolated = VISCOUS_FLUID_NORMALIZE * viscousFluid(input);
        if (interpolated > 0) {
            return interpolated + VISCOUS_FLUID_OFFSET;
        }
        return interpolated;
    }
}
```

- 反射设置ViewPager的Scroller

```
Context context = getContext();
ViewPager viewPager = findViewById(R.id.view_pager);
try {
    //获取原本的mScroller变量
    Field mField = ViewPager.class.getDeclaredField("mScroller");
    //由于是private修饰符，设置允许访问
    mField.setAccessible(true);
    //创建快进慢出效果的Scroller
    FixedSpeedScroller scroller = new FixedSpeedScroller(context, new ViscousFluidInterpolator());
    //反射替换掉mScroller变量为我们的FixedSpeedScroller
    mField.set(viewPager, scroller);
} catch (Exception e) {
    e.printStackTrace();
}
```