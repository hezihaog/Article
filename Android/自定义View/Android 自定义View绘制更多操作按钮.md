#### Android 自定义View绘制更多操作按钮

一般更多按钮，就是3个圆点，如果用自定义View绘制也很简单，就是绘制3个点，从左到右，或者从上到下。

![Android 自定义View绘制更多操作按钮.png](https://upload-images.jianshu.io/upload_images/1641428-0bd0663f2a020697.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理

关键参数

1. 圆点半径（画圆点必须的）
2. 圆点间的间距，一般采用百分比的方式，无论怎么改变View尺寸，都可以等比缩放，例如间距的计算就是将半径 * 固定比例计算的。

步骤：

- 水平3个圆点，画布移动到中心后，canvas.drawCircle()画一个中心圆点，再计算左边圆点的位置，和中心圆点坐标差别的地方就是x坐标不再为0，而是负的圆点间距，而右边圆点则是正的圆点间距。

- 垂直3个圆点，难道我们又要重新计算坐标？其实不需要，有了上面画水平3点的方法，水平和垂直的区别就是旋转90度而已，所以我们将Canvas旋转90度即可。

#### 完整代码

- 自定义属性

```
<declare-styleable name="MoreActionView">
    <!-- 点的颜色 -->
    <attr name="mav_color" format="color" />
    <!-- 点大小 -->
    <attr name="mav_dot_radius" format="integer|float|dimension" />
    <!-- 排列方向，水平或垂直 -->
    <attr name="mav_orientation" format="enum">
        <enum name="horizontal" value="1" />
        <enum name="vertical" value="2" />
    </attr>
</declare-styleable>
```

- Java代码

```
public class MoreActionView extends View {
    /**
     * 水平排列
     */
    private static final int ORIENTATION_HORIZONTAL = 1;
    /**
     * 垂直排列
     */
    private static final int ORIENTATION_VERTICAL = 2;

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
     * 点的半径
     */
    private float mDotRadius;
    /**
     * 每个点之间的距离
     */
    private float mDotDistance;
    /**
     * 排列方向
     */
    private int mOrientation;
    /**
     * 点颜色
     */
    private int mColor;

    public MoreActionView(Context context) {
        this(context, null);
    }

    public MoreActionView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MoreActionView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setAntiAlias(true);
        mPaint.setStrokeWidth(mDotRadius);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.MoreActionView, defStyleAttr, 0);
        mColor = array.getColor(R.styleable.MoreActionView_mav_color, Color.argb(255, 0, 0, 0));
        mOrientation = array.getInt(R.styleable.MoreActionView_mav_orientation, ORIENTATION_HORIZONTAL);
        mDotRadius = array.getDimension(R.styleable.MoreActionView_mav_dot_radius, dip2px(context, 2f));
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算半径
        float radius = Math.min(mViewWidth, mViewHeight) / 2f;
        //计算每个点之间的距离，半径的一般，再减去原点的直径
        mDotDistance = (radius / 2f) - (mDotRadius * 2);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //垂直排列
        if (mOrientation == ORIENTATION_VERTICAL) {
            canvas.rotate(90);
        } else {
            //水平排列
            canvas.rotate(0);
        }
        //画中心的点
        canvas.drawCircle(0, 0, mDotRadius, mPaint);
        //画左侧的点
        canvas.drawCircle(-mDotDistance, 0, mDotRadius, mPaint);
        //画右侧的点
        canvas.drawCircle(mDotDistance, 0, mDotRadius, mPaint);
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
<com.zh.cavas.sample.widget.MoreActionView
    android:id="@+id/more_action"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:mav_color="@android:color/black"
    app:mav_dot_radius="1.5dp"
    app:mav_orientation="vertical" />

<com.zh.cavas.sample.widget.MoreActionView
    android:id="@+id/more_action2"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:mav_color="@android:color/black"
    app:mav_dot_radius="1.5dp"
    app:mav_orientation="horizontal" />
```