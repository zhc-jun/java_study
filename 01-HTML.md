# HTML

**参考手册**：https://www.runoob.com/tags/html-reference.html

**网页的介绍**

- 当我们打开一个网站，首先映入眼帘的就是一个个华丽多彩的网页 
- 这些网页，不仅呈现着基本的内容，还具备优雅的布局和丰富的动态效果
- 多个网页汇聚在一起，就构成一个网站。

### 今日重点

- 所有案例编写一遍
- 课上讲的**标签和属性**，多记记，背不下来，要学会查阅文档。

### 1. HTML 快速入门

- 网页的构成

  > HTML：用来制作网页基础内容和基本结构 
  >
  > CSS：用来制作网页美化效果
  >
  > JavaScript：用来制作网页动态效果

- html  css javascript 号称 前端技术三剑客，是学习前端程序员必会的技能。



#### 1.1 HTML 的介绍

- HTML(HyperText Markup Language)：超文本标记语言 
- 超文本：比普通文本更强大 , 同时也不局限于文字，图片，视频，脚本文件.......
- 标记：就是标签。可以使用一系列的标签，将网络上的文档格式统一，使分散的资源连接为一个逻辑整体 
- 2014年10月，万维网联盟宣布，经过8年时间的艰苦努力，HTML5 的规范终于制作完成

<起始标签 属性名称='属性值' ...> 内容 </结束标签>

HTML超文本标记语言（写网页），有标签以及标签的属性和内容组成的。通过浏览器打开并展示！！



#### 1.2 HTML 的组成

- HTML 页面是由一系列的元素(Element)组成的，而元素是通过标签创建出来的, 说白了元素指的就是**标签**。

- 标签特点

  > 左右尖括号 : <>,  括号内就是标签的名字：<span>  <title>

- 标签的分类

  > 1）**围堵标签**：成对出现的标签， <h1></h1>  左侧为开始标签，右侧为结束标签, 不难发现结束标签比开始标签对了一个 / 
  >
  > 2）**自闭和标签**：非成对标签，<hr /> <br />
  >
  > 不难发现自闭和标签，只有一个标签。

- 标签的属性 

- 标签中还可以拥有属性，属性可以为标签提供更多的功能。 

- 属性只能添加到开始标签中。格式为：属性名=属性值 

  > 例如：align 属性，代表对齐方式。我们可以在开始标签中添加该属性，就能让内容在不同位置显示了,
  >
  > "<p align='center'>我是一个段文字</p>"

<font color=red>注意：无论是标签还是属性，都是由HTML官方提供的，我们不可以随意变更，用时查阅文档即可。</font>



#### 1.3 HTML 入门案例

- 实现步骤 

  > ① 在项目下的 web 目录中新建一个 HTML 文件。 
  >
  > ② 修改<title>标签中的内容为：01-入门案例。 
  >
  > ③ 在<body>标签中编写一个<h1>标签，内容为：这是我的第一个HTML入门案例。 
  >
  > ④ 在<h1>标签中指定属性align，属性值为center。 
  >
  > ⑤ 通过浏览器打开查看页面。

**示例代码**

```html
<!DOCTYPE html>  <!--H5的文档声明-->
<html lang="en"> <!--根标签，一个文件中只能有一个根标签 -->
<head> <!--头部标签 -->
    <meta charset="UTF-8">  <!--字符编码-->
    <title>01-入门案例</title>  <!--头部的标题标签-->
</head>

<body> <!--身体标签：主要展示内容区域-->

    <!--标题标签区别于title, 它展示在body区域-->
    <!--align 学名对齐方式属性, 它的三个值：center 居中, left 左, right右-->
    <h1 align="center">国内首家培训机构上线啦~~~</h1>
    <h1 align="center">讲师教学、助教辅导、班主任管理生活~~~</h1>
    <h1 align="center">服务到位！包您满意~~~</h1>
</body>
</html>
```



#### 1.4 HTML 概念小结

- idea设置网页用谷歌打开：https://blog.csdn.net/qq_36850813/article/details/95483690

- HTML 是一种标记语言，使用元素和属性来编写页面 

