#### iOS-UINavigationController基本使用

UINavigationController是iOS提供的栈视图控制器，它必须设置一个RootViewController根控制器，页面跳转时，通过它将下一个子ViewController的视图添加到RootViewController的视图中。

在Android中，可以联想到Activity和Fragment，它们都使用了栈来管理视图，而UINavigationController更加类似于Fragment，因为Activity之间的跳转是2个Window之前的切换，前页面布局和后页面的布局是没有关联的，而Fragment和UINavigationController一样，Fragment和Fragment之间的嵌套，是同一个Window下的布局视图之间的嵌套。

但Fragment的Bug和坑太多了，例如点击穿透，内存重启后重影，Fragment弹栈到根Fragment并不能清除根以上的Fragment实例等。所以一般Fragment不做栈跳转，而是内嵌到Activity和Fragment之间嵌套来使用。

![UINavigationController示例.png](https://upload-images.jianshu.io/upload_images/1641428-c044153214fa6a76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

本篇来介绍一下UINavigationController的3个方面：

1. UINavigationController创建和基本配置
2. UINavigationBar导航栏样式
3. UINavigationController的栈管理API
4. UIToolbar底部工具栏样式

#### UINavigationController创建和基本配置

本篇都是用纯代码方式，不使用Storyboard，所以记得在info.plist文件中删除掉MainStoryboard的设置。

- UINavigationController使用alloc和initWithRootViewController来创建和初始化

```
//创建根视图控制器
ViewController* rootVC = [[ViewController alloc] init];
UINavigationController* navVC = [[UINavigationController alloc] initWithRootViewController:rootVC];
```

- AppDelegate中，创建UINavigationController，将ViewController设置为UINavigationController的根视图控制器，再将UINavigationController设置为Window的根控制器

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:UIScreen.mainScreen.bounds];
    //创建根视图控制器
    ViewController* rootVC = [[ViewController alloc] init];
    //创建UINavigationController，将根视图控制器作为它的根视图
    UINavigationController* navVC = [[UINavigationController alloc] initWithRootViewController:rootVC];
    //设置window的根视图控制器为UINavigationController
    self.window.rootViewController = navVC;
    //显示Window
    [self.window makeKeyAndVisible];
    return YES;
}
```

#### UINavigationBar导航栏样式

经过上面的创建和配置，我们的ViewController视图控制器已经被UINavigationController控制了，我们会发现ViewController的视图上有一个导航栏，它就是UINavigationBar。

UINavigationBar是由UINavigationController管理的，但是它的样式由子控制器的self. navigationItem来设置。下面来说一下它的样式配置和按钮的添加。

- 设置导航栏标题，默认取视图控制器的self.title属性，如果没有设置则取self.navigationItem.title属性。

```
//设置导航栏标题
self.title = @"根视图";
//设置导航元素项目的标题，如果没有设置self.navigationItem.title，系统会使用self.title作为导航栏的标题
//self.navigationItem.title = @"我也是标题";
```

- 导航栏透明和不透明设置

UINavigationBar导航栏默认是透明的，控制器视图从设备的左上角(0,0)点开始计算，而UINavigationBar覆盖在控制器视图以上，我们可以设置它为不透明，不透明时，控制器视图从导航栏之下开始计算。

```
//设置导航栏是否透明，默认为YES: 透明，NO则为不透明
//如果不设置该属性，导航栏和视图控制器的View会重合，形成半透明
self.navigationController.navigationBar.translucent = NO;
```

- 导航栏风格设置，默认导航栏使用白底黑字的默认风格，还可以设置为黑底白字的黑色风格。设置方法有2种，第一种会将所有的UINavigationController都统一设置，第二种则只在调用的UINavigationController上设置。

```
//设置导航栏风格
//UIBarStyleDefault: 默认风格
//UIBarStyleBlack：黑色风格

//第一种，将所有的UINavigationController都统一设置（UINavigationController可以有多个）
[[UINavigationBar appearance] setBarStyle:UIBarStyleBlack];

