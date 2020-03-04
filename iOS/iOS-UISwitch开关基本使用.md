#### iOS-UISwitch开关基本使用

无论哪种客户端或者网页，开关控件都是必备的，在Android中提供了Switch控件，而iOS则提供了UISwitch。日常开发中，设计师一般都是按照iOS的设计风格来设计，所以安卓原生的Switch基本派不上用场，基本都是自定义View来实现。iOS客户端则可以直接用UISwitch。

![UISwitch默认样式.png](https://upload-images.jianshu.io/upload_images/1641428-bae09ed7ff139a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 创建开关

UISwitch通过alloc和init就可以创建了，注意要显示必须设置按钮的frame，否则不会显示，而且UISwitch的宽、高都不能修改，就算设置了也没效果。

```
@interface ViewController ()

@property(nonatomic, strong) UISwitch *pushSwitch;

@end

@implementation ViewController

/**
 * 懒加载按钮开关
 */
- (UISwitch *)pushSwitch {
    if (_pushSwitch == nil) {
        _pushSwitch = [[UISwitch alloc] init];
        //位置的x,y可以改，但是按钮宽、高不可以改，就算设置了也没效果
        _pushSwitch.frame = CGRectMake(100, 200, 80, 40);
        //设置按钮在屏幕中心
        _pushSwitch.center = CGPointMake(self.view.center.x, self.view.center.y);
    }
    return _pushSwitch;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    //将开关添加到控制器的View
    [self.view addSubview:self.pushSwitch];
}

@end
```

#### 设置样式

默认开关样式：

- 开：绿色背景，白色圆形滑块。
- 关：透明背景，白色圆形滑块。

除了默认样式，苹果爸爸还我我们提供了以下Api设置一些按钮的样式

1. 设置开关-开时的背景颜色

![UISwitch设置开样式.png](https://upload-images.jianshu.io/upload_images/1641428-e5877389bc01822b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

```
//设置开时的背景为橙色，默认绿色
[self.pushSwitch setOnTintColor: [UIColor orangeColor]];
```

2. 设置圆形滑块的颜色

![UISwitch设置滑块颜色.png](https://upload-images.jianshu.io/upload_images/1641428-8a1bfcfe2af85a2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

```
//设置圆形滑块的颜色为绿色，开关的开和关时都为这种颜色，默认为白色
[self.pushSwitch setThumbTintColor: [UIColor greenColor]];
```

3. 设置按钮关闭时的边框颜色

![UISwitch设置关闭时的边框颜色.png](https://upload-images.jianshu.io/upload_images/1641428-783152533b728bfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

```
//按钮关闭时的边框颜色为紫色，只在按钮关闭时边框的颜色，按钮背景为透明，不能被修改
[self.pushSwitch setTintColor:[UIColor purpleColor]];
```

#### 基本使用

- 手动设置按钮的开、关，分为带动画和不带动画2种

```
//设置开关状态，不带动画
self.pushSwitch.on = YES;
//设置开关状态，带动画
[self.pushSwitch setOn:YES animated:YES];
```

- 设置按钮切换事件回调监听

```
//设置开关切换事件
- (void)viewDidLoad {
    [super viewDidLoad];
    //将开关添加到控制器的View
    [self.view addSubview:self.pushSwitch];
    //设置开关切换事件
    [self.pushSwitch addTarget:self action:@selector(switchChange:) forControlEvents:UIControlEventValueChanged];
}

/**
 * 按钮切换事件监听回调方法
 */
- (void) switchChange:(UISwitch*)sw {
    if(sw.on == YES) {
        NSLog(@"开关切换为开");
    } else if(sw.on == NO) {
        NSLog(@"开关切换为关");
    }
}
```

#### 示例

例如设置页面的推送开关，每次开关记录到本地，每次页面进入时回显之前设置的开关状态。本地保存方式有很多种，我们这里选用简单的NSUserDefaults来保存即可。

```
@interface ViewController ()

@property(nonatomic, strong) UISwitch *pushSwitch;

@end

/**
 * 推送开关本地存储标识Key
 */
static NSString* const SWITCH_KEY = @"PUSH_IS_OPEN";

@implementation ViewController

//省略上面提到的按钮创建和初始化...

- (void)viewDidLoad {
    [super viewDidLoad];
    //...省略事件监听设置
    
    //使用NSUserDefaults，回显之前的开关配置
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    //设置开关状态，不带动画
    self.pushSwitch.on = [defaults objectForKey:SWITCH_KEY];
}

- (void) switchChange:(UISwitch*)sw {
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    if(sw.on == YES) {
        NSLog(@"开关切换为开");
    } else if(sw.on == NO) {
        NSLog(@"开关切换为关");
    }
    [defaults setBool:sw.on forKey:SWITCH_KEY];
}

@end
```