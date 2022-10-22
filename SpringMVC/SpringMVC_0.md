# SpringMVC

IDEA技巧：shift+鼠标滚轮，实现横向滚动轴滚动吧

File->Settings->Build,Execution,Deployment->Build Tools(构建工具)->选择maven

如何创建maven工程？

maven中导入依赖，有网环境下自动导入jar包。

实现web功能模块

基于spring，**容器的概念**，SpringMVC会创建容器，WebApplicationContext.它作为容器是创建和管理控制器对象的，使用@Controller注解创建控制器对象（用Ioc的概念）。底层访问依然是Servlet-DispatcherServlet(中央调度器)。

中央调度器的作用是创建容器（WebApplicationContext容器和容器对象）（读取配置容器，容器对象放到全局作用域中就可以使用了（进而创建控制器对象）），2.（在web.xml中注册）负责接收请求，显示处理结果（这和Servlet是一样的）

## 1、什么是MVC

是一种软件架构的思想，将软件按照**模型、视图、控制**器来划分（实现当前完整的web功能）

M：Model，模型层，指工程中的JavaBean（指的不是实体类）（指的是**当前工程里面所有处理数据的类**）

JavaBean分为两类：

- 一类称为实体类Bean
  
  **专门存储业务数据的**，如Student、User等
  
  属性、属性值结构

- 一类称为业务处理Bean：
  
  处理数据的类称为javaBean
  
  指**Service对象处理业务逻辑**
  
  **Dao对象用来查询数据库，实现持久化操作**
  
  它们两个也是对当前数据进行处理的。
  
  **处理业务逻辑和数据方法**。

V：View，视图，展示数据；指的是工程中的html或jsp等页面，与用户进行交互，展示数据。

C：Controller，控制层，指**工程中的servlet**，**接收请求和响应**。比如说现在有一个请求从浏览器到服务器，该如何处理这个请求响应它呢？由servlet来完成。

工作流程：V-C-M（Service,Dao,Service）-C-V

## 2、什么是SpringMVC

是基于spring的一个框架，技术和Spring是一样的~

**实际上就是spring的一个模块，专门做web开发的**

你可以把它理解是servlet的一个升级。

web开发底层是servlet，框架是在servlet基础之上加入一些功能，让你做web开发方便。

SpringMVC就是一个Spring。因为Spring是容器，通过ioc能够管理对象，使用`<bean>`,@Component..创建对象。所以SpringMVC它也能够创建对象，放到容器中（SpringMVC容器）

**SpringMVC容器中放的是控制器对象**。

这个控制器怎么创建呢？**可以用注解@Controller**，来创建对象。

我们要做的是**使用@Controller创建控制器对象**，把对象放到springmvc容器中，把创建的对象作为控制器使用。<mark>它是用来处理请求的</mark>。

这个控制器对象就应该能<u>接收用户的请求，显示处理结果</u>，就当作是一个Servlet使用（但**只是一个普通类的对象**，只是放到容器之中，不是Servlet，**但springmvc赋予了控制器对象一些额外的功能**）

因为Servlet要继承Http类的！

因为web开发底层是servlet,springmvc中**有一个对象是Servlet**：**DispatherServlet(类**)它叫<mark>中央调度器</mark>

它是**负责接收用户的所有请求**，用户把请求给它，之后它把请求转发给我们的Controller对象，最后是Controller对象处理请求。

### 基本流程

index.jsp/main.jsp/...

--->DispatherServlet(它是一个Servlet)(中央调度器）

--->把请求转发、分派给Controller对象

（用@Controller注解创捷的对象）

（还是用ioc技术）

（可以有多个）

### 实操一下

#### 步骤（大纲）

1. 新建web maven工程  

2. **加入依赖**
    spring-webmvc依赖，间接把spring的依赖都加入到项目  
    jsp，servlet依赖  

3. **重点：在web.xml中注册springmvc框架的核心对象**<mark>DispatcherServlet</mark>  
    它叫**中央调度器**，是一个servlet，它的父类是继承HttpServlet  
    它也叫**前端控制器**（front controller)  
    负责接收用户提交的请求，调用其他的控制器对象，并把请求的处理结果显示给用户。  

4. 创建一个**发起请求的页面** index.jsp  

5. **创建控制器类**  
   
   1. 在类的方法上加入 **@Controller注解，用它来创建对象**  
      并放到springmvc容器中
      
      它叫**后端控制器**
   
   2. 在类中**方法上加入@RequestMapping**注解  

6. 创建一个作为结果的jsp，显示请求的处理结果  

7. 创建springmvc配置文件（和spring的配置文件一样）  
   
   1. 声明**注解扫描器**，指定注解所在的包名  
   2. 声明视图解析器

