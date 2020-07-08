#### Android 仿vivo商店下载进度条

vivo商店在下载应用的时候，底部有一个圆角矩形的下载进度条，中间有一个进度文字，而且进度和文字交汇的时候，交汇部分的文字会从蓝色边为白色，会有一种一半白色字，一半蓝色字的效果。

#### 最终效果和对比vivo商店效果

![vivo应用商店下载效果.png](https://upload-images.jianshu.io/upload_images/1641428-de37599d6a597eaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

![最终效果.gif](https://upload-images.jianshu.io/upload_images/1641428-b6f12dd0a523a220.gif?imageMogr2/auto-orient/strip)

#### 分析1 - 计算进度

进度计算就比较简单了，我们通过复写onSizeChanged()方法，获取到控件的宽后，先计算当前进度百分比，再将百分比乘以宽度，就可以得到应该绘制的宽度了。

绘制圆角矩形需要传一个Rect，Rect的构造方法需要传4个位置，分别是left、top、right、bottom，我们主要是计算不断变化的right值。在drawProgress()方法中，right值为最大的宽度 * 进度百分比值。

```
/**
 * 控件宽
 */
private int mViewWidth;
/**
 * 控件高
 */
private int mViewHeight;
/**
 * 圆角弧度
 */
private float mRadius;
/**
 * 当前进度
 */
private int mProgress;
/**
 * 最大进度值
 */
private int mMaxProgress;

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    //画进度条
    drawProgress(canvas);
}

/**
 * 画进度
 */
private void drawProgress(Canvas canvas) {
    RectF rect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom());
    canvas.drawRect(rect, mProgressPaint);
}

/**
 * 获取当前进度值比值
 */
private float getProgressRatio() {
    return (mProgress / (mMaxProgress * 1.0f));
}

//------------ getFrameXxx()方法都是处理padding ------------
	
private float getFrameLeft() {
    return getPaddingStart();
}
	
private float getFrameRight() {
    return mViewWidth - getPaddingEnd();
}
	
private float getFrameTop() {
    return getPaddingTop();
}
	
private float getFrameBottom() {
    return mViewHeight - getPaddingBottom();
}
	
//------------ getFrameXxx()方法都是处理padding ------------
```

#### 分析2 - 绘制圆角矩形

背景和进度条，都是圆角矩形，我们可以使用canvas.drawRoundRect()的API来绘制。但是使用canvas.drawRoundRect()会有个问题，当进度比较小的时候，例如1%，计算出来的矩形长度比较小，会导致圆角不是满的，缺陷效果见动图（注意看刚从左侧出来时圆角的大小）。

![canvas.drawRoundRect()的问题.gif](https://upload-images.jianshu.io/upload_images/1641428-bebab462ef70f410.gif?imageMogr2/auto-orient/strip)

```
public class DownloadProgressView extends View {
    ...省略其他代码

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算出圆角半径
        mRadius = Math.min(mViewWidth, mViewHeight) / 2f;
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //画背景
        drawBg(canvas);
        //画进度条
        drawProgress(canvas);
    }
    
    /**
     * 画背景
     */
    private void drawBg(Canvas canvas) {
        canvas.drawRoundRect(new RectF(getFrameLeft(), getPaddingTop(), getFrameRight(), getFrameBottom()), mRadius, mRadius, mBgPaint);
    }

    /**
     * 画进度
     */
    private void drawProgress(Canvas canvas) {
        RectF rect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom());
        canvas.drawRoundRect(rect, mRadius, mRadius, mProgressPaint);
    }
    
    //------------ getFrameXxx()方法都是处理padding ------------

    private float getFrameLeft() {
        return getPaddingStart();
    }

    private float getFrameRight() {
        return mViewWidth - getPaddingEnd();
    }

    private float getFrameTop() {
        return getPaddingTop();
    }

    private float getFrameBottom() {
        return mViewHeight - getPaddingBottom();
    }

    //------------ getFrameXxx()方法都是处理padding ------------
}
```

- 解决方案

我的方案是这样的，不使用drawRoundRect()方法来绘制圆角矩形，而是先使用canvas.clipPath()方法，添加一个有圆角矩形的Path，来裁切画布，再drawRect()方法来绘制一个长矩形，就保持了圆角。

但clipPath()也有一个缺点，就是裁切出来的圆角会有锯齿，Paint设置了setAntiAlias(true)也没有用，大家有好的方法，可以评论中说出来，互相学习一下！

```
public class DownloadProgressView extends View {
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //裁剪圆角
        clipRound(canvas);
        //画背景
        drawBg(canvas);
        //画进度条
        drawProgress(canvas);
    }
    
    /**
     * 裁剪圆角
     */
    private void clipRound(Canvas canvas) {
        Path path = new Path();
        RectF roundRect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight(), getFrameBottom());
        path.addRoundRect(roundRect, mRadius, mRadius, Path.Direction.CW);
        canvas.clipPath(path);
    }

    /**
     * 画背景
     */
    private void drawBg(Canvas canvas) {
        canvas.drawRect(new RectF(getFrameLeft(), getPaddingTop(), getFrameRight(), getFrameBottom()), mBgPaint);
    }

    /**
     * 画进度
     */
    private void drawProgress(Canvas canvas) {
        RectF rect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom());
        canvas.drawRect(rect, mProgressPaint);
    }
}
```

#### 分析3 - 绘制文字和交汇

要让文字和进度交汇时，让蓝色变为白色，需要使用到PorterDuffXfermode。PorterDuffXfermode可以将2个图像进行合成，或者简单说为相交模式，而合成方式有16种，见下图。（Src蓝色方块为上层图层，Dst黄色圆形为下层图层。）

![PorterDuffXfermode模式图解.jpg](https://upload-images.jianshu.io/upload_images/1641428-9ea2f6643729c7aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查阅文档，我们发现SrcATop模式比较符合我们的需求，在SrcATop模式下，Src图层不覆盖Dst图层的像素会被抛弃，只保留Src图层覆盖Dst图层的图层像素。

我们的思路是这样的：画4个图层，最底部的灰色背景图层，再上一层蓝色进度图层，接着是文字图层，最上层是一层白色的进度图层，重点在画文字和白色进度图层是添加了PorterDuffXfermode（SrcATop模式）。

白色进度和蓝色进度的大小一直是同步的，但是因为PorterDuffXfermode的原因，白色图层未覆盖到文字时，一直都是被抛弃图层像素，相当于是透明的，只有当和文字相交时，才会绘制出白色图层，就显示出了一半白色文字，一半蓝色文字的效果，而其他部分都没有和文字相交都被抛弃，都为透明。

绘制文字、白色进度图层代码，见drawText()方法。

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    //裁剪圆角
    clipRound(canvas);
    //画背景
    drawBg(canvas);
    //画进度条
    drawProgress(canvas);
    //画文字图层
    drawText(canvas);
}

/**
 * 画文字
 */
private void drawText(Canvas canvas) {
    mTextPaint.setColor(mPercentageTextColor);
    //创建文字图层
    Bitmap textBitmap = Bitmap.createBitmap(mViewWidth, mViewHeight, Bitmap.Config.ARGB_8888);
    Canvas textCanvas = new Canvas(textBitmap);
    String textContent = mProgress + "%";
    //计算文字Y轴坐标
    float textY = mViewHeight / 2.0f - (mTextPaint.getFontMetricsInt().descent / 2.0f + mTextPaint.getFontMetricsInt().ascent / 2.0f);
    textCanvas.drawText(textContent, mViewWidth / 2.0f, textY, mTextPaint);
    //画最上层的白色图层，未相交时不会显示出来
    mTextPaint.setColor(mPercentageTextColor2);
    mTextPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP));
    textCanvas.drawRect(new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom()), mTextPaint);
    //画结合后的图层
    canvas.drawBitmap(textBitmap, getFrameLeft(), getFrameTop(), mTextPaint);
    mTextPaint.setXfermode(null);
    textBitmap.recycle();
}
```

#### 手势拓展

经过上面的进度、文字、交汇处理，进度条算是完整了，接下来拓展一下没有的东西，例如手势拖动，就像SeekBar一样，当我们触摸进度条时，进度会随着移动而改变进度。

我们先重写dispatchTouchEvent()方法，当down事件来到时，让父控件不拦截，我们进行拦截。接着重写onTouchEvent()方法，计算Move、Up时，移动的距离，计算出百分比和应有的进度值，然后不断调用setProgress()来更新进度。

```
public class DownloadProgressView extends View {
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int action = event.getAction();
        //拦截事件，然后让父类不进行拦截
        if (mControlMode == MODE_TOUCH && action == MotionEvent.ACTION_DOWN) {
            getParent().requestDisallowInterceptTouchEvent(true);
            return true;
        }
        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //拽托模式下才起效果
        if (mControlMode == MODE_TOUCH) {
            int action = event.getAction();
            //包裹Down事件时的x坐标
            if (action == MotionEvent.ACTION_DOWN) {
                mTouchDownX = event.getX();
                return true;
            } else if (action == MotionEvent.ACTION_MOVE || action == MotionEvent.ACTION_UP) {
                //Move或Up的时候，计算拽托进度
                float endX = event.getX();
                //计算公式：百分比值 = 移动距离 / 总长度
                float distanceX = Math.abs(endX - mTouchDownX);
                float ratio = (distanceX * 1.0f) / (getFrameRight() - getFrameLeft());
                //计算百分比应该有的进度：进度 = 总进度 * 进度百分比值
                float progress = mMaxProgress * ratio;
                setProgress((int) progress);
                return true;
            }
            return super.onTouchEvent(event);
        } else {
            return super.onTouchEvent(event);
        }
    }
}
```

#### 完整代码

- 自定义属性

```
<resources>
    <declare-styleable name="DownloadProgressView">
        <!-- 进度条背景 -->
        <attr name="dpv_bg" format="color" />
        <!-- 已有进度条背景 -->
        <attr name="dpv_progress_bg" format="color" />
        <!-- 百分比字体颜色 -->
        <attr name="dpv_percentage_text_color" format="color" />
        <!-- 上层百分比字体颜色 -->
        <attr name="dpv_percentage_text_color2" format="color" />
        <attr name="dpv_percentage_text_size" format="integer|float|dimension|reference" />
        <!-- 当前进度值 -->
        <attr name="dpv_progress" format="integer" />
        <!-- 最大进度值 -->
        <attr name="dpv_max_progress" format="integer" />
        <!-- 控制模式 -->
        <attr name="dpv_control_mode" format="enum">
            <enum name="normal" value="1" />
            <enum name="touch" value="2" />
        </attr>
    </declare-styleable>
