# JavaScript-ES6学习

## let

- let特性：块级作用域

```
var声明的变量会越域
let声明的变量有严格的作用域
{
    var a = 1;
    let b = 2;
}
console.log(a);//1
console.log(b);//ReferenceError：b is not defined
```

- let特性：不能重复声明

```
//var可以声明多次
//let只能声明一次
var m = 1;
var m = 2;
let n = 3;
//重复声明，就会报错，Uncaught SyntaxError: Identifier 'n' has already been declared
let n = 4;
console.log(m);//2
console.log(n);//Uncaught SyntaxError: Identifier 'n' has already been declared
```

- let特性：不会变量提升

```
//var会变量提升
//let不存在变量提升
console.log(x);//undefined
var x = 10;
console.log(y);//Uncaught ReferenceError: Cannot access 'y' before initialization
let y = 20;
```

## const常量

```
//1：const声明后，不允许改变
//2：一旦声明必须初始化，否则会报错
const a = 1;
a = 3;//修改常量，报错：Uncaught TypeError: Assignment to constant variable.
```

## 解构表达式

- 数组解构

```
let arr = [1,2,3];
//一个个赋值，太麻烦了
let a = arr[0];
let b = arr[1];
let c = arr[2];

//数组解构，一行搞定
let[a,b,c] = arr;
console.log(a);
console.log(b);
console.log(c);
```

- 对象解构

```
const person = {
    name: "jack",
    age: 21,
    language: ['java', 'js', 'css']
}
//一行行取出对象中的属性，麻烦
const name = person.name;
const age = person.age;
const language = person.language;

//对象解耦，一行搞定
const {name,age,language} = person;
console.log(name, age, language);

//如果想解构后，改名，也是可以的
const {name:n,age:a,language:l} = person;
console.log(n, a, l);
```

## 字符串拓展

-  字符串新增的4个方法

```
let str = "hello.vue";
//判断是不是以指定字符串开始
console.log(str.startsWith('hello'));//true
//判断是不是以指定字符串结束
console.log(str.endsWith('vue'));//true
//是否包含某个字符串
console.log(str.includes('e'));//true
//是否包含某个字符串
console.log(str.includes('hello'));//true
```

- 字符串模板

```
//2个反引号包裹字符串，保留换行格式
let ss = `<div>
        <span>hello world</span>
        </div>
`
console.log(ss);

//插值表达式，用反引号来包裹内容
const person = {
    name: "jack",
    age: 21,
    language: ['java', 'js', 'css']
}
function fun() {
    return "这是一个函数"
}
const {name,age,language} = person;
//支持插值变量，表达式，方法运算等
let info = `我是${name}, 今年${age + 10}岁了，我想说${fun()}`;
console.log(info);
```

## 函数优化

- 函数默认值

```
//在ES6之前，我们无法给一个函数的参数设置默认值，只能通过变通写法
function add(a, b) {
    //判断b是否为空，为空则赋值一个1为默认值
    b = b || 1;
    return a + b;
}
//传1个值
console.log(add(10));

//现在可以这么写，直接给参数写上默认值，没传则会自动设置上默认值
function add2(a, b = 1) {
    return a + b;
}
console.log(add2(20));
```

- 不定参数

```
function fun(...values) {
    console.log(values.length);
}
fun(1, 2);//2
fun(1, 2, 3, 4);//4
```

- 箭头函数

```
//以前声明一个方法
// var print = function(obj) {
//     console.log(obj);
// }
//箭头函数，声明方法
var print = obj => console.log(obj);
print('hello');

//2个参数的方法
var sum = function(a, b) {
    var c = a + b;
    return c;
}
//多个参数用括号括起来，一行的方法体可以不写花括号
var sum2 = (a,b) => a + b;
console.log(sum2(11, 12));

//多行方法体
var sum3 = (a,b) => {
    var c = a + b;
    return c;
}
console.log(sum3(10, 20));

const person = {
    name: "jack",
    age: 21,
    language: ['java', 'js', 'css']
}

function hello(person) {
    console.log("hello，" + person.name);
}

//箭头函数声明方法
//var hello2 = (param) => console.log("hello，" + person.name);
//箭头函数+解构，既然只需要name属性，那直接解耦对象的name属性出来即可
var hello2 = ({name}) => console.log("hello，" + name);
hello2(person)
```

## 对象优化

- Object新增了4个方法

