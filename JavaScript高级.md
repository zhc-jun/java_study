# JavaScript高级

### 今日重点

- 所有案例**先理解，在盲敲**，能够玩转案例即可，其他所有知识点随用随查笔记。

### 1. JavaScript 面向对象 (了解)

- 在 Java 中我们学习过面向对象，核心思想是万物皆对象。在JavaScript 中同样也有面向对象。思想类似。

#### 1.1 类定义和使用

- 类中有属性：类的属性无需提前定义

- 类中有方法

  > 构造方法
  >
  > 普通方法
  >
  > 静态方法

- 注意：JavaScript中没有方法的重载，只要名称一样，就会覆盖。

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>类的定义和使用</title>
</head>
<body>
    
</body>
<script>
    //定义Person类
    class Person{

        //构造方法
        constructor(name,age){
            this.name = name;
            this.age = age;
        }

        //普通方法-show
        show(){
            document.write(this.name + "," + this.age + "<br>");
            // document.write(this.name + "," + this.age + "," +  Person.address + "<br>");
        }

        //静态方法-eat
        static eat(){
            document.write("吃饭...");
        }
    }

    //使用Person类
    let p = new Person("张三",23);
    //添加静态属性
    // Person.address = "南京传智黑马~";
    p.show();
    Person.eat();
</script>
</html>
```

#### 1.2 字面量定义类和使用 (敲一遍即可)

- <font color=red>后面玩到json的时候，会涉及到这种类</font>

-  类、方法、属性都为静态类型 

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>字面量定义类和使用</title>
</head>
<body>
    
</body>
<script>
    //定义person
    let person = {
        name : "张三",
        age : 23,
        hobby : ["听课","学习"],

        eat : function() {
            document.write("吃饭...");
        }
    };

    //使用person
    document.write(person.name + "," + person.age + "," + person.hobby[0] + "," + person.hobby[1] + "<br>");
    person.eat();
</script>
</html>
```



#### 1.3 继承

- 继承：让类与类产生子父类的关系，子类可以使用父类有权限的成员。
- 继承关键字：extends 
- 顶级父类：Object

示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>继承</title>
</head>
<body>
    
</body>
<script>
    //定义Person类
    class Person{
        //构造方法
        constructor(name,age){
            this.name = name;
            this.age = age;
        }

        //eat方法
        eat(){
            document.write("吃饭...");
        }
    }

    //定义Worker类继承Person
    class Worker extends Person{
        constructor(name,age,salary){
            super(name,age);
            this.salary = salary;
        }

        show(){
            document.write(this.name + "," + this.age + "," + this.salary + "<br>");
        }
    }

    //使用Worker
    let w = new Worker("张三",23,10000);
    w.show();
    w.eat();
</script>
</html>
```



#### 1.4 面向对象小结

- 面向对象 把相关的数据和方法组织为一个整体来看待，从更高的层次来进行系统建模，更贴近事物的自然运行模式。
- 类的定义 class 类{} 字面量定义 
- 类的使用 let 对象名 = new 类名(); 对象名.变量名 对象名.方法名() 
- 继承 让类和类产生子父类关系，提高代码的复用性和维护性。 子类 extends 父类 Object 顶级父类



### 2. JavaScript 内置对象 （了解）

参考手册：https://www.runoob.com/jsref/prop-document-activeelement.html



#### 2.1 Number

- 基础时提到过，+号可以将string转成number
- **注意：如果string中有非数字字符，+号转换将变成NAN** (not a number  不是数字的数字类型);

| 方法名       | 说明<br/>                           |
| ------------ | ----------------------------------- |
| parseFloat() | 将传入的字符串浮点数转为浮点数<br/> |
| parseInt()   | 将传入的字符串整数转为整数          |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Number</title>
</head>
<body>
    
</body>
<script>
    //1. parseFloat()  将传入的字符串浮点数转为浮点数
    document.write(Number.parseFloat("3.14") + "<br>");

    //2. parseInt()    将传入的字符串整数转为整数
    document.write(Number.parseInt("100") + "<br>");
    document.write(Number.parseInt("200abc") + "<br>"); // 从数字开始转换，直到不是数字为止

</script>
</html>
```



#### 2.2 Math

- 重点掌握: floor   和 random 方法

