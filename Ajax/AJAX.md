# AJAX

- Asynchronous异步

- Javascript（在浏览器上发送ajax请求，这些代码是js语法的代码）

- And

- Xml（服务器响应处理完返回的数据可以是xml格式的，但也可能是json，也可能是普通字符串。**ajax也可以是这些传数据**）

严格说它不是一种技术，是多种技术结合的产物。

## 传统请求示例

url,超链接,form表单,js代码..

示例：

1. (Java程序--右键添加框架支持Web，其实和在创建之初用Maven模块创建是一样的，模板都是web模板）

2. 配置一下依赖`servelt-api.jar`和`jsp-api.jar`（在猫下，我的版本号是8.5.73）

前台html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>演示传统请求，以及传统请求的缺点</title>
</head>
<body>
<!--直接在浏览器栏上输入URL-->

<!--超链接-->
<a href="/old/request">传统请求（超链接）</a>
<!--form表单提交-->
<form action="/old/request" method="get">
    <input type="submit" value="传统请求(form表单提交)">
</form>
<!--通过JS代码来发送请求(单击事件)-->
<input type="button" value="通过JS代码发送请求" onclick="sendRequest">
<!--sendRequest函数没有，我们自己写一个-->
<script type="text/javascript">
    function sendRequest(){
        //发送请求
        //window.location.href = ""
        document.location.href = "/old/request"
    }
</script>


</body>
</html>
```

后台：（src根路径下建包）

快捷键**Ctrl+O**实现接口**doGet方法**

```java
//这里路径注意！这里不要加项目名了
@WebServlet("/request")
public class OldRequestServlet extends HttpServlet {

//【二周目下面有问题，你发现了吗？】
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        super.doGet(request, response);
        //响应信息到浏览器
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        //向浏览器输出HTML页面（代码），浏览器接收到HTML代码后渲染页面，展现页面给用户
        out.print("<h1>欢迎学习Ajax</h1>");

    }
}
```

3. 部署一下。

![ApplicationFrameHost_QAwt9N4NRY.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_QAwt9N4NRY.png)

~~（哎哎我去，这里出错，页面转乱码..）~~重新检查了一下代码，发现原来是后台方法多加了调用了父类的super.doGet方法..去掉后是成功运行的。新问题：js无响应..因为js没学so..

我们会发现上面三种操作会导致页面全部重新刷新一遍。有可能出现空白期：服务器处理中...——用户体验间断了。

通过HttpServeletRequest对象发送请求，就是通过ajax方式发送的请求，可以进行局部处理。ajax1请求和ajax2互相不耽误处理，无需互相等待。相当于浏览去内部的多个线程请求。让用户体验连贯而不会间断。

ajax可以在浏览器中发送异步请求！请求A和请求B是**异步的：谁也不需要等谁**，类似于多线程并发的。

服务器响应给前端可以有三种数据：

1. 普通文本

2. XML字符除按

3. JSON字符串

AJAX接收到了它们之后解析这些响应的数据，将解析后的数据渲染到div图层中，这个div就更新了，这样就完成了页面的局部刷新。

## 异步和同步

- 假设有t1和t2线程，t1和t2线程并发，就是异步。

- t2在执行的时候，必须等待t1线程执行到某个位置之后才能执行。t2在等t1，显然它们是排队的。排队就是同步。

- ajax是可以发送异步请求的。也就是说，在同一浏览器页面当中，可以发送多个ajax请求，这些请求不需要排队等待，是并发的。

## 补习一下JS知识

我们的需求：页面上有一个按钮，用户点击按钮之后，执行一段JS代码——弹出“hello javascript"的”alert警告"

常规操作：

```j
<script type="text/javascript">
    function say(){
        alert("hello javascript")
    }
</script>

<input type="button" value="hello" onclick="say()">
```

其中：

```js
<script type="text/javascript">
```

表明放在之间的是文本类型text。

javascript是告诉浏览器里面的test是个javascript脚本。h5后省略也不会出错，默认脚本语言就是js。

而现在我们要会做到的是：通过JS代码**给按钮绑定事件**

### JS显示方案

[JavaScript 输出](https://www.w3school.com.cn/js/js_output.asp)

### Window对象

浏览器对象模型BOM

所有浏览器都支持window对象。它<mark>**表示浏览器窗口**</mark>

所有 JavaScript 全局对象、函数以及变量**均自动成为 window 对象的成员**。

全局变量是 window 对象的属性。

全局函数是 window 对象的方法。

甚至**HTML DOM 的 document 也是 window 对象的属性之一**

### Document对象

在 HTML DOM (Document Object Model) 中 , 每一个元素都是 **节点**

当浏览器载入 HTML 文档, 它就会成为 **Document 对象**。

Document 对象是 HTML 文档的根节点。

Document 对象<mark>**使我们可以从脚本中对 HTML 页面中的所有元素进行访问**</mark>

**提示**Document 对象是 Window 对象的一部分，可通过 window.document 属性对其进行访问。

(重点代码)

```javascript
<!--通过JS代码给按钮绑定事件-->
<input type="button" value="hello" id="helloBtn">
<script type="text/javascript">
    //页面加载完毕之后，给id="helloBtn"的元素【绑定鼠标单击事件】
    //这个function就是一个回调函数，现在定义当load事件发生之后，它才执行
    //页面加载完毕之后load事件发生
    window.onload = function (){
        //获取id="helloBtn”的对象
        var helloBtn = document.getElementById("helloBtn");
        //给这个元素绑定click事件(这个function也是一个回调函数）
        //当helloBtn被click鼠标单击时，这个回调函数会被执行（有先后顺序）
        //现在只是将程序赋上去，只有事件发生时才会被回调
        helloBtn.onclick = function (){
            alert("javascript2");
        }

    }
