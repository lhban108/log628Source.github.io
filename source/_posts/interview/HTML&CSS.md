---
layout: js
title: HTMLCSS
tags: [HTMLCSS, interview]
---

1.手写图片瀑布流效果
2.使用CSS绘制几何图形（圆形、三角形、扇形、菱形等）
3.使用纯CSS实现曲线运动（贝塞尔曲线）
4.实现常用布局（三栏、圣杯、双飞翼、吸顶），可是说出多种方式并理解其优缺点

### 1 三栏布局

```HTML
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>test1</title>
    <style media="screen">
      .left {
        width: 300px;
        background-color: blue;
        height: 200px;
        text-align: center;
      }
      .center {
        background-color: #ccc;
        height: 200px;
        text-align: center;
      }
      .right {
        text-align: center;
        width: 300px;
        background-color: red;
        height: 200px;
      }
    </style>
  </head>
  <body>
    <!-- 浮动 -->
    <section class="sec1">
      <style media="screen">
        .sec1 .left {
          float: left;
        }
        .sec1 .right {
          float: right;
        }
      </style>
      <div class="left">left</div>
      <div class="right">right</div>
      <div class="center">浮动 ： center--center--center--</div>
    </section>
 
    <!-- 绝对定位 -->
    <section class="sec2">
      <style media="screen">
      .sec2 {
        margin-top: 20px;
        position: relative;
      }
      .sec2 > div {
        position: absolute;
      }
      .sec2 .left {
        left: 0;
      }
      .sec2 .right {
        right: 0;
      }
      .sec2 .center {
        left: 300px;
        right: 300px;
      }
      </style>
      <div class="left">left</div>
      <div class="center">绝对定位 ： center--center--center--</div>
      <div class="right">right</div>
    </section>
 
    <!-- flex布局 -->
    <section class="sec3">
      <style media="screen">
        .sec3 {
          margin-top: 240px;
          display: flex;
        }
        .sec3 .center {
          flex: 1;
        }
      </style>
      <div class="left">left</div>
      <div class="center">flex : center--center--center--</div>
      <div class="right">right</div>
    </section>
 
    <!-- 表格布局 -->
    <section class="sec4">
      <style media="screen">
        .sec4 {
          margin-top: 20px;
          display: table;
          width: 100%
        }
        .sec4 > div {
          display: table-cell;
        }
      </style>
      <div class="left">left</div>
      <div class="center">表格布局 : center--center--center--</div>
      <div class="right">right</div>
    </section>
 
    <!-- 网格布局 -->
    <!-- CSS3下一代标准 -->
    <section class="sec5">
      <style media="screen">
        .sec5 {
          margin-top: 20px;
          display: grid;
          width: 100%;
          /* grid-template-rows: 100px; */
          grid-template-columns: 300px auto 300px;
        }
      </style>
      <div class="left">left</div>
      <div class="center">网格布局 : center--center--center--</div>
      <div class="right">right</div>
    </section>
  </body>
</html>
<!--
1、以上各种布局的优缺点，以及比较推荐哪一种方案
  (1)浮动 :
    - 缺点 :浮动是脱离文档流的，有些时候需要清除浮动，需要很好的处理浮动周边元素的关系
    - 优点 :兼容性比较好
  (2)绝对定位 :
    - 缺点 :该布局脱离文档流，所以子元素也必须脱离文档流，因此可使用性比较差
    - 优点 :快捷，比较不容易出问题
  (3)flex :
    - 缺点 :兼容性比较差(css3的属性)，不兼容IE8及以下
    - 优点 :非常有效的解决了浮动和绝对定位的问题
  (4)表格布局 :
    - 缺点 :操作繁琐，当三栏中其中某一栏高度超出时，其他两栏的高度也会自动跟着调整(不符合某些场景)
    - 优点 :兼容性非常好，补缺了flex布局兼容的问题
  (5)网格布局 :
    - 新技术，CSS3下一代局部方案
2、如果去掉"高度已知"， 以上哪种方案同样适用？
  只有 flex布局 和 表格布局 同样适用 
 -->
 ```

### 2 水平垂直居中

（1）绝对定位+margin:auto

```CSS
	div {
		width: 200px;
		height: 200px;
		background: green;
		
		position: absolute;
		left: 0;
		top: 0;
		bottom: 0;
		right: 0;
		margin: auto;
	}
```

(2) 绝对定位+负margin

```CSS
	div{
		width: 200px;
		height: 200px;
		background: green;
		
		position: absolute;
		left: 50%;
		top: 50%;
		margin-left: -100px;
		margin-top: -100px;
	}
```

