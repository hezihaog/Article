#### Android透明状态栏和软键盘配合的坑

一般我们为了让键盘自动将界面弹起，会在清单文件中配置windowSoftInputMode，配置为adjustResize或adjustPan。

1. adjustResize，键盘弹起时，将界面Layout高度压缩，留出空间显示软键盘。
2. adjustPan，需要存在滚动控件，键盘弹起时，滚动列表，留出控件显示软键盘，如果没有滚动控件，则将全部控件上移，而不会压缩界面Layout。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.hule.dashi.live">

    <application>
        <activity
            android:name=".AudioLiveRoomActivity"
            android:windowSoftInputMode="adjustResize" />
    </application>
</manifest>
```

#### 遇到的问题

在直播模块开发中，需要将状态栏透明。然而透明后，却发现一个大坑，软键盘模式失效了。无论windowSoftInputMode设置adjustResize还是adjustPan，键盘都会将界面覆盖，而不会将输入框顶起。

- 搜索了一番，结果发现是谷歌在2.2年代就留下的坑，却一直没修复...那么我们是不是可以监听软键盘弹起事件，获取软键盘高度，手动压缩布局Layout高度不就可以了？

- 结果Android并没有提供软键盘弹起的监听事件，擦，这么坑的？那么还有什么办法么？

#### 办法还是有的

搜索了一下，原来已经有大神提供了处理类，名叫AndroidBug5497Workaround，大概原理就是给Activity的根View注册ViewTreeObserver.OnGlobalLayoutListener。键盘弹起时，监听会回调，我们拿取原Activity始布局高度和Activity可见高度进行相减，即可计算出键盘高度，再将布局高度重新设置，就可以将布局上移。同时可以将键盘改变作为回调监听提供给外部。

- 结果又发现问题，在刘海屏上有问题，底部会空出一大片，而我修改后的做法是：计算出键盘高度后，给布局底部控件设置一个MarginBottom，自然会有一种顶出输入框的效果。

#### 源码解析

首先是生成软键盘的监听。提供给外部监听软键盘监听。

1. 首先我们传入rootView构造KeyboardFix。
2. 给rootView设置addOnGlobalLayoutListener回调。
3. addOnGlobalLayoutListener的onGlobalLayout回调时，就是键盘弹起和下降。
4. possiblyResizeChildOfContent()，computeUsableHeight()计算可见高度。
5. 可见高度小于原始高度一定阀值，则当做为键盘弹起。否则为下降软键盘。
6. 最后回调监听者。

```
public class KeyboardFix implements ViewTreeObserver.OnGlobalLayoutListener {
    /**
     * 父容器
     */
    private View vRootView;
    /**
     * 父容器的高度
     */
    private int mContentHeight;
    /**
     * 标记，第一次回调时保存父容器的高度
     */
    private boolean isFirst = true;
    /**
     * 最后一次显示父容器的高度，用于判断是否显示虚拟键盘
     */
    private int mLastShowHeight;

	//1.首先我们传入rootView构造KeyboardFix。
	public KeyboardFix(View rootView) {
	    if (rootView != null) {
	        //2. 给rootView设置addOnGlobalLayoutListener回调。
	        rootView.getViewTreeObserver().addOnGlobalLayoutListener(this);
	    }
	}
	
	@Override
	public void onGlobalLayout() {
	    //3. addOnGlobalLayoutListener的onGlobalLayout回调时，就是键盘弹起和下降。
	    possiblyResizeChildOfContent();
	}
	
	/**
	 * 重新调整跟布局的高度
	 */
	private void possiblyResizeChildOfContent() {
	    //4. possiblyResizeChildOfContent()，computeUsableHeight()计算可见高度。
	    //计算内容的可见高度
	    int usableHeightNow = computeUsableHeight();
	    if (isFirst) {
	        //兼容华为等机型
	        mContentHeight = usableHeightNow;
	        isFirst = false;
	    }
	    //没有改变，忽略
	    if (usableHeightNow == mLastShowHeight) {
	        return;
	    }
	    mLastShowHeight = usableHeightNow;
	    //5. 可见高度小于原始高度一定阀值，则当做为键盘弹起。否则为下降软键盘。
	    boolean isShowKeyboard = usableHeightNow + 200 < mContentHeight;
	    //6. 最后回调监听者。
	    for (OnKeyboardChangeCallback listener : mOnKeyboardChangeCallbacks) {
	        listener.onChange(isShowKeyboard, mContentHeight, usableHeightNow);
	    }
	}
	
