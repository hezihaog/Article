#### UIViewController、Activity生命周期异同

UIViewController和Activity都是视图的容器，都有页面切换的场景，所以自然后页面创建、销毁的过程，生命周期也有一些类似。

#### Activity生命周期回顾

Activity生命周期，一共6个回调方法，可以分为3组：

- onCreate()，页面创建。

- onStart()，显示页面，即将获取焦点。
- onResume()，已获取焦点，可以和用户交互。

- onPause()，页面即将消失，即将失去焦点。
- onStop()，页面已消失，失去焦点。

- onDestroy()，页面销毁。

Activity的生命周期是成对存在和调用的，例如：

1. onCreate()对应onDestroy()。
2. onStart()对应onPause()。
3. onResume()对应onStop()。

#### UIViewController生命周期

UIViewController的生命周期和Activity一样，也有6个方法，也可以分为三组：

- viewDidLoad，页面创建

- viewWillAppear，显示页面，即将获取焦点。
- viewDidAppear，已获取焦点，可以和用户交互。

- viewWillDisappear，页面即将消失，即将失去焦点。
- viewDidDisappear，页面已消失，失去焦点。

- dealloc，页面销毁。

UIViewController生命周期的方法名没有Activity那么明显，不过还是有规律的，例如视图即将显示的回调方法都以Appear结尾，而视图即将消失的回调方法都以Disappear结尾。而“即将”则使用Will单词，“已经”则使用Did。

#### UIViewController之间切换生命周期回调顺序

例如有2个UIViewController，第一个UIViewController为首页，第二页面为详情页。首页点击按钮跳转到详情页，详情页点按钮回到首页。

我在每个生命周期中，打上Log，我们来观察一下生命周期的调用流程。

- AppDelegate.m

```
@interface AppDelegate ()

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:UIScreen.mainScreen.bounds];
    self.window.rootViewController = [[UINavigationController alloc] initWithRootViewController:[[HomeViewController alloc] init]];
    [self.window makeKeyAndVisible];
    return YES;
}

@end
```

- 首页：HomeViewController.m

```
@interface HomeViewController ()

@property(nonatomic, strong) UIButton *nextBtn;

@end

@implementation HomeViewController

/**
 * 添加返回按钮到页面中央
 */
- (UIButton *)nextBtn {
    if (_nextBtn == nil) {
        _nextBtn = [UIButton buttonWithType:UIButtonTypeRoundedRect];
        _nextBtn.frame = CGRectMake(0, 0, 100, 40);
        _nextBtn.center = CGPointMake(self.view.center.x, self.view.center.y);
        [_nextBtn setTitle:@"跳转" forState:UIControlStateNormal];
        [_nextBtn addTarget:self action:@selector(goNext) forControlEvents:(UIControlEventTouchUpInside)];
    }
    return _nextBtn;
}

- (void) goNext {
    [self.navigationController pushViewController:[[DetailViewController alloc] init] animated:YES];
}

- (instancetype)init
{
    self = [super init];
    if (self) {
        self.title = @"首页";
    }
    return self;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    [self.view addSubview:self.nextBtn];
    NSLog(@"Home - viewDidLoad 视图加载完毕 => 类似onCreate()");
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    NSLog(@"Home - viewWillAppear 视图即将显示 => 类似onStrart()");
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    NSLog(@"Home - viewDidAppear 视图已经显示 => 类似onResume()");
    NSLog(@"=============== Detail 获取焦点 ===============");
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    NSLog(@"Home - viewWillDisappear 视图即将消失 => 类似onPause()");
}

- (void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    NSLog(@"Home - viewDidDisappear 视图已经消失 => 类似onStop()");
    NSLog(@"=============== Detail 失去焦点 ===============");
}

- (void)dealloc
{
    NSLog(@"Home - dealloc 视图已经销毁 => 类似onDestroy()");
}

@end
```

- 详情：DetailViewController.m

