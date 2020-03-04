#### Android 自定义View绘制汉堡菜单

汉堡菜单，其实就是三条横杠，Google Material Design中一般用来做侧边栏开关，微软的UWP也是同样的做法。

![Android 自定义View绘制汉堡菜单.png](https://upload-images.jianshu.io/upload_images/1641428-4310801c6927b363.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理

将画布中心移动到控件中心，先绘制中间那一条横线，再计算上面和下面的横线的位置，如果不计算，也可以使用Canvas.translate()，移动2次画布中心去画。

#### 完整代码

- 自定义属性

```
<declare-styleable name="AndroidMenuView">
    <!-- 线的颜色 -->
    <attr name="amv_color" format="color" />
    <!-- 线高 -->
    <attr name="amv_line_height" format="integer|float|dimension" />
</declare-styleable>
```

- Java代码

```
public class AndroidMenuView extends View {
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
     * 每条线之间的距离
     */
    private float mLineDistance;
    /**
     * 每条线的长度
     */
    private float mLineLength;
    /**
     * 一半的线长度
     */
    private float mHalfLineLength;
    /**
     * 线的颜色
     */
    private int mColor;
    /**
     * 线高
     */
    private float mLineHeight;

    public AndroidMenuView(Context context) {
        this(context, null);
    }

    public AndroidMenuView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public AndroidMenuView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setAntiAlias(true);
        mPaint.setStrokeWidth(mLineHeight);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.AndroidMenuView, defStyleAttr, 0);
        mColor = array.getColor(R.styleable.AndroidMenuView_amv_color, Color.argb(255, 0, 0, 0));
        mLineHeight = array.getDimension(R.styleable.AndroidMenuView_amv_line_height, dip2px(context, 2f));
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算半径
        float radius = Math.min(mViewWidth, mViewHeight) / 2f;
        //计算每条线之间的距离
        mLineDistance = (radius / 3f);
        //每条线的长度
        mLineLength = radius;
        //计算一半的线的长度
        mHalfLineLength = mLineLength / 2f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int centerX = mViewWidth / 2;
        int centerY = mViewHeight / 2;
        canvas.translate(centerX, centerY);
        //画中间的线
        canvas.drawLine(-mHalfLineLength, 0, mHalfLineLength, 0, mPaint);
        //画顶部的线
        canvas.drawLine(-mHalfLineLength, -mLineDistance, mHalfLineLength, -mLineDistance, mPaint);
        //画底部的线
        canvas.drawLine(-mHalfLineLength, mLineDistance, mHalfLineLength, mLineDistance, mPaint);
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
<com.zh.cavas.sample.widget.AndroidMenuView
    android:id="@+id/menu_view"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:amv_color="@android:color/black"
    app:amv_line_height="1dp" />
```