	 /**
     * 计算内容的可见高度
     *
     * @return 父容器的可见高度
     */
    private int computeUsableHeight() {
        Rect rect = new Rect();
        vRootView.getWindowVisibleDisplayFrame(rect);
        return (rect.bottom - rect.top);
    }
    
    /**
     * 虚拟键盘显示隐藏的监听
     */
    public interface OnKeyboardChangeCallback {
        /**
         * 虚拟键盘显示发生变化时调用
         *
         * @param isVisible     true为可见，false为不可见
         * @param contentHeight 原始内容高度
         * @param usableHeight  当前可用高度
         */
        void onChange(boolean isVisible, int contentHeight, int usableHeight);

        /**
         * 界面暂时回调
         */
        void onPause();

        /**
         * 界面销毁时回调
         */
        void onDestroy();
    }
}
```

#### 拓展KeyboardFix，处理输入框弹起

平时我们使用软键盘模式，最多就是将输入框上移，而透明状态栏让键盘模式失效，那么让输入框上移的活，就只能我们自己做了。

原理：

1. 例如我们给输入框设置一个父容器，点击输入框弹起软键盘，这时我们的键盘回调会回调，计算软键盘高度，我们将软键盘高度作为输入框父容器的MarginBottom，就形成了输入框被软键盘弹起的情况。

2. 由于这种情况很通用，所以可以抽取为一个通用的Callback。同时为Callback提供生命周期回调。

```
/**
 * 回调空实现
 */
public static class OnKeyboardChangeAdapter implements OnKeyboardChangeCallback {

    @Override
    public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
    }

    @Override
    public void onPause() {
    }

    @Override
    public void onDestroy() {
    }
}

/**
 * 存在输入框场景的监听，键盘弹起时，设置输入框距离底部一个键盘高度，键盘下降时取消距离
 */
public static class CallbackToInput extends OnKeyboardChangeAdapter {
    /**
     * 输入框容器
     */
    private View vInputContainer;

    public CallbackToInput(View inputContainer) {
        vInputContainer = inputContainer;
    }

    @Override
    public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
        super.onChange(isVisible, contentHeight, usableHeight);
        if (isVisible) {
            showInputView(usableHeight, contentHeight);
        } else {
            setBottomMargin(0);
        }
    }

    @Override
    public void onPause() {
        super.onPause();
        //界面暂停时，下降输入框
        setBottomMargin(0);
    }

    /**
     * 显示虚拟键盘时调用,显示输入框，如果绑定了列表控件则滚动到后一个item
     *
     * @param height 用于计算虚拟键盘的高度
     */
    private void showInputView(int height, int contentHeight) {
        if (vInputContainer == null) {
            return;
        }
        //虚拟键盘的高度
        int keyboardHeight = contentHeight - height;
        setBottomMargin(keyboardHeight);
    }

    /**
     * 重新设置inputContainer的底部边距
     */
    private void setBottomMargin(int bottomMargin) {
        if (vInputContainer == null) {
            return;
        }
        ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) vInputContainer.getLayoutParams();
        if (params.bottomMargin != bottomMargin) {
            params.bottomMargin = bottomMargin;
            vInputContainer.requestLayout();
        }
    }
}
```

3. 使用监听

```
mKeyboardFix.addOnKeyboardChangeListener(new KeyboardFix.CallbackToInput(inputContainer));
```

#### 拓展KeyboardFix，键盘弹起，自动滚动RecyclerView

像QQ、微信，软键盘弹起时，滚动列表控件到底部，由于我们已经有了软键盘回调了，所以处理起来就很简单了，一样我们将这种通用的行为封装为Callback即可。

```
/**
 * 存在列表场景的监听，一般键盘弹起时，列表需要滚动到底部，就可以添加该类型监听
 */
public static class CallbackToList extends OnKeyboardChangeAdapter {
    private RecyclerView vRecyclerView;
    private boolean mIsReverse;

