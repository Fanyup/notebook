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

在common目录下写一个新的普通类，加上 **@ControllerAdvice注解**让它变得不同。(指定拦截哪些Controller，預設對所有Controller 生效，也可以指定特定的)

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

```java
//异常处理方法，拦截异常
    @ExceptionHandler(SQLIntegrityConstraintViolationException.class)
    public R<String> exceptionHandler(SQLIntegrityConstraintViolationException ex){
        log.error(ex.getMessage());
        if(ex.getMessage().contains("Duplicate entry")){
            //用split以看空格为分界符进行字符串拼接
            String[] split = ex.getMessage().split(" ");
            String msg = split[2] + "已存在";
            return R.error(msg);
        }
        return R.error("未知错误");
    }
```

现在基本使用请求-响应模式了。

# 员工信息分页查询

过滤条件+分页查询

mybatis-plus提供了一个分页插件

创建一个配置类：

```java
//配置分页插件(interceptor”拦截器“)
@Configuration
public class MybatisPlusConfig {

    //DI注入依赖（组件），让Spring来管理它
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

接下来写controller处理层了（接收页面发送参数请求等）

![idea64_Fppp6kYHbJ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_Fppp6kYHbJ.png)

前端返回属性名让我们看清它想要干嘛：

![idea64_qd8gpK2wYV.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_qd8gpK2wYV.png)

也就是页面需要拿到的数据。由Controller转为json后返回给前端它就正好能够取到。

![idea64_WqlwukH4Lw.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_WqlwukH4Lw.png)

页面需要什么数据，我们就给它什么数据。通过下面请求发送可以知道我们的controller中要传入什么参数（page默认值1,当前页面1。pageSize为10，查10条）

![chrome_TPhIRKYLHx.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_TPhIRKYLHx.png)

情况2，如果用查询框时，get请求还会多发一个我们输入的自定义的name属性

写好大框架后：**记得，每步完成先测试！先搭大框架。** 走一步算一步而不用要等到最后一起，因为你不知道你的错误发生在哪里。这里我们搭建好框架后先用日志测试一下能否接收到传入参数信息。

```java
//员工信息分页查询
    @GetMapping("/page")//Restful风格
    public R<Page> page(int page, int pageSize,String name){
        //占位符一一对应
        log.info("page = {},pageSize = {},name = {}",page,pageSize,name);
        return null;
    }
```

成功获取请求，日志成功输出，参数封装没有问题。**注意：这里我总是忘记，如果你想debug断点调式就不能用正常允许，应该走debug模式才可以**。

### 搭好大框架后的具体代码

因为没有学过Mybatis-Plus，我理解这里创建条件构造器时构造了一个LambdaQueryWrapper类的对象，wrapper有封装皮，包纸的意思（这里要指定泛型，后面加了）。而且我们前面在根据用户名查询数据库时也用到了这个对象，之前我们调用的是该对象的**eq方法，它是等值查询，对于sql就是where name=...**

这里我们添加过滤条件，也和数据库相关sql代码有关，这里我们建议**使用like，也就是相似度查询。**

![idea64_eakdNr4hEC.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_eakdNr4hEC.png)

细节：

![idea64_0ZYtSVZhc1.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_0ZYtSVZhc1.png)

它的作用是，指定当name不等于空时，才会添加这句代码👆

这里我们**执行查询统统都用employeeService这个东东，调用它的方法了！** 它是什么东东呢？它是我们**最开始在controller类中DI注入的对象类型属性，它是我们定义的EmployeeService接口的对象（不是实现类，因为这里体现了多态）该接口继承IService这个父接口，查看它会发现里面有N多封装好的sql语句**（也就是说Mybatis-Plus帮我们封装好了，不需要我们再去与dao层和mapper层打交道）（其中注意：里面添置了很多Spring框架注解@Transational“事务”）

这里我怕我自己忘了，再重复一下@Bean注解和@Autowired注解两个是不一样功能的。前者是方法组件（声明）注入，后者是属性对象（常用）注入。

![idea64_W7YMigJ5Jv.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_W7YMigJ5Jv.gif)

```java
 //员工信息分页查询
    @GetMapping("/page")//Restful风格
    public R<Page> page(int page, int pageSize,String name){
        //占位符一一对应
        log.info("page = {},pageSize = {},name = {}",page,pageSize,name);

        //构造分页构造器（就是Page对象）
        Page pageInfo = new Page(page,pageSize);//第一页，查10条
        //条件构造器where name = ..动态封装过滤条件的，条件也可以没有（非默认）
        LambdaQueryWrapper<Employee> queryWrapper = new LambdaQueryWrapper();
        //添加过滤条件
        queryWrapper.like(StringUtils.isNotEmpty(name),Employee::getName,name);
        //添加排序条件
        queryWrapper.orderByDesc(Employee::getUpdateTime);
        //执行查询
        employeeService.page(pageInfo,queryWrapper);

        return R.success(pageInfo);

    }
