#### Android 自定义View 对勾CheckBox

今天在美团点外卖，有一个商品推荐的条目，上面的CheckBox是自定义的，虽然我们大部分都是用图片来自定义样式。但是还是可以自定义View来绘制的，只要画一个圆和对勾即可。

#### 最终效果

![最终效果.png](https://upload-images.jianshu.io/upload_images/1641428-644ef7b963db12dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1024)

![样式对比.png](https://upload-images.jianshu.io/upload_images/1641428-e8cd4cd652da0799.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1024)

#### 原理

1. 获取到View宽高后，计算半径，画一个圆。
2.  画一个对勾，仔细观察你会发现对勾，其实就是一个90度的直角，只是2条边长度不同。所以我们使用Canvas的平移、旋转的技巧就可以做出来了。

风格提供2种：

1. 美团外卖

	- 未勾选时，实心圆，背景为淡灰色，中间对勾为白色。
	- 勾选时，实心圆，背景为黄色，中间对勾为褐色。

2. 微软To-Do

	- 未勾选时，空心圆，中间没有对勾。
	- 勾选时，实心圆，背景为紫色，中间对勾为透明，漏出View的背景。（镂空）

美团外卖的样式，比较简单，只要先画圆，再画对勾即可。微软To-Do的样式稍微麻烦点，勾选时对勾为透明，漏出View的背景，镂空的效果，方案就是使用PorterDuffXfermode混合模式，让背景和对勾重合的部分（其实就是对勾的部分），不绘制颜色。

#### 完整代码

- 自定义属性

```
<declare-styleable name="HookCheckBox">
    <!-- 选中时圆的颜色 -->
    <attr name="hcb_check_circle_color" format="color" />
    <!-- 未选中时圆的颜色 -->
    <attr name="hcb_uncheck_circle_color" format="color" />
    <!-- 选中时的钩子的颜色 -->
    <attr name="hcb_check_hook_color" format="color" />
    <!-- 未选中时的钩子的颜色 -->
    <attr name="hcb_uncheck_hook_color" format="color" />
    <!-- 是否选中 -->
    <attr name="hcb_is_check" format="boolean" />
    <!-- 风格 -->
    <attr name="hcb_style" format="enum">
        <!-- 普通，对勾是有颜色的 -->
        <enum name="normal" value="1" />
        <!-- 镂空，就是对勾是透明的 -->
        <enum name="hollow_out" value="2" />
    </attr>
    <!-- 线宽 -->
    <attr name="hcb_line_width" format="float|dimension|reference" />
</declare-styleable>
```

- Java代码

```
public class HookCheckBox extends View {
    /**
     * View默认最小宽度
     */
    private static final int DEFAULT_MIN_WIDTH = 80;

    /**
     * 普通风格
     */
    private static final int STYLE_NORMAL = 1;
    /**
     * 镂空风格
     */
    private static final int STYLE_HOLLOW_OUT = 2;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;
    /**
     * 原型半径
     */
    private float mRadius;
    /**
     * 画笔
     */
    private Paint mPaint;
    /**
     * 钩子的线长度
     */
    private float mHookLineLength;
    /**
     * 是否选中
     */
    private boolean isCheck;
    /**
     * 选中时，圆的颜色
     */
    private int mCheckCircleColor;
    /**
     * 未选中时，圆的颜色
     */
    private int mUncheckCircleColor;
    /**
     * 选中时，钩子的颜色
     */
    private int mCheckHookColor;
    /**
     * 未选中时，钩子的颜色
     */
    private int mUncheckHookColor;
    /**
     * 混合模式
     */
    private PorterDuffXfermode mPorterDuffXfermode;
    /**
     * 线宽
     */
    private float mLineWidth;
    /**
     * 风格策略
     */
    private BaseStyleStrategy mStyleStrategy;
    /**
     * 切换改变回调
     */
    private OnCheckChangeListener mCheckListener;

    public HookCheckBox(Context context) {
        this(context, null);
    }

    public HookCheckBox(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public HookCheckBox(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        //画笔
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(mUncheckCircleColor);
        mPaint.setStrokeWidth(mLineWidth);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        //View禁用掉GPU硬件加速，切换到软件渲染模式
        setLayerType(View.LAYER_TYPE_SOFTWARE, null);
        mPorterDuffXfermode = new PorterDuffXfermode(PorterDuff.Mode.XOR);
        //设置点击事件
        setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
            }
        });
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        //默认的选中颜色
        int defaultCheckCircleColor = Color.argb(255, 254, 201, 77);
        //默认的未选中颜色
        int defaultUncheckCircleColor = Color.argb(255, 234, 234, 234);
        //默认选中的钩子颜色
        int defaultCheckHookColor = Color.argb(255, 53, 40, 33);
        //默认未选中的钩子颜色
        int defaultUncheckHookColor = Color.argb(255, 255, 255, 255);
        //默认风格
        int defaultStyle = STYLE_NORMAL;
        //线宽
        float defaultLineWidth = dip2px(context, 1.5f);
        //风格
        int style;
        if (attrs != null) {
            TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.HookCheckBox, defStyleAttr, 0);
            mCheckCircleColor = array.getColor(R.styleable.HookCheckBox_hcb_check_circle_color, defaultCheckCircleColor);
            mUncheckCircleColor = array.getColor(R.styleable.HookCheckBox_hcb_uncheck_circle_color, defaultUncheckCircleColor);
            mCheckHookColor = array.getColor(R.styleable.HookCheckBox_hcb_check_hook_color, defaultCheckHookColor);
            mUncheckHookColor = array.getColor(R.styleable.HookCheckBox_hcb_uncheck_hook_color, defaultUncheckHookColor);
            style = array.getInt(R.styleable.HookCheckBox_hcb_style, defaultStyle);
            isCheck = array.getBoolean(R.styleable.HookCheckBox_hcb_is_check, false);
            mLineWidth = array.getDimension(R.styleable.HookCheckBox_hcb_line_width, defaultLineWidth);
            array.recycle();
        } else {
            mCheckCircleColor = defaultCheckCircleColor;
            mUncheckCircleColor = defaultUncheckCircleColor;
            mCheckHookColor = defaultCheckHookColor;
            mUncheckHookColor = defaultUncheckHookColor;
            style = defaultStyle;
            mLineWidth = defaultLineWidth;
            isCheck = false;
        }
        if (style == STYLE_HOLLOW_OUT) {
            mStyleStrategy = new HollowOutStyleStrategy();
        } else if (style == STYLE_NORMAL) {
            mStyleStrategy = new NormalStyleStrategy();
        }
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算圆的半径
        mRadius = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.90f;
        //计算对勾的长度
        mHookLineLength = (Math.min(mViewWidth, mViewHeight) / 2f) * 0.8f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float left = -mViewWidth / 2f;
        float top = -mViewHeight / 2f;
        //保存图层
        int layerId = canvas.saveLayer(left, top, getWidth(), getHeight(), null, Canvas.ALL_SAVE_FLAG);
        //将画布中心移动到中心点
        canvas.translate(mViewWidth / 2, mViewHeight / 2);
        //画圆形背景
        mStyleStrategy.drawCircleBg(canvas);
        //画钩子
        mStyleStrategy.drawHook(canvas);
        //恢复图层
        canvas.restoreToCount(layerId);
    }

    private class BaseStyleStrategy {
        /**
         * 画圆形背景
         */
        protected void drawCircleBg(Canvas canvas) {
            //设置背景圆的颜色
            if (isCheck) {
                mPaint.setColor(mCheckCircleColor);
            } else {
                mPaint.setColor(mUncheckCircleColor);
            }
            canvas.drawCircle(0, 0, mRadius, mPaint);
        }

        /**
         * 画钩子
         */
        protected void drawHook(Canvas canvas) {
            //设置钩子的颜色
            if (isCheck) {
                mPaint.setColor(mCheckHookColor);
            } else {
                mPaint.setColor(mUncheckHookColor);
            }
            //画钩子要用描边风格
            mPaint.setStyle(Paint.Style.STROKE);
            canvas.save();
            //画布向下平移一半的半径长度
            canvas.translate(-(mRadius / 8f), mRadius / 3f);
            //旋转画布45度
            canvas.rotate(-45);
            Path path = new Path();
            path.reset();
            path.moveTo(0, 0);
            //向右画一条线
            path.lineTo(mHookLineLength, 0);
            //回到中心点
            path.moveTo(0, 0);
            //向上画一条线
            path.lineTo(0, -mHookLineLength / 2f);
            //画路径
            canvas.drawPath(path, mPaint);
            canvas.restore();
        }
    }

    private class NormalStyleStrategy extends BaseStyleStrategy {
        @Override
        protected void drawCircleBg(Canvas canvas) {
            //普通风格用填充风格
            mPaint.setStyle(Paint.Style.FILL);
            super.drawCircleBg(canvas);
        }
    }

    /**
     * 镂空风格
     */
    private class HollowOutStyleStrategy extends BaseStyleStrategy {
        @Override
        protected void drawCircleBg(Canvas canvas) {
            if (isCheck) {
                //镂空风格，选中时用填充
                mPaint.setStyle(Paint.Style.FILL);
            } else {
                //镂空风格，未选中时用描边
                mPaint.setStyle(Paint.Style.STROKE);
            }
            super.drawCircleBg(canvas);
        }

        @Override
        protected void drawHook(Canvas canvas) {
            //镂空风格，选中时，才画钩子
            if (isCheck) {
                //设置混合模式
                mPaint.setXfermode(mPorterDuffXfermode);
                super.drawHook(canvas);
                //去除混合模式
                mPaint.setXfermode(null);
            }
        }
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

    @Override
    public void setOnClickListener(@Nullable OnClickListener l) {
        super.setOnClickListener(new OnClickWrapper(l));
    }

    /**
     * 点击事件包裹，避免外部设置点击事件将内部的切换事件替换
     */
    private class OnClickWrapper implements OnClickListener {
        private OnClickListener mOriginListener;

        OnClickWrapper() {
        }

        OnClickWrapper(OnClickListener originListener) {
            mOriginListener = originListener;
        }

        @Override
        public void onClick(View view) {
            isCheck = !isCheck;
            invalidate();
            if (mCheckListener != null) {
                mCheckListener.onCheckChange(isCheck);
            }
            if (mOriginListener != null) {
                mOriginListener.onClick(view);
            }
        }
    }

    public boolean isCheck() {
        return isCheck;
    }

    public HookCheckBox setCheck(boolean check) {
        isCheck = check;
        if (mCheckListener != null) {
            mCheckListener.onCheckChange(check);
        }
        return this;
    }

    public interface OnCheckChangeListener {
        /**
         * 切换时回调
         *
         * @param isCheck 是否选中
         */
        void onCheckChange(boolean isCheck);
    }

    public void setOnCheckChangeListener(OnCheckChangeListener listener) {
        mCheckListener = listener;
    }
}
```

#### 简单使用

- 布局

```
<com.zh.cavas.sample.widget.HookCheckBox
    android:id="@+id/hook_checkbox"
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:hcb_is_check="false" />

<com.zh.cavas.sample.widget.HookCheckBox
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:hcb_is_check="true" />

<com.zh.cavas.sample.widget.HookCheckBox
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:hcb_check_circle_color="#6576D5"
    app:hcb_is_check="false"
    app:hcb_line_width="1dp"
    app:hcb_style="hollow_out"
    app:hcb_uncheck_circle_color="#818181" />

<com.zh.cavas.sample.widget.HookCheckBox
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_margin="10dp"
    app:hcb_check_circle_color="#6576D5"
    app:hcb_is_check="true"
    app:hcb_line_width="1dp"
    app:hcb_style="hollow_out"
    app:hcb_uncheck_circle_color="#818181" />
```

- Java代码

```
//设置切换监听
vHookCheckBox.setOnCheckChangeListener(new HookCheckBox.OnCheckChangeListener() {
    @Override
    public void onCheckChange(boolean isCheck) {
        String msg;
        if (isCheck) {
            msg = "开";
        } else {
            msg = "关";
        }
        Toast.makeText(getApplicationContext(), msg, Toast.LENGTH_SHORT).show();
    }
});
```