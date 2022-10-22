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

# <url-pattern>用/方式

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

### 加斜杠的访问地址:

参考地址就变成了：【服务器ip到端口号】这部分+你访问地址

但这种方式不够灵活，不可取，因为前缀的项目名是可以灵活改变的，牵一发而动全身。

如何解决这个问题？可以用`${pageContext.request.contextPath}`表达式

![chrome_m6pbbQw6ww.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_m6pbbQw6ww.png)

这个表达式代表的是你项目的访问路径

**不推荐**，每个链接都要加，不如前面一劳永逸！
