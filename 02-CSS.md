# CSS层叠样式表

### 今日重点

- 基础选择器：id   class  元素
- 扩展选择器:   属性 伪类 
- 案例中样式属性多看看 

### 1. CSS 快速入门

- CSS (Cascading Style Sheets)：层叠样式表。 
- 用于设置和布局网页的一种计算机语言。告知浏览器如何渲染解析页面元素。

#### 1.1 CSS 的组成

- 选择器：过滤整个html文档中，对标签进行筛选。

- 属性：用于美化html标签，可以直接作用在标签上，也可以结合选择器先进行过滤，在进行美化。

  > 格式是:     属性名:属性值  
  >
  > **注意**：多个属性用 ；号隔开



#### 1.2 入门案例

- 实现步骤 

  > ① 新建一个 html 文件。 
  >
  > ② 在<body>标签中编写一个<h1>标签，内容为：今天开始变漂亮。 
  >
  > ③ 在<head>标签中编写一个<style>标签。 
  >
  > ④ 为<h1>标签设置样式：颜色为红色、字体大小为100px。 
  >
  > ⑤ 通过浏览器打开页面查看效果。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>入门案例</title>
    <style>
        /*元素选择器*/
        h1{
            /*样式属性: color 颜色*/
            color: red;
            /*样式属性: 字体大小 100像素*/
            font-size: 100px;
        }
    </style>
</head>
<body>
    <h1>今天开始变漂亮！</h1>
</body>
</html>
```



### 2. CSS 基本语法

#### 2.1 css 与 html结合方式

- 推荐使用外部样式 和 内部样式

##### **内联样式**

- 耦合度最高，最难维护
- 直接定义css样式属性，无需考虑选择器
- 作用范围是：当前标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>引入方式1</title>
</head>
<body>
    <!--内联样式-->
    <h1 style="color: red; font-size: 20px">Hello World</h1>

    <h1>CSS</h1>
</body>
</html>
```

##### 内部样式

- 先定义选择器, 在定义样式属性
- 作用范围是：当前整个html文档

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>引入方式2</title>
    <!--内部样式-->
    <style>
        div{
            color: red;
            font-size: 20px;
        }
    </style>
</head>
<body>
    <div>div1</div>
    <div>div2</div>
</body>
</html>
```

##### 外部样式

- 耦合度最低，作用范围最大，那个文档想用，都可以引入

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>引入方式3</title>
    <!--外部样式-->
    <link rel="stylesheet" href="css/01.css"/>
</head>
<body>
    <div>div1</div>
    <div>div2</div>
</body>
</html>
```

#### 2.2 CSS 的注释

- 注释快捷键: ctrl + / 

-  注释的格式 

  > /* 注释的内容*/ 

- 注释的特点：1）被注释掉的样式，不会被浏览器解析



#### 2.3 CSS 的选择器

**选择器参考手册**: https://www.runoob.com/cssref/css-selectors.html

**属性参考手册**：https://www.runoob.com/cssref/css-reference.html

- 说白了，选择器就是用来过滤当前HTML文档，筛选出来我们想要的标签（元素）。

- 好处：一个 HTML 文件中会存在很多个元素，如果想对不同的元素添加不同的样式，可以使用选择器分别筛选出来想要标签进行样式的控制。

- 选择器和选择器之间可以结合使用，例如

  > 1）a:link{}  元素选择器a 和 伪类 选择器 : link 结合使用
  >
  > 2）input[type='raddio']{}   元素选择器input  和 属性选择器结合使用
  >
  > 3）body   div   img{}  三个元素选择器结合为一个 后代选择器，过滤出来img标签

##### 基本选择器

| 名称       | 特点       | 作用                    | 示例        |
| ---------- | ---------- | ----------------------- | ----------- |
| 元素选择器 | 元素名一致 | 根据标签名匹配元素      | div{}       |
| 类选择器   | .          | 根据class属性值匹配元素 | .center{}   |
| id选择器   | #          | 根据id属性值匹配元素    | #username{} |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>基本选择器</title>
    <style>
        /*元素选择器*/
        div{
            color: red;
        }

        /*类选择器*/
        .cls{
            color: blue;
        }

        /*id选择器*/
        #d1{
            color: green;
        }

        #d2{
            color: pink;
        }
    </style>
