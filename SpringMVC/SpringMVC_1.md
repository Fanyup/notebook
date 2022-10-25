# 3、SpringMVC注解是开发

## 返回对象Object

### 返回字符串对象

此时方法返回的String代表的就不是视图了，**代表的是文本数据**。区分是有没有@ResponseBody注解。

注意：这时类型时文本数据text而不是json了。

#### 乱码问题

默认是8859-1编码，所以中文会产生乱码

"text/plain;charset=ISO-8859-1"作为contentType

解决方案，**给@RequestMapping增加一个属性produces**，指定新的contentType，改成produces="text/plain;charset=utf-8"就OK了。

此时用过滤器是不好使的，因为通过网络的方式输入输出，中间不走过滤器，其实web.xml我们之前是有配置过滤器的，但此时它是无效的。

# `<url-pattern>`用斜杠/方式

## 引言

jsp,js,png，.html都是tomcat猫处理的，和服务器没有关系。说明猫本身能处理静态资源的访问，像html，图片,js文件都是静态资源。猫的conf中的web.xml默认default有一个servlet，1.它能处理静态资源，**2.能够处理未映射到其他servlet的请求**。

发起的some.do是由中央调度器（springmvc框架）处理的。中央调度器本质就是一个servlet，它没有处理静态资源的能力。

**/用来表示静态资源和未映射的请求都给default来处理。**

## 问题：

使用它优先级更高，会把猫中的default给替代了。

**导致所有的静态资源都给中央调度器DispatcherServlet处理**，默认情况下它是没有处理静态资源的能力的。没有控制器能处理静态资源(html,css,图片,js)的访问，因此此时无法加载报错404。

而动态资源some.do是可以访问的，因为我们程序中有MyController控制器对象，能处理some.do请求。

那么该如何解决使用斜杠/映射的问题呢？

## 第一种静态资源的处理方式

在springmvc的配置文件中加标签`<mvc:default-servlet-handler>`

原理是：加入这个标签后，框架会创建一个控制器对象DefaultServletHttpRequestHandler(它就是类似我们自己创建的MyController处理器对象)，这个对象可以把接收到的**请求转发给猫default这个servlet**

### 解决冲突问题

default-servlet-handler标签和@RequestMapping注解有冲突，需要加入annotaiton-driven标签（注解驱动）解决这个问题。

此时some.do（和静态资源）也能正常执行处理了。

第一种方式的优点是方便，缺点是你的猫（**依赖于你的服务器**）中得有这个default的servlet才行，否则依旧报错。

## （掌握）第二种静态资源的处理方式

它是更进化的一步，springmvc配置文件中加个标签`<mvc:resources/>`,它和第一种类似，也会创建控制器对象，但此时它是ResourceServletHttpRequestHandler，它是专门用来处理静态资源的。

框架自己来处理静态资源的访问，**不依赖于服务器了**。

它有两个属性：mapping和location

- mapping
  
  **访问静态资源的uri地址，使用通配符****

- location

- 静态资源**在你的项目中的目录位置**

我们可以这样写：

静态资源：【图片】

`<mvc:resources mapping="/images/**" location="/images/"> />"`

**代表的是images后面的任何字符，如images/user/logo.gif,images/order/histroy/list.png。可以表示单一文件，也可以表示一级目录，多级目录。

还可以这样写：【网页】

`<mvc:resources mapping="/html/**" location="/html/（表示根目录下面的目录）"> />"`

【js文件】也是一样的

### 同样要解决冲突

加注解驱动标签。

## （掌握）更进一步

加入你要写css标签，那可就麻烦了！岂不是要写很多次？为了避免冗杂重复问题，我们可以**一条配置处理所有静态资源**。

在webapp下新建一个目录（就是包）叫static，把静态资源的location包，全部拷贝到这个目录之下

`<mvc:resources mapping="/static/**" location="/static/"> />"`

**也能代表子目录~

# 4、SSM整合开发

## 温习相对地址和绝对地址

相对地址有两种情况，加斜杠和不加斜杠，它们都能成为是访问地址，但情况不同。

### 不加斜杠的访问地址:

在jsp,html中使用的地址，都是在页面前端的地址，**都是相对地址**。

