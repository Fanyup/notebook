# Mybatis框架

E:\mybatis-3.5.1

[MyBatis参考文档](https://mybatis.net.cn/getting-started.html)

它是操作数据库的，相当于是一个增强的JDBC。

## 框架概述

### 软件开发常用结构

**<mark>三层架构</mark>**

- **界面层**
  
  跟用户直接打交道的，接收用户请求参数，显示处理结果：html,jsp,servlet
  
  - 对应包controller包（servlet)
  
  - 对应处理框架springmvc（代替servlet)

- **业务逻辑层**
  
  接受了界面层传递的数据，计算逻辑，调用数据库，获取数据
  
  service层
  
  - service包（XXXservice类）
  
  - 对应框架：spring框架（service类）

- **数据访问层（持久层）**
  
  访问数据库，执行数据增删改查等
  
  dao层
  
  - dao包（XXXDao类）
  
  - 对应框架：MyBatis框架（dao类）

mybatis它很小，很简单。

### 什么是框架

“模板”，整个或部分系统的可重用设计。

1. 提供基础功能，这些功能是可用的。

2. 可以加入项目中自己的功能，可以利用框架中的基础功能创造。

框架特点：

1. 一般不是全能的，不能做所有事情

2. 针对某一领域有校。特长在某一个方面。

3. 是一个软件。（半成品，定义好了基础功能，它们可重复使用，可升级）

### JDBC缺陷

大量代码重复，开发效率低，开发周期长！！业务代码和数据操作混合在一起，代码维护难度高。

以前我们用过DButil这样一些工具类，但是不能完全解决一些缺陷，所以出现了mybatis框架。

### Mybatis能做什么

SQL Mapper Framework(**sql映射框架**)

早期叫ibatis上，发布于github上

tar.gz是在Linux上压缩文件的格式。

**网课用到3.5.1版本。**

1. sql mapper: **sql映射**
   
   可以把数据库表中的一行数据，映射为一个java对象
   
   **一行数据可以看作是一个java对象**
   
   操作这个对象相当于是操作表中的数据

2. Data Abceess Objects
   
   (DAOs)
   
   数据访问，对数据库执行增删改查。

提供了哪些功能：

1. 提供了创建Connection,Statement,ResultSet的的能力。不用开发人员再创建这些对象了。

2. 提供了执行sql语句的能力，不用你执行sql了

3. 提供了循环sql，把sql的结果**转为java对象，List集合**的能力。(不需我们再循环记录集)

4. 提供了关闭资源的能力~

我们只要做：**提供sql语句**~

## 入门案例

创建新空项目，创建新model，选择Maven（勾选骨架模板）（archetype-quickstart)

目前示例只是访问数据库，而不需要web应用操作。

注意这里，我的Maven路径已经修改了！是3.6.3版本。

**main目录下**创建一个resources目录，并右键

![idea64_nNrpdZmLmx.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_nNrpdZmLmx.png)

修整一下pom.xml里的东西：

![idea64_VHsf0WImno.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VHsf0WImno.png)

打包默认是Jar(packaging标签，不加也是jar)

![idea64_rkXpjmOaKJ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_rkXpjmOaKJ.png)

build里内容先全删掉，我们暂时都不需要。

### 实现步骤：

1. 新建的数据库表
   
   （这里我建在mydb库下了）

2. 加入maven的mybatis坐标，mysql驱动的坐标

3. 创建实体类，Student，保存表中的一行数据

4. 创建持久层的dao接口，定义操作数据库的方法

5. 创建一个mybatis使用的配置文件
   
   叫做**sql映射文件**：写sql语句的。**<mark>一般一个表一个sql映射文件</mark>**。
   
   这个文件是xml文件。

6. 创建**mybatis的主配置文件**：
   
   一个项目就一个主配置文件。
   
   它**提供了数据库的连接信息**和sql映射文件的位置信息。

7. 创建使用mybatis类
   
   通过mybatis访问数据库。

#### 2、加入mybatis依赖和mysql驱动

驱动也叫依赖(不同说法)

```xml
    <!--mybatis依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.1</version>
    </dependency>
    <!--mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
    </dependency>
```

记得刷新一下maven，要连网才能下载的。

#### 3、创建Student实体类

每个属性都设置它们的getter和setter方法，重写toString()方法

```java
//定义属性，目前要求是属性名和列名一样
    private Integer id;
    private String name;
    private String email;
    private Integer age;
```

#### 4、创建dao接口

表中每一个数据都可以看作是Student对象，所以我们要获取student表中所有的数据，则应该返回一个泛型是Student对象的List集合。

```java
//接口，操作student表
public interface StudentDao {

    //查询student表的所有数据
    public List<Student> selectStudents();
}
```

