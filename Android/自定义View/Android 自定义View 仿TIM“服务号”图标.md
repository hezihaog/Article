#### Android 自定义View 仿TIM“服务号”图标

]因疫情延长了放假时间，放假时间瞬间多了一个星期时间，那么就继续自定义控件学习吧~本篇来用自定义View来绘制TIM“服务号”的图标。武汉加油，中国加油！

#### 最终效果

![最终效果.png](https://upload-images.jianshu.io/upload_images/1641428-c7bba5e834e6dc21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![TIM效果.png](https://upload-images.jianshu.io/upload_images/1641428-24583c2b4227e153.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

#### 原理

绘制分3步，分别为：

1. 绘制背景的圆形
2. 绘制中间的八角形
3. 绘制中心的勾

- 绘制圆形很简单，知道圆的半径和位置坐标，使用canvas.drawCircle()即可。

- 细看这个八角形的角还是带圆角的，如果使用计算每个角的坐标的话，会很复杂！但还是有投机取巧的办法，我们可以将这个8角形看成2个圆角矩形，他们重叠在一起，只是上层的圆角矩形是经过画布旋转45度再进行绘制的，这样就得出了一个八角形，哈哈，难度瞬间减去了80%。

- 画勾子，可以参考上篇文章[Android 自定义View 对勾CheckBox](https://www.jianshu.com/p/1af1ccd776f5)

)，我们也是采用画布旋转，在中间，画一个直角的方式，来得到一个对勾，这样就大功告成了！

#### 完整代码

- 自定义属性

```
<declare-styleable name="TIMServiceNoView">
    <!-- 背景圆的颜色 -->
    <attr name="tim_snv_bg_circle_color" format="color" />
    <!-- 圆角矩形图标的颜色 -->
    <attr name="tim_snv_round_rect_icon_color" format="color" />
</declare-styleable>
```

- Java代码

```
public class TIMServiceNoView extends View {
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
     * 背景圆的半径
     */
    private float mBgCircleRadius;
    /**
     * 背景圆的画笔
     */
    private Paint mBgCirclePaint;
    /**
     * 背景圆的颜色
     */
    private int mBgCircleColor;
    /**
     * 圆角矩形的图标的画笔
     */
    private Paint mRoundRectIconPaint;
    /**
     * 圆角矩形的图标的颜色
     */
    private int mRoundRectIconColor;
    /**
     * 图标圆角矩形的半径
     */
    private float mRoundRectIconRadius;
    /**
     * 图标的圆角矩形的边圆角半径
     */
    private float mRoundRectIconEdgeRoundRadius;
    /**
     * 钩子的画笔
     */
    private Paint mHookPaint;
    /**
     * 钩子的线长度
     */
    private float mHookLineLength;

    public TIMServiceNoView(Context context) {
        this(context, null);
    }

    public TIMServiceNoView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TIMServiceNoView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        //取消硬件加速
        setLayerType(LAYER_TYPE_SOFTWARE, null);
        //背景圆的画笔
        mBgCirclePaint = new Paint();
        mBgCirclePaint.setAntiAlias(true);
        mBgCirclePaint.setStyle(Paint.Style.FILL);
        mBgCirclePaint.setColor(mBgCircleColor);
        mBgCirclePaint.setStrokeWidth(dip2px(context, 2f));
        //圆角矩形的图标的画笔
        mRoundRectIconPaint = new Paint();
        mRoundRectIconPaint.setAntiAlias(true);
        mRoundRectIconPaint.setStyle(Paint.Style.FILL);
        mRoundRectIconPaint.setColor(mRoundRectIconColor);
        mRoundRectIconPaint.setStrokeWidth(dip2px(context, 2f));
        //钩子的画笔
        mHookPaint = new Paint();
        mHookPaint.setAntiAlias(true);
        mHookPaint.setStyle(Paint.Style.STROKE);
        mHookPaint.setColor(mBgCircleColor);
        //设置连接处为直角
        mHookPaint.setStrokeJoin(Paint.Join.MITER);
        //设置笔触为圆形
        mHookPaint.setStrokeCap(Paint.Cap.ROUND);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        //默认背景圆的颜色
        int defaultBgCircleColor = Color.argb(255, 251, 146, 69);
        int defaultRoundRectIconColor = Color.argb(255, 255, 255, 255);
        if (attrs != null) {
            TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TIMServiceNoView, defStyleAttr, 0);
            mBgCircleColor = array.getColor(R.styleable.TIMServiceNoView_tim_snv_bg_circle_color, defaultBgCircleColor);
            mRoundRectIconColor = array.getColor(R.styleable.TIMServiceNoView_tim_snv_round_rect_icon_color, defaultRoundRectIconColor);
            array.recycle();
        } else {
            mBgCircleColor = defaultBgCircleColor;
            mRoundRectIconColor = defaultRoundRectIconColor;
        }
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算背景圆的半径
        mBgCircleRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.95f;
        //计算图标圆角矩形的半径
        mRoundRectIconRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.45f;
        //计算图标的圆角矩形的边圆角半径
        mRoundRectIconEdgeRoundRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.1f;
        //计算对勾的长度
        mHookLineLength = mRoundRectIconRadius * 0.5f;
        //设置对勾的线宽
        mHookPaint.setStrokeWidth(mRoundRectIconRadius * 0.13f);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //画背景圆
        drawBgCircle(canvas);
        //画圆角矩形的图标
        drawRoundRectIcon(canvas);
        //画钩子
        drawHook(canvas);
    }

    /**
     * 画背景圆
     */
    private void drawBgCircle(Canvas canvas) {
        canvas.drawCircle(0, 0, mBgCircleRadius, mBgCirclePaint);
    }

    /**
     * 画圆角矩形的图标
     */
    private void drawRoundRectIcon(Canvas canvas) {
        RectF rect = new RectF(-mRoundRectIconRadius, -mRoundRectIconRadius, mRoundRectIconRadius, mRoundRectIconRadius);
        canvas.drawRoundRect(rect, mRoundRectIconEdgeRoundRadius, mRoundRectIconEdgeRoundRadius, mRoundRectIconPaint);
        canvas.save();
        //旋转画布45度再画一次
        canvas.rotate(45);
        canvas.drawRoundRect(rect, mRoundRectIconEdgeRoundRadius, mRoundRectIconEdgeRoundRadius, mRoundRectIconPaint);
        canvas.restore();
    }

    /**
     * 画钩子
     */
    private void drawHook(Canvas canvas) {
        //其中一个钩子长度会多一点点
        float edgeLength = mRoundRectIconRadius / 3f;
        canvas.save();
        //画布向下平移一半的半径长度
        canvas.translate(-(mRoundRectIconRadius / 10f), mRoundRectIconRadius / 3.5f);
        //旋转画布45度
        canvas.rotate(-45);
        Path path = new Path();
        path.reset();
        path.moveTo(0, 0);
        //向右画一条线
        path.lineTo(mHookLineLength + edgeLength, 0);
        //回到中心点
        path.moveTo(0, 0);
        //向上画一条线
        path.lineTo(0, -mHookLineLength);
        //画路径
        canvas.drawPath(path, mHookPaint);
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

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }
}
```

#### 简单使用

- 布局

```
<com.zh.cavas.sample.widget.TIMServiceNoView
    android:layout_width="50dp"
    android:layout_height="50dp" />
```