#### HTML之语义标签学习笔记

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

### HTML语义标签

本篇，我们来学习HTML的语义标签，标签的语义话，是什么意思呢？HTML5的语义话目的是让非IT人士能直观地读懂标签和标签内的属性，更重要是语义化标签能让搜索引擎、爬虫、SEO读懂我们的页面。例如使用文字朗读软件读取我们的HTML时，如果出现strong标签，strong标签内的文字就会被重读。

- blockquote标签，表示文本的引用，会将文本左、右两侧进行缩进，cite表示引用的地址

```
<blockquote cite="https://blog.csdn.net/lfdfhl/article/details/78320428">HTML不是程序设计语言，而是一种标记语言，它用一些标记、标签来说明文本的显示效果。要制作网页和建立网站，就必须对HTML语言有所了解</blockquote>
```

- cite其实还是一个标签，cite标签表示参考某个文献的引用

```
这段话出自<cite>Android艺术探索</cite>
```

- address标签，表示地址，展现为斜体

```
<address>广东省广州市天河区</address>
```

- code标签表示代码段

```
<code>System.out.println("hello world")</code>
```

- var标签表示变量

```
<var>public static final BASE_URL = "xxx"</var>
```

- dfn标签，用于表示专业术语，对应英文为defining instance

```
<dfn>量子网络通信</dfn>
```

- del标签，表示已删除，会在文字中间添加一条删除线

```
<del>该方法已被废弃</del>
```

- pre标签，表示预先格式化，对应英文为preformatted，标签内的空格、回车、换行都会被保留

```
<pre>
		<p>       我是段落</p>


		<p>我是段落2</p>
</pre>
```

- mark标签，表示重点内容，会以荧光笔的黄色进行标记

```
<mark>数据结构和算法为本次考试首要考察的</mark>
```

- details标签表示详细信息，summary标签表示摘要信息，一般这2个标签会结合起来使用

```
<details>
    Android艺术探索
    <summary>
        这本书为安卓开发进阶必备书籍，值得一看
    </summary>
</details>
```