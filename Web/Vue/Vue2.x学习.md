# Vue2.x学习

## 指令

- v-text和v-html

v-text指令，用于渲染data中的数据，如果数据是html标签，会直接输出，而不会渲染为html元素。
v-html指令，则可以将数据渲染为html元素。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-text、v-html</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        {{msg}} {{1+1}} {{hello()}}<br>
        <span v-html="msg"></span>
        <br>
        <span v-text="msg"></span>
        <!-- href里面不能用插值表达式取得变量，需要用v-bind -->
        <a href="{{link}}">gogogo</a>
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                msg: "<h1>Hello<h1>",
                link: "http://www.baidu.com"
            },
            methods: {
                hello() {
                    return "world"
                }
            },
        })
    </script>
</body>
</html>
```

- v-bind指令

v-bind指令，为标签的属性绑定值，而插值是给标签体里面绑定。
一般使用v-bind绑定标签的class和style。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-bind</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <!-- v-bind指令，为标签的属性绑定值，插值是给标签体里面绑定 -->
    <!-- v-bind指令的格式：v-bind:属性名=要绑定的变量的名称 -->
    <!-- v-bind指令，可以简写，不写v-bind，直接一个:，例如v-bind:class=""，可以简写成:class="" -->
    <!-- v-bind指令是单向绑定，就是V层数据变动，不会影响M层数据，就是不会同步 -->
    <div id="app">
        <!-- 动态绑定超链接 -->
        <a v-bind:href="link">gogogo {{link}}</a>
        <!-- 动态绑定class，语法：{class名: 控制的变量名, class名: 控制的变量名} -->
        <!-- 动态绑定style，语法：{style名: 控制的变量名, style名: 控制的变量名} -->
        <!-- 如果遇到用-分割的，可以转换为驼峰，例如font-size，变为fontSize -->
        <span v-bind:class="{active:isActive, 'text-danger':hasError}"
        v-bind:style="{color: color1,fontSize: size}">你好</span>
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                link: "http://www.baidu.com",
                isActive: true,
                hasError: true,
                color1: 'red',
                size: '36px'
            }
        })
    </script>
</body>
</html>
```

- v-model指令

v-model指令，是双向绑定的，视图层和数据层是绑定并同步的。
一般v-model用于表单项，或自定义组件。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-model</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <!-- v-model，是双向绑定的，就是V层和M层同步 -->
    <!-- v-model，一般用于表单项，或者自定义组件 -->
    <div id="app">
        <h1>精通的语言：</h1>
        <input v-model="language" type="checkbox" value="Java"/>Java<br/>
        <input v-model="language" type="checkbox" value="PHP"/>PHP<br/>
        <input v-model="language" type="checkbox" value="Python"/>Python<br/>
        选中了：{{language.join(",")}}
    </div>
    <script>
        var vm = new Vue({
           el: "#app",
           data: {
                language: []
           }
        });
    </script>
