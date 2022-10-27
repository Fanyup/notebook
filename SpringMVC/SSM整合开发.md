# 实现步骤

## 1、大纲（基本思路）

使用mydbl库下的student表（其中定义主键id值自增）

![MySQLWorkbenchO7GGBxUBkopng](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_O7GGBxUBko.png)

1. 新建一个Maven的web项目

2. **加入依赖**
   
   - springmvc,spring,mybatis三个框架的依赖加入到pom.xml中
   
   - jackson依赖（转换json的）
   
   - mysql驱动（也就是依赖）
   
   - druid连接池
   
   - jsp,servlet依赖

项目一大的话依赖就很多。

3. **写web.xml**
   
   1. **注册前端（中央）控制器**DispathcherServlet。，
      
      - 创建springmvc容器对象，才能取创建Controller对象。
      
      - 它是Servlet，创建它才能接收用户的请求
   
   2. **注册spring的监听器:ContextLoaderListener**
      
      - 创建spring的容器对象，它通过读取配置文件，才能创建service，dao等对象
      
      （这里是注解扫描器吗？没什么印象，稍后看一下）
   
   3. **注册字符集过滤器**，解决post请求乱码的问题。
   
   4. **创建包**，Controller包，service，dao，实体类包名创建好
   
   5. **写springmvc,spring,mubatis的配置文件**。
      
      1. springmvc配置文件
      
      2. spring配置文件
      
      3. mybatis主配置文件
      
      4. 数据库的属性配置文件（xxx.properties)
   
   6. 写代码，dao接口和mapper（sql映射）文件，service和实现类，controller，实体类（存放属性的）。
   
   7. 写jsp页面。

## 2、pom.xml加入依赖

原先模板中build标签里的插件全删掉，jdk编译版本改为1.8，还有页面三行也删掉，用不到。

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <!-- jsp依赖 -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2.1-b03</version>
      <scope>provided</scope>
    </dependency>
    <!--springmvc依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
    <!--事务相关的-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
    <!--jackson（json转换相关的）-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
    </dependency>
    <!--mybatis的,spring整合用到的-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>
    <!--mybatis的-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>8.0.28</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.9</version>
    </dependency>
    <!--druid连接池的-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.12</version>
    </dependency>
  </dependencies>
```

spring核心ioc依赖在springmvc中都有。

尼玛的，看了一个人的笔记导的依赖全爆红，不知道问题出在哪里。后来去看了网课的doc文档，就成了...

先说一下基本思路：maven会先从本地仓库中找jar依赖包，如果没有，才会去中央仓库找（这里我们在conf下的settings文件中设置了中央仓库的镜像地址，米错就是阿里和腾讯，所以你导不进去很可能不是和maven仓库的问题）

E:\BaiduNetdiskDownload\01-文档\SpringMVC课程文档

**build下加插件，和resource编译时指向文件路径。**

#### 报错：编译插件导入失败问题

草泥马的，JDK1.8编译插件maven-compiler-plugin怎么都安装不上，查了一下，原来要加个组id

参见这一篇文章：

[解决报错问题](https://www.jianshu.com/p/9466132d51c6)

```xml
 <build>
    <resources>
      <resource>
        <directory>src/main/java</directory><!--所在的目录-->
        <includes><!--包括目录下的.properties,.xml 文件都会扫描到-->
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>

    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

## 3、写web.xml文件

这个文件在webapp项目下的WEB-INF下。为什么在它下面，因为它是对用户不公开的，外来用户无法访问，更安全。

首先我们还是来改一下它模板给我们的版本（初始太低了）。它可以从我们之前学习springmvc创建的项目中直接拷贝它。

### 1、注册中央调度器+指定配置文件路径+启动load1

**注册（创建）中央调度器，指定springmvc配置文件自定义路径，服务器启动时创建中央调度器对象。**

