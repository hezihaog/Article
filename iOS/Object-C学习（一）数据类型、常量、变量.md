## Object-C学习（一）数据类型、常量、变量

- Object-C的数据类型

	* 基本类型（科普一下：1个字节，8位）
		* 整形
			* short int 短整形（2字节，16位）
			* int整形（4字节，32位）
			* long int（Mac上为8字节 64位，IOS为4字节，32位）
			* long long（简称long）长整形（8个字节，64位）
		* 字符型（1字节，所以不支持中文）
		* 浮点型
			* float类型
			* double类型
		* 枚举型
	* 构造类型
		* 数组类型
		* 结构体
		* 共用体
	* 指针类型

### 整形

```
//short int
short shortNum = 10086;
NSLog(@"数值：%hd", shortNum); 

//int
int intNum = 10010;
NSLog(@"int 数值：%d", intNum);

//long int
long int longIntNum = 6666;
NSLog(@"long int 数值：%ld", longIntNum);

//long long
long long longLongNum = 88888888;
NSLog(@"long long 数值：%lld", longLongNum);
```

### 字符型

```
char myChar = 'A';
NSLog(@"%c", myChar);
```

### 浮点型

```
float myFloat = 3.14;
//输出小数点后6位，不够补全，这里输出3.140000
NSLog(@"%f", myFloat);

double myDouble = 3.1415926;
NSLog(@"%lf", myDouble);
```

#### 枚举

- 枚举实际是无符号整形，因此可以将枚举赋值给整形，也可以将整形赋值给枚举

```
//有名枚举，flag
enum flag {success = 1, fail = 0};
int result = 1;
if (result == success) {
    NSLog(@"成功了");
} else {
    NSLog(@"失败了");
}

//匿名枚举，同时定义2个枚举变量
enum {male = 1, femal = 2} me, you;
me = male;
you = femal;
NSLog(@"我是：%u", me);
NSLog(@"你是：%u", femal);
```

#### 布尔值
```
//布尔值，OC底层使用signed char 来代表YES和NO
BOOL boolResult = YES;
BOOL boolResult2 = NO;
NSLog(@"%d", boolResult);//1
NSLog(@"%d", boolResult2);//0
```

#### 常量，只能赋值一次
```
//常量
const int constA = 123;
//常量只能赋值一次，再次赋值报错
//constA = 876;
NSLog(@"常量值：%d", constA);
```

#### 静态变量
```
//静态变量
static int staticB = 01234;
//重新赋值staticB = 444;
NSLog(@"静态变量：%d", staticB);
```

#### 静态常量
```
//静态常量
static const int MAX_VALUE = 10000;
NSLog(@"静态常量：%d", MAX_VALUE);
```