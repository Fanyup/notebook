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

## 入门案例：select查询操作

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
   
   - <u>mapper</u> 是当前文件的根标签，必须的
   
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
     
     <mapper namespace="com.bjpowernode.dao.StudentDao">
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

**现在我们要做的是：修改填写sql mapper(也就是映射文件的位置)**

```xml
    <!--sql mapper(sql映射文件）的位置-->
    <mappers>
        <!--一个mapper标签指定一个文件的位置
        从【类路径】开始的路径信息
        就是代码编译(compile）后target/classes(类路径)

        -->
        <mapper resource="类路径/xxx.xml"/>
    </mappers>
```

现在我们要找**从类路径开始**的路径信息：

也就是**target/classes（类路径）/下**的**不带斜杠的相对路径**。

![chrome_oSbwpDLLZM.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_oSbwpDLLZM.png)

（稍等一下，这上面1，2，3步骤不少使，不知道是不是maven版本的问题，我换成以下方式解决了👇）

![idea64_BeZTHHNYT4.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_BeZTHHNYT4.png)

maven编译文件后会生成target目录

mvn compile（编译）

编译main/java/目录下的java 为class文件，通知把class拷贝到target目录下面、

##### 编译后没有xml的情况

**得在pom.xml中添加resources标签，才能保证我们的sql映射文件拷贝到classes下面。**

但现在通过上上图发现没有这个文件，是因为在默认情况下maven编译java目录下的这些文件是自动被忽略调的。解决这个问题需要加一个插件（plugin)（可加可不加，因为compile我们已经指定1.8了，resources插件得放到build里，它是指定文件用的。

```xml
  <build>
    <--!下面我有问题，你发现了吗？-->
    <resources>
      <!--所在的目录-->
      <directory>src/main/java</directory>
      <includes>
        <!--包括目录下的.properties,.xml文件都会扫描到-->
        <include>**/*.properties</include>
        <include>**/*.xml</include>
      </includes>
      <filtering>false</filtering>
    </resources>

  </build>
```

上面这个是网课参考，我一直爆红，终于查出问题所在：因是一个一个打的，漏掉了resources下的resource标签！这里是我不懂得maven规则犯的错

- build插件（**只需要一个**）
  
  - resources（**只需要一个**）
    
    - resource（可以有多个）

修改后的：

```xml
  <build>

      <resources>
       <!-- <resource>
          <directory>src/main/resources</directory>
          <includes>
            <include>config/*.properties</include>
            <include>*.xml</include>
          </includes>
        </resource>-->

        <resource>
          <directory>src/main/java</directory>
          <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
          </includes>
          <filtering>false</filtering>
        </resource>
      </resources>


  </build>
```

（注释掉的这段之后会用到👇）

```xml
  <build>

      <resources>
        <resource>
          <directory>src/main/resources</directory>
          <includes>
            <include>config/*.properties</include>
            <include>*.xml</include>
          </includes>
        </resource>
      </resources>

  </build>
```

再次编译：

![idea64_i2XI1TRXKk.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_i2XI1TRXKk.png)

现在就能获取到target/classes下的路径了，我们拷贝一下，将它粘贴在mapper。

<mark>终于找到他了~</mark>：修改一下

```xml
  <!--sql mapper(sql映射文件）的位置-->
    <mappers>
        <!--一个mapper标签指定一个文件的位置
        从【类路径】开始的路径信息
        就是代码编译(compile）后target/classes(类路径)

        -->
        <mapper resource="com/bjpowernode/dao/StudenDao.xml"/>
    </mappers>
```

**我们找它做什么？** 因为Student.xml(也就是sql映射文件)中有执行sql语句，而主配置文件的两个作用1.连接数据库，2.配置才能指向到sql映射文件；否则不就断连了嘛！**sql映射文件一般在同该dao接口一个目录夏的**

这个mapper标签可以出现多次，可指向多个sql映射文件嘛。

##### 特殊情况，做好一切后还是没有xml

这有可能是maven自带的小Bug，这时可以执行maven的**clean，清掉编译文件，再用compile重写编译一下**。实在不行，则最上方Build->Rebuild Project强制重置一下该项目。还不行，手工CV复制黏贴吧！

#### 7、创建mybatis类，访问数据库

创建SqlSession对象执行sql语句

```java
public class MyApp {
    public static void main(String[] args) throws IOException {
        //访问mybatis读取student数据
        //1.定义mybatis主配置文件名称
        //从类路径的根开始
        String config="mybatis.xml";
        //2.读取这个config表示的文件
        InputStream in = Resources.getResourceAsStream(config);
        //3.创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //4.创建SqlSessionFactory对象
        SqlSessionFactory factory = builder.build(in);
        //5.【重要】获取SqlSession对象，从SqlSessionFactory中获取SqlSession
        SqlSession sqlSession = factory.openSession();
        //6.【重要】指定要执行的sql语句的标识。
        //sql映射文件中的namespace + "." + 标签的id值
        String sqlId = "com.bjpowernode.dao.StudentDao" + "." + "selectStudents";
        //7.通过sqlId,找到并执行sql语句
        List<Student> studentList = sqlSession.selectList(sqlId);
        //8.输出结果(这里用的是lambda表达式）
        studentList.forEach(stu-> System.out.println(stu));
        //9.关闭SqlSession对象
        sqlSession.close();
    }
}
```

