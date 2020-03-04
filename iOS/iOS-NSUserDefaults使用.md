#### iOS-NSUserDefaults使用

无论是Android、H5还是PC端，都有需要将数据保存到本地，后续获取使用的需求，例如Key-Value保存，在Android中提供了SharedPreferences来保存，iOS则提供了NSUserDefaults。

在NSUserDefaults中，可以保存的数据结构有：

1. NSString 字符串
2. NSNumber 数字
3. NSArray 数组
4. NSDictionary 字典
5. NSDate 日期
6. NSData 二进制数据

如果需要保存自定义对象，则需要将对象归档为NSData，再进行保存。类比安卓的话，安卓要将自定义对象保存到SharedPreferences，有2种方式：

1. 一种是将对象转为json作为字符串保存。
2. 使用对象输出流ObjectOutputStream序列化对象后保存，获取时再反序列化回对象。

iOS的归档，应该就类似Java的序列化和反序列化机制。

#### 基本Api

- 获取NSUserDefaults实例

```
NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
```

- 存储数据

```
//注意要保存的对象object在前，key在后
[defaults setObject:object forKey:key];

//例如：保存字符串
[defaults setObject:@"Wally" forKey:@"NAME"];
//保存Number对象
[defaults setObject:number forKey:@"NUMBER"];
```

- 获取数据

```
//用key获取保存的对象，返回值为id类型，所以可以接任何指针
id object = [defaults objectForKey:key];

//例如：获取保存的字符串
id name = [defaults objectForKey:@"NAME"];
NSLog(@"名字: %@", name);
//获取保存的Number对象
id num = [defaults objectForKey:@"NUMBER"];
NSLog(@"num: %@", num);
```

- 马上写入数据，保存完数据后，如果马上抛出异常，可能会丢失数据，所以建议保存完后，调用synchronize方法，通知马上写入。

```
[defaults synchronize];
```

- 删除数据

```
//使用key删除保存的值
[defaults removeObjectForKey:@"ARRAY"];
```

#### 快捷Api

NSUserDefaults可以存储上面说到的6种类型，苹果爸爸还给我们提供了直接转换出对应类型的方法，避免每次都写强转代码。

1. 存储

```
//存储Int值
[defaults setInteger:123 forKey:@"INT"];
    
//存储布尔值
[defaults setBool:YES forKey:@"BOOL"];
    
//存储浮点型
[defaults setFloat:3.14 forKey:@"FLOAT"];
    
//存储数组
NSArray* arr = [NSArray arrayWithObjects:@"Wally", @"Barry", @"Rose", nil];
[defaults setObject:arr forKey:@"ARRAY"];
```

2. 获取

```
//获取Int值
NSLog(@"int: %ld", [defaults integerForKey:@"INT"]);
    
//获取BOOL布尔值
NSLog(@"bool: %d", [defaults boolForKey:@"BOOL"]);

//获取浮点值
NSLog(@"float: %f", [defaults floatForKey:@"FLOAT"]);

//获取数组
NSArray* arr = [defaults objectForKey:@"ARRAY"];
NSLog(@"%@", arr);
```

#### 保存自定义对象

上面说到如果需要保存自定义对象，需要将对象转为NSData再进行保存。例如我们保存用户登录信息，保存用户名和密码，操作分为以下步骤：

1. 定义用户信息模型，实现NSCoding协议。

```
//文件：FSUser.h

@interface FSUser : NSObject<NSCoding>

/**
 * 用户名
 */
@property(nonatomic, strong) NSString *username;
/**
 * 密码
 */
@property(nonatomic, strong) NSString *password;

/**
 * 获取对象所有属性组合到的一个字符串
 */
- (NSString *) toString;

@end
```

2. 复写NSCoding协议中的，initWithCoder方法和encodeWithCoder方法，initWithCoder方法是为了解档，用数据恢复对象，initWithCoder方法是为了将对象归档。

```
//文件：FSUser.m

@implementation FSUser

/**
 * 恢复
 */
- (instancetype)initWithCoder:(NSCoder *)coder
{
    self = [super init];
    if (self) {
        self.username = [coder decodeObjectForKey:@"_username"];
        self.password = [coder decodeObjectForKey:@"_password"];
    }
    return self;
}

/**
 * 归档写入
 */
- (void)encodeWithCoder:(NSCoder *)coder
{
    [coder encodeObject:_username forKey:@"_username"];
    [coder encodeObject:_password forKey:@"_password"];
}

- (NSString *)toString {
    return[NSString stringWithFormat:@"用户名: %@, 密码: %@", _username, _password];
}

@end
```

3. 归档（序列化）：使用NSKeyedUnarchiver，调用archivedDataWithRootObject方法，归档对象。

```
NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

//存储自定义对象
FSUser *user = [[FSUser alloc] init];
user.username = @"Wally";
user.password = @"123456";
//归档保存
NSData *userData = [NSKeyedArchiver archivedDataWithRootObject:user];
[defaults setObject:userData forKey:@"USER"];
```

4. 解档（反序列化）：使用NSKeyedUnarchiver，调用unarchiveObjectWithData方法，解档对象。

```
NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

//获取自定义对象
NSData *userData = [defaults objectForKey:@"USER"];
//解档，恢复对象
FSUser *user = [NSKeyedUnarchiver unarchiveObjectWithData:userData];
//输出：用户名: Wally, 密码: 123456
NSLog(@"%@", [user toString]);
```