因为现在做的是web应用了，所有选择（新建）模块Model是Maven，这里选不选骨架都OK，选的话找到web模板：`maven-archetype-webapp`

~~这里出现了一个小问题：还是跟网络环境有关，记得像pip安装一样，把节点关了再导，否则配置会爆红的，即网络下载失败。~~问题解决了，是我的maven版本与idea不适配，换成3.6.3版本就能用了。

1. 指定java包【右键Make Directory as】是【Sources Root】——它得变蓝才正确。

2. 我们现在打开pom.xml文件：因为它是一个web项目，所以打包是以【war】（packaging)。
   
   因为是模板，所以build标签里面的东西可以删掉。

3. jdk是1.8的
   
   ```xml
     <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <maven.compiler.source>1.8</maven.compiler.source>
       <maven.compiler.target>1.8</maven.compiler.target>
     </properties>
   ```
   
   现在我们开始加入依赖——spring依赖和springmvc依赖，会自动导包，无需在手动加jar包了。

#### 1.pom.xml文件

##### 导入依赖

```xml
    <!--servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>

    <!--springmvc依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
```

##### 引入插件

build标签下引入插件：

```xml
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>

    </plugins>
```

#### 2.web.xml文件

##### 修改版本

模板默认版本很低，`"http://java.sun.com/dtd/web-app_2_3.dtd"`我们改一下：点击界面右上角【设置】选择【工程模块】“Project Structure”，选择【模板】Model，找到【Web】，点击【➖】先把默认的给删了。再添加：

重点：**<mark>添加时修改一下"web.xml"文件名！</mark>**

才会重新生成相应配置文件！（随便加个数字什么的）

**<mark>这时再把它改回去</mark>**。

##### “创建“对象

```xml
    <!--(声明)注册springmvc的核心对象DispatcherServlet-->
    <servlet>
        <!--名字随便起-->
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
```

声明了，但它这个对象现在是不会创建的，因为我们现在没有访问这个servlet，它就不会创建。但我们**现在的期望是：在猫启动时就把这个servlet对象创建出来。** 因为它有一个作用：**用来创建SpringMVC容器对象的！** 会读取springmvc的配置文件，把这个配置文件的对象都创建好，当用户发起请求时，就可以直接使用对象了~

servlet的初始化会执行()方法。

DispatcherServlet在init()中相当于：

```java
{
//创建容器，读取配置文件
    webApplicationContext ctx = new ClassPathXmlApplicationContext("springmvc.xml");
//把容器对象放入到ServletContext中
    getServletContext().setAttribute(key, ctx);
//接下来就可以用容器对象了
}
```

g跟spring的处理逻辑完全一样：**加载配置文件，获取对象**。

##### 猫启动时创建对象

再说一遍，我们现在希望DispatcherServlet能够在猫启动时就**创建对象。**

我们先发布一下这个项目：

上头【Add Configuration】-->【➕】-->Tomcat Server（Local）

先添加一个服务器。

选择【Deployment】-->【➕】发布我们刚刚的应用项目（**war exploded**)--下面context名字可以改短一点

我们的猫现在没有处理DiapatcherServlet，没有访问它，也没有创建对象。

我们希望的是在猫启动后就能创建Servlet对象，用到标签`<load-on-startup>1</load-on-startup>`，表示在启动时进行加载。

数字表示在猫启动后创建对象的顺序。整数。

**数值越小，猫创建该对象的时间越早。（>0)**

##### 自定义配置文件位置

为什么name报错呢？

springmvc创建容器时，读取的配置文件默认是/WEB-INF/`<servlet-name>-servlet.xml`

目录是固定的，所以我们一般自定义springmvc读取的配置文件的位置，用`<init-param>`

```xml
    <!--(声明)注册springmvc的核心对象DispatcherServlet-->
    <servlet>
        <!--名字随便起-->
        <servlet-name>myweb</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--自定义springmvc配置文件的位置-->
        <init-param>
            <!--springmvc的配置文件的位置-->
            <param-name>contextConfigLocation</param-name>
            <!--指定自定义文件的位置-->
            <param-value>classpath:springmvc.xml</param-value>
            <!--类路径，所以应该是recourse目录，我们给它创一个-->
        </init-param>


        <!--希望的是在猫启动后就能创建Servlet对象-->
        <load-on-startup>1</load-on-startup>
    </servlet>
```

##### 设置url-pattern（mapping映射）

在上面的**外部**设置

```xml
    <servlet-mapping>
        <servlet-name>myweb</servlet-name>
        <!--
        使用框架时，url-pattern可以使用两种值
        1.使用扩展名方式，语法 *.xxxx,xxxx时自定义的扩展名。
        常用的方式 *.do, *.action, *.mvc等，后面这些是自定义扩展名
        意思是：如果请求是以.do这些结尾的，都交给这个(名为webapp)servlet处理。
        http://localhost:9090/myweb/some.do
        http://localhost:9090/myweb/url.do
        -->
        <!--2.使用斜杠“/”，以后会讲到-->
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>et-mapping>
```

