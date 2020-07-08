#### ItemTouchHelper-拖动排序

18年底的时候，自己写了一个仿MIUI悬浮球的项目。当中有一个拖动排序功能菜单的功能，做之前了解到RecyclerView官方有提供拽托和侧滑的帮助类ItemTouchHelper，所以毫不犹豫就用RecyclerView来实现了。

![仿MIUI悬浮球，拽托排序菜单.gif](https://upload-images.jianshu.io/upload_images/1641428-ec523e1ef3eae412.gif?imageMogr2/auto-orient/strip)

#### 基本使用

使用非常简单，只需要2步，ItemTouchHelper.Callback需要复写的方法，在代码里注释都有写，大家一看就明白。RecyclerView的Adapter和条目Item布局我就不贴了，不是本文重点。

1. 创建ItemTouchHelper实例，定义ItemTouchHelper.Callback子类，复写方法。

```
public class CustomMenuItemTouchCallback extends ItemTouchHelper.Callback {
    private OnItemMoveCallback mOnItemMoveCallback;

    public CustomMenuItemTouchCallback(OnItemMoveCallback onItemMoveCallback) {
        this.mOnItemMoveCallback = onItemMoveCallback;
    }

    /**
     * 设置支持滑动的方向
     */
    @Override
    public int getMovementFlags(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder) {
        int dragFlags;
        int swipeFlags = 0;
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        //网格允许上下左右
        if (layoutManager instanceof GridLayoutManager) {
            dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN |
                    ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
        } else {
            dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
        }
        return makeMovementFlags(dragFlags, swipeFlags);
    }

    /**
     * 是否长按后才进行操作拽托操作
     */
    @Override
    public boolean isLongPressDragEnabled() {
        super.isLongPressDragEnabled();
        return true;
    }

    /**
     * 是否支持条目滑动，我们只需要拽托，所以这个滑动我们不需要
     */
    @Override
    public boolean isItemViewSwipeEnabled() {
        super.isItemViewSwipeEnabled();
        //不支持Item滑动
        return false;
    }

    /**
     * 发生拽托时，回调该方法，返回true、false决定是否可以拽托，可以使用target参数判断条目的ViewType来决定
     */
    @Override
    public boolean canDropOver(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder current, @NonNull RecyclerView.ViewHolder target) {
        if (mOnItemMoveCallback != null) {
            boolean isCanMove = mOnItemMoveCallback.isCanMove(current, target);
            //如果不能拖，直接拦截
            if (!isCanMove) {
                return false;
            }
        }
        return super.canDropOver(recyclerView, current, target);
    }

    /**
     * 条目发生交换时回调
     */
    @Override
    public boolean onMove(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, @NonNull RecyclerView.ViewHolder target) {
        //Item交换回调，交换adapter的数据
        if (mOnItemMoveCallback != null) {
            mOnItemMoveCallback.onItemMove(viewHolder.getAdapterPosition(), target.getAdapterPosition());
        }
        return true;
    }

    /**
     * 当滑动时回调，我们只有拽托，所以不处理
     */
    @Override
    public void onSwiped(@NonNull RecyclerView.ViewHolder viewHolder, int i) {
        //不支持滑动，不做处理
    }

    /**
     * 提供给外部实现的接口
     */
    public interface OnItemMoveCallback {
        /**
         * 是否可以移动，
         *
         * @param current 当前拽托的条目
         * @param target  想交换的位置的条目
         */
        boolean isCanMove(RecyclerView.ViewHolder current, @NonNull RecyclerView.ViewHolder target);

        /**
         * Item移动交换回调，这个回调要做Adapter数据交换
         *
         * @param fromPosition 从哪个位置
         * @param toPosition   到哪个位置
         */
        void onItemMove(int fromPosition, int toPosition);
    }
}
```

2. OnItemMoveCallback，自己定义的回调接口，主要是将：
	1. isCanMove()，拽托前，判断是否可以拽托。
	2. onItemMove()，发生拽托后，数据改变。

2. ItemTouchHelper.attachToRecyclerView()，将RecyclerView依附到ItemTouchHelper。

```
//条目数据
private val mDatas: ArrayList<FloatWindowActionModel> by lazy {
    ArrayList<FloatWindowActionModel>()
}

val touchCallback = CustomMenuItemTouchCallback(object : CustomMenuItemTouchCallback.OnItemMoveCallback {
    override fun isCanMove(current: RecyclerView.ViewHolder, target: RecyclerView.ViewHolder): Boolean {
        //这里我简单判断，2种ViewType类型相同的条目才可以互换
        return current.itemViewType == target.itemViewType
    }

    override fun onItemMove(fromPosition: Int, toPosition: Int) {
        //交换数据
        Collections.swap(mDatas, fromPosition, toPosition)
        mAdapter.notifyItemMoved(fromPosition, toPosition)
        //将新数据保存到本地
        saveNewActions(mDatas);
    }
})
val itemTouchHelper = ItemTouchHelper(touchCallback)
itemTouchHelper.attachToRecyclerView(mRecyclerView)
```

#### 项目地址

如果大家有兴趣的话，可以看一下这个仿MIUI悬浮球的项目。[Github地址]([https://github.com/hezihaog/miui_touch_assistant](https://github.com/hezihaog/miui_touch_assistant)
)