</resources>
```

- 自定义View

```
public class DownloadProgressView extends View {
    /**
     * 控制模式-普通
     */
    private static final int MODE_NORMAL = 1;
    /**
     * 控制模式-触摸
     */
    private static final int MODE_TOUCH = 2;

    /**
     * View默认最小宽度
     */
    private int mDefaultMinWidth;
    /**
     * View默认最小高度
     */
    private int mDefaultMinHeight;

    /**
     * 控件宽
     */
    private int mViewWidth;
    /**
     * 控件高
     */
    private int mViewHeight;

    /**
     * 圆角弧度
     */
    private float mRadius;
    /**
     * 背景画笔
     */
    private Paint mBgPaint;
    /**
     * 进度画笔
     */
    private Paint mProgressPaint;
    /**
     * 文字画笔
     */
    private Paint mTextPaint;
    /**
     * 当前进度
     */
    private int mProgress;
    /**
     * 背景颜色
     */
    private int mBgColor;
    /**
     * 进度背景颜色
     */
    private int mProgressBgColor;
    /**
     * 进度百分比文字的颜色
     */
    private int mPercentageTextColor;
    /**
     * 第二层进度百分比文字的颜色
     */
    private int mPercentageTextColor2;
    /**
     * 进度百分比文字的字体大小
     */
    private float mPercentageTextSize;
    /**
     * 最大进度值
     */
    private int mMaxProgress;
    /**
     * 进度更新监听
     */
    private OnProgressUpdateListener mOnProgressUpdateListener;
    /**
     * 控制模式
     */
    private int mControlMode;
    /**
     * 按下时Down事件的x坐标
     */
    private float mTouchDownX;