![chrome_371RDkbIZS.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_371RDkbIZS.png)

#### 问题：嵌套点击参考地址出错

![chrome_xuc8a7Brhe.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_xuc8a7Brhe.png)

#### 解决：base标签(基地址)

base标签是html中自带的，他一般是带协议的绝对地址。你可以把它理解成 **（地动山摇根基也不会发生改变的参考地址**）

`<base>`标签为页面上的所有链接规定默认地址或默认目标。

其中它有属性：

- href
  
  写项目的访问地址

![chrome_VGEyk9YZJb.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_VGEyk9YZJb.png)

**只是在原来参考地址基础上加了base标签，<mark>为了固定它</mark>。防止嵌套访问出错问题。**

但还是同样的问题，如果项目名变了，要修改还是很头疼！所以我们程序中有一段专门代码来获取路径的。这样就能动态获取项目名了。

![chrome_TStfpGKKcO.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_TStfpGKKcO.png)

**注意：它们只针对当前页面有效。**

### 加斜杠的访问地址

参考地址就变成了：【服务器ip到端口号】这部分+你访问地址

但这种方式不够灵活，不可取，因为前缀的项目名是可以灵活改变的，牵一发而动全身。

如何解决这个问题？可以用`${pageContext.request.contextPath}`表达式

![chrome_m6pbbQw6ww.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_m6pbbQw6ww.png)

这个表达式代表的是你项目的访问路径

**不推荐**，每个链接都要加，不如前面一劳永逸！

## SSM整合开发思路

重新new一个模板Model,maven选择maven-archetype-webapp骨架。

![chrome_3hEJcYnJRg.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_3hEJcYnJRg.png)

用户发起请求--SpringMVC接收--Spring中的Service对象--Mybatis处理数据（dao层）--数据库

SSM整合也叫做SSI

IBatis也就是Mybatis的前身。

**整合中有容器，有两个容器**：

1. SpringMVC容器，控制Controller控制器对象的

2. Spring容器，管理Service，Dao,工具类对象的

我们要做的是把使用的对象交给合适的容器创建、管理。

- 把Controller还有web开发的相关对象交给springmvc容器，把web用的对象写在springmvc配置文件中。

- service,dao对象定义在spring的配置文件中，让spring管理这些对象。

两个盒子（容器）是由关系的，**关系已经确定好了**，我们之前也讲过。

**springmvc容器时spring容器的子容器**，类似java中的继承（但不等于）。**子可以访问父的内容**。

也就是说**在子容器中的Controller（后端控制器）可以访问（使用）父容器中的Service对象。**

### 实现步骤

使用mydb库下的student表（其中定义主键id值自增）

![MySQLWorkbench_O7GGBxUBko.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_O7GGBxUBko.png)

1. 新建一个Maven的web项目

2. 加入依赖
   
   - springmvc,spring,mybatis三个框架的依赖加入到pom.xml中
   
   - jackson依赖（转换json的）
   
   - mysql驱动（也就是依赖）
   
   - druid连接池
   
   - jsp,servlet依赖
   
   项目一大的话依赖就很多。

3. **写web.xml**
   
   1. 注册前端（中央）控制器DispathcherServlet。，
      
      目的：
      
      - 创建springmvc容器对象，才能取创建Controller对象。
      
      - 它是Servlet，创建它才能接收用户的请求
   
   2- **注册spring的监听器:ContextLoaderListener**
   
      目的：
   
   - 创建spring的容器对象，它通过读取配置文件，才能创建service，dao等对象
     
     （这里是注解扫描器吗？没什么印象，稍后看一下）
   3. 注册字符集过滤器，解决post请求乱码的问题。
   
   4. 创建包，Controller包，service，dao，实体类包名创建好
   
   5. 写springmvc,spring,mubatis的配置文件。
      
      1. springmvc配置文件
      
      2. spring配置文件
      
      3. mybatis主配置文件
      
      4. 数据库的属性配置文件（xxx.properties)
   
   6. 写代码，dao接口和mapper（sql映射）文件，service和实现类，controller，实体类（存放属性的）。
   
   7. 写jsp页面。
