# JavaScript基础

**学习网址**: https://www.runoob.com/js/js-tutorial.html

**高级：**https://www.runoob.com/jsref/prop-document-activeelement.html

### 今日目标

- 所有**案例准确吸收，最好盲敲**，其他知识点随用随查即可。



### 1. JavaScript 快速入门

**发展历程**

- 1995 年，NetScape (网景)公司，开发的一门客户端脚本语言：LiveScript。后来，请来SUN 公司的专家来 进行修改，后命名为：JavaScript。 
- 1996 年，微软抄袭 JavaScript 开发出 JScript 脚本语言。
- 1997 年，ECMA (欧洲计算机制造商协会)，制定出客户端脚本语言的标准：ECMAScript，统一了所有客户 端脚本语言的编码方式。

#### 1.1 JavaScript 介绍

- JavaScript 是一种客户端脚本语言。运行在客户端浏览器中，每一个浏览器都具备解析JavaScript 的引擎。 
- 脚本语言：不需要编译，就可以被浏览器直接解析执行了。 
- 核心功能就是增强用户和HTML 页面的交互过程，让页面有一些动态效果。以此来增强用户的体验！



#### 1.2 JS引入方式

- 内部方式 

  > html 为我们提供了 <script>标签，它是一个围堵标签，我们可以将JavaScript代码编写在<script>标签中。

- 外部方式 

  > JavaScript也有自己的文件，后缀为.js 文件，我们可以将JavaScript代码编写在.js 文件中，如果.html 文件需要该.js 文件的话，可以使用 <script src=文件路径> 来引入，**注意：**如果<script src=文件路径> 标签引入了js文件，那么它将不能在用**内部方式**编写JS代码，可以在定义一个script标签使用。

```javascript
//a.js
alert("我是外部的js文件");
```

```html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title >Title</title>
    <!--
       内部JS
       外部JS
   -->
</head>
<body>
<input type="text">

</body>

<script>
    alert("Hello World");
</script>

<script src="js/a.js"></script>

</html>
```

#### 1.3 快速入门小结

- JavaScript 是一种客户端脚本语言。 

- 组成部分 ECMAScript、DOM、BOM 

- 和 HTML 结合方式 

  > 内部方式：<script></script> 
  >
  > 外部方式：<script src=文件路径></script>



### 2. JavaScript 基本语法 (重点)

#### 2.1 注释

- 单行注释      // 注释的内容 
- 多行注释      /* 注释的内容 */
- idea快捷键:  ctrl + /     或者 ctrl + shift + /

#### 2.2 输入输出语句

- 输入框 prompt(“提示内容”);
- 弹出警告框 alert(“提示内容”); 
- 控制台输出 console.log(“显示内容”); 
- 页面内容输出 document.write(“显示内容”);

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>输入输出语句</title>
</head>
<body>
    
</body>
<script>
    //1.输入框
    //prompt("请输入数据");

    //2.弹出警告框
    //alert("hello");

    //3.控制台输出
    //console.log("hello js");

    //4.页面内容输出
    document.write("hello js");
    document.write("<br>");
    document.write("hello js");
</script>
</html>
```

#### 2.3 变量和常量

- JavaScript 属于弱类型的语言，定义变量时不区分具体的数据类型。

- 定义局部变量 

  > 高版本：let 变量名 = 值;      let  name = "李四";
  >
  > 低版本：var 变量名 = 值；  var name = "张三";
  >
  > 

- 定义全局变量 

  > 变量名 = 值;    name = "王五";

- 定义常量 

  > const 常量名 = 值;   const price = 200;

**原始数据类型**

| 数据类型  | 说明                         |
| --------- | ---------------------------- |
| boolean   | 布尔类型，true或false        |
| null      | 声明null值的特殊关键字       |
| undefined | 代表变量未定义               |
| number    | 整数或浮点数                 |
| string    | 字符串                       |
| bigint    | 大整数，例如：let num = 10n; |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>变量和常量</title>
</head>
<body>
    
</body>
<script>
    //1.定义局部变量
    let name = "张三";
    let age = 23;
    document.write(name + "," + age +"<br>");

    //2.定义全局变量
    {
        let l1 = "aa";
        l2 = "bb";
    }
    //document.write(l1);
    document.write(l2 + "<br>");

    //3.定义常量
    const PI = 3.1415926;
    //PI = 3.15;
    document.write(PI);
</script>
</html>
```