```
const person = {
    name: "jack",
    age: 21,
    language: ['java', 'js', 'css']
}
//keys()方法，获取对象中的所有属性的名称
console.log(Object.keys(person));//["name", "age", "language"]
//values()方法，获取对象中所有属性的值
console.log(Object.values(person));//["jack", 21, Array(3)]
//entries()方法，获取对象中所有属性的名称和值，数组中有多个数组，每个数组里有2个值，分别是key和value
console.log(Object.entries(person));//[Array(2), Array(2), Array(2)]

//合并属性
const target = { a:1 };
const source1 = { b: 2 }
const source2 = { c: 3 }
//合并成：{a:1,b:2,c:3}，assign()方法，第一个参数是目标对象，后面的参数都是要背复制属性的对象
Object.assign(target, source1, source2);
console.log(target);//{a: 1, b: 2, c: 3}
```

- 声明对象的简写

```
const age = 23;
const name = "张三"
//属性名和属性值，一一对应赋值
const person1 = {age: age, name: name}

//2、简写：如果属性名和属性值是一样的，就可以只写一个
const person2 = {age, name}
console.log(person2);//{age: 23, name: "张三"}

//3、对象的函数属性简写
let person3 = {
    name: 'Jack',
    //以前的写法
    eat: function(food) {
        console.log(this.name + "在吃" + food);
    },
    //箭头函数this不能使用，需要用对象来调用
    eat2: food => console.log(person3.name + "在吃" + food),
    //另外一种写法
    eat3(food) {
        console.log(this.name + "在吃" + food);
    }
}
person3.eat("香蕉");
person3.eat2("苹果");
person3.eat3("橘子");
```

- 对象拓展运算符

```
//1）拷贝对象（深拷贝）
let p1 = {name: 'Amy', age: 15}
let someone = {...p1}
console.log(someone);

//2）合并对象
let age1 = { age: 15 }
let name1 = { name: 'Amy' }
//如果之前已经有一个同名属性，那么拷贝的值会覆盖掉之前的
let p2 = {
    name: "张三"
}
//将age1的所有属性放到p2，再将name1的所有属性也放到p2中
p2 = {...age1, ...name1}
console.log(p2);//{age: 15, name: "Amy"}
```

## map和reduce

```
//数组中新增了map和reduce方法
//map()，接收一个函数，将原数组中的所有元素用这个函数处理后，再放入新数组返回
let arr = ['1', '20','-5','3'];

//传入一个函数，对每个元素，进行乘以2，再返回
// arr = arr.map((item) => {
//     return item * 2
// })
arr = arr.map(item => item * 2);

//[2, 40, -10, 6]
console.log(arr);

//reduce()，为数组中每一个元素依次执行回调函数，不包括数组中被删除或从未赋值的元素
//格式：arr.reduce(callback, [initialValue]);，initialValue为初始值，可不传
/**
 * 参数
 * 1、previousValue：上一次调用回调返回的值，或者是提供的初始化值initialValue
 * 2、currentValue：数组中当前被处理的元素
 * 3、index：当前元素在数组中的索引
 * 4、array：调用reduce的数组
 */
 //没有指定初始值时，从数组的第一个元素，就是2开始，和后面的值进行累加
 //如果指定了初始值，就从初始化值开始，和第一个元素开始，相当于强行插入了一个值进数组的第一位一样
 let result = arr.reduce((previousValue, currentValue) => {
    console.log("上一次处理后：" + previousValue);
    console.log("当前正在处理：" + currentValue);
    return previousValue + currentValue;
}, 100);
console.log(result);
```

## promise

演示jQuery的ajax和promise，这2种方式的写法对比，promise化解了ajax的嵌套地狱。

准备模拟数据，在根目录建立mock文件夹，再建立以下3个文件，放到其中

1. user.json。用户信息，用id查询下该用户的科目数据（user_corse_1.json）

```
{
    "id": 1,
    "name": "zhangsan",
    "password": "123456"
}
```

2. user_corse_1.json。用户的科目id和科目名，用科目id查询，科目成绩（corse_score_10.json）

```
{
    "id": 10,
    "name": "chinese"
}
```

3. corse_score_10.json。科目的成绩信息

```
{
    "id": 100,
    "score": 90
}
```

