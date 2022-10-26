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
      <version>3.5.1</version>
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

我们要用注解方式来写springmvc项目，那**注解的组件扫描器**必不可缺呀！

所以我们第一步要先声明组件扫描器，指定扫描controller包下的（全类名）
