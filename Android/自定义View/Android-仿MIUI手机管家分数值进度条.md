#### Android-仿MIUI手机管家分数值进度条

这个也是一个进度条，之前在MIUI9上的手机管家上看到，效果是一段弧度，中间一个百分比文字。



#### 原理分析

后续补

#### 完整代码

- 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 圆形CD进度View -->
    <declare-styleable name="CircleProgressWithTextView">
        <!-- 外圆颜色 -->
        <attr name="cpt_circle_color" format="reference|color" />
        <!-- 圆弧颜色 -->
        <attr name="cpt_remain_circle_color" format="reference|color" />
        <!-- 文字颜色 -->
        <attr name="cpt_text_color" format="reference|color" />
        <!--当前进度值-->
        <attr name="cpt_progress" format="integer" />
        <!--进度的最大值-->
        <attr name="cpt_max" format="integer" />
        <!--圆弧轮廓的宽度-->
        <attr name="cpt_remain_circle_border_width" format="dimension" />
        <!-- 是否显示进度百分比文字 -->
        <attr name="cpt_text_enable" format="boolean" />
        <!-- 背景颜色 -->
        <attr name="cpt_bg_color" format="reference|color" />
    </declare-styleable>
</resources>
```

- Java代码

```
public class CircleProgressWithTextView extends View {
    /**
     * 默认值
     */
    /**
     * 默认的当前进度，默认为0
     */
    private static final int DEFAULT_PROGRESS = 0;
    /**
     * 默认的最大值，默认为100
     */
    private static final int DEFAULT_MAX = 100;
    /**
     * 默认进度圆弧颜色
     */
    private final int DEFAULT_CIRCLE_COLOR = Color.parseColor("#0AA4A2");
    /**
     * 圆弧颜色
     */
    private final int DEFAULT_REMAIN_CIRCLE_COLOR = Color.parseColor("#EFEFF0");
    /**
     * 默认背景颜色
     */
    private final int DEFAULT_BG_COLOR = Color.parseColor("#00000000");
    /**
     * 默认文字颜色
     */
    private final int DEFAULT_TEXT_COLOR = Color.parseColor("#0AA4A2");
    /**
     * 是否显示进度百分比文字
     */
    private static final boolean DEFAULT_TEXT_ENABLE = true;
    /**
     * 绘制相关
     */
    private RectF mRect;
    private Paint mCirclePaint;
    private Paint mTextPaint;
    private Paint mPercentPaint;
    /**
     * 画笔颜色
     */
    private int mCircleColor;
    private int mRemainCircleColor;
    private int mTextColor;
    private int mBgColor;
    /**
     * 圆弧宽度
     */
    private float mCircleBorderWidth;
    /**
     * View相关尺寸
     */
    private int mWidth;
    private int mHeight;
    /**
     * 外圆半径
     */
    private float mRadius;
    /**
     * 当前进度
     */
    private float mProgress;
    /**
     * 进度最大值
     */
    private int mMax;
    /**
     * 是否显示百分比文字
     */
    private boolean mEnableText;
    /**
     * 弧线的开始角度，默认是0，是水平的，我们要从上面开始画
     */
    private float mStartAngle = -90f;
    /**
     * 中心点X、Y坐标
     */
    private int mCenterX;
    private int mCenterY;
    /**
     * 进度条进度
     */
    private ValueAnimator mProgressAnimator;
    private OnProgressUpdateListener mProgressUpdateListener;

    public CircleProgressWithTextView(Context context) {
        super(context);
        init(null);
    }

