# Vue和ElementUI高级

###  1. Vue 高级使用 

#### 1.1 自定义组件 

- 学完了 Element 组件后，我们会发现组件其实就是自定义的标签。例如 <el-button>就是对<button>的封装。 

- 本质上，组件是带有一个名字且可复用的 Vue 实例，我们完全可以自己定义。 

- 定义格式 

  ```javascript
  Vue.component(组件名称, {         
      props:组件的属性,         
      data: 组件的数据函数,         
      template: 组件解析的标签模板 
  }) 
  ```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>自定义组件</title>
    <script src="vue/vue.js"></script>
</head>
<body>
    <div id="div">
        <my-button>我的按钮</my-button>
    </div>
</body>
<script>
    Vue.component("my-button",{
        // 自定义属性
        // props:["style"],
        // 数据函数
        data: function(){
            return{
                msg:"我的按钮"
            }
        },
        //解析标签模板
        template:"<button style='color:red'>{{msg}}</button>"
    });

    new Vue({
        el:"#div"
    });
</script>
</html>
```

#### 1.2  Vue 生命周期 

- 生命周期的八个阶段 

  |     状态      | 阶段周期 |
  | :-----------: | :------: |
  | beforeCreate  |  创建前  |
  |    created    |  创建后  |
  |  beforeMount  |  载入前  |
  |    mounted    |  载入后  |
  | beforeUpdate  |  更新前  |
  |    updated    |  更新后  |
  | beforeDestroy |  销毁前  |
  |   destroyed   |  销毁后  |

   

#### 1.3 Vue 异步操作 

- 在 Vue 中发送异步请求，本质上还是 AJAX。我们可以使用 axios 这个插件来简化操作！

-  使用步骤 

  > 1) 引入 axios 核心 js 文件。         
  >
  > 2) 调用 axios 对象的方法来发起异步请求。         
  >
  > 3) 调用 axios 对象的方法来处理响应的数据。 

- axios 常用方法 

  |             方法名              |                       作用                       |
  | :-----------------------------: | :----------------------------------------------: |
  | get(请求的资源路径与请求的参数) |                 发起GET方式请求                  |
  | post(请求的资源路径,请求的参数) |                 发起POST方式请求                 |
  |         then(response)          | 请求成功后的回调函数，通过response获取响应的数据 |
  |          catch(error)           |   请求失败后的回调函数，通过error获取错误信息    |

**示例代码**

TestServlet.java

```java
package com.itheima;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
@WebServlet("/testServlet")
public class TestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置请求和响应的编码
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");

        //获取请求参数
        String name = req.getParameter("name");
        System.out.println(name);

        //响应客户端
        resp.getWriter().write(name + "  你好, 我是 李四, 收到了你的请求成功");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req,resp);
    }
}

```

test.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>异步操作</title>
    <script src="js/vue.js"></script>
    <script src="js/axios-0.18.0.js"></script>
</head>
<body>
    <div id="div">
        {{name}}
        <button @click="mysend()">发起请求</button>
    </div>
</body>
<script>
    new Vue({
        el:"#div",
        data:{
            name:"张三"
        },
        methods:{
            mysend(){
                // GET方式请求
                axios.get("testServlet?name=" + this.name)
                    .then(resp => {
                        // alert(resp.data);
                        this.name = resp.data;
                    })
                    .catch(error => {
                        alert(error);
                    })

                // POST方式请求
                // axios.post("testServlet","name="+this.name)
                //     .then(resp => {
                //         // alert(resp.data);
                //         this.name = resp.data;
                //     })
                //     .catch(error => {
                //         alert(error);
                //     })
            }
        }
    });
</script>
</html>
```

#### 1.4 Vue 高级使用小结 

- 自定义组件：本质上，组件是带有一个名字且可复用的 Vue 实例，我们可以自己来定义。 

  ```
  Vue.component(组件名称, {         
      props:组件的属性,         
      data: 组件的数据函数,         
      template: 组件解析的标签模板 
  }) 
  ```