</body>
</html>
```

- v-on指令

v-on指令，用于绑定事件，还可以简写为@事件名，例如v-on:click="",可以简写为@click=""。
v-on还可以用来防止事件冒泡。
v-on还能绑定键盘事件，做一些按键处理，以及一些按键的组合键。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-on</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <!-- v-on指令，用于绑定事件，还可以简写为@事件名，例如v-on:click="",可以简写为@click="" -->
        <!-- 事件中，可以直接写js片段 -->
        <input type="text" v-model="num">
        <!-- 如果事件处理比较复杂，还可以传一个方法名，该方法必须是Vue实例中的方法 -->
        <button v-on:click="num++">点赞</button>
        <button @click="cancel">取消点赞</button>
        <h1>有{{num}}个赞</h1>

        <!-- 防止事件冒泡 -->
        <!-- v-on:click.stop，表示停止事件冒泡，点小dev，大div不糊响应事件 -->
        <!-- v-on:click.prevent，表示阻止元素的默认行为，例如下面的a标签，点击超链接，不会跳转，在prevent后传入方法名，则指定点击后，做什么 -->
        <!-- 既想阻止默认行为，还想阻止冒泡，则连起来即可，@click.prevent.stop=”“ -->
        <!-- v-on:click.once，once，只可以点击一次 -->
        <div style="border: 1px solid red; padding: 20px;" v-on:click="hello">
            大div
            <div style="border: 1px solid blue; padding: 20px;" @click.stop="hello">
                小div
                <a href="http://www.baidu.com" @click.prevent.stop="hello">去百度</a>
            </div>
        </div>

        <!-- 按键修饰符 -->
        <!-- v-on.keyup.up，按键按下再弹起时，keyup后面跟绑定哪个按键，例如up，就是向上键 -->
        <!-- v-on.keyup.down，按键按下再弹起时，keyup后面跟down，就是向下键 -->
        <!-- 还可以监听组合键，例如按住ctrl键，再加点击，则将事件用.号连接即可，就是@click.ctrl -->
        <input type="text" v-model="num" v-on.keyup.up="num+=2" @keyup.down="num-=2" @click.ctrl="num=10">
    </div>
    <script>
        var vm = new Vue({
            el: "#app",//el绑定元素
            data: {//data，封装数据
                num: 1
            },
            methods: {//methods封装方法
                //取消点赞
                cancel() {
                    this.num --;
                },
                hello() {
                    alert("点击了");
                }
            }
        });
    </script>
</body>
</html>
```

- v-for指令

v-for指令，常用用于遍历数据到列表项。
v-for除了可以遍历数组，还可以遍历对象的属性列表。
v-for一般会指定一个key属性，key属性指向的数据属性必须是唯一的，例如id，或者index，用于提高效率。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-for</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <ul>
            <!-- key属性，写上，提高效率，一定要唯一，例如id或其他的属性，这里用name来做唯一 -->
            <!-- v-if，配合v-for，可以过滤出都是女的对象，v-if的优先级比v-for低，所以先v-for遍历，再用v-if判断 -->
            <li v-for="(user,index) in users" :key="user.name" v-if="user.gender == '女'">
                <!-- 1、显示user信息：v-for="item in items" -->
                {{index}} == {{user.name}} == {{user.gender}} == {{user.age}} <br>
                <!-- 2.获取数组下标：v-for="(item,index) in items" -->
                <!-- 3.遍历对象（属性值value在前，属性名key在后！）
                    v-for="value in object"
                    v-for="(value,key) in object"
                    v-for="(value,key,index) in object"
                 -->
                 对象信息：
                 <span v-for="(v,k,i) in user">{{k}} -- {{v}} -- {{i}}，</span>
                 <!-- 遍历的时候都加上：key来区分不同数据，提高vue渲染效率 -->
            </li>
        </ul>
        <ul>
            <!-- 如果是单纯的数据数组，没有一个位置，那么久拿索引index -->
            <li v-for="(num, index) in nums" :key="index">
                Item {{index}}：{{num}}
            </li>
        </ul>
    </div>
    <script>
        var vm = new Vue({
           el: "#app",
           data: {
                users: [
                    {name: "刘岩", gender: "女", age: 21},
                    {name: "张三", gender: "男", age: 18},
                    {name: "范冰冰", gender: "女", age: 24},
                    {name: "刘亦菲", gender: "女", age: 18},
                    {name: "古力娜扎", gender: "女", age: 25}
                ],
                nums: [1,2,3,4]
           }
        });
    </script>
</body>
</html>
```

- v-if和v-show指令

v-if和v-show都可以用来显示、隐藏元素。
v-if，是对DOM添加元素来显示，移除元素来做隐藏，由于会频繁更新DOM，所以不经常切换的元素，所以使用该标签性能比较好。
v-show，是对元素设置display属性来切换，设置display属性为none隐藏元素，去掉display属性为显示，经常切换的元素，使用该标签比较好。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-if和v-show</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <!-- 
        v-if：顾名思义，条件判断，当得到的结果为true时，所在元素才会被渲染（切换时，是对DOM进行移除和添加，所以不经常切换的，使用这个）
        v-show：当得到的结果为true时，所在的元素才会显示（切换显示和不显示，使用display属性来切换，经常切换的，则使用这个）
     -->
    <div id="app">
        <button @click="show = !show">点我呀</button>
        <!-- 使用v-if显示 -->
        <h1 v-if="show">if=看到我</h1>
        <!-- 使用v-show显示 -->
        <h1 v-show="show">show=看到我</h1>
    </div>
    <script>
        var vm = new Vue({
           el: "#app" ,
           data: {
                show: true
           }
        });
    </script>
</body>
</html>
```

