#### iOS-UIProgress进度条、UISlider拖动条

Android中，进度条为ProgressBar、拖动条控件为SeekBar。而iOS则分别为UIProgress和UISlider。进度条和拖动条在音乐播放模块中经常出现。

#### 创建UIProgress进度条

- UIProgressView进度条，通过alloc和init进行创建和初始化。
- 进度条的位置、宽度可设置，但高度不可设置。

```
@interface ViewController ()

@property(retain, nonatomic) UISlider* slider;

@end

@implementation ViewController

/**
 * 懒加载进度条
 */
- (UIProgressView *)progressView {
    if (_progressView == nil) {
        //滑动条
        _progressView = [[UIProgressView alloc] init];
        //设置位置，宽度可设置，但高度不可设置
        _progressView.frame = CGRectMake(10, 100, 300, 40);
    }
    return _slider;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    //添加进度条到控制器的View
    [self.view addSubview:self.progressView];
}

@end
```

#### UIProgress进度条常用Api

- 设置进度条的风格

```
//进度条风格枚举
typedef NS_ENUM(NSInteger, UIProgressViewStyle) {
    UIProgressViewStyleDefault,//normal progress bar 普通进度条
    UIProgressViewStyleBar __TVOS_PROHIBITED,//for use in a toolbar 用于工具栏
};
self.progressView.progressViewStyle = UIProgressViewStyleDefault;
```

- 设置进度条已有进度的颜色，默认为蓝色

```
//设置已有进度为红色
self.progressView.progressTintColor = [UIColor redColor];
```

- 设置剩余的进度的颜色，默认为灰色

```
//设置剩余进度为红色
self.progressView.trackTintColor = [UIColor blueColor];
```

- 设置进度条的当前进度，最大值：1，最小值：0

```
self.progressView.progress = 0.5;
```

----------------------------

#### 创建UISlider拖动条

- UISlider拖动条和UIProgress进度条一样，也是使用alloc和init来创建和初始化。
- 同样也只能设置位置、宽度，但是高度不可设置。

```
@interface ViewController ()

@property(retain, nonatomic) UISlider* slider;

@end

@implementation ViewController

/**
 * 懒加载滑动条
 */
- (UISlider *)slider {
    if (_slider == nil) {
        //滑动条
        _slider = [[UISlider alloc] init];
        //设置位置，宽度可设置，但高度不可设置
        _slider.frame = CGRectMake(10, 200, 300, 40);
    }
    return _slider;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    //添加拖动条到控制器的View
    [self.view addSubview:self.slider];
}

@end
```

#### UISlider拖动条常用Api

- 设置拖动条的最大值

```
self.slider.maximumValue = 1;
```

- 设置拖动条的最小值，可以设置为负值

```
self.slider.minimumValue = 0;
```

- 设置滑块左侧背景颜色，默认为蓝色

```
self.slider.minimumTrackTintColor = [UIColor blueColor];
```

- 设置滑块右侧背景颜色，默认为灰色

```
self.slider.maximumTrackTintColor = [UIColor greenColor];
```

- 设置滑块颜色，默认为白色

```
self.slider.thumbTintColor = [UIColor orangeColor];
```

- 设置滑动条滑动事件回调

```
[self.slider addTarget:self action:@selector(pressSlider:) forControlEvents:UIControlEventValueChanged];
```

#### 示例

我们对UISlider拖动条设置拖动监听，同时改变UIProgress进度条的进度。

```
@interface ViewController ()

@property(retain, nonatomic) UIProgressView* progressView;
@property(retain, nonatomic) UISlider* slider;

@end

@implementation ViewController

/**
 * 懒加载进度条
 */
- (UIProgressView *)progressView {
    if (_progressView == nil) {
        _progressView = [[UIProgressView alloc] init];
        //进度条的高度不可变
        _progressView.frame = CGRectMake(10, 100, 300, 40);
        //设置进度条风格颜色
        _progressView.progressTintColor = [UIColor redColor];
        //设置进度条剩余的进度颜色
        _progressView.trackTintColor = [UIColor blueColor];
        //设置进度条当前进度，最大值：1，最小值：0
        _progressView.progress = 0.5;
        //设置进度条的风格特征
        _progressView.progressViewStyle = UIProgressViewStyleDefault;
    }
    return _progressView;
}

/**
 * 懒加载拖动条
 */
- (UISlider *)slider {
    if (_slider == nil) {
        //滑动条
        _slider = [[UISlider alloc] init];
        //高度不可设置
        _slider.frame = CGRectMake(10, 200, 300, 40);
        //设置最大值
        _slider.maximumValue = 1;
        //设置最小值，可以为负值
        _slider.minimumValue = 0;
        //设置滑块左侧背景颜色
        _slider.minimumTrackTintColor = [UIColor blueColor];
        //设置滑块右侧背景颜色
        _slider.maximumTrackTintColor = [UIColor greenColor];
        //设置滑块颜色
        _slider.thumbTintColor = [UIColor orangeColor];
        //设置滑动条滑动事件回调
        [_slider addTarget:self action:@selector(pressSlider:) forControlEvents:UIControlEventValueChanged];
    }
    return _slider;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    //添加进度条
    [self.view addSubview:self.progressView];
    //添加拖动条
    [self.view addSubview:self.slider];
}

- (void) pressSlider:(UISlider*) slider {
    //滑动条和进度条同步进度
    _progressView.progress = slider.value;
    NSLog(@"滑动条滑动，值为: %f", slider.value);
}

@end
```