CV大师放一下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--（中央调度器）注册核心对象DispatcherServlet-->
    <servlet>
        <!--名字随便起-->
        <servlet-name>myweb</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--自定义springmvc配置文件的位置-->
        <init-param>
            <!--springmvc的配置文件的位置-->
            <param-name>contextConfigLocation</param-name>
            <!--指定自定义文件的位置-->
            <param-value>classpath:【不叫springmvc.xml了】conf/dispatcherServlet.xml</param-value>
            <!--类路径，所以应该是recourse目录，我们给它创一个-->
        </init-param>

        <!--希望的是在猫启动后就能创建Servlet对象-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>myweb</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
```

#### 1.1创建springmvc的配置文件

它用来**声明Controller后端控制器和其他web相关的对象**。

因为我们的配置文件有好几个，如果都把它们放在resources目录下会比较乱。那么springmvc.xml配置文件应该放到哪呢？

我们首先在src的main目录下创建java目录（编译classes的地方），和resources目录

![idea641n1B9Rvg0Npng](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_1n1B9Rvg0N.png)

可以在resources目录下创建一个子目录（如叫conf)，在它下面来创建我们的配置文件。原先我们给它去名叫springmvc.xml，**现在我们想给它改名了，它就叫conf/（目录下的）dispatcherServlet.xml**

(**这是一个相对路径**，**不带斜杠的**，通过我们用的都是不带斜杠的；因为一旦加上斜杠，它就是从端口号下直接找了！)

### 2、注册spring的监听器+指定配置文件路径

到这里大家还记得spring的**ioc面向切面编程**吗？（面向切面，好像有一毛钱关系）

我们要**指定自定义的spring配置文件的位置**。这里就和面向切面ioc有关系了！

```xml
  <!--注册spring的监听器+指定配置文件路径-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:conf/applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

我的理解是，有了监听器，你对springmvc容器才能连上spring容器，他俩之间才能进行交互，数据通信。

这里我们给 **<mark>spring配置文件命名为：applicationContext.xml；</mark>**

注意它和springmvc配置文件路径指向不同的区别是一个是一个是`<context-param>`标签，一个是`<init-param>`标签

#### 2.1创建spring配置文件

它用来**声明service，dao和工具类等对象**。

这里我们说创建就是真的在resources/conf/目录下创建想要的.xml配置文件，**只不过它是穿插在编写web.xml文件的过程中。**

![chromeKBimM3ArAgpng](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_KBimM3ArAg.png)

### 3、注册字符集过滤器filter

**防止post乱码问题**

filter的name名字一般是类名就行了（首字母小写）

再指定它用到的三个属性（你可以直接去CharacterEncodingFilter过滤类中找它）（快捷键Ctri+左单击）：encoding，和其他两个布尔类型的属性。

![idea64_Ru3x6SqqHA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_Ru3x6SqqHA.png)

并value填写定义它的编码方式。布尔两个都设置为真，意思是说强制request请求和response应答都是用utf-8的编码。

```xml
  <!--注册字符集过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>utf-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceRequestEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
    <init-param>
      <param-name>forceResponseEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
```