| 方法名   | 说明                                       |
| -------- | ------------------------------------------ |
| ceil(x)  | 向上取整                                   |
| floor(x) | 向下取整                                   |
| round(x) | 把数四舍五入为最接近的整数                 |
| random() | 随机数,返回的是0.0-1.0之间范围(含头不含尾) |
| pow(x,y) | 幂运算 x的y次方                            |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Math</title>
</head>
<body>
    
</body>
<script>
    //1. ceil(x) 向上取整
    document.write(Math.ceil(4.4) + "<br>");    // 5
    
    //2. floor(x) 向下取整
    document.write(Math.floor(4.4) + "<br>");   // 4
    
    //3. round(x) 把数四舍五入为最接近的整数
    document.write(Math.round(4.1) + "<br>");   // 4
    document.write(Math.round(4.6) + "<br>");   // 5
    
    //4. random() 随机数,返回的是0.0-1.0之间范围(含头不含尾)
    document.write(Math.random() + "<br>"); // 随机数
    
    //5. pow(x,y) 幂运算 x的y次方
    document.write(Math.pow(2,3) + "<br>"); // 8
</script>
</html>
```

#### 2.3 Date

| 构造方法                                                   | 说明                             |
| ---------------------------------------------------------- | -------------------------------- |
| Date()                                                     | 根据当前时间创建对象             |
| Date(value)                                                | 根据指定毫秒值创建对象           |
| Date(year,month,[day,hours,minutes,seconds ,milliseconds]) | 根据指定字段创建对象(月份是0~11) |

| 成员方法         | 说明                           |
| ---------------- | ------------------------------ |
| getFullYear()    | 获取年份                       |
| getMonth()       | 获取月份                       |
| getDate()        | 获取天数                       |
| getHours()       | 获取小时                       |
| getMinutes()     | 获取分钟                       |
| getSeconds()     | 获取秒数                       |
| getTime()        | 返回据1970年1月1日至今的毫秒数 |
| toLocaleString() | 返回本地日期格式的字符串       |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Date</title>
</head>
<body>
    
</body>
<script>
    //构造方法
    //1. Date()  根据当前时间创建对象
    let d1 = new Date();
    document.write(d1 + "<br>");

    //2. Date(value) 根据指定毫秒值创建对象
    let d2 = new Date(10000);
    document.write(d2 + "<br>");

    //3. Date(year,month,[day,hours,minutes,seconds,milliseconds]) 根据指定字段创建对象(月份是0~11)
    let d3 = new Date(2222,2,2,20,20,20);
    document.write(d3 + "<br>");

    //成员方法
    //1. getFullYear() 获取年份
    document.write(d3.getFullYear() + "<br>");

    //2. getMonth() 获取月份
    document.write(d3.getMonth() + "<br>");

    //3. getDate() 获取天数
    document.write(d3.getDate() + "<br>");

    //4. toLocaleString() 返回本地日期格式的字符串
    document.write(d3.toLocaleString());
</script>
</html>
```

#### 2.4 String

| 构造方法         | 说明                   |
| ---------------- | ---------------------- |
| String(value)    | 根据指定字符串创建对象 |
| let s = "字符串" | 直接赋值               |

| 成员方法             | 说明                                    |
| -------------------- | --------------------------------------- |
| length               | 属性 获取字符串长度                     |
| charAt(index)        | 获取指定索引处的字符                    |
| indexOf(value)       | 获取指定字符串出现的索引位置,找不到为-1 |
| substring(start,end) | 根据指定索引范围截取字符串(含头不含尾)  |
| split(value)         | 根据指定规则切割字符串，返回数组        |
| replace(old,new)     | 使用新字符串替换老字符串                |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>String</title>
</head>
<body>
    