//第二种，只在调用的UINavigationController上设置
self.navigationController.navigationBar.barStyle = UIBarStyleBlack;
```

- 设置导航栏的颜色，设置该项会将状态栏设置为不透明，例如设置为红色

```
//设置导航栏颜色，该属性会将上面的透明属性和BarStyle进行覆盖
self.navigationController.navigationBar.barTintColor = [UIColor redColor];
```

- 设置导航栏上的元素（例如按钮、文字的风格，例如文字的颜色会因风格颜色而改变字体颜色）

```
//设置导航栏上的元素颜色风格，例如影响导航栏上的文字颜色
self.navigationController.navigationBar.tintColor = [UIColor orangeColor];
```

- 隐藏UINavigationBar导航栏，有2种方式，第一种是UIView上的属性，第二种是UINavigationController的属性。

```
//隐藏导航栏，设置为YES则隐藏掉导航栏，这个属性是UIView上面的属性
self.navigationController.navigationBar.hidden = YES;
//或者导航控制器上的属性也可以隐藏
self.navigationController.navigationBarHidden = YES;
```

#### UINavigationBar导航栏添加按钮

导航栏除了标题外，还可以摆放元素，常见就是添加操作按钮和返回按钮，也可以放置自定义的元素。

导航栏上的按钮需要使用UIBarButtonItem，来包裹我们自定义内容或者自定义视图。

按钮风格可以分为3种，文字按钮、系统图片按钮和自定义视图，文字按钮通过initWithTitle来设置，系统图片按钮则是通过initWithBarButtonSystemItem，最后自定义视图使用initWithCustomView。

- 以文字风格，设置左侧按钮，默认左侧按钮是一个返回键，我们可以覆盖它，并设置按钮事件，例如点击事件。

```
//创建一个导航栏左侧按钮
//参数一：按钮标题
//参数二：按钮风格
//参数三：事件拥有者
//参数四：按钮事件
UIBarButtonItem* leftBtn = [[UIBarButtonItem alloc] initWithTitle:@"编辑" style:UIBarButtonItemStyleDone target:self action:@selector(pressLeft:)];
//设置到导航栏上
self.navigationItem.leftBarButtonItem = leftBtn;

/**
 * 左侧按钮点击事件回调方法
 */
- (void) pressLeft:(UIBarButtonItem*)btn {
    NSLog(@"左侧按钮被按下");
}
```

- 以系统图片风格，设置右侧按钮。

```
//创建一个导航栏右侧按钮，使用系统风格来创建，无需标题文字，因为它不能被改变
UIBarButtonItem* rightBtn = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemAdd target:self action:@selector(pressRight:)];
//设置到导航栏上
self.navigationItem.rightBarButtonItem = rightBtn;

/**
 * 右侧按钮点击事件回调方法
 */
- (void) pressRight:(UIBarButtonItem*)btn {
    NSLog(@"右侧按钮被按下");
}
```

- 以自定义View方式，设置右侧按钮，我们需要使用UIBarButtonItem的initWithCustomView方法将自定义View包裹成UIBarButtonItem来设置。

```
UILabel* label = [[UILabel alloc] initWithFrame:CGRectMake(10, 10, 40, 40)];
label.text = @"测试";
label.textColor = [UIColor grayColor];
label.textAlignment = NSTextAlignmentCenter;
//将Label包装为UIBarButtonItem
UIBarButtonItem* labelItem = [[UIBarButtonItem alloc] initWithCustomView:label];
```

- 还支持设置多个按钮，例如我们将上面的自定义按钮和系统风格按钮放到一个数组，再一起设置到右侧。

```
//省略上面系统图片风格按钮和自定义View按钮的创建和初始化...