</script>
```

执行结果成功（在html的boty里）。(点击五次，就会被调用五次)

alert(this)中的this其实就是发生事件的**事件源对象**（这里就是这个按钮对象）

## XMLHttpRequest对象

前端浏览器内置的一个对象。我们只有通过new这个对象，才能发送请求。

因为是内置的想用它，直接new一下它就有了。之后就可以调用它的方法。

先讲它的**两个重要方法**：

<u>open()</u>表示客户端和服务器端的通道打开

<u>send()</u>就是发送请求

**重要属性**：

<u>readyState</u>：**保存（记录）XMLHttpReques的状态。**

处理AJAX请求的Servlet对象，要把处理结果返回给浏览器。浏览器接收响应，解析数据，局部渲染/刷新div（假设页面有一个div)。

请求和响应全靠XMLHttpReques对象。AJAX的请求以及AJAX接收服务器的响应，完全依靠它。它其中的readyState属性记录下来XMLHttpReques对象的状态。

它对应的**状态值**有：（记录时间点）

- 0：请求未初始化

- 1：服务器连接已建立

- 2：请求已收到

- 3：正在处理请求

- 4：请求已完成且响应已就绪

也就是说，当XMLHttpReques对象的readState属性的值变成4时，表示AJAX请求以及响应已经全部完成了。

<u>onreadystatechange</u>:当readystate属性值发生改变时

## 发送get请求

新建java项目，右键加入web框架支持，引入依赖（猫里的jsp和servlet.jar）

先做一个简单基本大概，做一步测试一步：(记得部署猫)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ajax get请求</title>
</head>
<body>
<script type="text/javascript">
    window.onload = function (){
        document.getElementById("helloBtn").onclick = function () {
            //发送ajax get请求
            console.log("发送ajax get请求")
        }
    }
</script>
<!--给一个按钮，用户点击这个按钮的时候发送ajax请求-->
<input type="button" value="hello ajax" id="helloBtn">
<!--给一个div图层，这个div将来接收服务器响应回来的数据-->
<div id="mydiv"></div>
</body>
</html>
```

绑定事件，在鼠标点击按钮时，在控制台显示“发送ajax get请求“这行话，如果成功发出则证明先前代码没有报错。这里我又出错了，它不显示！查看了一下发现是回调函数function选择了普通成员属性而不是函数方法！修改后代码运行成功。

注意：是前端的浏览器发请求，所以以斜杠开始，要加项目名/ajax/ajaxrequest1

```javascript
<script type="text/javascript">
    window.onload = function (){
        document.getElementById("helloBtn").onclick = function () {
            //发送ajax get请求
            console.log("发送ajax get请求")
            //第一步：创建AJAX核心对象XMLHttpRequest
            var xhr = new XMLHttpRequest();
            //第二步：注册回调函数
            //当我们的状态值发生改变时，回调函数被调用
            xhr.onreadystatechange = function () {
                //这里的回调函数会被调用多次
                //0 -> 1被调用一次..（一共会被调用四次）【现在控制台输出一下】
                console.log(xhr.readyState)

            }
            //第三步：开启通道(只是浏览器和服务器建立连接，并不会发送请求)
            //XMLHttpRequest对象的open方法
            //open(method: string, url: string, async: boolean, username?: string | null, password?: string | null)
            //method:请求的方式，可以时get,post，也可以是其他请求方式
            //url:请求的路径
            //async:布尔值，true表示此ajax请求是一个异步请求（大多），否则false表示是同步请求（极少）。
            //user:用户名 pwd:密码，例如spt服务的口令，是进行身份认证的。要想访问服务器上的资源可能需要
            //主要看服务器的态度↑
            xhr.open("GET","/ajax/ajaxrequest1",true)
            //第四步：发送请求
            xhr.send()
        }
    }
</script>
```