- 组成部分 

  > 元素(Element)：开始标签、结束标签与内容相结合，便是一个完整的元素 
  >
  > ​      开始标签(Opening tag)：包含元素的名称，被左、右角尖括号所包围。表示元素从这里开始或者开始起作用 
  >
  > ​      结束标签(Closing tag)：与开始标签相似，只是其在元素名之前包含了一个斜杠。这表示着元素的结尾 
  >
  > 
  >
  > 内容(Content)：元素的内容，本例中就是所输入的文本本身 
  >
  > 
  >
  > 属性(Attribute)：标签的附加信息 
  >
  > 

- 学习 HTML 要抓住两个重点：

  > 1) 掌握标签所代表的含义 
  >
  > 2) 掌握在标签中属性的含义



### 2. HTML 基本语法

#### 2.1 HTML 的注释

- idea快捷键:  ctrl + /   

- 注释的格式  <!--注释的内容 --> 
- 注释的特点： 1）被注释掉的标签，不会被浏览器解析



#### 2.2 HTML 的标签

- 标签的分类 : 

  1）开始和结束标签 <h1></h1>   

  2）自闭和标签  横线 <hr/>   换行 <br/>

   

- 标签的嵌套: 

  ```
  标签可以嵌套：
      * 需要正确嵌套，不能你中有我，我中有你
      正确的嵌套格式：<h1><u>文本</u></h1> 
      错误的嵌套格式：<h1><u>文本</h1></u> 
  ```

  

-  块级元素和行内元素 

  1）块级元素：在页面中以块的形式展现，自己独占一行，后面的内容会自动换行。 <p>  <hr>  <div> 

  2）行内元素：在页面中以行的形式展现，不会换行。 <b>  <i>  <u>   <span> 

- div和span

  1）<div>：是一个通用的内容容器，没有特殊语义。一般用来对其它元素进行分组，用于样式化相关的需求。 

  2）<span>：是一个通用的行内容器，没有特殊语义。一般被用来编织元素以达到某种样式。 

  3）<font color=red> <div>和<span>标签核心作用是布局页面</font>

#### 2.3 HTML 的属性

- 官方提供，部分属性，所有标签都可以用，例如：name  id  等属性, 还有一部分属性为一些标签特有属性，具体要参考文档。

- 属性的作用: 

  1） 属性可以提供一些额外的信息，这些信息不会直接显示在内容中。但可以改变标签的样式或提供数据使用 

- 定义格式： 属性名=属性值 

- 属性的规范 ：

  1）同一个标签中属性的名称必须唯一

  2）不区分大小写，**建议使用小写**

  3）属性值可以使用单引号或双引号括起来，建议使用双引号 

- 常用的属性 

  1）class：定义元素的类名，用来选择和访问特定的元素 

  2）id：定义元素的唯一标识，在整个文档中必须是唯一的 

  3）name：定义元素的名称，一般用于表单数据提交到服务器 

  4）value：定义在元素内显示的默认值，一般常用于表单标签中 

  5）style：定义元素的css样式，(第一天简单使用，第二天主要学习)

​       .......

####  2.4 HTML 的特殊字符

- 什么是特殊字符 

  > 在 html 中，像< > ” ’ 空格 & 都是特殊字符，它们是语法本身的一部分。 

- 详见:  https://www.runoob.com/html/html-entities.html

- 除此之外，**搜狗输入法的符号大全**也可以编写在html代码中。



### 3. HTML 案例 新闻文本

- <font color=red>style 标签内的代码均为 CSS 语法，先照着抄即可，认知一下，明天会细讲。</font>

#### 3.1 样式演示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>样式演示</title>
    <style>
        div{
            /*显示边框*/
            border: 1px solid red;

            /*宽度 占屏幕的60%*/
            width: 60%;

            /*高度 500像素*/
            height: 500px;

            /*边框外边距：auto的意思是页面左右空白自适应并且相等*/
            margin: auto;
        }
    </style>
</head>
<body>
    <div>第一个div</div>
