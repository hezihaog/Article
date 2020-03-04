#### Kotlin相见恨晚之Block

新项目中接入了Kotlin，先不说有些坑的地方，先来说说Kotlin的Block特性。
如果了解过OC语言的同学，就会知道Block，Block是什么呢？具体来说就是一段代码块，类似方法，在特定的时候调用。Kotlin中也有Block特性，宏观来讲他们的作用是一样的。

曰：用Kotlin一时爽，一直用一直爽~

#### 有什么问题

- Android开发中，我们会对控件设置点击事件，写动画时，设置开始、结束、变化的回调，Java是用接口来定义，我们一般创建匿名内部类进行设置。

  - 定义回调需要3个步骤
	  1. 定义回调接口
	  2. 提供set方法注入回调对象
	  3. 适当时机调用注入的回调对象的回调方法

- 而我们自定义控件时，有一些状态或者执行分支需要外部来决定时，也会自定义接口，让外部传进来。如果回调很多，定义的接口就越来越多，一般解决方法就是特性类似的作为一组接口回调，但是从Java8开始有函数式接口和Lambda，接口只有一个抽象方法时是可以简写为Lambda的，写多果然很爽，但是分拆接口出来就犯难了，接口会写很多啊！

- 使用Block，以上步骤可以简化为2步（省略接口定义）

#### Java怎么写？Kotlin怎么写？

既然Block能解决接口方法定义的问题，那么方法有的，Block肯定有。

- 方法，无非就几个可以定义的部分。

	- 方法返回值
	- 方法入参

#### Block语法格式

```
//语法格式
块名:(参数:参数类型) -> 返回值类型
```

1. （无入参，无返回值）简单回调

```
//java
public interface Function {
	void call();
}

public void setFunction(Function function) {
	mFunction = function;
}

//调用
setFunction(new Function() {
    @Override
    public void call() {
        System.out.println("hello");
    }
});

mFunction.call();
//输出hello
```

```
//Kotlin
fun test1(block: () -> Unit) {
    block()
}
//调用
test1 {
	println("hello")
}
//输出hello
```

2. （无入参，有返回值）调用返回一个字符串

```
//java
public interface Function {
    String call();
}

public void setFunction(Function function) {
    mFunction = function;
}

//调用
setFunction(new Function() {
    @Override
    public String call() {
        return "hello";
    }
});

String value = mFunction.call();
System.out.println("返回值；" + value);

//输出
返回值：hello
```

```
//kotlin
fun test2(block: () -> String) {
    val result = block()
    println(result)
}

//kotlin不需要写return，最后一行则是返回值，这里相当于Java的return "hello"
test2 {
    "返回值：hello"
}

//输出
返回值：hello
```

3. （有入参，有返回值）传2个数字，返回他们的结果

```
//java
public interface Function {
    int call(int x, int y);
}

public void setFunction(Function function) {
    mFunction = function;
}

//调用
setFunction(new Function() {
    @Override
    public int call(int x, int y) {
        return x + y;
    }
});

mFunction.call(1, 2);
//输出3
```

```
//kotlin
fun test3(block: (x: Int, y: Int) -> Int) {
    val result = block(1, 2)
    println(result)
}

test3 { x, y ->
    x + y
}
//输出3
```

4. （有入参，无返回值）传2个数字，回调处打印参数

```
//java
public interface Function {
    void call(int x, int y);
}

public void setFunction(Function function) {
    mFunction = function;
}

setFunction(new Function() {
    @Override
    public void call(int x, int y) {
        System.out.println("参数1：" + x + "，参数2：" + y);
    }
});

mFunction.call(1, 2);

//输出
参数1：1，参数2：2
```

```
//kotlin
fun test4(block: (x: Int, y: Int) -> Unit) {
    block(1, 2)
}

test4 { x, y ->
    println("参数一：$x，参数二：$y")
}

//输出
参数1：1，参数2：2
```

#### 实践

是不是看完上面的对比还是一脸懵逼，嗯，我也是的。那么就来几个简单的例子实践吧一下吧！

- 案例一：点击事件

例如我们实现一个公告列表，点击一条公告标记一条公告为已读。

1. 我们在顶级构造方法里面添加一个clickItemBlock，作为点击回调（无返回值，有形参的回调）

```
class AnnouncementViewBinder(private val clickItemBlock: (model: SystemAnnouncementModel.ListModel) -> Unit)
    : RItemViewBinder<SystemAnnouncementModel.ListModel, AnnouncementViewBinder.ViewHolder>() {
    ...

    override fun onBindViewHolder(holder: ViewHolder, itemModel: SystemAnnouncementModel.ListModel) {
        itemModel.run {
            ...
            holder.itemView.click {
                clickItemBlock(itemModel)
            }
            ...
        }
    }

    inner class ViewHolder(context: Context, itemView: View) : RViewHolder(context, itemView) {
			...
    }
}
```

- 案例二：子定义View展开、收起，事件回调

```
//定义控件
class ReminderFlowView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0)
    : FrameLayout(context, attrs, defStyleAttr) {
    
    private var mToggleBlock: ((isOpen: Boolean) -> Unit)? = null
    
    //init是kotlin中构造方法时调用的方法，相当于构造方法中调用
    init {
        setup()
    }
    
    private fun setup() {
        //省略查找控件...
        vSwitchAction.click {
            toggleSwitch(!isOpen)
        }
        vDimMask.click {
            toggleSwitch(false)
        }
        vChildContent.click {
            toggleSwitch(!isOpen)
        }
    }
    
    ...
    
    /**
     * 切换开关
     */
    private fun toggleSwitch(isOpen: Boolean) {
        ...
        mToggleBlock?.run {
            this(isOpen)
        }
    }
    
    /**
     * 设置回调
     */
    fun setOnToggleCallback(toggleBlock: (isOpen: Boolean) -> Unit) {
        mToggleBlock = toggleBlock
    }
}

//使用
vReminderFlowView.apply {
	//...省略其他设置
    setOnToggleCallback { isOpen ->
        L.i("切换自定义列表：${if (isOpen) "开" else "关"}")
    }
}
```

#### 总结

Kotlin的Block，能让我们的代码更简洁，定义set方法时就定义了回调的入参和返回值，省略了冗余的接口定义，轻量的定义方式，让我们可以使用Lambda表达式减少内部类的冗余。