    /**
     * 是否反转，反转则滚动到第0位，非反转则滚动到列表的最后1位
     *
     * @param isReverse true为反转
     */
    public CallbackToList(RecyclerView recyclerView, boolean isReverse) {
        vRecyclerView = recyclerView;
        mIsReverse = isReverse;
    }

    @Override
    public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
        super.onChange(isVisible, contentHeight, usableHeight);
        //键盘弹起时滚动到底部
        if (isVisible) {
            int position;
            if (mIsReverse) {
                position = 0;
            } else {
                if (vRecyclerView.getAdapter() == null) {
                    return;
                }
                position = vRecyclerView.getAdapter().getItemCount() - 1;
            }
            if (vRecyclerView != null && vRecyclerView.getAdapter() != null) {
                vRecyclerView.scrollToPosition(position);
            }
        }
    }
}
```

- 使用监听

```
mKeyboardFix.addOnKeyboardChangeListener(new KeyboardFix.CallbackToList(recyclerView, false));
```

- 分发生命周期事件

最后要记得在Activity或Fragment分发生命周期事件给KeyboardFix。

```
@Override
protected void onPause() {
    super.onPause;
    if (mKeyboardFix != null) {
        mKeyboardFix.onPause();
    }
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (mKeyboardFix != null) {
        mKeyboardFix.onDestroy();
    }
}
```

#### 完整代码

```
public class KeyboardFix implements ViewTreeObserver.OnGlobalLayoutListener {
    /**
     * 父容器
     */
    private View vRootView;
    /**
     * 父容器的高度
     */
    private int mContentHeight;
    /**
     * 标记，第一次回调时保存父容器的高度
     */
    private boolean isFirst = true;
    /**
     * 最后一次显示父容器的高度，用于判断是否显示虚拟键盘
     */
    private int mLastShowHeight;
    /**
     * 键盘监听
     */
    private List<OnKeyboardChangeCallback> mOnKeyboardChangeCallbacks;

    /**
     * 根容器
     */
    public KeyboardFix(View rootView) {
        mOnKeyboardChangeCallbacks = new CopyOnWriteArrayList<>();
        vRootView = rootView;
        if (rootView != null) {
            rootView.getViewTreeObserver().addOnGlobalLayoutListener(this);
        }
    }

    @Override
    public void onGlobalLayout() {
        possiblyResizeChildOfContent();
    }

    public void addOnKeyboardChangeListener(OnKeyboardChangeCallback onKeyboardChangeCallback) {
        mOnKeyboardChangeCallbacks.add(onKeyboardChangeCallback);
    }

    /**
     * 重新调整跟布局的高度
     */
    private void possiblyResizeChildOfContent() {
        //计算内容的可见高度
        int usableHeightNow = computeUsableHeight();
        if (isFirst) {
            //兼容华为等机型
            mContentHeight = usableHeightNow;
            isFirst = false;
        }
        //没有改变，忽略
        if (usableHeightNow == mLastShowHeight) {
            return;
        }
        mLastShowHeight = usableHeightNow;
        //判断是否弹起软键盘
        boolean isShowKeyboard = usableHeightNow + 200 < mContentHeight;
        for (OnKeyboardChangeCallback listener : mOnKeyboardChangeCallbacks) {
            listener.onChange(isShowKeyboard, mContentHeight, usableHeightNow);
        }
    }

    /**
     * 计算内容的可见高度
     *
     * @return 父容器的可见高度
     */
    private int computeUsableHeight() {
        Rect rect = new Rect();
        vRootView.getWindowVisibleDisplayFrame(rect);
        return (rect.bottom - rect.top);
    }

    /**
     * 界面暂时时调用
     */
    public void onPause() {
        if (mOnKeyboardChangeCallbacks != null) {
            for (OnKeyboardChangeCallback callback : mOnKeyboardChangeCallbacks) {
                callback.onPause();
            }
        }
    }

    /**
     * 界面销毁时调用，取消注册，防止内存泄漏
     */
    public void onDestroy() {
        if (vRootView != null) {
            vRootView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
            vRootView = null;
        }
        if (mOnKeyboardChangeCallbacks != null) {
            for (OnKeyboardChangeCallback callback : mOnKeyboardChangeCallbacks) {
                callback.onDestroy();
            }
            mOnKeyboardChangeCallbacks.clear();
            mOnKeyboardChangeCallbacks = null;
        }
    }