</body>
<script>
    //1. 构造方法创建字符串对象
    let s1 = new String("hello");
    document.write(s1 + "<br>");

    //2. 直接赋值
    let s2 = "hello";
    document.write(s2 + "<br>");

    //属性
    //1. length   获取字符串的长度
    document.write(s2.length + "<br>");

    //成员方法
    //1. charAt(index)     获取指定索引处的字符
    document.write(s2.charAt(1) + "<br>");

    //2. indexOf(value)    获取指定字符串出现的索引位置
    document.write(s2.indexOf("l") + "<br>");

    //3. substring(start,end)   根据指定索引范围截取字符串(含头不含尾)
    document.write(s2.substring(2,4) + "<br>");

    //4. split(value)   根据指定规则切割字符串，返回数组
    let s3 = "张三,23,男";
    let arr = s3.split(",");
    for(let i = 0; i < arr.length; i++) {
        document.write(arr[i] + "<br>");
    }

    //5. replace(old,new)   使用新字符串替换老字符串
    let s4 = "你会不会跳伞啊？让我落地成盒。你妹的。";
    let s5 = s4.replace("你妹的","***");
    document.write(s5 + "<br>");
</script>
</html>
```

#### 2.5 RegExp

- 正则表达式：是一种对字符串进行匹配的规则。
- 创建正则对象时，**推荐使用**  let reg = /^规则$/  这种方式。

| 构造方法           | 说明                 |
| ------------------ | -------------------- |
| RegExp(规则)       | 根据指定规则创建对象 |
| let reg = /^规则$/ | 直接赋值             |

| 成员方法           | 说明                           |
| ------------------ | ------------------------------ |
| test(匹配的字符串) | 根据指定规则验证字符串是否符合 |



**正则规则**

| 表达式 | 说明                 |
| ------ | -------------------- |
| [a]    | 只能是a              |
| [abc]  | 只能是abc中的某一个  |
| [1]    | 只能是1              |
| [123]  | 只能是123中的某一个  |
| [a-z]  | 可以是a到z中的某一个 |
| [A-Z]  | 可以是A到Z中的某一个 |
| [0-9]  | 可以是0到9中的某一个 |
| {5}    | 只能出现5次          |
| {4,6}  | 只能出现4到6次       |



**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RegExp</title>
</head>
<body>
    
</body>
<script>
    //1.验证手机号
    //规则：第一位1，第二位358，第三到十一位必须是数字。总长度11
    let reg1 = /^[1][358][0-9]{9}$/;
    document.write(reg1.test("18688888888") + "<br>");

    //2.验证用户名
    //规则：字母、数字、下划线组成。总长度4~16
    let reg2 = /^[a-zA-Z_0-9]{4,16}$/;
    document.write(reg2.test("zhang_san123"));
</script>
</html>
```

#### 2.6 Array

| 成员方法       | 说明                     |
| -------------- | ------------------------ |
| push(元素)     | 添加元素到数组的末尾     |
| pop()          | 删除数组末尾的元素       |
| shift()        | 删除数组最前面的元素     |
| includes(元素) | 判断数组是否包含给定的值 |
| reverse()      | 反转数组中的元素         |
| sort()         | 对数组元素进行排序       |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Array</title>
</head>
<body>
    
</body>
<script>

    let arr = [1,2,3,4,5];

    //1. push(元素)    添加元素到数组的末尾
    arr.push(6);
    document.write(arr + "<br>");

    //2. pop()         删除数组末尾的元素
    arr.pop();
    document.write(arr + "<br>");

    //3. shift()       删除数组最前面的元素
    arr.shift();
    document.write(arr + "<br>");

    //4. includes(元素)  判断数组中是否包含指定的元素
    document.write(arr.includes(2) + "<br>");

    //5. reverse()      反转数组元素
    arr.reverse();
    document.write(arr + "<br>");

    //6. sort()   对数组元素排序
    arr.sort();
    document.write(arr + "<br>");

</script>
</html>
```

#### 2.7 Set

- JavaScript 中的 Set 集合，元素唯一，存取顺序一致。 

| 构造方法 | 说明            |
| -------- | --------------- |
| Set()    | 创建Set集合对象 |

| 成员方法     | 说明             |
| ------------ | ---------------- |
| add(元素)    | 向集合中添加元素 |
| size属性     | 获取集合长度     |
| keys()       | 获取迭代器对象   |
| delete(元素) | 删除指定元素     |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Set</title>
</head>
<body>
    
</body>
<script>
    // Set()   创建集合对象
    let s = new Set();

    // add(元素)   添加元素
    s.add("a");
    s.add("b");
    s.add("c");
    s.add("c");

    // size属性    获取集合的长度
    document.write(s.size + "<br>"); 

    // keys()   获取迭代器对象
    let st = s.keys();
    for(let i = 0; i < s.size; i++){
        document.write(st.next().value + "<br>");
    }

    // delete(元素) 删除指定元素
    document.write(s.delete("c") + "<br>");
    let st2 = s.keys();
    for(let i = 0; i < s.size; i++){
        document.write(st2.next().value + "<br>");
    }
</script>
</html>
```

