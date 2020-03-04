#### Kotlin-你迟到的许多年之拓展函数、拓展变量

Kotlin特性总结的第二篇，上一篇我们讲大了Block，这篇我们来说下Kotlin的另一个特性，拓展方法和拓展变量。如果了解过OC语言的同学，就会想到类目，可以动态给某个类拓展方法和属性，即使是系统提供的类、第三方库中的类。

曰：用Kotlin一时爽，一直用一直爽~

#### 有什么问题

做项目肯定会对系统Api做一层封装，而Java一般是写一个工具类，提供一个静态方法，把自身和需要的参数传进去进行包装。而其他动态语言则可以直接指定类进行添加，例如OC和JavaScript。举个例子：

Kotlin中的拓展方法，就像让一个类自身添加一个方法一样，无需将自身传入，方法体默认this就是被拓展的对象实例，只需要传入需要的参数即可，感觉比工具类优雅多了。

那么如果以前写的工具类，难道就要用Kotlin重写么？不需要的，Kotlin是可以和Java互操作的，可以直接调用Java中的类和方法。

#### Java怎么写？Kotlin怎么写？

Java怎么写就不说了，大家都会哈~

- Kotlin写法

```
//标准方式
fun 要拓展的类的类名.方法名(参数名:参数类型) {
	方法体
}

//如果只有一行，可以简写为以下格式
fun 要拓展的类的类名.方法名(参数名:参数类型) = 方法体
```

- Toast提示

一般Toast提示都会封装为一个工具类，例如ToastUtil。一般分短Toast和长Toast，来对比一下Java和Toast是怎么写的。

```
//Java
public class ToastUtil {
	 private static Toast toast;

    public static void showMsg(Context context, int str) {
        if (context == null) {
            return;
        }
        showMsg(context, context.getResources().getString(str));
    }

    public static void showMsg(Context context, String str) {
        showToast(context, str, Toast.LENGTH_SHORT);
    }

    public static void showMsgLong(Context context, int str) {
        if (context == null) {
            return;
        }
        showMsgLong(context, context.getResources().getString(str));
    }

    public static void showMsgLong(Context context, String str) {
        showToast(context, str, Toast.LENGTH_LONG);
    }

    private static void showToast(Context context, String str, int duration) {
        //...具体Toast，大家都会写，我就不贴了
    }
}

//调用
ToastUtil.showMsg("Hello");
```

```
//kotlin
fun Context.toast(str: Int) {
    ToastUtil.showMsg(this, str)
}

fun Context.toast(str: String) {
    ToastUtil.showMsg(this, str)
}

fun Context.toastLong(str: Int) {
    ToastUtil.showMsgLong(this, str)
}

fun Context.toastLong(str: String) {
    ToastUtil.showMsgLong(this, str)
}

//调用
toast("Hello")
```

上面Kotlin代码中动态给Context类拓展了一个toast、toastLong方法。直接传入字符串或字符串资源Id即可。由于指定给Context，那么Context的所有子类都可以使用，例如Activity。

#### 拓展方法-实践

从上面代码来看，拓展方法很简单，和定义函数的基本区别就是方法名前加多了一个类名。下面来针对一些问题，我们来拓展吧

- TextUtils拓展isNotEmpty()方法

Android提供的TextUtils工具类，我们经常使用它的isEmpty(text)来判断字符串是否为null或空字符串。当我们需要判断字符串不为null不为空字符串时，TextUtils并没有提供isNotEmpty(text)方法来判断字符串是否不为null或不为空字符串。

一般我们就会使用在前面加!，而有时候却可能因为写得太快而忘记，造成错误。最好的方法则是提供一个isNotEmpty()方法，java则需要再新建一个工具类进行包裹转调，而Kotlin则可以直接在原类添加方法。

那么我们先来给TextUtils拓展一个isNotEmpty()方法吧

```
/**
 * 判断字符串是否不为null，不为空字符串
 */
fun TextUtils.isNotEmpty(text: CharSequence): Boolean {
    return !TextUtils.isEmpty(text)
}
```

其实这点Kotlin已经早就想到啦，并且提供了更加强大的拓展方法，在String.kt中就提供了以下方法：

```
//判断是否为null或空字符串
@kotlin.internal.InlineOnly
public inline fun CharSequence?.isNullOrEmpty(): Boolean {
    contract {
        returns(false) implies (this@isNullOrEmpty != null)
    }
    return this == null || this.length == 0
}

//判断是否为空字符串
@kotlin.internal.InlineOnly
public inline fun CharSequence.isEmpty(): Boolean = length == 0

//判断是否不为空字符串
@kotlin.internal.InlineOnly
public inline fun CharSequence.isNotEmpty(): Boolean = length > 0
```

