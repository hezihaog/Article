#### Android-仿微信旋转圆弧进度条

记得旧版本Android微信版本，加载图片时，会有一个无限圆弧转动Loading效果，但是现在去掉了，之前仿写过一次，现在将代码分享出来，如果大家需要可以拿去用~

#### 原理分析

后续补

#### 完整代码

- Java代码

```
public class RotateCircleProgressBar extends View {
    /**
     * View宽高
     */
    private int mWidth;
    private int mHeight;
    /**
     * 画笔
     */
    private Paint mCircleBorderPaint;
    private Paint mArcPaint;
    /**
     * 外圆颜色
     */
    private int mCircleColor = Color.parseColor("#4DEFEFF0");
    /**
     * 弧线颜色
     */
    private int mArcColor = Color.parseColor("#FFFFFF");
    /**
     * 中心点X、Y坐标
     */
    private int mCenterX;
    private int mCenterY;
    /**
     * 圆半径
     */
    private float mRadius;
    /**
     * 旋转角度
     */
    private float mAngle;
    /**
     * 绘制范围
     */
    private RectF mRect;

    public RotateCircleProgressBar(Context context) {
        super(context);
        init();
    }

    public RotateCircleProgressBar(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public RotateCircleProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        //外圆画笔
        mCircleBorderPaint = new Paint();
        mCircleBorderPaint.setStyle(Paint.Style.STROKE);
        mCircleBorderPaint.setColor(mCircleColor);
        mCircleBorderPaint.setStrokeWidth(dip2px(getContext(), 4f));
        mCircleBorderPaint.setAntiAlias(true);
        //弧线画笔
        mArcPaint = new Paint();
        mArcPaint.setStyle(Paint.Style.STROKE);
        mArcPaint.setColor(mArcColor);
        mArcPaint.setStrokeWidth(dip2px(getContext(), 3.5f));
        mArcPaint.setAntiAlias(true);
        //设置笔触为圆形
        mArcPaint.setStrokeCap(Paint.Cap.ROUND);

        ValueAnimator animator = ValueAnimator.ofFloat(0, 360);
        animator.setInterpolator(new LinearInterpolator());
        animator.setDuration(600);
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.setRepeatMode(ValueAnimator.RESTART);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //更新旋转角度
                mAngle = (Float) animation.getAnimatedValue();
                postInvalidate();
            }
        });
        animator.start();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = w;
        mHeight = h;
        //取出padding值
        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        int paddingTop = getPaddingTop();
        int paddingBottom = getPaddingBottom();
        //绘制范围
        mRect = new RectF();
        mRect.left = (float) paddingLeft;
        mRect.top = (float) paddingTop;
        mRect.right = (float) mWidth - paddingRight;
        mRect.bottom = (float) mHeight - paddingBottom;
        float diameter = (Math.min(mWidth, mHeight)) - paddingLeft - paddingRight;
        mRadius = (float) ((diameter / 2) * 0.98);
        //计算圆心的坐标
        mCenterX = mWidth / 2;
        mCenterY = mHeight / 2;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(measureSpec(widthMeasureSpec), measureSpec(heightMeasureSpec));
    }

    private int measureSpec(int measureSpec) {
        int result;
        int mode = MeasureSpec.getMode(measureSpec);
        int size = MeasureSpec.getSize(measureSpec);
        //默认大小
        int defaultSize = dip2px(getContext(), 30f);
        //指定宽高则直接返回
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else if (mode == MeasureSpec.AT_MOST) {
            //wrap_content的情况
            result = Math.min(defaultSize, size);
        } else {//未指定，则使用默认的大小
            result = defaultSize;
        }
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.scale(0.87f, 0.87f, mCenterX, mCenterY);
        //让画布旋转指定角度
        canvas.rotate(mAngle, mCenterX, mCenterY);
        //画外圆
        canvas.drawCircle(mCenterX, mCenterY, mRadius, mCircleBorderPaint);
        //画弧线
        canvas.drawArc(mRect, -90f, 200f, false, mArcPaint);
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }

    public static int px2dp(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }

    private int sp2px(Context context, float spVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,
                spVal, context.getResources().getDisplayMetrics());
    }
}
```

#### 布局代码

由于是无限进度的不断旋转，所以只需要布局里面引入，后面做显示、隐藏即可，大家记得拷贝完控件后，布局里面全包名要改成自己的。

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:background="#000000"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.hzh.rotate.circle.MainActivity">

    <com.hzh.rotate.circle.widget.RotateCircleProgressBar
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true" />
</RelativeLayout>
```