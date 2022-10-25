# Mybatis框架Dao代理

## 主要类的介绍

![chrome_VbTDPVUhBy.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_VbTDPVUhBy.png)

下面两个重点讲

### SqlSessionFactory对象

- 重量级对象
  
  程序创建这个对象耗时较长、使用资源较多
  
  在整个项目中，有一个就够用了。
  
  **SqlSessionFactory是一个接口**，提供的**openSession()方法来获取SqlSession对象**。
  
  光标放他上面：Ctrl+H看它的是实现类

- 作用
  
  获取SqlSession对象

- openSession()方法
  
  1. **默认是无参数的，获取的是非自动提交事务的SqlSession对象**
  
  2. 带参boolean，为true获取自动提交事务SqlSession对象

### SqlSession对象

- SqlSession是接口
  
  定义了操作数据库的方法。(增删改查，事务提交，事务回滚)

- 光标放他上面：Ctrl+H看它的是实现类

- 这个对象不是线程安全的，**需要在方法内部使用**，
  
  在执行sql语句之前，使用openSession()获取该对象，执行完sql语句后，sqlSession.close()关闭它。
  
  **这样能保证它的使用是线程安全的！**

至此我们发现很多工作是重复的，下面我们来写个工具类封装这些重复的工作吧！！

### 写一个工具类封装它们

```java
public class MyBatisUtils {

    //局部变量我们用不了，得把它声明在外面让他变成全局（成员）（类）变量。
    private static SqlSessionFactory factory = null;
    //静态块，在加载这个类时只执行一次
    static {
        String config="mybatis.xml";
        try {
            InputStream in = Resources.getResourceAsStream(config);
            //创建SSF对象，使用SSFBuild
            factory = new SqlSessionFactoryBuilder().build(in);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //获取SqlSession的方法
    public static SqlSession getSqlSession(){
        SqlSession sqlSession = null;
        if (factory != null) {
            sqlSession = factory.openSession();//非自动提交事务，它得到的是一个对象
        }
        return sqlSession;
    }
}
```

MyApp类中删除1-4，并修改：

```java
//5.【重要】获取SqlSession对象，从SqlSessionFactory中获取SqlSession
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
```

写到目前为止，你有没有发现StudentDao接口似乎没什么用，和它没关系。那为什么还有这个接口呢？

### 引入：传统dao与简化升级

以前我们传统dao方法应该是实现类实现接口方法，这样才是正常的。但这样做规律：方法代码内部重复高。有规律可循。

如何简化流程？

### mybatis的动态代理

mybatis根据dao的方法调用，执行获取sql语句的信息。

mybatis根据你的dao接口，创建出一个dao接口的实现类，并创建这个类的对象，完成sqlSession调用方法，访问数据库。

简而言之，**你的StudentDaoImpl实现类不用再创建了！mybatis根据dao接口在内部帮你把这个实现类实现出来~**

并且完成dao实现类中这些方法的调用。

**这是我们以后主要用到的方式。**

那么该如何用动态代理这个机制呢？——使用`SqlSession.getMapper(dao接口)`这个方法

#### getMapper方法

getMapper能获取**dao接口对应的实现类对象**。

测试类方法修改：

```java
 @Test
    public void testSelectStudents(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        StudentDao dao = sqlSession.getMapper(StudentDao.class);
        //调用dao的方法，来执行数据库的操作
        List<Student> students = dao.selectStudents();
        for (Student stu: students){
            System.out.println("学生="+stu);
        }
```

![idea64_M4jUkrWSIc.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_M4jUkrWSIc.png)

这样相较于第一个入门案例更简化了。不仅用到了工具类封装创建对象方法，同时使用动态代理创建了dao接口的实现类，程序员只需要认真写dao接口和其相应的sql映射文件即可。和第一个案例的区别是，案例一中没有用到dao实现类，而这个案例中我们调用了该实现类的方法。

**通过getMapper方式获取到动态代理的dao对象（可以理解为实现类对象）**

其中**dao实现类的方法正是sql映射文件中那个对应sql语句的唯一id值。**

查询：

![idea64_AF1XhmjexA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_AF1XhmjexA.png)

![idea64_2us6xGe9rq.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_2us6xGe9rq.png)

插入：

（入门案例一）

![idea64_eIUWgqnEJ8.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_eIUWgqnEJ8.png)

（动态代理dao接口内部创建实现类）

![chrome_hKfvWmt30j.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_hKfvWmt30j.png)

因为有接口，所以它是我们之前讲过的**JDK动态代理**

## 深入理解参数

现在我们研究的问题是，**如何把java代码中填入的参数传到sql映射文件的mapper标签下的sql语句中**，mybatis可以怎样使用这些数据？（就是如何传参的问题）

![idea64_aL4XN92JdF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_aL4XN92JdF.png)

### 属性parameterType

写在mapper文件中的一个**属性**，表示dao接口中方法的参数的数据类型。

示例实现一下：

dao接口定义抽象方法：

```java
//接口，操作student表
public interface StudentDao {
    public Student selectStudentById(Integer id);
}
```

