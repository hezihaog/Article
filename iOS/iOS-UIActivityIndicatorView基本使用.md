#### iOS-UIActivityIndicatorView基本使用

在安卓系统中，Loading指示器系统为我们提供了ProgressDialog，后面被废弃了，推荐直接使用ProgressBar来显示，但风格和设计师要求的往往冲突，都需要我们自定义。而iOS提供了UIActivityIndicatorView控件来表示Loading状态，而且还蛮好看的。

![iOS-UIActivityIndicatorView示例.png](https://upload-images.jianshu.io/upload_images/1641428-404580cecd40020d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 指示器创建

UIActivityIndicatorView就是一个View，我们将它实例化后，添加到ViewController的View即可。

- 指示器样式

指示器alloc后，需要通过initWithActivityIndicatorStyle进行初始化，需要传入一个样式枚举，枚举值有以下3种：

```
typedef NS_ENUM(NSInteger, UIActivityIndicatorViewStyle) {
	UIActivityIndicatorViewStyleWhiteLarge,//大号，白色
	UIActivityIndicatorViewStyleWhite,//小号，白色
	UIActivityIndicatorViewStyleGray,//小号，灰色
};
```

- 指示器创建

懒加载的方式，初始化指示器，并且给指示器设置在控制器的中间显示，注意，默认只有一个菊花在转，黑色背景和圆角需要自己设置。

```
@interface ViewController ()

@property(nonatomic, strong) UIActivityIndicatorView *activityIndicator;

@end

@implementation ViewController

/**
 * 懒加载初始化UIActivityIndicatorView
 */
- (UIActivityIndicatorView *)activityIndicator {
    if (_activityIndicator == nil) {
        /**
         * 创建指示器，并设置样式
         * UIActivityIndicatorViewStyleWhiteLarge，大号，白色
         * UIActivityIndicatorViewStyleWhite，小号，白色
         * UIActivityIndicatorViewStyleGray，小号，灰色
         */
        _activityIndicator = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleWhiteLarge];
        //设置指示器的位置和宽高
        _activityIndicator.frame = CGRectMake(0, 0, 100, 100);
        //设置在屏幕中心显示
        _activityIndicator.center = CGPointMake(self.view.center.x, self.view.center.y);
        _activityIndicator.backgroundColor = [UIColor blackColor];
        //将背景设置圆角
        _activityIndicator.layer.cornerRadius = 8;
    }
    return _activityIndicator;
}
```

#### 样式拓展

- 菊花的圈圈可以设置颜色，iOS5时添加的Api

```
self.activityIndicator.color = [UIColor redColor];
```

#### 基本使用

指示器为了代表Loading加载状态，所以就只有3个方法常用，分别是：显示、隐藏和判断是否正在显示。

- 显示指示器

```
/**
 * 显示转圈菊花
 */
- (void) showIndicator {
    [self.activityIndicator startAnimating];
}
```

- 隐藏指示器

```
/**
 * 隐藏转圈菊花
 */
- (void) hideIndicator {
    [self.activityIndicator stopAnimating];
}
```

- 判断是否正在显示

```
/**
 * 是否正在显示
 */
- (BOOL) isShowIndicator {
    return [self.activityIndicator isAnimating];
}
```

#### 示例

```
@interface ViewController ()

@property(nonatomic, strong) UIActivityIndicatorView *activityIndicator;

@end

@implementation ViewController

//...省略上面提到的Api

/**
 * 点击屏幕空白处，显示指示器，1.5秒后隐藏
 */
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //正在显示，忽略
    if ([self isShowIndicator]) {
        return;
    }
    [self showIndicator];
    [self performSelector:@selector(hideIndicator) withObject:nil afterDelay:1.5];
}

@end
```