#### 2.8 Map

- JavaScript 中的 Map 集合，key 唯一，存取顺序一致。 

| 构造方法 | 说明            |
| -------- | --------------- |
| Map()    | 创建Map集合对象 |

| 成员方法       | 说明              |
| -------------- | ----------------- |
| set(key,value) | 向集合中添加元素  |
| size属性       | 获取集合长度      |
| get(key)       | 根据key获取value  |
| entries()      | 获取迭代器对象    |
| delete(key)    | 根据key删除键值对 |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Map</title>
</head>
<body>
    
</body>
<script>
    // Map()   创建Map集合对象
    let map = new Map();

    // set(key,value)  添加元素
    map.set("张三",23);
    map.set("李四",24);
    map.set("李四",25);

    // size属性     获取集合的长度
    document.write(map.size + "<br>");

    // get(key)     根据key获取value
    document.write(map.get("李四") + "<br>");

    // entries()    获取迭代器对象
    let et = map.entries();
    for(let i = 0; i < map.size; i++){
        // alert(et.next().value)
        document.write(et.next().value + "<br>");
    }

    // delete(key)  根据key删除键值对
    document.write(map.delete("李四") + "<br>");
    let et2 = map.entries();
    for(let i = 0; i < map.size; i++){
        document.write(et2.next().value + "<br>");
    }
</script>
</html>
```

#### 2.9 JSON

- JSON(JavaScript Object Notation)：是一种轻量级的数据交换格式。 
- 它是基于 ECMAScript 规范的一个子集，采用完全独立于编程语言的文本格式来存储和表示数据。
- 简洁和清晰的层次结构使得JSON 成为理想的数据交换语言。易于人阅读和编写，同时也易于计算机解析和 生成，并有效的提升网络传输效率。

| 成员方法        | 说明                           |
| --------------- | ------------------------------ |
| stringify(对象) | 将指定对象转换为json格式字符串 |
| parse(字符串)   | 将指定json格式字符串解析成对象 |

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JSON</title>
</head>
<body>
    
</body>
<script>
    //定义天气对象
    let weather = {
        city : "北京",
        date : "2088-08-08",
        wendu : "10° ~ 23°",
        shidu : "22%"
    };

    //1.将天气对象转换为JSON格式的字符串
    let str = JSON.stringify(weather);
    document.write(str + "<br>");

    //2.将JSON格式字符串解析成JS对象
    let weather2 = JSON.parse(str);
    document.write("城市：" + weather2.city + "<br>");
    document.write("日期：" + weather2.date + "<br>");
    document.write("温度：" + weather2.wendu + "<br>");
    document.write("湿度：" + weather2.shidu + "<br>");
</script>
</html>
```

#### 2.10 表单校验-案例

- 案例分析和实现

  > 1）为表单绑定提交事件(true提交、false不提交)。 
  >
  > 2）获取用户名和密码。 
  >
  > 3）判断用户名是否满足条件(4到16位纯字母)。 
  >
  > 4）判断密码是否满足条件(6位纯数字)。

- <font color=red>注意</font>：如果使用 <form onsubmit="sub();"> 这种方式绑定提交事件，如果想要控制表单是否提交，需要改写为：<form onsubmit="sub();">，如果sub() 返回的是false，则表单不提交。

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>表单校验</title>
    <link rel="stylesheet" href="css/style.css"></link>
</head>
<body>
    <div class="login-form-wrap">
        <h1>黑马程序员</h1>
        <form class="login-form" action="#" id="regist" method="get" autocomplete="off">
            <label>
                <input type="text" id="username" name="username" placeholder="Username..." value="">
            </label>
            <label>
                <input type="password" id="password" name="password" placeholder="Password..." value="">
            </label>
            <input type="submit" value="注册">
        </form>
    </div>