</body>
</html>
```

#### 3.2 文本标签演示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文本标签演示</title>
</head>
<body>
    <!--
        段落标签：<p>
    -->
    <p>这些年，马云的风头正盛，但是上个月他毅然辞去了阿里巴巴的职务。而马云所做的很多事情也的确改变了这个世界，特别是在移动支付领域，更是走在了世界的前列。如今中国的移动支付已经成为老百姓的必备，支付宝对中国社会的变革都带来了深远的影响。不过马云依然没有满足，他认为移动互联网将会成为人类的基础设施，而且这里面的机会和各种挑战还非常多。</p>
    <p>支付宝的诞生就是为了解决淘宝网的客户们的买卖问题，而随着支付宝的用户的不断增加，支付宝也推出了一系列的附加功能。比如生活缴费、转账汇款、还信用卡、车主服务、公益理财等，往简单的说，支付宝既可以满足人们的日常生活，又可以利用芝麻信用进行资金周转服务。除了芝麻分能够进行周转以外，互联网信用体系下的产品多多，我们对比以下几个产品看看区别:</p>
    <!--
         标题标签：<h1> ~ <h6>
     -->
    <h1>一级标题</h1>
    <h2>二级标题</h2>
    <h3>三级标题</h3>
    <h4>四级标题</h4>
    <h5>五级标题</h5>
    <h6>六级标题</h6>
    <!--
        水平线标签：<hr/>
        属性：
            size-大小
            color-颜色
    -->
    <hr size="4" color="red"/>
    <!--
        无序列表：<ul>
        属性：type-列表样式(disc实心圆、circle空心圆、square实心方块)
        列表项：<li>
    -->
    <ul type="circle">
        <li>javaEE</li>
        <li>HTML</li>
    </ul>
    <!--
        有序列表：<ol>
        属性：type-列表样式(1数字、A或a字母、I或i罗马字符)   start-起始位置
        列表项：<li>
    -->
    <ol type="1" start="10">
        <li>传智播客</li>
        <li>黑马程序员</li>
    </ol>
    <!--
        斜体标签：<i>    <em>
    -->
    <i>我倾斜了</i>
    <em>我倾斜了</em>
    <br/>
    <!--
        加粗标签：<strong>  <b>
    -->
    <strong>加粗文本</strong>
    <b>加粗文本</b>
    <br/>
    <!--
        文字标签：<font>
        属性：
            size-大小
            color-颜色
    -->
    <font size="5" color="yellow">这是一段文字</font>
</body>
</html>
```

#### 3.3 案例实现

##### 实现步骤

​     1）创建一个 html 文件。 

​     2）使用四个<div>标签划分区域(标题、作者、副标题、正文)。 

​     3）使用 <style> 标签设置div样式：宽度60%    外边距自动。 

​     4）使 用<h1>标签加入标题。 

​     5）使用<font>标签加入作者信息，颜色设置为灰色，字体大小为2。 

​     6）使用<hr>标签加入水平线。 

​     7）使用<h3>标签加入副标题。 

​     8）使用<p>标签加入正文段落。 

​     9）使用<ol>标签，加入有序列表。 

​     10）使用<b>标签，加入部分文字加粗。

##### **示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>新闻文本</title>
    <style>
        div{
            /*宽度  60%*/
            width: 60%;
            /*外边距*/
            margin: auto;
        }
    </style>