现在我想告诉mybatis这个id是一个整型Integer类型的，这样它就知道更多信息了。于是可以**在select标签上**加上这个属性

sql映射文件中：

```xml
<mapper namespace="com.bjpowernode.dao.StudentDao">

    <select id="selectStudentById" parameterType="java.lang.Integer " resultType="com.bjpowernode.domain.Student">

        select id,name,email,age from student where id=#{id}
    </select>
</mapper>
```

它是表示**dao接口中抽象方法参数的数据类型**。

它的值是java的数据类型全限定名称，或mybatis的别名

如java.lang.Interger和Int(这是mybatis起的，短小好记)

**不是强制的**，有也行，没有也行。

mybatis通过反射机制能够发现接口参数的数据类型，所以可以没有。一般我们也不写。

#### 测试中出现爆红问题

按照网课示例查询id号为1002的同学时发生了爆红，让我们来读一下这段异常：

先来看一下测试方法：

![idea64_bE2VaL6vl7.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_bE2VaL6vl7.png)

。。。现在测试又没了，一开始报错问题说是连接失败。我检查了StudentDao.xml后依旧没有解决。然后打开了数据库查看一下有没有1002并刷新了maven，结果是没有的。但现在用1002查询发现也可以了，返回的是null。所以我怀疑是数据库没有启动导致的？

### 传一个简单的参数（掌握）

一个简单类型的参数：

1. mybatis把Java的基本数据类型和String都叫简单类型。（其中也包含了包装类如Integer一类）

2. 在mapper标签文件中获取简单类型的一个参数的值，使用`#{任意字符}`。占位符。

刚刚上面查询示例就是一个例子。

![chrome_c1DzGlVlUq.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_c1DzGlVlUq.png)

在底层中Mybatis是用jdbc封装的。它在这一步中使用的是jdbc中的PreparedStatement对象，底层中没遇到一个占位符`#{随便填}`都会把它们变成通配符`?`，执行sql语句后将结果集封装为Student对象。

注意：Student对象得有无参构造。（mybatis默认启动）

### 传多个参数：

有以下四种方式：

#### 使用@Param（掌握）

使用这个注解来命名参数，也就是给参数起名。

