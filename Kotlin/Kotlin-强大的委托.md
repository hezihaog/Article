#### Kotlin-强大的委托

委托也叫代理，是一种可以以代理方式控制目标对象的访问，设计模式中成为-代理模式。

Java中，我们实现一个代理模式，会有以下对象；

- Base接口，代理和被代理对象都需要实现的接口。
- BaseImpl类，实现Base接口，是被代理的类。
- Derived类，代理类、委托类，也实现了Base接口，一般以构造方法或set方法注入BaseImpl类。

Java实现

- Base接口，被代理类和代理类都需要实现该接口

```
public interface Base {
	void print();
}
```

- BaseImpl，被代理类

```
public class 	BaseImpl implements Base {
	private String msg;

	public BaseImpl(String msg) {
		this.msg = msg;
	}
	
	@Override
	public void print() {
		System.out.println("msg：" + msg);
	}
}
```

- Derived，代理、委托类

```
public class 	Derived implements Base {
	private Base base;

	//构造方法注入被代理类
	public Derived(Base base) {
		this.base = base;
	}
	
	@Override
	public void print() {
		//复写Base接口中的print()方法，转调被代理类的print()方法
		this.base.print();
	}
}
```

Java中并没有对代理模式做特定的封装和语法糖，而实际代理默认很常用。Kotlin为代理模式提供了语法糖，提供了by关键字来方便实现代理模式，by关键字用在代理类中，格式：by 被代理类实例，Kotlin会默认生成代理类覆写所有的Base接口抽象方法，都转调被代理类，如果需要特殊定义，直接复写目标方法即可。

- Base接口，被代理类和代理类都需要实现该接口

```
interface Base {
    fun print()
}
```

- BaseImpl，被代理类

```
/**
 * 被代理类
 */
class BaseImpl(private val msg: String) : Base {
    override fun print() {
        println("msg：$msg")
    }
}
```

- Derived，代理、委托类

```
/**
 * 委托类，实现类接口，通过by关键字，代理具体实现，编译会实现所有抽象方法，并且转调给实现类，如果需要修改，直接复写即可
 */
class Derived(private val impl: Base) : Base by impl {
    override fun print() {
        impl.print()
    }
}
```

接下来介绍Kotlin的委托，Kotlin的委托分为2种：

1. 类委托，就是上面那种
2. 属性委托，将属性的get、set方法委托给另外一个类

#### 自定义属性委托

自定义属性委托，需要新建一个类，提供getValue()方法和setValue()，并且这个2个方法都是模板，每个代理类都是一样的，只是方法内的处理需要自定义。

```
class MyDelegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        //...
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        //...
    }
}
```

属性委托和类委托一样，也是使用by关键字，但是类委托是类名，by关键字后面跟着实现类实例名，而属性委托中是属性后，by关键字后面跟着代理类的类名。

实例：

新建一个代理类MyDelegate，添加getValue()和setValue()方法，这2个方法，我们简单打印一下thisRef被代理类的引用和property属性对象。Example类为被代理类，给msg属性，通过by关键字指定被MyDelegate代理。main()方法中，我们给Example的实例的msg属性调用set()、get()方法，就会转调到MyDelegate定义的setValue()和getValue()

```
class Example {
    var msg: String by MyDelegate()
}

/**
 * 建立一个委托类，编译器会生成Example的get、set方法转调MyDelegate中的getValue和setValue，再将结果返回
 */
class MyDelegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, 这里委托了 ${property.name} 属性"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$thisRef 的 ${property.name} 属性赋值为 $value")
    }
}

fun main(args: Array<String>) {
    //属性代理
    val example = Example()
    //访问属性的get方法
    println(example.msg)
    //访问属性的set方法
    example.msg = "你好"
}

//输出
com.wally.hellokt.parttwo.Example@1a6c5a9e, 这里委托了 msg 属性
com.wally.hellokt.parttwo.Example@1a6c5a9e 的 msg 属性赋值为 你好
```

属性代理实践：

安卓开发中，Activity、Fragment之间的Intent数据传值，写起来比较繁琐，会写很多getXXExtra()的代码，而借助Kotlin的属性代理，可以更优雅的实现，将变量声明和参数获取合二为一。

我们新建一个Argument类（代理类），我们可以为Activity、Fragment提供拓展方法bindArgument，给属性声明代理给Argument，getValue()和setValue()分别从Activity、Fragment中获取Bundle实例，获取对应的数据即可。

