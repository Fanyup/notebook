# SpringMVC

IDEA技巧：shift+鼠标滚轮，实现横向滚动轴滚动吧

File->Settings->Build,Execution,Deployment->Build Tools(构建工具)->选择maven

如何创建maven工程？

maven中导入依赖，有网环境下自动导入jar包。

实现web功能模块

## 1、什么是MVC

是一种软件架构的思想，将软件按照**模型、视图、控制**器来划分（实现当前完整的web功能）

M：Model，模型层，指工程中的JavaBean（指的不是实体类）（指的是**当前工程里面所有处理数据的类**）

JavaBean分为两类：

- 一类称为实体类Bean
  
  **专门存储业务数据的**，入Student、User等
  
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

这里出现了一个小问题：还是跟网络环境有关，记得像pip安装一样，把节点关了再导，否则配置会爆红的，即网络下载失败。

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

servlet的初始化会执行init()方法。

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

springmvc创建容器时，读取的配置文件默认时/WEB-INF/`<servlet-name>-servlet.xml`

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
        //相当于request.settAttribute("msg,"欢迎使用..."）
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

![](C:\Users\up\AppData\Roaming\marktext\images\2022-10-18-12-55-17-image.png)

![](C:\Users\up\AppData\Roaming\marktext\images\2022-10-18-12-56-55-image.png)

项目越小，做框架越麻烦。项目越大，重复性工作减少，越方便。

请求是如何处理的？

### springmvc请求处理流程

1. 用户发起请求some.do--给到tomcat

2. 猫读取的是web.xml文件（webapp根目录下的，WEB-INF文件夹下的web.xml）
   
   根据`<url-pattern>`能知道*.do请求给中央调度器myweb的（mapping映射）
   
   myweb对应中央调度器class是给DispatcherServlet给它的。

3. 请求到达中央调度器，器它会根据【自定义配置文件位置】读取（resouces文件夹下的）springmvc配置文件。

4. 通过配置文件能知道所对应的组件扫描器，找到对应MyController创建对象。（后台控制器）(java蓝包中)

5. 找到后台控制器后，@RequestMapping中传value的接收值就是"/some.do"）
   
   用这个对应的doSome()方法
   
   简单说就是：
   
   **DispatcherServlet把some.do转发给MyController.doSome{}方法**

6. 框架执行doSome()，把得到的ModelAndView进行处理，转发到show.jsp

上面过程简化方式：

some.do--DispatcherServlet--MyController类