**typeof 方法**

- typeof 是一个关键字，作用就是得到某个let关键字定义的变量具体是哪一种原始数据类型。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>typeof方法</title>
</head>
<body>
    
</body>
<script>
    let l1 = true;
    document.write(typeof(l1) + "<br>");    // boolean

    let l2 = null;
    document.write(typeof(l2) + "<br>");    //    object  js原始的一个错误

    let l3;
    document.write(typeof(l3) + "<br>");    // undefined

    let l4 = 10;
    document.write(typeof(l4) + "<br>");    // number

    let l5 = "hello";
    document.write(typeof(l5) + "<br>");    // string

    let l6 = 100n;
    document.write(typeof(l6) + "<br>");    // bigint
</script>
</html>
```



#### 2.4 运算符

##### 算数运算符

| 运算符 | 说明     |
| ------ | -------- |
| +      | 加法运算 |
| -      | 减法运算 |
| *      | 乘法运算 |
| /      | 除法运算 |
| %      | 取余数   |
| ++     | 自增     |
| --     | 自减     |

##### **赋值运算符**

| 运算符 | 说明                               |
| ------ | ---------------------------------- |
| =      | 将等号右边的值赋值给等号左边的变量 |
| +=     | 相加后赋值                         |
| -=     | 相减后赋值                         |
| *=     | 相乘后赋值                         |
| /=     | 相除后赋值                         |
| %=     | 取余数后赋值                       |

##### 比较运算符

| 运算符 | 说明                     |
| ------ | ------------------------ |
| ==     | 判断值是否相等           |
| ===    | 判断数据类型和值是否相等 |
| >      | 大于                     |
| >=     | 大于等于                 |
| <      | 小于                     |
| <=     | 小于等于                 |
| !=     | 不等于                   |

##### 逻辑运算符

| 运算符 | 说明               |
| ------ | ------------------ |
| &&     | 逻辑与，并且的功能 |
| \|\|   | 逻辑或，或者的功能 |
| !      | 取反               |

##### 三元运算符

- (比较表达式) ? 表达式1 : 表达式2;

  > 如果比较表达式为true，则取表达式1 
  >
  > 如果比较表达式为false，则取表达式2



##### 运算符-示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>运算符</title>
</head>
<body>
    
</body>
<script>
    let num = "10";
    document.write(num+ 5 + "<br>");   // 105
    document.write(num + "5" + "<br>"); // 105
    document.write(num - 5 + "<br>");   // 5
    document.write(num * 5 + "<br>");   // 50  字符串类型数字进行运算，会自动类型转换

    let num2 = 10;
    document.write(num == num2);    // true  == 只比较值是否相同
    document.write(num === num2);   // false === 比较数据类型和值是否相同
</script>
</html>
```



#### 2.5 流程控制和循环语句

- if 语句 
- switch 语句 
- for 循环 
- while 循环

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>流程控制和循环语句</title>
</head>
<body>
    
</body>
<script>
    //if语句
    let month = 3;
    if(month >= 3 && month <= 5) {
        document.write("春季");
    }else if(month >= 6 && month <= 8) {
        document.write("夏季");
    }else if(month >= 9 && month <= 11) {
        document.write("秋季");
    }else if(month == 12 || month == 1 || month == 2) {
        document.write("冬季");
    }else {
        document.write("月份有误");
    }

    document.write("<br>");

    //switch语句
    switch(month){
        case 3:
        case 4:
        case 5:
            document.write("春季");
            break;
        case 6:
        case 7:
        case 8:
            document.write("夏季");
            break;
        case 9:
        case 10:
        case 11:
            document.write("秋季");
            break;
        case 12:
        case 1:
        case 2:
            document.write("冬季");
            break;
        default:
            document.write("月份有误");
            break;
    }

    document.write("<br>");

    //for循环
    for(let i = 1; i <= 5; i++) {
        document.write(i + "<br>");
    }

    //while循环
    let n = 6;
    while(n <= 10) {
        document.write(n + "<br>");
        n++;
    }
