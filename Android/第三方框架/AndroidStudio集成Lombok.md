# AndroidStudio集成Lombok

Lombok是一个编译时帮助我们生成类的getter、setter、toString等的第三方库，让我们不需要给实体类手动写getter、setter，代码更加干净。

## 步骤

### AndroidStudio安装Lombok插件

File => Settings => Plugins，插件市场搜索Lombok，安装，重启工具

### 开启工具的编译注解开关 AnnotationProcessors

File => Settings => 搜索Annotation Processors => 把Enable annotation processing

### 添加Lombok依赖

模块的build.gradle中，添加以下依赖

```
//1、lombok依赖
implementation 'javax.annotation:javax.annotation-api:1.2'
//依赖lombok的API，内部虽然已经声明了注解解释器，但在AndroidStudio上有Bug（导致编译报错找不到生成的方法），需要自己主动再声明一次
implementation 'org.projectlombok:lombok:1.16.6'
//主动声明注解解释器
annotationProcessor 'org.projectlombok:lombok:1.16.6'
```

### 添加Lombok配置文件

在项目根目录，添加`lombok.config`文件，填入以下内容

```
lombok.anyConstructor.suppressConstructorProperties=true
config.stopBubbling=true
lombok.equalsAndHashCode.callSuper=call
```

### 配置注解处理器

模块的build.gradle文件，找到`defaultConfig`，加入以下配置，同步一下即可

```
defaultConfig {
    //3.配置注解处理器
    javaCompileOptions {
        annotationProcessorOptions {
            includeCompileClasspath = true
        }
    }
}
```

## 试验

### 准备实体类

- @Data：生成get、set方法
- @NoArgsConstructor：生成空参构造方法
- @AllArgsConstructor：生成全参构造方法

其他用法，自行搜索哈

```
//生成get、set方法
@Data
//生成空参构造方法
@NoArgsConstructor
//生成全参构造方法
@AllArgsConstructor
public class User {
    private String userName;
    private String password;
}
```

### 使用

布局添加2个按钮，布局就不贴了。
点击时，调用set、get方法，toString等方法，能Toast输出即可

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.btn);
        Button button1 = findViewById(R.id.btn2);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                User user = new User();
                //实验set方法
                user.setUserName("admin");
                user.setPassword("admin");
                //实验get方法
                String userName = user.getUserName();
                String password = user.getPassword();
                Toast.makeText(getApplicationContext(),
                        "用户名：" + userName + " 密码：" + password,
                        Toast.LENGTH_SHORT).show();
            }
        });
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                User user = new User("root", "root");
                //试验toString方法
                String toString = user.toString();
                int hashCode = user.hashCode();
                Toast.makeText(getApplicationContext(),
                        "toString：" + toString + " hashCode：" + hashCode,
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

## 总结

在AndroidStudio中使用还是比较简单的，如果项目还没有用Kotlin，还是用Java，使用Lombok简化代码还是不错的。

代码我上传到了[GitHub](https://github.com/hezihaog/LombokSample)，有需要的同学，可以clone