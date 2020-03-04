#### Android 7.x Toast BadTokenException处理

7.x版本，对Toast添加了Token验证，这本是对的，但是调用show()显示Toast时，如果有耗时操作卡住了主线程超过5秒，就会抛出BadTokenException的异常，而8.x系统开始，Google则在内部进行了try-catch。而7.x系统则是永久的痛，只能靠我们自己来修复了。

![Api对比.png](https://upload-images.jianshu.io/upload_images/1641428-89c66c9131ac4c8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Toast显示流程

后续补充

#### 修复方案一

反射代理View的Context，Context内进行try-catch，处理Toast的BadTokenException问题

- BadTokenListener，Toast抛出BadTokenException监听器

```
public interface BadTokenListener {
    /**
     * 当Toast抛出BadTokenException时回调
     *
     * @param toast 发生异常的Toast实例
     */
    void onBadTokenCaught(@NonNull Toast toast);
}
```

- SafeToastContext，包裹Toast使用的Context

```
public class SafeToastContext extends ContextWrapper {
    private Toast mToast;
    private BadTokenListener mBadTokenListener;

    public SafeToastContext(Context base, Toast toast) {
        super(base);
        mToast = toast;
    }

    public void setBadTokenListener(@NonNull BadTokenListener badTokenListener) {
        mBadTokenListener = badTokenListener;
    }

    @Override
    public Context getApplicationContext() {
        //代理原本的Context
        return new ApplicationContextWrapper(super.getApplicationContext());
    }

    private class ApplicationContextWrapper extends ContextWrapper {
        public ApplicationContextWrapper(Context base) {
            super(base);
        }

        @Override
        public Object getSystemService(String name) {
            if (Context.WINDOW_SERVICE.equals(name)) {
                //获取原来的WindowManager，交给WindowManagerWrapper代理，捕获BadTokenException异常
                Context baseContext = getBaseContext();
                return new WindowManagerWrapper((WindowManager) baseContext.getSystemService(name));
            }
            return super.getSystemService(name);
        }
    }

    private class WindowManagerWrapper implements WindowManager {
        /**
         * 被包裹的WindowManager实例
         */
        private WindowManager mImpl;

        public WindowManagerWrapper(@NonNull WindowManager readImpl) {
            mImpl = readImpl;
        }

        @Override
        public Display getDefaultDisplay() {
            return mImpl.getDefaultDisplay();
        }

        @Override
        public void removeViewImmediate(View view) {
            mImpl.removeViewImmediate(view);
        }

        @Override
        public void addView(View view, ViewGroup.LayoutParams params) {
            //在addView动刀，捕获BadTokenException异常
            try {
                mImpl.addView(view, params);
            } catch (BadTokenException e) {
                e.printStackTrace();
                if (mBadTokenListener != null) {
                    mBadTokenListener.onBadTokenCaught(mToast);
                }
            }
        }

        @Override
        public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
            mImpl.updateViewLayout(view, params);
        }

        @Override
        public void removeView(View view) {
            mImpl.removeView(view);
        }
    }
}
```

- ToastCompat，替代Toast的门面

```
public class ToastCompat extends Toast {
    private Toast mToast;

    public ToastCompat(Context context, Toast toast) {
        super(context);
        mToast = toast;
    }

    public static ToastCompat makeText(Context context, CharSequence text, int duration) {
        @SuppressLint("ShowToast")
        Toast toast = Toast.makeText(context, text, duration);
        setContextCompat(toast.getView(), context);
        return new ToastCompat(context, toast);
    }

    public static Toast makeText(Context context, @StringRes int resId, int duration)
            throws Resources.NotFoundException {
        return makeText(context, context.getResources().getText(resId), duration);
    }

    public @NonNull
    ToastCompat setBadTokenListener(@NonNull BadTokenListener listener) {
        final Context context = getView().getContext();
        if (context instanceof SafeToastContext) {
            ((SafeToastContext) context).setBadTokenListener(listener);
        }
        return this;
    }

    @Override
    public void show() {
        mToast.show();
    }

    @Override
    public void setDuration(int duration) {
        mToast.setDuration(duration);
    }

    @Override
    public void setGravity(int gravity, int xOffset, int yOffset) {
        mToast.setGravity(gravity, xOffset, yOffset);
    }

    @Override
    public void setMargin(float horizontalMargin, float verticalMargin) {
        mToast.setMargin(horizontalMargin, verticalMargin);
    }

    @Override
    public void setText(int resId) {
        mToast.setText(resId);
    }

    @Override
    public void setText(CharSequence s) {
        mToast.setText(s);
    }

    @Override
    public void setView(View view) {
        mToast.setView(view);
        setContextCompat(view, new SafeToastContext(view.getContext(), this));
    }

    @Override
    public float getHorizontalMargin() {
        return mToast.getHorizontalMargin();
    }

    @Override
    public float getVerticalMargin() {
        return mToast.getVerticalMargin();
    }

    @Override
    public int getDuration() {
        return mToast.getDuration();
    }

    @Override
    public int getGravity() {
        return mToast.getGravity();
    }

    @Override
    public int getXOffset() {
        return mToast.getXOffset();
    }

    @Override
    public int getYOffset() {
        return mToast.getYOffset();
    }

    @Override
    public View getView() {
        return mToast.getView();
    }

    @NonNull
    public Toast getBaseToast() {
        return mToast;
    }

    /**
     * 反射设置View中的Context，Toast会获取View的Context来获取WindowManager
     */
    private static void setContextCompat(@NonNull View view, @NonNull Context context) {
        //7.1.1版本才进行处理
        if (Build.VERSION.SDK_INT == 25) {
            try {
                Field field = View.class.getDeclaredField("mContext");
                field.setAccessible(true);
                field.set(view, context);
            } catch (Throwable e) {
                e.printStackTrace();
            }
        }
    }
}
```

- 使用ToastCompat代替Toast

```
ToastCompat.makeText(context,"我是Toast的内容", Toast.LENGTH_SHORT)
	.setBadTokenListener(toast ->
	        Logger.d("Toast：" + toast + " => 抛出BadTokenException异常")
	)
	.show();
```

#### 修复方案二

对Toast进行Hook，替换Toast中TN对象的Handler，对分发消息dispatchMessage()方法进行try-catch

```
public class SafeToast {
    private static Field sField_TN;
    private static Field sField_TN_Handler;

    static {
        try {
            //反射获取TN对象
            sField_TN = Toast.class.getDeclaredField("mTN");
            sField_TN.setAccessible(true);
            sField_TN_Handler = sField_TN.getType().getDeclaredField("mHandler");
            sField_TN_Handler.setAccessible(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private SafeToast() {
    }

    @SuppressLint("ShowToast")
    public static void show(Context context, CharSequence message, int duration) {
        show(context, message, duration, null);
    }

    @SuppressLint("ShowToast")
    public static void show(Context context, CharSequence message, int duration, BadTokenListener badTokenListener) {
        Toast toast = Toast.makeText(context.getApplicationContext(), message, duration);
        hook(toast, badTokenListener);
        toast.setDuration(duration);
        toast.setText(message);
        toast.show();
    }

    @SuppressLint("ShowToast")
    public static void show(Context context, @StringRes int resId, int duration) {
        show(context, resId, duration, null);
    }

    @SuppressLint("ShowToast")
    public static void show(Context context, @StringRes int resId, int duration, BadTokenListener badTokenListener) {
        Toast toast = Toast.makeText(context.getApplicationContext(), resId, duration);
        hook(toast, badTokenListener);
        toast.setDuration(duration);
        toast.setText(context.getString(resId));
        toast.show();
    }

    private static void hook(Toast toast, BadTokenListener badTokenListener) {
        try {
            Object tn = sField_TN.get(toast);
            Handler originHandler = (Handler) sField_TN_Handler.get(tn);
            sField_TN_Handler.set(tn, new SafeHandler(toast, originHandler, badTokenListener));
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }

    /**
     * 替换Toast原有的Handler
     */
    private static class SafeHandler extends Handler {
        private Toast mToast;
        private Handler mOriginImpl;
        private BadTokenListener mBadTokenListener;

        SafeHandler(Toast toast, Handler originHandler, BadTokenListener badTokenListener) {
            mToast = toast;
            mOriginImpl = originHandler;
            mBadTokenListener = badTokenListener;
        }

        @Override
        public void dispatchMessage(Message msg) {
            //对分发Message的处理方法进行try-catch
            try {
                super.dispatchMessage(msg);
            } catch (WindowManager.BadTokenException e) {
                e.printStackTrace();
                if (mBadTokenListener != null) {
                    mBadTokenListener.onBadTokenCaught(mToast);
                }
            }
        }

        @Override
        public void handleMessage(Message msg) {
            //需要委托给原Handler执行
            mOriginImpl.handleMessage(msg);
        }
    }
}
```

- 使用SafeToast代替Toast

```
SafeToast.show(context,"我是Toast的内容", Toast.LENGTH_SHORT, toast ->
	        Logger.d("Toast：" + toast + " => 抛出BadTokenException异常"));
```