</script>
</html>
```



#### 2.6 数组

- 数组的使用和 java 中的数组基本一致，但是在 JavaScript 中的数组更加灵活，数据类型和**长度**都没有限制。
- 定义格式 let 数组名 = [元素1,元素2,…]; 
- 索引范围 从 0 开始，最大到数组长度-1 
- 数组长度 数组名.length
- 数组高级运算符… 数组复制 合并数组 字符串转数组

##### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>数组</title>
</head>
<body>
    
</body>
<script>
    //定义数组
    let arr = [10,20,30];
    //arr[3] = 40;  js中的数组长度可变
    //遍历数组
    for(let i = 0; i < arr.length; i++) {
        document.write(arr[i] + "<br>");
    }
    document.write("==============<br>");

    // 数组高级运算符 ...
    //复制数组
    let arr2 = [...arr];
    //遍历数组
    for(let i = 0; i < arr2.length; i++) {
        document.write(arr2[i] + "<br>");
    }
    document.write("==============<br>");

    //合并数组
    let arr3 = [40,50,60];
    let arr4 = [...arr2 , ...arr3];
    //遍历数组
    for(let i = 0; i < arr4.length; i++) {
        document.write(arr4[i] + "<br>");
    }
    document.write("==============<br>");

    //将字符串转成数组--了解即可, 不实用。
    let arr5 = [..."heima"];
    //遍历数组
    for(let i = 0; i < arr5.length; i++) {
        document.write(arr5[i] + "<br>");
    }
</script>
</html>
```



#### 2.7 函数

- 函数类似于java 中的方法，可以将一些代码进行抽取，达到复用的效果。
- 定义格式 function 方法名(参数列表) { 方法体; return 返回值; } 
- 可变参数 function 方法名(…参数名) { 方法体; return 返回值; } 
- 匿名函数 function(参数列表) { 方法体; } 
- <font color=red>函数javascrpt 中 是对象，有名函数的名字就是对象的引用，匿名函数 = 号前的变量就是对象的引用</font>
- <font color=red>函数javascrpt 中 方法不存在重载, 下面的同名函数会覆盖调用上面的同名函数，无需考虑形参和返回值类型是否一致，名字一样就覆盖</font>

##### **示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>函数</title>
</head>
<body>
    
</body>
<script>
    //无参无返回值的方法
    function println(){
        document.write("hello js" + "<br>");
    }
    //调用方法
    println();

    //有参有返回值的方法
    function getSum(num1,num2){
        return num1 + num2;
    }
    //调用方法
    let result = getSum(10,20);
    document.write(result + "<br>");

    //可变参数  对n个数字进行求和
    function getSum(...params) {
        let sum = 0;
        for(let i = 0; i < params.length; i++) {
            sum += params[i];
        }
        return sum;
    }
    //调用方法
    let sum = getSum(10,20,30);
    document.write(sum + "<br>");

    //匿名函数
    let fun = function(){
        document.write("hello");
    }
    fun();
</script>
</html>
```



#### 2.8 基本语法小结

- 注释：单行//  多行/**/ 
- 输入输出语句：prompt()、alert()、console.log()、document.write() 
- 变量和常量：let、const 
- 数据类型：boolean、null、undefined、number、string、bigint 
- typeof 关键字：用于判断变量的数据类型 
- 运算符：算数、赋值、逻辑、比较、三元运算符 
- 流程控制和循环语句：if、switch、for、while 
- 数组：数据类型和长度没有限制，let 数组名 = [长度/元素] 
- 函数：类似方法，抽取代码，提高复用性



### 3. JavaScript DOM

####   3.1 DOM 介绍

- DOM(Document Object Model)：文档对象模型。 
- 将 HTML 文档的各个组成部分，封装为对象。借助这些对象，可以对 HTML 文档进行增删改查的动态操作。

- HTML 四个对象需要我们熟知：

  > Document：文档对象。 
  >
  > Element：元素对象。 
  >
  > Attribute：属性对象。 
  >
  > Text：文本对象。

#### 3.2 文档对象-Document

#####  获取元素对象Element

| 方法名                              | 说明<br/>                      |
| ----------------------------------- | ------------------------------ |
| getElementById(id属性值)            | 根据id获得一个元素<br/>        |
| getElementsByTagName(标签名称)      | 根据标签名称获得多个元素<br/>  |
| getElementsByName(name属性值)       | 根据name属性获得多个元素<br/>  |
| getElementsByClassName(class属性值) | 根据class属性获得多个元素<br/> |
| 子元素对象.parentElement属性        | 获取当前元素的父元素           |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>元素的获取</title>
</head>
<body>
    <div id="div1">div1</div>
    <div id="div2">div2</div>
    <div class="cls">div3</div>
    <div class="cls">div4</div>
    <input type="text" name="username"/>
</body>
<script>
    //1. getElementById()   根据id属性值获取元素对象
    let div1 = document.getElementById("div1");
    //alert(div1);

    //2. getElementsByTagName()   根据元素名称获取元素对象们，返回数组
    let divs = document.getElementsByTagName("div");
    //alert(divs.length);

    //3. getElementsByClassName()  根据class属性值获取元素对象们，返回数组
    let cls = document.getElementsByClassName("cls");
    //alert(cls.length);

    //4. getElementsByName()   根据name属性值获取元素对象们，返回数组
    let username = document.getElementsByName("username");
    //alert(username.length);

    //5. 子元素对象.parentElement属性   获取当前元素的父元素
    let body = div1.parentElement;
    alert(body);
</script>
</html>
```



