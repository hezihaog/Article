#### Kotlin-那些好用的标准高阶函数

Kotlin特性总结的第二篇，上一篇我们谈了拓展函数和拓展属性。本篇我们来学习一下高阶函数。

拓展函数是对类的拓展，那么高阶函数是什么呢？

```
高阶函数定义：函数的参数是函数，或者返回值是函数的函数，就为高阶函数。
```

本篇讲解到的高阶函数都在Standard.kt文件中。用好这些高阶函数，有利于我们的kotlin代码可读性更强，更简洁。

曰：用Kotlin一时爽，一直用一直爽~

#### 标准高阶函数有哪些？

#### run和T.run

1. run函数和带T的T.run是2种高阶函数。

run函数需要传入的一个block闭包，闭包中的作用域是和括号外层一致的。返回值是block的返回值，所以可以作为一段代码块去处理，并返回一个结果。可以让我们的代码分层，而不像Java是堆在一块。

```
//定义
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

- 实践：

例如我们需要判断用户的类型，弹出不同的操作对话框，就可以将处理包裹在run方法中，并返回一个结果就是dialog。

```
//获取指定id用户的用户信息
val userInfo = LoginService.getUserInfo(userId)
//判断是否是上麦用户
val isSeatUpUser = userInfo.isSeatUpUser()
val dialog = run {
    if(isSeatUpUser) {
    	//上麦用户
    	SeatUpUserOperationDialog()
    } else {
    	//普通用户
    	RoomUserOperationDialog()
    }
}
//省略一些，dialog配置
dialog.show()
```

2. T.run，是对T类型的一个拓展方法。它和上面的run()不一样的地方是作用域不同，同样接收一个block闭包，并且this作用域是拓展的类，即为T，这样的好处是，可以直接获取T类中的属性。

```
//定义
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

- 实践：

我们经常使用的RecyclerView的Adapter，在onBindView中对模型进行取字段设置到条目控件中，如果是java我们对model做各种get操作。kotlin的T.run()拓展可以直接解开模型直接取属性使用，让我们的代码更简洁。

```
override fun onBindViewHolder(holder: ViewHolder, itemModel: FreeConsultModel.ListModel) {
    itemModel.run {
        //头像设置
        holder.vAvatar.loadUrlImageToRound(avatar)
        //昵称设置
        holder.vName.text = nickname
        //消息内容设置
        holder.vMsgContent.text = text
        //消息日期设置
        holder.vDate.text = createTime
    }
}
```

#### T.with

with高阶函数，和上面的T.run()拓展不一样，T.run()是直接将作用域作为调用方T，而with中作用域是由传入的receiver对象作为作用域，同时传入一个闭包，闭包内的this则为receiver。所以也可以直接解出receiver的属性进行使用，返回值为block的返回值。

```
//定义
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

- 实践：

相信我们对RecyclerView的使用已经熟烂如心了，配置LayoutManager、配置Adapter，还有一些滚动监听等，使用with()高阶函数，我们可以对传入的RecyclerView进行配置操作。

```
with(vRecyclerView) {
	layoutManager = LinearLayoutManager(context)
	adapter = RAdapter(List<UserModel>())
}
```

#### T.let

let高阶函数，对T类型进行拓展，接收一个T类型参数的block，返回block的返回值。let和run很相似，但是run中作用域是直接使用this，而let则是将传入参数作为一个为it的参数（默认的，可以手动指定别的名字）

```
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

- 实践：

模型级联非空判断和非空执行闭包，相比java的逐层!=null判断，kotlin优雅了许多。

```
//调用使用ARouter注入的登录服务，跳转到登录页面，由于我们使用组件化开发，可以单独运行，所以LoginService可能会找不到，而为null，所以需要判空，我们就可以使用let，调用前加一个?号，则代表不为空时才进入到闭包中。
mLoginService?.let {
	it.goLogin(activity)
}

//模型解包非空判断
model.data?.list?.let {
	//一路走来，获取到模型中的list属性，我们一般会添加到列表的数据集中再更新列表
	mListItem.addAll(it)
	mListAdapter.notifyDataSetChange()
}
```

#### T.also

also高阶方法，和let很类似，区别是let最后的返回值是block闭包中的返回值，而also则是返回自身。如果需要最后返回值为自身，则可以使用also。

```
//定义
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

- 实践：

例如，我们已有一个文件路径，我们需要创建文件前，先创建好文件夹，则可以先使用let创建File文件，再使用also进行文件夹创建，最后返回一个File文件。

```
//普通函数
fun makeDir(filePath: String): File  {
   val file = File(filePath)
   file.mkdirs()
   return file
}

//also改造
val file = filePath.let {
    File(it)
}.also {
    it.mkdirs()
}
//file处理
```

#### T.apply

apply高阶函数，对T类进行拓展，接收一个T的拓展闭包，和also很像，但是also接收的闭包的T是作为参数传入的，所以使用it来使用，而apply传入的闭包是自身的拓展，所以直接是this就可以访问，可以调用自身属性和方法。

```
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

- 实践：

一般我们列表页面，会有一个标题栏TopBar，一个下拉刷新、下拉加载更多的刷新布局等控件，而我们对他们的相关配置就可以使用apply来包裹。

```
//顶部栏配置
vTopBar.apply {
    setTitle(R.string.mine_main_system_announcement)
    addLeftBackImageButton().click {
        activity?.finish()
    }
}
//下拉刷新、上拉加载布局配置
vRefreshLayout.apply {
    setOnRefreshListener {
        refresh()
    }
    setOnLoadMoreListener {
        loadMore()
    }
}
//列表控件配置
vRefreshList.apply {
    layoutManager = LinearLayoutManager(activity)
    adapter = mListAdapter
}
```

#### 高阶函数那么多，怎么选择？

下面的图是搜索到其他博客中的，总结得很好，这里也参考一下。

![kotlin标准高阶函数图表.png](https://upload-images.jianshu.io/upload_images/1641428-e6747079dc01b71e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 总结

Kotlin提供了众多的高阶方法，合理使用会让我们的代码更整洁、简洁，可读性更强，如果不知道使用哪个高阶方法可以参考上图。