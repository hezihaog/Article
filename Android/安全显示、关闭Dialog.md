#### 安全显示、关闭Dialog

Android中显示、关闭Dialog，正常调用show()、dismiss()有可能会引发一些异常：

1. Activity已经销毁时，调用上述方法就会抛出空指针或Activity的Window被销毁的异常。
2. 在非UI线程，调用show()会显示不出来。

#### 解决方案

针对以上问题，我创建了一个单例类SafeDialogHandle。

1. 提供safeShow()和safeDismiss()，内部会判断Activity是否销毁，销毁了则不进行操作。
2. 进行操作前，判断当前线程，如果是非UI线程调用，则将操作推到主线程的Handler上进行执行。

safeShow()和safeDismiss()，都是Dialog的拓展方法，所以直接替换Dialog的show()、dismiss()为safeShow()和safeDismiss()即可。

```
object SafeDialogHandle {
    /**
     * 主线程Handler
     */
    private val mMainHandler by lazy {
        Handler(Looper.getMainLooper())
    }

    /**
     * 安全显示Dialog
     */
    fun Dialog.safeShow() {
        safeOperation {
            if (this.isShowing) {
                return@safeOperation
            }
            if (getActivity().isDestroy()) {
                return@safeOperation
            }
            show()
        }
    }

    /**
     * 安全关闭Dialog
     */
    fun Dialog.safeDismiss() {
        safeOperation {
            if (!isShowing) {
                return@safeOperation
            }
            if (getActivity().isDestroy()) {
                dismiss()
            }
        }
    }

    /**
     * 安全操作Dialog
     * @param block 要执行的操作
     */
    private fun Dialog.safeOperation(block: Dialog.() -> Unit) {
        //如果不是主线程操作，使用Handler推到主线程执行
        if (Looper.myLooper() != Looper.getMainLooper()) {
            mMainHandler.post {
                block(this)
            }
        } else {
            block(this)
        }
    }

    private fun Dialog.getActivity(): Activity? {
        return getDialogActivity(this.context)
    }

    /**
     * 获取Dialog的Activity
     */
    private tailrec fun getDialogActivity(context: Context): Activity? {
        return when (context) {
            is Activity -> context
            is ContextThemeWrapper -> getDialogActivity(context.baseContext)
            else -> null
        }
    }

    /**
     * 判断Activity是否已经销毁，不再可用
     */
    private fun Activity?.isDestroy(): Boolean {
        return this == null || isFinishing || isDestroyed
    }
}
```

#### 方案使用

```
//升级提示弹窗
val dialog = UpgradeTipDialog(activity)
	..//省略其他配置
//显示弹窗
dialog safeShow()
//关闭弹窗
dialog.safeDismiss()
```