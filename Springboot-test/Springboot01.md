# SpringBoot

# 问题

感觉关于spring容器对象创建不是很理解，也就是servlet相关。我理解是获取容器对象后，才能调用容器中的某些java类对象实例，也就是说你不获取spring容器，就像没有指向bean管理的xml文件的，没有spring容器，无从创建对象。

ss需要写大量xml配置文件，还需要配置各种对象，把使用的对象放到spring容器中才能使用对象。

springboot解决了这个痛点，能够实现**spring+springmvc**功能，不需要写它俩的配置文件了。常用的框架和第三方库都已经配置好了。开发效率高。

# 1、什么是JavaConfig？

是spring提供的，可以使用Java类当作配置文件来用，**用纯Java方式**来配置创建容器。代替xml配置文件。

在这个类中可以创建java对象，把对象注入到spring容器中。用它需要2个注解的支持：

1. `@Configuration`
   
   放在一个类的上面，表示这个类是作为配置文件使用的。（相当于xml文件）

2. `@Bean`
   
   声明对象，把对象注入到容器中。 

通过示例来说明：

### 示例

创建maven时我们未使用任何模板，这里我们加入了spring依赖，和junit测试库依赖，插件加入了maven编译插件。

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.1</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
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

一切准备条件处理好后首先得有个自定义的java普通类，再使用两个注解让它变成java配置文件类，代替xml配置文件的地位。

```java
/*
* 表示当前类是作为配置文件使用的。就是用来配置容器，获取对象的！
* */
@Configuration
public class SpringConfig {

    //创建方法，方法的返回值是对象。在方法上加注解@Bean
    //方法的返回值对象s1就自动注入到容器中。
    @Bean
    public Student createStudent(){
        Student s1 = new Student();
        s1.setAge(20);
        s1.setName("张三");
        s1.setSex("男");
        return s1;
    }
}
```

#### 测试一下

![idea64_39fgRYQLSN.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_39fgRYQLSN.png)

（注：这里只列举了关键步骤即javaconfig配置文件创建+测试方法，关于实体类Student还是老方法，声明它的setter和getter方法及重写toString方法打印。

![idea64_luwG5ZZ7ay.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_luwG5ZZ7ay.png)

```java
public class MyTest {

    @Test public void test01(){
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        Object student = ctx.getBean("createStudent");
        System.out.println("student="+student);
    }
}
```

**注意：@Bean不指定对象的名称，默认方法名是id**。就是bean标签中的属性id名。你也可以指定它。@Bean(name="定义我")

测试成功，同样能获取bean对象（我们定义它返回实体类型对象，会自动打印它的toString方法）

```html
student=Student{name='张三', age=20, sex='男'}
```

### @ImportResource

是用来导入其他的xml配置文件的，**等同于xml文件的resources**,配置文件导入叠加一起用。

`<import resources`

==

`@ImportResource(value ="ctasspath:resource目录下的其他配置文件.xml"）`

这个value是一个数组，可用大括号`{"a.xml","b,xml"}`形式导入

### @PropertyResource

"所有物，财产，财务"

让配置文件**读到(指向)properties属性配置文件的**。最常用的是把我们jdbc数据库配置信息放到它里面。

`@Value(value='${key}')`

但有它们两个远远不够，因为我们示例二创建的对象是用注解方式创建的（我们现在统统不用xml配置文件方式啦！）那你的配置文件（现在是javaconfig类）要找到你用注解方式创建的对象，得通过组件扫描器吧？那用注解的组件扫描器就是在配置文件类上加上@ComponentScan(basePackages="指定扫描的(全限定)包名")

```java
@Configuration
@ImportResource(value={"classpath:applicationcontext.xml'
@PropertySource(value="c1asspath：config.properties")
@ComponentScan(basePackages = "com.bjpowernode.vo")
public class SpringConfig{
}
```

要按以前xml配置文件形式：

```xml
<context:property-placehotder location="ctasspath:config.properties"/>
<context:component-scan base-package="com/bjppyyecppge.vo"/>
<import resource="ctasspath:beans.xml"/>
```

# 2、正式入门

其实springboot就是spring+springmvc，核心还是ioc容器。把对象交给容器创建。（注入）

### 特点

特点：

- 创建spring应用

- 内嵌的tomcat,jetty,Undertow服务器
  
  不需要你安装tomcat，它自己带了一个

- 提供了**starter起步依赖**，简化应用的配置。
  
  这是它的名字。比如使用mybatis框架，需要在spring项目中配置Mybatis的对象SqlSessionFactory，Dao的代理对象。
  
  现在在springboot项目中，在Pom.xml里面，加入一个mybatis-spring-boot-starter依赖就🆗了~