- 防暴击，防重点的点击事件监听器

1. 我们先对点击事件做一层代理，如果2次点击间隔事件小于300毫秒，则当为重复点击，直接过滤，否则响应我们另外定义的抽象方法，这部分Java可以共用，所以直接是Java写的了。

```
public abstract class DelayOnClickListener implements View.OnClickListener {
    /**
     * 默认延时时间
     */
    private static final int DEFAULT_DELAY_TIME = 300;
    /**
     * 上一次的点击时间
     */
    private long mLastClickTime;
    /**
     * 延迟时间
     */
    private int mDelayTime;
	
    public DelayOnClickListener() {
        this(DEFAULT_DELAY_TIME);
    }
	
    public DelayOnClickListener(int delayTime) {
        if (delayTime < 0) {
            return;
        }
        mDelayTime = delayTime;
    }
	
    @Override
    public final void onClick(View view) {
        if (System.currentTimeMillis() - mLastClickTime < mDelayTime) {
            return;
        }
        onDelayClick(view);
        this.mLastClickTime = System.currentTimeMillis();
    }
	
    public abstract void onDelayClick(View view);
}
```

2. 第二步，定义点击事件的拓展方法，listener回调我们就直接使用上一篇我们说的Block去写，定义一个回调。

```
/**
 * 给View设置带有防暴击的监听
 */
fun View.click(listener: (view: View) -> Unit) {
    this.setOnClickListener(object : DelayOnClickListener() {
        override fun onDelayClick(view: View?) {
            listener(view!!)
        }
    })
}
	
//调用，直接Lambda，比匿名内部类简洁多了，如果不需要view参数，可以不写，或者直接使用it，it是默认定义的参数名，不需要我们写。
vShareQq.click {view->
    toast("分享到QQ")
}
```
	
- View显示、隐藏，我们一般直接调用View.setVisibility()，传入View的3个标识位。我们可以更加语义化调用。

```
fun View.setVisible() {
    this.visibility = View.VISIBLE
}

fun View.setGone() {
    this.visibility = View.GONE
}

fun View.setInVisible() {
    this.visibility = View.INVISIBLE
}
```

- 更新View的Margin值。我们想动态设置View的margin值时，发现只有一个setMargins()的方法，需要传入leftMargin、topMargin、rightMargin、bottomMargin，例如我们需要设置leftMargin，后面的3个参数则需要获取自身的topMargin、rightMargin、bottomMargin，回传回去，基本是多余的。由于Kotlin拓展方法中可以直接获取自身的属性和调用方法，所以我们可以自己封装类似setMarginLeft()和setMarginRight()，只需要传入需要更新的具体值即可。

```
/**
 * 更新MarginLeft
 */
fun ViewGroup.MarginLayoutParams.setMarginLeft(newLeft: Int) {
    setMargins(newLeft, topMargin, rightMargin, bottomMargin)
}

/**
 * 更新MarginRight
 */
fun ViewGroup.MarginLayoutParams.setMarginRight(newRight: Int) {
    setMargins(leftMargin, topMargin, newRight, bottomMargin)
}

/**
 * 更新MarginTop
 */
fun ViewGroup.MarginLayoutParams.setMarginTop(newTop: Int) {
    setMargins(leftMargin, newTop, rightMargin, bottomMargin)
}

/**
 * 更新MarginBottom
 */
fun ViewGroup.MarginLayoutParams.setMarginBottom(newBottom: Int) {
    setMargins(leftMargin, topMargin, rightMargin, newBottom)
}
```

- 拓展StringBuilder的delete()和deleteLast()，平时我们使用StringBuilder时，需要想清除已拼接的字符串和删除最后一个字符，发现StringBuilder并没有提供这种具体的方法，而是提供了delete(start, end)方法和deleteCharAt(index)，则需要我们计算位置，这里则可以封装拓展起来。

```
/**
 * 拓展StringBuilder的清空
 */
fun StringBuilder.clear() {
    this.delete(0, this.length - 1)
}

/**
 * 删除StringBuilder的最后一个字符
 */
fun StringBuilder.deleteLast(lastChar: String) {
    if (lastChar == get(this.length - 1).toString()) {
        this.deleteCharAt(this.length - 1)
    }
}
```

- 集合元素逗号分隔，组合为字符串，一般用于接口上传id批量操作。并且分隔符支持自定义（可选参数）。删除最后一个逗号，刚好用得到上面的deleteLast()拓展。