(3) 绝对定位+transform

```CSS
	div {
		width: 200px;
		height: 200px;
		background: green;
		
		position: absolute;
		left: 50%;    /* 定位父级的50% */
		top: 50%;
		transform: translate(-50%,-50%); /*自己的50% */
	}
```

(4) flex布局

```CSS
   .box {
         height:600px;  
         
         display:flex;
         justify-content:center;  /* 子元素水平居中 */
         align-items:center;      /* 子元素垂直居中 */
         /* 只要三句话就可以实现不定宽高水平垂直居中 */
    }
    .box>div{
        background: green;
        width: 200px;
        height: 200px;
    }
```

(5) table-cell实现居中

```CSS
  .box {
      display: table-cell;
		text-align: center;
		vertical-align: middle;
  }
```

### 3 BFC

三种文档流的定位方案：

常规流(Normal flow)

- 在常规流中，盒一个接着一个排列;
- 在块级格式化上下文里面， 它们竖着排列；
- 在行内格式化上下文里面， 它们横着排列;
- 当position为static或relative，并且float为none时会触发常规流；
- 对于静态定位(static positioning)，position: static，盒的位置是常规流布局里的位置；
- 对于相对定位(relative positioning)，position: relative，盒偏移位置由top、bottom、left、right属性定义。即使有偏移，仍然保留原有的位置，其它常规流不能占用这个位置。

浮动(Floats)

- 左浮动元素尽量靠左、靠上，右浮动同理
- 这导致常规流环绕在它的周边，除非设置 clear 属性
- 浮动元素不会影响块级元素的布局
- 但浮动元素会影响行内元素的布局，让其围绕在自己周围，撑大父级元素，从而间接影响块级元素布局
- 最高点不会超过当前行的最高点、它前面的浮动元素的最高点
- 不超过它的包含块，除非元素本身已经比包含块更宽
- 行内元素出现在左浮动元素的右边和右浮动元素的左边，左浮动元素的左边和右浮动元素的右边是不会摆放浮动元素的

绝对定位(Absolute positioning)

- 绝对定位方案，盒从常规流中被移除，不影响常规流的布局；
- 它的定位相对于它的包含块，相关CSS属性：top、bottom、left、right；
- 如果元素的属性position为absolute或fixed，它是绝对定位元素；
- 对于position: absolute，元素定位将相对于上级元素中最近的一个relative、fixed、absolute，如果没有则相对于body；

BFC触发方式：

- 根元素，即HTML标签
- 浮动元素：float值为left、right
- overflow值不为 visible，为 auto、scroll、hidden
- display值为 inline-block、table-cell、table-caption、table、inline-table、flex、inline-flex、grid、inline-grid
- 定位元素：position值为 absolute、fixed

浏览器对BFC区域的约束规则：

- 内部的Box会在垂直方向上一个接一个的放置
- 内部的Box垂直方向上的距离由margin决定。（完整的说法是：属于同一个BFC的两个相邻Box的margin会发生折叠，不同BFC不会发生折叠。）
- 每个元素的左外边距与包含块的左边界相接触（从左向右），即使浮动元素也是如此。（这说明BFC中子元素不会超出他的包含块，而position为absolute的元素可以超出他的包含块边界）
- BFC的区域不会与float的元素区域重叠
- 计算BFC的高度时，浮动子元素也参与计算

作用：

- BFC是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素
- 阻止元素被浮动元素覆盖
  一个正常文档流的block元素可能被一个float元素覆盖，挤占正常文档流，因此可以设置一个元素的float、display、position值等方式触发BFC，以阻止被浮动盒子覆盖。
- 可以包含浮动元素
  通过改变包含浮动子元素的父盒子的属性值，触发BFC，以此来包含子元素的浮动盒子。
- 阻止因为浏览器因为四舍五入造成的多列布局换行的情况
  有时候因为多列布局采用小数点位的width导致因为浏览器因为四舍五入造成的换行的情况，可以在最后一列触发BFC的形式来阻止换行的发生。
- 阻止相邻元素的margin合并
  属于同一个BFC的两个相邻块级子元素的上下margin会发生重叠，所以当两个相邻块级子元素分属于不同的BFC时可以阻止margin重叠

BFC、IFC、GFC、FFC：