</head>
<body>
    <!--标题-->
    <div>
        <h1>支付宝特权福利！芝麻分600以上用户惊喜，网友：幸福来得突然？</h1>
    </div>

    <!--作者信息-->
    <div>
        <i><font size="2" color="gray">作者：itheima 2088-08-08</font></i>
        <hr/>
    </div>

    <!--副标题-->
    <div>
        <h3>支付宝特权福利！芝麻分600以上用户惊喜，网友：幸福来得突然？</h3>
    </div>

    <!--正文内容-->
    <div>
        <p>这些年，马云的风头正盛，但是上个月他毅然辞去了阿里巴巴的职务。而马云所做的很多事情也的确改变了这个世界，特别是在移动支付领域，更是走在了世界的前列。如今中国的移动支付已经成为老百姓的必备，支付宝对中国社会的变革都带来了深远的影响。不过马云依然没有满足，他认为移动互联网将会成为人类的基础设施，而且这里面的机会和各种挑战还非常多。</p>
        <p>支付宝的诞生就是为了解决淘宝网的客户们的买卖问题，而随着支付宝的用户的不断增加，支付宝也推出了一系列的附加功能。比如生活缴费、转账汇款、还信用卡、车主服务、公益理财等，往简单的说，支付宝既可以满足人们的日常生活，又可以利用芝麻信用进行资金周转服务。除了芝麻分能够进行周转以外，互联网信用体系下的产品多多，我们对比以下几个产品看看区别:
            <ol>
                <li>蚂蚁借呗，芝麻分600并且受到邀请开通福利，这个就是支付宝贷款，直接秒杀了银行贷款和线下金融公司，是现在支付宝用户使用最多的。</li>
                <li>微粒贷：于2015年上线，主要面向QQ和微信征信极好的用户而推出，受到邀请才能申请开通，额度最高有30万，难度较大</li>
                <li>蚂蚁巴士：这个在微信 蚂蚁巴士 公众平台申请,对于信用分要求530分以上才可以,额度1-30万不等，目前非常火爆</li>
            </ol>
        </p>
        <p>说起支付宝中的芝麻信用功能，相信更是受到了许多人的推崇，因为随着自己使用的不断增多，信用分会慢慢提高，而达到了一个阶段，就可以获得许多的福利。而当我们的芝麻信用分可以达到600分以上的时候，会有令我们想象不到的惊喜，接下来就让我们一起来看看，具体都有哪些惊喜吧。</p>
        <p><b>一、芝麻分600以上福利之信用购。</b>网购相信大家都不陌生，但是很多时候，网购都有一个通病，就是没办法试用，导致很多人买了很多自己不喜欢的东西。但是只要你的支付宝芝麻分在650及以上，就能立马享有0元下单，收到货使用满意了再进行付款。还能享用美食的专属优惠，是不是很耐斯</p>
        <p><b>二、芝麻分600以上福利之信用免押。</b>芝麻信用与木鸟短租联合推出信用住宿服务，芝麻分600及以上的用户可享受免押入住特权。木鸟短租拥有全国50万套房源，是国内领先的短租民宿预订平台。包括大家知道的飞猪信用住，大部分酒店可以免押金入住，离店再交钱。</p>
        <p><b>三、芝麻分600以上福利之国际驾照。</b>我们经常听说的可能只是中国驾照，但现在芝麻分已经应用到了国际领域，只要你的芝麻分够550就可以免费办理国际驾照，也有不少人非常佩服马云，一个简单的芝麻分居然有如此大的功能，也从侧面反应出来马云在国际上的地位，这个国际驾照是由新西兰、德国、澳大利亚联合认证，可以在全球200多个国家通行，相信大家一定都有一个自驾全球的梦想吧，而现在支付宝就给了你一把钥匙，剩下的就你自己搞定了！有没有想带着你的女神来一次浪漫之旅呢？</p>
        <p>随着互联网对我们生活的改变越来越大，信用这一词也被大家推上风口浪尖，不论是生活出行，还是其他的互联网服务，与信用体系已经密不可分了，马云当初说道，找老婆需要拼芝麻分，如今似乎也要成为现实，那么你们的芝麻分有多少了呢？</p>
    </div>
</body>
</html>
```



### 4. HTML 案例 头条页面

- 要想完成这个页面，首先要进行页面的布局，然后再填充文本、图片、超链接
- <font color=red>style 标签内的代码均为 CSS 语法，先照着抄即可，认知一下，明天会细讲。</font>

#### 4.1 样式演示

- 属性：float浮动(left|right|none)、clear清除浮动(both)、text-align文本对齐方式(left|center|right)、background背景色

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>样式演示</title>
    <style>
        /*给div标签添加边框*/
        div{
            border: 1px solid red;
        }

        /*左侧图片的div样式*/
        .left{
            width: 20%;
            float: left;
            height: 500px;
        }

        /*中间正文的div样式*/
        .center{
            width: 59%;
            float: left;
            height: 500px;
        }

        /*右侧广告图片的div样式*/
        .right{
            width: 20%;
            float: left;
            height: 500px;
        }

        /*底部超链接的div样式*/
        .footer{
            /*清除浮动效果*/
           /* 如果前面的DIV盒子里用到了浮动，
            那么在编写下一个DIV盒子之前，
            应该在前面加上<div style="clear:both"></div>，
            这样才能清除掉前面DIV盒子浮动带来的影响。*/
            clear: both;
            /*文本对齐方式*/
            text-align: center;
            /*背景颜色*/
            background: blue;
        }
    </style>
</head>
<body>
    <!--顶部登陆注册-->
    <div>top</div>

    <!--导航条-->
    <div>navibar</div>

    <!--左侧图片-->
    <div class="left">left</div>

    <!--中间正文-->
    <div class="center">center</div>

    <!--右侧广告图片-->
    <div class="right">right</div>

    <!--底部页脚超链接-->
    <div class="footer">footer</div>
</body>
</html>
```

#### 4.2 图片标签演示