#### 3.3 标签对象-Element

##### Element 对象的操作

- createElement(标签名) 该方法为 document对象调用的方法，放置在这里为了演示其他方法。

| 方法名                       | 说明                       |
| ---------------------------- | -------------------------- |
| createElement(标签名)        | 创建一个新元素             |
| appendChild(子元素)          | 将指定子元素添加到父元素中 |
| removeChild(子元素)          | 用父元素删除指定子元素     |
| replaceChild(新元素, 旧元素) | 用新元素替换子元素         |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>元素的增删改</title>
</head>
<body>
    <select id="s">
        <option>---请选择---</option>
        <option>北京</option>
        <option>上海</option>
        <option>广州</option>
    </select>
</body>
<script>
    //1. createElement()   创建新的元素
    let option = document.createElement("option");
    //为option添加文本内容
    option.innerText = "深圳";

    //2. appendChild()     将子元素添加到父元素中
    let select = document.getElementById("s");
    select.appendChild(option);

    //3. removeChild()     通过父元素删除子元素
    //select.removeChild(option);

    //4. replaceChild()    用新元素替换老元素
    let option2 = document.createElement("option");
    option2.innerText = "杭州";
    select.replaceChild(option2,option);

</script>
</html>
```



##### Element对象操作属性

- 每个HTML标签都有自己的固有属性，都可以操作，这块要去查找HTML在线手册，进行操作。
- 除了每个标签的固有属性外，还有一些属性来自于 JavaScript提供的，例如：style    className   innerHTML  innerText.......

| 方法名                       | 说明                          |
| ---------------------------- | ----------------------------- |
| setAtrribute(属性名, 属性值) | 设置属性                      |
| getAtrribute(属性名)         | 根据属性名获取属性值          |
| removeAtrribute(属性名)      | 根据属性名移除指定的属性      |
| style属性                    | 为元素添加样式                |
| className属性                | 为元素指定css类选择，添加样式 |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>属性的操作</title>
    <style>
        .aColor{
            color: blue;
        }
    </style>
</head>
<body>
    <a>点我呀</a>
</body>
<script>
    //1. setAttribute()    添加属性
    let a = document.getElementsByTagName("a")[0];
    a.setAttribute("href","https://www.baidu.com");

    //2. getAttribute()    获取属性
    let value = a.getAttribute("href");
    //alert(value);

    //3. removeAttribute()  删除属性
    //a.removeAttribute("href");

    //4. style属性   添加样式
    //a.style.color = "red";

    //5. className属性   添加指定样式
    a.className = "aColor";

</script>
</html>
```



##### Element对象操作文本

| 属性名    | 说明                     |
| --------- | ------------------------ |
| innerText | 添加文本内容，不解析标签 |
| innerHTML | 添加文本内容，解析标签   |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文本的操作</title>
</head>
<body>
    <div id="div"></div>
</body>
<script>
    //1. innerText   添加文本内容，不解析标签
    let div = document.getElementById("div");
    div.innerText = "我是div";
    //div.innerText = "<b>我是div</b>";

    //2. innerHTML   添加文本内容，解析标签
    div.innerHTML = "<b>我是div</b>";