##### 需要注意的细节

![idea64_QGFFVi3EOR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_QGFFVi3EOR.png)

![idea64_AAbgN7fbIU.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_AAbgN7fbIU.png)

![idea64_m8sHTjFoSS.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_m8sHTjFoSS.png)

##### 连接失败到问题解决

普大喜奔，泪目，一共测试3回。第一次爆红失败时简直要心态崩了，毕竟一遍流程下来全照搬，每个步骤都不是理解的很透彻，但心态还是稳的：处理这个mybatis类，前面步骤好在还有笔记。下面来说以下三次主要错因在哪。

- 一周目异常爆红第一句提示数据库连接失败，我查看mybatis.xml主配置文件后发现是**连接信息写错了**，密码前漏删了个$。修改后我重建了数据库中的student表

- 显示连接上了，（没有异常就是正常，无为这点和Linux很像）但是异常提示找不到id这个，我看了下表，尼玛原来id多加了个s写成ids了，无奈谨慎又重建了个。

- 这回输出了重写方法，期间我按弹幕的指示在sql语句中加了库名（即from mydb.student)，成功后除去库名发现执行也是一样成功的。

![idea64_vhVve7TTaE.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_vhVve7TTaE.png)

##### 回顾该示例关键细节

![idea64_qcTN3UblIG.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_qcTN3UblIG.png)

主配置文件（后面很多东西我们不需要的，这里主讲关键细节）

![idea64_NBulvCokMH.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_NBulvCokMH.png)

1. 我要首先要知道你连接哪个数据库，得读主配置文件。到哪去找他你？**得从类路径根开始**
   
   因为代码**经过编译之后是放在target目录下**
   
   我们得执行target目录下的类，即classes下的目录
   
   它下面才是我们执行时要用到的文件和各种资源。
   
   因为我们把它放到了resources目录（**我们将它设置成了资源文件夹**）下，所以它经过编译后直接在mybatis根目录下。

![idea64_yZXqzdmHfG.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_yZXqzdmHfG.png)

2. 通过Resources类的静态方法得到**输入流**
   
   第一步是找到它，第二步才是读取它。
   
   1. 读到内部自动创建Connection对象，这就算连接上了。
   
   2. 找到sql映射对象了，知道sql语句

3. 4.我们把这个输入流最终给了build方法，目的是创建SqlSessionFactory对象，再调用它的openSession()方法**得到SqlSession对象，它很重要**。

4. **通过这个对象，执行selectList()方法，表示执行数据查询得到多条记录。**
   
   它的结果是一个List集合

5. 那么它怎么知道我们要执行哪个sql语句呢？
   
   我们得通过**标识**来”告诉“它。标识规定：
   
   返回类型是String字符串由：
   
   "namespace" + ”.“ + "具体sql语句的唯一id标识"组成
   
   它们之间用.号相连，注意**上面是结合成一整段字符串的**。

6. 执行输出完后，最后关闭SqlSession对象。

## 入门案例：insert插入操作

当基本结构写完后，直接添加就方便多了。

1. 接口里添加插入的抽象方法。
   
   ![chrome_sE9Sbtr7ys.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_sE9Sbtr7ys.png)

2. StudentDao.xml
   
   insert标签加插入操作。
   
   注意我们传入的参数时对象，因此不能写死，要通过这个对象来查询数据。
   
   ![idea64_CvxNiTmJmf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_CvxNiTmJmf.png)
   
   这里传入对象属性值有专门规定，我们先拿来用，以后会讲，格式是`#{属性名}`

```xml
    <insert id="insertStudent">
        insert into student values (#{id},#{name},#{email},#{age})
    </insert>
```

建议用单元测试测试方法，方便一点，不用重写了。

### 测试一下

帮刚刚写好的方法拷贝到测试类的测试方法中，这样就不用重写了。这里不再是setList()查询方法了，我们把它换成SqlSession对象的insert()方法。

![idea64_6XwoqIgkyA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_6XwoqIgkyA.png)

执行成功，**但数据库还是没有添加数据！**

为什么呢？**因为mybatis默认不是自动提交事务的**。

需要我们在之后**手工去添加事务。**

![idea64_1tL0fIVzEL.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_1tL0fIVzEL.png)

再试试，现在表中数据就被成功添加了~

![MySQLWorkbench_jC7AH9Fg43.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_jC7AH9Fg43.png)

### 程序员实际要做的

其实实际要我们手工做的很简单：

1. 接口中添加抽象方法定义

2. sql映射文件中指定sql语句

3. 执行时在执行类中替换成新的标识id（三段＋）和方法（select呀，insert呀）

现在又有个问题：

我们执行只能看到单方法返回的结果，而不知道sql语句执行的详细信息，这时该怎么办呢？需要**开启日志**

### 开启日志

在主配置文件中加上一句话即可。

它表示**打印日志到控制台**。

(一般我们加在配置文件稍前面一点)

```xml
//注意是<configuration>下：

    <!--settings:控制mybatis全局行为-->
    <settings>
        <!--设置mybatis输出日志-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

现在再看刚刚的**查询语句**结果：

![idea64_cXV1hYHUTR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_cXV1hYHUTR.png)

再来看看这次的插入语句执行结果：

![idea64_seTIRHQ25o.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_seTIRHQ25o.png)
