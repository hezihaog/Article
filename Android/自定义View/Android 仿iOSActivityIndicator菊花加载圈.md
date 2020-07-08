#### Android 仿iOSActivityIndicator菊花加载圈

iOS上有一个UIActivityIndicator的控件，就是俗称转菊花的控件，一般UI设计师会按照iOS的风格来出设计稿，也要求使用这种Loading效果，实现方式有很多：

- 切图，做旋转动画
- 自定义View，绘制效果
- gif图

1. 切图会增加体积，但相对简单，不过在换肤的场景下，会使用不同颜色，需要准备多张图，不够灵活。
2. 由于自定义的好处，不同颜色只需要提供自定义属性，换肤时切换属性设置即可，比较灵活。
3. gif图普遍比较大，而且加载gif没有原生支持，需要引入第三方库，而且消耗内存比较大，不推荐。

![Android 仿iOSActivityIndicator菊花加载圈.jpeg](https://upload-images.jianshu.io/upload_images/1641428-f9b80c6d9f0d5a09.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

#### 原理分析

后续补

#### 完整代码

- 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="ActivityIndicatorView">
        <!--  菊花颜色  -->
        <attr name="aiv_color" format="color" />
        <!--  开始角度  -->
        <attr name="aiv_startAngle" format="integer" />
        <!--  线条宽度  -->
        <attr name="aiv_strokeWidth" format="dimension" />
        <!--  是否自动开始  -->
        <attr name="aiv_auto_start" format="boolean" />
    </declare-styleable>
</resources>
```

- Java代码

```
public class ActivityIndicatorView extends View {
    /**
     * 菊花颜色
     */
    private int mColor = Color.argb(255, 255, 255, 255);
    /**
     * 开始角度
     */
    private int mStartAngle = 0;
    /**
     * 每条线的宽度
     */
    private float mStrokeWidth = 0;
    /**
     * 是否自动开始旋转
     */
    private boolean isAutoStart;
    /**
     * 一共多少条先
     */
    private final int mLineCount = 12;
    private final int minAlpha = 0;
    /**
     * 每一条线叠加的
     */
    private final int mAngleGradient = 360 / mLineCount;
    /**
     * 每条线的颜色
     */
    private int[] mColors = new int[mLineCount];
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 动画Handler
     */
    private Handler mAnimHandler = new Handler(Looper.getMainLooper());
    /**
     * 动画任务
     */
    private Runnable mAnimRunnable = new Runnable() {
        @Override
        public void run() {
            mStartAngle += mAngleGradient;
            invalidate();
            mAnimHandler.postDelayed(mAnimRunnable, 50);
        }
    };

    public ActivityIndicatorView(Context context) {
        this(context, null);
    }

    public ActivityIndicatorView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ActivityIndicatorView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setup(context, attrs, defStyleAttr, 0);
    }

    private void setup(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.ActivityIndicatorView, defStyleAttr, defStyleRes);
        mColor = typedArray.getColor(R.styleable.ActivityIndicatorView_aiv_color, mColor);
        mStartAngle = typedArray.getInt(R.styleable.ActivityIndicatorView_aiv_startAngle, mStartAngle);
        mStrokeWidth = typedArray.getDimension(R.styleable.ActivityIndicatorView_aiv_strokeWidth, mStrokeWidth);
        isAutoStart = typedArray.getBoolean(R.styleable.ActivityIndicatorView_aiv_auto_start, true);
        typedArray.recycle();
        initialize();
    }

    private void initialize() {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        int alpha = Color.alpha(mColor);
        int red = Color.red(mColor);
        int green = Color.green(mColor);
        int blue = Color.blue(mColor);
        int alphaGradient = Math.abs(alpha - minAlpha) / mLineCount;
        for (int i = 0; i < mColors.length; i++) {
            mColors[i] = Color.argb(alpha - alphaGradient * i, red, green, blue);
        }
        mPaint.setStrokeCap(Paint.Cap.ROUND);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int centerX = getWidth() / 2;
        int centerY = getHeight() / 2;
        float radius = Math.min(getWidth() - getPaddingLeft() - getPaddingRight(), getHeight() - getPaddingTop() - getPaddingBottom()) * 0.5f;
        if (mStrokeWidth == 0) {
            mStrokeWidth = pointX(mAngleGradient / 2, radius / 2) / 2;
        }
        mPaint.setStrokeWidth(mStrokeWidth);
        for (int i = 0; i < mColors.length; i++) {
            mPaint.setColor(mColors[i]);
            canvas.drawLine(
                    centerX + pointX(-mAngleGradient * i + mStartAngle, radius / 2),
                    centerY + pointY(-mAngleGradient * i + mStartAngle, radius / 2),
                    //这里计算Y值时, 之所以减去线宽/2, 是防止没有设置的Padding时,图像会超出View范围
                    centerX + pointX(-mAngleGradient * i + mStartAngle, radius - mStrokeWidth / 2),
                    //这里计算Y值时, 之所以减去线宽/2, 是防止没有设置的Padding时,图像会超出View范围
                    centerY + pointY(-mAngleGradient * i + mStartAngle, radius - mStrokeWidth / 2),
                    mPaint);
        }
    }

    private float pointX(int angle, float radius) {
        return (float) (radius * Math.cos(angle * Math.PI / 180));
    }

    private float pointY(int angle, float radius) {
        return (float) (radius * Math.sin(angle * Math.PI / 180));
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        if (isAutoStart) {
            start();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        if (mAnimHandler != null) {
            stop();
        }
    }

    /**
     * 开始动画
     */
    public void start() {
        mAnimHandler.post(mAnimRunnable);
    }

    /**
     * 停止动画
     */
    public void stop() {
        mAnimHandler.removeCallbacks(mAnimRunnable);
    }

    /**
     * 设置菊花颜色
     */
    public void setColor(int color) {
        this.mColor = color;
    }

    /**
     * 设置线宽
     */
    public void setStrokeWidth(float strokeWidth) {
        this.mStrokeWidth = strokeWidth;
    }

    /**
     * 设置开始的角度
     */
    public void setStartAngle(int startAngle) {
        this.mStartAngle = startAngle;
    }
}
```

#### 布局代码

布局中添加控件，适当时候显示、隐藏即可，最好都设置宽、高、颜色以及线宽，默认效果比较不符合使用。

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        
    <com.tongwei.bootrunsample.base.widget.ActivityIndicatorView
        android:id="@+id/activity_indicator"
        android:layout_width="25dp"
        android:layout_height="25dp"
        android:layout_gravity="center"
        android:layout_centerInParent="true"
        app:aiv_color="#A2A3B0"
        app:aiv_strokeWidth="2.5dp" />
</RelativeLayout>
```