</body>
<script>
    //1.为表单绑定提交事件
    document.getElementById("regist").onsubmit = function() {
        //2.获取填写的用户名和密码
        let username = document.getElementById("username").value;
        let password = document.getElementById("password").value;

        //3.判断用户名是否符合规则  4~16位纯字母
        let reg1 = /^[a-zA-Z]{4,16}$/;
        if(!reg1.test(username)) {
            alert("用户名不符合规则，请输入4到16位的纯字母！");
            return false;
        }

        //4.判断密码是否符合规则  6位纯数字
        let reg2 = /^[0-9]{6}$/;
        //let reg2 = /^\d{6}$/;
        if(!reg2.test(password)) {
            alert("密码不符合规则，请输入6位纯数字的密码！");
            return false;
        }

        //5.如果所有条件都不满足，则提交表单
        return true;
    }
    
</script>
</html>
```



### 3. JavaScript BOM (多练练)

- BOM(Browser Object Model)：浏览器对象模型。
- 将浏览器的各个组成部分封装成不同的对象，方便我们进行操作。

#### 3.1 Window 窗口对象

- 定时器 唯一标识 

  > setTimeout(功能，毫秒值)：设置一次性定时器。 
  >
  > clearTimeout(标识)：取消一次性定时器。 唯一标识 
  >
  > setInterval(功能，毫秒值)：设置循环定时器。 
  >
  > clearInterval(标识)：取消循环定时器。

- 加载事件 window.onload：在页面加载完毕后触发此事件的功能。

- window对象可以省略不写，例如：

  > window.alert() 可以简写为：alert();

- window对象还可以通过属性获取其他BOM对象和DOM对象，比如：

  > 其他BOM对象: window.location;  window.history; window.document;

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>window窗口对象</title>
    <script>
        //一、定时器
        function fun(){
            alert("该起床了！");
        }
    
        //设置一次性定时器
        //let d1 = setTimeout("fun()",3000);
        //let d1 = setTimeout(fun,3000);
        //取消一次性定时器
        //clearTimeout(d1);
    
        //设置循环定时器
        //let d2 = setInterval("fun()",3000);
        //取消循环定时器
        //clearInterval(d2);
    
        //加载事件
        window.onload = function(){
            let div = document.getElementById("div");
            alert(div);
        }
    </script>
</head>
<body>
    <div id="div">dddd</div>
</body>
</html>
```



#### 3.2 Location 地址栏对象

- href 属性 

  > 就是浏览器的地址栏。我们可以通过为该属性设置新的URL，使浏览器读取并显示新的URL 的内容。

- reload() 函数

  > 刷新当前页面

**示例代码**

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Location对象</title>

</head>
<body>
    <input type="button" id="btn" value="刷新">

    <input type="button" id="goItcast" value="去传智">


    <script>
        //reload方法，定义一个按钮，点击按钮，刷新当前页面
        //1.获取按钮
        var btn = document.getElementById("btn");
        //2.绑定单击事件
        btn.onclick = function(){
            //3.刷新页面
            location.reload();
        }

        //点击按钮，去访问www.itcast.cn官网
        //1.获取按钮
        var goItcast = document.getElementById("goItcast");
        //2.绑定单击事件
        goItcast.onclick = function(){
            //3.去访问www.itcast.cn官网
            location.href = "https://www.baidu.com";
            // window.open("https://www.baidu.com", "_self");
        }

    </script>
</body>
</html>
```

##### location对象练习

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>location地址栏对象</title>
    <style>
        p{
            text-align: center;
        }
        span{
            color: red;
        }
    </style>
</head>
<body>
    <p>
        注册成功！<span id="time">5</span>秒之后自动跳转到首页...
    </p>
</body>
<script>
    //1.定义方法。改变秒数，跳转页面
    let num = 5;
    let intervalId;
    function showTime() {
        num--;

        if(num <= 0) {
            clearInterval(intervalId)
            //跳转首页
            // location.href = "index.html";
            // 谷歌网址: https://www.google.com/  https://www.baidu.com
            location.href = "https://www.baidu.com";
            return;
        }

        let span = document.getElementById("time");
        span.innerHTML = num;
    }

    //2.设置循环定时器，每1秒钟执行showTime方法
    intervalId = setInterval("showTime()",1000);
</script>
</html>
```

#### 3.3 案例 – 动态广告