    public CircleProgressWithTextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init(attrs);
    }

    public CircleProgressWithTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(attrs);
    }

    /**
     * 初始化自定义属性
     */
    private void initAttributeVar(AttributeSet attrs) {
        //默认圆弧宽度
        int defaultCircleBorderWidth = dip2px(getContext(), 5f);
        if (attrs != null) {
            TypedArray array = getContext().obtainStyledAttributes(attrs, R.styleable.CircleProgressWithTextView);
            mProgress = array.getInt(R.styleable.CircleProgressWithTextView_cpt_progress, DEFAULT_PROGRESS);
            mMax = array.getInt(R.styleable.CircleProgressWithTextView_cpt_max, DEFAULT_MAX);
            //Xml设置的进度圆弧颜色
            mCircleColor = array.getColor(R.styleable.CircleProgressWithTextView_cpt_circle_color, DEFAULT_CIRCLE_COLOR);
            //Xml设置的圆弧颜色
            mRemainCircleColor = array.getColor(R.styleable.CircleProgressWithTextView_cpt_remain_circle_color, DEFAULT_REMAIN_CIRCLE_COLOR);
            //文字颜色
            mTextColor = array.getColor(R.styleable.CircleProgressWithTextView_cpt_text_color, DEFAULT_CIRCLE_COLOR);
            //读取设置的圆弧轮廓宽度，读取dimension
            mCircleBorderWidth = array.getDimensionPixelSize(R.styleable.CircleProgressWithTextView_cpt_remain_circle_border_width, defaultCircleBorderWidth);
            //是否显示进度百分比文字
            mEnableText = array.getBoolean(R.styleable.CircleProgressWithTextView_cpt_text_enable, DEFAULT_TEXT_ENABLE);
            //背景颜色
            mBgColor = array.getColor(R.styleable.CircleProgressWithTextView_cpt_bg_color, DEFAULT_BG_COLOR);
            array.recycle();
        } else {
            //没有在Xml中设置属性，使用默认属性
            mProgress = DEFAULT_PROGRESS;
            mMax = DEFAULT_MAX;
            mCircleColor = DEFAULT_CIRCLE_COLOR;
            mRemainCircleColor = DEFAULT_REMAIN_CIRCLE_COLOR;
            mTextColor = DEFAULT_TEXT_COLOR;
            mCircleBorderWidth = defaultCircleBorderWidth;
            mEnableText = DEFAULT_TEXT_ENABLE;
        }
    }

    private void init(AttributeSet attrs) {
        initAttributeVar(attrs);
        //外圆画笔
        mCirclePaint = new Paint();
        mCirclePaint.setColor(mCircleColor);
        mCirclePaint.setStrokeWidth(mCircleBorderWidth);
        mCirclePaint.setStyle(Paint.Style.STROKE);
        //设置笔触为圆角
        mCirclePaint.setStrokeCap(Paint.Cap.ROUND);
        mCirclePaint.setAntiAlias(true);
        //文字画笔
        mTextPaint = new Paint();
        mTextPaint.setColor(mTextColor);
        mTextPaint.setStrokeWidth(dip2px(getContext(), 1f));
        mTextPaint.setStyle(Paint.Style.FILL);
        mTextPaint.setTextSize(sp2px(getContext(), 17f));
        mTextPaint.setAntiAlias(true);
        //百分比画笔
        mPercentPaint = new Paint();
        mPercentPaint.setColor(mTextColor);
        mPercentPaint.setStrokeWidth(dip2px(getContext(), 1f));
        mPercentPaint.setStyle(Paint.Style.FILL);
        mPercentPaint.setTextSize(sp2px(getContext(), 13f));
        mPercentPaint.setAntiAlias(true);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        //控件的总宽高
        mWidth = w;
        mHeight = h;
        //取出padding值
        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        int paddingTop = getPaddingTop();
        int paddingBottom = getPaddingBottom();
        //绘制范围
        mRect = new RectF();
        mRect.left = (float) paddingLeft;
        mRect.top = (float) paddingTop;
        mRect.right = (float) mWidth - paddingRight;
        mRect.bottom = (float) mHeight - paddingBottom;
        //计算直径和半径
        float diameter = (Math.min(mWidth, mHeight)) - paddingLeft - paddingRight;
        mRadius = (float) ((diameter / 2) * 0.98);
        //计算圆心的坐标
        mCenterX = mWidth / 2;
        mCenterY = mHeight / 2;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(measureSpec(widthMeasureSpec), measureSpec(heightMeasureSpec));
    }

    private int measureSpec(int measureSpec) {
        int result;
        int mode = MeasureSpec.getMode(measureSpec);
        int size = MeasureSpec.getSize(measureSpec);
        //默认大小
        int defaultSize = dip2px(getContext(), 55f);
        //指定宽高则直接返回
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else if (mode == MeasureSpec.AT_MOST) {
            //wrap_content的情况
            result = Math.min(defaultSize, size);
        } else {//未指定，则使用默认的大小
            result = defaultSize;
        }
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.scale(0.93f, 0.93f, mCenterX, mCenterY);
        drawBg(canvas);
        float curProgress = getProgress();
        //画圆弧
        drawCircle(canvas, curProgress);
        if (mEnableText) {
            //画文字
            String progressText = String.valueOf((int) curProgress);
            drawProgressText(canvas, progressText);
        }
    }

    /**
     * 画背景
     */
    private void drawBg(Canvas canvas) {
        canvas.drawColor(mBgColor);
    }

    /**
     * 画圆弧
     */
    private void drawCircle(Canvas canvas, float curProgress) {
        mCirclePaint.setColor(mRemainCircleColor);
        canvas.drawCircle(mCenterX, mCenterY, mRadius, mCirclePaint);
        //绘制当前进度的弧线
        mCirclePaint.setColor(mCircleColor);
        float angle = 360 * (curProgress * 1.0f / getMax());
        canvas.drawArc(mRect, mStartAngle, angle, false, mCirclePaint);
        //绘制剩下的度数的弧线
        //float remainAngle = 360f - angle;
        //mCirclePaint.setColor(mRemainCircleColor);
        //canvas.drawArc(mRect, mStartAngle + angle, remainAngle, false, mCirclePaint);
    }

    /**
     * 画进度文字
     */
    private void drawProgressText(Canvas canvas, String progressText) {
        Paint.FontMetrics fontMetrics = mTextPaint.getFontMetrics();
        float baseLine = -(fontMetrics.ascent + fontMetrics.descent) / 2;
        float textWidth = mTextPaint.measureText(progressText);
        float startX = mCenterX - (textWidth / 2);
        float endY = mCenterY + baseLine;
        canvas.drawText(progressText, startX, endY, mTextPaint);
        //画百分比
        String percentText = "%";
        //计算百分比文字的起始X坐标
        float percentStartX = 0;
        if (progressText.length() == 1) {
            percentStartX = mCenterX + dip2px(getContext(), 5f);
        } else if (progressText.length() == 2) {
            percentStartX = mCenterX + dip2px(getContext(), 10f);
        }
        //百分比文字Y坐标，中心点Y坐标，和数值文字的Y坐标一样
        float percentEndY = mCenterY + baseLine;
        canvas.drawText(percentText, percentStartX, percentEndY, mPercentPaint);
    }

    /**
     * 指定时间，开始进度
     *
     * @param preProgress 之前的进度
     * @param duration    执行时间
     */
    public void startProgressByTime(int preProgress, long duration) {
        if (mProgressAnimator == null) {
            mProgressAnimator = ValueAnimator.ofInt(preProgress, mMax);
        }
        mProgressAnimator.setInterpolator(new LinearInterpolator());
        mProgressAnimator.setDuration(duration);
        mProgressAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                Integer cValue = (Integer) animation.getAnimatedValue();
                setProgress(cValue);
                if (mProgressUpdateListener != null) {
                    mProgressUpdateListener.onProgressUpdate(cValue);
                }
            }
        });
        mProgressAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                if (mProgressUpdateListener != null) {
                    mProgressUpdateListener.onStart();
                }
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                //结束，设置回0
                setProgress(0);
                if (mProgressUpdateListener != null) {
                    mProgressUpdateListener.onEnd();
                }
            }
        });
        mProgressAnimator.start();
    }

    /**
     * 进度更新监听
     */
    public interface OnProgressUpdateListener {
        void onStart();

        /**
         * 进度更新
         *
         * @param curProgress 当前进度
         */
        void onProgressUpdate(int curProgress);

        void onEnd();
    }

    public static class OnProgressUpdateAdapter implements OnProgressUpdateListener {

        @Override
        public void onStart() {
        }

        @Override
        public void onProgressUpdate(int curProgress) {
        }

        @Override
        public void onEnd() {
        }
    }

    public void setOnProgressUpdateListener(OnProgressUpdateListener progressUpdateListener) {
        mProgressUpdateListener = progressUpdateListener;
    }

    public float getProgress() {
        return mProgress;
    }

    public void setProgress(float mProgress) {
        this.mProgress = mProgress;
        postInvalidate();
    }

    public float getMax() {
        return mMax;
    }

    public void setMax(int max) {
        this.mMax = max;
        postInvalidate();
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }

    public static int px2dp(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }

    private int sp2px(Context context, float spVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,
                spVal, context.getResources().getDisplayMetrics());
    }
}
```

#### 示例代码

- Xml布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.hzh.circle.progress.sample.MainActivity">

    <com.hzh.circle.progress.sample.widget.CircleProgressWithTextView
        android:id="@+id/circleProgress"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_centerInParent="true"
        android:paddingLeft="10dp"
        android:paddingTop="10dp"
        android:paddingRight="10dp"
        android:paddingBottom="10dp"
        app:cpt_circle_color="#0AA4A2"
        app:cpt_text_color="#0AA4A2"
        app:cpt_max="100"
        app:cpt_progress="30"
        app:cpt_remain_circle_border_width="3dp"
        app:cpt_remain_circle_color="#EFEFF0"
        app:cpt_text_enable="true" />
</RelativeLayout>
```

- Java代码

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final CircleProgressWithTextView circleProgress = findViewById(R.id.circleProgress);

        //使用值动画，不断更新进度
        ValueAnimator animator = ValueAnimator.ofFloat(0, 100);
        animator.setInterpolator(new LinearInterpolator());
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.setRepeatMode(ValueAnimator.RESTART);
        animator.setDuration(3000);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                Float cValue = (Float) animation.getAnimatedValue();
                circleProgress.setProgress(cValue);
            }
        });
        animator.start();
    }
}
```