</head>
<body>
    <div>div1</div>

    <div class="cls">div2</div>
    <div class="cls">div3</div>

    <div id="d1">div4</div>
    <div id="d2">div5</div>
</body>
</html>
```

##### 扩展选择器

- 伪类选择器不光光可以针对标签状态进行过滤，具体查阅文档
- 组合选择器的过滤条件还是各自成员选择器，他只是将相同样式的选择器并到一起去定义样式属性，减少代码的冗余。

| 名称       | 特点       | 作用                       | 示例                       |
| ---------- | ---------- | -------------------------- | -------------------------- |
| 属性选择器 | []         | 根据指定标签属性匹配元素   | [type]{}<br/>[type=text]{} |
| 伪类选择器 | :link      | 未访问的状态               | a:link{}                   |
|            | :visited   | 已访问的状态               | a:visited{}                |
|            | :hover     | 鼠标悬浮的状态             | a:hover                    |
|            | :active    | 已选中的状态               | a:active                   |
| 后代选择器 | 空格       | 选择器2是选择器1的后代就行 | .center li{}               |
| 组合选择器 | , （逗号） | 可以同时匹配多个元素       | div,span,p{}               |

###### 属性选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>属性选择器</title>
    <style>
        [type]{
            color: red;
        }
        /*多个选择器的结合*/
        /*intpu[type]{
            color: red;
        }*/
        [type=password]{
            color: blue;
        }

    </style>
</head>
<body>
    用户名：<input type="text"/> <br/>
    密码：<input type="password"/> <br/>
    邮箱：<input type="email"/> <br/>

    个人简介: <textarea type="text"></textarea>
</body>
</html>
```

###### 伪类选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>伪类选择器</title>
    <style>
        a{
            text-decoration: none;
        }

        /*未访问的状态*/
        a:link{
            color: black;
        }

        /*已访问的状态*/
        a:visited{
            color: blue;
        }

        /*鼠标悬浮的状态*/
        a:hover{
            color: red;
        }

        /*已选中的状态*/
        a:active{
            color: yellow;
        }

        /*选择div标签，并且是它爹的第一个孩子的div*/
        div:first-child{
            border: 1px solid red;
        }
    </style>
</head>
<body>

    <div>
        <a href="https://www.baidu.com">百度一下</a>
    </div>

    <div>
        我是第二个div
    </div>

</body>
</html>
```

###### 层级选择器

- 后代和子代选择器都是层级选择器

- 二者都是在找 辈分最小的孩子标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>后代和子代选择器</title>
    <style>
        /*
         后代选择器：空格
            过滤条件：筛选选择器2， 只要选择器2 是 选择器1 的后代就行，孩子 孙子 太孙子。。。都可以
        .center通过该选择器选择的标签内部包含的li标签
        */
        .center li{
            color: red;
        }

        /*
         子代选择器：>
            过滤条件：只能筛选孩子标签，选择器2 必须是 选择器1的孩子
            特点：>

        */
        /*div > ol{
            color: blue;
        }*/


    </style>
</head>
<body>

    <div class="top">
        <ol>
            <li>aa</li>
            <li>bb</li>
        </ol>
    </div>

    <div class="center">
        <form>
            <ol>
                <li>cc</li>
                <li>dd</li>
            </ol>
        </form>

    </div>

</body>
</html>
```

###### 组合选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>组合选择器</title>
    <style>

        /*分组选择器*/
        span,p,.center,#d1{
            color: blue;
        }
    </style>
</head>
<body>
    <span>span</span> <br/>
    <p>段落</p>