##### 案例分析和实现

- 在 css 样式中，display 属性可以控制元素是否显示 

  > display : none     不显示 
  >
  > display : block     显示 

- 设置定时器，3 秒后显示广告图片
- 设置定时器，3 秒后隐藏广告图片

##### 代码实现

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>头条页面</title>
    <link rel="stylesheet" href="css/news.css"/>
</head>
<body>
    <!-- 广告图片 -->
    <img src="img/ad_big.jpg" id="ad_big" width="100%" style="display: none;"/>

    <!--顶部登录注册更多-->
    <div class="top">
        <a href="#" target="_self">登录&nbsp;&nbsp;</a>
        <a href="#">注册&nbsp;&nbsp;</a>
        <a href="#">更多&nbsp;&nbsp;</a>
    </div>

    <!--导航条-->
    <div class="nav">
        <img src="img/logo.png"/>
        <a href="#">首页&nbsp;&nbsp;</a>/
        <a href="#">科技&nbsp;&nbsp;</a>/
        <font color="gray">正文</font>
        <hr/>
    </div>

    <!--左侧分享-->
    <div class="left">
        <a href="#"> <img src="img/cc.png"/> <span>&nbsp;评论</span> </a>
        <hr/>
        <a href="#"> <img src="img/repost.png"/> <span>&nbsp;转发</span> </a>  <br/>
        <a href="#"> <img src="img/weibo.png"/> <span>&nbsp;微博</span> </a>  <br/>
        <a href="#"> <img src="img/qq.png"/> <span>&nbsp;空间</span> </a>  <br/>
        <a href="#"> <img src="img/wx.png"/> <span>&nbsp;微信</span> </a>  <br/>
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
            <img src="img/1.jpg" width="100%"/>
            </p>
            <p>说起支付宝中的芝麻信用功能，相信更是受到了许多人的推崇，因为随着自己使用的不断增多，信用分会慢慢提高，而达到了一个阶段，就可以获得许多的福利。而当我们的芝麻信用分可以达到600分以上的时候，会有令我们想象不到的惊喜，接下来就让我们一起来看看，具体都有哪些惊喜吧。</p>
            <p><b>一、芝麻分600以上福利之信用购。</b>网购相信大家都不陌生，但是很多时候，网购都有一个通病，就是没办法试用，导致很多人买了很多自己不喜欢的东西。但是只要你的支付宝芝麻分在650及以上，就能立马享有0元下单，收到货使用满意了再进行付款。还能享用美食的专属优惠，是不是很耐斯</p>
            <p><b>二、芝麻分600以上福利之信用免押。</b>芝麻信用与木鸟短租联合推出信用住宿服务，芝麻分600及以上的用户可享受免押入住特权。木鸟短租拥有全国50万套房源，是国内领先的短租民宿预订平台。包括大家知道的飞猪信用住，大部分酒店可以免押金入住，离店再交钱。</p>
            <img src="img/2.jpg" width="100%"/>
            <p><b>三、芝麻分600以上福利之国际驾照。</b>我们经常听说的可能只是中国驾照，但现在芝麻分已经应用到了国际领域，只要你的芝麻分够550就可以免费办理国际驾照，也有不少人非常佩服马云，一个简单的芝麻分居然有如此大的功能，也从侧面反应出来马云在国际上的地位，这个国际驾照是由新西兰、德国、澳大利亚联合认证，可以在全球200多个国家通行，相信大家一定都有一个自驾全球的梦想吧，而现在支付宝就给了你一把钥匙，剩下的就你自己搞定了！有没有想带着你的女神来一次浪漫之旅呢？</p>
            <p>随着互联网对我们生活的改变越来越大，信用这一词也被大家推上风口浪尖，不论是生活出行，还是其他的互联网服务，与信用体系已经密不可分了，马云当初说道，找老婆需要拼芝麻分，如今似乎也要成为现实，那么你们的芝麻分有多少了呢？</p>
        </div>
    </div>

    <!--右侧广告图片-->
    <div class="right">
        <img src="img/ad1.jpg" width="100%"/>
        <img src="img/ad2.jpg" width="100%"/>
        <img src="img/ad3.jpg" width="100%"/>
        <img src="img/ad1.jpg" width="100%"/>
        <img src="img/ad2.jpg" width="100%"/>
        <img src="img/ad3.jpg" width="100%"/>
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
<script>
    //1.设置定时器，3秒后显示广告图片
    setTimeout(function(){
        let img = document.getElementById("ad_big");
        img.style.display = "block";
    },3000);

    //2.设置定时器，3秒后隐藏广告图片
    setTimeout(function(){
        let img = document.getElementById("ad_big");
        img.style.display = "none";
    },6000);
