#### Android 自定义View 绘制五角星

之前写过的App里有评分的功能，而显示评分一般使用系统的RatingBar再加自定义，一切都很完美，但是产品提了一个需求，例如4.6、4.7、5.8分，不要显示为4个星星加一个半星（4.5分），而是能显示出区别。（系统的RatingBar必须满正整数才可以满星，如果没满，还是显示一半的效果）

这时系统的RatingBar就不满足需求了，需要我们自定义控件，当时需求太赶了，需求先放下来了，既然要绘制星星，我们就从绘制单个星星开始吧，后续只要对星星做一层PorterDuffXfermode处理即可。

![Android 自定义View 绘制五角星.png](https://upload-images.jianshu.io/upload_images/1641428-ecbb9970e580ca16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 原理

我们以星星的中心为圆心，对五角星的5个端点画一个外接圆。以五角星的内部5个小角画一个内接圆，所以有2个圆。

![原理图解.png](https://upload-images.jianshu.io/upload_images/1641428-50db6bff25924e23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 五角星的5个顶点，将360的圆平分5份，平均角度为72度。
2. 取一个90度直角为参考，90度直角将右上角的部分分为2个角，分别是大的角和小的角，大的角为72度，所以小角的角度为90度减去72度，为18度。
3.  我们再计算出一半的平均角度，72除以2，为36度。
4. 而内角，就是凹进去的那个小角的角度就可以计算出来，36度加18度，为54度。
5. 知道2个角的角度，以及外接圆和内接圆的半径，就可以用三角函数计算出坐标点。

文字描述有点不清楚，具体原理可以观看视频教程，[慕课网Web前端 Canvas画星星教程](https://www.imooc.com/video/3488?t=1474563809992)。

我也是听了一遍讲解后，用Android的Canvas画一次，Web端的Canvas虽然API有点不一样，但也是类似的，有些地方要稍微处理一下，例如Web端的Canvas的beginPath(x,y)是直接在点坐标x,y开始，而不经过中心，Android端的Canvas会经过0,0点，所以第一个点我们要先将Path调用moveTo(x,y)，移动到第一个点，再继续lineTo(x,y)下一个点。最后调用close()闭合Path。

#### 完整代码

- 自定义属性

```
<declare-styleable name="StarsView">
    <!-- 星星的颜色 -->
    <attr name="stv_color" format="color" />
    <!-- 星星的边数 -->
    <attr name="stv_num" format="integer|dimension|reference" />
    <!-- 边的线宽 -->
    <attr name="stv_edge_line_width" format="float|dimension|reference" />
    <!-- 填充风格 -->
    <attr name="stv_style" format="enum">
        <!-- 填满 -->
        <enum name="fill" value="1" />
        <!-- 描边 -->
        <enum name="stroke" value="2" />
    </attr>
</declare-styleable>
```

- Java代码

```
public class StarsView extends View {
    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 100;
    /**
     * 风格，填满
     */
    private static final int STYLE_FILL = 1;
    /**
     * 风格，描边
     */
    private static int STYLE_STROKE = 2;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;
    /**
     * 外边大圆的半径
     */
    private float mOutCircleRadius;
    /**
     * 里面小圆的的半径
     */
    private float mInnerCircleRadius;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 多少个角的五角星
     */
    private int mAngleNum;
    /**
     * 星星的路径
     */
    private Path mPath;
    /**
     * 星星的颜色
     */
    private int mColor;
    /**
     * 边的线宽
     */
    private float mEdgeLineWidth;
    /**
     * 填充风格
     */
    private int mStyle;

    public StarsView(Context context) {
        this(context, null);
    }

    public StarsView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public StarsView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
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
        if (mStyle == STYLE_FILL) {
            mPaint.setStyle(Paint.Style.FILL);
        } else if (mStyle == STYLE_STROKE) {
            mPaint.setStyle(Paint.Style.STROKE);
        }
        mPaint.setColor(mColor);
        mPaint.setStrokeWidth(mEdgeLineWidth);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        int defaultColor = Color.argb(255, 0, 0, 0);
        int defaultNum = 5;
        int mineNum = 2;
        float defaultEdgeLineWidth = dip2px(context, 1f);
        int defaultStyle = STYLE_STROKE;
        if (attrs != null) {
            TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.StarsView, defStyleAttr, 0);
            mColor = array.getColor(R.styleable.StarsView_stv_color, defaultColor);
            int num = array.getInt(R.styleable.StarsView_stv_num, defaultNum);
            mAngleNum = num <= mineNum ? mineNum : num;
            mEdgeLineWidth = array.getDimension(R.styleable.StarsView_stv_edge_line_width, defaultEdgeLineWidth);
            mStyle = array.getInt(R.styleable.StarsView_stv_style, defaultStyle);
            array.recycle();
        } else {
            mColor = defaultColor;
            mAngleNum = defaultNum;
            mEdgeLineWidth = defaultEdgeLineWidth;
            mStyle = defaultStyle;
        }
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算外边大圆的半径
        mOutCircleRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.95f;
        //计算里面小圆的的半径
        mInnerCircleRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.5f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //画星星
        drawStars(canvas);
    }

    /**
     * 画星星
     */
    private void drawStars(Canvas canvas) {
        //计算平均角度，例如360度分5份，每一份角都为72度
        float averageAngle = 360f / mAngleNum;
        //计算大圆的外角的角度，从右上角为例计算，90度的角减去一份角，得出剩余的小角的角度，例如90 - 72 = 18 度
        float outCircleAngle = 90 - averageAngle;
        //一份平均角度的一半，例如72 / 2 = 36度
        float halfAverageAngle = averageAngle / 2f;
        //计算出小圆内角的角度，36 + 18 = 54 度
        float internalAngle = halfAverageAngle + outCircleAngle;
        //创建2个点
        Point outCirclePoint = new Point();
        Point innerCirclePoint = new Point();
        if (mPath == null) {
            mPath = new Path();
        }
        mPath.reset();
        for (int i = 0; i < mAngleNum; i++) {
            //计算大圆上的点坐标
            //x = Math.cos((18 + 72 * i) / 180f * Math.PI) * 大圆半径
            //y = -Math.sin((18 + 72 * i)/ 180f * Math.PI) * 大圆半径
            outCirclePoint.x = (int) (Math.cos(angleToRadian(outCircleAngle + i * averageAngle)) * mOutCircleRadius);
            outCirclePoint.y = (int) -(Math.sin(angleToRadian(outCircleAngle + i * averageAngle)) * mOutCircleRadius);
            //计算小圆上的点坐标
            //x = Math.cos((54 + 72 * i) / 180f * Math.PI ) * 小圆半径
            //y = -Math.sin((54 + 72 * i) / 180 * Math.PI ) * 小圆半径
            innerCirclePoint.x = (int) (Math.cos(angleToRadian(internalAngle + i * averageAngle)) * mInnerCircleRadius);
            innerCirclePoint.y = (int) -(Math.sin(angleToRadian(internalAngle + i * averageAngle)) * mInnerCircleRadius);
            //第一次，先移动到第一个大圆上的点
            if (i == 0) {
                mPath.moveTo(outCirclePoint.x, outCirclePoint.y);
            }
            //坐标连接，先大圆角上的点，再到小圆角上的点
            mPath.lineTo(outCirclePoint.x, outCirclePoint.y);
            mPath.lineTo(innerCirclePoint.x, innerCirclePoint.y);
        }
        mPath.close();
        canvas.drawPath(mPath, mPaint);
    }

    /**
     * 角度转弧度，由于Math的三角函数需要传入弧度制，而不是角度值，所以要角度换算为弧度，角度 / 180 * π
     *
     * @param angle 角度
     * @return 弧度
     */
    private double angleToRadian(float angle) {
        return angle / 180f * Math.PI;
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
<com.zh.cavas.sample.widget.StarsView
    android:id="@+id/stars"
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:stv_color="#0000FF"
    app:stv_edge_line_width="1dp"
    app:stv_num="5"
    app:stv_style="stroke" />

<com.zh.cavas.sample.widget.StarsView
    android:id="@+id/stars2"
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:stv_color="#0000FF"
    app:stv_edge_line_width="1dp"
    app:stv_num="5"
    app:stv_style="fill" />
```