最后加上filter-mapping标签，**指定 /* 下都强制遍历一遍。**（大概是这个意思）

```xml
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

注意细节：**这个映射实在filter标签外指定的**。

## 4、创建包

把整个结构中的包都创建好，目的是把程序中的类都安置好位置。

domain（实体类的）

![idea64_zHck3zwajV.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_zHck3zwajV.png)

OK，这样我们就可以继续具体编写配置文件了。

## 5、具体编写配置文件

首先先写springmvc这个，内容少，简单一点

### springmvc配置文件

我们要用注解方式来写springmvc项目，那**1、注解的组件扫描器**必不可缺呀！

所以我们第一步要先声明组件扫描器，指定扫描controller包下的（全类名），和与它配套的**2、视图解析器**（配置前缀和后缀）。

至于这一块为什么要用value来赋值呢？因为它是set注入。这两个属性都是字符串类型的。

![idea64_7ZeBqvfxMt.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_7ZeBqvfxMt.png)

```xml
//applicationContext.xml
<!--spring的配置文件-->
    <context:component-scan base-package="com.bjpowernode.controller"/>
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

视图解析器自定义路径，如果没有我们可以自己建一个目录~

最后我们需要**3、注解驱动**（注意这里有很多个，要选结尾是mvc的）

注解驱动的大意大概就是启动注解方式将中央处理器接收到了用户请求转发交给后端处理器。不声明注册它它是不会执行的。

响应ajax请求和解决静态资源访问问题都要用到注解驱动，不用它会产生冲突。

```xml
<mvc:annotation-driven/>   
<!--记得它是在外面的，另起一行-->
```

### spring配置文件

也就是我们放在resources目录下conf下的appplicationContext.xml

**声明数据源，连接数据库**。现在conf下**1、创建一个.properties配置文件**存放jdbc信息。

```properties
jdbc.url=jdbc:mysql://localhost:3306/mydb?characterEncoding=utf8&useUnicode=true&useSSL=false&autoReconnect=true
jdbc.username=root
jdbc.password=xxxx
```

创建好后，记得在spring配置文件里声明该文件的位置👇

```xml
<context:property-placeholder location="classpath:conf/jdbc.properties"/>
```

#### 报错：配置文件识别失败

![idea64_mOuopghkGH.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_mOuopghkGH.png)

问题：resources忘记声明是资源目录了。👆

声明后，🆗

![idea64_yZzzz1SPeq.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_yZzzz1SPeq.png)

**2、声明数据源、连接数据库。**

他有两个**固定的属性需要声明**init-method和destroy-method

```xml
    <!--声明数据源，连接数据库-->
    <bean id="DataSource" class="com.alibaba.druid.pool.DruidDataSource"
          init-method="init" destroy-method="close">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

写完后先**发布猫部署Deployment该项目**测试一下。（-exploded)

发布成功没问题就可以继续往下写了。

#### 新：合并细节【配置文件间】的连接修改

##### 1、创建SqlSessionFactory对象

首先我们本项目中把spring配置文件命名为applicationContext.xml

但这里我们加入了声明数据源并连接数据库的步骤，其实这一步本来是在mybatis主配置文件中进行的。查看声明文档：[MyBatis中文网](https://mybatis.net.cn/getting-started.html)

```xml
    <!--通过bean标签声明SqlSessionFactoryBean目的是创建SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="DataSource"/>
        <property name="configLocation" value="classpath:conf/mybatis.xml"/>
    </bean>
```

![chrome_KfTBzH5CeX.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_KfTBzH5CeX.png)

![n1ytn63EdF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/n1ytn63EdF.png)

##### 2、注册mybatis的扫描器，创建dao接口实现类对象

我们之前讲的mybatis框架dao动态代理是通过测试方法模拟，调用SqlSession对象的静态方法getMapper(dao接口.class)获取到对应的dao接口实现类对象。

现在在该配置文件中通过bean标签方式创建。

![idea64_meOkyERHe1.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_meOkyERHe1.png)

我看了一下，倒2行的那个sqlSessionFactoryBeanName方法就是setter方法，this.x = x，素这样。

```xml
 <!--声明mybatis的扫描器，创建dao接口实现类对象-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="basePackage" value="com.bjpowernode.dao"/>
    </bean>
```

##### 3、service的注解扫描

在spring中通过aop面向切面（AspectJ注解创建代理对象）和注解的方式创建service对象

**复习一下**：spring针对Bean管理中创建对象可以用注解的方式：

![MarkText_QpTbPHtatv.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MarkText_QpTbPHtatv.png)

开启注解扫描器就知道哪些包下的类需要扫描了。（完成对service对象的创建工作）

```xml
 <!--声明service的注解@Service所在的包名-->
    <!--开启该注解扫描-->
    <context:component-scan base-package="com.bjpowernode.service"/>

    <!--事务配置：可以用注解的配置，也可以用AspectJ配置，二选一-->
```

**事务配置**：可以用注解的配置，也可以用AspectJ配置，二选一。

可以留到以后配。程序基本调式OK之后再加事务功能也OK。到这里我稍微去温习了一下spring的事务内容，注解是我们常用的，在spring配置文件中声明后可以通过在类上加注解方式配置事务。

### mybatis的主配置文件

目前有什么：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--spring的配置文件-->
    <context:property-placeholder location="classpath:conf/jdbc.properties"/>
    <!--声明数据源，连接数据库-->
    <bean id="DataSource" class="com.alibaba.druid.pool.DruidDataSource"
          init-method="init" destroy-method="close">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--通过bean标签声明SqlSessionFactoryBean目的是创建SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="DataSource"/>
        <property name="configLocation" value="classpath:conf/mybatis.xml"/>
    </bean>
</beans>
```

### 至此，程序基本骨架写好了

![idea64_H0EPRv4za0.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_H0EPRv4za0.png)

（我们把它放到Git服务器上），接下来可以写代码了。

## 6、开始写代码

功能代码写两个：查询和添加数据

- dao接口

- sql映射文件（mapper文件）执行sql代码

- controller后端控制器接受请求

- service接口处理业务逻辑（接口的实现）(spring)，及它的实现类

### 1、domain和dao目录部分

**1、先写实体类Student**（实体类就是储存数据的）

```java
public class Student {
    private Integer id;
    private String name;
    private Integer age;//及它的getter和setter方法
```

2、dao接口写抽象方法；

```java
public interface StudentDao {
    int insertStudent(Student student);//传输实体类中的参数，set传值
    List<Student> selectStudents();//查询，无需传参
}
```

3、写**该dao接口的**mapper文件（注意这里：还得去从刚刚的doc官方文档找一下规范）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bjpowernode.dao.StudentDao">
    <select id="selectStudents" resultType="Student">
        select id,name,age from student order by id desc
    </select>

    <insert id="insertStudent">
        insert into student(name,age) values(#{name},#{age})
    </insert>
</mapper>
```

#### 关键细节展示

编写mapper文件

![idea64_9YhZlqtams.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_9YhZlqtams.png)

![idea64_Bjs02l1TDQ.png](C:\Users\up\AppData\Roaming\marktext\images\b72162609349bb4821fd352f2e7732b7e3c76afa.png)

### 2、service目录部分

service接口：

```java
public interface StudentService {
    int addStudent(Student student);
    List<Student> findStudent();
}
```

写service接口及它的实现类时的小技巧👇

![hGFGFqWtzq.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/hGFGFqWtzq.png)

**4、service实现类细节**：

```java
//用注解快速创建Service对象
@Service
public class StudentServiceImpl implements StudentService {

    //引用类型【自动注入】@Autowired,@Resource等
    //就是对象类型属性注入，这样就能赋给service实现类调用的对象属性了
    //声明dao实现类对象
    @Resource
    //该接口的创建已在刚刚applicationContext.xml中bean声明创建了
    private StudentDao studentDao;

    @Override
    public int addStudent(Student student) {
        //调用dao实现类对象的方法
        int nums = studentDao.insertStudent(student);

        return 0;
    }

    @Override
    public List<Student> findStudent() {

        return studentDao.selectStudents();
    }
```

#### 新：合并细节【service实现类】

之前我们在学习mybatis时只是用测试类中的测试方法实现，现在SSM合并之后测试方法就“演变”成了service实现类（提供给业务逻辑层）。

### 3、controller部分和jsp文件

controller目录下创建了一个StudenController后端控制器类。

@Resource自动注入**可根据名称**注入对象属性，**也可以根据对象类型**注入。

```java
//别忘了加controller注解，否则它只是一个普通类
@Controller
//合并前缀
@RequestMapping("/student")
public class StudentController {
    @Resource
    //声明service,自动注入创建接口实现类对象
    private StudentService service;
    //注册学生
    @RequestMapping("/addStudent.do")
    public ModelAndView addStudent(Student student){
        ModelAndView mv = new ModelAndView();
        String tips = "注册失败";
        //调用service处理student
        int nums = service.addStudent(student);
        if (nums >0) {
            //注册成功
            tips = "学生：" + student.getName() +"注册成功";
        }
        //指定结果页面：添加数据+视图
        mv.addObject("tips",tips);
        mv.setViewName("result");//逻辑名称
        return mv;
    }
}
```

到目前位置有个问题，在哪里注入实体类的属性值？何时用setAttribute注入（写好后要**前端发请求**）

因为返回的视图是逻辑名称，所有要下创建一个jsp

![idea64_2hjE9fcCKf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_2hjE9fcCKf.png)

```jsx
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
result.jsp 结果页面，注册结果：${tips}
</body>
</html>
```

### 4、前端页面发起请求

在webapp目录下创建index.jsp首页，提供用户操作的功能入口

而不是WEB-INF目录下，因为它下面是对用户不公开的。

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String basePath = request.getScheme() + "://" +
            request.getServerName() + ":" + request.getServerPort() +
            request.getContextPath() + "/";
%>
<html>
<head>
    <title>功能入口</title>
    <base href="<%=basePath%>"/>
</head>
<body>
    <div align="center">
    <p>SSM整合的例子</p>
    <table>
        <tr>
            <td><a href="addStudent.jsp">注册学生</a> </td>
        </tr>
        <tr>
            <td><a href="listStudent.jsp">浏览学生</a> </td>
        </tr>
    </table>
    </div>
</body>
</html>
```

猫跑一下效果图如下：

![ApplicationFrameHost_GMKJcJpGLo.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_GMKJcJpGLo.png)

#### 1、编写注册页面

我们想写一个注册页面

![idea64_BRYQvCpLwy.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_BRYQvCpLwy.png)

没有加斜杠开头，因此加个base标签吧！

##### 加base标签：基访问路径（页面中的地址信息）

指定路径：

```html
<%
 String basePath = request.getScheme() + "://" + 
request.getServerName() + ":" + request.getServerPort() + 
request.getContextPath() + "/";
%>
```

再head标签里加上base标签的内容

```html
<head>
    <title>注册学生</title>
    <base href="<%=basePath%>"/>
</head>
```

同样的，它加了那首页也得加了。

**前端页面代码中不加斜杠，controller后端处理中是带斜杠的。**

测试问题：

##### 报错一：数据库连接失败

检查后发现pom.xml引入依赖时mysql驱动依赖版本错了，从mybatis笔记中修改后连接成功。

![idea64_8irdP04mRX.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_8irdP04mRX.png)

```xml
<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
    </dependency>
```

##### 报错二：注册失败

草泥马了，一堆错误。一步一步检查后发现dao实现类方法调用返回竟然没调用。

![idea64_ZVKHtTEFFI.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_ZVKHtTEFFI.png)

忘记该返回值了。修改return nums;后大功告成~这回跑猫执行成功了！

![chrome_WH8vOw8rpN.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_WH8vOw8rpN.png)

皇天不负有心人，即使是跟着视频敲的，但还需要稍后自己再推导一遍将知识转换成自己的。以及经验教训是一定要写详细的开发文档，不然久了我真的会忘...

##### 细节：经验教训

我发现数据库里我加了两次相同的数据，也就是说报错二发生时是service层以及成功调用且执行了dao层的sql插入语句方法，只是没有成功返回nums值给最终的结果显示给用户，但插入信息这一步却是成功了的。

![chrome_nIMNKUEjPf.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_nIMNKUEjPf.gif)

用了一下ShareX的GIF录制功能（快捷键设Ctrl+Q)，感谢，伟大的发明...

##### 基本流程回顾

用户访问index.jsp，点击后跳转到addStudent.jsp，用post方式

发起请求（该请求action叫：）student(目录下的)/addStudent.do。而相应后端控制器会接受它并处理（service方法，调用dao接口实现类方法）。...执行完后向用户响应result.jsp（注册成功？注册失败？）

**这里的student/到底是什么**，其实执行后会发现它在url中出现了：

`http://localhost:8080/ch07_ssm/student/addStudent.do`

#### 2、编写查询功能

前端用ajax实现，要加入jq库文件，之前写了有，拷贝一下，创建一个web/appjs目录存放它。（原来之前学springmvc返回值void一直出错是没有导入js文件；其实这里用它的cdn也可以，但是我不会。）

js-3.4.1官方：[右键另存为可用](https://code.jquery.com/jquery-3.4.1.min.js)

它长这样👇

![idea64_G2SXdrTADD.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_G2SXdrTADD.png)

##### 引入jq文件+Controller控制器加处理方法

引入jquery文件，**注意:一定要<></>这么用，不然</>这样是不起作用的！**

```html
<html>
<head>
    <title>查询学生ajax</title>
    <base href="<%=basePath%>"/>
    <script type="text/javascript" src="js/jquery-3.4.1.min.js"></script>
</head>
<body>
```

Controller类加一个方法处理查询功能，响应ajax

1.pom.xml检查jackson依赖有没有（处理转换json的）

2.springmvc配置文件中有没有加注解驱动？（如果没有加，无法正常处理静态资源）

```java
 //处理查询，响应ajax
    @RequestMapping("/queryStudent.do")
    //不加就不是处理器而是普通类了
    @ResponseBody
    public List<Student> queryStudent(){
        //参数检查，简单的数据处理
        List<Student> students = service.findStudent();
        //该结果会被框架转成Json的数组
        return students;
    }
```

测试一下这个对外提供的接口（不是说他是Interface，而是可供外部即用户访问的），直接添加后缀访问，看能不能拿到数据，能拿到说明OK。`http://localhost:8080/ch07_ssm/student/queryStudent.do`

![chrome_rammTsCDsy.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_rammTsCDsy.png)

下面可以放心去写ajax请求了。

##### jsp页面编写

写个btn按钮做个单击事件，点击即查询数据。用Jquery来绑定事件（js库）（不完整，部分）（测试：有弹窗说明连接成功）

```html
<head>
    <title>查询学生ajax</title>
    <base href="<%=basePath%>"/>
    <script type="text/javascript" src="js/jquery-3.4.1.min.js"></script>
    <script type="text/javascript">
        $(function () {
            $("#btnLoader").click(function () {
                $.ajax({
                    url:"student/queryStudent.do",
                    type:"get",
                    dataType:"json",
                    success:function (data) {
                        alert("data="+data);
                    }
                })
            })
        })
    </script>
</head>
```

##### 404报错：js文件查找失败

跑猫时记得刷新一下，导入jd文件IDEA很可能没有刷新同步，所以控制端网络会报错404找不到js文件

![chrome_VvUJw8brAn.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_VvUJw8brAn.png)

阶段测试成功后可以继续写代码了。

拿到数据后我们就可以解析它了：取出数组数据，注入到tbody中。

![idea64_kmYzXjowkN.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_kmYzXjowkN.png)

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String basePath = request.getScheme() + "://" +
            request.getServerName() + ":" + request.getServerPort() +
            request.getContextPath() + "/";
%>
<html>
<head>
    <title>查询学生ajax</title>
    <base href="<%=basePath%>"/>
    <script type="text/javascript" src="js/jquery-3.4.1.min.js"></script>
    <script type="text/javascript">
        $(function () {
            $("#btnLoader").click(function () {
                $.ajax({
                    url:"student/queryStudent.do",
                    type:"get",
                    dataType:"json",
                    success:function (data) {
                        //清除旧的数据
                        $("#info").html("");
                        //增加新的数据
                        $.each(data,function (i,n) {
                            $("#info").append("<tr>")
                            .append("<td>"+n.id+"</td>")
                            .append("<td>"+n.name+"</td>")
                            .append("<td>"+n.age+"</td>")
                            .append("</tr>")
                        })
                    }
                })
            })
        })
    </script>
</head>
<body>
    <div align="center">
        <table>
            <thead>
            <tr>
                <td>学号</td>
                <td>姓名</td>
                <td>年龄</td>
            </tr>
            </thead>
            <tbody id="info">

            </tbody>
            <input type="button" id="btnLoader" value="查询数据"/>
        </table>
    </div>
</body>
</html>
```
