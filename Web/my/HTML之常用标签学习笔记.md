#### HTML之常用标签学习笔记

本文按着CSDN博主，谷歌的小弟的[讲给Android程序员看的前端教程](https://blog.csdn.net/lfdfhl/column/info/17220)系列博客进行学习，本系列作为个人的学习笔记，感谢博主写的系列博客~

本系列笔记按照博主提供的顺序记录~

> - HTML常用标签
> - HTML文本标签
> - HTML语义标签
> - HTML结构标签
> - HTML列表标签
> - HTML表格标签
> - HTML表单标签
> - HTML新增标签和API

### HTML常用标签

本篇介绍HTML中的常用标签，一起来学习一下吧~

- p标签，段落标签，对应英文paragraph。

```
<p>今天是乔布斯发布iPhone4s纪念日,balabalabalabalabalabalabalabala</p>
```

- h标签，标题标签，对应英文header，h标签一共有6种，从h1到h6。数字越大，文字越小，所以h1最大，h6最小。

```
<h1>我是h1</h1>
<h2>我是h2</h2>
<h3>我是h3</h3>
<h4>我是h4</h4>
<h5>我是h5</h5>
<h6>我是h6</h6>
```

- hr标签，水平分割线标签，对应英文Horizontal Rule。

```
<hr>
```

- br标签，表示换行，对应英文break。

```
今天<br>天气真好
```

- nobr标签，表示不换行，什么时候用呢？例如需要显示一个数学公式，但它很长，换行显示会引起歧义等，这时候就需要用到nobr标签。

```
<nobr>我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...</nobr>
```

- center标签，能将文字行内横向居中显示。

```
<center>我在中间显示喔</center>
```

- marquee标签，跑马灯，一行文字太长，需要跑马灯滚动的效果时，就可以使用该标签。

```
<marquee behavior="scroll" direction="left">
    <p>我是marquee标签中的文字</p>
</marquee>
```

- button标签，按钮，这里我们让一个抽奖按钮，点击时alert弹出一个弹窗提示用户抽奖已结束。

```
<head>
	...
    <title>HTML常用标签</title>
    <script>
        function onClick() {
            alert("抱歉，抽奖已结束，下次再来吧");
        }
    </script>
    
    ...
</head>

<button onclick="onClick()">点我抽奖</button>
```

- a标签，超链接标签，href属性填入跳转地址，title为鼠标悬停在该超链接时显示浮窗的提示文字，target为点击跳转的方式，有2个取值，_blank为新开一个窗口进行跳转，_self则为当前页面跳转。

```
<a href="https://www.baidu.com" title="点我去百度喔" target="_blank">点我去百度</a>
```

- img标签，图片标签，对应英文image。

```
<img src="./img/js.jpeg">
```

#### 完整代码

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>HTML常用标签</title>
    <script>
        function onClick() {
            alert("抱歉，抽奖已结束，下次再来吧")
        }
    </script>
</head>
<body>
<!-- 段落 -->
<p>今天是乔布斯发布iPhone4s纪念日,balabalabalabalabalabalabalabala</p>

<!-- 号数越大，文字越小 -->
<h1>我是h1</h1>
<h2>我是h2</h2>
<h3>我是h3</h3>
<h4>我是h4</h4>
<h5>我是h5</h5>
<h6>我是h6</h6>

<!-- 水平横线 -->
<hr>

<!-- 换行 -->
今天<br>天气真好

<nobr>我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...我是数学公式，不能换行喔...</nobr>

<!-- 让文字中间显示 -->
<center>我在中间显示喔</center>

<!-- 跑马灯，从右到左 -->
<marquee behavior="scroll" direction="left">
    <p>我是marquee标签中的文字</p>
</marquee>

<!-- 按钮 -->
<button onclick="onClick()">点我抽奖</button>

<!-- 超链接 -->
<a href="https://www.baidu.com" title="点我去百度喔" target="_blank">点我去百度</a>

<!-- img标签 -->
<img src="./img/js.jpeg">
</body>
</html>
```