上面都是固定的，不用背，会用就行了！

用斜杠/方式有什么问题

#### 3.请求页面

删除默认的。在webapp根目录下新建一个index.jsp

```jsx
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <p>第一个springmvc项目</p>
    <p><a href="some.do">发起some.do的请求</a></p>
</body>
</html>
```

现在我们需要一个处理器（也就是控制器）类来处理请求

#### 4. 创建控制器类

在蓝色文件夹java目录下面创建类【系列包+类名：**这样可以直接创建包**】

```java
/*
* 创建处理器（控制器）类对象
* 对象放在springmvc容器中
*
* 和spring中讲的@Service,@Component是一样的
* value不写默认开头小写+类名
* */
@Controller(value = "myController")
public class MyController {
    /*
    * 处理用户提交的请求
    * springmvc中式使用方法来处理请求的
    * 方法是自定义的，可以有多种返回值、参数
    * 方法名称也是自定义的
    * */
    /*
    * 准备使用doSome方法处理some.do请求。
    * @RequestMapping：请求映射
    *   作用是把一个请求地址和一个方法绑定在一起
    *   即一个请求指定一个方法处理。
    *   属性：1. value
    *       是一个String,表示请求的uri地址
    *       （刚刚写的地址herf是some.do)
    *       value值必须是唯一的，不能重复
    *       推荐地址以：/斜杠为开头(表示它是一个根地址）
    *   使用位置：1.在方法上面（常用）
    *           2.在类的上面
    *   说明：使用该注解修饰的方法，叫处理器方法，或控制器方法。
    *   是可以处理请求的，类似与servlet中的doGet，doPost
    * */
    @RequestMapping(value = "/some.do")
    //doGet()
    public ModelAndView doSome(){
        //可以处理some.do请求了

    }
}
```

再来讲讲返回值ModelAndView类型

```java
    @RequestMapping(value = "/some.do")
    //doGet()--service请求处理
    /*
    * 返回值：ModelAndView
    * Model:数据，请求处理完成后，要显示给用户的数据
    * View：视图，比如Jsp等
    *
    * 本次请求的处理结果（数据+视图）（框架中的类）
    * */
    public ModelAndView doSome(){
        //可以处理some.do请求了
        //这里我们假设service处理完成了
        //现在要做的是将数据和视图往返回类型填充
    }
}
```

传值，响应操作（完整代码）

```java
    public ModelAndView doSome(){
        //可以处理some.do请求了
        //这里我们假设service处理完成了
        //现在要做的是将数据和视图往返回类型填充
        ModelAndView mv = new ModelAndView();
        //添加数据，框架在请求的最后把数据放到request作用域
        //相当于request.setAttribute("msg,"欢迎使用..."）
        //参数名，值
        mv.addObject("msg","欢迎使用springmvc做方法开发");
        mv.addObject("fun","执行的是doSome方法");

        //指定视图,指定视图的完整路径
        //框架对视图执行的是forward转发操作
        //即request.getRequestDispather("/show.jsp").forward(...)
        mv.setViewName("/show.jsp");
        //你只用加进去，框架会帮你做的，不像以前一样麻烦

        //返回mv
        return mv;
    }
```

能处理请求的都是控制器，我们创建的这个MyController控制器类能处理请求，叫做**后端控制器**（back controller)

#### 5.展示页面

在webapp根目录文件夹下创建show.jsp返回页面

```jsx
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>show.jsp</h3><br/>
    <h3>msg数据：${msg}</h3><br/>
    <h3>fun数据：${fun}</h3>
</body>
</html>
```

#### 6.声明注解组件扫描器

在resources文件夹下的springmvc.xml配置文件中声明组件扫描器（因为我们用的是注解方式创建的对象）

我发现一个**快捷方式**！不需要再像之前学spring时那样自己创建名称空间了 **！直接打component-scan..会自动载入的**！

```xml
    <!--声明组件扫描器-->
    <context:component-scan base-package="com.bjpowernode.controller"></context:component-scan>
```

#### 7.开启猫，测试一下

我去！成功了！

`http://localhost:8080/spring_course_test/)`

点猫，选择Edit Configuration查询Deployment可以看到/访问地址。

这里关掉了默认启动这一项。

![ApplicationFrameHost_zDbb9OG4Gk.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_zDbb9OG4Gk.png)

![ApplicationFrameHost_gTeJPniRup.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_gTeJPniRup.png)

项目越小，做框架越麻烦。项目越大，重复性工作减少，越方便。

请求是如何处理的？

### 实操详解

#### springmvc请求处理流程

