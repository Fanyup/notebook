# 分布式业务系统

假设原来做了一个OA系统（**办公自动化（OA），英文Office Automation**），里面包含了权限迷哦快、员工模块等，一个工程，里面包含了一堆模块，模块间会相互去调用。一台机器部署。

若把这个系统拆开，则又多个工程，分别**在多个机器上部署**。

一个请求过来，需要依次调用这几个模块，才能完成这个请求，四个系统分别完成了一部分的事情，都干完后，才认为是这个请求已经完成了。

一个请求，是4台机器上部署的4个系统一起完成的，每台机器的每个系统完成请求的一部分操作

**员工系统+请假系统+权限系统+财务系统=分布式的OA系统**

扩展方便√双十一：哪个需求量大扩展哪个，其他可以原封不动

反义：crm集中式开发系统

单以架构（扩展难）->垂直应用架构（根据业务拆分，扩展容易，但可能业务和见面没有分离开，藕断丝连）->分布式架构（好框架：阿里巴巴dubbo；烂框架：学校内部选课宕机】】系统

# Dubbo

**rpc框架的一种实现，实现方式是java**

## 基于RPC：远程过程（方法）调用

（两个都在内部系统里）A服务器调用B服务器

高性能RPC框架，解决了分布式中的调用问题。

缺点：没有统一管理调度中心（公交车总站） ，但其他框架有，可相互调用。

## 提升性能：序列化和网络通信

1. 将本地对象通过网络进行传输——序列化——实现Seriliazable接口。

方案很多:xml,json,二进制流（效率最高，因为计算机就是二进制的），**Dubbo就是采用效率最高的二进制**。

2. 网络通信：不同于HTTP（7步走，3次握手，4次挥手）
   
   **Dubbo采用Socket通信机制**，一步到位。不同的协议。并且可以建立长连接（一直连着），不用反复连接，直接传输数据。

## 别的RPC框架

- gRPC

- SpringCloud(2017横空出世)

- Thrift

- HSF...

2018年阿里将dubbo交给了apache基金会，走出国门，交给他们管理了。

## 三大核心功能

- 面向接口的远程方法调用

- 智能容错，负载均衡

- 服务自动注册和发现

`dubbo.apache.org`(rrganizations)

面向代理对象，以服务为接口（对外提供完整的业务功能，最小单位/粒度）面向接口变成，根本看不到是实现类，为开发者屏蔽远程调用底层细节。**也就是类似用mybatis，就不需要去手动实现dao接口了！**

## 结构概述

1. 当系统初始化时，要将服务提供者加载到容器(spring)里。

2. 把需要对外提供的服务注射到注册中心

3. 消费者(也是一个spring容器)去registry注册中心去订阅服务

4. 订阅后会将给地址，消费者根据地址去调用提供的服务。

5. 消费者和启动容器都会反馈情况给monitor监控中心
   
   ![chrome_khf75mfWdb.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_khf75mfWdb.png)

（基本架构图）

商家-提供菜单-消费者订饭-告知订饭成功-消费者线下取饭，平台拥有者相当于监控中心。

（监控中心是现成的不需要我们去写，去关心）

## 支持的协议

支持多种协议(消费者从注册中心拿到地址，找商家调用的协议)

- dubbo

- http

- thrift

- redis...

官方推荐使用dubbo协议，该协议默认端口20880

## 直连方式-1

**不通过（即不需要）注册中心，直接调用提供者提供的服务。**

| alt + insert +fn | 新建目录（文件夹）               |
| ---------------- | ----------------------- |
|                  | 也可以在类中创建setter,getter这些 |
| alt+enter        | 还可以快速创建实现类              |

因为dubbo与spring无缝对接，所以首先得在xml文件里配置spring-context(上下文容器),spring-mvc的依赖：

```xml
<dependencies>
    <!--spring的依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.6</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.3.6</version>
    </dependency>
  </dependencies>
```