</script>
</html>
```

#### 3.4 BOM 小结

- BOM(Browser Object Model)：浏览器对象模型。 

- 将浏览器的各个组成部分封装成不同的对象，方便我们进行操作。 

  > Window：窗口对象 
  >
  > Location：地址栏对象 
  >
  > Navigator：浏览器对象 
  >
  > History：当前窗口历史记录对象 
  >
  > Screen：显示器屏幕对象 

- Window 窗口对象 

  > setTimeout()、clearTimeout()：一次性定时器 
  >
  > setInterval()、clearInterval()：循环定时器 
  >
  > onload 事件：页面加载完毕触发执行功能 

- Location 地址栏对象 

  > href 属性：跳转到指定的URL 地址
  >
  > reload()函数：刷新当前页面



### 4. JavaScript 封装

- 在企业中，尽可能的将JavaScript代码写在 .js 文件中，谁用谁引入接口。

<font color=red>我们之前的操作都是基于原生JavaScript 的，比较繁琐。 JQuery是一个前端框架技术，针对JavaScript 进行了一系列的封装，使得操作变得非常简单！ 期待吧……</font>

### 5.扩展

#### 5.1 正则表达式

正则语法教程：https://www.runoob.com/regexp/regexp-syntax.html

正则在线测试网页：https://c.runoob.com/front-end/854

##### 正则基础语法

```java
/**
    1. 正则表达式：定义字符串的组成规则。
        1. 单个字符出现范围:[]
            如： [a] [ab] [a-zA-Z0-9]
            * 特殊符号代表特殊含义的单个字符:
            \d:单个数字字符 [0-9]
            \w:单个单词字符[a-zA-Z0-9_]
    2. 数量词符号：
        ?：表示出现0次或1次
        *：表示出现0次或多次
        +：出现1次或多次
        {m,n}:表示 m<= 数量 <= n
            * m如果缺省： {,n}:最多n次
            * n如果缺省：{m,} 最少m次
            * 出现指定次数: {n} 只能出现n次
    3. 开始结束符号
        * ^:开始
        * $:结束
    2. 正则对象：
        1. 创建
            1. var reg = new RegExp("正则表达式");
            2. var reg = /正则表达式/;
        2. 方法	
            1. test(参数):验证指定的字符串是否符合正则定义的规范
**/
```

##### 校验手机号和邮箱

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RegExp</title>
</head>
<body>
    
</body>
<script> 
    //验证手机号   \d 是 [0-9] 的简写
    let regPhone = /^1(3|5|8)\d{9}$/;
    document.write(regPhone.test("18688888888") + "<br>");

    //验证邮箱 + 号为量词, 表示出现一次或多次
    let regEmail = /^\w+@\w+\.\w+$/;
    document.write(regEmail.test("zoulaoshi@itcast.cn") + "<br>");
    document.write(regEmail.test("937334241@qq.com") + "<br>");
    
    //必须出现数字+字母 5到10长度
    //(?![a-zA-Z]+$)[0-9A-Za-z]{5,10} 或者 (?![0-9]+$)[0-9A-Za-z]{5,10}

</script>
</html>
```



#### 5.2 BOM-History

- 切记：当前窗口的历史记录，不是整个浏览器的历史记录。

##### **04-History对象-前进.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>History对象</title>
</head>
<body>

