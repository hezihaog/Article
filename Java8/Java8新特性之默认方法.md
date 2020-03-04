#### Java8新特性之默认方法和静态接口方法

在Java8以前，接口中的抽象方法只能在实现类中进行实现，这就导致接口的每个实现类都要实现一套，这明显是不合理的，所以常规的方式是，再写一个抽象类实现接口，做一些方法的默认实现，子类再继承这个抽象类，对需要的方法进行复写。这时候就出现了一个问题，在Java语言中继承是只能单继承的，所以继承是非常宝贵的，每个类只能继承一个父类，而我们应该多利用接口实现而不是继承。

曾经何时，思考过，如果接口可以默认实现该多好呀，但是这样就是一个抽象类了，存在抽象方法和非抽象方法，但是抽象类是类，只能继承！

在Java8中，就提供了默认方法这个新特性，默认方法使用**default**关键字来修饰。例如Java8中Iterable接口的forEach，就是一个默认方法:

```
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

#### 日常使用

- MVP架构中，3层一般都会抽象接口，例如V层，View层提供IView接口，里面有toast方法，一般我们每个接口或每个界面都会建立对应的View层接口来继承IView接口，那么toast方法就会每个实现都需要重写。这时候我们就会想，那建立一个BaseView类来给每个View接口的实现类去继承，再实现View接口不就好了？同学，View层一般为Activity或Fragment去实现，继承的位置早已被Activity或Fragment占用了！这时候就很尴尬了...

- 其实解决方案还是有的，就是新建一个Activity或Fragment的基类，实现IView接口，统一实现，子类继承，后续对应的View层接口再实现。但总觉得不优雅，应该多用实现少用继承。现在还得强制继承，万一原本继承的是SDK中的类，不可修改，又只能再新建一个基类继承SDK中的基类，再给项目中进行继承...硬是要搞个类做间接，蛋疼

```
public interface IView {
    //...省略其他方法

    void toast(String msg);
}

public abstract class BaseActivity extends SDKActivity implements IView {
	//...
	
	@Override
    public void toast() {
    	//...
    }
}
```

#### 使用接口默认方法改善

既然接口默认方法可以提供实现，那么IView中的toast就可以内部统一实现了。如果子类有其他逻辑，和以前一样，直接复写方法进行复写即可。

```
public interface IView {
    //...省略其他方法

    default void toast(Context context, String msg) {
        ToastUtil.showMsg(context, msg);
    }
}

public class MyActivity extends SDKActivity implements IView {
	@Override
    public void toast(Context context, String msg) {
        //...依然可以复写默认方法中的实现
    }
}
```

#### 继承含有默认方法的接口

如果一个接口，继承一个含有默认方法的接口，一般有3种情况

1. 直接继承，什么都不用管，就是直接继承了默认方法。

```
public interface SearchView extends IView {
    
}
```

2. 将默认方法重新声明为抽象方法，那就和以前一样了，就是将继承的默认方法抛弃，让子类继承

```
public interface SearchView extends IView {
    @Override
    void toast(Context context, String msg);
}

public class MyActivity extends SDKActivity implements SearchView {
    @Override
    public void toast(Context context, String msg) {
        //...强制需要实现toast方法
    }
}
```

3. 在子类接口中，重新提供默认方法的实现。就是复写父接口上的默认方法，一样是一个默认方法。实现类也无需强制实现。

```
public interface SearchView extends IView {
    @Override
    default void toast(Context context, String msg) {
        //...重新提供默认实现
    }
}

public class MyActivity extends SDKActivity implements IView {
	//...没有强制要求要实现toast方法了
}
```

#### 默认方法的冲突

当实现的多个接口中，有相同的默认方法(方法名、参数、返回值相同)，这时候实现类是必须复写这个方法！

- 实现类自己复写方法后实现自己的逻辑。（2个爸爸的财产都不要，我要自己闯天涯!）
- 调用super关键字去调用指定接口的默认方法。（太穷了，选个有钱的爸爸...）

```
public interface IView {
	default void toast(Context context, String msg) {
        ToastUtil.showMsg(context, msg);
    }
}

public interface FakeView {
    default void toast(Context context, String msg) {
        ToastUtil.showMsg(context, "fake: " + msg);
    }
}

public class MyActivity extends SDKActivity implements IView, FakeView {

    @Override
    public void toast(Context context, String msg) {
    	 //指定选择某个接口的默认实现
        IView.super.toast(Context, msg);
        //或者自己去实现
        ...
    }
}
```

#### 实现了多个接口，接口是继承另外一个接口

例如SearchView接口是IView的接口，IView实现SearchView和IView，那么调用toast会选择哪个实现呢？

- 经过测试，调用toast方法，调用的是子类SearchView的实现，就是说会优先选择子类中复写的方法。

```
public interface IView {
    default void toast(Context context, String msg) {
        ToastUtil.showMsg(context, msg);
    }
}

public interface SearchView extends IView {
    @Override
    default void toast(Context context, String msg) {
        //...重新提供默认实现
        ToastUtil.showMsg("search msg: " + msg);
    }
}

public class MyActivity extends SDKActivity implements IView, SearchView {
}
```

#### 如果实现2个接口都是继承同一个父接口，并且都重写了默认方法呢？

例如上面的，再加一个SearchView实现IView，再加一个HomeView，也实现了IView，MyActivity实现这2个接口会怎么样呢？

- 测试后，发现其实和上面IView和FackView的情况是一样的，强制子类复写toast方法，可以使用super关键字指定某个接口的默认实现。或者自己去实现。

```
public class MyActivity extends SDKActivity implements SearchView, HomeView {
@Override
    public void toast(Context context, String msg) {
		  //指定选择某个接口的默认实现
        SearchView.super.toast(context, msg);
        //或者自己去实现
        ...
    }
}
```

#### 如果接口中的默认方法和继承的父类方法一致

如果接口中提供一个默认方法，在实现类继承的父类中，也同样有一个，那么会选择哪个实现呢？(难道可以接口中的覆盖实现类中的？实现替换实现方法的骚操作？)

测试发现，其他会选择继承父类中的实现，而不是接口中提供的默认方法实现。这个也叫做类优先原则，它可以保证与Java7的兼容性。也就是说，在接口中实现的一个默认方法，它不会对Java8之前写的代码产生影响。所以在接口中写Object中的toString()、equals()，这些是不允许的，IDE会直接报错。

```
public class MyActivity extends SDKActivity implements IView {
	//...
}

public class SDKActivity {
    public void goHome() {
        System.out.println("跳转到Home");
    }
}

public interface IView {
    default void goHome() {
        System.out.println("IView 跳转到Home");
    }
}

//调用父类SDKActivity中的goHome()方法，而不是调用IView接口上的默认方法
activity.goHome();
```

#### 静态接口方法

Java8中，接口除了能写默认方法外，还可以写静态方法，Java8以前是不能写静态方法的，接口中只能写抽象方法。

静态方法没有继承一说，所以调用需要用类名.静态方法名

```
public interface IView {
    default void toast(Context context, String msg) {
        ToastUtil.showMsg(context, msg);
    }

    static void handleErrorCode(int code) {
        //静态方法...
    }
}

public class MyActivity extends SDKActivity implements SearchView {

    @Override
    public void toast(String msg) {
    	  //静态方法没有继承一说，所以调用需要用类名.静态方法名
        IView.handleErrorCode(-1);
    }
}
```