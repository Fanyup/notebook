# 开发环境搭建

## 数据库环境搭建

1. 新建库reggie，导入.sql文件

![MySQLWorkbench_UtiCJbqZLk.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_UtiCJbqZLk.gif)

### 问题：找不到更新内容

记得右键刷新数据库，Refresh All，才能看见更新的表单。

## Maven项目搭建

pom.xml依赖添加+yml配置文件导入

### 经典问题：依赖又双叒叕导入失败

无数次的经验教训总结才能记得点什么...花了将近1h时间检查为什么springboot启动父项导入失败，最终发现虽然检查了settings的Maven，但我忘记修改了。原先也遇见过同样的问题，幸好知道问题在哪：我的3.8.6Maven版本似乎不支持当前IDE版本，换成3.6.3就可以使用了。所以说有些步骤真的不是走走形式表面功夫啊啊啊啊，必要的检查能减少工作查错时间。中途去拉了个粑粑，感悟了。

### 创建主程序并添加前端静态资源

```java
@Slf4j
@SpringBootApplication
public class ReggieApplication {
    public static void main(String[] args) {
        SpringApplication.run(ReggieApplication.class,args);
        log.info("项目启动成功...");
    }
}
```

主程序记得放在包的最外层，这样springboot才能扫描到该包及其子包的。

添加Lombok插件的@Slf4j注解可以快速创建日志，调用log.info("方便我们管理查看");

### 配置静态资源映射

由于Springboot默认在resources的static目录下查找静态资源，因此我们这里需要给它加一个**配置类@Configuration**。配置类我们统一放到java子目录下的config包里。**自定义addResourcesHandler资源映射器，指定静态资源路径**。由于经过target后的resources是直接在类路径下的，因此可以直接添加斜杠/+文件名查询（之前讲过，加了斜杠的相对路径就是从端口号8080后开始直接查找的。）

```java
@Slf4j
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    /*
    * 设置静态资源映射
    * */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        log.info("开始进行静态资源映射...");
        //资源处理器
        registry.addResourceHandler("/backend/**").addResourceLocations("classpath:/backend/");
        registry.addResourceHandler("/front/**").addResourceLocations("classpath:/front/");
    }
}
```

## 后台系统登录功能

发起请求，F12控制查看请求信息：

![chrome_0tERRDYaFX.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_0tERRDYaFX.png)

浅读一下/backend/login/login.html的数据，了解我们应该响应会前端什么数据。（json格式）

![idea64_gC4bnedLCT.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_gC4bnedLCT.png)

后端大致思路：Controller控制层-->Service业务逻辑层-->Mapper(Mybatis-plus，也就是Dao层)-->数据库

1. 创建实体类Employee，和employee数据库表进行映射。因为数据封装一一对应，属性名最好与一段名一一对应上，免得麻烦修改嗷！
   
   其中关于getter,setter等等方法用Lombok封装好了，添个注解@Data就完事啦~

关于表中idNumber与数据表中id_number不一样怎么办，没关系，我们.yml配置类中配置了驼峰命名转换：

```yml
mybatis-plus:
  configuration:
    #在映射实体或者属性时，将数据库中表名和字段名中的下划线去掉，按照驼峰命名法映射
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: ASSIGN_ID
```

2. 创建整体包结构
   
   ![idea64_8nZnzrvkre.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_8nZnzrvkre.png)

有了框架才方便日后的完善。就像一个目录一样，才能继续往里头填写东东。

3. 创建Mapper接口（用到Mybatis-plus）

![idea64_LOKYngj6io.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_LOKYngj6io.png)

我们可以看一下它的父接口：

![idea64_rN8ThsAXgP.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_rN8ThsAXgP.png)

总之就是超级多。

4. 创建service接口，该接口也是继承mybatis-plus的东东，一个父接口，叫IService，同样定义泛型为指定实体类。并创建它的实现类，别忘了加上@Service注解，由Spring来管理它。

![idea64_SgTzloNXDE.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_SgTzloNXDE.png)

继承父类，实现父接口。

我想这里继承它的父类其中指定泛型，是因为其父类中由两个抽象方法，分别是获取到刚刚定义的BaseMapper与实体类。

5. 创建Controller