```
/**
 * 拓展Activity获取方法
 */
fun <T> Activity.bindArgument(name: String, default: T): Argument<T> {
    return Argument(name, default) {
        this.intent?.extras ?: Bundle()
    }
}

/**
 * 拓展Fragment获取方法
 */
fun <T> Fragment.bindArgument(name: String, default: T): Argument<T> {
    return Argument(name, default) {
        this.arguments ?: Bundle()
    }
}

/**
 * 属性代理，代理到Bundle上
 * @param name 属性名
 * @param default 默认值
 * @param block 获取Bundle对象的闭包
 */
class Argument<T>(private val name: String, private val default: T, private val block: () -> Bundle) : ReadWriteProperty<Any?, T> {
    override fun getValue(thisRef: Any?, property: KProperty<*>): T = findArgument(name, default)

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) = putArgument(name, value)

    @Suppress("UNCHECKED_CAST")
    private fun <U> findArgument(name: String, default: U): U = with(this.block()) {
        val res: Any = when (default) {
            is Long -> getLong(name, default)
            is String -> getString(name, default)!!
            is Int -> getInt(name, default)
            is Boolean -> getBoolean(name, default)
            is Float -> getFloat(name, default)
            is Serializable -> getSerializable(name) ?: default
            is Parcelable -> getParcelable(name) ?: default
            else -> throw IllegalArgumentException("This type can be saved into Argument")
        }
        res as U
    }

    private fun <U> putArgument(name: String, value: U) = with(this.block()) {
        when (value) {
            is Long -> putLong(name, value)
            is String -> putString(name, value)
            is Int -> putInt(name, value)
            is Boolean -> putBoolean(name, value)
            is Float -> putFloat(name, value)
            is Serializable -> putSerializable(name, value)
            is Parcelable -> putParcelable(name, value)
            else -> throw IllegalArgumentException("This type can be saved into Argument")
        }
    }
}
```

- 代理使用，我们从第一个Activity跳转到第二个Activity，Intent配置了个一个问题Id，key为AskTeacherConstant.Extra.QUESTION_ID，默认值为空字符串，声明的变量为mQuestionId。省去了问题Id在Intent中getStringExtra()的获取，更加简洁。

```
private val mQuestionId: String by bindArgument(AskTeacherConstant.Extra.QUESTION_ID, "")
```

#### 标准委托

委托作为Kotlin的一大特性，提供了一些标准委托给予我们使用。

#### lazy懒加载委托

使用lazy委托，需要我们提供一个Block闭包，Block闭包内写我们变量的初始化逻辑，这个Block只有第一个调用时初始化，后续调用则直接使用第一次的返回值。这样可以很好的节省内存。

- 格式为：

```
val(var) 变量名 by lazy {
	初始化逻辑...
}
```

下面我们使用一个例子来讲解，例如一个Human类，内部有一个friendList变量，它是一个好友列表，加载比较耗时，main()方法中我们访问friendList变量，希望第一次获取时才初始化，第二次以及以后不会重复加载，复用之前的结果。

```
fun main(args: Array<String>) {
    //懒加载委托，生成一个闭包，包裹获取的值的代码，调用被代理的属性的get方法时调用，然后保存值起来，第二次调用则直接返回
    val human = Human()
    println("---------------- 第一次加载，才加载数据 ----------------")
    println(human.friendList)
    println("---------------- 第二次加载，直接输出，不会重复加载 ----------------")
    println(human.friendList)
}

class Human {
    /**
     * 好友列表
     */
    //默认模型为LazyThreadSafetyMode.SYNCHRONIZED同步，如果不需要则可以使用NONE标识不不要保证线程安全
    val friendList by lazy(mode = LazyThreadSafetyMode.NONE) {
        loadFriendList()
    }

    //这里模拟，加载比较耗时
    private fun loadFriendList(): MutableList<String> {
        println("---------------- 加载耗时的好友列表 ----------------")
        return mutableListOf("Wally", "Barry", "Rose")
    }
}

//输出
---------------- 第一次加载，才加载数据 ----------------
---------------- 加载耗时的好友列表 ----------------
[Wally, Barry, Rose]
---------------- 第二次加载，直接输出，会重复加载 ----------------
[Wally, Barry, Rose]
```

lazy关键字后有一个mode字段，默认不写则为同步，所以是线程安全的，如果确保不会涉及多线程的问题，则可以设置为LazyThreadSafetyMode.NONE，提高性能。

#### observable监听属性的改变

Kotlin还提供了一个observable委托，让我们能监听到属性值的改变，类似MVVM中的属性值改变，UI组件监听自动改变。同样需要我们提供一个block闭包来回调我们，提供属性值、旧值和新值给我们。