- 相对路径：../  或者 ./都是相对路径写法，../ 表示当前文件所属目录的上一级目录，./指当前文件所属的目录
- 绝对路径写法：C:// 从盘符出发的写法都是绝对路径
- img 支持 相对 和 绝对两种写法

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>图片标签演示</title>
</head>
<body>
    <!--
        图片标签：<img>
        属性：
            src-图片的路径
            title-鼠标悬浮时显示的内容
            alt-图片找不到时显示的内容
            width-图片的宽度
            height-图片的高度
    -->
    <img src="../img/ad1.jpg" title="广告" alt="图片找不到啦" width="150px" height="150px"/>
</body>
</html>
```

#### 4.3 超链接标签演示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>超链接标签演示</title>
    <style>
        a{
            /*去掉超链接的下划线*/
            text-decoration: none;
            /*超链接的颜色*/
            color: black;
        }

        /*鼠标悬浮的样式控制*/
        a:hover{
            color: red;
        }
    </style>
</head>
<body>
    <!--
        超链接标签：<a>
        属性：
            href-跳转的地址
            target-跳转的方式(_self当前页面、_blank新标签页)
    -->
    <a href="01案例二：样式演示.html" target="_blank">点我跳转到样式演示</a>  <br/>
    <a href="http://www.itcast.cn" target="_blank">传智播客</a>  <br/>
    <a href="http://www.itheima.com" target="_self">黑马程序员</a>  <br/>
    <a href="http://www.itheima.com" target="_blank"><img src="../img/itheima.png" width="150px" height="50px"/></a>
</body>
</html>
```

