#### Java中的注解

注解是JDK1.5新增的特性，注解是附加在代码上一些信息，在编译时或运行时被解析，一般起到配置的作用。在Android中很多著名的框架大量使用注解，例如ButterKnife、EventBus、Retofit等。本篇来学习一下注解。

#### 元注解

学习注解前，先来了解一下元注解，自定义注解都会使用到元注解。元注解指的就是注解上使用的注解，有以下4种：

1. 	@Retention，定义注解的保留策略
2. @Target，标识注解的作用目标
3. @Document，表明该注解会被包含在javadoc中
4. @Inherited，表明该注解可以被子类继承

#### 注解例子

我们就举一个ButterKnife反射绑定View实例到变量的例子

```
//指明该注解会被包含在javadoc中
@Documented
//运行时可保留
@Retention(RetentionPolicy.RUNTIME)
//只能注解在变量上
@Target(ElementType.FIELD)
public @interface BindView {
    //View的Id
    int value();
}
```

#### @Retention注解

@Retention注解，用来定义注解的保留策略，只能设置单个值。它有以下值可选：

1. @Retention(RetentionPolicy.SOURCE)，注解只存在于源码中，字节码中不包含
2. @Retention(RetentionPolicy.CLASS)，默认为RetentionPolicy.CLASS，注解在class中包含，但是运行时不包含
3. @Retention(RetentionPolicy.RUNTIME)，注解在class中包含，并且运行时也存在，用反射即可获取到

#### @Target注解

@Target注解，定义注解的作用目标，它是可以设置多个值。它有以下值可选：

1. @Target(ElementType.TYPE)，接口、类、枚举
2. @Target(ElementType.FIELD)，字段、枚举的常量
3. @Target(ElementType.METHOD)，方法
4. @Target(ElementType.PARAMETER)，方法参数
5. @Target(ElementType.CONSTRUCTOR)，构造函数
6. @Target(ElementType.LOCAL_VARIABLE)，局部变量
7. @Target(ElementType.ANNOTATION_TYPE)，注解
8. @Target(ElementType.PACKAGE)，包

#### @Inherited注解

例如A类上使用了一个注解叫MyAnno，这个注解被@Inherited注解了，那么B类继承A，则会被这个MyAnno注解进行注解。

#### 注解自定义

- View实例绑定到变量上

```
//指明该注解会被包含在javadoc中
@Documented
//运行时可保留
@Retention(RetentionPolicy.RUNTIME)
//只能注解在变量上
@Target(ElementType.FIELD)
public @interface BindView {
    //View的Id
    int value();
}
```

- 点击事件方法绑定

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnClick {
    int[] value();
}
```

- 长按事件方法绑定

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnLongClick {
    int[] value();
}
```

#### 注解结合反射，绑定View和事件

注解只是配置了信息，一般会配合反射获取后，做对应的处理，例如我们要实现上面说到的：

1. View实例绑定变量，在Activity中调用bindViewId方法，传入Activity自身即可。


```
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
            ViewInject annotation = field.getAnnotation(BindView.class);
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

2. View点击、长按事件绑定到方法。在Activity中调用bindViewEvent方法，传入Activity自身即可。

```
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

#### 总结

本篇学习了注解的自定义，注解配合反射或编译时注解解释器，除了View绑定外，还能做很多有趣的事情

- Android的6.0权限请求
- 方法耗时上报
- 数据库增删改的前后事务处理
- 方法抛异常后重试