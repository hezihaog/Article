#### Android 仿网易云音乐 音轨跳动效果

网易云音乐的Loading效果，大家应该也比较熟悉了，效果是一个红色音轨不断跳动的效果，一般用于Loading等待时填充使用。本篇来自定义这个效果。

![Android 仿网易云音乐 音轨跳动View.gif](https://upload-images.jianshu.io/upload_images/1641428-9d7df7e54677936f.gif?imageMogr2/auto-orient/strip)

#### 原理

原理就是画4条垂直线，使用随机数不断更新，只要速度够快，就会形成跳动的效果。

画线调用canvas.drawLine()方法就可以画出来，而主要是计算每一条的垂直线的x坐标，y坐标使用随机比值（0 ~ 1）乘以总音轨高度即可算出。

其实x坐标可以不计算，我们可以使用canvas.translate()方法，每次画1条垂直线的时候，平移画布一个固定的间距距离，x坐标还是第一条先的垂直线的x坐标，再进行画线，疑似画4次后，就出现了4条垂直线了。

#### 遇到的问题

按照上面的平移的方式，就可以画出4条垂直线，但是效果是反的，为什么呢？如下图所示：

![反方向.png](https://upload-images.jianshu.io/upload_images/1641428-927b5190291db0e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为按照Android的坐标系，我们知道，默认坐标系的(0,0)原点在控件的左上角，原点发射出来的2条边都为正轴，如果按照这个坐标系，我们画出来的线是反的。

- 解决这个问题也很简单，我们只要将画布旋转180度，而且旋转中心指定在控件的中心点，就可以摆正它。效果如下：

![正方向.png](https://upload-images.jianshu.io/upload_images/1641428-7343f37b2fcd9d7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//旋转画布，按控件中心旋转180度，即可让音轨反转
canvas.rotate(180, mViewWidth / 2f, mViewHeight / 2f);
```

#### 完整代码

- 自定义属性

```
<declare-styleable name="CloudMusicLoadingView">
    <!-- 音轨数量 -->
    <attr name="cmlv_rail_count" format="integer" />
    <!-- 音轨的颜色 -->
    <attr name="cmlv_rail_color" format="color" />
    <!-- 音轨线宽粗细 -->
    <attr name="cmlv_line_width" format="integer|float|dimension" />
</declare-styleable>
```

- Java代码

```
public class CloudMusicLoadingView extends View implements Runnable {
    /**
     * 随机数
     */
    private static Random mRandom = new Random();

    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 65;

    /**
     * 默认4条音轨
     */
    private static final int DEFAULT_RAIL_COUNT = 4;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 音轨数量
     */
    private int mRailCount;
    /**
     * 音轨颜色
     */
    private int mRailColor;
    /**
     * 每条音轨的线宽
     */
    private float mRailLineWidth;
    /**
     * Float类型估值器，用于在指定数值区域内进行估值
     */
    private FloatEvaluator mFloatEvaluator;

    public CloudMusicLoadingView(Context context) {
        this(context, null);
    }

    public CloudMusicLoadingView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CloudMusicLoadingView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mRailColor);
        mPaint.setStrokeWidth(mRailLineWidth);
        mPaint.setStyle(Paint.Style.FILL);
        //设置笔触为方形
        mPaint.setStrokeCap(Paint.Cap.SQUARE);
        mPaint.setAntiAlias(true);
        mFloatEvaluator = new FloatEvaluator();
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.CloudMusicLoadingView, defStyleAttr, 0);
        mRailCount = array.getInt(R.styleable.CloudMusicLoadingView_cmlv_rail_count, DEFAULT_RAIL_COUNT);
        mRailColor = array.getColor(R.styleable.CloudMusicLoadingView_cmlv_rail_color, Color.argb(255, 255, 255, 255));
        mRailLineWidth = array.getDimension(R.styleable.CloudMusicLoadingView_cmlv_line_width, dip2px(context, 1f));
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //计算可用高度
        float totalAvailableHeight = mViewHeight - getPaddingBottom() - getPaddingTop();
        //计算每条音轨平分宽度后的位置
        float averageBound = (mViewWidth * 1.0f) / mRailCount;
        //计算每条音轨的x坐标位置
        float x = averageBound - mRailLineWidth;
        float y = getPaddingBottom();
        //旋转画布，按控件中心旋转180度，即可让音轨反转
        canvas.rotate(180, mViewWidth / 2f, mViewHeight / 2f);
        //保存画布
        canvas.save();
        for (int i = 1; i <= mRailCount; i++) {
            //估值x坐标
            float fraction = nextRandomFloat(1.0f);
            float evaluateY = (mFloatEvaluator.evaluate(fraction, 0.3f, 0.9f)) * totalAvailableHeight;
            //第一个不需要偏移
            if (i == 1) {
                canvas.drawLine(x, y, x, evaluateY, mPaint);
            } else {
                //后续，每个音轨都固定偏移间距后，再画
                canvas.translate(x, 0);
                canvas.drawLine(x, y, x, evaluateY, mPaint);
            }
        }
        //恢复画布
        canvas.restore();
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
        start();
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        stop();
    }

    @Override
    public void run() {
        invalidate();
        postDelayed(this, 100);
    }

    public void start() {
        postDelayed(this, 700);
    }

    private void stop() {
        removeCallbacks(this);
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }

    /**
     * 产生一个随机float
     *
     * @param sl 随机数范围[0,sl)
     */
    public static float nextRandomFloat(float sl) {
        return mRandom.nextFloat() * sl;
    }
}
```

- 简单使用

```
<com.zh.cavas.sample.widget.CloudMusicLoadingView
    android:layout_width="25dp"
    android:layout_height="25dp"
    android:layout_marginTop="20dp"
    android:paddingBottom="2dp"
    app:cmlv_rail_color="#FB0006" />
```