# SpringMVC核心技术

## 请求重定向和转发

forward转发：用户发起请求给服务器端的资源1，1转发给资源2继续处理请求；在用户端只发起了一次请求。地址栏是一个地址（资源1的）；在服务器端内部实现，所以能访问WEB-INF下受保护的资源。

redirect重定向：资源1把地址告诉浏览器，浏览器主动地向资源2带发起请求；请求次数是两次；用户的地址栏发生变化：资源2的。因为两次都是用户发起的，因此不能访问WEB-INF下的资源。

其实这两个是servlet完成的，但springmvc框架为我们整合了更简便的方式，不用像以前那样麻烦了。

### forward转发

语法： `setViewName("forward:/视图文件完整路径")`

特点是：显示转发，**不和视图解析器一起使用**（就是定义前缀后缀那个家伙），就当项目中没有视图解析器，不写逻辑名称。

它可以转发到视图解析器以外的界面（即不在定义下，其他地方也可以转发到）

**示例是用ModelAndView转发用户反馈页面的视图。**

### redirect重定向

语法： `setViewName("redirect:/视图文件完整路径")`

它也是不和视图解析器一块使用的。

![chrome_glAinaBviP.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_glAinaBviP.png)

为什么我们先前输入页面传递的信息取不到值呢？

![chrome_V9cob0jZ1G.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_V9cob0jZ1G.png)

因为重定向发起了两次请求。request请求1中传入了数据到request1的作用域，而重定向request请求2中是没有的。但是我们可以获取get方式给我们的1请求参数。 

![chrome_8QtuDqZSyz.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_8QtuDqZSyz.png)

在请求2页面中，可以通过：`${param}`代表的是参数集合对象，通过它可以取出请求参数中某个对应的值了。

再次注意：重定向不能访问/WEB-INF下的资源（因为它们受保护）

### 不太清晰的点

视图路径指定似乎不需要加项目名。

## 统一全局异常处理

以前conrtoller的每个代码都要加try,catch，非常麻烦，框架帮我们做了一个集中的位置，各方法异常都统一到这儿进行处理。

aop思想。实际上就是业务分离，异常处理实际上和我们的基本功能没有关系，现在做的其实就是方法分离。基本功能方法现在只保留业务逻辑代码，这样就解耦合了。

会用到两个注解：

### @ControllerAdvice加类上

控制器增强，也就是说给我们的控制器Controller类增加功能——异常处理功能。它的方法和Controller方法的定义是一样的，也就是说可以有多个参数，也可以有ModelAndView、String、void、对象类型类型的返回值。

创建一个普通类，作为全局异常处理类。如何让它“摇身一变”？

在类上给它加上这个注解！

注意：我们在配置springmvc配置文件时，现在不仅要添加扫描@Controller注解的组件扫描器，**还要添加扫描@ControllerAdvice所在的包的组件扫描器**。**声明注解驱动。**

### @ExceptionHandler加方法上

它有一个属性：

`@ExceptionHandler(异常的class)`:表示当发生此类型异常时，由它来处理。

在该类的方法上加上这个注解！

![chrome_OjFu86ts2c.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_OjFu86ts2c.png)

异常发生处理逻辑：

1. 把异常记录下来，记录到数据库，日志文件；
   
   发生时间，哪个方法发生的，错误内容。

2. 发送通知，把异常的信息通过邮件、短信、微信发送给相关人员。

3. 给用户友好的提示。

1、2我们先不做，先做第3个。

还需要一个处理视图异常的jsp页面：得告诉用户发生了什么错误吧。

如何捕获处理其他异常呢？——注解后不加value值（就变成了默认异常处理方法）

## 拦截器

实现指定接口`HandlerInterceptor`的类，都叫拦截器；和过滤器挺像的，但功能侧重点不同。

**过滤器是用来过滤请求参数的、设置编码字符集等工作**；

**拦截器是用来拦截用户请求，可以拦截请求，对请求做预先判断处理工作。** 对请求做判断和处理的。

是全局的，可以对多个Controller做拦截。

一个项目中可以由0或多个拦截器，常用在**用户登陆处理，权限检查，记录日志**。

使用步骤：

1. 定义类，**<mark>实现HandlerInterceptor接口</mark>**

2. 在springmvc配置文件中 **<mark>声明拦截器</mark>** ，让框架知道拦截器的存在。

蓝机器执行时间：

1. 在请求处理之前，也就是controller类的方法执行之前执行。

