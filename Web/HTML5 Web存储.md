#### HTML5 Web存储

本篇是学习[菜鸟教程HTML5 Web存储篇](https://www.runoob.com/html/html5-webstorage.html)的学习笔记。

HTML5之前，Web端进行本地存储数据一般使用的是Cookie，而HTML5后可以使用localStorage、sessionStorage来存储本地数据。

#### 浏览器支持情况

Internet Explorer 8+, Firefox, Opera, Chrome, 和 Safari支持Web 存储。（Internet Explorer 7 及更早IE版本不支持web 存储）

#### localStorage和sessionStorage

Web端本地存储数据，可以使用localStorage和sessionStorage，他们的区别如下：

- localStorage，长久保存数据到本地，没有过期时间，直到手动删除。
- sessionStorage，临时数据保存在同一个窗口或标签页，当关闭窗口或标签页，则删除保存的数据。

检查是否支持Web存储

```
if(typeof(Storage)!=="undefined")
{
    //支持Web存储，可以进行增删查改
} else {
    //当前浏览器，不支持Web存储
}
```

localStorage和sessionStorage的API是一致的，所以只要会使用localStorage，sessionStorage就会了。

#### localStorage使用

使用数据存储，莫过于增、删、查、改，其中增和改的API是一致的，改是增加时如果已经存在则覆盖数据。

- 保存数据（增）或 修改数据（改）

```
//第一种，直接将key名作为属性名来使用
localStorage.myName = "Wally";
//第二种，调用setItem(key, value)
localStorage.setItem("myName", "Barry");
```

- 删除数据（删）

```
//删除单个数据
localStorage.removeItem("myName");
//移除所有数据
localStorage.clear();
```

- 查询数据（查）

```
//获取一条数据
var value = localStorage.getItem("myName");
```

其他辅助API

- 获取key在存储中的索引

```
var index = localStorage.key(0);
```

- 获取保存的数据个数

```
var length = localStorage.length;
```

- 持久化保存点击数例子

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>localStorage 例子</title>
    <script>
        /**
         * 计数方法
         */
        function clickCounter() {
            if(typeof(Storage) != 'undefined') {
                //获取原来的计数，如果存在，则+1
                if(localStorage.clickCount) {
                    var currentCount = Number(localStorage.clickCount);
                    localStorage.clickCount = currentCount + 1;
                } else {
                    localStorage.clickCount = 1;
                }
                document.getElementById("result").innerHTML = "您已经点击了按钮" + localStorage.clickCount + "次";
            } else {
                document.getElementById("result").innerHTML = "您的浏览器不支持Web存储";
            }
        }
    </script>
</head>
<body>
    <p><button onclick="clickCounter()" type="button">点我计数</button></p>
    <div id="result"></div>
    <p>点击该按钮查看计数器的增加。</p>
    <p>关闭浏览器选项卡(或窗口),重新打开此页面,计数器将继续计数(不是重置)。</p>
</body>
</html>
```

#### sessionStorage使用

由于sessionStorage的API和localStorage一致，就不再赘述了。

- 窗口、标签页打开期间保存点击数

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>sessionStorage 例子</title>
    <script>
        /**
         * 计数方法
         */
        function clickCounter() {
            if(typeof(Storage) != 'undefined') {
                //获取原来的计数，如果存在，则+1
                if(sessionStorage.clickCount) {
                    var currentCount = Number(sessionStorage.clickCount);
                    sessionStorage.clickCount = currentCount + 1;
                } else {
                    sessionStorage.clickCount = 1;
                }
                document.getElementById("result").innerHTML = "您已经点击了按钮" + sessionStorage.clickCount + "次";
            } else {
                document.getElementById("result").innerHTML = "您的浏览器不支持Web存储";
            }
        }
    </script>
</head>
<body>
    <p><button onclick="clickCounter()" type="button">点我计数</button></p>
    <div id="result"></div>
    <p>点击该按钮查看计数器的增加。</p>
    <p>关闭浏览器选项卡(或窗口),重新打开此页面,计数器将重置。</p>
</body>
</html>
```

#### 保存网址到本地例子

1. 输入key名，网站名、网址，点击按钮保存。
2. 提供输入框，输入key名，查询出网站名、网址。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Websit 网站保存例子</title>
</head>
<body>
    <label>别名（key）：</label>
    <input type="text" id="keyName" class="text">
    <br/>
    <label>网站名：</label>
    <input type="text" id="siteName" class="text">
    <br/>
    <label>网址：</label>
    <input type="text" id="siteUrl" class="text">
    <br/>
    <button onclick="save()">新增记录：</button>
    <hr/>
    <label>输入别名(key)：</label>
    <input type="text" id="searchSiteInput">
    <button id="searchSite" onclick="find()">查找网站</button>
    <br/>
    <div id="result"></div>

    <script>
        /**
         * 保存数据到本地
         */
        function save() {
            var siteModel = new Object();
            siteModel.keyName = document.getElementById("keyName").value;
            siteModel.siteName = document.getElementById("siteName").value;
            siteModel.siteUrl = document.getElementById("siteUrl").value;
            //将模型转为json并保存到本地
            var json = JSON.stringify(siteModel);
            localStorage.setItem(siteModel.keyName, json);
            alert("保存成功");
        }

        /**
         * 读取配置
         */
        function find() {
            var searchSiteKeyName = document.getElementById("searchSiteInput").value;
            //查询本地存储，value值为模型转换出来的json
            var json = localStorage.getItem(searchSiteKeyName);
            //解析json为model
            var siteModel = JSON.parse(json);
            var resultDiv = document.getElementById("result");
            resultDiv.innerHTML = "网站名：" + siteModel.siteName + " 网址：" + siteModel.siteUrl;
        }
    </script>
</body>
</html>
```