1. 用户发起请求some.do--**给到tomcat**

2. 猫读取的是**web.xml文件**（webapp根目录下的，WEB-INF文件夹下的web.xml）
   
   根据`<url-pattern>`能知道*.do请求**给中央调度器myweb**的（mapping映射）
   
   myweb对应中央调度器class是给DispatcherServlet给它的。

3. 请求到达中央调度器，器它会根据【自定义配置文件位置】读取（resouces文件夹下的）**springmvc配置文件**。

4. 通过配置文件能知道所对应的**组件扫描器**，找到对应**MyController**创建对象。（后台控制器）(java蓝包中)

5. 找到后台控制器后，@RequestMapping中传value的接收值就是"/some.do"）
   
   用这个对应的doSome()方法
   
   简单说就是：
   
   **DispatcherServlet把some.do转发给MyController.doSome{}方法**

6. 框架执行doSome()，把得到的ModelAndView进行处理，转发到show.jsp

上面过程简化方式：

some.do--DispatcherServlet（中央调度器）--MyController类（它可以有很多个）

中央调度器：

1. 负责创建springmvc容器对象，读取xml配置文件创建文件中的Controller（后台处理器）对象。

2. 负责接收用户的请求，分派给自定义的Controller（后台处理器）对象。

我可以有多个请求some.do，other.do，它们都给中央调度器，再转发给MyController（它有一个方法处理/some.do的），再添一个后台处理器OtherController（它的方法处理/other.do）

还可以有很多后台处理器(xxxController)

注意：这些都是**在springmvc容器对象中进行处理的。** 和servlet比多了个中央调度器。

中央调度器能增加一些额外的功能，以后会说。

#### springmvc执行过程分析

源代码分析：

1. 猫启动，创建容器的过程
   
   web.xml文件有一个load-on-startup
   
   在猫启动时，会创建中央调度器的实例对象。
   
   该器的父类是继承HttpServlet类，是一个servlet，在被创建时会执行init()方法。
   
   在该方法中会执行：（这一步被封装隐藏了！）
   
   ```java
   {
   //创建容器，读取配置文件
       webApplicationContext ctx = new ClassPathXmlApplicationContext("springmvc.xml");
   //把容器对象放入到ServletContext（全局作用域）中
       getServletContext().setAttribute(key, ctx);
   //接下来就可以用容器对象了
   }
   ```
   
   - 读springmvc.xml配置文件，会执行组件扫描器，就能创建MyController后置处理器类对象了。**这个对象放到springmvc容器中**，容器是map，类似map.put("MyController",MyController对象)
   
   debug模式，查看源代码， **<mark>快捷键Ctrl+Fn+F12</mark>** 找方法，打断点。F8单步执行

2. 请求的处理过程
   
   1. 执行servlet的service()方法

![ApplicationFrameHost_jC0tOuTzfO.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_jC0tOuTzfO.png)

```java
protected void service(HttpServletRequest request, HttpServletResponse response)
```

Fn+F8执行断点下一步

doService方法--**doDispatcher方法：调用MyController的doSome方法。（它是中央调度器的类的方法）**

#### 配置视图解析器

##### 引入

如果我们单点show.jsp是没有数据的，因为它没有经过MyController处理的。这是一个bug，如何让用户不能直接方法show.jsp而是按照我们规定的顺序点击访问呢？

把show.jsp放到**WEB-INF文件夹里面，因为在它里面是不对用户开放的。** 资源受保护，不能直接访问。

创建一个子目录view，把视图放里面。

那么后置处理器中相应路径也得变。

```java
mv.setViewName("/WEB-INF/view/show.jsp");
```

以指定扩展名时有很多重复编写操作（如路径名，.jsp后缀等），可以框架帮我们处理——视图解析器。

##### 视图解析器

在**springmvc.xml文件中**引入视图解析器。框架中的类，类名InternalResourceViewResolver(IRR快速打出)

里面的属性也是封装好的！**prefix前缀**。路径值

第一个斜杠/:web应用的根。后一个：代表它是一个路径。

**suffix后缀**

```xml
    <!--声明调用springmvc框架中的视图解析器，帮助开发人员设置视图文件的路径-->
    <!--它是框架中的类，不需要声明，不需要id，-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--需要两个属性-->
        <!--前缀：视图文件的路径-->
        <property name="prefix" value="/WEB-INF/view/"/>
        <!--后缀：视图文件的扩展名-->
        <property name="suffix" value=".jsp"/>
    </bean>
```

j简化了步骤（MyController控制器类中）

```java
//mv.setViewName("/WEB-INF/view/show.jsp");

//当配置了视图解析器后，可以使用文件名（逻辑名称）指定视图
mv.setViewName("show");
//框架会根据视图解析器的前缀+逻辑名称+后缀组成完成路径，这就是字符连接操作。
```