- BFC BFC(Block Formatting Contexts)直译为"块级格式化上下文"。Block Formatting Contexts就是页面上的一个隔离的渲染区域，容器里面的子元素不会在布局上影响到外面的元素，反之也是如此。如何产生BFC？ float的值不为none。  overflow的值不为visible。  position的值不为relative和static。 display的值为table-cell,table-caption, inline-block中的任何一个。  那BFC一般有什么用呢？比如常见的多栏布局，结合块级别元素浮动，里面的元素则是在一个相对隔离的环境里运行。 
- IFC IFC(Inline Formatting Contexts)直译为"内联格式化上下文"，IFC的line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响) IFC中的line box一般左右都贴紧整个IFC，但是会因为float元素而扰乱。float元素会位于IFC与与line box之间，使得line box宽度缩短。 同个ifc下的多个line box高度会不同。
IFC中时不可能有块级元素的，当插入块级元素时（如p中插入div）会产生两个匿名块与div分隔开，即产生两个IFC，每个IFC对外表现为块级元素，与div垂直排列。 那么IFC一般有什么用呢？ 水平居中：当一个块要在环境中水平居中时，设置其为inline-block则会在外层产生IFC，通过text-align则可以使其水平居中。 垂直居中：创建一个IFC，用其中一个元素撑开父元素的高度，然后设置其vertical-align:middle，其他行内元素则可以在此父元素下垂直居中。
- GFC GFC(GridLayout Formatting Contexts)直译为"网格布局格式化上下文"，当为一个元素设置display值为grid的时候，此元素将会获得一个独立的渲染区域，我们可以通过在网格容器（grid container）上定义网格定义行（grid definition rows）和网格定义列（grid definition columns）属性各在网格项目（grid item）上定义网格行（grid row）和网格列（grid
columns）为每一个网格项目（grid item）定义位置和空间。  那么GFC有什么用呢，和table又有什么区别呢？首先同样是一个二维的表格，但GridLayout会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制。 
- FFC FFC(Flex Formatting Contexts)直译为"自适应格式化上下文"，display值为flex或者inline-flex的元素将会生成自适应容器（flex container），可惜这个牛逼的属性只有谷歌和火狐支持，不过在移动端也足够了，至少safari和chrome还是OK的，毕竟这俩在移动端才是王道。 Flex Box 由伸缩容器和伸缩项目组成。通过设置元素的 display 属性为 flex 或 inline-flex
可以得到一个伸缩容器。设置为 flex 的容器被渲染为一个块级元素，而设置为 inline-flex 的容器则渲染为一个行内元素。 伸缩容器中的每一个子元素都是一个伸缩项目。伸缩项目可以是任意数量的。伸缩容器外和伸缩项目内的一切元素都不受影响。简单地说，Flexbox 定义了伸缩容器内伸缩项目该如何布局。

### 4 选择器和权重

选择器分类：

- id选择器 #id { }
- 类选择器 .link { }
- 元素选择器 a { }
- 属性选择器  [ type = radio ] { }
- 伪元素选择器  ::before { }
- 伪类选择器  :hover { }
- 否定选择器 :not ( .link ) { }
- 通用选择器  * { }
- 组合选择器  [ type = radio] + babel { }

同样的方式权重计算规则：

- id选择器 #id { }    +100
- 类/属性/伪类       +10
- 元素/伪元素        +1
- 其他选择器         +0

不同方式的权重计算规则：

- !important 优先级最高
- 元素属性的内联样式 优先级高
- 相同权重 后写的生效

### 5 超过省略号

```CSS
	div {
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
	}
```

```CSS
	div {
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-line-clamp: 2;  /* 控制多行的行数 */
		/* ! autoprefixer: off */
		-webkit-box-orient: vertical;
		/* autoprefixer: on */
	}
```

### 6 上中下三栏布局

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>headerBodyFooter</title>
	<style>
		body {
			margin: 0;
			padding: 0;
		}
		.content {
			min-height: 100vh;
			display: flex;
			flex-direction: column;
			text-align: center;
		}
		.head {
			height: 100px;
			position: fixed;
			width: 100%;
    		top: 0;
			background-color: cadetblue;
		}
		.center {
			flex-grow: 1;
			margin-top: 100px;
			background-color: mediumspringgreen;
		}
		.footer {
			height: 100px;
			background-color: gold;
		}
		.center-1 {
			height: 200px;
		}
		.center-2 {
			height: 900px;
		}
	</style>
</head>
<body>
	<div class="content">
		<div class="head">head</div>
		<div class="center">center
			<div class="center-1">1</div>
			<div class="center-2">2</div>
		</div>
		<div class="footer">footer</div>
	</div>
</body>
</html>
```
