#### Android 自定义View 绘制六边形设置按钮

今天逛酷安的时候，发现酷安的设置按钮（截图的右上角），是一个六边形 + 中心圆的图标，所以又是一个自定义View练习对象了。画圆很简单，知道半径即可，而重点就在画出六边形。

![酷安截图.png](https://upload-images.jianshu.io/upload_images/1641428-ffa5d7eb220342df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 最终效果

![Android 自定义View 绘制六边形设置按钮.png](https://upload-images.jianshu.io/upload_images/1641428-7cae916a1e155797.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 原理

1. 中心画一个小圆
2. 再画一个六边形（难点）

怎样才能画六边形，？只要我们计算出6个角的坐标点，用Path连接即可。至于怎么计算，就要使用三角函数。我们这样分析：

在六边形中心画一个内切圆，6个角都在圆上。360被六边形的6个角平分，每个角就为60度，而每个角的2条腰的长都是圆的半径。如下图所示：

![中心角.png](https://upload-images.jianshu.io/upload_images/1641428-a5f4dc6dc574eadd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至于三角函数，可能我们会忘记，我们对着这张图解图来回忆一下。

![三角函数图解.jpg](https://upload-images.jianshu.io/upload_images/1641428-054bdd2841072248.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只要我们计算出中心角的角度，用三角函数中的cos()就可以计算出点的x坐标，sin()就可以计算出y坐标（注意这个2个方法传入的角度需要转为弧度）。重点计算代码在drawPolygon()方法。

#### 完整代码

- 自定义属性

```
<declare-styleable name="PolygonSettingView">
    <!-- 开始颜色 -->
    <attr name="psv_color" format="color|reference" />
    <!-- 多边形多少条边，至少3条边，如果小于3条，会强制3条 -->
    <attr name="psv_num" format="integer" />
    <!-- 边的线宽 -->
    <attr name="psv_line_width" format="float|dimension|reference" />
</declare-styleable>
```

- Java代码

```
public class PolygonSettingView extends View {
    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 100;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;

    /**
     * 多边形的边数
     */
    private int mNum;
    /**
     * 最小的多边形的半径
     */
    private float mRadius;
    /**
     * 360度对应的弧度（为什么2π就是360度？弧度的定义：弧长 / 半径，一个圆的周长是2πr，如果是一个360度的圆，它的弧长就是2πr，如果这个圆的半径r长度为1，那么它的弧度就是，2πr / r = 2π）
     */
    private final double mPiDouble = 2 * Math.PI;
    /**
     * 多边形中心角的角度（每个多边形的内角和为360度，一个多边形2个相邻角顶点和中心的连线所组成的角为中心角
     * 中心角的角度都是一样的，所以360度除以多边形的边数，就是一个中心角的角度），这里注意，因为后续要用到Math类的三角函数
     * Math类的sin和cos需要传入的角度值是弧度制，所以这里的中心角的角度，也是弧度制的弧度
     */
    private float mCenterAngle;
    /**
     * 颜色
     */
    private int mColor;
    /**
     * 中心小圆的半径
     */
    private float mSmallCircleRadius;
    /**
     * 线宽
     */
    private float mLineWidth;

    public PolygonSettingView(Context context) {
        this(context, null);
    }

    public PolygonSettingView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public PolygonSettingView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        //取消硬件加速
        setLayerType(LAYER_TYPE_SOFTWARE, null);
        //画笔
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(mColor);
        mPaint.setStrokeWidth(mLineWidth);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        //默认边数和最小边数
        int defaultNum = 6;
        int minNum = 3;
        int defaultColor = Color.argb(255, 0, 0, 0);
        int defaultLineWidth = dip2px(context, 1.5f);
        if (attrs != null) {
            TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.PolygonSettingView, defStyleAttr, 0);
            mColor = array.getColor(R.styleable.PolygonSettingView_psv_color, defaultColor);
            int num = array.getInt(R.styleable.PolygonSettingView_psv_num, defaultNum);
            mNum = num <= minNum ? minNum : num;
            mLineWidth = array.getDimension(R.styleable.PolygonSettingView_psv_line_width, defaultLineWidth);
            array.recycle();
        } else {
            mColor = defaultColor;
            mNum = defaultNum;
        }
        //计算中心角弧度
        mCenterAngle = (float) (mPiDouble / mNum);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算最小的多边形的半径
        mRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.95f;
        //计算中心小圆的半径
        mSmallCircleRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.3f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //画小圆
        drawSmallCircle(canvas);
        //画多边形
        drawPolygon(canvas);
    }

    /**
     * 画小圆
     */
    private void drawSmallCircle(Canvas canvas) {
        canvas.drawCircle(0, 0, mSmallCircleRadius, mPaint);
    }

    /**
     * 画多边形
     */
    private void drawPolygon(Canvas canvas) {
        //多边形边角顶点的x坐标
        float pointX;
        //多边形边角顶点的y坐标
        float pointY;
        //总的圆的半径，就是全部多边形的半径之和
        Path path = new Path();
        //画前先重置路径
        path.reset();
        for (int i = 1; i <= mNum; i++) {
            //cos三角函数，中心角的邻边 / 斜边，斜边的值刚好就是半径，cos值乘以斜边，就能求出邻边，而这个邻边的长度，就是点的x坐标
            pointX = (float) (Math.cos(i * mCenterAngle) * mRadius);
            //sin三角函数，中心角的对边 / 斜边，斜边的值刚好就是半径，sin值乘以斜边，就能求出对边，而这个对边的长度，就是点的y坐标
            pointY = (float) (Math.sin(i * mCenterAngle) * mRadius);
            //如果是一个点，则移动到这个点，作为起点
            if (i == 1) {
                path.moveTo(pointX, pointY);
            } else {
                //其他的点，就可以连线了
                path.lineTo(pointX, pointY);
            }
        }
        path.close();
        canvas.drawPath(path, mPaint);
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

- 简单使用

```
<com.zh.cavas.sample.widget.PolygonSettingView
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_margin="10dp"
            app:psv_color="@android:color/black"
            app:psv_line_width="1.5dp"
            app:psv_num="6" />
```