补充一点:web.xml配置自定义springmvc.xml文件路径时它的名字也是自定义的。（**就是自己建的一个根resources包下的xml文件**）

## 3、SpringMVC注解式开发

<mark>**主讲注解@RequestMapping的用法**</mark>

### @RequestMapping

#### 1、放在类上面（value属性）

它可以放在方法上，也可以放在类上面。

复制并创建一份新的项目，记得要改一下pom.xml里的项目名。导入这个新的模块Model（右上角设置，也可以File-->Project Structure找到）

<mark>value:所有请求部分**公用前缀**，</mark>我们叫它**模块名称**。（比如”/test“)（以什么为公共开头的）（不加也可以，不强制要求）

#### 2、方法上：指定请求方法(method属性)

现在我们都是通过get方式，通过页面点击超链接发送请求，springmvc可以控制你是用get方式，还是post方式

**属性method，表示请求的方式**

**<mark>它的值式RequestMethod类的枚举值</mark>**

例如表示get方式：

    RequestMethod.GET

表示post方式：

    RequestMethod.POST

这两个最常用

它的使用：

现在指定some.do使用get请求方式

```java
 @RequestMapping(value = "/some.do",method = RequestMethod.GET)
```

指定other.do式post请求方式

```java
@RequestMapping(value = "/other.do",method = RequestMethod.POST)
```

post用表单。index.jsp修改：

```jsx
<body>
    <p>第一个springmvc项目</p>
    <p><a href="some.do">发起some.do的get请求</a></p><br/>

//【下面的代码你发现潜在报错问题了吗？】
    <form action="/other.do" method="post">
        <input type="submit" value="post请求other.do">
    </form>
</body>
```

**<mark>不指定请求方式，没有限制，即不强制要求，通用</mark>**

这里出现了一个问题。

#### 运行时爆红问题

other.do点进去后找不到相应的other.jsp。检查后发现问题，你看我上面写的框中some.do用的地址直接而没有斜杠，所以能找到。这里我们先前已经对公共前缀进行了封装。那么下面那个我一葫芦画瓢依照视频的加了斜杠（不同情况，原视频是有前文件user/的），这里的/other.do就将根路径地址直接编程localhost:8080/other.do了！当然无法找到，去掉后重启猫，可以正常运行。

那么，该如何**接收用户提交的参数**呢？

### 处理器方法的参数

**<mark> 接收形参类型</mark>**

<mark>**一、常用到的三类形参类型：**</mark>

- HttpServletRequest代表请求

- HttpServletResponse代表应答

- HttpSession代表会话

它们放在后台处理器类的方法的形参位置，用来接收请求中的参数。

用法就是把它们放到形参框中就可以了，springmvc会给它们自动赋值。+形参名

示例：用HttpServeltRequest类型获取请求参数

request对象的getParameter方法

parameter"参数"

```java
 public ModelAndView doSome(HttpServletRequest request){
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","欢迎使用springmvc做方法开发"+request.getParameter("name"));
        mv.addObject("fun","执行的是doSome方法");
 //不完整代码
```

c重启猫，加参数，<mark>**在地址后加参数**</mark>`?name=zhangsan`

能收到值，**说明形参request对象已经实例化**，可以直接调用它的方法了。

**<mark>二、请求中所携带的请求参数</mark>**

即用户真正所**提交的数据**（姓名啊一些）

这一大类展开讨论：（先将最常用的两种方式）

#### 1、逐个接收

要求：<mark>**处理器（控制器）方法的形参名和请求中参数名必-须一致。**</mark>

```java
@RequestMapping(value = "/receiveproperty.do")
    public ModelAndView doSome(String name, int age){

        ModelAndView mv = new ModelAndView();

        mv.addObject("myname",name);
        mv.addObject("myage",age);
        mv.setViewName("show");
        return mv;
    }
```

**<mark>同名的请求参数，赋值给同名的形参</mark>**

页面index.jsp修改：（提交为表格）

```jsx
   <form action="receiveproperty.do" method="post">
        姓名:<input type="text" name="name"><br/>
        年龄:<input type="text" name="age"><br>
        <input type="submit" value="提交参数">
    </form>
```

这里我们示例的请求参数名为name和age

**记得复制新项目后，要再发布一下项目:猫Edit..Deployment(部署)**

一定一定一定要看一下你在猫上新部署的应用的地址！我趣尼玛啥了，之前怪不得为什么一直读取不到，原来是地址还是用了之前的地址！服了。

![ApplicationFrameHost_wKkxDgxRHr.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_wKkxDgxRHr.png)

框架接收请求参数：

1. 使用request对象接收请求参数
   
   String strName = request》getParameter("name");
   
   String strAge = request.getParameter("age");