    public DownloadProgressView(Context context) {
        this(context, null);
    }

    public DownloadProgressView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public DownloadProgressView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    private void init(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        initAttr(context, attrs, defStyleAttr);
        //取消硬件加速
        setLayerType(LAYER_TYPE_SOFTWARE, null);
        //背景画笔
        mBgPaint = new Paint();
        mBgPaint.setAntiAlias(true);
        mBgPaint.setColor(mBgColor);
        //进度画笔
        mProgressPaint = new Paint();
        mProgressPaint.setColor(mProgressBgColor);
        mProgressPaint.setAntiAlias(true);
        //文字画笔
        mTextPaint = new Paint();
        mTextPaint.setAntiAlias(true);
        mTextPaint.setTextSize(mPercentageTextSize);
        mTextPaint.setTextAlign(Paint.Align.CENTER);
        //计算默认宽、高
        mDefaultMinWidth = dip2px(context, 180f);
        mDefaultMinHeight = dip2px(context, 40f);
    }

    private void initAttr(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.DownloadProgressView, defStyleAttr, 0);
        mBgColor = array.getColor(R.styleable.DownloadProgressView_dpv_bg, Color.argb(100, 169, 169, 169));
        mProgressBgColor = array.getColor(R.styleable.DownloadProgressView_dpv_progress_bg, Color.GRAY);
        //进度百分比文字的颜色，默认和进度背景颜色一致
        mPercentageTextColor = array.getColor(R.styleable.DownloadProgressView_dpv_percentage_text_color, mProgressBgColor);
        //第二层，进度百分比文字的颜色
        mPercentageTextColor2 = array.getColor(R.styleable.DownloadProgressView_dpv_percentage_text_color2, Color.WHITE);
        //进度百分比文字的字体颜色
        mPercentageTextSize = array.getDimension(R.styleable.DownloadProgressView_dpv_percentage_text_size, sp2px(context, 15f));
        //当前进度值
        mProgress = array.getInt(R.styleable.DownloadProgressView_dpv_progress, 0);
        //最大进度值
        mMaxProgress = array.getInteger(R.styleable.DownloadProgressView_dpv_max_progress, 100);
        //控制模式
        mControlMode = array.getInt(R.styleable.DownloadProgressView_dpv_control_mode, MODE_NORMAL);
        array.recycle();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mViewWidth = w;
        mViewHeight = h;
        //计算出圆角半径
        mRadius = Math.min(mViewWidth, mViewHeight) / 2f;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //裁剪圆角
        clipRound(canvas);
        //画背景
        drawBg(canvas);
        //画进度条
        drawProgress(canvas);
        //画文字图层
        drawText(canvas);
    }

    //------------ getFrameXxx()方法都是处理padding ------------

    private float getFrameLeft() {
        return getPaddingStart();
    }

    private float getFrameRight() {
        return mViewWidth - getPaddingEnd();
    }

    private float getFrameTop() {
        return getPaddingTop();
    }

    private float getFrameBottom() {
        return mViewHeight - getPaddingBottom();
    }

    //------------ getFrameXxx()方法都是处理padding ------------

    /**
     * 裁剪圆角
     */
    private void clipRound(Canvas canvas) {
        Path path = new Path();
        RectF roundRect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight(), getFrameBottom());
        path.addRoundRect(roundRect, mRadius, mRadius, Path.Direction.CW);
        canvas.clipPath(path);
    }

    /**
     * 画背景
     */
    private void drawBg(Canvas canvas) {
        canvas.drawRect(new RectF(getFrameLeft(), getPaddingTop(), getFrameRight(), getFrameBottom()), mBgPaint);
    }

    /**
     * 画进度
     */
    private void drawProgress(Canvas canvas) {
        RectF rect = new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom());
        canvas.drawRect(rect, mProgressPaint);
    }

    /**
     * 画文字
     */
    private void drawText(Canvas canvas) {
        mTextPaint.setColor(mPercentageTextColor);
        //创建文字图层
        Bitmap textBitmap = Bitmap.createBitmap(mViewWidth, mViewHeight, Bitmap.Config.ARGB_8888);
        Canvas textCanvas = new Canvas(textBitmap);
        String textContent = mProgress + "%";
        //计算文字Y轴坐标
        float textY = mViewHeight / 2.0f - (mTextPaint.getFontMetricsInt().descent / 2.0f + mTextPaint.getFontMetricsInt().ascent / 2.0f);
        textCanvas.drawText(textContent, mViewWidth / 2.0f, textY, mTextPaint);
        //画最上层的白色图层，未相交时不会显示出来
        mTextPaint.setColor(mPercentageTextColor2);
        mTextPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP));
        textCanvas.drawRect(new RectF(getFrameLeft(), getFrameTop(), getFrameRight() * getProgressRatio(), getFrameBottom()), mTextPaint);
        //画结合后的图层
        canvas.drawBitmap(textBitmap, getFrameLeft(), getFrameTop(), mTextPaint);
        mTextPaint.setXfermode(null);
        textBitmap.recycle();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(handleMeasure(widthMeasureSpec, true), handleMeasure(heightMeasureSpec, false));
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int action = event.getAction();
        //拦截事件，然后让父类不进行拦截
        if (mControlMode == MODE_TOUCH && action == MotionEvent.ACTION_DOWN) {
            getParent().requestDisallowInterceptTouchEvent(true);
            return true;
        }
        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //拽托模式下才起效果
        if (mControlMode == MODE_TOUCH) {
            int action = event.getAction();
            //包裹Down事件时的x坐标
            if (action == MotionEvent.ACTION_DOWN) {
                mTouchDownX = event.getX();
                return true;
            } else if (action == MotionEvent.ACTION_MOVE || action == MotionEvent.ACTION_UP) {
                //Move或Up的时候，计算拽托进度
                float endX = event.getX();
                //计算公式：百分比值 = 移动距离 / 总长度
                float distanceX = Math.abs(endX - mTouchDownX);
                float ratio = (distanceX * 1.0f) / (getFrameRight() - getFrameLeft());
                //计算百分比应该有的进度：进度 = 总进度 * 进度百分比值
                float progress = mMaxProgress * ratio;
                setProgress((int) progress);
                return true;
            }
            return super.onTouchEvent(event);
        } else {
            return super.onTouchEvent(event);
        }
    }

    /**
     * 处理MeasureSpec
     */
    private int handleMeasure(int measureSpec, boolean isWidth) {
        int result;
        if (isWidth) {
            result = mDefaultMinWidth;
        } else {
            result = mDefaultMinHeight;
        }
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

    /**
     * 设置进度最大值
     */
    public DownloadProgressView setMaxProgress(int maxProgress) {
        mMaxProgress = maxProgress;
        invalidate();
        return this;
    }

    /**
     * 设置进度
     */
    public void setProgress(int progress) {
        if (progress >= 0 && progress <= 100) {
            mProgress = progress;
            invalidate();
            if (mOnProgressUpdateListener != null) {
                mOnProgressUpdateListener.onProgressUpdate(progress);
            }
        }
    }

    /**
     * 获取当前进度
     */
    public int getProgress() {
        return mProgress;
    }

    /**
     * 获取最大进度
     */
    public int getMaxProgress() {
        return mMaxProgress;
    }

    public interface OnProgressUpdateListener {
        /**
         * 进度更新时回调
         *
         * @param progress 当前进度
         */
        void onProgressUpdate(int progress);
    }

    public void setOnProgressUpdateListener(OnProgressUpdateListener onProgressUpdateListener) {
        mOnProgressUpdateListener = onProgressUpdateListener;
    }

    /**
     * 获取当前进度值比值
     */
    private float getProgressRatio() {
        return (mProgress / (mMaxProgress * 1.0f));
    }

    public static int sp2px(Context context, float spValue) {
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }
}
```

#### 具体使用

- 在布局中添加控件

```
<com.zh.cavas.sample.widget.DownloadProgressView
        android:id="@+id/download_progress"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        android:layout_marginEnd="20dp"
        app:dpv_bg="#E6EAFB"
        app:dpv_max_progress="100"
        app:dpv_percentage_text_color2="@color/white"
        app:dpv_percentage_text_size="14sp"
        app:dpv_progress="0"
        app:dpv_progress_bg="#3548FE"
        tools:dpv_progress="50" />
```

- Java代码

```
private DownloadProgressView vDownloadProgressView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        vDownloadProgressView = view.findViewById(R.id.download_progress);
        //设置进度监听
        vDownloadProgressView.setOnProgressUpdateListener(new DownloadProgressView.OnProgressUpdateListener() {
            @Override
            public void onProgressUpdate(int progress) {
                vProgressIndicator.setText("当前进度：" + progress);
            }
        });
        //手动设置当前进度
        vDownloadProgressView.setProgress(0);
        //设置最大值
        vDownloadProgressView.setMaxProgress(100);
    }
}
```

#### 参考文章

[一个简单又不简单的进度条
](https://www.jianshu.com/p/398aefc62dec)

[Android - PorterDuffXfermode实现进度条]([https://www.jianshu.com/p/eab4564c8382](https://www.jianshu.com/p/eab4564c8382)
)