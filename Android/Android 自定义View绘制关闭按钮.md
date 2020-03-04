#### Android 自定义View绘制关闭按钮

一个❌关闭按钮，一般用于弹窗、搜索框等执行关闭，本篇来自定义View绘制它。

![Android 自定义View绘制关闭按钮.png](https://upload-images.jianshu.io/upload_images/1641428-5a2b18563f5596a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理

按传统的坐标计算方式，需要画2条线，则需要计算2条线的端点坐标，由于我们可以获取到控件的宽、高，所以就能计算出坐标，就可以画线了，不过这次我们不使用这种方式，而是投机取巧一下。

思路是这样的，我们从控件中心画一条横线，然后旋转45度再画一条，旋转4次，就画出了一个十字，最后再旋转45度，十字就变成❌了，相对上面传统的坐标计算，会更加直观，又不需要计算太多的坐标。

并且我们还加一个小样式，外围有一个圈，一般这种UI风格也常见。

#### 完整代码

- 自定义属性

```
<declare-styleable name="CloseView">
    <!-- 线的颜色 -->
    <attr name="cv_color" format="color" />
    <!-- 线高 -->
    <attr name="cv_line_width" format="integer|float|dimension" />
    <!-- 模式，普通模式、圆边模式 -->
    <attr name="cv_mode" format="enum">
        <enum name="normal" value="1" />
        <enum name="circle" value="2" />
    </attr>
    <!-- 圆的边颜色 -->
    <attr name="cv_circle_color" format="color" />
    <!-- 圆的边线宽 -->
    <attr name="cv_circle_line_width" format="integer|float|dimension" />
</declare-styleable>
```

- Java代码

```
public class CloseView extends View {
    /**
     * 普通模式
     */
    private static final int MODE_NORMAL = 1;
    /**
     * 有圆模式
     */
    private static final int MODE_CIRCLE = 2;

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
     * 线的长度
     */
    private float mLineLength;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 线颜色
     */
    private int mColor;
    /**
     * 线宽
     */
    private float mLineWidth;
    /**
     * 模式
     */
    private int mMode;
    /**
     * 圆的颜色
     */
    private int mCircleColor;
    /**
     * 圆的边线宽
     */
    private float mCircleLineWidth;

    public CloseView(Context context) {
        this(context, null);
    }

    public CloseView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CloseView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setAntiAlias(true);
        mPaint.setStrokeWidth(mLineWidth);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.CloseView, defStyleAttr, 0);
        mColor = array.getColor(R.styleable.CloseView_cv_color, Color.argb(255, 0, 0, 0));
        mLineWidth = array.getDimension(R.styleable.CloseView_cv_line_width, dip2px(context, 1.5f));
        mMode = array.getInt(R.styleable.CloseView_cv_mode, MODE_NORMAL);
        //如果不指定圆的颜色，颜色和线的颜色一致
        mCircleColor = array.getColor(R.styleable.CloseView_cv_circle_color, mColor);
        //如果不指定圆的线宽，则和线的线宽的一致
        mCircleLineWidth = array.getDimension(R.styleable.CloseView_cv_circle_line_width, mLineWidth);
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        mLineLength = (Math.min(mViewWidth, mViewHeight) * 0.65f) / 2f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //线旋转45，将十字旋转
        canvas.rotate(45);
        //画交叉线
        drawCrossLine(canvas);
        //画圆模式
        if (mMode == MODE_CIRCLE) {
            drawCircle(canvas);
        }
    }

    /**
     * 画交叉线
     */
    private void drawCrossLine(Canvas canvas) {
        int count = 4;
        int angle = (int) (360f / count);
        //画十字
        canvas.save();
        mPaint.setColor(mColor);
        mPaint.setStrokeWidth(mLineWidth);
        for (int i = 0; i < count; i++) {
            //旋转4次，每次画一条线，每次45度，合起来就是一个十字了
            canvas.rotate(angle * i);
            canvas.drawLine(0, 0, mLineLength, 0, mPaint);
        }
        canvas.restore();
    }

    /**
     * 画圆
     */
    private void drawCircle(Canvas canvas) {
        mPaint.setColor(mCircleColor);
        mPaint.setStrokeWidth(mCircleLineWidth);
        float radius = (Math.min(mViewWidth, mViewHeight) * 0.9f) / 2f;
        canvas.drawCircle(0,0, radius, mPaint);
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
<com.zh.cavas.sample.widget.CloseView
    android:id="@+id/close_view"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:cv_circle_color="@android:color/darker_gray"
    app:cv_circle_line_width="1dp"
    app:cv_color="@android:color/black"
    app:cv_line_width="1.5dp"
    app:cv_mode="normal" />

<com.zh.cavas.sample.widget.CloseView
    android:id="@+id/close_view2"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:cv_circle_color="@android:color/darker_gray"
    app:cv_circle_line_width="1dp"
    app:cv_color="@android:color/darker_gray"
    app:cv_line_width="1.5dp"
    app:cv_mode="circle" />
```