2. springmvc框架通过中央调度器调用MyController的doSome()方法。调用方法时，按名称对应，把接收的参数赋值给形参。
   
   doSome(strName, Integer.valueOf(strAge))
   
   框架会提供类型转换的功能，把String转为int, long, float, double等类型。它帮我们做了，我们就可以在方法中直接使用name, age了。

**400状态码时客户端错误，表示提交请求参数过程中发生了问题。**（提交数据出错了）

其中一个报错的情况：这里我设置的传入参数类型是int，这时我在输入年龄栏什么也不填写，这是应该传入的是null空值（int自动封装成Integer，然后再转成Int,因为我们最终要的int），但这时无法转换成数字，故报错400。

spring会做一个记录，**警告**，放在自己的**日志**里。

要解决不能传入null的问题，只需要将参数类型改成包装类Integer就可以了。（即最终我们要的是Integer)

嫌太麻烦怎么办？把所有值都设置成String类型！，但其实我们在操作数据时还得进行Integer,ValueOf()的操作...

就是说你参数不合法，doSome根本就不会执行了~也不用再进行年龄类型转换什么的了。

最后重申一下：**形参名字对应，同位置无关。**

现在又出现一个新问题：post方式输入中文会返回乱码，get方式则没有这个问题。如何解决？

以前servlet方法时设置request对象方法：

```java
    public void doGet(HttpServletRequest request){
        request.setCharacterEncoding("utf-8");
    }
```

一个不方法：它要在每个方法最前面都加这个方法。非常不方便。

我们在项目开发中，可以将它**放到过滤器中**，**<mark>在过滤器中设置字符编码</mark>**。处理乱码问题。

过滤器可以自定义，也可以使用框架中提供的过滤器：`CharacterEncodingFilter`

##### 过滤器解决乱码问题

过滤器位于spring-web这个依赖：

![ApplicationFrameHost_A2hbbTqfhn.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_A2hbbTqfhn.png)

项目中有，所以我们可以直接用。

打开web.xml文件，

这个过滤器里有三个属性需要赋值，encoding编码，两个布尔值设为真。

`init-param`标签是用来初始化属性的。（我猜）

```xml
    <!--注册声明过滤器，解决post请求乱码的问题-->
    <filter>
        <!--名字随便起,习惯用类名，首字母小写-->
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--设置项目中使用的字符编码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <!--强制请求对象（HttpServletRequest)使用encoding编码的值-->
        <init-param>
            <param-name>forceRequestEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
        <!--强制应答对象（HttpServletResponse)使用encoding编码的值-->
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>

    </filter>
```

定义好之后还需在外面加一个mapping映射指向。