</script>
</html>
```



#### 3.4 DOM 小结

- DOM(Document Object Model)：

  > 1）文档对象模型 Document：文档对象、
  >
  > 2）Element：元素对象、
  >
  > 3）Attribute：属性对象、
  >
  > 4）Text：文本对象 

- Document：文档对象

  > getElementById() 
  >
  > getElementsByTagName() 
  >
  > getElementsByName() 
  >
  > getElementsByClassName() 
  >
  > 子元素对象.parentElement属性 

- Element: 元素对象之间的操作

  > createElement() 
  >
  > appendChild() 
  >
  > removeChild() 
  >
  > replaceChild()
  >

- Element: 元素对象操作属性

  > setAtrribute() 
  >
  > getAtrribute() 
  >
  > removeAtrribute() 
  >
  > style属性 
  >
  > className属性

- Element: 元素对象操作文本

  > 不支持嵌套标签: innerText 
  >
  > 支持嵌套标签:  innerHTML



### 4. JavaScript 事件

在线网址：https://www.runoob.com/jsref/dom-obj-event.html

- 事件指的就是当某些组件执行了某些操作后，会触发某些代码的执行。

#### 4.1 常见的事件

| 事件名     | 说明                     |
| ---------- | ------------------------ |
| onload     | 某个页面或图像被完成加载 |
| onsubmit   | 当表单提交时触发该事件   |
| onclick    | 鼠标单击事件             |
| ondblclick | 鼠标双击事件             |
| onblur     | 元素失去焦点             |
| onfocus    | 元素获得焦点             |
| onchange   | 用户改变域的内容         |

#### 4.2 了解的事件

| 事件名      | 说明                     |
| ----------- | ------------------------ |
| onkeydown   | 某个键盘的键被按下       |
| onkeypress  | 某个键盘的键被按下或按住 |
| onkeyup     | 某个键盘的键被松开       |
| onmousedown | 某个鼠标按键被按下       |
| onmouseup   | 某个鼠标按键被松开       |
| onmouseover | 鼠标被移到某元素之上     |
| onmouseout  | 鼠标从某元素移开         |



#### 4.3 绑定事件

- 方式一 通过标签中的**事件属性**进行绑定。

- 方式二 通过 DOM 元素对象.事件属性绑定

  > document.getElementById("btn").onclick= 执行的功能

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>事件</title>
</head>
<body>
    <img id="img" src="img/01.png"/>
    <br>
    <!-- <button id="up" onclick="up()">上一张</button> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    <button id="down" onclick="down()">下一张</button> -->
    <button id="up">上一张</button> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    <button id="down">下一张</button>
</body>
<script>
    //显示第一张图片的方法
    function up(){
        let img = document.getElementById("img");
        img.setAttribute("src","img/01.png");
    }

    //显示第二张图片的方法
    function down(){
        let img = document.getElementById("img");
        img.setAttribute("src","img/02.png");
    }

    //为上一张按钮绑定单击事件
    let upBtn = document.getElementById("up");
    upBtn.onclick = up;

    //为下一张按钮绑定单击事件
    let downBtn = document.getElementById("down");
    downBtn.onclick = down;
</script>
</html>
```



### 5. JavaScript 综合案例

















### 6. 扩展

#### 6.1 原始数据类型-string

- 在js中，双引号  和 单引号 都表示string类型 例如:  

  > let name = “张三” ;
  >
  > let name = ‘李四’ ;

#### 6.2 +号运算符

- js 中 + 号在string类型中表示**拼接**，在number类型中表示 做 **加法**。

- 还有一种特殊的作用，就是 string 类型 转number，看如下示例代码。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>运算符</title>
</head>
<body>
    
</body>
<script>
    let num = "10";
    document.write((+num )+ 5 );   // 15
   
</script>
</html>
```



#### 6.3 流程控制分支

**switch**

- 因为js是弱类型语言，不同于java，switch 的 case 可以是任意类型的匹配

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>switch语句</title>
    <script>
        let a;
        switch (a){
            case 1:
                alert("number");
                break;
            case "abc":
                alert("string");
                break;
            case true:
                alert("true");
                break;
            case null:
                alert("null");
                break;
            case undefined:
                alert("undefined");
                break;

            default:

        }
    </script>
</head>
<body>

</body>
</html>
```

#### 6.4 数组

- 在js中，数组实际上是一个对象，我们可以new 关键字创建。[] 这种是快捷方式

- 既然是对象，肯定可以调用方法： 

  >  arr.push(11);   //push 方法表示向数组末尾追加一个元素