去[maven中央仓库](https://mvnrepository.com/)找dubbo的依赖

![chrome_RtnWo5Ronf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_RtnWo5Ronf.png)

课程用了2.6.2的版本（导入爆红记得刷新一下就好了）

还需导入**JDK1.8编译插件**：(记得要放在dependencies标签的外面，project的里面!)

```xml
  <build>
    <plugins>
      <!--JDK1.8编译插件-->
      <plugin>
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

**目的：避免编译器是1.8，但编译级别却是1.5的情况。**

![idea64_fRXK6r9uED.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_fRXK6r9uED.png)

我们接下来要是写：

**根据用户标识获取用户信息**（查询用户详情）

java包目录下创建model包装实体bean

创建业务方法——查询用户详情

创建service包

![chrome_5AbACJR90O.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_5AbACJR90O.png)

这一步相当于Provider，**我们还需要Spring容器去启动**

我们对外提供的是一个面向**接口**（最小粒度），消费者会面向接口去编程，因此我们还需要提供它对应的实现方法。

![idea64_NMNkDpq22H.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_NMNkDpq22H.png)

| 插件ctrl+shift+y | 选中单词翻译            |
| -------------- | ----------------- |
|                | 按住shift即可左右拖动选中文字 |

## 核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--服务提供者必须声明名称：必须保证服务名称的唯一性-->
    <!--它的名称是dubbo内部使用的唯一标识-->
    <dubbo:application name="001-link-userservice-provider"/>
    <!--我们现在使用的是官方推荐的dubbo协议，端口号默认20880-->
    <!--name：指定协议名称
        port：指定协议端口号-->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!--暴露服务的接口（关键）-->
    <dubbo:service interface="com.wkcto.dubbo.service.UserService"
                   ref="userService" registry="N/A"/>
    <!--最终通过代理找到的其实还是它的是实现类，因此
        需要ref引用,
        即接口引用的是实现类在spring容器中的标识
        还需告诉它我们用的是直连方式，没有用到注册中心
        即为N/A
        -->

    <!--将接口的实现类加载到spring容器中👇-->
    <bean id="userService" class="com.wkcto.dubbo.service.impl.UserServiceImpl"/>


</beans>
```

- 名称

- 使用的协议端口号

- 需要暴露的接口

- 通过注册中心/直连方式？

- 接口的实现类

最后需要去启动它——**一个监听器**！（在web.xml中配置）

全限定类名忘了怎么办？去哪找？不要着急！**两次shift搜索它！**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

  <!--加载配置文件-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:dubbo-userservice-provider.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```

要启动的话，别忘了给它配个tomcat哦！

## 总结流程（添加服务的提供者）

1. 创建一个maven web工程：服务的提供者

2. 创建一个实体bean查询的结果

3. 提供一个服务接口：xxxx

4. 实现这个服务接口：xxxxImpl

5. 配置dubbo服务提供者的**核心配置文件**
   
   1. 声明dubbo服务提供者的名称name：保证唯一
   
   2. 声明dubbo使用的协议和端口号
   
   3. 暴露服务，使用直连方式

6. 添加监听器(web.xml)

现在我们还需要一个服务的消费者——创建一个新的web工程

## 总结流程（添加服务的消费者）

1. 创建一个maven web工程：服务的消费者

2. 配置pom文件：添加需要的依赖（spring,dubbo)

3. 设置dubbo的核心配置文件(引用哪些服务，服务的名称)

4. 编写控制层controller让别人可以去调用

5. 配置中央调度器（就是一个servlet：DispatcherServlet）

我们至少得知道业务接口提供了哪些方法，如何能知道？

需要提供者将它打成jar包，发布到本地仓库。

因为是java工程，一个web工程，所以我们需要先将packaging注释掉（一i那我它默认打成一个jar包）

**双击Ctrl，弹出RunAnything**或者**点击IDEA右面maven**，再点击弹出窗口的M标志

![idea64_VHrQmTCVts.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VHrQmTCVts.png)

## 报错：执行失败

报错说找不到对应的maven xml文件，于是我去settings重新设置了一下，这次改成了3.8.3新安装的。

![idea64_JuMRjz2R1Q.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_JuMRjz2R1Q.png)

之后爆红说我的project标签有问题，我复制了一份备份过来就OK了。我想原因应时删除掉了中间重要标识子标签的缘故。

![idea64_aBJO2nITbj.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_aBJO2nITbj.png)

我们install之后就将它打成了一个jar包，发布到本地仓库。

![idea64_5l8YyYOii4.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_5l8YyYOii4.png)

## 报错：找不到dubbo-01无法安装

![idea64_ynVgi3p4mF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_ynVgi3p4mF.png)

解决方案：服了老铁，直接点这个，别自己输👇

![idea64_d4Ivdf9aB6.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_d4Ivdf9aB6.png)

打包成功后，再将注释消掉，还原它为war工程。而由消费者依赖服务提供者。

### 报错：For artifact {com.wkcto.dubbo::null:jar}: The version cannot be empty

刷新一下maven，clean，再编译。就这几招。然后就OK了。

言归正传，我将本地仓库设置在了E盘的repository文件夹中。

![idea64_5k3BvEJIkN.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_5k3BvEJIkN.png)

经过多次尝试后，我总是无法在该文件夹的com目录下找到wkcto，经过多次install后（我试过如果只有一个maven项目的话是可以不写项目名的，同样能执行。)仔细检查后发现，原来不仅是在项目名，还包括我先前创建目录时带的前缀（以后创建前一定要看清楚😓）

![idea64_5k3BvEJIkN.png](C:\Users\up\AppData\Roaming\marktext\images\688688ac4ecc76e598ced427193a8d83d5cd52dc.png)

消费者依赖服务者

```xml
 <!--依赖服务提供者-->
    <dependency>
      <groupId>fy.dubbo</groupId>
      <artifactId>dubbo-01</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
```

有了依赖，接下来我们要去配它的核心配置文件

## 消费者的核心配置文件

dubbo-userservice-consumer.xml

```xml
<!--声明服务消费者的名称：保证唯一性-->
    <dubbo:application name="002-link-consumer"/>
    <!--
        引用远程服务接口：
        id：远程服务接口对象的名称
        interface:调用远程接口的全限定类名
        url:访问服务接口的地址
        registry:不使用注册中心，值为N/A
    -->
    <dubbo:reference id="userService" interface="com.wkcto.dubbo.service.UserService"
                     url="dubbo://localhost:20880" registry="N/A"/>
```

（注意是在整个大的bean标签里哦，因为我们创建的是一个spring容器配置文件）

**(这些都是在resources资源目录下创建的)**

## 消费者的中央调度器

因为我们还需去扫描，所以需要一个spring的配置文件

我们要做的是要它扫描我们web控制层下的东西（等会会完善）

![idea64_BxF7iFefAZ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_BxF7iFefAZ.png)

小技巧，当你不记得那么长一串英文时，只需双击Shift，搜索关键字即可找到。**从而获取它的全限定类名**

```xml
<!--扫描组件-->
    <context:component-scan base-package="com.wkcto.dubbo.web"/>

    <!--配置注解驱动-->
    <mvc:annotation-driven/>

    <!--如果用到页面就会写一个视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

大功告成后，我们开始编写web（目录）控制层

![idea64_VLYDf0sqxr.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VLYDf0sqxr.png)

此时还无法查询return返回的结果，是因为我们还未配置web.xml里的中央调度器。

中央调度器实质上就是一个servlet。

web.xml👇

```xml
<servlet>
   <servlet-name>dispatcherServlet</servlet-name>
   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   <!--初始化参数-->
   <init-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:application.xml,classpath:dubbo-userservice-consumer.xml</param-value>
   </init-param>
 </servlet>

  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

最后，给它一个页面userDetail.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>用户详情</title>
</head>
<body>
 <h1>用户详情</h1>
<div>用户标识：${user.id}</div>
<div>用户名称：${user.username}</div>
</body>
</html>
```

## 已解决小问题

把jsp页面不小心仿佛WEB-INF下了，这时是无论如何也没法在Controller控制器下（扫描）找到它的。应与WEB-INF同级，同在web目录下就OK了。

## jsp和JS的区别

*jsp*不是*javascript*。***jsp*全称是java server page 是JAVA企业应用的一种动态技术**用于java语言的web开发方向。**JS全称是*javaScript* 是一种页面脚本语言。**

**JS是在客户端执行的，需要浏览器支持Javascript。** **JSP是在服务器端执行的，需要服务器上部署支持Servlet的服务器程序**。

---

接下来就可以进入运行阶段了。消费者部署tomcat。

先启动服务提供者的猫√再启动消费者的猫√

运行(消费者)localhost:8080/user?=2001报错(网课老师写错了)

![chrome_xaT42vqicG.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_xaT42vqicG.png)

检查一下👇

![idea64_FUrIkP0qc9.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_FUrIkP0qc9.png)

（这里少标一行:**xmlns=mvc那一行也是要去掉的**）

也就是说我们下面的注解驱动要重新导👇

![idea64_gqIjz5cml5.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_gqIjz5cml5.png)

![idea64_J4RZAMgdD7.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_J4RZAMgdD7.png)

## 学会改BUG

![idea64_xnjmo19Zhs.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_xnjmo19Zhs.png)

老铁们，有时候问题会有几个，而不是单独的。走每一步前最好保持清醒确认一下。

首先摆正心态。这里我出现了几个错误：

首先原视频教学老师犯了一个错：实体类没有实体序列化接口（在开篇的**提升性能**中有讲到），但我先前出现的报错与网课老师不一致，是因为我启动的两个猫都是项目2（也就是消费者的），再我将它们全部关闭后重新依次启动「服务提供者」与「消费者」后，报错就与网课一致了，也就是说进度赶上了。排除教学意料外的错误。

这也解释了为什么我上面报的错是请求超时而不是👇这个了：

![idea64_qXipCbXIdn.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_qXipCbXIdn.png)

**实体对象在网络上传输必须序列化**

![idea64_qGqpAc5ynW.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_qGqpAc5ynW.png)

重新依次启动，成功了👇

![chrome_WDvbMnFAnC.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_WDvbMnFAnC.png)

(一般图床图片上传Github仓库失败，可以预先登录Github网站试试)

官方推荐必须有一个接口工程，它就是一个Maven javaa工程

要求接口工程里存放的内容如下：

1. 对外暴露的服务接口(service接口)

2. 实体Bean对象

所以我们不能直接调用，而是通过RPC远程调用过程这个方法协议，这里就是dubbo所推荐的自己的协议。
