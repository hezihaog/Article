#### Android 渐变圆环加载View

渐变圆环Loading效果，淘宝、微信都有用，今天写了一个类似的自定义View。

![自定义圆环.png](https://upload-images.jianshu.io/upload_images/1641428-d69f6b0795f8233d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理分析

提供开始、结束的颜色，绘制一个渐变的圆环，定时调用invalidate重绘，并旋转画布，我使用View的postDelayed()达到定时重绘，如果需要差值器可以使用动画的形式来做。

#### 完整代码

- 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="RingLoadingView">
        <!-- 圆环开始颜色 -->
        <attr name="rlv_start_color" format="color|reference" />
        <!-- 圆环结束颜色 -->
        <attr name="rlv_end_color" format="color|reference" />
        <!-- 圆环的边宽 -->
        <attr name="rlv_ring_width" format="dimension|reference|float" />
        <!-- 是否自动开启旋转，默认开启 -->
        <attr name="rlv_auto_start" format="boolean" />
    </declare-styleable>
</resources>
```

- Java代码

```
public class RingLoadingView extends View implements Runnable {
    /**
     * 默认开始颜色，灰色
     */
    private final int mDefaultStartColor = Color.argb(255, 180, 180, 180);
    /**
     * 默认结束颜色，白色
     */
    private final int mDefaultEndColor = Color.argb(255, 255, 255, 255);

    /**
     * 圆环的开始、结束颜色
     */
    private int[] mColors;
    /**
     * 圆环宽度
     */
    private float mRingWidth;
    /**
     * 是否自动开始
     */
    private boolean isAutoStart;

    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 100;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;
    /**
     * 间隔时间
     */
    private static final int INTERVAL_TIME = 15;
    /**
     * 总旋转角度
     */
    private static final int TOTAL_ROTATION_ANGLE = 360;
    /**
     * 当前旋转到的角度
     */
    private int mCurrentAngle = 1;
    /**
     * 半径
     */
    private float mRadius;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 圆环位置
     */
    private RectF mCircleRect;

    public RingLoadingView(Context context) {
        this(context, null);
    }

    public RingLoadingView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RingLoadingView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(mRingWidth);
        mPaint.setAntiAlias(true);
        //设置渐变着色
        mPaint.setShader(new SweepGradient(0, 0, mColors, null));
    }

    /**
     * 初始化自定义属性
     */
    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.RingLoadingView, defStyleAttr, 0);
        int startColor = array.getColor(R.styleable.RingLoadingView_rlv_start_color, mDefaultStartColor);
        int endColor = array.getColor(R.styleable.RingLoadingView_rlv_end_color, mDefaultEndColor);
        mColors = new int[]{startColor, endColor};
        mRingWidth = array.getDimensionPixelSize(R.styleable.RingLoadingView_rlv_ring_width, dip2px(context, 2f));
        isAutoStart = array.getBoolean(R.styleable.RingLoadingView_rlv_auto_start, true);
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //圆环半径，取宽、高中最小值
        mRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.8f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //旋转画布，让圆环旋转起来
        canvas.rotate(mCurrentAngle, 0, 0);
        //画渐变圆环
        if (mCircleRect == null) {
            //圆环位置，因为坐标系移动到控件中心，所以left、top都要为负值
            mCircleRect = new RectF(-mRadius, -mRadius, mRadius, mRadius);
        }
        canvas.drawArc(mCircleRect, 0, 360, false, mPaint);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(handleMeasure(widthMeasureSpec), handleMeasure(heightMeasureSpec));
    }

    /**
     * 处理MeasureSpec
     */
    private int handleMeasure(int measureSpec) {
        int result = DEFAULT_MIN_WIDTH;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            //处理wrap_content的情况
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        //自动开启旋转
        if (isAutoStart) {
            start();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        stop();
    }

    @Override
    public void run() {
        if (mCurrentAngle >= TOTAL_ROTATION_ANGLE) {
            mCurrentAngle = mCurrentAngle - TOTAL_ROTATION_ANGLE;
        } else {
            //每次叠加10步长
            mCurrentAngle += 10;
        }
        //通知重绘
        invalidate();
        postDelayed(this, INTERVAL_TIME);
    }

    private void start() {
        postDelayed(this, INTERVAL_TIME);
    }

    private void stop() {
        removeCallbacks(this);
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }
}
```

#### 布局代码

```
<com.zh.cavas.sample.RingLoadingView
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_marginTop="20dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/loading"
    app:rlv_auto_start="true"
    app:rlv_end_color="#FFFFFF"
    app:rlv_ring_width="2.3dp"
    app:rlv_start_color="#B4B4B4" />
```