- 生命周期：核心八个阶段         

  > beforeCreate：创建前         
  >
  > created：创建后         
  >
  > beforeMount：载入前         
  >
  > mounted：载入后         
  >
  > beforeUpdate：更新前         
  >
  > updated：更新后         
  >
  > beforeDestroy：销毁前         
  >
  > destroyed：销毁后 
  > 

- axios 常用方法 

  |             方法名              |                       作用                       |
  | :-----------------------------: | :----------------------------------------------: |
  | get(请求的资源路径与请求的参数) |                 发起GET方式请求                  |
  | post(请求的资源路径,请求的参数) |                 发起POST方式请求                 |
  |         then(response)          | 请求成功后的回调函数，通过response获取响应的数据 |
  |          catch(error)           |   请求失败后的回调函数，通过error获取错误信息    |



###  2. 综合案例 学生管理系统 

#### 2.1 数据准备

```mysql
CREATE DATABASE vue;

USE vue;

-- 创建用户表
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,  -- 主键id
	username VARCHAR(10),               -- 账号
	PASSWORD VARCHAR(30)                -- 密码
);
INSERT INTO USER VALUES (NULL,'admin','123');

-- 创建学生表
CREATE TABLE student(
	number VARCHAR(10) UNIQUE,   -- 学号
	NAME VARCHAR(10),            -- 姓名
	birthday DATE,               -- 生日
	address VARCHAR(200)         -- 地址
);
INSERT INTO student VALUES ('hm001','张三','1995-05-05','北京市昌平区');
INSERT INTO student VALUES ('hm002','李四','1996-06-06','北京市海淀区');
INSERT INTO student VALUES ('hm003','王五','1997-07-07','北京市朝阳区');
INSERT INTO student VALUES ('hm004','赵六','1998-08-08','北京市丰台区');
INSERT INTO student VALUES ('hm005','周七','1999-09-09','北京市顺义区');
INSERT INTO student VALUES ('hm006','孙悟空','2000-01-01','花果山水帘洞');
INSERT INTO student VALUES ('hm007','猪八戒','2001-02-02','高老庄翠兰家');
INSERT INTO student VALUES ('hm008','沙和尚','2002-03-03','流沙河河底');
INSERT INTO student VALUES ('hm009','唐玄奘','2003-04-04','东土大唐');
```

#### 2.2 前端代码介绍

##### 2.2.1 列表标签

- :data 是 v-bind:data的简写，作用是给标签绑定属性，el-table标签是ElementUI自己定义的, :data也是它自己定义的
- :data 绑定属性一般都是去和new Vue({ data： {tableData: [] } }); 中的data 的（key）去做数据的绑定
- tableData:[], 它的value是一个js的数组对象，该数据中需要存放{} 对象，而且每个JSON对象中，必须要有key --> number  name  birthday  address，这样才能合理的展示出来。
- el-table标签会自动的去遍历tableData:[] 数组，取出来数组中每一项完成赋值，展示，我们只需要知道这些代码我们要遵循什么样的格式，以及new Vue({ data： {tableData: [] } }); 中的key命名要与标签属性对应上即可。
- template 标签的 slot-scope="props" 属性 可以获取到当前展示行对应的 JSON {} 对象; 有该对象，我们可以获取到对象中所有数据； "props" 该值可以随意命名，但是获取当前行的 json 对象时， 必须要 .row;

```html
<el-table :data="tableData">
    <el-table-column prop="number" label="学号" width="120">
    </el-table-column>
    <el-table-column prop="name" label="姓名" width="120">
    </el-table-column>
    <el-table-column prop="birthday" label="生日" width="140">
    </el-table-column>
    <el-table-column prop="address" label="地址">
    </el-table-column>
    <el-table-column label="操作" width="180">
        <template slot-scope="props">
            <el-button type="warning" @click="showEditStu(props.row)">编辑</el-button>
            <el-button type="danger" @click="deleteStu(props.row)">删除</el-button>
        </template>
    </el-table-column>
</el-table>

<script>
    new Vue({
        el:"#div",
        data:{
            dialogTableVisibleAdd: false,  //添加窗口显示状态
            dialogTableVisibleEdit: false, //编辑窗口显示状态
            addFormData:{},//添加表单的数据
            editFormData: {},//编辑表单的数据
            tableData:[],//表格数据需要被el-table 标签 data属性绑定
        }
    });
</script>
```

