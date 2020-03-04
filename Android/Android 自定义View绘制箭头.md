#### Android 自定义View绘制箭头

一般顶部栏的返回键都是让UI切图，其实还可以自定义View绘制，本篇就闲来蛋疼绘制一个箭头返回键吧，有Material Design和微信风格（就是没有中间那条横线）。

![Android 自定义View绘制箭头.png](https://upload-images.jianshu.io/upload_images/1641428-78b1f2e751ab3e29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 原理

我们观察一下，控件中点，向右一条横线，再从中心点2个45度角的2条线，如果用数学的惯性思维，会想到使用三角函数，计算出2条45度线终点的x，y坐标。但其实Canvas有rotate旋转的API，就不需要那么多麻烦的计算了，2个45度的角，其实就是一个90度的直角，我们可以先画一个直角，然后再旋转45度，就可以了。

#### 完整代码

- 自定义属性

```
<declare-styleable name="BackArrowView">
        <!-- 箭头颜色 -->
        <attr name="bav_color" format="color" />
        <!-- 箭头线宽 -->
        <attr name="bav_stroke_width" format="integer|float|dimension" />
        <!-- 模式 -->
        <attr name="bav_arrow_style" format="enum">
            <!-- Google Material Design -->
            <enum name="material_design" value="1" />
            <!-- 微信风格 -->
            <enum name="wechat_design" value="2" />
        </attr>
    </declare-styleable>
```

- Java代码

```
public class BackArrowView extends View {
    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 100;
    /**
     * Material Design风格
     */
    private static final int ARROW_STYLE_MATERIAL_DESIGN = 1;
    /**
     * 微信风格
     */
    private static final int ARROW_STYLE_WECHAT_DESIGN = 2;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;
    /**
     * 箭头开始的距离
     */
    private float mArrowStartDistance;
    /**
     * 箭头的2个边的长度
     */
    private float mArrowLineLength;
    /**
     * 箭头颜色
     */
    private int mColor;
    /**
     * 箭头粗细
     */
    private float mArrowStrokeWidth;
    /**
     * 风格模式
     */
    private int mArrowStyle;

    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 箭头Path
     */
    private Path mArrowPath;

    public BackArrowView(Context context) {
        this(context, null);
    }

    public BackArrowView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public BackArrowView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        //使用Path必须使用STROKE，使用FILL是画不了的
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setAntiAlias(true);
        mPaint.setStrokeWidth(mArrowStrokeWidth);
        //设置拐角形状为圆形，3条线相接处则不会有缝隙了
        mPaint.setStrokeJoin(Paint.Join.ROUND);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.BackArrowView, defStyleAttr, 0);
        mColor = array.getColor(R.styleable.BackArrowView_bav_color, Color.argb(255, 0, 0, 0));
        mArrowStrokeWidth = array.getDimension(R.styleable.BackArrowView_bav_stroke_width, dip2px(context, 2f));
        mArrowStyle = array.getInt(R.styleable.BackArrowView_bav_arrow_style, ARROW_STYLE_MATERIAL_DESIGN);
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算半径
        float radius = Math.min(mViewWidth, mViewHeight) / 2f;
        //计算箭头起始位置
        if (mArrowStyle == ARROW_STYLE_MATERIAL_DESIGN) {
            mArrowStartDistance = radius / 3f;
        } else if (mArrowStyle == ARROW_STYLE_WECHAT_DESIGN) {
            mArrowStartDistance = radius / 4f;
        }
        //计算箭头长度
        mArrowLineLength = radius * 0.63f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将画布中心移动到中心点偏左位置
        canvas.translate((mViewWidth / 2f) - mArrowStartDistance, mViewHeight / 2);
        //将画布旋转45度，让后面画的直角旋转
        canvas.rotate(45);
        if (mArrowPath == null) {
            mArrowPath = new Path();
        }
        mArrowPath.reset();
        //画第一条线
        mArrowPath.lineTo(0, -mArrowLineLength);
        //画第二条线
        mArrowPath.moveTo(0, 0);
        mArrowPath.lineTo(mArrowLineLength, 0);
        //Google Material Design风格才有中间的线
        if (mArrowStyle == ARROW_STYLE_MATERIAL_DESIGN) {
            //画中间的线
            mArrowPath.moveTo(0, 0);
            mArrowPath.lineTo(mArrowLineLength, -mArrowLineLength);
        }
        //闭合路径
        mArrowPath.close();
        //画路径
        canvas.drawPath(mArrowPath, mPaint);
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
<com.zh.cavas.sample.widget.BackArrowView
    android:id="@+id/back_arrow"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:bav_arrow_style="wechat_design"
    app:bav_color="@android:color/black"
    app:bav_stroke_width="1.5dp" />

<com.zh.cavas.sample.widget.BackArrowView
    android:id="@+id/back_arrow2"
    android:layout_width="40dp"
    android:layout_height="40dp"
    android:layout_marginTop="20dp"
    app:bav_arrow_style="material_design"
    app:bav_color="@android:color/black"
    app:bav_stroke_width="1.5dp" />
```