- 建立html，ajax请求数据，3个接口要串联起来调用，会形成回调地狱

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>promise</title>
</head>
<!-- 引入jQuery -->
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
<body>
    <script>
        //1.查出当前用户信息
        //2.按照当前用户的id，查出他的课程
        //3.按照当前课程id，查出分数
        $.ajax({
            url: "mock/user.json",
            success(data) {
                console.log("查询用户：" + JSON.stringify(data));
                $.ajax({
                    url: `mock/user_corse_${data.id}.json`,
                    success(data) {
                        console.log("查询到课程：" + JSON.stringify(data));
                        $.ajax({
                            url: `mock/corse_score_${data.id}.json`,
                            success(data){
                                console.log("查询到分数：" + JSON.stringify(data));
                            },
                            error(error) {
                                console.log("出现异常了：" + error);
                            }
                        });
                    },
                    error(error) {
                        console.log("出现异常了：" + error);
                    }
                });
            },
            error(error) {
                console.log("出现异常了：" + error);
            }
        });
    </script>
</body>
</html>
```

- 使用promise优化

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>promise</title>
</head>
<!-- 引入jQuery -->
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
<body>
    <script>
        //1.查出当前用户信息
        //2.按照当前用户的id，查出他的课程
        //3.按照当前课程id，查出分数
        
        //...省略ajax回调地狱的方式

        //1、Promise可以封装异步操作
        let p = new Promise((resolve, reject) => {
            //1）异步操作
            $.ajax({
                url: "mock/user.json",
                success(data) {
                    //请求成功
                    console.log("查询用户：" + JSON.stringify(data));
                    resolve(data);
                },
                error(error) {
                    //请求失败
                    reject(error);
                }
            })
        });
        p.then((data) => {
            return new Promise((resolve, reject) => {
                $.ajax({
                    url: `mock/user_corse_${data.id}.json`,
                    success(data) {
                        console.log("查询课程成功：" + JSON.stringify(data));
                        resolve(data);
                    },
                    error(error) {
                        reject(error)
                    }
                });
            });
        }).then((data) => {
            $.ajax({
                url: `mock/corse_score_${data.id}.json`,
                success(data){
                    console.log("查询到分数：" + JSON.stringify(data));
                },
                error(error) {
                    console.log("出现异常了：" + error);
                }
            });
        }).catch((error) => {
            console.log("出现异常了：" + error);
        });
    </script>
</body>
</html>
```

- 上面的promise代码还是有点多，可以抽取方法来简化

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>promise</title>
</head>
<!-- 引入jQuery -->
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
<body>
    <script>
        //1.查出当前用户信息
        //2.按照当前用户的id，查出他的课程
        //3.按照当前课程id，查出分数

        //...省略ajax回调地狱的方式

        //抽取方法
        function get(url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    url: url,
                    data: data,
                    success: function(result) {
                        resolve(result);
                    },
                    error: function(error) {
                        reject(error);
                    }
                });
            });
        }

        //------------------ 抽取方法后 ------------------

        //查询当前用户信息
        get("mock/user.json")
            .then((data) => {
                console.log("~~~用户查询成功：" + JSON.stringify(data));
                //查询完用户，下一步查询课程
                return get(`mock/user_corse_${data.id}.json`)
            }).then((data) => {
                console.log("~~~课程查询成功：" + JSON.stringify(data));
                return get(`mock/corse_score_${data.id}.json`);
            }).then((data) => {
                console.log("~~~课程成绩查询成功：" + JSON.stringify(data));
            }).catch((error) => {
                console.log("~~~出现异常：" + error);
            });
    </script>
</body>
</html>
```

## 模块化

准备2个工具js文件，分别是hello.js和user.js

- hello.js，提供一个sum方法，传入2个值，返回2个值相加后的结果，通过export关键字导出

```
//直接在声明处，使用export进行导出，提供方这里直接写util，依赖方就只能写util了，所以还有下面一种方式
// export const util = {
//     sum(a, b) {
//         return a + b;
//     }
// }

//导出时，不起名字，依赖方导入时，再起名字
export default {
    sum(a, b) {
        return a + b;
    }
}

//export不仅可以导出对象，一切JS变量都可以导出，比如：基本类型变量、函数、数组、对象
//export {util}
```

- user.js，提供2个变量，name和age，以及add方法，通过export关键字导出

```
var name = "jack"
var age = 21

function add(a, b) {
    return a + b;
}

//批量导出变量
export {name, age, add}
```

- main.js，主函数js文件，通过import关键字导入hello.js的util，和user.js的变量name、age，以及方法add

```
// 导入hello.js中的util对象
import util from "./hello.js"
//导入user.js的变量和方法
import {name, age, add} from "./user.js"

//使用util对象的sum()方法
util.sum(1, 2);
//使用name变量和add方法
console.log(name);
add(1, 3);
```