- let  与 var 都是定义变量的关键字，高版本中推荐使用let, 无需过多考虑二者区别。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Array对象</title>
    <script >

        /*
            Array:数组对象
                1. 创建：
                    1. var arr = new Array(元素列表);
                    2. var arr = new Array(默认长度);
                    3. var arr = [元素列表];
                2. 方法
                    join(参数):将数组中的元素按照指定的分隔符拼接为字符串
                    push()	向数组的末尾添加一个或更多元素，并返回新的长度。
                3. 属性
                    length:数组的长度
                4. 特点：
                    1. JS中，数组元素的类型可变的。
                    2. JS中，数组长度可变的。
         */
        //1.创建方式1
       /* let arr1 = new Array(1,2,3);
        var arr2 = new Array(5);
        var arr3 = [1,2,3,4];

        var arr4 = new Array();

        document.write(arr1 +"<br>");
        document.write(arr2 +"<br>");
        document.write(arr3 +"<br>");
        document.write(arr4 +"<br>");*/

        var arr = [1,"abc",true];

        document.write(arr +"<br>");

        document.write(arr[0] +"<br>");
        document.write(arr[1] +"<br>");
        document.write(arr[2] +"<br>");

        //document.write(arr[10] +"<br>");
        arr[10] = "hehe";
        document.write(arr[10] +"<br>");
        document.write(arr[9] +"<br>");

        //alert(arr.length);//11
        document.write(arr.join("--")+"<br>");
        arr.push(11);
        document.write(arr.join("--")+"<br>");

    </script>
</head>
<body>

</body>
</html>
```

#### 6.5 for遍历数组

- for(let i = 0; i < arr.length; i++)
- for(let value of arr)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>遍历数组</title>
</head>
<body>

<script>

    //定义数组
    let arr = [10,20,30, "苹果"];

    document.write("方式一==============<br>");
    //遍历数组
    for(let i = 0; i < arr.length; i++) {
        document.write(arr[i] + "<br>");
    }

    document.write("方式二==============<br>");
    for (let value of arr){
        document.write(value + "<br>");
    }

</script>

</body>
</html>
```



#### 6.6 函数-(重点)

- JavaScript给java中的方法，称之为 **Function 对象**

- C 语言 叫  函数，  Java  叫  方法 ,   JS最好叫做Function 对象

##### 函数的定义

```js
//定义方法就按照下方去定义
function name(a, b){

    //返回值想有就有，没有就拉到，看你喜欢
    return "sssss";
}

//佚名函数定义，function为关键字
var funName = function(){
    
    //返回值想有就有，没有就拉到，看你喜欢
    return "bbbbb";
}

```

##### 总结

- Function对象都是由  function 关键字来定义的。

- Function对象的引用就是方法的名字，比如上面实例代码中：

  > name 和 funName 就是该Function对象的引用。

<font color=red>注意事项: js 方法中没有方法的重载, 如果定义了多个同名方法, 则最后定义的那个 会覆盖掉上面所有定义的同名方法。</font>



#### 6.7 DOM-document对象

- document对象还可以结合css 选择器的语法，来查找标签对象Element

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>获取标签</title>
</head>
<body>
<ul>
    <li>阿里巴巴</li>
    <li>腾讯</li>
    <li>字节跳动</li>
    <li id="jd">京东</li>
</ul>

<input type="radio" name="sex" value="male">男
<input type="radio" name="sex" value="female">女


<script>
    /**
     * 操作标签：
     *      获取标签（查找标签）：想操作谁就要找到谁
     *          let element = document.querySelector("选择器")
     *              获取一个标签。如果选择器找到多个，那么只获取第一个
     *
     *          let elements  = document.querySelectorAll("选择器")
     *              获取一批标签。选择器选中多少个，就得到多少个
     *
     *      创建标签：
     *      插入标签：
     *      删除标签：
     */
    //1.要求：获取所有的li标签，使用标签选择器
    let items = document.querySelectorAll("li");
    for (let item of items) {
        console.log(item);
    }

    console.log("-----------------------");

    //2.要求：获取京东的li标签，使用id选择器
    let item = document.querySelector("#jd");
    console.log(item);

    let item2 = document.querySelector("li");
    console.log(item2);

    let ul = document.querySelector("ul");
    console.log(ul);

    let radois = document.querySelectorAll("input[name='sex']");
    for (let radio of radois) {
        console.log(radio);
    }
