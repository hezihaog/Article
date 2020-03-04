#### Android-仿微博加载饼形圈

之前写过一个仿微博Loading加载的饼图圈自定义View，一直放在磁盘里面，现在拿出来分享给有需要的小伙伴~

![Android-仿微博加载饼形圈示意图.jpeg](https://upload-images.jianshu.io/upload_images/1641428-075183640c4d3c1f.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理分析

后续补

#### 完整代码

- 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleAnnulusProgressBar">
        <!--当前进度值-->
        <attr name="progress" format="integer" />
        <!--进度的最大值-->
        <attr name="max" format="integer" />
        <!--外圆的颜色-->
        <attr name="outer_circle_color" format="reference|color" />
        <!--内饼图的颜色-->
        <attr name="pie_color" format="reference|color" />
        <!--外圆轮廓的宽度-->
        <attr name="outer_circle_border_width" format="dimension" />
    </declare-styleable>
</resources>
```

- Java代码

```
public class CircleAnnulusProgressBar extends View {
    //-------------------- 默认值 -------------------
    /**
     * 默认的当前进度，默认为0
     */
    private static final int DEFAULT_PROGRESS = 0;
    /**
     * 默认的最大值，默认为100
     */
    private static final int DEFAULT_MAX = 100;
    /**
     * 默认的外圆轮廓宽度，默认是1dp
     */
    private static int DEFAULT_OUTER_CIRCLE_BORDER_WIDTH = 3;
    /**
     * 默认的外圆的颜色
     */
    private final int DEFAULT_OUTER_CIRCLE_COLOR = Color.parseColor("#FFFFFF");
    /**
     * 默认的内饼形的颜色
     */
    private final int DEFAULT_PIE_CIRCLE = Color.parseColor("#FFFFFF");

    //-------------------- 自定义属性 -------------------
    /**
     * 默认的外圆轮廓宽度，默认是1dp
     */
    private float mOuterCircleBorderWidth;
    /**
     * 外圆的颜色，默认白色
     */
    private int mOuterCircleColor;
    /**
     * 内饼图填充的颜色，默认是白色
     */
    private int mPieColor;

    //-------------------- 绘制相关对象 -------------------
    /**
     * 外圆的画笔
     */
    private Paint mOuterCirclePaint;
    /**
     * 内部的饼形的画笔
     */
    private Paint mPiePaint;
    /**
     * View绘制区域，去除了padding
     */
    private RectF mRect;

    //-------------------- View宽高等参数 -------------------
    /**
     * View的宽，包括padding
     */
    private int mWidth;
    /**
     * View的高，包括padding
     */
    private int mHeight;
    /**
     * 设置的上下左右padding值
     */
    private int mPaddingTop;
    private int mPaddingBottom;
    private int mPaddingLeft;
    private int mPaddingRight;
    /**
     * 外圆的半径，已经处理了padding
     */
    private float mRadius;
    /**
     * 当前进度，默认为最大值
     */
    private int mProgress;
    /**
     * 设置的最大进度，默认为100
     */
    private int mMax;

    //-------------------- 对外使用的对象 -------------------
    /**
     * 监听器集合
     */
    private ArrayList<OnProgressUpdateListener> mListeners;

    public CircleAnnulusProgressBar(Context context) {
        super(context);
        init(null);
    }

    public CircleAnnulusProgressBar(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init(attrs);
    }

    public CircleAnnulusProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(attrs);
    }

    private void init(AttributeSet attrs) {
        //由于dp转换操作必须在初始化后context才不为空，所以在这里初始化默认的外圆轮廓宽度
        DEFAULT_OUTER_CIRCLE_BORDER_WIDTH = dip2px(getContext(), 1f);

        //取出Xml设置的自定义属性，当前进度，最大进度
        if (attrs != null) {
            TypedArray array = getContext().obtainStyledAttributes(attrs, R.styleable.CircleAnnulusProgressBar);
            //Xml设置的进度
            mProgress = array.getInt(R.styleable.CircleAnnulusProgressBar_progress, DEFAULT_PROGRESS);
            //Xml设置的最大值
            mMax = array.getInt(R.styleable.CircleAnnulusProgressBar_max, DEFAULT_MAX);
            //Xml设置的外圆颜色，先读取直接写#FFFFFF等样式的，如果没有，则读取使用引用方式的，就是@color/white这样的
            int resultOuterCircleColor = array.getColor(R.styleable.CircleAnnulusProgressBar_outer_circle_color, DEFAULT_OUTER_CIRCLE_COLOR);
            if (resultOuterCircleColor != DEFAULT_OUTER_CIRCLE_COLOR) {
                mOuterCircleColor = resultOuterCircleColor;
            } else {
                int outerCircleResId = array.getResourceId(R.styleable.CircleAnnulusProgressBar_outer_circle_color, android.R.color.white);
                mOuterCircleColor = getContext().getResources().getColor(outerCircleResId);
            }
            //Xml设置的内饼图颜色，同上，先读取直接写颜色值的，没有再读取使用引用方式的
            int resultPieCircleColor = array.getColor(R.styleable.CircleAnnulusProgressBar_pie_color, DEFAULT_PIE_CIRCLE);
            if (resultPieCircleColor != DEFAULT_PIE_CIRCLE) {
                mPieColor = resultPieCircleColor;
            } else {
                int pieColorResId = array.getResourceId(R.styleable.CircleAnnulusProgressBar_pie_color, android.R.color.white);
                mPieColor = getContext().getResources().getColor(pieColorResId);
            }
            //读取设置的外圆轮廓宽度，读取dimension
            mOuterCircleBorderWidth = array.getDimensionPixelSize(R.styleable.CircleAnnulusProgressBar_outer_circle_border_width, DEFAULT_OUTER_CIRCLE_BORDER_WIDTH);
            //记得回收资源
            array.recycle();
        } else {
            //没有在Xml中设置属性，使用默认属性
            //当前进度
            mProgress = DEFAULT_PROGRESS;
            //最大值
            mMax = DEFAULT_MAX;
            //外圆的颜色
            mOuterCircleColor = DEFAULT_OUTER_CIRCLE_COLOR;
            //内饼形的颜色
            mPieColor = DEFAULT_PIE_CIRCLE;
            //外圆的宽度
            mOuterCircleBorderWidth = DEFAULT_OUTER_CIRCLE_BORDER_WIDTH;
        }
        //外层圆的画笔
        mOuterCirclePaint = new Paint();
        mOuterCirclePaint.setColor(mOuterCircleColor);
        mOuterCirclePaint.setStyle(Paint.Style.STROKE);
        mOuterCirclePaint.setStrokeWidth(mOuterCircleBorderWidth);
        mOuterCirclePaint.setAntiAlias(true);
        //中间进度饼形画笔
        mPiePaint = new Paint();
        mPiePaint.setColor(mPieColor);
        mPiePaint.setStyle(Paint.Style.FILL);
        mPiePaint.setAntiAlias(true);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        //取出总宽高
        mWidth = w;
        mHeight = h;
        //取出设置的padding值
        mPaddingTop = getPaddingTop();
        mPaddingBottom = getPaddingBottom();
        mPaddingLeft = getPaddingLeft();
        mPaddingRight = getPaddingRight();
        //计算外圆直径，取宽高中最小的为圆的直径，这里要处理添加padding的情况。
        float diameter = (Math.min(mWidth, mHeight)) - mPaddingLeft - mPaddingRight;
        //直径除以2算出半径
        mRadius = (float) ((diameter / 2) * 0.98);

        //建立一个Rect保存View的范围，后面画饼形也需要用到
        mRect = new RectF(mPaddingLeft, mPaddingTop, mWidth - mPaddingRight, mHeight - mPaddingBottom);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        setBackgroundColor(getContext().getResources().getColor(android.R.color.transparent));
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //取出宽的模式和大小
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        //取出高的模式和大小
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        //设置的宽高不相等时，将宽高都进行校正，取最小的为标准
        if (widthSize != heightSize) {
            int finalSize = Math.min(widthSize, heightSize);
            widthSize = finalSize;
            heightSize = finalSize;
        }

        //默认宽高值
        int defaultWidth = dip2px(getContext(), 55);
        int defaultHeight = dip2px(getContext(), 55);

        if (widthMode != MeasureSpec.EXACTLY && heightMode != MeasureSpec.EXACTLY) {
            //当宽高都设置wrapContent时设置我们的默认值
            if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
                setMeasuredDimension(defaultWidth, defaultHeight);
            } else if (widthMode == MeasureSpec.AT_MOST) {
                //宽、高任意一个为wrapContent都设置我们默认值
                setMeasuredDimension(defaultWidth, heightSize);
            } else if (heightMode == MeasureSpec.AT_MOST) {
                setMeasuredDimension(widthSize, defaultHeight);
            }
        } else {
            setMeasuredDimension(widthSize, heightSize);
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //获取当前的进度
        int curProgress = getProgress();
        //越界处理
        if (curProgress < 0) {
            curProgress = 0;
        }
        if (curProgress > mMax) {
            curProgress = mMax;
        }
        //画外圆，这里使用宽和高都可以，因为我们限定宽和高都是相等的
        //这里圆心坐标一直都在View的宽高的中间，就算有padding都是不会变的，变的只是半径，半径初始化前已经去处理了padding，这里要注意
        canvas.drawCircle(mWidth / 2, mWidth / 2, mRadius, mOuterCirclePaint);
        //要进行画布缩放操作，先保存图层，因为缩放、平移等操作是叠加的，所以使用完必须恢复，否则下次的onDraw就会累加缩放
        canvas.save();
        //用缩放画布，进行缩放中心的饼图，设置缩放中心是控件的中心
        canvas.scale(0.90f, 0.90f, mWidth / 2, mHeight / 2);
        //计算当前进度对应的角度
        float angle = 360 * (curProgress * 1.0f / getMax());
        //画饼图，-90度就是12点方向开始
        canvas.drawArc(mRect, -90, angle, true, mPiePaint);
        //还原画布图层
        canvas.restore();
        //回调进度给外面的监听器
        for (OnProgressUpdateListener listener : mListeners) {
            listener.onProgressUpdate(curProgress);
        }
    }

    /**
     * 设置进度
     *
     * @param progress 要设置的进度
     */
    public synchronized void setProgress(int progress) {
        if (progress < 0) {
            progress = 0;
        }
        this.mProgress = progress;
        //设置进度可能是子线程，所以将重绘调用交给主线程
        postInvalidate();
    }

    /**
     * 获取当前进度
     *
     * @return 当前的进度
     */
    public int getProgress() {
        return mProgress;
    }

    /**
     * 设置最大值
     *
     * @param max 要设置的最大值
     */
    public synchronized void setMax(int max) {
        if (max < 0) {
            max = 0;
        }
        this.mMax = max;
        //设置进度可能是子线程，所以将重绘调用交给主线程
        postInvalidate();
    }

    /**
     * 进度更新的回调监听
     */
    public interface OnProgressUpdateListener {
        //当进度更新时回调
        void onProgressUpdate(int progress);
    }

    /**
     * 设置更新回调
     *
     * @param listener 监听器实例
     */
    public void addOnProgressUpdateListener(OnProgressUpdateListener listener) {
        if (mListeners == null) {
            mListeners = new ArrayList<OnProgressUpdateListener>();
        }
        this.mListeners.add(listener);
    }

    /**
     * 获取设置的最大值
     *
     * @return 设置的最大值
     */
    public int getMax() {
        return mMax;
    }

    //------------------ 一些尺寸转换方法 ------------------
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
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#66000000">

    <TextView
        android:id="@+id/tipText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="15dp"
        android:text="当前进度:"
        android:textColor="#FFFFFF"
        android:textSize="14sp" />

    <com.hzh.circle.annulus.progressbar.widget.CircleAnnulusProgressBar
        android:id="@+id/progressBar"
        android:layout_width="55dp"
        android:layout_height="55dp"
        android:layout_centerInParent="true"
        app:max="100"
        app:outer_circle_border_width="1dp"
        app:outer_circle_color="#FFFFFF"
        app:pie_color="@android:color/white"
        app:progress="30" />
</RelativeLayout>
```

- Java代码

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //饼图控件
        final CircleAnnulusProgressBar progressBar = findViewById(R.id.progressBar);
        //提示问题
        final TextView tipText = findViewById(R.id.tipText);
        //设置进度最大值
        progressBar.setMax(100);
        //设置进度更新监听，每次更新时重新设置提示文字
        progressBar.addOnProgressUpdateListener(new CircleAnnulusProgressBar.OnProgressUpdateListener() {
            @Override
            public void onProgressUpdate(int progress) {
                tipText.setText("当前进度: ".concat(String.valueOf(progress)));
            }
        });
        //用值动画不断改变进度，测试进度
        ValueAnimator animator = ValueAnimator.ofInt(0, 100);
        animator.setRepeatMode(ValueAnimator.RESTART);
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.setInterpolator(new LinearInterpolator());
        animator.setDuration(3000);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                Integer cValue = (Integer) animation.getAnimatedValue();
                progressBar.setProgress(cValue);
            }
        });
        animator.start();
    }
}
```