#### 4.4 头条页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>头条页面</title>
    <style>
        /*给div标签添加边框*/
        /*div{
            border: 1px solid red;
        }*/

        /*左侧图片的div样式*/
        .left{
            width: 20%;
            float: left;
        }

        /*中间正文的div样式*/
        .center{
            width: 60%;
            float: left;
        }

        /*右侧广告图片的div样式*/
        .right{
            width: 20%;
            float: left;
        }

        /*底部超链接的div样式*/
        .footer{
            /*清除浮动效果*/
            clear: both;
            /*文本对齐方式*/
            text-align: center;
            /*背景颜色*/
            background: blue;
        }

        /*超链接的样式控制*/
        a{
            color: white;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <!--顶部登陆注册-->
    <div>
        <img src="../img/j1.png" width="100%"/>
    </div>

    <!--导航条-->
    <div>
        <img src="../img/j2.png" width="100%"/>
        <hr/>
    </div>

    <!--左侧图片-->
    <div class="left">
        <img src="../img/j3.png" width="100%"/>
    </div>

    <!--中间正文-->
    <div class="center">
        <div>
            <h1>支付宝特权福利！芝麻分600以上用户惊喜，网友：幸福来得突然？</h1>
        </div>

        <!--作者信息-->
        <div>
            <i><font size="2" color="gray">作者：itheima 2088-08-08</font></i>
            <hr/>
        </div>

        <!--副标题-->
        <div>
            <h3>支付宝特权福利！芝麻分600以上用户惊喜，网友：幸福来得突然？</h3>
        </div>

        <!--正文内容-->
        <div>
            <p>这些年，马云的风头正盛，但是上个月他毅然辞去了阿里巴巴的职务。而马云所做的很多事情也的确改变了这个世界，特别是在移动支付领域，更是走在了世界的前列。如今中国的移动支付已经成为老百姓的必备，支付宝对中国社会的变革都带来了深远的影响。不过马云依然没有满足，他认为移动互联网将会成为人类的基础设施，而且这里面的机会和各种挑战还非常多。</p>
            <p>支付宝的诞生就是为了解决淘宝网的客户们的买卖问题，而随着支付宝的用户的不断增加，支付宝也推出了一系列的附加功能。比如生活缴费、转账汇款、还信用卡、车主服务、公益理财等，往简单的说，支付宝既可以满足人们的日常生活，又可以利用芝麻信用进行资金周转服务。除了芝麻分能够进行周转以外，互联网信用体系下的产品多多，我们对比以下几个产品看看区别:
            <ol>
                <li>蚂蚁借呗，芝麻分600并且受到邀请开通福利，这个就是支付宝贷款，直接秒杀了银行贷款和线下金融公司，是现在支付宝用户使用最多的。</li>
                <li>微粒贷：于2015年上线，主要面向QQ和微信征信极好的用户而推出，受到邀请才能申请开通，额度最高有30万，难度较大</li>
                <li>蚂蚁巴士：这个在微信 蚂蚁巴士 公众平台申请,对于信用分要求530分以上才可以,额度1-30万不等，目前非常火爆</li>
            </ol>
            <img src="../img/1.jpg" width="100%"/>
            </p>
            <p>说起支付宝中的芝麻信用功能，相信更是受到了许多人的推崇，因为随着自己使用的不断增多，信用分会慢慢提高，而达到了一个阶段，就可以获得许多的福利。而当我们的芝麻信用分可以达到600分以上的时候，会有令我们想象不到的惊喜，接下来就让我们一起来看看，具体都有哪些惊喜吧。</p>
            <p><b>一、芝麻分600以上福利之信用购。</b>网购相信大家都不陌生，但是很多时候，网购都有一个通病，就是没办法试用，导致很多人买了很多自己不喜欢的东西。但是只要你的支付宝芝麻分在650及以上，就能立马享有0元下单，收到货使用满意了再进行付款。还能享用美食的专属优惠，是不是很耐斯</p>
            <p><b>二、芝麻分600以上福利之信用免押。</b>芝麻信用与木鸟短租联合推出信用住宿服务，芝麻分600及以上的用户可享受免押入住特权。木鸟短租拥有全国50万套房源，是国内领先的短租民宿预订平台。包括大家知道的飞猪信用住，大部分酒店可以免押金入住，离店再交钱。</p>
            <img src="../img/2.jpg" width="100%"/>
            <p><b>三、芝麻分600以上福利之国际驾照。</b>我们经常听说的可能只是中国驾照，但现在芝麻分已经应用到了国际领域，只要你的芝麻分够550就可以免费办理国际驾照，也有不少人非常佩服马云，一个简单的芝麻分居然有如此大的功能，也从侧面反应出来马云在国际上的地位，这个国际驾照是由新西兰、德国、澳大利亚联合认证，可以在全球200多个国家通行，相信大家一定都有一个自驾全球的梦想吧，而现在支付宝就给了你一把钥匙，剩下的就你自己搞定了！有没有想带着你的女神来一次浪漫之旅呢？</p>
            <p>随着互联网对我们生活的改变越来越大，信用这一词也被大家推上风口浪尖，不论是生活出行，还是其他的互联网服务，与信用体系已经密不可分了，马云当初说道，找老婆需要拼芝麻分，如今似乎也要成为现实，那么你们的芝麻分有多少了呢？</p>
        </div>
    </div>

    <!--右侧广告图片-->
    <div class="right">
        <img src="../img/ad1.jpg" width="100%"/>
        <img src="../img/ad2.jpg" width="100%"/>
        <img src="../img/ad3.jpg" width="100%"/>
        <img src="../img/ad1.jpg" width="100%"/>
        <img src="../img/ad2.jpg" width="100%"/>
        <img src="../img/ad3.jpg" width="100%"/>
    </div>

    <!--底部页脚超链接-->
    <div class="footer">
        <a href="#">关于黑马</a> &nbsp;
        <a href="#">帮助中心</a> &nbsp;
        <a href="#">开放平台</a> &nbsp;
        <a href="#">诚聘英才</a> &nbsp;
        <a href="#">联系我们</a> &nbsp;
        <a href="#">法律声明</a> &nbsp;
        <a href="#">隐私政策</a> &nbsp;
        <a href="#">知识产权</a> &nbsp;
        <a href="#">廉政举报</a> &nbsp;
    </div>
</body>
</html>
```



### 5. HTML 案例 注册页面

- 表单标签 和 div标签+css实现布局
- <font color=red>style 标签内的代码均为 CSS 语法，先照着抄即可，认知一下，明天会细讲。</font>



#### 5.1 样式演示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>背景图片</title>
    <style>
        body{
            /*添加背景图片*/
            background: url("../img/bg.png");
        }
    </style>
</head>
<body>

</body>
</html>
```

#### 5.2 表单标签

表单标签  + 表单项标签 ：

​	提供页面和用户的交互的方式，用户可以在页面进行编辑和输入。在页面收集这些信息，提交(submit)给后台！！



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表单标签的演示</title>
</head>
<body>
    <!--
        表单标签：<form>
        属性：
            action-提交的路径
            method-提交的方式(get和post)
            autocomplete-记录补全(on和off)
    -->
    <!--get方式的表单-->
    <form action="#" method="get" autocomplete="off">
        用户名：<input type="text" name="username"/>
        <button type="submit">提交</button>
    </form>

    <!--post方式的表单-->
    <form action="#" method="post" autocomplete="off">
        用户名：<input type="text" name="username"/>
        <button type="submit">提交</button>
    </form>
</body>
</html>
```

#### 5.3 表单项标签

- input 标签通过type属性可以显示很多种展示效果
- <font color=red>表单项标签中的数据要想被form标签收集并传给服务器, 必须指定name属性</font>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表单项标签</title>
</head>
<body>
    <!--
        表单项标签：<label>  表单元素说明
        属性：for属性，属性值必须和表单项标签id属性值一致

        表单项标签：<input>   多种类型数据
        属性：
            type-数据类型
            id-唯一标识
            name-提交服务器的标识
            value-默认的数据值
            placeholder-默认的提示信息
            required-是否必须

        按钮标签：<button>
        属性：
            type-按钮的类型(submit提交、reset重置、button普通按钮)
    -->
    <form action="#" method="get" autocomplete="off">
        <label for="username">用户名：</label>
        <input type="text" id="username" name="username" value="" placeholder=" 请在此处输入用户名" required/>
        
        <button type="submit">提交</button>
        <button type="reset">重置</button>
        <button type="button">按钮</button>
    </form>
</body>
</html>
```

##### 5.3.1 Input标签type属性演示

text、password、radio、checkbox、file、hidden



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>type属性演示</title>
</head>
<body>
    <!--
        type属性
        <input type="text"/>       普通文本输入框
        <input type="password"/>   密码输入框
        <input type="file"/>       文件上传框
        <input type="hidden"/>     隐藏域   value属性设置实际提交的值  
        <input type="radio"/>      单选框。必须有相同的name属性值，value属性真实提交的数据，checked属性默认选中
        <input type="checkbox"/>   多选框。必须有相同的name属性值，value属性真实提交的数据，checked属性默认选中



 		<input type="email"/>      邮箱输入框
        <input type="date"/>       日期框
        <input type="time"/>       时间框
        <input type="datetime-local"/>  本地日期时间框
        <input type="number"/>     数字框
        <input type="range"/>      滚动条数值框  min-最小值   max-最大值  step-步进值
        <input type="search"/>     可清除文本框
        <input type="tel"/>        电话框
        <input type="url"/>        网址框


    -->
    <form action="#" method="get" autocomplete="off">
        <label for="username">用户名：</label>
        <input type="text" id="username" name="username"/>  <br/>

        <label for="password">密码：</label>
        <input type="password" id="password" name="password"/> <br/>

        <label for="email">邮箱：</label>
        <input type="email" id="email" name="email"/> <br/>

        <label for="gender">性别：</label>gender=
        <input type="radio" id="gender" name="gender" value="men"/>男
        <input type="radio" name="gender" value="women"/>女
        <input type="radio" name="gender" value="other"/>其他<br/>

        <label for="hobby">爱好：</label>
        <input type="checkbox" id="hobby" name="hobby" value="music" checked/>音乐
        <input type="checkbox" name="hobby" value="game"/>游戏 <br/>

        <label for="birthday">生日：</label>
        <input type="date" id="birthday" name="birthday"/> <br/>

        <label for="time">当前时间：</label>
        <input type="time" id="time" name="time"/> <br/>

        <label for="insert">注册时间：</label>
        <input type="datetime-local" id="insert" name="insert"/> <br/>

        <label for="age">年龄：</label>
        <input type="number" id="age" name="age"/> <br/>

        <label for="range">心情值(1~10)：</label>
        <input type="range" id="range" name="range" min="1" max="10" step="1"/> <br/>

        <label for="search">可全部清除文本：</label>
        <input type="search" id="search" name="search"/> <br/>

        <label for="tel">电话：</label>
        <input type="tel" id="tel" name="tel"/> <br/>

        <label for="url">个人网站：</label>
        <input type="url" id="url" name="url"/> <br/>

        <label for="file">文件上传：</label>
        <input type="file" id="file" name="file"/> <br/>

        <label for="hidden">隐藏信息：</label>
        <input type="hidden" id="hidden" name="hidden" value="itheima"/> <br/>

        <button type="submit">提交</button>
        <button type="reset">重置</button>
    </form>
</body>
</html>
```

##### 5.3.2 其他表单项标签

- select标签 和 option标签实现下拉列表
- textarea超文本区，多用于意见反馈和文字留言

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>其他常用表单项标签演示</title>
</head>
<body>
    <!--
        下拉列表标签：<select>
        列表项标签：<option>
        列表项分组标签：<optgroup>   属性：label设置分组名称

        文本域标签：<textarea>
        属性：
            rows-行数
            cols-列数
    -->
    <form action="#" method="get" autocomplete="off">
        所在城市：
        <select name="city">
            <option>---请选择城市---</option>
            <optgroup label="直辖市">
                <option>北京</option>
                <option>上海</option>
            </optgroup>
            <optgroup label="省会市">
                <option>杭州</option>
                <option>武汉</option>
            </optgroup>
    	</select>
        <br/>

        个人介绍：<textarea name="desc" rows="5" cols="20"></textarea>

        <button type="submit">提交</button>
        <button type="reset">重置</button>
    </form>
</body>
</html>
```

#### 5.4 注册页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
    <style>
        body{
            background: url("../img/bg.png");
        }

        .center{
            /*背景颜色*/
            background: white;
            /*宽度*/
            width: 400px;
            /*文本对齐方式*/
            text-align: center;
            /*外边距*/
            margin: auto;
        }
    </style>
</head>
<body>
    <!--顶部-公司图标-->
    <div>
        <img src="../img/logo.png"/>
    </div>

    <!--中间-注册信息-->
    <div class="center">
        <div>注册详情</div>
        <hr/>

        <!--表单标签-->
        <form action="#" method="get" autocomplete="off">
            <div>
                <label for="username">姓名：</label>
                <input type="text" id="username" name="username" value="" placeholder=" 在此输入姓名" required/>
            </div>

            <div>
                <label for="password">密码：</label>
                <input type="password" id="password" name="password" value="" placeholder=" 在此输入密码" required/>
            </div>

            <div>
                <label for="email">邮箱：</label>
                <input type="email" id="email" name="email" value="" placeholder=" 在此输入邮箱" required/>
            </div>

            <div>
                <label for="tel">手机：</label>
                <input type="tel" id="tel" name="tel" value="" placeholder=" 在此输入手机" required/>
            </div>
            <hr/>

            <div>
                <label for="gender">性别：</label>
                <input type="radio" id="gender" name="gender" value="men"/>男 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <input type="radio" name="gender" value="women"/>女 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
            </div>

            <div>
                <label for="hobby">爱好：</label>
                <input type="checkbox" id="hobby" name="hobby" value="music"/>音乐
                <input type="checkbox" name="hobby" value="movie"/>电影
                <input type="checkbox" name="hobby" value="game"/>游戏
            </div>

            <div>
                <label for="birthday">出生日期：</label>
                <input type="date" id="birthday" name="birthday" value=""/>
            </div>

            <div>
                <label for="city">所在城市：</label>
                <select id="city" name="city">
                    <option>---请选择所在城市---</option>
                    <optgroup label="直辖市">
                        <option>北京</option>
                        <option>上海</option>
                        <option>广州</option>
                        <option>深圳</option>
                    </optgroup>
                    <optgroup label="省会市">
                        <option>西安</option>
                        <option>杭州</option>
                        <option>郑州</option>
                        <option>武汉</option>
                    </optgroup>
                </select>
            </div>
            <hr/>
            <div>
                <label for="desc">个性签名：</label>
                <textarea id="desc" name="desc" rows="5" cols="40" placeholder=" 请写下您的与众不同"></textarea>
            </div>
            <hr/>
            <button type="submit">注册</button>
            <button type="reset">重置</button>
        </form>
    </div>
</body>
</html>
```





总结：

html要求：掌握基本的标签，能够阅读和修改html页面！！

掌握基本的标签：

​	表单标签： 收集用户数据，提交给后台！！

```html
<form action="" method="get/post">
    <label for="">用户名</label>
    <input id="" name="" type="text/password/radio/checkbox/file/hidden" />
    <select name="">
        <optgroup>
        	<option>xxx</option>
        </optgroup>
    </select>
    <textarea name=""></textarea>
</form>
```

图片标签

```html
<img src="" alt="" title="" width="" height=""/>
```

链接标签

```html
<a href="" target="_blank/_self"></a>
```

页面布局的标签

div  span

文本相关的标签

h1~h6  p   hr  br  .....