- 尽可能去配置spring和第三方库，自动配置。把spring和第三方库中的对象都创建好，放到容器中，开发人员可以直接使用

- 提供了健康检查、统计（时长等）、外部化配置（.properties这些）

- 不用去生成代码，不用使用xml做配置

### 如何在IDEA中创建springboot项目？

方式一：使用springboot提供的初始化器，它就是一个向导。

springboot也是基于maven管理的，所以会有pom.xml配置文件。

使用[https://start.spring.io](https://start.spring.io)，国外地址失败是可以用[https://start.springboot.io](https://start.springboot.io))镜像地址。不仅可以在IDEA中添加，也可以直接在浏览器栏上输入。它会生成一个zip压缩包Model，可在IDEA中import Model导入它使用~

注意：必须实在联网状态下

方式二：直接maven，不用向导，创建项目。

不用选骨架，引入父依赖和起步以来，自己建相关内容。

### 写一个带web功能的小例子

内嵌了猫服务器，不需要我们自己部署了~

👇Application这个主启动类的名字是是自定义的。类里包含一个主方法来启动。（非web，web项目都是它）

![idea64_sN5UrOHUke.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_sN5UrOHUke.png)

不再需要写中央调度器的配置了，直接专注于业务逻辑就好，Controller这些。

## @SpringBootApplication

一定会有的主类的注解。

**复合注解，是由多个注解的功能组成的。**
**@SpringBootConfiguration**

- 可以**当作配置文件来用**，定义bean，把对象注入到容器中

- 可以声明对象，对象能注入到容器中

**@EnableAutoConfiguration**

- **启用自动配置**，把java对象配置好，注入到spring容器中。（常见框架，第三方对象等等）

- 例如可以把mybatis的对象创建好，放入到容器中

**@ComponentScan**

- 组件扫描器，扫描包，找到注解，根据注解的功能**创建对象**，给属性赋值等。

- **它默认所扫描的包：该注解所在的类所在的包和子包中的所有类**。
  
  ![idea64_nQ0lQIN6Vf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_nQ0lQIN6Vf.png)

所以主类放在主包之下，其他的放在主包的子包之下。这样就可以找到程序中的各种各样的注解。

## application.properties

就是它👇：

![idea64_73hyTi1vzA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_73hyTi1vzA.png)

它是重要的配置文件。它有两种格式，另一种是.yml。

表达的内容是一样的。**现在主推.yml格式**。

它是怎么对项目进行控制和影响？

配置文件名称不能改：application

扩展名有两种：

- .properties
  
  用`key=value`方式（<mark>**冒号后面必须有空格**</mark>）

- .yml
  
  `key: value`，表达数据结构更清晰一些

继承上面那个web示例，现在我们想改一下浏览器访问的端口号，以及访问上下文路径。就是**修改了配置规则**。

![chrome_6tCP5yqo57.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_6tCP5yqo57.png)

application.properties对应修改：

```properties
#设置端口号
server.port=8081
#设置访问应用的上下文路径，也就是contextpath
server.servlet.context-path=/myboot
```

## application.yml

可以采用一种结构化的方式显示（有层级关系）

```yml
server:
  port: 8083
  servlet:
    context-path: /myboot2
    #尽量不用tab，用空格的形式
```

（.yaml）

推荐使用的格式。

如果你配置文件有两个，两种格式的，那会跑那个呢？

默认跑properties文件，我们不希望你用两个，二者选择其中一个就行了。

## 多环境配置

开发环境一般在本机上，交给测试人员测试，测试环境肯定和本机不一样，上线后的用户用的更不一样。如何方便的切换到各环境中呢？可以通过简单信息配置一下。

**创建三个或更多的配置文件**

**application（开头）-标识名称（自定义）**

格式选两个都可以。

可以方便切换不同的配置。

比如创建一个开发环境的配置文件:

**application-dev.properties**/yml

测试中使用的配置：**application-test.properties**等等。

![chrome_6gLxURzbHF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_6gLxURzbHF.png)

可以在里面指定端口号和上下文路径，配置完成后我们得告诉当前项目用哪个文件。**默认读的是application.properties。所以我们得在它中指定激活哪个配置文件。**

```properties
#激活使用按个配置文件
spring.properties.active=dev
```

## 自定义配置

还是操作配置文件，从配置文件中获取数据。

### @Value

取值，可以来自于application.properties属性配置文件（我们叫他主配置文件）来获取数据。

`@Value("${key}")`

写一个（里面的数据我都可以拿来用）

```properties
#设置端口号
server.port=8081
#设置访问应用的上下文路径，也就是contextpath
server.servlet.context-path=/myboot
#自定义key=value
school.name=博客
school.website=catoo.info
school.address=广州

site=www.baidu.com
```

写一个Controller类，通过它来读取里面的数据（和之前学的springmvc是一样的，只不过这里简化了，全用注解创建对象，实现类，不需要xml配置文件注册注解驱动等）

如果出现中文乱码问题：

### 中文乱码问题

Settings-encoding改成utf-8应用一下就OK了。（一般要先配置，因为应用后原文件中文会乱码）

![idea64_6yMdtAzWaV.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_6yMdtAzWaV.png)

### @ConfigurationProperties

将整个文件映射成一个对象，用与自定义配置项比较多。

我们在刚刚上面那样一个一个取数据值很辛苦，现在想用一个java对象，将值赋给java对象的属性。

**创建一个实体类。**

里面属性名和之前是一一对应的。生成setter和getter方法，以及toString方法。

将这个类上加上这个注解，它有一个属性：前缀，也就是开头字符），不加前缀就得同properties里左边key一模一样才行！

再在类上加注解@Component，标识创建此类的对象

就是告诉框架找school开头的对象的属性值。👇

![chrome_w28D5URFO5.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_w28D5URFO5.png)

现在我们可以取Controller类中**通过自动注入方式注入该实体类的对象**，@Resource注入对象

## 使用JSP（不推荐使用）

首先我们并**不推荐**使用JSP，我们用推荐用模板技术，用它作为视图来显示数据和应用交互。**Thymeleaf模板**，以后会讲。

1. 加入一个处理jsp的依赖。负责编译jsp文件.
   
   注意我们用的这个tomcat是springboot内嵌的，所以必须加入它的依赖才行。不是我们自己安装的哪个
   
   ```xml
   <dependency>
   <groupld>org·apache.tomcat.embed</groupld>
   <artifactld>tomcat-embed-jasper</artifactld>
   </dependency>
   ```

2. 如果需要使用servley,jsp,jstl的功能，还要引入其他依赖项。
   
   ```xml
   <dependency>
       <groupld>javax.servlet</groupld>
       <artifactld>jstl</artifactld>
   </dependency>
   <dependency>
       <groupld>javax.servelt</groupld>
       <artifactld>javax.servlet-api</artifactld>
   </dependency>
   <dependency>
       <groupld>javax.servlet.jsp</groupld>
       <artifactld>javax.servelt.jsp-api</artifactld>
       <versl>2.3.1</verslon>
   </dependency>
   ```

3. 创建一个存放jsp的目录，一般叫webapp（你需要手动去改变目录性质，让在作为web资源目录）。
   
   ![idea64_CbvY48d7RT.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_CbvY48d7RT.png)
   
   ![19LBhmOoE2.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/19LBhmOoE2.png)

4. 在Pom.xml指定jsp文件编译后的存放目录（build下）
   
   ```xml
           <!--指定jsp编译compile后的存放目录-->
           <resources>
               <resource>
                   <!--jsp原来的目录-->
                   <directory>src/main/webapp</directory>
                   <!--指定编译后的存放目录-->
                   <targetPath>META-INF/resources</targetPath>
                   <!--指定处理的目录和文件-->
                   <includes>
                       <include>**/*.*</include>
                   </includes>
               </resource>
           </resources>
   
       </build>
   ```
   
   META-INF/resources
   
   ![idea64_bSooAnUuTc.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_bSooAnUuTc.png)

5. 创建Controller，访问jsp
   
   可以使用servlet的request(HttpServletRequest)的setAttribute传值，也可以用Model传。返回是一个逻辑名称（但不知道为什么这里我**最后测试的失败了报错404，但编译文件能找到**）（注意这个Model类型应选择springboot.ui这个）（如果改了返回视图为“index.jsp"测试会变成了下载）

6. 在application.properties文件中配置视图解析器（前缀后缀那些）
   
   ```properties
   #配置视图解析器
   #/ = src/main/webapp
   spring.mvc.view.prefix=/
   spring.mvc.view.suffix=.jsp
   ```

## 使用容器

我们以前用spring和springmvc时可以通过监听器、中央调度器等去创建容器，现在springboot里怎么用呢？

我们追溯SpringApplication的静态方法run可以发现，它的接口源头是继承ApplicationContext接口的，它是一个容器，那么执行这个方法就可以获取容器。若我们想获取到容器的对象，.var接收它就OK了。

示例：创建一个service实现类，用注解@Service创建它的对象，现在我们想手工去用它

先创建。再从容器中获取你期望的某个对象。

![chrome_04FA6e3Dkg.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_04FA6e3Dkg.png)

**什么情况用？自己测试的时候**，想看看这个实现类能否用的时候

## CommandLineRunner接口

**在容器对象创建好后**，会创建该接口的方法帮助我们连接数据库等。它有个方法run，参数是字符串。

我们容器对象创建好后，它会自动执行run方法，可以完成自定义的在容器对象创建好的一些操作。

![chrome_1EZhCCYIJb.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_1EZhCCYIJb.png)

上个示例方法：能拿到容器中的某个java对象（没有写出代码）。

# 3、springboot和web组件

## 拦截器

### 回顾一下SpringMVC框架中拦截器

是springmvc中的一种对象。能拦截对Controller的请求。

框架中有系统的拦截器，还可以自定义拦截器。实现对请求预先处理。

实现自定义拦截器：

1. 创建类实现springmvc框架中的HandlerInterceptor接口（它有三个方法pre-post-afterCompletion，它们三个被设置成default，所以不用都实现，用哪个重写哪个，一般pre常用。）（**快捷键Ctrl+i重写**）

2. 在springmvc的配置文件中，声明拦截器`<mvc:interceptors..`

那么在springboot中该如何做呢？

### 在SpringBoot中如何用？

按往常定义好后需要把它加到springmvc容器中才会起作用呀！（就像之前的在配置文件中声明它一样的意思）

创建一个普通类**实现接口WebMvcConfigurger**,**加上注解@Configuration**它才能变成**配置类**

这个接口有很多和springmvc有关的功能，把我们过去对xml配置文件中的配置**移到了这个接口的抽象方法中。（都是default，用哪个实现哪个就好了）**

我们现在拦截器，所以重写Ctrl+i的方法是addInterceptors。 **添加拦截器对象（刚刚建的哪个），注入到容器中。目的是告诉容器拦截器是谁** 

```java
//创建拦截器对象
//接口类型来指它的实现类
@Override
public void addInterceptor(InterceptorRegistry registry){
    HandelerInterceptor interceptor = new LoginInterceptor();
    registry.addInterceptor(interceptor);
//拦截谁：指定uri地址
    String path[]= {"/user/login"};
    registry.addInterceptor(path);
}
```

## Servlet

其对象能够处理请求。

1. **创建Servlet类**，继承HttpServlet（创建它的对象）

2. Ctrl+O重写doGet和doPost方法

3. 再**注册**Servlet对象，**创建它的配置类**，让框架能找到它
   
   用到**ServletRegistrationBean类**
   
   同样写个普通类加上注解@Configuration让它变成配置类。
   
   写方法，返回ServletRegistrationBean类对象，第一个参数是Servlet对象，第二个参数 是url地址。（new一个，”/myservlet")，return bean;(别忘了在方法加@Bean注解，才能将创建的对象放到容器中。)
   
   他其实还有一个无参构造，通过它的方法setServlet和addUrlMappings方法手工配置~

## Filter过滤器（自定义情况）

使用步骤和上面Servlet十分相似。用到的是**FilterRegistrationBean类**来注册Filter对象。

我们可以对请求参数和应答属性做赋值处理。它是Servlet规范中的过滤器，可以处理请求。（最常用就是处理字符编码utf-8。）

1. 创建自定义过滤器类（就是实现Filter接口）（javax.servlet)

