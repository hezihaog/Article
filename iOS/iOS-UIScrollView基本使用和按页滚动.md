#### iOS-UIScrollView基本使用和按页滚动

要让一组视图View一起滚动，就需要滚动视图。在Android上提供了ScrollView，而iOS则提供了UIScrollView。UIScrollView比Android上的ScrollView多出3种特性：

1. 原生支持内容视图超出自身大小范围外。
2. 原生支持拖动越界回弹的功能。
3. 支持按页滚动视图。

前2个特性是Android项目中一直费尽心思想实现的功能，但都不理想，或者还有手势冲突等问题。第三个特性一般用ViewPager来做，而不是用ScrollView，并且还自带复用功能。那UIScrollView可以算是没有复用功能的ViewPager了。本篇就来看看这个牛逼的UIScrollView吧。

![效果截图.png](https://upload-images.jianshu.io/upload_images/1641428-f8b36a575f6399e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### UIScrollView创建

- UIScrollView通过alloc和init进行创建和初始化，UIScrollView除了要设置frame外，还需要设置contentSize，代表内滚动的范围，就是因为这个属性，才能有内容视图超过自身大小范围的能力。

```
#import "ViewController.h"

@interface ViewController ()

/**
 * 滚动视图
 */
@property(retain,nonatomic) UIScrollView* scrollView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //获取控制器View的宽、高
    CGFloat vcViewWidth = self.view.frame.size.width;
    CGFloat vcViewHeight = self.view.frame.size.height;
    
    //创建UIScrollView
    _scrollView = [[UIScrollView alloc] init];
    //设置UIScrollView的位置和宽高为控制器View的宽高
    _scrollView.frame = CGRectMake(0, 0, vcViewWidth, vcViewHeight);
    //设置画布大小，一般比frame大，这里设置横向能拖动4张图片的范围
    _scrollView.contentSize = CGSizeMake(vcViewWidth * 4, vcViewHeight);
    
    //将ScrollView添加到控制器的View上
    [self.view addSubview:_scrollView];
}

@end
```

#### 添加内容

经过上面的设置，我们的UIScrollView已经添加到屏幕中了，但是还没有内容，接下来我们给UIScrollView添加4张图片，分别横向排列。

1. 我们往项目中放4张图片，文件名分别为1、2、3、4.jpg。

![UIScrollView的4张图片.png](https://upload-images.jianshu.io/upload_images/1641428-fe4aab2f5c64cce2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

2. 实例化4个ImageView，添加到UIScrollView中。

```
//创建图片ImageView添加到ScrollView中
for(int i = 0; i < 4; i++) {
    NSString* imgName = [NSString stringWithFormat:@"%d.jpg", i + 1];
    UIImage* img = [UIImage imageNamed:imgName];
    UIImageView* imgView = [[UIImageView alloc] initWithImage:img];
    imgView.frame = CGRectMake(vcViewWidth * i, 0, vcViewWidth, vcViewHeight);
    [_scrollView addSubview:imgView];
}
```

#### UIScrollView的其他设置

- 开启、禁用滚动，默认为YES

```
_scrollView.scrollEnabled = YES;
```

- （特性一）是否可以边缘弹动效果，默认为YES，能让内容超过自身大小

```
_scrollView.bounces = YES;
```

- （特性二）开启、禁用纵向、横向弹动，这2个属性控制不同方向的越界拖动、回弹，默认都为YES

```
//开启横向弹动效果
_scrollView.alwaysBounceHorizontal = YES;
//关闭纵向弹动效果
_scrollView.alwaysBounceVertical = NO;
```

- （特性三）是否按照整页滚动，默认为NO，设置为YES就是按页来滚动

```
_scrollView.pagingEnabled = YES;
```

- 显示、隐藏滚动进度条，默认为YES

```
//隐藏横向滚动条
_scrollView.showsHorizontalScrollIndicator = NO;
//隐藏竖向滚动条
_scrollView.showsVerticalScrollIndicator = NO;
```

#### UIScrollView设置代理

UIScrollView的代理，就是设置UIScrollView上的事件回调的协议，一般我们让当前控制器来实现协议。

1. 视图控制器实现UIScrollView的代理协议UIScrollViewDelegate。

```
文件：ViewController.h

@interface ViewController : UIViewController<UIScrollViewDelegate>

@end
```

2. 给UIScrollView设置代理，并复写代理方法。代理方法都在代码里，就不一一列举了。

```
文件：ViewController.m

#import "ViewController.h"

@interface ViewController ()

/**
 * 滚动视图
 */
@property(retain,nonatomic) UIScrollView* scrollView;

@end

- (void)viewDidLoad {
    [super viewDidLoad];
    
    //...省略上面提到的设置
 
	 //将ScrollView添加到控制器的View上
    [self.view addSubview:_scrollView];
    //设置代理
    _scrollView.delegate = self;   
}

//滚动视图移动时回调
- (void) scrollViewDidScroll:(UIScrollView *)scrollView {
    NSLog(@"视图移动 x: %f", scrollView.contentOffset.x);
}

//滚动视图结束拖动时回调
- (void) scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate {
    NSLog(@"视图结束拖动");
}

//滚动视图即将开始拖动时回调
- (void) scrollViewWillBeginDragging:(UIScrollView *)scrollView {
    NSLog(@"滚动视图即将开始拖动");
}

//滚动视图结束拖动时回调
- (void) scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset {
    NSLog(@"滚动视图结束拖动");
}

//视图即将减速时调用
- (void) scrollViewWillBeginDecelerating:(UIScrollView *)scrollView {
    NSLog(@"视图即将减速");
}

//视图已经结束减速时回调
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView {
    NSLog(@"视图已经结束减速");
}

@implementation ViewController
```

#### 让UIScrollView滚动到指定位置

1. 马上让ScrollView滚动到指定位置，没有缓慢滚动的效果。

```
self.scrollView.contentOffset = CGPointMake(0, 0);
//或者
[self.scrollView scrollRectToVisible:CGRectMake(0, 0, 300, 400) animated:NO];
```

2. 支持缓慢的动画滚动效果。

```
[self.scrollView scrollRectToVisible:CGRectMake(0, 0, 300, 400) animated:YES];
```