    /**
     * 虚拟键盘显示隐藏的监听
     */
    public interface OnKeyboardChangeCallback {
        /**
         * 虚拟键盘显示发生变化时调用
         *
         * @param isVisible     true为可见，false为不可见
         * @param contentHeight 原始内容高度
         * @param usableHeight  当前可用高度
         */
        void onChange(boolean isVisible, int contentHeight, int usableHeight);

        /**
         * 界面暂时回调
         */
        void onPause();

        /**
         * 界面销毁时回调
         */
        void onDestroy();
    }

    /**
     * 回调空实现
     */
    public static class OnKeyboardChangeAdapter implements OnKeyboardChangeCallback {

        @Override
        public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
        }

        @Override
        public void onPause() {
        }

        @Override
        public void onDestroy() {
        }
    }

    /**
     * 显示软键盘
     */
    public void showSoftInput(final View view) {
        InputMethodManager manager = (InputMethodManager) view.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
        if (manager == null) {
            return;
        }
        view.setFocusable(true);
        view.setFocusableInTouchMode(true);
        view.requestFocus();
        manager.showSoftInput(view, InputMethodManager.SHOW_FORCED);
    }

    /**
     * 存在输入框场景的监听，键盘弹起时，设置输入框距离底部一个键盘高度，键盘下降时取消距离
     */
    public static class CallbackToInput extends OnKeyboardChangeAdapter {
        /**
         * 输入框容器
         */
        private View vInputContainer;

        public CallbackToInput(View inputContainer) {
            vInputContainer = inputContainer;
        }

        @Override
        public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
            super.onChange(isVisible, contentHeight, usableHeight);
            if (isVisible) {
                showInputView(usableHeight, contentHeight);
            } else {
                setBottomMargin(0);
            }
        }

        @Override
        public void onPause() {
            super.onPause();
            //界面暂停时，下降输入框
            setBottomMargin(0);
        }

        /**
         * 显示虚拟键盘时调用,显示输入框，如果绑定了列表控件则滚动到后一个item
         *
         * @param height 用于计算虚拟键盘的高度
         */
        private void showInputView(int height, int contentHeight) {
            if (vInputContainer == null) {
                return;
            }
            //虚拟键盘的高度
            int keyboardHeight = contentHeight - height;
            setBottomMargin(keyboardHeight);
        }

        /**
         * 重新设置inputContainer的底部边距
         */
        private void setBottomMargin(int bottomMargin) {
            if (vInputContainer == null) {
                return;
            }
            ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) vInputContainer.getLayoutParams();
            if (params.bottomMargin != bottomMargin) {
                params.bottomMargin = bottomMargin;
                vInputContainer.requestLayout();
            }
        }
    }

    /**
     * 存在列表场景的监听，一般键盘弹起时，列表需要滚动到底部，就可以添加该类型监听
     */
    public static class CallbackToList extends OnKeyboardChangeAdapter {
        private RecyclerView vRecyclerView;
        private boolean mIsReverse;

        /**
         * 是否反转，反转则滚动到第0位，非反转则滚动到列表的最后1位
         *
         * @param isReverse true为反转
         */
        public CallbackToList(RecyclerView recyclerView, boolean isReverse) {
            vRecyclerView = recyclerView;
            mIsReverse = isReverse;
        }

        @Override
        public void onChange(boolean isVisible, int contentHeight, int usableHeight) {
            super.onChange(isVisible, contentHeight, usableHeight);
            //键盘弹起时滚动到底部
            if (isVisible) {
                int position;
                if (mIsReverse) {
                    position = 0;
                } else {
                    if (vRecyclerView.getAdapter() == null) {
                        return;
                    }
                    position = vRecyclerView.getAdapter().getItemCount() - 1;
                }
                if (vRecyclerView != null && vRecyclerView.getAdapter() != null) {
                    vRecyclerView.scrollToPosition(position);
                }
            }
        }
    }
}
```

#### 总结

虽然谷歌没有提供键盘监听，但是我们可以曲线救国，当然希望官方可以修复这个bug，并且提供回调为最好，本篇作为记录而写。