```

不需要接收它，它内部会返回给PageInfo对象相应的records，total等等。

一共创建了两个对象：Page对象（实现page分页功能，也就是那个插件，mybatis-plus帮我们封装好的）和LambdaQueryWrapper对象（我理解是调用sql语句的）

## 问题：debug小插曲

我打好断点后debug执行时发现它并没有弹出经过，检查后发现是将断点打再了方法最上面的@GetMapping上，这时候红色的断点小表示也变成了禁用。因为先前没用过断点方式很熟练，所以我理解这里的问题是断点阶段了get发送的分页请求，需要手动申请通过，打在方法里后就解决了这个问题。

我在这儿还学到一个妙招，（其实不算）就是控制台在debug间切换的东西👇

![idea64_NGsTXYHRJS.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_NGsTXYHRJS.gif)

重点看查询结构：limit分页 orderby..DESC降序查询（添加到排序条件）

![idea64_GxcJM9mUaj.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_GxcJM9mUaj.gif)

重重点来看一下pageInfo对象里现在传入了什么值：👇

光标I放在它上面可以默认识别到的

![idea64_YW6sNmN1fx.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_YW6sNmN1fx.gif)

可以看到，无需返我们写回什么，值是被这个方法自动传入添加的。

## Bejson校验格式化

搜索[JSON校验格式化工具（Be JSON）](https://www.bejson.com/json/format/)，格式化，方便我们查看浏览器返回的响应内容（网络）

`Ctrl+回车自动查询`

![chrome_ViD6QMndZQ.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_ViD6QMndZQ.gif)

# 启用/禁用员工账号

注意：只有管理员admin才有权限。账户状态与status有关。普通用户（员工）是没有权限的。

请求方式：PUT

请求携带参数：id和status（注意，是以JSON的形式）

前端是如何发送AJAX请求的呢？

![chrome_Inqjm4ZKjr.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_Inqjm4ZKjr.png)

上面我说错了，是emabled...这个方法封装到了一个JSON文件当中。

本质上是**sql语句更新update操作**（0禁用，1正常）

我们希望在controller里做一个方法，它是一个**通用的update操作**，这样再以后【编辑】【启用】等操作时就能复用它了。

还是老规矩：搭简单整体框架测试一下：

```java
//根据id修改员工信息
    @PutMapping
    public R<String> update(@RequestBody Employee employee){
        log.info(employee.toString());
        return null;
    }
```

温习：传入的是json格式的对象类型属性，所以需要加入注解@RequestBody！

点击禁用，前端请求传的值只有id和status，我们后台打印日志可以看出是接收到了它的。

![chrome_O04LhwIgRi.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_O04LhwIgRi.gif)

## 搭好大框架后的具体代码

记得：默认Object，向下强制转换成Long型，因为setUpdateUser()方法只接收Long型

```java
    //根据id修改员工信息
    @PutMapping
    public R<String> update(HttpServletRequest request,@RequestBody Employee employee){
        log.info(employee.toString());
        //具体完善
        //记得：默认Object，向下强制转换成Long型，因为setUpdateUser()方法只接收Long型
        Long empId = (Long) request.getSession().getAttribute("employee");
        employee.setUpdateTime(LocalDateTime.now());
        employee.setUpdateUser(empId);//当前登录用户，可以通过当前session获取~
        employeeService.updateById(employee);
        return R.success("员工信息修改成功");
    }