2. 创建配置类，注册Filter过滤器对象
   
   通过FilterRegistrationBean类创建Filter对象（和Servlet一样，可传一个或多个参，还可以用无参）

## 字符集过滤器

### 方式一：改注册，不用框架自带

注意哦，它是spingmvc里面的，用它处理post请求和应答乱码问题的。**CharacterEncodingFilter**，spingmvc提供的。

以前在web.xml注册过滤器，配置它的属性。（强制，必须扫描）

这回我知道了，不能每一次settings设置字符串编码方式，因为不同环境编译是不同的，你可以在contentType设置规定它，也可以用框架给我们写好的过滤器。它就叫CharacterEncodingFilter。再调用对象的方法设置编码防止和强迫遍历应答都使用这种编码（两个布尔值都为真）。

它以前虽然是springmvc的东西，但是到springboot中关于Servlet和字符集过滤器的注册可以放在一起说明了（**一个配置类！里面可以用注解@Bean创建多个对象！**）。

**别忘了配置主配置文件.properties**

`server.servlet.encoding.enabled=false`

再springboot中默认已经启用了这个过滤器，默认编码是ISO-8859-1，现在我们重定义了，所以关闭它默认的。

### 方式二：用系统自带过滤器（推荐，简单）

换个思路，能不能用它自有的，而不用再新注册一个过滤器了呢？你也可以这样写：**启用系统自己设置的字符串过滤器**

```properties
server.servlet.encoding.enabled=true
#指定使用的编码方式
server.servlet.encoding.charset=utf-8
#强制请求request和应答response都是用chartset属性地址
server.servlet.encoding.force=true
```