- 格式为：

```
val(var) 变量名 by Delegates.observable(初始化值) {
	property, oldValue, newValue -> ...
}
```

举一个列子，例如MyPerson类中有一个name属性，初始化值为Wally，我们让它的值改变为Barry，我们的委托Delegates.observable就会回调我们的Block，提供属性值、旧值和新值给我们。

```
fun main(args: Array<String>) {
    //属性观察，可以监听属性的新、旧值，但不能改变
    val person = MyPerson()
    person.name = "Barry"
}

class MyPerson {
    var name: String by Delegates.observable("Wally") { property, oldValue, newValue ->
        println("对象属性值发生改变 -> 属性名：${property.name}，旧值：$oldValue，新值：$newValue")
    }
}

//输出
对象属性值发生改变 -> 属性名：name，旧值：Wally，新值：Barry
```

#### vetoable监听属性改变，并且可以拦截

vetoable和observable类似，同样可以监听到属性值的改变，但是vetoable可以拦截值的改变，而observable只能获取不能拦截。也需要我们提供一个block闭包，返回true为拦截，不允许改变，返回false，允许改变。

- 格式为：

```
val(var) 变量名 by Delegates.observable(初始化值) {
	property, oldValue, newValue -> ...
	
	//满足某种条件，返回true代表拦截，不允许改变
	if(xxx) {
		true
	} else {
		//返回false为不拦截，允许改变
		false
	}
```

举个例子，和上个observable类型，监听Person2类中的name属性，默认值为Wally，我们让它的值改变为Barry，我们的委托Delegates.vetoable就会回调我们提供的block，提供属性值，旧值和新值，要求我们返回一个boolean布尔值，我们判断新值为Barry返回true，不允许改变，其他情况返回false允许改变。

```
fun main(args: Array<String>) {
    //属性改变监听+拦截（保留原来的值）
    val person2 = Person2()
    person2.name = "Barry"
    println("name的值为：${person2.name}")
    person2.name = "Wally2"
    println("name的值为：${person2.name}")
}

class Person2 {
    var name: String by Delegates.vetoable("Wally") { property, oldValue, newValue ->
        println("对象属性值发生改变 -> 属性名：${property.name}，旧值：$oldValue，新值：$newValue")
        //如果设置为Barry不允许设置
        if (newValue == "Barry") {
            println("!!!!!! <不允许设置为Barry> !!!!!!")
            false
        } else {
            true
        }
    }
}

//输出
对象属性值发生改变 -> 属性名：name，旧值：Wally，新值：Barry
!!!!!! <不允许设置为Barry> !!!!!!
name的值为：Wally
对象属性值发生改变 -> 属性名：name，旧值：Wally，新值：Wally2
name的值为：Wally2
```

#### notNull非空代理

如果我们期望某个属性值使用前必须赋值，那么我们可以对属性值做可空类型，但是可空类型会导致我们使用时必须判空，到处的判空代码，代码会显得十分丑陋，可空属性是用在某几种阶段可用时使用，如果我们的属性赋值一次后就可以一直使用，那么可以使用Delegates.notNull委托。对属性使用Delegates.notNull委托，必须使用前赋值一次，否则会抛出异常。

格式为：

```
var 变量名 by Delegates.notNull()
```

举个例子，安卓开发中，一般我们会让Application类作为单例，方便后续直接使用，而App单例的实例在onCreate()中才初始化，那么就需要设置为可空属性，那么App.instant属性就需要每次使用时都判空，而我们知道他不会为空，希望不用判空直接使用，那么我们就可以使用Delegates.notNull委托。

```
class App :Application() {
	companion object {
        var instant: App by Delegates.notNull()
    }
    
    override fun onCreate() {
    	super.onCreate()
    	instant = this;
    }
}

class MainActivity: Activity{
	override fun onCreate() {
    	super.onCreate()
    	val application = App.instant;
    }
}
```

#### 总结

委托在Kotlin是一个很重要的特性，通过委托，我们可以做到很多事情，例如findViewById能委托为一个委托类获取，而可以书写得像声明属性一样。再或者让Sp保存、获取也可以做成变量声明的形式，以及获取Activity、Fragment的Bunlde参数。Kotlin还为我们提供了标准委托，例如lazy懒加载让我们的属性在第一次获取时才输出，但是变量可以书写为变量声明的方式，还有observable、vetoable代理，让我们监听到属性改变，还可以作为MVVM模式下的ViewModel的补充。