现在我们测试执行绝对是404。因为通道对应的后端还没有写。我们只是看有没有状态值响应，它变了没有。

![ApplicationFrameHost_u4GRG8wBqt.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_u4GRG8wBqt.png)



状态值为4表示响应结束了，**响应结束之后一般会有一个HTTP的状态码**

常见HTTP状态码：

1. 200：成功了

2. 404：资源找不到

3. 500：服务器内部错误

响应HTTP状态码时HTTP协议的一部分，HTTP协议规定的。服务器之后都会有一个响应状态码。

1. 后台创建类，继承HttpServlet,

2. 重写（实现）接口doGet方法（因为是get请求）

3. 类上加注解@WebServelt填写连接路径/ajaxrequest1

```java
@WebServlet("/ajaxrequest1")
public class AjaxRequest1Servlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        //Servlet向浏览器响应一段数据
        PrintWriter out = response.getWriter();

        //out对象向浏览器输出信息
        //服务器的代码实际上和以前的代码是完全一样的
        //只不过这个out在响应的时候，xhr对象会接收到响应的信息
        //那该如何获取这个信息呢？
        out.print("welcome to study ajax!!");
    }
}
```

可以使用xhr对象的**responseText属性**，以字符串返回响应数据

```javascript
if(this.status == 200){
                    //通过xhr对象的该属性获取响应的信息
                    alert(this.responseText)
                    //把响应信息放到div图层中，渲染【这里有错误，你发现了吗？】
                    document.getElementById("myid").innerHTML = this.responseText
                }
```

图层div的id名写错了，应该是“mydiv"，修改后，页面成功渲染显示html



![ApplicationFrameHost_GMSPpnR3V6.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_GMSPpnR3V6.png)

### 熟悉一下流程

状态值变化五次，只有等于4时才处理，因为此时表示响应就绪了。

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>发送ajax get请求</title>
</head>
<body>
<script type="text/javascript">
    window.onload = function () {
        document.getElementById("btn").onclick = function () {
            //1.创建AJAX核心对象
            var xhr = new XMLHttpRequest();
            //2.注册回调函数
            xhr.onreadystatechange = function () {
                if (this.readyState == 4) {
                    if (this.status == 200) {
                        //不管服务器响应回来的是什么，都以普通文本形式获取
                        //获取到之后就可以把它设到span里了
                        document.getElementById("myspan").innerHTML = this.responseText
                    }else {
                        alert(this.status)
                    }
                }

            }
            //3.开启通道
            xhr.open("GET", "/ajax/ajaxrequest2", true)
            //4.发送请求
            xhr.send()
        }
    }
</script>
<button id="btn">发送ajax get请求</button>
<span id="myspan"></span>
</body>
</html>
```

（这里又有一个小技巧，代码+.if，自动包括If语句）

注意，**<u>后端这里路径不加项目名了</u>**

```java
@WebServlet("/ajaxrequest2")
public class AjaxRequest2Servlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        //设置响应的内容以及字符集
        response.setContentType("text/html;charset=UTF-8");
        //获取响应流
        PrintWriter out = response.getWriter();
        //响应
        out.print("<font color='red'>用户名已存在！</font>");
    }
}
```

记得去掉super一行，不复调用父类方法。

关于servlet的<u>respense.setContentType()</u>方法

[对于response.setContentType(MIME)的解释 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1691469)

作用是**使客户端浏览器，区分不同种类的数据**，并根据不同的MIME**调用浏览器内不同的程序嵌入模块**来处理相应的数据。
例如web浏览器就是通过MIME类型来判断文件是GIF图片。通过MIME类型来处理json字符串。

**一般在Servlet中，习惯性的会首先设置请求以及响应的<u>内容类型</u>以及<u>编码方式</u>**：
response.setContentType("text/html;charset=UTF-8");
request.setCharacterEncoding("UTF-8");

注意：状态值为4表示响应就绪，此时才会渲染加载响应返回的数据

innerHTML属性是js中的语法，**和xhr对象无关**，它可以设置元素内部的HTML代码。它可以讲后面的内容当作一段HTML代码解释并执行。它同师门还有一个innerText，也是设置元素中的内容，但将它堪称普通字符串解释；都和xhr对象无关。

温习一下：div独占一行，它是一个块；span同行

到目前为止我们还没有提交数据。

### get请求如何提交数据

- 在“请求行“上提交，格式是`url?name=value&name=value&...`

- 提交格式是HTTP协议规定的，遵循协议即可

现在我们来尝试提交一下（*写死*写法）：

```javascript
 //3.开启通道
 xhr.open("GET", "/ajax/ajaxrequest2?usercode=110&username=zhangsan", true)
```

```java
        //获取ajax get请求提交的数据
        String usercode = request.getParameter("usercode");
        String username = request.getParameter("username");

        out.print("usercode=" + usercode + ",username = " + username);