##### 2.2.2 分页标签

- el-pagination标签所有的属性都是官方提供的，我们只需要知道每个属性可以给什么值，有什么作用即可。

- layout="total, sizes, prev, pager, next, jumper" 该属性的值都是固定的，我们只能根据需要作出适当的删减，其他属性的值我们可以随意给定。

- 下方的属性值都是我们自己定义的, 特别要注意的是: handleSizeChange   handleCurrentChange 是回调函数的名字，elementUI在调用该函数时，会将**当前页码**和**每页展示条数**的值作为参数传递给我们。:current-page  :page-size :total    :page-sizes这四个属性的值，来自于我们 new Vue({ data： {pagination: {} } }); 中的data下面的pagination， 关于 :page-sizes="[3,5,8]" 属性我们只能对数组中的 3 5 8 去做出调整，其他都不可以动 

  >    @size-change="handleSizeChange"
  >    @current-change="handleCurrentChange"
  >    :current-page="pagination.currentPage"
  >
  > :page-sizes="pagination.pageSizes"   
  >
  > :page-size="pagination.pageSize"
  > :total="pagination.total"



```html
<!--
分页组件
    @size-change： 当改变每页展示条数时, 会触发的函数, 触发时，会自动传递当前展示条数值
    @current-change：当改变页码时触发的函数, 触发时，会自动传递改变后的页码值
    current-page ：当前选中的页码
    :page-sizes：每页条数选择框中显示的值
    :page-size : 默认的每页条数
    layout： 分页组件的布局: value值固定写法, 可以根据需要适当删减
    total（总条数）, sizes(每页条数), prev（上一页）, pager(所有的页码), next(下一页), jumper（跳转页码）
    :total: 总条数
-->
<el-pagination
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"
    :current-page="pagination.currentPage"
    :page-sizes="pagination.pageSizes"
    :page-size="pagination.pageSize"
    layout="total, sizes, prev, pager, next, jumper"
    :total="pagination.total">
</el-pagination>

<script>
    new Vue({
        el:"#div",
        data:{
            pagination: {
                currentPage: 1, //当前页
                pageSize: 5,    //每页显示条数
                total: 0,        //总条数
                pageSizes: [3,5,8] //每页展示条数选项
            }
        }
        methods:{
            //改变每页条数时执行的函数
            handleSizeChange(pageSize) {
                //修改分页查询的参数
                this.pagination.pageSize = pageSize;
                //重新执行查询
                this.selectByPage();
            },
            //改变页码时执行的函数
            handleCurrentChange(pageNum) {
                //修改分页查询的参数
                this.pagination.currentPage = pageNum;
                //重新执行查询
                this.selectByPage();
            }
        }

    });
</script>
```

##### 2.2.3 弹框标签嵌套表单标签

- el-dialog 标签是ElementUI提供的弹框标签，:visible.sync 该属性的value如果为true 就展示弹框，如果为false就不展示弹框, 在下方代码我们通过: 号 将该属性的具体值绑定在了 new Vue({ data： {dialogTableVisibleAdd: {} } }); 中，好处就是可以通过js来动态的把控该标签是否展示。
- el-dialog 标签 的 @close属性绑定了一个函数，当弹框被关闭时，ElementUI会调用该函数，在调用时，我们把该弹框嵌套的el-form 的ref属性值，传递了过去，便于对标签进行重置，这里的ref 好比我们html 标签中的id属性。
- el-form 是表单标签，:model="addFormData" 是给该标签的model 属性绑定值，其值来自于new Vue({ data： {addFormData: [] } });   
- el-form 标签的  :rules="rules"属性用来绑定表单校验的规则，其值来自于new Vue({ rules：{} } });  注意: {required: true, message: '请输入学号', trigger: 'blur'}  中的 key : required     message   trigger 是不可以随意变更名字的，这个ElementUI规定死的。ref="addForm" 该属性类似id, 确保唯一和见名知意即可。
- el-form 标签的 表单项标签 el-input 的  v-model="addFormData.number" 属性，是在做双向数据绑定
- **修改和添加功能在标签的使用上基本一致。**