**放在形参的前面使用的**。这点和springmvc中的请求指定方法注解很像，都是放在方法上的@RequestMapping(value="指定uri“（使用）method="（post或get方法等）")

![chrome_zbqRXToClx.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_zbqRXToClx.png)

此时自定义的传参名是用在mapper文件标签中的，所以我们不能像之前一样随便写`#{}`里面的参数名了。

#### 使用对象（掌握）

**传入的java对象类型很灵活，你甚至可以自定义。**

**本示例中使用的对象的类型就是我们自定义的类。**

用一个java对象，它里有多个属性，这些属性值就是我们的参数值。

简述一下大概流程。就是我们要创建一个新的Java对象，在测试类方法作为传入参数的对象，这个对象里的属性值可以通过对象的setter方法注入，注入后再通过sql映射文件传入sql语句中。

注意：本方法和上面使用注解传入的方法都不用在sql映射文件中声明任何和定义本身有关的东西，就是说关于对象声明的任何定义都不用写，**只需要修改的地方就是它了**：`#{属性名}`，**该sql语句返回的resultType依旧是Student对象属性类型的**。因此我们可以在传入多个参数时用List列表集合类返回。

新java对象（传参）定义：

```java
public class QueryParam {
    private String paramName;
    private Integer paramAge;

    //下面还有setter和getter方法，这里我省略没写
    //...
}
```

sql映射文件：

```xml
    <!--多个属性，使用Java对象的属性值，作为参数实际值-->
    <!--使用对象语法： #{属性名，javaTyle=类型名称，jdbcType=数据库中的数据类型}，很少用-->
    <!--javaType:指java中的属性类型-->
    <!--例如#{paramName,javaType=java.lang.String,javaType=VARCHAR}-->

    <!--简化方式：#{属性名}，其他mybatis反射能够获取-->
    <select id="selectMultiObject" resultType="com.bjpowernode.domain.Student">
        select id,name,email,age from mydb.student
        where name=#{paramName}
        or age=#{paramAge}
    </select>
```

别忘了在接口写对应抽象方法。该方法的形参类型就是我们已经注入参数的类型对象。

```java
//接口，操作student表
public interface StudentDao {
    public Student selectStudentById(Integer id);

    //多个参数，使用java对象作为接口抽象方法的参数
    List<Student> selectMultiObject(QueryParam param);
}
```

测试方法：

```java
  @Test
    public void testSelectMultiObject(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        StudentDao dao = sqlSession.getMapper(StudentDao.class);
        //调用dao的方法，来执行数据库的操作

        QueryParam param = new QueryParam();
        param.setParamAge(20);
        param.setParamName("张飞");
        List<Student> student = dao.selectMultiObject(param);
        System.out.println("学生="+student);
    }
```

通过测试方法可以查到名字叫“张飞”或年龄是”20“的同学了（sql映射文件select标签新定义的）。

##### 爆红数据库连接失败原因

和昨天问题一样，这回终于确信了，本地mysql未启动。

把workbench启动一下就连成功了。

#### 按位置(了解)

mybatis版本3.3之前使用#{0}，#{1}方式，3.4开始以后用#{arg0}，#{arg1}的方式了。

#### 使用Map(了解)

通过Map集合类的key作为参数名传值。

`Map<String,Object> map`

![chrome_0esz7f8htU.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_0esz7f8htU.png)

![ApplicationFrameHost_vtUAhrE9Vv.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_vtUAhrE9Vv.png)

牵一发而动全身，不知道有几个参数，类型是什么，可读性很差。**不建议用**。

### 占位符#和$的区别（掌握）

#{}：占位符，告诉mybatis使用实际的参数值代替。并使用PreparedStatement对象执行sql语句（它实际上做了比sql语句中通配符“？”多做了一件事——预处理）同时也做到了避免sql注入。

**能用#的地方，也可以用$替换**，它不适用展位符，是**字符串的连接**。

因此可以看出它使用的是Statement对象执行sql语句，效率比PreparedStatement低，还可能出现sql注入的严重问题！破坏我们的数据库。（Jdbc里有讲）

**但它也不是一无是处，它可以替换表名或列名**，你能确定数据是安全的是，可以用它。比如说我想按name排，又想也用age排。（用到的比较少，90％情况都不用它）

## 封装Mybatis输出结果

现在就是来进一步解析我们之前的操作，即resultType属性是如何神奇转换将sql语句执行值，转如成我们定义的java对象呢？

### 1、resultType

#### 返回简单类型

![chrome_QpVbI9p0fC.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_QpVbI9p0fC.png)

**可以用别名，也可以用全限定名称**。

一行一列，说白了就一个值。

#### 返回对象类型

如之前我们示例演示：sql语句**执行后的列的数据**结果值以Student对象的形式存入（或输出）。

它指sql语句执行完毕后，数据转为的java对象。相当于是mybatis帮我们做了创建对象并将值赋给它的了而已。我们现在就可以直接用这个对象了。

**<mark>规则是同名的列赋给对象同名的属性。</mark>**

你可以把它看成接口声明该抽象方法的返回值。

resultType里面是一个任意的java类型，我们可以像Student对象类型一样自定义一个~

##### 定义自定义类型的别名

方式一：

1. 在mybatis**主配置文件中定义**，使用`<typeAliases>`标签定义

2. 里面`<typeAlias>`下定义，就可以使用了。
   
   里面其中两个属性：
   
   type:自定义类型的全限定名称
   
   alias:别名，自定义，短小方便~

一个标签一次只能给一个类型自定义别名

方式二：**<mark>（比较常用）</mark>**

用标签`<package>`；

属性name是包名，这个包中的所有类，**这样类名就是别名**（类名不区分大小写）

**简单来说就是指定包名，那么这个包下里面的类的类名就是自定义类型的别名了。**

虽然它很方便，但我们还是建议使用全限定名称。因为它更安全。

举个例子，不同包你都指定了，但它们下面都有个叫Student的类名，这是再执行就会报错，因为它有歧义性。

#### 返回Map类型

返回Map

少用，建议接口抽象方法Map泛型键值都用Obeject。

最多只能返回一行记录，多余一行会报错的。所以它用的比较少。

### 2、实体类属性名和列名不同的处理方式：

### resultMap

**定义结果的映射关系**。

指定列名和java对象的属性对应关系。

1. 你自定义列值赋值给哪个属性。
   
   因为我们刚刚说了，规定默认是相同的。

2. **当你的列名和属性名不一样时，一定使用resultMap标签。**

这个标签是放在select标签外面的。

它有的属性：

属性id：自定义名称，表示你定义的这个resultMap

属性type:java类型的全限定名称。

![chrome_pqOzkBjXbh.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_pqOzkBjXbh.png)

灵活，可复用。

**注意：resultType和resultMap<mark>不能同时使用</mark>，二选一！！！**

### 方法二

利用数据库sql语句，将列的别名定义(as)为java对象的属性名。

![chrome_7BUcP30U16.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_7BUcP30U16.png)

（推荐使用方式一）

## 模糊查询like

和上面方法二一样，本质上是把mysql中sql语句应用到mybatis中。

### 第一种方式

在java的代码中指定Like的内容。

就是说把Like模糊查询的内容给准备好，在sql映射文件的占位符#{准备好的内容}。

那么准备的内容在哪写呢？

你可以写在测试方法中。把它当作参数（就是用到了**传一个简单参数**的方法一样，将它传进去就OK了）

其实和之前讲的换汤不换药。我们推荐使用这个方式。

### 第二种方式

在sql映射文件中拼接like的内容。

![chrome_MkvqOCROCy.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_MkvqOCROCy.png)

注意第二种方式，**空格必须有**。

方式一中传入的#{name}是 **“%李%”**

方式二中传入的是 **“李”**

**推荐方式一**。
