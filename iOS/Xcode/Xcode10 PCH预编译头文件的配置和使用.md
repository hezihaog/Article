#### Xcode10 PCH预编译头文件的配置和使用

- 首先来看一下PCH文件是什么？

PCH：预编译头文件（Precompile Prefix Header File），以.pch后缀结尾。

- PCH文件的优点

	1. 将常用的头文件（例如：第三方库头文件）引入，不再需要每个使用的文件中引入。
	2. 可以定义对象宏，直接使用即可（例如：屏幕宽高、机型判断）。
	3. 定义函数宏进行条件编译（例如：Debug环境和Release环境下NSLog的输出）。

- PCH的缺点

	1. 不利于代码移植，因为层层依赖。
	2. PCH中引入太多的头文件，导致编译时间加长。

#### 创建PCH文件

1. command + N，选择iOS -> Other -> PCH File，选择后，点击Next进行创建。

![定义PCH文件名.png](https://upload-images.jianshu.io/upload_images/1641428-63777a0c3201e301.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 定义.pch文件的文件名，一般与项目名一致。

![选择创建PCH文件.png](https://upload-images.jianshu.io/upload_images/1641428-f50d13b68ede8eeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 选中项目，按照图片中的步骤选中，然后再搜索框中搜索**Prefix Header**，找到Apple Clang - Language。
	- 将Precompile Prefix Header，设置为YES，默认为NO。
	- 下面的Prefix Header，双击右侧空格，将拷贝的地址填入，地址要将前面的绝对路径改为相对路径，替换为$(SRCROOT)，格式为：$(SRCROOT)/项目名/.pch 文件名。

![配置PCH文件到项目步骤.png](https://upload-images.jianshu.io/upload_images/1641428-9139b39fe23b626d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![开启PCH预编译.png](https://upload-images.jianshu.io/upload_images/1641428-f54f1cf317bbf05c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![拷贝PCH文件路径.png](https://upload-images.jianshu.io/upload_images/1641428-42ed26526ffde75d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![配置PCH文件路径.png](https://upload-images.jianshu.io/upload_images/1641428-b5aa2d39cd0ad0aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### PCH文件中添加常用宏和常用头文件

贴上我使用到的宏和常用头文件。

```
// Include any system framework and library headers here that should be included in all compilation units.
// You will also need to set the Prefix Header build setting of one or more of your targets to reference this file.

//----------------------- 对象宏 -----------------------

//----------------------- 屏幕相关 -----------------------
#define SCREEN_WIDTH [UIScreen mainScreen].bounds.size.width
#define SCREENH_HEIGHT [UIScreen mainScreen].bounds.size.height

//型号相关
//判断是否为iPhone
#define IS_IPHONE (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone)
//判断是否为iPad
#define IS_IPAD (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad)
//判断是否为ipod
#define IS_IPOD ([[[UIDevice currentDevice] model] isEqualToString:@"iPod touch"])
// 判断是否为 iPhone 5SE
#define iPhone5SE [[UIScreen mainScreen] bounds].size.width == 320.0f && [[UIScreen mainScreen] bounds].size.height == 568.0f
// 判断是否为iPhone 6/6s
#define iPhone6_6s [[UIScreen mainScreen] bounds].size.width == 375.0f && [[UIScreen mainScreen] bounds].size.height == 667.0f
// 判断是否为iPhone 6Plus/6sPlus
#define iPhone6Plus_6sPlus [[UIScreen mainScreen] bounds].size.width == 414.0f && [[UIScreen mainScreen] bounds].size.height == 736.0f

//----------------------- 版本相关 -----------------------

//获取系统版本
#define IOS_SYSTEM_VERSION [[[UIDevice currentDevice] systemVersion] floatValue]
//判断 iOS 8 或更高的系统版本
#define IOS_VERSION_8_OR_LATER (([[[UIDevice currentDevice] systemVersion] floatValue] >=8.0)? (YES):(NO))

//判断是真机还是模拟器
#if TARGET_OS_IPHONE
//iPhone Device
#endif
#if TARGET_IPHONE_SIMULATOR
//iPhone Simulator
#endif

//弱指针self
#define WeakSelf(weakSelf)  __weak __typeof(&*self)weakSelf = self

//----------------------- 函数宏 -----------------------

//自定义log输出，debug时，正常NSLog输出，release状态，为空，不打印
#ifdef DEBUG //调试环境

#define XJLog(...) NSLog(__VA_ARGS__)//__VA_ARGS__为调用时传递的参数

#else //发布环境

#define XJLog(...)

#endif

//----------------------- 公共头文件引用 -----------------------
#ifdef __OBJC__

#import "AppDelegate.h"

//-------------------- 第三方类库 --------------------

//网络请求
#import "AFNetworking.h"
//Toast提示
#import "Toast.h"
//Loading加载
#import "MBProgressHUD.h"
//模型转换
#import "MJExtension.h"
//图片加载
#import<SDWebImage/UIImageView+WebCache.h>
//下拉刷新
#import "MJRefresh.h"
//图片浏览
#import <SYPhotoBrowser/SYPhotoBrowser.h>

//可以不写mas前缀
//define this constant if you want to use Masonry without the 'mas_' prefix
#define MAS_SHORTHAND

//自动装箱，mas_equalTo可以写为equalTo
//define this constant if you want to enable auto-boxing for default syntax
#define MAS_SHORTHAND_GLOBALS

//上面2个宏，必须在导入Masonry.h头文件之间添加，头文件会判断是否包含上面2个宏进行处理
//自动布局
#import "Masonry.h"

//-------------------- 第三方类库 --------------------
```