<input type="button" id="btn" value="获取历史记录个数">
<a href="04-History对象-回退.html">04-History对象-回退.html</a>
<input type="button" id="forward" value="前进">
<script>
    /*
        History：历史记录对象
            1. 创建(获取)：
                1. window.history
                2. history

            2. 方法：
                * back()	加载 history 列表中的前一个 URL。
                * forward()	加载 history 列表中的下一个 URL。
                * go(参数)	    加载 history 列表中的某个具体页面。
                    * 参数：
                        * 正数：前进几个历史记录
                        * 负数：后退几个历史记录
            3. 属性：
                * length	返回当前窗口历史列表中的 URL 数量。


     */
    //1.获取按钮
    var btn = document.getElementById("btn");
    //2.绑定单机事件
    btn.onclick = function(){
        //3.获取当前窗口历史记录个数
        var length = history.length;
        alert(length);
    }


    //1.获取按钮
    var forward = document.getElementById("forward");
    //2.绑定单机事件
    forward.onclick = function(){
        //前进
        // history.forward();
        history.go(1);
    }


</script>

</body>
</html>
```

##### **04-History对象-回退.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>History对象</title>
</head>
<body>
<input type="button" id="forward" value="回退到--04-History对象-前进.html">
<script>
    /*
        History：历史记录对象
            1. 创建(获取)：
                1. window.history
                2. history

            2. 方法：
                * back()	加载 history 列表中的前一个 URL。
                * forward()	加载 history 列表中的下一个 URL。
                * go(参数)	    加载 history 列表中的某个具体页面。
                    * 参数：
                        * 正数：前进几个历史记录
                        * 负数：后退几个历史记录
            3. 属性：
                * length	返回当前窗口历史列表中的 URL 数量。

     */
    //1.获取按钮
    var forward = document.getElementById("forward");
    //2.绑定单机事件
    forward.onclick = function(){
        //前进
        // history.forward();
        history.go(-1);
    }

</script>

</body>
</html>
```



#### 5.3 指定位数随机字符串

- Math 对象 和 String 对象

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>生成指定位数的随机字符串</title>
</head>
<body>
<script>

    //指定随机字符串的位数
    var number = 4;
    
    //设定随机字符串可以出现的字符范围
    var str = "1023456789abcdefghijklmnopqrstuvwxyz";

    //调用方法获取随机数
    var sjs = getRandomStr(number);
    //将随机数进行打印
    document.write(sjs);

    /**
     *  该方法用来获取指定长度的随机字符串
     * @param length 长度
     * @returns {string|string}  随机字符串
     */
    function getRandomStr(length) {

        //用来接收每次循环产生的单个字符
        var string = "";
        for (var i = 0; i < length; i++) {

            //获取范围内的字符'随机' 下标
            var index = Math.floor(Math.random()* str.length);
            //根据index下标获取对应的单个字符
            var char = str.charAt(index);

            //进行字符串的拼接
            string = string.concat(char);
        }

        //将随机数返回回去
        return string;
    }
    
</script>

</body>
</html>
```



#### 5.4 轮播图

- setInterval()  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>轮播图</title>

</head>
<body>

    <img id="img" src="img/banner_1.jpg" width="100%">

    <script>
        /*
            分析：
                1.在页面上使用img标签展示图片
                2.定义一个方法，修改图片对象的src属性
                3.定义一个定时器，每隔3秒调用方法一次。
         */

        //修改图片src属性
        var number = 1;

        function fun(){
            number ++ ;
            //判断number是否大于3
            if(number > 3){
                number = 1;
            }
            //获取img对象
            var img = document.getElementById("img");
            img.src = "img/banner_"+number+".jpg";

        }

        //2.定义定时器
        setInterval(fun, 3000);

    </script>
</body>
</html>
```

#### 5.5 伪造a标签

- location 和 document对象； 事件: onmousedown(鼠标按下)   onmouseup(鼠标松开)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>span 模仿 a </title>

    <style>

        span{
            color: #551A8b;
            cursor:pointer;
        }

        .span_color{
            color: red;
        }

    </style>


</head>
<body>

<a href="https://www.baidu.com" >去百度</a> <br>

<span><u>去百度</u></span>

<script>

    var spanElement = document.getElementsByTagName("span")[0];
    spanElement.onmousedown = down;
    spanElement.onmouseup = up;

    //鼠标按下
    function down() {
        // spanElement.style.color = "red";
        spanElement.className = "span_color";

    }
    //鼠标松开
    function up() {
        spanElement.style.color = "#551A8b";

        //页面跳转
        window.location.href = "https://www.baidu.com";
        // window.open("https://www.baidu.com", "_self");
    }


</script>


</body>
</html>
```

