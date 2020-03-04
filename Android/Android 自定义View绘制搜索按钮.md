#### Android 自定义View绘制搜索按钮

搜索View，样式类似微信顶部栏上的搜索图标🔍，下面一起来自定义View绘制它吧。

![Android 自定义View绘制搜索按钮.png](https://upload-images.jianshu.io/upload_images/1641428-95713646e5416a64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理

原理很简单，搜索图标由2个部分组成，一个圆和一条45度的线，画圆很简单，画有角度的线稍微有点麻烦，但这里我们投机取巧一下，依靠Canvas的rotate()方法，我们可以减少很多计算，我们只需要在圆的右边画一条水平线，再用Canvas.rotate()旋转45度，即可！

#### 完整代码

- 自定义属性

```
<declare-styleable name="AndroidSearchView">
    <!-- 线的颜色 -->
    <attr name="asv_color" format="color" />
    <!-- 线宽 -->
    <attr name="asv_line_width" format="integer|float|dimension" />
</declare-styleable>
```

- Java代码

```
public class AndroidSearchView extends View {
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
     * 画笔
     */
    private Paint mPaint;
    /**
     * 圆的半径
     */
    private float mCircleRadius;
    /**
     * 线的长度
     */
    private float mLineLength;
    /**
     * 颜色
     */
    private int mColor;
    /**
     * 线宽
     */
    private float mLineWidth;

    public AndroidSearchView(Context context) {
        this(context, null);
    }

    public AndroidSearchView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public AndroidSearchView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(mLineWidth);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.AndroidSearchView, defStyleAttr, 0);
        mColor = array.getColor(R.styleable.AndroidSearchView_asv_color, Color.argb(255, 0, 0, 0));
        mLineWidth = array.getDimension(R.styleable.AndroidSearchView_asv_line_width, dip2px(context, 1.5f));
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算半径
        mCircleRadius = (Math.min(mViewWidth, mViewHeight) * 0.35f) / 2f;
        //计算把柄的长度
        mLineLength = (Math.min(mViewWidth, mViewHeight) * 0.18f) * 2f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        canvas.rotate(45f);
        canvas.drawCircle(0, 0, mCircleRadius, mPaint);
        canvas.drawLine(mCircleRadius, 0, mLineLength, 0, mPaint);
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
<com.zh.cavas.sample.widget.AndroidSearchView
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:asv_color="@android:color/black"
    app:asv_line_width="1.5dp" />
```