```xml
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <!--/*：表示强制所有的请求先通过过滤器处理-->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

现在再测试，就可解决编码问题了~

##### @RequestPara注解

现在有一个新问题，我们不能保证形参名和参数名总是一样的。

现在我们示例假设，它们就是不一样的！那么此时我们可以借助一个注解来解决这个问题。

它有属性：

1. value：请求中参数的名称
   
   位置：在处理器方法的形参定义的前面

```java
 @RequestMapping(value = "/receiveproperty.do")
    public ModelAndView doSome(@RequestParam(value = "rname") String name,
                               @RequestParam("rage") int age){{
```

value可以省，只写里面的。表示请求中rname的参数赋给形参

2. required 是一个布尔类型boolean，默认是true
   
   true:表示请求中必须包含此参数。为假的时候，请求参数可以有也可以没有。

**注意：这个注解在逐个接收参数才有效。**

#### 2、对象接收参数（常用）

还有一种方式

用一个java对象一次接收多个参数。

定义一个普通类，没有特殊要求，有set,get方法，有构造参数就行了。

现在我们建一个包叫vo，其类是用于保存参数值的。

```java
//保存请求参数值的一个普通类
public class Student {
    //属性名和请求参数名摇一摇
    private String name;
    private Integer age;

    public Student(){
        System.out.println("==Student的无参数构造方法==");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("setName"+name);
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        System.out.println("setAge:"+age);
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

Controller控制器类中添加一个新的方法：

```java
    @RequestMapping(value = "/receiveobject.do")
    public ModelAndView receiveParam(Student myStudent){
        System.out.println("receiveRara,name = "+myStudent.getName()+"age:"+myStudent.getAge());

        ModelAndView mv = new ModelAndView();

        mv.addObject("myname",myStudent.getName());
        mv.addObject("myage",myStudent.getAge());
        /*对象也可以放到模型之中！*/
        mv.addObject("mystudent",myStudent);
        mv.setViewName("show");
        return mv;
    }
```

测试一下，结果返回成功。

![ApplicationFrameHost_Ift1DPcO35.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_Ift1DPcO35.png)

![ApplicationFrameHost_2Fzph3ayX6.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_2Fzph3ayX6.png)

为了观看结果更加清晰，我们在创建普通对象类时为它的set方法调用时增加了一个输出语句，(并重写了toString方法，调用了它的无参构造）那么当它调用set方法时就能更好看见了效了。一切都是为了更好的体现效果~

**<mark>用对象接收参数是我们在实际应用中最多的。</mark>** 

我们可以在方法内部直接去用，还可以把它传给service和dao。而且它是形参，所以可以有多个，如School school等等。因为是按照同名原则查找的，互相并不干扰，也不会有顺序问题。

温习：这个示例中的控制器方法中（数据+视图show）就是我们处理的返回结果（模拟）

### 处理器方法的返回值类型

处理器方法的返回值表示请求的处理结果。

有四大类返回类型：

#### 返回ModelAndView

1. 有Model数据最终是放到了request作用域里；

2. 有View视图，框架**对视图执行forward转发操作**。

数据+视图

```java
    public ModelAndView doSome(){
        ModelAndView mv = new ModelAndView();
        //添加数据，框架在请求的最后把数据放到request作用域
        //相当于request.setAttribute("msg,"欢迎使用..."）
        //参数名，值
        mv.addObject("msg","欢迎使用springmvc做方法开发");
        mv.addObject("fun","执行的是doSome方法");
        //指定视图,指定视图的完整路径
        //框架对视图执行的是forward转发操作
        //即request.getRequestDispather("/show.jsp").forward(...)
        mv.setViewName("/show.jsp");
        //你只用加进去，框架会帮你做的，不像以前一样麻烦

        //返回mv
        return mv;
    }
```

若你只需其中的某一个，它就有些大材小用了。我们会讲后面这些。

#### 返回String

**<mark>代表视图View</mark>** 部分。处理器方法返回的字符串可以**指定视图名称（或视图路径）**。

示例：老调重弹：

复制，新建项目，删除target文件夹，修改pom.xml中的项目名称为新建项目名称。IDE导入新项目。（注意：“导入”并不等于猫的“部署”，所以这是两个步骤）

1. **指定视图名称方式**
   
   ```java
       /*
       * 处理器方法返回String
       * --表示逻辑视图名称，需要配置视图解析器（它存在，在springmvc.xml文件里，之前我们配过）
       * */
       @RequestMapping(value = "/returnString-view.do")
       public String doReturnView(String name, int age){
           System.out.println("doReturnView,name="+name+"age="+age);
           //show:逻辑视图名称，项目中配置了视图解析器
           //框架对视图执行的是forward转发操作
           return "show";
       }
   ```

如果我想既能返回String，又能够存储数据，我们可以这么做：

```java
 public String doReturnView(HttpServletRequest request, String name, int age){
        System.out.println("doReturnView,name="+name+"age="+age);
        //可以自己手工添加数据到request作用域
        request.setAttribute("myname",name);
        request.setAttribute("myage",age);

        //show:逻辑视图名称，项目中配置了视图解析器
        //框架对视图执行的是forward转发操作
        return "show";
    }
```

2. **表示完整视图路径**
   
   此时不能配置视图解析器，否则它会报错，**会重复操作**。
   
   ```java
       //处理器方法返回String，表示完整视图路径，此时不能配置视图解析器
       @RequestMapping(value = "/returnString-view.do")
       public String doReturnView2(HttpServletRequest request, String name, int age){
           System.out.println("==doReturnView2,name="+name+"age="+age);
           //可以自己手工添加数据到request作用域
           request.setAttribute("myname",name);
           request.setAttribute("myage",age);
   
           //show是一个逻辑名称，我们现在是要【完整视图路径】
           //那项目中不能配置视图解析器
           //框架对视图执行forward转发操作
           return "/WEB-INF/view/show.jsp";
       }
   ```

#### 返回void(了解)响应ajax

了解即可，我们很少用。

它既不能表示数据，也不能表示视图。

在处理ajax的时候（它把请求是发给servelt的），可以使用void返回值。

完全可以通过**应答对**象返回输出数据的。public void doGet(HttpServeltRequest request,HttpServeltResponse **reponse**)相应ajax请求。

**ajax请求服务器端返回的就是数据，与视图无关。**

ajax常用jQuery库文件(js)创建(jquery-3.4.1.js)，处理数据用到是json格式，需要用到包。那么我们就在pom.xml中加入依赖：(记得刷新一下Maven让他在有网环境下下载引入包)

```xml
    <!--Jackson依赖-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
    </dependency>
    <!--用来处理json的-->
```

1. jsp文件引入js库`<script type="text/javascript" src="js/jquery-3.4.1.js</script>"`

2. 接着在该jsp文件中做ajax了，简单点做一个按钮
   
   `<button id="btn">发起ajax请求</button>`

3. jsp文件中绑定实践：按钮单继。$("button")标签选择器
   
   ```jsx
   <script type="text/javascript>
       $(function(){
           $("button").click(function(){
               alert("button click：按钮被单击了");
   })
   }) 
   ```

#### （P24，25跳过）

#### 返回对象Object

代表广，任何的返回值皆可以是对象。

**返回对象是数据**，代表json数据，**不是作为逻辑视图出现的，**

**而是直接作为页面显示的数据出现**。主要是响应ajax请求的。

对象有属性，属性就是数据，所以返回Object表示数据，和视图无关。

可以使用对象表示的数据，响应ajax请求。

现在做ajax，主要使用json的数据格式。

**实现步骤**：

1. 加入处理json的工具库的依赖。springmvc默认使用的jackson。

2. **把java对象转成json**，在springmvc配置文件中**加标签**`<mvc:annotation-driven>`它叫 **<mark>注解驱动</mark>**。

3. 在处理器方法上**加@ResponseBody注解**，**做输出的**。

`<mvc:annotation-driven>`注解驱动的功能是完成java对象的json,xml,text二进制等数据格式的转换。底层是HttpMessageConverter接口（消息转换器）。接口定义的java对象转为json，xml等数据格式的方法。它有很多的实现类，这些实现类完成了数据格式的转换。

里面有两个抽象方法：canWrite（查看能转换成什么格式）和write（假设是json格式，利用jackson库实现转换）

常用的两个实现类：

- StringHttpMessageConverter
  
  处理字符串时用它

- MappingJackson2HttpMessageConverter
  
  处理java对象转为json格式的

这个标签在加入到springmvc配置文件后会自动创建这个接口的七个实现类对象。

##### @ResponseBody注解

放在处理器方法的上面。

把处理器方法返回对象转为json后，通过HttpServletResponse输出数据，响应ajax请求。

```java
PrintWriter out  = response.getWriter();
out.println(json);
out.flush();//刷新缓存
out.close();
```

##### 实现一下

需求：处理器方法返回一个Student,通过框架转为json,响应ajax请求

pom.xml中添加jaskson依赖

```xml
    <!--Jackson依赖-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databink</artifactId>
      <version>2.9.0</version>
    </dependency>
```

springmvc.xml配置文件中添加注解驱动

（记得打开视图解析器，前面任务示例中取消调了它）

![idea64_7etpKijSGp.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_7etpKijSGp.png)

注意我们在选择标签时出现了多个“重名”的！一定不能一股脑回车！/cache缓存不是我们要的，/schema **/mvc** 它是我们要用到的！

![idea64_vFo8T2ypTW.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_vFo8T2ypTW.png)

此时它会自动帮我们加好mvc这个名称空间（图里最下一行，不完整截图，下面还有）

加之后，实现类中才有MappingJackson2HttpMessageConverter这个实现类。

前端ajax请求：

```javascript
<script type="text/javascript">
    $(function () {
        $("button").click(function () {

            $.ajax({
                url:"returnStudentJson.do"
                data:{
                    name:"zhangsan",
                    age:20
                },
                type:"post"
                dataType:"json",
                success:function (resp) {
                    //resp从服务器端返回的是json格式的字符串
                    //jquery会把字符除按转为json对象，赋给resp形参。
                    alert(resp.name + "  " +resp.age);

                }
            })
        })
    })
</script>
    })
</script>
```

```java
 //注解没有先后关系
    @ResponseBody
    @RequestMapping(value = "/returnStudentJson.do")
    public Student doStudentJsonObject(String name, int age){
        //调用service，获取请求结果数据，Student对象表示结果数据。
        Student student = new Student();
        student.setName("里斯");
        student.setAge(20);
        return student;//会被框架转为json

    }
```

返回对象框架的处理流程：

1. 框架会把返回的Student类型，调用框架终端ArryList`<HttpMessageConverter>`中每个类的canWriter()方法，来检查哪个HttpMessageConverter接口的实现类能处理Student类型

2. 框架会调用实现类的write()
   
   本例中是MappingJackson2HttpMessageConverter类的write()方法。
   
   把看里斯同学的的student对象转为json，调用Jackson的ObjectMapper实现转为json（这些都已封装，不用你管了）

3. 框架会调用@ResponseBody把2的结果数据输出到浏览器，ajax请求处理完成。
   
   ContentType:application/json;charset=utf-8

专注业务功能实现。

这个示例中讲的是单一对象student，它最终转成json对象。但在我们项目开发中，大多情况下，可能需要返回多个对象，此时用到List集合。

###### JsonArray

`List<Student> list = new ArrayList<>;`

最终会被转为json数组。能保证顺序。index.jsp

里用jquery的循环函数

```jq
$.each(resp,function(i,n){
    alert(n.name+" "+n.age)
})
```

List集合用的比较多。