</script>
</body>
</html>
```



#### 6.8 DOM-练习

##### 生成下拉选项

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>生成下拉选项</title>
</head>
<body>

<input type="button" value="把数组里的市添加到下拉框" onclick="addOption()">

<br>
<select name="city" id="city">
    <option value="">--选择市---</option>
</select>

<script>
    let cityArr = ["深圳市", "广州市", "东莞市"];

    /**
     * 1. 需要使用js代码生成标签：<option></option>
     * 2. 需要把一些内容设置到标签体里，标签变成：<option>深圳市</option>
     * 3. 需要把创建出来的option标签，插入到city下拉框里
     *      需要获取到city下拉框
     *      需要给city下拉框插入一些内容
     *
     * 分析：
     *  1.使用什么事件可以知道按钮被点击了？单击事件onclick
     *  2.循环遍历数组cityArr
     *      每个城市的名称
     *      每个城市，需要创建一个option标签：<option></option>
     *      给option标签设置标签体内容：城市名称  <option>深圳市</option>
     *      把创建出来的option标签插入到city下拉框里：父标签对象.appendChild(子标签对象)
     *
     */

    function addOption(){
        let city = document.querySelector("#city");
        //把下拉框里原有的选项内容不要，只要一个提示选项 <option value="">--选择市---</option>
        //使用innerHTML，用提示选项<option value="">--选择市---</option>把原有内容覆盖掉
        city.innerHTML = "<option value=\"\">--选择市---</option>";


        //把数组里的城市插入到下拉框里
        for (let cityName of cityArr) {
            //创建一个option标签：<option></option>
            let opt = document.createElement("option");
            //设置标签体内容：城市名称 <option>深圳市</option>
            opt.innerHTML = cityName;
            //把创建出来的option标签插入到city下拉框里
            city.appendChild(opt);
        }
    }
</script>
</body>
</html>
```



#### 6.9 事件-练习

##### 显示密码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>显示密码</title>
</head>
<body>
<input type="password" id="password">
<input type="button" value="查看密码" onmousedown="showPassword()" onmouseup="hidePassword()">

<script>
    let password = document.querySelector("#password");
    //要求：鼠标按下“查看密码”按钮，显示密码；onmousedown
    //     鼠标弹起“查看密码”按钮，隐藏密码  onmouseup
    function showPassword() {
        //把密码框的type改成text
        password.type = "text";
    }
    function hidePassword() {
        //把密码框的type重新改成password
        password.type = "password";
    }
</script>
</body>
</html>
```

##### 点灯开关

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>电灯开关</title>

</head>
<body>

<img id="light" src="img/off.gif">


</body>

<script>
    /*
        分析：
            1.获取图片对象
            2.绑定单击事件
            3.每次点击切换图片
                * 规则：
                    * 如果灯是开的 on,切换图片为 off
                    * 如果灯是关的 off,切换图片为 on
                * 使用标记flag来完成

     */

    //1.获取图片对象
    var light = document.getElementById("light");

    //2.绑定单击事件
    light.onclick = changeImg;

    //用来记录当前 灯的状态是：开 or 关
    var flag = "off";//代表灯是灭的。 off图片

    function changeImg(){

        if(flag == "on"){//判断如果灯是开的，则灭掉
            light.src = "img/off.gif";
            flag = "off";

        }else{
            //如果灯是灭的，则打开
            light.src = "img/on.gif";
            flag = "on";
        }
    }

</script>

</html>
```

