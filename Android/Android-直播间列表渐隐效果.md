#### Android-直播间列表渐隐效果

直播间的打赏榜需要加一个渐变效果，类似映客APP直播间的消息列表，一开始使用xml-shape的gradient标签层叠到RecyclerView上，但是发现效果不太对，总有一层蒙版割裂列表。

随后和设计大佬沟通，设计师说这个不是渐变效果，是渐隐，没有渐变的2个颜色值。渐隐效果安卓并没有原生api可以支持呀，随后问了iOS的同学，他们实现是添加一个CAGradientLayer（渐变蒙版图层）和TableView（列表控件）的图层合并。

这时候，我才明白，并不是单纯的叠一层渐变，而是要将渐变和RecyclerView的图层合并，再draw。

最后，如果列表滚动到顶部，则不绘制，其他时候需要绘制，我们监听RecyclerView滚动即可，滚动监听我封装到了RecyclerViewScrollHelper这个类。

#### 最终效果

![Android-直播间列表渐隐效果.jpeg](https://upload-images.jianshu.io/upload_images/1641428-cb18fee360262a06.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

![Android-直播间列表渐隐效果_动效.gif](https://upload-images.jianshu.io/upload_images/1641428-32ea6f0a165cef39.gif?imageMogr2/auto-orient/strip)

#### 思路

要在RecyclerView上的Canvas上draw，可以继承RecyclerView来实现，但是耦合到了RecyclerView，我们可以使用ItemDecoration，添加一个条目装饰器，在RecyclerView上绘制。使用Xfermode，融合2个图层。

#### 辅助工具类

- RecyclerViewScrollHelper，列表滚动帮助类

```
public class RecyclerViewScrollHelper {
    /**
     * 第一次进入界面时也会回调滚动，所以当手动滚动再监听
     */
    private boolean isNotFirst = false;
    /**
     * 列表控件
     */
    private RecyclerView scrollingView;
    /**
     * 回调
     */
    private Callback callback;

    public void attachRecyclerView(RecyclerView scrollingView, Callback callback) {
        this.scrollingView = scrollingView;
        this.callback = callback;
        setup();
    }

    private void setup() {
        scrollingView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                isNotFirst = true;
                if (callback != null) {
                    //如果滚动到最后一行，RecyclerView.canScrollVertically(1)的值表示是否能向上滚动，false表示已经滚动到底部
                    if (newState == RecyclerView.SCROLL_STATE_IDLE &&
                            !recyclerView.canScrollVertically(1)) {
                        callback.onScrolledToBottom();
                    }
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                if (callback != null && isNotFirst) {
                    //RecyclerView.canScrollVertically(-1)的值表示是否能向下滚动，false表示已经滚动到顶部
                    if (!recyclerView.canScrollVertically(-1)) {
                        callback.onScrolledToTop();
                    }
                    //下滑
                    if (dy < 0) {
                        callback.onScrolledToDown();
                    }
                    //上滑
                    if (dy > 0) {
                        callback.onScrolledToUp();
                    }
                }
            }
        });
    }

    public interface Callback {
        /**
         * 向下滚动
         */
        void onScrolledToDown();

        /**
         * 向上滚动
         */
        void onScrolledToUp();

        /**
         * 滚动到了顶部
         */
        void onScrolledToTop();

        /**
         * 滚动到了底部
         */
        void onScrolledToBottom();
    }

    public static class CallbackAdapter implements Callback {
        @Override
        public void onScrolledToDown() {
        }

        @Override
        public void onScrolledToUp() {
        }

        @Override
        public void onScrolledToTop() {
        }

        @Override
        public void onScrolledToBottom() {
        }
    }

    /**
     * 马上滚动到顶部
     */
    public void moveToTop() {
        if (scrollingView != null) {
            scrollingView.scrollToPosition(0);
        }
    }

    /**
     * 缓慢滚动到顶部
     */
    public void smoothMoveToTop() {
        if (scrollingView != null) {
            scrollingView.smoothScrollToPosition(0);
        }
    }
}
```

- ViewUtils，dp转换px

```
public class ViewUtils {
    /**
     * dip换算成像素数量
     */
    public static int dipToPx(Context context, float dip) {
        float density = context.getApplicationContext().getResources().getDisplayMetrics().density;
        return roundUp(dip * context.getResources().getDisplayMetrics().density);
    }

    private static int roundUp(float f) {
        return (int) (0.5f + f);
    }
}
```

#### 类结构

- 渐变策略，我将顶部、底部的渐变绘制抽成2个策略类，而绘制方法onDrawOver()，getShader()获取着色器，则分拆到一个ShadowStrategy策略接口。

```
/**
 * 渐变策略
 */
private abstract class ShadowStrategy(
    val shadowHeight: Float,
    val paint: Paint
) {
    /**
     * 绘制
     */
    abstract fun onDrawOver(canvas: Canvas, parent: RecyclerView, state: RecyclerView.State)

    /**
     * 获取着色器
     */
    abstract fun getShader(parent: RecyclerView): Shader
}
```

- 顶部渐变

```
/**
 * 顶部渐变
 */
private inner class TopShadowStrategy(shadowHeight: Float, paint: Paint) :
    ShadowStrategy(shadowHeight, paint) {

    private lateinit var mScrollHelper: RecyclerViewScrollHelper
    /**
     * 是否可以绘制渐变
     */
    private var isCanDrawShadow: Boolean = false

    override fun onDrawOver(canvas: Canvas, parent: RecyclerView, state: RecyclerView.State) {
        if (isCanDrawShadow) {
            val left = 0f
            val top = 0f
            val right = parent.width.toFloat()
            val bottom = shadowHeight
            val topShadowRect = RectF(left, top, right, bottom)
            canvas.drawRect(topShadowRect, paint)
        }
    }

    override fun getShader(parent: RecyclerView): Shader {
        if (!this::mScrollHelper.isInitialized) {
            mScrollHelper = RecyclerViewScrollHelper()
            mScrollHelper.attachRecyclerView(
                parent,
                object : RecyclerViewScrollHelper.CallbackAdapter() {
                    override fun onScrolledToTop() {
                        //到了顶部就不能渲染
                        isCanDrawShadow = false
                    }

                    override fun onScrolledToUp() {
                        super.onScrolledToUp()
                        //向上滚动，列表向下移动，则需要渲染
                        isCanDrawShadow = true
                    }
                })
        }
        return run {
            //渐变起始x，y坐标
            val x0 = 0f
            val y0 = 0f
            //渐变结束x，y坐标
            val x1 = 0f
            val y1 = shadowHeight
            //渐变颜色的开始、结束颜色
            val startColor = Color.TRANSPARENT
            val endColor = Color.BLACK
            val colors = intArrayOf(startColor, endColor)
            //渐变位置数组
            val positions = null

            //指定控件区域大于指定的渐变区域时，空白区域的颜色填充方法
            //CLAMP：边缘拉伸，为被shader覆盖区域，使用shader边界颜色进行填充
            //REPEAT：在水平和垂直两个方向上重复，相邻图像没有间隙
            //MIRROR：以镜像的方式在水平和垂直两个方向上重复，相邻图像有间隙
            val tile = Shader.TileMode.CLAMP
            LinearGradient(
                x0, y0, x1, y1,
                colors, positions, tile
            )
        }
    }
}
```

- 底部渐变

```
/**
 * 底部渐变
 */
private inner class BottomShadowStrategy(shadowHeight: Float, paint: Paint) :
    ShadowStrategy(shadowHeight, paint) {

    /**
     * 是否可以绘制渐变
     */
    private var isCanDrawShadow: Boolean = true

    override fun onDrawOver(canvas: Canvas, parent: RecyclerView, state: RecyclerView.State) {
        //底部渐变，必须指定数量的条目才可以绘制
        isCanDrawShadow = (parent.adapter?.itemCount ?: 0) > 8
        if (isCanDrawShadow) {
            val left = 0f
            val top = parent.height - shadowHeight
            val right = parent.width.toFloat()
            val bottom = parent.height.toFloat()
            val topShadowRect = RectF(left, top, right, bottom)
            canvas.drawRect(topShadowRect, paint)
        }
    }

    override fun getShader(parent: RecyclerView): Shader {
        return run {
            //渐变起始x，y坐标
            val x0 = 0f
            val y0 = parent.height - shadowHeight
            //渐变结束x，y坐标
            val x1 = 0f
            val y1 = parent.height.toFloat()

            //渐变颜色的开始、结束颜色
            val startColor = Color.BLACK
            val endColor = Color.TRANSPARENT
            val colors = intArrayOf(startColor, endColor)
            //渐变位置数组
            val positions = null

            //指定控件区域大于指定的渐变区域时，空白区域的颜色填充方法
            //CLAMP：边缘拉伸，为被shader覆盖区域，使用shader边界颜色进行填充
            //REPEAT：在水平和垂直两个方向上重复，相邻图像没有间隙
            //MIRROR：以镜像的方式在水平和垂直两个方向上重复，相邻图像有间隙
            val tile = Shader.TileMode.CLAMP
            LinearGradient(
                x0, y0, x1, y1,
                colors, positions, tile
            )
        }
    }
}
```

- 配置到RecyclerView，通过addItemDecoration()方法添加装饰器。

```
val context = this
val paint = Paint()
//渐变的高度
val shadowHeight = ViewUtils.dipToPx(context, 80f).toFloat()
//顶部渐变
val topShadowStrategy = TopShadowStrategy(shadowHeight, paint)
//底部渐变
val bottomShadowStrategy = BottomShadowStrategy(shadowHeight, paint)

//混合模式
val xfermode: Xfermode = PorterDuffXfermode(PorterDuff.Mode.DST_IN)
var layerId = 0
//配置装饰器
vRankList.addItemDecoration(object : RecyclerView.ItemDecoration() {
    /**
     * 可以实现类似绘制背景的效果，内容在上面
     */
    override fun onDraw(canvas: Canvas, parent: RecyclerView, state: RecyclerView.State) {
        super.onDraw(canvas, parent, state)
        layerId = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            canvas.saveLayer(
                0.0f,
                0.0f,
                parent.width.toFloat(),
                parent.height.toFloat(),
                paint
            )
        } else {
            canvas.saveLayer(
                0.0f,
                0.0f,
                parent.width.toFloat(),
                parent.height.toFloat(),
                paint,
                Canvas.ALL_SAVE_FLAG
            )
        }
    }

    /**
     * 可以绘制在内容的上面，覆盖内容
     */
    override fun onDrawOver(
        canvas: Canvas,
        parent: RecyclerView,
        state: RecyclerView.State
    ) {
        super.onDrawOver(canvas, parent, state)
        paint.xfermode = xfermode

        //画顶部渐变
        paint.shader = topShadowStrategy.getShader(parent)
        topShadowStrategy.onDrawOver(canvas, parent, state)

        //画底部渐变
        paint.shader = bottomShadowStrategy.getShader(parent)
        bottomShadowStrategy.onDrawOver(canvas, parent, state)

        paint.xfermode = null
        canvas.restoreToCount(layerId)
    }
})
```

#### 添加渐变到列表

```
class MainActivity : AppCompatActivity() {
    private lateinit var vRankList: RecyclerView

    private val mListItems = Items()
    private val mListAdapter = MultiTypeAdapter(mListItems).apply {
        register(RankListItemModel::class.java, RankListItemBinder())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        findView()
        bindView()
        setData()
    }

    private fun findView() {
        vRankList = findViewById(R.id.rank_list)
    }

    private fun bindView() {
        supportActionBar?.title = "打赏榜"
        vRankList.run {
            adapter = mListAdapter
            layoutManager = LinearLayoutManager(this@MainActivity)
            setupRankListAlphaStyle()
        }
    }

    private fun setData() {
        val textColorResId = android.R.color.white
        for (index in 1..15) {
            mListItems.add(
                RankListItemModel(
                    index,
                    generateNickName(index),
                    "",
                    (100 + index).toString(),
                    textColorResId
                )
            )
        }
        mListAdapter.notifyDataSetChanged()
    }

    /**
     * 生成昵称
     */
    private fun generateNickName(index: Int): String {
        val nicknames = listOf(
            "迪丽热巴", "黄晓明", "杨幂",
            "彭于晏", "柳岩", "李易峰", "陈伟霆", "刘诗诗", "张艺兴", "成龙",
            "蔡徐坤", "赵丽颖", "王一博", "阚清子", "刘亦菲", "郑爽", "杨紫",
            "关晓彤", "唐嫣", "胡歌", "宋茜", "周杰伦", "吴亦凡", "周冬雨", "华晨宇"
        )
        val position = index % nicknames.size
        return nicknames[position]
    }

    /**
     * 设置排行榜列表透明度风格
     */
    private fun setupRankListAlphaStyle() {
        val context = this
        val paint = Paint()
        val shadowHeight = ViewUtils.dipToPx(context, 80f).toFloat()
        //顶部渐变
        val topShadowStrategy = TopShadowStrategy(shadowHeight, paint)
        //底部渐变
        val bottomShadowStrategy = BottomShadowStrategy(shadowHeight, paint)

        //混合模式
        val xfermode: Xfermode = PorterDuffXfermode(PorterDuff.Mode.DST_IN)
        var layerId = 0
        //设置装饰器
        vRankList.addItemDecoration(object : RecyclerView.ItemDecoration() {
            /**
             * 可以实现类似绘制背景的效果，内容在上面
             */
            override fun onDraw(canvas: Canvas, parent: RecyclerView, state: RecyclerView.State) {
                super.onDraw(canvas, parent, state)
                layerId = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    canvas.saveLayer(
                        0.0f,
                        0.0f,
                        parent.width.toFloat(),
                        parent.height.toFloat(),
                        paint
                    )
                } else {
                    canvas.saveLayer(
                        0.0f,
                        0.0f,
                        parent.width.toFloat(),
                        parent.height.toFloat(),
                        paint,
                        Canvas.ALL_SAVE_FLAG
                    )
                }
            }

            /**
             * 可以绘制在内容的上面，覆盖内容
             */
            override fun onDrawOver(
                canvas: Canvas,
                parent: RecyclerView,
                state: RecyclerView.State
            ) {
                super.onDrawOver(canvas, parent, state)
                paint.xfermode = xfermode

                //画顶部渐变
                paint.shader = topShadowStrategy.getShader(parent)
                topShadowStrategy.onDrawOver(canvas, parent, state)

                //画底部渐变
                paint.shader = bottomShadowStrategy.getShader(parent)
                bottomShadowStrategy.onDrawOver(canvas, parent, state)

                paint.xfermode = null
                canvas.restoreToCount(layerId)
            }
        })
    }
}
```

#### 总结

这种效果需要用到Xfermode，所以需要了解一下，常用的几个模式，以及LinearGradient的构造方法的那些参数，尤其是渐变开始、结束坐标，如果算不对，渐变方向就不对，再叠加Xfermode时，会比较难看出问题，最好先不加Xfermode，先让渐变方向正确后，再添加Xfermode。

完整代码我上传到了[Github](https://github.com/hezihaog/LiveRankListGradientExample)，有需要或感兴趣的同学可以clone。

### 参考资料

- iOS：[iOS实现参考](https://www.jianshu.com/p/a433ff0b2949)
- Android：[Android实现参考](https://blog.csdn.net/linyukun6422/article/details/52516022)
- 其他：[LinearGradient线性渐变-构造属性介绍](https://blog.csdn.net/u010126792/article/details/85237085)