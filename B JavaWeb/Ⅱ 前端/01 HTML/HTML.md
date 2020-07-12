# HTML

## 1 HTML简介

HTML（HeyperText MarkUp Language）全称称之为超文本标记语言，它是世界上最简单的语言。

在开发的时候我们只需要进行页面布局（利用标签:element）

注意：超文本标记语言（HTML）又称之为web(开发)，它诞生(1993~2019)这门语言大大小小经历过变化有五次，最近一次2014称之为HTML5（超文本标记语言第五次重大变化）

## 2 开发工具

Sublime： http://www.sublimetext.com/

中文网：https://sublimetextcn.com/

VScode：https://code.visualstudio.com/

Webstorm：http://www.jetbrains.com/webstorm/

## 3 HTML标签

HTML标记语言开发的程序的时候，利用就是标签（element）进行布局页面。

HTML（超文本标记语言）,它的页面（静态页面）是由标签组成，最终要一部分就是块元素；

### 3.1 常用块元素

#### 3.1.1 静态页面骨架

Sublime快捷键

1. 输入html:5，再按下tab键

2. 输入 !，再按下tab键

```html
<!DOCTYPE html> //这是超文本标记语言第五次重大变化文档声明方式
<html lang="en">
<head>
	<meta charset="UTF-8"> //chartset:设置字符集
	<title>Document</title>
</head>
<body>
	
</body>
</html>
```

- HTML标签是整个网页根元素（进行页面布局的其他元素：都是嵌套在HTML标签里面作为子元素）
- HTML标签右侧有一个lang属性（前端当中管这种写法叫做属性），属性值en(英文简写：代表的是英文下开发)

![image-20200713005028317](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200713005036.png)

### 3.1.2 常用的块元素

在web开发中块元素非常常用：块(block)元素是独占一行,在页面中是由上到下进行排列。

常用块元素有很多：h1~h6(标题)、div、p、ul、li.....等等

注意：在书写学习这些标签（块元素），一定要注意在body标签内部进行书写。

标签小技巧：打出标签名（ctrl + E）标签自动补齐；

运行方式：鼠标右键+赋值路径（在浏览器中运行）