##### 表格全选

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格全选</title>
    <style>
        table{
            border: 1px solid;
            width: 500px;
            margin-left: 30%;
        }

        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            margin-top: 10px;
            margin-left: 30%;
        }

        .out{
            background-color: white;
        }
        .over{
            background-color: pink;
        }
    </style>

  <script>
      /*
        分析：
            1.全选：
                * 获取所有的checkbox
                * 遍历cb，设置每一个cb的状态为选中  checked

       */

      //1.在页面加载完后绑定事件
      window.onload = function(){

          //2.给全选按钮绑定单击事件
          document.getElementById("selectAll").onclick = function(){
                //全选
                //1.获取所有的checkbox
                var cbs = document.getElementsByName("cb");
                //2.遍历
                  for (var i = 0; i < cbs.length; i++) {
                      //3.设置每一个cb的状态为选中  checked
                      cbs[i].checked = true;
                  }
          }

          document.getElementById("unSelectAll").onclick = function(){
              //全不选
              //1.获取所有的checkbox
              var cbs = document.getElementsByName("cb");
              //2.遍历
              for (var i = 0; i < cbs.length; i++) {
                  //3.设置每一个cb的状态为未选中  checked
                  cbs[i].checked = false;
              }
          }

          document.getElementById("selectRev").onclick = function(){
              //反选
              //1.获取所有的checkbox
              var cbs = document.getElementsByName("cb");
              //2.遍历
              for (var i = 0; i < cbs.length; i++) {
                  //3.设置每一个cb的状态为相反
                  cbs[i].checked = !cbs[i].checked;
              }
          }

          document.getElementById("firstCb").onclick = function(){
              //第一个cb点击
              //1.获取所有的checkbox
              var cbs = document.getElementsByName("cb");
              //获取第一个cb

              //2.遍历
              for (var i = 0; i < cbs.length; i++) {
                  //3.设置每一个cb的状态和第一个cb的状态一样
                  cbs[i].checked =  this.checked;
              }
          }

          //给所有tr绑定鼠标移到元素之上和移出元素事件
          var trs = document.getElementsByTagName("tr");
          //2.遍历
          for (var i = 0; i < trs.length; i++) {
              //移到元素之上
              trs[i].onmouseover = function(){
                  // this.className = "over";
                  trs[i].className = "over";
              }

              //移出元素
              trs[i].onmouseout = function(){
                     this.className = "out";
              }

          }

      }



  </script>

</head>
<body>

<table>
    <caption>学生信息表</caption>
    <tr  class="">
        <th><input type="checkbox" name="cb" id="firstCb"></th>
        <th>编号</th>
        <th>姓名</th>
        <th>性别</th>
        <th>操作</th>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb" ></td>
        <td>1</td>
        <td>令狐冲</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb" ></td>
        <td>2</td>
        <td>任我行</td>
        <td>男</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

    <tr>
        <td><input type="checkbox"  name="cb" ></td>
        <td>3</td>
        <td>岳不群</td>
        <td>?</td>
        <td><a href="javascript:void(0);">删除</a></td>
    </tr>

</table>
<div>
    <input type="button" id="selectAll" value="全选">
    <input type="button" id="unSelectAll" value="全不选">
    <input type="button" id="selectRev" value="反选">
</div>
</body>
</html>
```



#### 6.10 动态表格案例-简单实现

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>动态表格</title>

    <style>
        table{
            border: 1px solid;
            margin: auto;
            width: 500px;
        }

        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            text-align: center;
            margin: 50px;
        }
    </style>

</head>
<body>

<div>
    <input type="text" id="name" placeholder="请输入姓名" autocomplete="off">
    <input type="text" id="age"  placeholder="请输入年龄" autocomplete="off">
    <input type="text" id="gender"  placeholder="请输入性别" autocomplete="off">
    <input type="button" value="添加" id="add">
</div>

    <table id="tb">
        <caption>学生信息表</caption>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>性别</th>
            <th>操作</th>
        </tr>

        <tr>
            <td>张三</td>
            <td>23</td>
            <td>男</td>
            <td><a href="JavaScript:void(0);" onclick="drop(this)">删除</a></td>
        </tr>

        <tr>
            <td>李四</td>
            <td>24</td>
            <td>男</td>
            <td><a href="JavaScript:void(0);" onclick="drop(this)">删除</a></td>
        </tr>

    </table>

</body>
<script>
    //一、添加功能
    //使用InnerHTML实现动态表格的添加
    document.getElementById("add").onclick = function() {

        //获取三个文本框的内容
        var id = document.getElementById("id").value;
        var name = document.getElementById("name").value;
        var gender = document.getElementById("gender").value;

        //获取table
        var table = document.getElementsByTagName("table")[0];
        table.innerHTML += "<tr>\n" +
            "        <td>"+ id +"</td>\n" +
            "        <td>"+ name +"</td>\n" +
            "        <td>"+ gender +"</td>\n" +
            "        <td><a href=\"javascript:void(0);\" onclick=\"delTr(this);\" >删除</a></td>\n" +
            "    </tr>";
    }
    
    
    //二、删除的功能
    //1.为每个删除超链接标签添加单击事件的属性
    //2.定义删除的方法
    function drop(obj){
        //3.获取table元素
        let table = obj.parentElement.parentElement.parentElement;
        //4.获取tr元素
        let tr = obj.parentElement.parentElement;
        //5.通过table删除tr
        table.removeChild(tr);
    }
    
</script>
</html>
```

