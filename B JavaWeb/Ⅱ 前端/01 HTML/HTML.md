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

HTML（超文本标记语言），它的页面（静态页面）是由标签组成，最终要一部分就是块元素；

### 3.1 常用块元素

#### 3.1.1 静态页面骨架

首先右下键选择html

![image-20200825224050350](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200825224050350.png)

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

#### 3.1.2 常用的块元素

在web开发中块元素非常常用：块(block)元素是独占一行，在页面中是由上到下进行排列。

常用块元素有很多：h1~h6(标题)、div、p(paragraph)、ul、li.....等等

注意：在书写学习这些标签（块元素），一定要注意在body标签内部进行书写。

标签小技巧：打出标签名（ctrl + E）标签自动补齐；

运行方式：鼠标右键+赋值路径（在浏览器中运行）

![image-20200717003925882](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200717003925882.png)

#### 3.1.3 常用的块元素-列表系列

概述：列表（也是块元素）有两个这两个块元素用来显示列表

- 无序列表ul
- 有序列表ol

而li标签经常作为他们的子元素一起使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<!-- 无序列表 -->
	<ul>
		<li>吃饭</li>
		<li>睡觉</li>
		<li>打豆豆</li>
	</ul>
	<!-- 有序列表 -->
	<ol>
		<li>学习</li>
		<li>喝酒</li>
		<li>烫头</li>
	</ol>
</body>
</html>
```

![image-20200825224900432](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200825224900432.png)

总结：

```
h1~h6（不同级别标题
div（盒子标签）
p（段落）
ul（无序列表）
ol（有序列表）
```

### 3.2 行内元素

概述：前端开发中行内元素（内联元素）

行内元素特征：不是独占一行，从左到右进行排列

常用的行内元素有：span、img、a

注意：在web领域中标签（双闭合标签）、单闭合标签

- a标签href属性是用来设置超链接的地址属性

- img标签的src属性是用来设置显示图片路径的（图片名称） 图片没有情况下默认显示文字

  ```html
  <img src="" alt="默认文字">
  ```

  ![image-20200825230421353](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200825230421353.png)    

### 3.3 表单元素

前端中比较重要的标签：标元素（表单元素经常用来收集用户输入信息），将用户输入信息提交给服务器。

表单元素即为input标签（单闭合标签），这个标签经常集合form标签一起使用。

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<form>
		<!-- 表单元素即为input标签 -->
		<p>同户名：<input type="text" name=""></p>
		<p>密&nbsp;&nbsp;&nbsp;码：<input type="password" name=""></p>
		<p>日历<input type="date" name=""></p>
		<p>你喜欢的颜色：<input type="color" name=""></p>
		<p>你的身高：<input type="range" name=""></p>
		<p>
			<!-- 多选按钮 -->
			你喜欢的食物：
			<input type="checkbox" name="">烤鸭
			<input type="checkbox" name="">汉堡
		</p>
		<p>
			<!-- 单选按钮，必须要有name，且一致 -->
			你喜欢的国家：
			<input type="radio" name="A">中国
			<input type="radio" name="A">美国
			<input type="radio" name="A">日本
			<input type="radio" name="A">韩国
		</p>
		<p><input type="submit" name=""></p>
	</form>
</body>
</html>
```

![image-20200825235831437](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200825235831437.png)