- v-else和v-else-if指令

v-else和v-else-if指令，对应 else 和 else if 流程语句，一般配合v-if一起使用，就不多解释了

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-else和v-else-if</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <!-- v-if，相当于if判断 -->
    <!-- v-else-if，相当于else if判断 -->
    <!-- v-else，相当于else判断 -->
    <div id="app">
        <button v-on:click="random=Math.random()">点我呀</button>
        <span>{{random}}</span>
        <h1 v-if="random>=0.75">
            看到我啦！&gt;=0.75
        </h1>

        <h1 v-else-if="random>=0.5">
            看到我啦！&gt;=0.5
        </h1>

        <h1 v-else-if="random>=0.2">
            看到我啦！&gt;=0.2
        </h1>

        <h1 v-else>
            看到我啦！&lt;=0.2
        </h1>
    </div>
    <script>
        var vm = new Vue({
           el: "#app" ,
           data: {
                random: 1
           }
        });
    </script>
</body>
</html>
```

## 计算属性和侦听器

- 计算属性

某些结果是基于之前数据实时计算出来的，我们可以利用计算属性来完成。
例如购物车，商品数量的添加和减少，总价的显示需要商品数量和总价进行乘法计算才能得出结果。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>计算属性和侦听器</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <!-- 某些结果是基于之前数据实时计算出来的，我们可以利用计算属性来完成 -->
        <ul>
            <li>西游记; 价格：{{xyjPrice}}，数量：<input type="number" v-model="xyjNum"></li>
            <li>水浒传; 价格：{{shzPrice}}，数量：<input type="number" v-model="shzNum"></li>
            <li>总价：{{totalPrice}}</li>
            {{msg}}
        </ul>
    </div>
    <script>
        //监听器watch可以让我们监控一个值的变化，从而做出相应的反应
        var vm = new Vue({
           el: "#app" ,
           data: {
                xyjPrice: 99.98,
                shzPrice: 98,
                xyjNum: 1,
                shzNum: 1,
                msg: ""
           },
           computed: {//计算属性
            totalPrice: function() {
                return (this.xyjPrice * this.xyjNum) + (this.shzPrice * this.shzNum)
            }
           },
           watch: {
                //监听变量值的变化，属性名写要监听的属性名，值写为一个方法。方法回调传入2个值，分别是当前变化的新值，以及变化前的旧值
                xyjNum: function(newVal, oldVal) {
                    var info = "newVal" + "==>" + newVal + "，" + "oldVal" + "==>" + oldVal;
                    console.log(info);
                    if(newVal >= 3) {
                        this.msg = "库存超出限制"
                        this.xyjNum = 3
                    } else {
                        this.msg = ""
                    }
                }
           },
        });
    </script>
</body>
</html>
```

- 过滤器

过滤器常用来处理文本格式化的操作，过滤器可以用在两个地方：双花括号插值和v-bind表达式。
过滤器有全局过滤器和局部过滤器。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>过滤器</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <!-- 过滤器常用来处理文本格式化的操作，过滤器可以用在两个地方：双花括号插值和v-bind表达式 -->
        <ul>
            <li v-for="user in userList">
                <!-- 插值表达式中使用插值表达式可以，但是如果判断过程很复杂得时候，就不好了，我们使用过滤器 -->
                <!-- {{user.id}} == {{user.name}} == {{user.gender == 1 ? "男" : "女"}} -->
                <!-- 使用局部过滤器，|是管道符，格式为：属性值 | 过滤器 -->
                {{user.id}} == {{user.name}} == {{user.gender | genderFilter}}
                <!-- 使用全局过滤器 -->
                全局过滤器：{{user.gender | gFilter}}
            </li>
        </ul>
    </div>
    <script>
        //定义全局过滤器
        Vue.filter("gFilter", function(val) {
            if(val == 1) {
                       return "男"
                   } else {
                       return "女"
                   }
        })
        var vm = new Vue({
           el: "#app",
           data: {
               userList: [
                   {id:1, name: "jacky", gender: 1},
                   {id:2, name: "peter", gender: 0}
               ]
           },
           filters: {
               //定义过滤器，过滤器是一个方法，这里定义的过滤器都是局部的
               genderFilter(val) {
                   if(val == 1) {
                       return "男"
                   } else {
                       return "女"
                   }
               }
           }
        });
    </script>