```
@interface DetailViewController ()

@property(nonatomic, strong) UIButton *backBtn;

@end

@implementation DetailViewController

/**
 * 添加返回按钮到页面中央
 */
- (UIButton *)backBtn {
    if (_backBtn == nil) {
        _backBtn = [UIButton buttonWithType:UIButtonTypeRoundedRect];
        _backBtn.frame = CGRectMake(0, 0, 100, 40);
        _backBtn.center = CGPointMake(self.view.center.x, self.view.center.y);
        [_backBtn setTitle:@"返回" forState:UIControlStateNormal];
        [_backBtn addTarget:self action:@selector(goBack) forControlEvents:(UIControlEventTouchUpInside)];
    }
    return _backBtn;
}

- (void) goBack {
    [self.navigationController popViewControllerAnimated:YES];
}

- (instancetype)init
{
    self = [super init];
    if (self) {
        self.title = @"详情";
    }
    return self;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    [self.view addSubview:self.backBtn];
    NSLog(@"Detail - viewDidLoad 视图加载完毕 => 类似onCreate()");
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    NSLog(@"Detail - viewWillAppear 视图即将显示 => 类似onStrart()");
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    NSLog(@"Detail - viewDidAppear 视图已经显示 => 类似onResume()");
    NSLog(@"=============== Detail 获取焦点 ===============");
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    NSLog(@"Detail - viewWillDisappear 视图即将消失 => 类似onPause()");
}

- (void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    NSLog(@"Detail - viewDidDisappear 视图已经消失 => 类似onStop()");
    NSLog(@"=============== Detail 失去焦点 ===============");
}

- (void)dealloc
{
    NSLog(@"Detail - dealloc 视图已经销毁 => 类似onDestroy()");
}

@end
```

- 首页进入时。和Activity流程一样。首页 onCreate => onStrart() => onResume()

```
Home - viewDidLoad 视图加载完毕 => 类似onCreate()
Home - viewWillAppear 视图即将显示 => 类似onStrart()
Home - viewDidAppear 视图已经显示 => 类似onResume()
=============== Home 获取焦点 ===============
```

- 点击首页按钮，跳转去详情页面。Detail详情页创建后，先通知Home首页即将焦点，再显示Detail开始获取焦点，最后让Home消失，Detail详情页显示。

```
Detail - viewDidLoad 视图加载完毕 => 类似onCreate()
Home - viewWillDisappear 视图即将消失 => 类似onPause()
Detail - viewWillAppear 视图即将显示 => 类似onStrart()
Home - viewDidDisappear 视图已经消失 => 类似onStop()
=============== Home 失去焦点 ===============
Detail - viewDidAppear 视图已经显示 => 类似onResume()
=============== Detail 获取焦点 ===============
```

- 点击详情页返回按钮，回到首页。先通知Detail详情页失去焦点，通知Home主页即将显示，让Detail消失，Home显示，最后因为Detail不会重新使用，而销毁Detail。

```
Detail - viewWillDisappear 视图即将消失 => 类似onPause()
Home - viewWillAppear 视图即将显示 => 类似onStrart()
Detail - viewDidDisappear 视图已经消失 => 类似onStop()
=============== Detail 失去焦点 ===============
Home - viewDidAppear 视图已经显示 => 类似onResume()
=============== Home 获取焦点 ===============
Detail - dealloc 视图已经销毁 => 类似onDestroy()
```

#### UIViewController和Activity生命周期的不同

如果是Android，按任务键或者Home键，会回调栈顶的Activity的生命周期，顺序：onPause() => onStop()。

而UIViewController则没有回调，而是回调AppDelegate，回调applicationDidEnterBackground，当再点击桌面上App的图标，则是回调applicationWillEnterForeground。

AppDelegate类似Android中的Application，App生命周期中只有一个，是一个单例，applicationDidEnterBackground为App进入后台，applicationWillEnterForeground为App进入后台。

这是2个平台设计的差异，iOS的提供了前、后台切换的回调，而Android则没有，需要自己去实现。详情可以看我这篇文章。