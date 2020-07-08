#### 动手撸一个ButterKnife

ButterKnife是Android中，对View绑定的一个类库，本文来动手实现一个基于反射的简单版的ButterKnife，实现View绑定和点击、长按事件绑定。

#### 核心思想

- 自定义注解，反射拿到Activity上的控件变量，取出控件变量的注解的id值，用Activity的findViewById，找该id，找到后，将控件变量设置到设置了注解的对应的变量。

#### 步骤

1. 定义一个ViewInjector注入器接口

```java
public interface ViewInjector {
    /**
     * 以Activity为注入对象
     *
     * @param activity Activity实例
     */
    void inject(Activity activity);
}
```
2. 定义ViewInject注解，提供外部绑定id使用

```java
@Target(ElementType.FIELD)//设置注解使用范围为变量
@Retention(RetentionPolicy.RUNTIME)//设置注解生命时长，运行时
public @interface ViewInject {
    /**
     * View value
     */
    int value();
}
```

3. 一个类实现ViewInjector接口，作为注入器实例，提供一个inject()入口方法，使用时在Activity的onCreate()时调用，传入Activity的实例。

```java
public class ViewInjectorImpl implements ViewInjector {
    private ViewInjectorImpl() {
    }

    private static final class SingletonHolder {
        private static final ViewInjectorImpl instance = new ViewInjectorImpl();
    }

    public static ViewInjectorImpl getInstance() {
        return SingletonHolder.instance;
    }

    @Override
    public void inject(Activity activity) {
        if (activity == null) {
            return;
        }
        //绑定控件
        bindViewId(activity);
    }
}
```

4. 提供bindViewId()方法，开始查找Activity上的变量，取出变量，取出变量上的注解，取出注解上的id，使用Activity的findViewById查找id，找到后，如果不为空，则将该View对象设置回那个使用了注解的变量。(代码上的注释已经注释得很清楚呐)

```java
/**
 * 绑定View的Id
 */
private void bindViewId(Activity activity) {
    try {
        //1.获取所有的成员变量
        Class<? extends Activity> clazz = activity.getClass();
        //2.遍历所有成员变量，找到使用了ViewInject注解的成员变量（所有类型，包括private）
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            //设置允许访问
            field.setAccessible(true);
            ViewInject annotation = field.getAnnotation(ViewInject.class);
            if (annotation != null) {
                //3.将使用了注解的成员变量上标记的id值取出
                int id = annotation.value();
                //4.调用activity的findViewById查找控件
                if (id > 0) {
                    View view = activity.findViewById(id);
                    //5.将控件设置给成员变量
                    field.set(activity, view);
                } else {
                    throw new RuntimeException("ViewInject annotation must have view value");
                }
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

5. 新建ViewInjectManager管理类，提供调用

```java
public class ViewInjectManager {
    /**
     * 获取注入器实现对象
     *
     * @return 注入器实例
     */
    public static ViewInjector getOperate() {
        return ViewInjectorImpl.getInstance();
    }
}
```

6. 使用，使用就很简单啦

```java
@ViewInject(R.id.startBtn)
public Button startBtn;
    
@Override
protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main);
     ViewInjectorImpl.getInstance().inject(this);
     //如果注入成功，Button的文字就会变为“bind success”
     toastBtn.setText("bind success");
}
```

#### 添加OnClick、OnLongClick使用

- 能绑定控件还不够，ButterKnife我们用得最多的就是onClick和OnLongClick。下面我们就开始定义吧。

- 基本思想也是和绑定控件一样，只是注解作用于方法上，这时候反射的就不是变量，而是方法，然后找出使用OnClick、OnLongClick注解的方法，取出注解上的id，找控件，设置onClick、onLongClick，在监听回调时，invoke调用Activity上写的方法。

1. 定义接口

```java
//点击事件注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnClick {
    int[] value();
}

//长按事件注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnLongClick {
    int[] value();
}
```

2. 反射Activity方法，找出方法上使用的注解，设置监听，监听回调时反射调用Activity上使用了注解的方法。(同样，代码上的注释已经解释了步骤，大家应该看得懂的)

```java
/**
 * 绑定View的OnClick事件
 *
 * @param activity
 */
private void bindViewEvent(final Activity activity) {
    //1.获取所有的方法
    Class<? extends Activity> clazz = activity.getClass();
    //2.遍历所有的方法
    Method[] methods = clazz.getDeclaredMethods();
    //3.获取标记了OnClick注解的方法
    for (final Method method : methods) {
        OnClick onClickAnnotation = method.getAnnotation(OnClick.class);
        if (onClickAnnotation != null) {
            //4.取出id，查找View
            int id = onClickAnnotation.value();
            View view = activity.findViewById(id);
            //5.给View绑定onClick，点击时执行
            view.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    try {
                        Class<?>[] parameterTypes = method.getParameterTypes();
                        int paramsCount = parameterTypes.length;
                        if (paramsCount == 0) {
                            method.invoke(activity, new Object[]{});
                        } else {
                            method.invoke(activity, v);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        //长按事件
        OnLongClick onLongAnnotation = method.getAnnotation(OnLongClick.class);
        if (onLongAnnotation != null) {
            int id = onLongAnnotation.value();
            View view = activity.findViewById(id);
            if (view != null) {
                view.setOnLongClickListener(new View.OnLongClickListener() {
                    @Override
                    public boolean onLongClick(View v) {
                        try {
                            Class<?>[] parameterTypes = method.getParameterTypes();
                            int paramsCount = parameterTypes.length;
                            Object o;
                            if (paramsCount == 0) {
                                o = method.invoke(activity, new Object[]{});
                            } else {
                                o = method.invoke(activity, new Object[]{v});
                            }
                            return (boolean) o;
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                        return false;
                    }
                });
            }
        }
    }
}
```

3.  使用，点击事件的方法加上@OnClick()注解，长按事件的方法加上@OnLongClick()注解

```java
@OnClick(R.id.toastBtn)
public void OnClick(View view) {
    switch (view.getId()) {
        case R.id.toastBtn:
            toast("onClick !!!");
            break;
    }
}

@OnLongClick(R.id.toastBtn)
public boolean onLongClick(View view) {
    toast("onLongClick !!!");
    return true;
}

private void toast(String msg) {
    Toast.makeText(getApplicationContext(), msg, Toast.LENGTH_SHORT).show();
}
```

### 缺陷

已经实现了绑定控件、绑定控件点击、长按事件，还有什么可以改进的呢？

- 拿取变量只在传进来的Activity对象，如果控件变量写在父类就找不到了，所以找之前，应该先递归去找一轮的父类，全部绑定一遍。

- 像Activity、Fragment，这些系统提供的类，我们无法去添加注解，并且变量巨多和方法巨多，递归查找他们根本就是没必要的！！所以递归父类的时候，应该去忽略掉这些系统类。

- 反射效率比较低，并且如果反射拿取的Activity上变量很多的时候，遍历的个数就会增加，速度自然会慢。如果可以，可以进阶改为编译时注解和注解解释器，在编译时生成对应的代码，引用时引用生成的代码，自然效率会更高，毕竟少了反射和遍历。

### 结语
- 文章上面只是一个简单的在Activity绑定控件和控件事件，像Fragment上去绑定，其实也是一个道理，只是抽取多几个方法，最后都是调用到同一个绑定方法。