//将多个按钮放置到NSArray数组中，再添加到导航栏的位置
NSArray* arrays = [NSArray arrayWithObjects: rightBtn, labelItem, nil];
//设置多个按钮到导航栏上
self.navigationItem.rightBarButtonItems = arrays;
```

- 左侧返回按钮定制

默认子控制器的导航栏左侧的是返回键按钮，如果需要改变它，有2种方式，第一种就是上面说到的使用self.navigationItem.leftBarButtonItem来设置，第二种是设置self.navigationItem.backBarButtonItem属性。leftBarButtonItem的优先级大于backBarButtonItem。

1. 第二种，在跳转前的控制器中设置self.navigationItem.backBarButtonItem属性，如果设置了，下一个控制器页面的返回键则使用backBarButtonItem属性设置的。

```
//设置下个页面的返回按钮
self.navigationItem.backBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"首页" style:(UIBarButtonItemStyleDone) target:nil action:@selector(backToMy)];

/**
 * 返回当前控制器
 */
- (void) backToMy {
    [self.navigationController popToViewController:self animated:YES];
}
```

#### UINavigationController的栈管理API

除了上面的设置按钮，UINavigationController最重要的还是栈容器管理视图控制器，下面来了解一下栈方面的Api。

- 入栈下一个视图控制器

首先先创建下一个控制器SecondViewController的实例，再通过控制器上的self.navigationController，调用pushViewController方法，将SecondViewController推入栈中，第二个animated参数代表是否需要跳转动画。

```
//创建下一个页面的ViewController
SecondViewController* nextVC = [[SecondViewController alloc] init];
//跳转到下一个页面
[self.navigationController pushViewController:nextVC animated:YES];
```

- 入栈多少视图控制器，传入一个控制器数组，跳转后会显示最后一个控制器

```
//创建2个视图控制器
OneViewController *oneVC = [[OneViewController alloc] init];
TwoViewController *twoVC = [[TwoViewController alloc] init];
//将2个视图控制器放到数组中
NSArray *vcArray = [[NSArray alloc] initWithObjects:oneVC,twoVC, nil];
[self.navigationController setViewControllers:vcArray animated:YES];
```

- 出栈视图控制器自身

```
[self.navigationController popViewControllerAnimated:YES];
```

- 出栈到指定控制器，会将指定控制器以上的控制器都进行出栈

```
[self.navigationController popToViewController:targetVC animated:YES];
```

#### UIToolbar底部工具栏样式

UINavigationController还附带一个工具栏，默认是隐藏的，而且也比较少用

- 显示工具栏

```
//是否隐藏工具栏，默认为YES，默认隐藏
self.navigationController.toolbarHidden = NO;
```

- 设置工具栏透明和不透明

```
//设置工具栏透明度为透明
self.navigationController.toolbar.translucent = NO;
```

- 配置工具栏按钮，按钮也是使用UIBarButtonItem，和导航栏按钮的创建方式一样

```
//创建2个工具栏按钮
UIBarButtonItem* btn01 = [[UIBarButtonItem alloc] initWithTitle:@"点赞" style:UIBarButtonItemStyleDone target:nil action:nil];
UIBarButtonItem* btn02 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:nil action:nil];
//将按钮放到数组里
NSArray* toolBtns = [NSArray arrayWithObjects:btn01, btn02, nil];
//添加到工具栏
self.toolbarItems = toolBtns;
```

- 平分按钮，上面添加2个按钮是不会平分工具栏控件的，而是都靠左排列，如果想让按钮平分工具栏控件控件，则需要使用UIBarButtonSystemItemFlexibleSpace的风格创建UIBarButtonItem，按位置添加在数组中即可，不用创建多个按钮实例，一个即可。

```
//创建2个工具栏按钮
UIBarButtonItem* btn01 = [[UIBarButtonItem alloc] initWithTitle:@"点赞" style:UIBarButtonItemStyleDone target:nil action:nil];
UIBarButtonItem* btn02 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:nil action:nil];

//-------------- 自动平分按钮（重点在这里！！！） --------------
UIBarButtonItem* btnSpace = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace target:nil action:nil];

//将按钮放到数组里
NSArray* toolBtns = [NSArray arrayWithObjects:btnSpace, btn01, btnSpace, btn02, btnSpace, nil];
//添加到工具栏
self.toolbarItems = toolBtns;
```