```java
@Slf4j
@RestController
@RequestMapping("/employee") //对应刚刚我们看到的/employee/login
public class EmployeeController {

    //注入接口对象类型属性，类型是接口
    @Autowired
    private EmployeeService employeeService;
}
```

6. 导入返回结果类R（自定义名字咯）

它是一个通用的结果类，**之后我们要写很多Controller，所有返回结果都包装成此类型**，返回给前端页面。

![idea64_7i592RV2hy.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_7i592RV2hy.png)

用泛型可以增强它的通用性。

### Controller代码编写（逻辑处理）

员工登录

![idea64_MRrYTf7frk.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_MRrYTf7frk.png)

1. 我们在数据库中将密码进行了md5加密，所以首先要将页面提交过来的密码处理成md5

2. 再根据用户名查数据库。

3. 找到后下一步判断数据库密码比对。

4. 判断当前员工状态status（是否锁定账号）
   
   一共四步。

**先理清楚业务逻辑，再进行编码！**

```java
//1.页面提交密码进行md5加密处理
        String password = employee.getPassword();
        password = DigestUtils.md5DigestAsHex(password.getBytes());
```

wrapper“包装纸，封装”，用Mybatis-plus东东包装一个查询对象。

并添加查询条件（根据用户名，等值查询）。

用户名字段添加了**唯一约束**👇，因此可以调用getOne方法查出包装对象唯一的数据。

![MySQLWorkbench_zAmuTGTAk9.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_zAmuTGTAk9.gif)

Inspector"检察员"

```java
 //2.根据用户名查询数据库,包装查询对象
        LambdaQueryWrapper<Employee> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Employee::getUsername,employee.getUsername());
        //查询唯一数据（因为在数据表中索引定义用户名唯一），返回成Employee对象
        Employee emp = employeeService.getOne(queryWrapper);
```

(这是Mybatis-plus的东东..)

```java
//3.是否查到数据，没有返回失败结果
        if (emp == null) {
            return R.error("登录失败");
        }
        //4.密码比对
        if (!emp.getPassword().equals(password)) {
            return R.error("登录失败");
        }
        //5.查看员工状态（是否可用）
        if (emp.getStatus() == 0) {
            return R.error("账号已禁用");
        }
        //6.确定登录成功，将用户id放到session里面
        request.getSession().setAttribute("employee",emp.getId());

        return R.success(emp);//这个对象就是我们刚从数据库中查出来的那个emp对象
```

debug加打断点测试一下：

### 初步debug断点测试

看一下数据封装有没有问题。（随便打一个密码）Fn+F8下一步，F7进入它。

![idea64_VY88gsP5m6.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VY88gsP5m6.gif)

### 问题：找不到用户名情况

yaml配置文件忘记修改数据库密码了...

解决问题后，可以根据admin的这个username查到employee数据库表中对应的值了。此时其实已经返回给定义的emp实体对象。

![idea64_QCRvaZjAmz.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_QCRvaZjAmz.gif)

![MySQLWorkbench_d9QJhA82PR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_d9QJhA82PR.png)

### 问题：关于前端请求发送超时问题

![chrome_IyQcRg2ICT.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_IyQcRg2ICT.png)

默认10s未得到响应即为超时，但我们在后面的debug断电调式中经常要很长的时间，所以需要修改它。

登录成功后返回的数据会会存到（应用下）浏览器本地storage(储存）中。返回的是json格式数据。

![idea64_xkVsYz1CaP.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_xkVsYz1CaP.gif)

## 后台系统退出功能

```java
 //员工退出
    @PostMapping("/logout")
    public R<String> logout(HttpServletRequest request){
        //清理Session中保存的当前登录员工的id
        request.getSession().removeAttribute("employee");
        return R.success("推出成功");
    }
```

注意：session会话存储与浏览器本地storage存储是两个东西！它之所以浏览器显示没了是因为前端代码中我们声明：当ajax请求成功删除方法code=1时，也会执行logout方法，删除它。

![chrome_2Jh92oIvo8.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_2Jh92oIvo8.gif)

![idea64_pROaqyluj7.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_pROaqyluj7.png)

静态页面更新可以用dev-tools插件工具的热更新Ctrl+Fn+F9
