#### iOS-UIRefreshControl自带刷新控件

Android原生提供下拉刷新控件是SwipeRefreshLayout，iOS则是UIRefreshControl，原生控件好处就是不同导入第三方库，减少包体积，不过原生控件的拓展性会差一点，也有一些缺点。

例如UIRefreshControl，只要下拉达到一个阀值就触发刷新行为，不能再反向拖回去取消刷新行为。

![UIRefreshControl示例.png](https://upload-images.jianshu.io/upload_images/1641428-2344bd51423baa16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

#### 常用配置

既然是刷新控件，就必然有提供开始刷新、结束刷新、以及刷新回调方法。

- 创建UIRefreshControl

```
_refreshControl = [[UIRefreshControl alloc] init];
```

- 绑定下拉刷新到TableView，refreshControl是UIScrollView的属性，而UITableView是继承于UIScrollView，所以UITableView也有这个属性

```
_tableview.refreshControl = _refreshControl;
```

- 开始刷新

```
[_refreshControl beginRefreshing];
```

- 结束刷新

```
[_refreshControl endRefreshing];
```

- 是否正在刷新

```
BOOL isRefreshing = [_refreshControl isRefreshing];
```

- 设置下拉刷新事件回调

```
[_refreshControl addTarget:self action:@selector(onRefresh) forControlEvents:UIControlEventValueChanged];
```

#### 样式配置

经过上面3个基本配置，就可以实现下拉刷新功能了，UIRefreshControl也提供了样式的一些例如刷新菊花的颜色、刷新标题等配置。

- 设置刷新菊花的颜色

```
_refreshControl.tintColor = [UIColor lightGrayColor];
```

- 设置刷新的标题

```
//设置刷新的标题，不想显示文字，则不调用或者传空字符串，不能传nil
_refreshControl.attributedTitle = [[NSAttributedString alloc] initWithString:@"松手刷新"];
```

#### UIRefreshControl配合UITableView实现下拉刷新

UIRefreshControl一般配合UITableView来使用，下拉刷新调用后端接口，解析Json后生成Model实体，将实体插入到列表数据后，再重新加载数据，刷新列表。

```
@interface ViewController ()<UITableViewDelegate, UITableViewDataSource>

/**
 * 列表控件
 */
@property (strong, nonatomic) UITableView *tableview;
/**
 * 下拉刷新控件
 */
@property (strong, nonatomic) UIRefreshControl* refreshControl;

@end

@implementation ViewController

//...省略UITableView代理和数据集方法的复写...

- (void)viewDidLoad {
    [super viewDidLoad];
    //初始化列表控件    
    _tableview = [[UITableView alloc] initWithFrame:self.view.bounds];
    //设置代理
    _tableview.delegate = self;
    _tableview.dataSource = self;
    //添加到控制器的View上
    [self.view addSubview:_tableview];
    
    //配置下拉刷新
    _refreshControl = [[UIRefreshControl alloc] init];
    //设置转圈的颜色
    _refreshControl.tintColor = [UIColor lightGrayColor];
    //设置刷新的标题，不想显示文字，则传空字符串，不能传nil
    _refreshControl.attributedTitle = [[NSAttributedString alloc] initWithString:@"松手刷新"];
    //设置下拉刷新事件回调
    [_refreshControl addTarget:self action:@selector(onRefresh) forControlEvents:UIControlEventValueChanged];
    //绑定下拉刷新到TableView
    _tableview.refreshControl = _refreshControl;
}

/**
 * 下拉刷新事件回调
 */
- (void) onRefresh {
    //延迟1.5秒调用，模拟异步
    [self performSelector:@selector(handlerData) withObject:nil afterDelay:1.5];
}

- (void) handlerData {
    //解析Json，生成Model实体
    Model *newModel = parseJsonToModel(json);
    //向列表中，添加一条新数据
    [_listData insertObject:newModel atIndex:0];
    //结束刷新
    [_refreshControl endRefreshing];
    //重新加载数据，刷新列表
    [_tableview reloadData];
}

@end
```

#### 总结

UIRefreshControl使用还是挺简单的，不过它也有自己的缺点，例如UIRefreshControl只要下拉达到一个阀值就触发刷新行为，不能再反向拖回去取消刷新行为。如果需要上拉加载更多，就需要使用第三方库或者自己写了，第三方下拉刷新库一般使用MJRefresh。