</body>
</html>
```

#### 2.4 CSS 小结

- CSS 三种引入方式:   

  1）内联样式

  2）内部样式  

  3）外部样式 

- CSS 的注释 ：/* 注释的内容 */

- CSS 的选择器 

  1）基本选择器：可以通过元素、类、id来选择元素 

  2）属性选择器：可以通过属性来选择元素 

  3）伪类选择器：可以为选择器添加一些特殊效果 

  4）并集选择器：将多个选择器并到一起，进行样式属性的定义

  5）层级选择器：后代 和 子代选择器进行标签层级过滤



### 3. CSS 案例 头条页面

- 要想完成这个页面，首先要进行页面的布局，然后再填充文本、图片、超链接





#### 3.1 边框样式演示

##### CSS边框样式属性

| 边框名称      | 作用         | 属性值        |
| ------------- | ------------ | ------------- |
| border        | 设置所有边框 |               |
| border-top    | 设置上边框   | solid-实线    |
| border-left   | 设置左边框   | dotted-圆点   |
| border-right  | 设置右边框   | double-双实线 |
| border-bottom | 设置下边框   | dashed-虚线   |
| border-radius | 设置边框弧度 |               |

##### **示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>边框样式演示</title>
    <style>
        #d1{
            /*设置所有边框*/
            /*border: 5px solid black;*/

            /*设置上边框*/
            border-top: 5px solid black;
            /*设置左边框*/
            border-left: 5px double red;
            /*设置右边框*/
            border-right: 5px dotted blue;
            /*设置下边框*/
            border-bottom: 5px dashed pink;

            width: 150px;
            height: 150px;
        }

        #d2{
            border: 5px solid red;
            /*设置边框的弧度*/
            border-radius: 25px;

            width: 150px;
            height: 150px;
        }
    </style>
</head>
<body>
    <div id="d1"></div>
    <br/>
    <div id="d2"></div>
</body>
</html>
```

#### 3.2 文本样式演示

##### CSS文本样式属性