2. 在控制器方法执行之后也会执行拦截器。

3. 在请求处理完成之后也会执行拦截器。

示例先写一个拦截器。

接口中有仨个抽象方法，它帮我们将方法**访问类型改成了default，所以我们不必三个都重写，需要用到哪个重写哪个就好了。**

Ctrl+O快捷键实现抽象方法。

**preHandler预处理方法**。第三个Object handler要处理的Controller类，返回布尔值。**真表示通过**处理器验证。假表示不通过。

它是在Controller控制器方法之前先执行的。

用户的请求首先到达此方法。该方法可以获取请求的信息，验证是否符合要求。可以验证用户是否登录，验证用户是否有权限取访问某个链接地址。验证失败，可以截断该请求，成功则放行。

其他两个方法仅作了解：

postHandle：在处理器方法之后执行，可以对原来的执行结果做二次修正。

afterCompletion：最后执行的方法。它规定是在请求处理完成之后执行的。就是在你的视图处理完成之后（进行了forward转发）就认为请求处理完成了，之后再执行它。一般是做资源回收工作的，**请求过程种创建了一些对象，可以把占用的内存回收**。

### 二：声明拦截器

 `<mvc:interceptors>`

在springmvc配置文件声明，有好几个，注意得是mvc的。

使用的依然是aop面向切面的思想。

可以用`/**`声明所有，前面可以加目录名

多拦截器保存实际上是ArrayList，所以声明多拦截器时要注意先后顺序，它会按照list先后顺序执行。

### 多拦截器情况：

两个拦截器都设置为真的情况：

![chrome_A3aJAulPxR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_A3aJAulPxR.png)

为什么这样执行呢？👇

### 拦截器执行链

也叫处理器执行链。

![chrome_5FFd5n6nzK.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_5FFd5n6nzK.png)

现在1为真，2为假的情况：（因为2是在1之后拦截的）

此时2被拦截，处理器方法无法执行，也就是示例种的doSome方法没有执行，但最后1的afterCompletion()会被执行。

```html
111111一拦截器的MyInterceptor的preHand1e0
22222一拦截器的MyInterceptor的preHand1e0
111111一拦截器的MyInterceptor的afterComp1etion0
```

### 和过滤器的区别

1. 过滤器是servlet规范中的对象，拦截器是框架中的对象（范围更小）

2. 过滤器是实现Filter接口的对象，拦截器是实现HandlerInterceptor接口的对象。

3. 过滤器是用来设置request，reponse的参数的，属性的，侧重对数据过滤的。
   
   拦截器是用来拦截验证请求的，能截断请求。

4. 过滤器是在拦截器之前先执行的。

5. 过滤器是猫服务器创建的对象
   
   拦截器是springmvc容器中创建的对象

6. 过滤器是一个执行时间点
   
   拦截器有三个执行时间点

7. 过滤器可以处理jsp,js,html等
   
   拦截器是侧重拦截对Controller的对象，如果你对请求不能被中央处理器接收，这个请求不会被执行拦截器内容。

简而言之，**拦截器拦截Controller类方法执行，过滤器过滤servlet请求响应**。

![chrome_S0OpZ6AVf9.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_S0OpZ6AVf9.png)

## springmvc映射器

我们之前在spring中使用容器对象时总是要扫描配置文件，获取bean实例对象(getBean方法)，为什么现在springmvc容器现在不用写getBean也能获取到Bean对象呢？

是因为在该容器中存在一些对象，帮我们完成了这些工作。

浏览器发起请求some.do，经过猫之后交给了中央调度器，再由中央调度器转发给了**处理器映射器**（框架把实现了HandlerMapping接口的类都叫做映射器，可以有多个）（根据请求地址从容器中拿到MyController处理器对象）。框架把找到的处理器对象放到一个叫做处理器执行链（HandlerExecutionChain)的类保存

HandlerExecutionChain类中保存着：

1. 处理器对象（MyController)

2. 项目中所有的拦截器（一个List集合类）

Dispatcherservlet把2中的HandlerExecutionChain中的处理器对象交给了处理器适配器对象〈可以有多个）
处理器适配器：springmvc框架中的对象，需要实现Hand1erAdapter接口。
**处理器适配器**作用：执行处理器方法，调用MyController.dosome()方法，得到返回值ModelAndView。

中央调度器拿到返回值交给**视图解析器**。

它的作用是使用前缀、后缀组成视图完整路径。