```

也可以**动态获取**：

js代码里写两个文本框

（不完整代码）

```javascript
           //3.开启通道
            //获取到用户填写的usercode和username
            var usercode = document.getElementById("usercode").value
            var username = document.getElementById("username").value
            xhr.open("GET", "/ajax/ajaxrequest2?usercode="+usercode+"&username="+username, true)
            //4.发送请求
            xhr.send()
        }
    }
</script>
usercode<input type="text" id ="usercode"><br>
username<input type="text" id ="username"><br>
<button id="btn">发送ajax get请求</button>
```

测试一下，成功。注意这里有一个细节：**中间加的是与&符号来连接的**。

## 对于低版本IE的ajax get缓存问题

## 发送post请求

**<mark>POST适合向服务器提交数据</mark>**

405：前后端发送处理请求不一致（如发的是get请求，却用doPost方法处理。

#### 模拟form表单数据提交

**send(string)** 里面可以写字符串的，写在里面的数据，**会自动在请求体当中提交数据。**（js代码获取用户填写的用户名和密码）

```javascript
<body>
<script type="text/javascript">
    window.onload = function () {
        document.getElementById("mybtn").onclick = function () {
            //发送AJAX POST请求
            //1.创建AJAX核心对象xhr
            var xhr = new XMLHttpRequest();
            //2.注册回调函数
            xhr.onreadystatechange = function () {
                if (this.readyState == 4) {
                    if (this.status == 200) {
                        document.getElementById("mydiv").innerHTML = this.responseText
                    }else {
                        alert(this.status)
                    }
                }
            }
            //3.开启通道
            xhr.open("POST", "/ajax/ajaxrequest3",true)

            //4.发送请求
            //js代码获取数据
            var username = document.getElementById("username").value;
            var password = document.getElementById("password").value;
            xhr.send("username="+username+"&password="+password)
        }
    }
</script>

用户名<input type="text" id="username"><br>
密码<input type="password" id="password"><br>
<button id="mybtn">发送AJAX POST请求</button>

<div id="mydiv"></div>
</body>
</html>
```

http请求包括：

- **get**（在**请求行上提交**）

- **post**（从**请求体当中提交**的）

注意：send(string)里面的格式不能随便来！还是得遵循http协议，和上面说的格式是一样的！

```java
@WebServlet("/ajaxrequest3")
public class AjaxRequest3Servlet extends HttpServlet {


    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        //out.print("用户名已存在");
        //获取提交的数据
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        out.print("用户名是："+username+"密码是："+password);
    }
}
```

后端执行那里打一下断点，猫执行测试一下。

**出现问题：响应值为null！** 后端没拿到

**少一行核心代码：<mark>需要设置请求头的内容类型</mark>**

怎么设置ajax请求头的内容类型?(固定写法)

//4.发送请求  
xhr.setRequestHeader("Content-Type","【这里面怎么写？！想不起来写个表格type，idea自动生成，我们拿过来！】")

```javascript
<form enctype="application/x-www-form-urlencoded"
```

```javascript
  //4.发送请求
xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded")
```

这样写请求头就 **<mark>申明我们模拟的是form表单提交</mark>** 。

不这样设置，数据就提交不到后端去。

重启服务器再测试一下。**此时控制台【网络】playload（载荷）发生改变**：此时请求信息变成了表单数据。

![ApplicationFrameHost_MZF92mIeeR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_MZF92mIeeR.png)



注意：请求头得放在send之前，open之后！

### 总结一下

- get适合向服务器请求数据，或者提交少量数据，不安全。

- post适合向服务器提交大量数据，并且安全性较高。

综上所述，get和post请求的区别在哪？其实就是前端代码的区别。都在第四步上。

1设置请求头的内容类型，模拟form表单提交数据。

2. 获取表单中的数据

3. send函数中的参数就是发送的数据，在”请求体“当中发生。

### 经典案例

**验证用户名是否可用**

（像微博，用户名可否注册？红标标）

要连数据库，后端底层需要一张数据库表

需求：

1. onblur失去焦点事件，只要失去焦点，就发送AJAX POST请求，提交用户名。

2. 后端服务器端接收到用户名，连接数据库，查找若发现没有，则后端响应消息：可以使用，否则span标签爆红：对不起，用户名已存在。

#### 数据库准备

我在mydb库下新建名为t_user表，表中含有两个数据id,name，注入张三和小李两人。

![ApplicationFrameHost_cZrlheUtER.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_cZrlheUtER.png)

#### 后端准备

在WEB-INF下建一个lib包，导入JDBC驱动jar包依赖放进去，我用到的是mysql8的8.0.27版本。黏贴后最好在上面框中**Build**，build ”依赖“一下。