</body>
</html>
```

## 组件化

组件化，就是自定义标签，提供属性绑定以及事件处理，提高代码复用率。
组件分为局部注册和全局注册。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>组件化</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <!-- 使用局部的自定义组件 -->
        <counter></counter>
        <!-- 使用全局的自定义组件 -->
        <button-counter></button-counter>
    </div>
    <script>
        //1、全局声明注册一个组件
        Vue.component("counter", {
            template: `<button @click="count++">我被点击了 {{count}} 次</button>`,
            //注意data，返回的是一个方法，里面返回一个对象，这样每个组件之间的数据都是隔离的
            data() {
                return  {
                    count: 1
                }
            }
        });
        //2、局部声明一个组件
        const buttonCounter = {
            template: `<button @click="count++">~~~~ 我被点击了 {{count}} 次 ~~~~</button>`,
            //注意data，返回的是一个方法，里面返回一个对象，这样每个组件之间的数据都是隔离的
            data() {
                return  {
                    count: 1
                }
            }
        };
        var vm = new Vue({
           el: "#app",
           data: {
                count: 1
           },
           components: {
                //注册局部组件，键值对方式注册
                'button-counter': buttonCounter
           }
        });
    </script>
</body>
</html>
```

## 生命周期钩子函数

Vue提供了一些生命周期钩子函数，让我们在每个阶段做一些事情。

- beforeCreate()，创建之前回调。
- created()，创建时回调
- beforeMount()，挂载前回调
- mounted()，挂载成功时回调
- beforeUpdate()，准备更新时回调
- updated()，更新完成时回调


```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>生命周期</title>
    <script src="../node_modules/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <span id="num">{{num}}</span>
        <button @click="num++">赞！</button>
        <h2>{{name}}，有{{num}}个人点赞</h2>
    </div>
    <script>
        var vm = new Vue({
           el: "#app",
           data() {
               return {
                    name: "张三",
                    num: 100
               }
           },
           methods: {
               show() {
                   return this.name
               },
               add() {
                   this.num++
               }
           },
           beforeCreate() {
               console.log("============= beforeCreate =============");
               console.log("数据模型未加载：" + this.name, this.num);
               console.log("方法未加载：" + this.show());
               console.log("html模板未加载：" + document.getElementById("num"));
           },
           created() {
               console.log("============= created =============");
               console.log("数据模型已加载：" + this.name, this.num);
               console.log("方法已加载：" + this.show());
               console.log("html模板已加载：" + document.getElementById("num"));
               console.log("html模板未渲染：" + document.getElementById("num").innerHTML);
           },
           beforeMount() {
               console.log("============= beforeMount =============");
               console.log("html模板未渲染：" + document.getElementById("num").innerHTML);
           },
           mounted() {
               console.log("============= mounted =============");
               console.log("html模板已渲染：" + document.getElementById("num").innerHTML);
           },
           beforeUpdate() {
               console.log("============= beforeUpdate =============");
               console.log("数据模型已更新：" + this.num);
               console.log("html模板未更新：" + document.getElementById("num").innerHTML);
           },
           updated() {
               console.log("============= updated =============");
               console.log("数据模型已更新：" + this.num);
               console.log("html模板已更新：" + document.getElementById("num").innerHTML);
           },
        });
    </script>
</body>
</html>
```