```html
<el-dialog title="添加学生信息" :visible.sync="dialogTableVisibleAdd" @close="resetForm('addForm')">
    <!--
        :model="formData": 关联数据
        :rules: 关联校验规则
        ref： 在获取表单对象时使用
    -->
    <el-form :model="addFormData" :rules="rules" ref="addForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="学生学号">
            <el-input v-model="addFormData.number"></el-input>
        </el-form-item>
        <el-form-item label="学生姓名" >
            <el-input v-model="addFormData.name"></el-input>
        </el-form-item>
        <el-form-item label="学生生日" >
            <!--v-model : 双向绑定 -->
            <el-input v-model="addFormData.birthday" type="date"></el-input>
        </el-form-item>
        <el-form-item label="学生地址" >
            <el-input v-model="addFormData.address"></el-input>
        </el-form-item>
        <el-form-item align="right">
            <el-button type="primary" @click="addStu()">添加</el-button>
            <el-button @click="resetForm('addForm')">重置</el-button>
        </el-form-item>
    </el-form>
</el-dialog>

<script>
    new Vue({
        el:"#div",
        data:{
            dialogTableVisibleAdd: false,  //添加窗口显示状态
            dialogTableVisibleEdit: false, //编辑窗口显示状态
            addFormData:{},//添加表单的数据
            rules: {
                number: [
                    {required: true, message: '请输入学号', trigger: 'blur'},
                    {min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur'}
                ],
                name: [
                    {required: true, message: '请输入姓名', trigger: 'blur'},
                    {min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur'}
                ],
                birthday: [
                    {required: true, message: '请选择日期', trigger: 'change'}
                ],
                address: [
                    {required: true, message: '请输入地址', trigger: 'blur'},
                    {min: 2, max: 200, message: '长度在 2 到 200 个字符', trigger: 'blur'}
                ],
            }
            
        }
    });
</script>
```

##### 2.2.4 介绍几个函数

```html
<script>
    new Vue({
        el:"#div",
        data:{
            dialogTableVisibleAdd: false,  //添加窗口显示状态
            dialogTableVisibleEdit: false //编辑窗口显示状态
          
        },
        methods:{
            //改变每页展示条数时执行的函数, 并会把用户选择的条数值传递过来
            handleSizeChange(pageSize) {
                //修改分页查询的参数
                this.pagination.pageSize = pageSize;
                //重新执行查询
                this.selectByPage();
            },
            //改变页码时执行的函数，并会把用户选择的页码值传递过来
            handleCurrentChange(pageNum) {
                //修改分页查询的参数
                this.pagination.currentPage = pageNum;
                //重新执行查询
                this.selectByPage();
            },
            // 点击添加按钮时调用; 目的是将添加的弹框展示出来。
            showAddStu() {
                //弹出窗口
                this.dialogTableVisibleAdd = true;
            },
            //针对 添加 和 修改功能 对表单项数据重置提供的函数
            resetForm(refForm) {
                //双向绑定，输入的数据都赋值给了formData， 清空formData数据
                this.addFormData = {};
                //清除表单的校验数据
                this.$refs[refForm].resetFields();
            },
            showEditStu(row) {
                //row 为当前el-table 标签展示行 对应的 json {} 对象++
                //1. 弹出窗口
                this.dialogTableVisibleEdit = true;

                //2. 显示表单数据, 因为做了v-model 双向数据绑定, 所以给editFormData 赋值, 对应的表单项也就展示了相同的数据；
                this.editFormData = {
                    number:row.number,
                    name:row.name,
                    birthday:row.birthday,
                    address:row.address,
                }
            }
           
        //生命周期函数: 载入之后开始执行，执行时调用分页方法
        mounted(){
            //调用分页查询功能
            this.selectByPage();
        }
    });
</script>
```