```

## 问题：JS精度丢失

按照这个代码出现问题：未匹配到记录——Updates:0(未更新成功)

原因是因为传过来的Id与数据库中id不一致。JS的问题。

**JS只能保证前16位数组，而Long型传过来的值有19位，后3位精度丢失了。（被四舍五入处理了）**

**解决办法**：在服务器给页面响应JSON数据时进行处理，**将long型数据统一转为“String字符串”**。这样JS就不会对他进行处理了，因为他是字符串！

具体实现：

服务端给页面响应数据时用到了**Spingmvc中一个组件：消息转换器**。现在我们要在配置类里去扩展一个消息转换器。在它中使用提供的**对象转换器进行java对象到json数据**的转换。

而这个对象转换器（网课老师给了，不用我们自己写了）

把它放到common目录下：

```java
package com.itheima.reggie.common;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalTimeSerializer;
import java.math.BigInteger;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import static com.fasterxml.jackson.databind.DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES;

/**
 * 对象映射器:基于jackson将Java对象转为json，或者将json转为Java对象
 * 将JSON解析为Java对象的过程称为 [从JSON反序列化Java对象]
 * 从Java对象生成JSON的过程称为 [序列化Java对象到JSON]
 */
public class JacksonObjectMapper extends ObjectMapper {

    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    public JacksonObjectMapper() {
        super();
        //收到未知属性时不报异常
        this.configure(FAIL_ON_UNKNOWN_PROPERTIES, false);

        //反序列化时，属性不存在的兼容处理
        this.getDeserializationConfig().withoutFeatures(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);


        SimpleModule simpleModule = new SimpleModule()
                .addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))

                .addSerializer(BigInteger.class, ToStringSerializer.instance)
                //注意：序列化器👇将Long型数据转成String字符串
                .addSerializer(Long.class, ToStringSerializer.instance)
                .addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));

        //注册功能模块 例如，可以添加自定义序列化器和反序列化器
        this.registerModule(simpleModule);
    }
}
```

扩展消息转换器（不扩展是用springmvc默认提供的）我们现在扩展它，在把这个对象转换器注入到里面去，相当于扩充、自定义了。

**注意：是在之前config目录下的WebMvcConfig配置类下扩展。**

重写该父类里面一个方法。

![explorer_SDZNipsFdl.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/explorer_SDZNipsFdl.png)

```java
 //扩展mvc框架的消息转换器（实际上默认自带了几个）
    @Override
    protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        log.info("扩展消息转换器...");
        //创建一个新的消息转换器对象
        MappingJackson2HttpMessageConverter messageConverter = new MappingJackson2HttpMessageConverter();
        //设置对象转换器，底层使用Jackson将Java对象转为json
        messageConverter.setObjectMapper(new JacksonObjectMapper());
        //将上面的消息转换器对象追加到mvc框架的转换器容器（就是一个集合）中
        converters.add(0,messageConverter);//有优先级，0优先使用
    }
```

它在mvc配置类里面，也就是说，在这个项目启动时就会被调用（之前那个dispatcherServlet中央调度器，还记得吗，一启动就会on-load加载Servlet对象）

![idea64_QR082VsBEJ.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_QR082VsBEJ.gif)

# 编辑员工信息

动态通过js方式获取到url参数（get请求）

程序执行操作流程：

![chrome_FNHSLMrZiI.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_FNHSLMrZiI.png)

跳转的页面add.html是对用户开发的公共页面。

解析字符串，动态取出url的值（Js完成）（前端进行url分段处理）

```java
    //根据id查询员工信息
    @GetMapping("/{id}")
    //id变量在整个请求路径里面
    public R<Employee> getById(@PathVariable Long id){
        log.info("根据id查询员工信息");
        Employee employee = employeeService.getById(id);
        if (employee != null){
            return R.success(employee);
        }
        return R.error("没有查询到对应员工信息");
    }
```

![chrome_en0idma7Vx.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_en0idma7Vx.gif)

那个**data**其实就是success方法中将employee对象传到r.data里去了。

注意这里男女性别返回的是标志位，返回的性别并不是男女这两个字。

这里点击保存会发现成功了，我们何是写过这个方法？就是上面写的那个通用的update方法！