```
/**
 * 集合内容拼接，每个元素之间用指定的分隔符分隔，默认分隔符是英文的逗号
 */
fun <E> MutableList<E>.listToString(separate: String = ","): String {
    val builder = java.lang.StringBuilder()
    for (e in this) {
        builder.append(e.toString())
        builder.append(separate)
    }
    //删除最后一个分隔符
    builder.deleteLast(separate)
    return builder.toString()
}
```

- RecyclerView Adapter的notifyItemRemoved()的坑。使用过notifyItemRemoved()方法的小伙伴肯定知道这个坑，我们想移除一个条目，并且执行动画时，直接调用notifyItemRemoved()后会使移除位置position后的条目调用bindView时数据错乱，原因是谷歌没有重新调用后续条目的onBindView，所以我们可以拓展一个fixNotifyItemRemoved()方法来解决这个问题。

```
/**
 * 修复Rv的notifyItemRemoved，由于notifyItemRemoved后，position位置后的条目无法自动onBindView
 * 所以增加该拓展自动调用notifyItemRangeChanged，在删除后重新绑定position后的条目
 */
fun RecyclerView.Adapter<*>.fixNotifyItemRemoved(position: Int) {
    notifyItemRemoved(position)
    if (position < itemCount) {
        notifyItemRangeChanged(position, itemCount - position)
    }
}
```

#### 拓展属性

说完拓展方法，接下来来说说拓展属性，既然拓展方法可以动态添加方法，那么拓展属性是不是可以动态添加属性呢？使用上是的，但是实际并不会生成属性喔。

#### 拓展属性语法

```
//可变属性，需要get和set方法
var 要拓展的类的类名.拓展属性名:属性数据类型
	get() {
		//获取值
	}
	set() {
		//保存值
	}
	
//不可变属性，只能有get方法
val 要拓展的类的类名.拓展属性名:属性数据类型
	get() {
		//获取值
	}
```

一般在Kotlin中声明属性，会生成一个成员变量字段来保存值，var可变类型会生成get、set方法，而val不可变类型则只生成get方法，拓展属性则不会生成成员变量，所以它不能存储值，所以拓展属性，并不是说动态添加属性，而是借用原有的属性的get方法获取值后做处理再返回，使用上相当于多了一个属性，但是实际是原有属性加上了一个方法处理后返回结果，set方法也一样，将需要设置的值做处理后调用原有属性的set方法进行设置。

所以其实拓展属性也是通过方法来进行的，效果和拓展方法一样，使用拓展属性可以做到的事情，拓展方法也可以做到。

#### 拓展属性实践

- 拓展TextView，增加notNullText属性，设置前将null字符串去掉，再设置

以前使用Java我们会使用工具类，去替换掉null字符串，再返回结果再调用TextView的setText()方法进行设置。

```
//Java
//工具类
publick static class TextUtil {
	public static String removeNullString(String target) {
			if(TextUtil.isEmpty(target)) {
				 return "";
			}
			return target.replace("null", "")
	}
}

//调用
vName.setText(removeNullString(model.getName()));
```

```
//kotlin
/**
 * 给TextView设置text时去掉null字样
 */
var TextView.notNullText: String?
    get() = text.toString()
    set(value) {
        text = value?.replace("null", "") ?: ""
    }
    
//调用
vName.notNullText = model.getName();
```

- ViewGroup添加属性获取第一个子View和最后一个子View

```
/**
 * 获取ViewGroup的第一个子View
 */
val ViewGroup.getFirstChildView: View?
    get() {
        return if (childCount > 0) {
            getChildAt(0)
        } else {
            null
        }
    }

/**
 * 获取最后一个子View
 */
val ViewGroup.getLastChildView: View?
    get() {
        return if (childCount > 0) {
            getChildAt(this.childCount - 1)
        } else {
            null
        }
    }
```

- 获取Context提供的系统服务，例如LayoutInflate布局填充、Vibrator振动器。

Java中，我们一般调用Context的getSystemService()进行获取，但是方法的返回值为Object，所以还需要强转为对应的服务类型，写多了会很冗余，所以一般会加工具类去包装获取，而有了Kotlin，我们可以直接给Context添加属性来获取这些服务。

```
/**
 * 获取布局填充服务
 */
val Context.layoutInflater: LayoutInflater
    get() = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater

/**
 * 获取震动器服务
 */
val Context.vibrator: Vibrator
    get() = getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
```

#### 总结

Kotlin的拓展方法和拓展属性，让我们更加容易的增加类的行为，让我们的代码更优雅、整洁。本来这种特性在动态语言中很常见，Kotlin将它带过来了，终于不用羡慕iOS有OC可以动态拓展了！