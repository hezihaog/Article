#### HTML之文本标签学习笔记

本文按着CSDN博主，谷歌的小弟的讲给[Android程序员看的前端教程](https://blog.csdn.net/lfdfhl/column/info/17220)系列博客进行学习，本系列作为个人的学习笔记，感谢博主写的系列博客~

本系列笔记按照博主提供的顺序记录~

> - HTML常用标签
> - HTML文本标签
> - HTML语义标签
> - HTML结构标签
> - HTML列表标签
> - HTML表格标签
> - HTML表单标签
> - HTML新增标签和API

### HTML文本标签
本篇介绍，HTML中的文本标签，就是用来显示文本的，并且能对文本做一些样式上的处理。

- b标签，表示粗体，能让标签内的文字展示为粗体。

```
<b>我是b标签内的文字，我会被展示成粗体</b>
```

- small标签，用于显示小号字体，例如版权信息、法律信息和免责信息等

```	
<small>我是small标签内的文字，你看我是不是比较小</small>
```

- i标签，对应英文italic，会让文字斜体显示

```
<i>我是不是变斜了?</i>
```

- em标签，对应英文为emphasize，和i标签一样，都会让文字以斜体显示，但在HTML5种em标签更加具有语义

```
<em>我也变斜了！</em>
```

- 上标，例如4的2次方

```
4的二次方4<sup>2</sup>
```

- 下标，例如双氧水化学式

```
H<sub>2</sub>O<sub>2</sub>
```

- span标签，用于行内文字样式

```
<span style="color:#FA0">皇上万岁万岁万万岁</span><span style="color:#F00">总卿家平身</span>
```

- font标签，用于给文本设置文字颜色和文字大小， 但在HTML5中不建议使用了，应该用CSS来实现

```
<font color="red" size="18">我是font中的文字</font>
```

#### 解惑时间

上面的b标签和strong标签功能一致，i标签和em标签页一样功能一致，那么为什么要搞2套功能一模一样的标签呢？

其实b和i标签为物理元素，它是告诉浏览器，b标签内的文字要显示为粗体，i标签内的文字显示为斜体，就没有其他作用了。而strong、em标签是逻辑元素，它不但能起到加粗或斜体的作用，还起到了强调的作用，例如在语音阅读器中，读取时如果读到了strong标签就会重读，相对于搜索引擎、爬虫、SEO而言，则会更加重视。

#### 完整代码

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>HTML文本标签</title>
</head>
<body>
<!-- b标签，表示粗体，能让标签内的文字展示为粗体 -->
<b>我是b标签内的文字，我会被展示成粗体</b>

<br/>

<!-- strong标签，作用和b标签效果相同，都是展示位粗体，但是HTML5为strong标签增强了语义 -->
<strong>我是strong标签内的文字，我也会展示为粗体喔</strong>

<br>

<!-- small标签，用于显示小号字体，例如版权信息、法律信息和免责信息等 -->
<small>我是small标签内的文字，你看我是不是比较小</small>

<br>

<!-- i标签，对应英文italic，会让文字斜体显示 -->
<i>我是不是变斜了?</i>

<br>

<!-- em标签，对应英文为emphasize，和i标签一样，都会让文字以斜体显示，但在HTML5种em标签更加具有语义 -->
<em>我也变斜了！</em>

<br>

<!-- 下划线，对应英文underline -->
<u>我是u标签中的文字喔，我有下划线加持</u>

<br>

<!-- 上标，例如4的2次方 -->
4的二次方4<sup>2</sup>

<br>

<!-- 下标，例如双氧水化学式 -->
H<sub>2</sub>O<sub>2</sub>

<br>

<!-- span标签，用于行内文字样式 -->
<span style="color:#FA0">皇上万岁万岁万万岁</span><span style="color:#F00">总卿家平身</span>

<br>

<!-- font标签，用于给文本设置文字颜色和文字大小， 但在HTML5中不建议使用了，应该用CSS来实现-->
<font color="red" size="18">我是font中的文字</font>

<br>
</body>
</html>
```