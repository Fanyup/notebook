# 完善登录功能

之前的问题，就算不登录直接输入也可以进入到系统里。我觉得应是没有将静态资源那文件放到WEB-INF下。

可以使用过滤器(web)或拦截器(spring)来实现，进行用户请求拦截。我们这里我们用前者方式。

1. 创建自定义过滤器LoginCheckFilter

2. 在启动类（主类）上加入注解@ServeltComponentScan开启组件扫描器

3. 完善过滤器的处理逻辑

```java
//检查用户是否已经完成登录
@Slf4j
@WebFilter(filterName = "loginCheckFilter",urlPatterns = "/*")
public class LoginCheckFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse)servletResponse;
        HttpServletRequest request = (HttpServletRequest)servletRequest;
      log.info("拦截到请求：{}",request.getRequestURI());
      filterChain.doFilter(request,response);
    }
}
```

执行成功：

![idea64_ml3JyrPImB.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_ml3JyrPImB.png)

## 问题：过滤器启动失败

检查了一下，发现是在添加定义过滤器注解时拦截请求的url路径笔误写错了，这里应该我们测试的是拦截所有路径下的，urlPatterns="/*".

## 过滤器具体逻辑

获取本次请求uri-->是否需要处理-->判断登录状态-->放行

一个问题：我们在定义不需要处理的请求路径是如果用通配符的方式/employee/**方式，如果出现了/employee/index.html则不适用了。该怎么办呢？

这时我们可以用spring框架为我们提供的**路径匹配器**，为匹配方法提供匹配功能。

也就是一个工具类。它是支持通配符的写法的。可以在方法外，类里面最上面声明这个成员变量。

```java
public static final AntPathMatcher PATH_MATCHER = new AntPathMatcher();
```

详细写法：

```java
//1.获取本次请求的URI
        String requestURI = request.getRequestURI();
        //判断本次请求是否需要处理①定义请求路径
        String[] urls = new String[]{
                //直接放行：静态资源，登录登出页面
                "/employee/login",
                "/employee/logout",
                "/backend/**",
                "/front/**"
        };//调用判断方法
        boolean check = check(urls,requestURI);
        //②若url不需要处理，直接放行
        if (check) {
            filterChain.doFilter(request,response);
            return;
        }
```

定义判断方法，用到刚刚那个路径匹配器新创建的对象：

```java
    public boolean check(String[] urls,String requestURI){
        //路径匹配，检查本次请求是否需要放行
        for (String url : urls) {
            boolean match = PATH_MATCHER.match(url,requestURI);
            if (match) {
                return true;
            }
        }
        return false;
    }
}
```

URL是URI的子集，URI统一资源标志符，**不管用什么方法表示，只要能定义一个资源，就叫URI**。有时编号方式。

知乎上看到一个很形象的例子：**去村子找个具体的人（URI）**，如果用地址：某村多少号房子第几间房的主人 就是URL， 如果用身份证号+名字 去找就是URN了。**目前WEB上流行URL的形式。**

URL用地址方式。

```java
 //③需要处理，判断登录状态，已登录则放行
        //用之前存入参数的会话Session对象获取登录用户。
        if (request.getSession().getAttribute("employee") != null){
            filterChain.doFilter(request,response);
            return;
        }
        //5.未登录则返回未登录结果，通过输出流方式向客户端页面响应数据
        response.getWriter().write(JSON.toJSONString(R.error("Noting")));
        return;
```

**总结**：

这里我们先用HttpServletRequets接口抽象方法获取请求的URI地址，再添加可以**不需要判断而直接通过**的url字符串数组，其中添加一个需要**判断的筛选方法**，用到spring框架的AntPathMatcher,调用其方法返回布尔值类型，若真则通过；通过后再查询**该用户状态，是否已登录**，已登录则最终放行。

可以适时在每一步添加log日志，方便我们日后调试监测。

# 新增员工

其实就是将员工存入到mysql员工表中，需要注意的是我们给username添加了唯一约束，也就是登录账号，它是唯一的。

![MySQLWorkbench_eQ4tONMVT0.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_eQ4tONMVT0.png)

![MySQLWorkbench_IOQANVwYQ0.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_IOQANVwYQ0.png)

上面这个图还需注意的是status默认值是1哦，就是正常，未被禁用的意思。

Controller层：

```java
 //添加员工
    @PostMapping
    public R<String> save(HttpServletRequest request,@RequestBody Employee employee){
        log.info("新增员工，员工信息：{}",employee.toString());
        //默认给一个初始密码，不要用明文的！md5加密处理！(getBytes将它变成数组）
        employee.setPassword(DigestUtils.md5DigestAsHex("123456".getBytes()));
        employee.setCreateTime(LocalDateTime.now());//获取当前系统时间
        employee.setUpdateTime(LocalDateTime.now());
        //获取当前用户id
        Long empId = (Long) request.getAttribute("employee");
        employee.setCreateUser(empId);
        employee.setUpdateUser(empId);

        employeeService.save(employee);

        return R.success("新增员工成功");
    }
}
```

测试报错

### 问题：空指针异常

![idea64_cb3d7gnrIA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_cb3d7gnrIA.png)

我去..debug后总算发现问题了，我申请调的request请求根本不是传入参数的employ这个的，漏掉了getSession了，如果不是从浏览器对话存储中调取那你无论调取多少次返回值都会为空！

![idea64_wklQsRuLJP.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_wklQsRuLJP.png)

再debug走一下：

![idea64_jTyST5Rqof.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_jTyST5Rqof.png)

最终mybatis-plus帮我们封装的Iservice接口（由我们定义的service接口继承）中有save方法，自动调用sql语句存储到数据库，哈哈..也就是说这个功能的实现只经历了Controller层和Service层，而不用我们书写mapper层和数据库打交道辽，好方便。

![vQZkdpXTDi.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/vQZkdpXTDi.png)

数据库添加成功：

![MySQLWorkbench_ZCDMP8riHS.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_ZCDMP8riHS.png)

雪花算法同Mybatis-Plus默认设置的id.Type为ASSIGN_ID有关，我们之前在yaml配置文件中写过：

![idea64_gLkUiGxOzN.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_gLkUiGxOzN.png)

### 待优化问题：全局异常捕获

500异常：服务器内部异常

username重复，即账号已存在的报错。

解决方法一：trycatch捕获异常（重复写，不方便，不推荐）

解决方法二：**全局异常捕获**（就是我们之前讲过的**aop面向切面**思想）

基于底层代理，代理controller

在common目录下写一个新的普通类，加上**@ControllerAdvice注解**让它变得不同。(指定拦截哪些Controller，預設對所有Controller 生效，也可以指定特定的)

注解细节使用查询：[这里面对于常用注解还挺详尽的](https://ithelp.ithome.com.tw/m/articles/10270418#:~:text=%40RestController%20%3A%20%E4%BD%9C%E7%94%A8%E7%9B%B8%E7%95%B6%E6%96%BC%40,%E5%85%AD%E5%80%8B%E9%87%8D%E8%A6%81%E7%9A%84%E5%8F%83%E6%95%B8%E3%80%82)

**再加一个注解@ResponseBody**，因为方法最终要返回JSON数据的。它就是把最终的结果数据封装成JSON再返回。

**再加一个Lombok提供的@Slf4j**方便输出日志。

别忘了**在方法上加@ExceptionHandler**来指定捕获的异常是什么。

先简单搭一个架子，日后再完善它：

```java
@ControllerAdvice(annotations = {RestController.class, Controller.class})
@ResponseBody
@Slf4j
public class GlobalExceptionHandler {

    //异常处理方法，拦截异常
    @ExceptionHandler(SQLIntegrityConstraintViolationException.class)
    public R<String> exceptionHandler(SQLIntegrityConstraintViolationException ex){
        log.error(ex.getMessage());
        return R.error("失败了");
    }
}
```

日志返回异常错误信息：它就是ex.getMessage()返回的信息。

![idea64_k0nZaXjLyk.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_k0nZaXjLyk.png)

username重复，不唯一！

接下来**完善它**：查询检索error报错信息中的关键字。

现在基本使用请求-响应模式了。

# 员工信息分页查询