| 样式名称        | 作用         | 属性值                                                       |
| --------------- | ------------ | ------------------------------------------------------------ |
| color           | 文字颜色     | 颜色单词(red)、RGB值(#ff0000)                                |
| font-family     | 字体         | 微软雅黑、宋体等等                                           |
| font-size       | 字体大小     | 像素点 20px                                                  |
| text-decoration | 文字下划线   | 1）none：无    2）underline：下划线   3）overline：上划线   4）line-through：删除线 |
| text-align      | 水平对齐方式 | left：居左 right：居右 center：居中                          |
| line-height     | 行间距       | 像素点20px                                                   |
| vertical-align  | 垂直对齐方式 | top：居上 bottom：居下 middle：居中 百分比调节               |

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文本样式演示</title>
    <style>
        div{
            /*文本颜色*/
            color: /*red*/ #ff0000;

            /*字体*/
            font-family: /*宋体*/ 微软雅黑;

            /*字体大小*/
            font-size: 25px;

            /*下划线  none：无  underline：下划线  overline：上划线  line-through：删除线*/
            text-decoration: none;

            /*水平对齐方式  left：居左  center：居中  right：居右*/
            text-align: center;

            /*行间距*/
            line-height: 60px;
        }

        span{
            /*文字垂直对齐  top：居上   bottom：居下  middle：居中   百分比*/
            vertical-align: 50%;
        }
    </style>
</head>
<body>

    <div>
        我是文字
    </div>
    <div>
        我是文字
    </div>

    <img src="../img/wx.png" width="38px" height="38px"/>
    <span>微信</span>
</body>
</html>
```

#### 3.3 案例实现

##### 3.3.1 实现步骤-顶部和导航条

1）创建一个 html 文件。 

2）创建顶部<div>标签。 

3）通过三个<a>标签实现登录、注册、更多三个超链接。 

4）设置顶部样式(背景色、行高、文字对齐方式、字体大小、超链接颜色、悬浮、和去除下划线)。 

5）创建导航条<div>标签。 

6）通过<img>标签引入logo 图片。 

7）通过两个<a>标签实现首页、科技两个超链接。

8）通过<font>标签实现正文两个字的显示。 

9）设置导航条样式(行高)。

##### 3.3.2 实现步骤-左侧分享、中间正文、右侧广告图片

1）创建左侧分享<div>标签。 

2）通过<a>标签嵌套<img>标签和<span>标签实现图片和文字的显示。 

3）设置左侧分享样式(宽度、文字水平对齐、浮动、图片大小、文字垂直对齐)。 4）创建中间正文<div>标签。 

5）实现正文内容的填充。 

6）设置中间正文样式(宽度、浮动)。 

7）创建右侧广告<div>标签。 

8）通过<img>标签引入广告图片。 

9）设置右侧广告样式(宽度、浮动)。

##### 3.3.3 实现步骤-底部页脚

1）创建底部页脚<div>标签。 

2）通过<a>标签实现超链接。 

3）设置底部页脚样式(浮动、背景色、文字水平对齐、行高、超链接颜色)。

##### 3.3.4 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>头条页面</title>
    <link rel="stylesheet" href="../css/news.css"/>
</head>
<body>
    <!--顶部登录注册更多-->
    <div class="top">
        <a href="../案例二/04案例二：登录页面.html" target="_self">登录&nbsp;&nbsp;</a>
        <a href="#">注册&nbsp;&nbsp;</a>
        <a href="#">更多&nbsp;&nbsp;</a>
    </div>

    <!--导航条-->
    <div class="nav">
        <img src="../img/logo.png"/>
        <a href="#">首页&nbsp;&nbsp;</a>/
        <a href="#">科技&nbsp;&nbsp;</a>/
        <font color="gray">正文</font>
        <hr/>
    </div>

    <!--左侧分享-->
    <div class="left">
        <a href="#"> <img src="../img/cc.png"/> <span>&nbsp;评论</span> </a>
        <hr/>
        <a href="#"> <img src="../img/repost.png"/> <span>&nbsp;转发</span> </a>  <br/>
        <a href="#"> <img src="../img/weibo.png"/> <span>&nbsp;微博</span> </a>  <br/>
        <a href="#"> <img src="../img/qq.png"/> <span>&nbsp;空间</span> </a>  <br/>
        <a href="#"> <img src="../img/wx.png"/> <span>&nbsp;微信</span> </a>  <br/>
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



### 4. CSS 案例 登录页面

- 要想完成这个页面，首先要进行页面的布局，然后使用表单标签完成表单项

#### 4.1 表格标签

| 标签名称 | 作用                 | 属性                                              |
| -------- | -------------------- | ------------------------------------------------- |
| table    | 表格标签             | width-宽度 height-高度 border-边框 align-对齐方式 |
| tr       | 行标签               | align-对齐方式：center   rignt  left              |
| td       | 单元格(列) 标签      | rowspan-合并行 colspan-合并列                     |
| th       | 表头标签(加粗和居中) |                                                   |
| thead    | 表头语义标签         |                                                   |
| tbody    | 表体语义标签         |                                                   |
| tfoot    | 表脚语义标签         |                                                   |

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格标签演示</title>
</head>
<body>
    <!--
        表格标签：<table>
        属性：
            width-宽度
            height-高度
            border-边框
            align-对齐方式

        行标签：<tr>
        属性：
            align-对齐方式

        单元格标签：<td>
        属性：
            rowspan-合并行
            colspan-和并列

        表头标签：<th> 自带居中和加粗效果
        表格标题：<caption>
        表头语义标签：<thead>
        表体语义标签：<tbody>
        表脚语义标签：<tfoot>
    -->
    <table width="400px" border="1px" align="center">

        <caption>学生成绩表</caption>
        <thead>
            <tr>
                <th>姓名</th>
                <th>性别</th>
                <th>年龄</th>
                <th>数学</th>
                <th>语文</th>
            </tr>
        </thead>

        <tbody>
            <tr align="center">
                <td>张三</td>
                <td rowspan="2">男</td>
                <td>23</td>
                <td colspan="2">90</td>
                <!--<td>90</td>-->
            </tr>

            <tr align="center">
                <td>李四</td>
                <!--<td>男</td>-->
                <td>24</td>
                <td>95</td>
                <td>98</td>
            </tr>
        </tbody>

        <tfoot>
            <tr>
                <td colspan="4">总分数：</td>
                <!--<td></td>
                <td></td>
                <td></td>-->
                <td>373</td>
            </tr>
        </tfoot>
    </table>
</body>
</html>
```

#### 4.2  样式控制

| 样式名称          | 作用         | 属性                                                         |
| ----------------- | ------------ | ------------------------------------------------------------ |
| background-repeat | 控制背景重复 | no-repeat：不重复 repeat-x：水平重复 repeat-y：垂直重复 repeat：水平+垂直重复 |
| outline           | 控制轮廓     | double：双实线 dotted：圆点 dashed：虚线 none：无            |
| display           | 控制元素     | inline：内联元素(无换行、无长宽) block：块级元素(有换行) inline-block：内联元素(有长宽) none：隐藏元素 |

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>样式演示</title>
    <style>
        /*背景图片重复  no-repeat：不重复  repeat-x：水平重复  repeat-y：垂直重复   repeat：水平+垂直重复*/
        body{
            background: url("../img/bg.jpg");

            background-repeat: repeat;
        }

        /*轮廓控制 double：双实线   dotted：圆点   dashed：虚线   none：无*/
        input{
            outline: none;
        }

        /*元素显示  inline：内联元素(无换行、无长宽)   block：块级元素(有换行)  inline-block：内联元素(有长宽)  none：隐藏元素*/
        div{
            display: none;
            width: 100px;
        }
    </style>
</head>
<body>
    用户名：<input type="text"/> <br/>

    <div>春季</div>
    <div>夏季</div>
    <div>秋季</div>
    <div>冬季</div>
</body>
</html>
```

#### 4.3 盒子模型 

- 盒子模型是通过设置边框和元素内容的边距，从而实现布局的方式。分为内边距和外边距两种方式！ 
- 如果想实现布局，可以采用设置内边距或外边距来实现。

| 外边距名称    | 作用                     | 举例                         |
| ------------- | ------------------------ | ---------------------------- |
| margin-top    | 上外边距                 | margin-top: 50px;            |
| margin-left   | 左外边距                 | margin-left: 50px;           |
| margin-right  | 右外边距                 | margin-right: 50px;          |
| margin-bottom | 下外边距                 | margin-bottom: 50px;         |
| margin        | 通用上下左右外边距       | margin: 50px;                |
| margin        | 通用单独上右下左外边距   | margin: 70px 35px 30px 65px; |
| margin        | 自动计算外边距并水平居中 | margin: auto;                |

| 内边距名称     | 作用                     | 举例                          |
| -------------- | ------------------------ | ----------------------------- |
| padding-top    | 上内边距                 | padding-top: 50px;            |
| padding-left   | 左内边距                 | padding-left: 50px;           |
| padding-right  | 右内边距                 | padding-right: 50px;          |
| padding-bottom | 下内边距                 | padding-bottom: 50px;         |
| padding        | 通用上下左右内边距       | padding: 50px;                |
| padding        | 通用单独上右下左内边距   | padding: 70px 35px 30px 65px; |
| padding        | 自动计算内边距并水平居中 | padding: auto;                |

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>盒子模型演示</title>
    <style>
        .wai{
            border: 1px solid red;
            width: 200px;
            height: 200px;

            /*设置内边距,会导致外框的变化, 可以通过来box-sizing: border-box解决*/
            /*padding: 50px;*/


            /*
                设置盒子的属性，让width和height就是最终盒子的大小
             */
            /*box-sizing: border-box;*/
        }

        .nei{
            border: 1px solid blue;
            width: 100px;
            height: 100px;

            /*设置外边距*/
            /*margin-top: 50px;
            margin-left: 50px;
            margin-right: 50px;
            margin-bottom: 50px;*/

            /*margin: 50px;*/

            /*上、右、下、左*/
            /*margin: 70px 35px 30px 65px;*/
        }
    </style>
</head>
<body>
    <div class="wai">
        <div class="nei">

        </div>
    </div>
</body>
</html>
```

#### 4.4 案例实现

##### 4.4.1 实现步骤-顶部和中间表单

1）创建一个 html 文件。 

2）创建三个<div>标签划分区域(顶部公司图标、中间表单、底部页脚)。 

3）在顶部<div>标签中通过<img>标签引入图片。 

4）在中间表单<div>标签中通过表单标签和表格标签填充表单项。 

5）设置样式(背景图片、背景色、宽度、外边距、弧度、文字水平对齐)。

##### 4.4.2 实现步骤-中间表单样式

1）设置表头样式(字体大小、颜色)。 

2）设置表体提示样式(文字大小)。 

3）设置表体输入框样式(边框、弧度、宽度、高度、轮廓)。 

4）设置表底按钮样式(边框、弧度、宽度、高度、背景色、字体颜色、字体大小)。 5）设置表行样式(行高)

##### 4.4.3 实现步骤-底部页脚

1）在底部页脚<div>标签中通过文字和<a>标签实现超链接。 

2）设置底部页脚样式(宽度、外边距、字体大小、字体颜色)。 

3）设置超链接样式(去除下划线、超链接文字颜色)。 

4）在案例一头条页面登录超链接中，引入跳转登录页面。



##### 4.4.5 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
    <link rel="stylesheet" href="../css/login.css"/>
</head>
<body>
    <!--顶部公司图标-->
    <div>
        <img src="../img/logo.png"/>
    </div>

    <!--中间表单-->
    <div class="center">
        <form action="#" method="get" autocomplete="off">
            <table width="100%">
                <thead>
                    <tr>
                        <th colspan="2">账&nbsp;密&nbsp;登&nbsp;录<hr/></th>
                    </tr>
                </thead>

                <tbody>
                    <tr>
                        <td>
                            <label for="username">账号</label>
                        </td>
                        <td>
                            <input type="text" id="username" name="username" placeholder=" 请输入账号" required/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <label for="password">密码</label>
                        </td>
                        <td>
                            <input type="password" id="password" name="password" placeholder=" 请输入密码" required/>
                        </td>
                    </tr>
                </tbody>

                <tfoot>
                    <tr>
                        <td colspan="2">
                            <button type="submit">确&nbsp;定</button>
                        </td>
                    </tr>
                </tfoot>
            </table>
        </form>
    </div>

    <!--底部页脚-->
    <div class="footer">
        <br/><br/>
        登录/注册即表示您同意&nbsp;&nbsp;
        <a href="#" target="_blank">用户协议</a>&nbsp;&nbsp;
        和&nbsp;&nbsp;
        <a href="#" target="_blank">隐私条款</a>&nbsp;&nbsp;&nbsp;&nbsp;
        <a href="#" target="_blank">忘记密码?</a>
    </div>
</body>
</html>
```



### 5. Nginx 服务器

#### 5.1 Nginx 介绍

- Nginx 是一款服务器软件, 我们可以将前端代码发布到Nginx中，这样别人就可以通过网络访问我们的页面了。

#### 5.2 Nginx 发布项目

```shell
#进入到nginx 默认发布项目文件夹下：/itcast/install/nginx-1.18/html
[root@localhost ~]$ cd /itcast/install/nginx-1.18/html
[root@localhost html]$ 

#上传项目文件夹到该html目录下; 先在window中将项目“案例一 案例二 img css” 四个文件夹打成web.zip
#使用rz命令上传web.zip 到 centos
[root@localhost html]$ rz

#上传后，解压缩
[root@localhost html]$ unzip web.zip

#启动nginx, 如果启动过了, 不需要重启
[root@localhost html]$ ../sbin/nginx

#测试访问如下地址, 查看效果
http://192.168.23.129/案例一/03案例一：头条页面.html

```



### 6. 扩展-(了解即可)

```html
<!-- 
    1. 选择器之间存在优先级, id选择器 > class选择器 > 元素选择器 > 其他选择器,  *{} 所有选择器优先级最低。！important 修饰的样式
    优先级最高

    2. 后代选择器 和 子代选择器, 都是过滤 备份最小的那个选择器筛选出来的标签, 给它加入样式。
       区别: 后代选择器可以定义 多个选择器（大于2个）， 而 子代选择器最多就两代

    3. CSS部分样式属性, 存在继承关系,详见参考: https://blog.csdn.net/qq_43553067/article/details/93519859

    4. 标签分为块级标签和行级标签
           块级: 定义多个相同的标签，每个标签独自沾满一行, 块级元素的盒子模型设置内边距和外边距对其他盒子都有影响
           行级: 定义多个相同的标签, 每个标签在一行显示
              在盒子模型中, 我们说 行级元素的上下 内边距和外边距是无效的，这个无效是指: 对自身会有样式的变化，对其他盒子是没有任何改变的。
-->
```