这个声明的抽象方法是对应的一个SQL执行。**这个SQL要写在叫做sql映射的xml文件里面。** 我们把它放到和这个接口同一目录之中。

#### 5、创建sql映射文件（xml格式）

**文件名称和接口保持一致。**

xml文件格式在mybatis参考文档里找[MyBatis中文网](https://mybatis.net.cn/getting-started.html)

也就是目录【入门】下的这个东东：

![chrome_AgBy7TCZOF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_AgBy7TCZOF.png)

 **<mark>sql映射文件：写sql语句的，mybatis会执行这些sql</mark>**

1. 指定约束文件
   
   ```xml
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   ```
   
   mybatis-3-mapper.dtd是约束文件的名称，扩展名是dtd的。
   
   【格式是固定的，不需要去记忆】  
   2.约束文件的作用：  
   它是一个文件，限制和检查在当前文件中出现的标签，属性必须符合mybatis的要求  
   3.
   
   - <u>mapper</u> 是当前文件的跟标签，必须的
   
   - <u>namespace</u>:命名空间，唯一值，可以是自定义的字符串。  
     **<mark>但我们规定使用dao接口的全限定名称</mark>**。（就是那个包括包名的一长串）

所以这里我们要将它复制过来,xml文件里修改一下：

![explorer_dIefS8eBoQ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/explorer_dIefS8eBoQ.png)

![idea64_YfPAeYEeM4.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_YfPAeYEeM4.png)

下面的select是表示查询的

4. 在当前文件中，可以使用特定的标签来表示数据库的特定操作:
   
   - **select标签**：查询操作
     
     **属性id**：你要执行的sql语法的唯一标识。
     
     mybatis会使用这个id的值来找到要执行的sql语句。
     
     可以自定义，但是我们自己要求**使用接口中的方法名。**（我们以后做项目也是这么用的）
     
     **属性resultType**：表示结果类型的
     
     是sql语句执行后得到的ResultSet得到的java对象的类型。
     
     **值写的是类型的全限定名称**。
     
     ![chrome_n9WsdutG60.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_n9WsdutG60.png)
     
     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="https://i.imgur.com/t0SSkHw.png">
         <select id="selectStudents" resultType="com.bjpowernode.domain.Student">
             select id,name,email,age from student order by id
         </select>
     </mapper>
     ```
   
   - update更新数据库的操作
   
   - insert插入
   
   - delete

#### 6、主配置文件(连接数据库)

[MyBatis中文网](https://mybatis.net.cn/getting-started.html)

还是在刚刚的doc文档中找，同一不用背，把它复制过来就OK。

把它放到哪呢？

**在resource根目录之下**

我们叫它mybatis.xml，内容不用去背，后期会去掉很多。

它是mybatis的主配置文件，**主要定义了数据库的配置信息，sql映射文件的位置**。

1. 约束文件（还是刚刚那个固定值）

2. configuration:根标签，表示各种的配置环境

```xml
<configuration>

    <!--环境配置：数据库的连接信息-->
    <environments default="development">
        <!--一environment:一个数据库信息的配置，环境-->
        <!--id：唯一一个值，自定义，表示环境的名称-->
        <!--注意，这里我们修改它叫mydev-->
        <environment id="mydev">
            <!--mybatis的事务类型-->
            <!--type的值有两个：JDBC（使用JDBC中的
            Connection对象的commit,rollback做事务处理-->
            <!--不用记，以后这下面都咔嚓没有了，用不上了-->
            <transactionManager type="JDBC"/>
            <!--
                👇表示数据源，连接数据库的
                type:表示数据源的类型，
                其中POOLED用来表示连接池的（内容都是固定的）
            -->
            <dataSource type="POOLED">
                <!--url名称这些值是固定的，不能自定i有随意改-->
                <!--数据库的驱动类名-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
</configuration>
```

数据源相关配置可以参见jdbcTemplate里的数据源配置（8.0可用）

url这里为了方便我直接拿过来了:`jdbc:mysql://localhost:3306/mydb?characterEncoding=utf8&useUnicode=true&useSSL=false&autoReconnect=true`

注意：

![chrome_KE7IZ446mu.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_KE7IZ446mu.png)

（实际上就是变相切换数据库了）

![chrome_oSbwpDLLZM.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_oSbwpDLLZM.png)

（稍等一下，这上面1，2，3步骤不少使，不知道是不是maven版本的问题，我换成以下方式解决了👇）

![idea64_BeZTHHNYT4.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_BeZTHHNYT4.png)



maven编译文件后会生成target目录

mvn compile（编译）

编译main/java/目录下的java 为class文件，通知把class拷贝到target目录下面
