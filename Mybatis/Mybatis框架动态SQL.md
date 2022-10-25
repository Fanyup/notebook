# Mybatis框架动态SQL

SQL语句的内容是变化的，叫它动态SQL。

可以根据条件获取到不同的sql语句。

**主要是where部分发生变化。**

它的实现，使用的是mybatis标签，**使用java对象作为参数**。

**if,where和foreach标签**最常用。

## if标签

- if标签
  
  判断条件
  
  `<if test="判断java对象的属性值">`部分sql语句`</if>`

sql映射文件:(也就是mapper文件)

```xml
<mapper namespace="com.bjpowernode.dao.StudentDao">
    <!--if
        <if:test="使用参数java对象的属性值作为条件"
        语法是 属性名=属性值
    -->
    <select id="selectStudentIf" resultType="com.bjpowernode.domain.Student">
        select id,name,age,email from student
        where
        <if test="name!=null and name!=''">
            name =#{name}
        </if>
        <if test="age >0">
            and age >#{age}
        </if>
    </select>

</mapper>
```

测试方法（注意，**我重新在接口写了一个抽象方法，传入参数是Student类型的对象，这一步省去了，期间对象属性名是不显山不露水**）（返回参数类和抽象方法的返回值类型很接近，但对于类型是对象时不完全相同：因为有集合）

```java
    @Test
    public void testSelectIF(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        StudentDao dao = sqlSession.getMapper(StudentDao.class);

        //调用dao的方法，来执行数据库的操作
        Student student = new Student();
        student.setName("张飞");
        student.setAge(20);
        List<Student> students = dao.selectStudentIf(student);
        for (Student stu:students){
            System.out.println("if==="+stu);
        }
    }
```

这里我是测试（如果name为null且字符串不为空时）找一个叫张飞的，且（年龄大于0时）年龄大于20的。

这里我又有一个报错了，它说我问题关键词是：\u2018。查了一下中文编码问题，于是检查了if标签中的sql语句，发现是name不等于（！=）空字符串后面的引号用成了中文的引号。修改后可以正常执行了。

（再看一眼我写的数据库）

![MySQLWorkbench_I6iQCFu48v.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_I6iQCFu48v.png)

**if标签单独使用的缺陷是**，当从上判断下的条件不满足时，会存在语句缺失，导致语法错误而整条sql语句都不能执行了。

**解决方法是**：在where后加一个恒成立的条件，如1=1或者id>0；

也可以通过**where标签**解决它的这个问题。

## where标签（与if一块使用很方便）

它是用来包含多个if标签的，**当多个if有一个成立的，where标签会自动增加一个where关键字，并去掉if中多余的and，or等**。

这时我们就不再sql写where了，而是写一个where标签，里面包含if标签~

## foreach标签

用来循环java中的数组、list集合的，**主要执行在sql的in语句中**。

示例：

![chrome_LaHsif3mix.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_LaHsif3mix.png)

能得到in里面这样的值。

手工模拟写：

![chrome_ijwb7FCOyx.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_ijwb7FCOyx.png)

很麻烦对吧？用foreach标签就已经帮我们写好了！

先介绍一下它的**语法规则**：

![chrome_ExtPUhq3Rq.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_ExtPUhq3Rq.png)

item：遍历的变量成员。

**夹心的里面是循环体**。

如果集合中是对象，如何遍历对象的属性值呢？

可以使用**对象.属性值**的方式取到。（这其实也是通用的，回想一下之前学spring中bean管理xml方式通过set方法传入对象类型的值时，是不是也是这样的？但是这里有一个前提条件是该对象类中需要有getter方法，否则是找不到的。（虽然说这后来我们用注解直接省去了这些麻烦的步骤）（以上都是我猜的）（没有具体实现）

![chrome_835KEKsYC4.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_835KEKsYC4.png)

截得有点丑，凑合着看吧~

![chrome_nuyN0u6sh8.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_nuyN0u6sh8.png)

## 代码片段

是复用代码的一种快捷方式。同样也是在mapper文件中执行的。

语法规则是用**sql标签**指定一个id名称，里面是我们想要重用的内容（sql语句，表名，字段等）

![chrome_XTNj0s4cnK.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_XTNj0s4cnK.png)

`<sql id="id">要复用的内容sql语句</sql>`

定义完之后，在我想用到该片段的地方用**include标签**将它引用进来。

![J1g2pOjGdx.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/J1g2pOjGdx.png)

`<include refid="id">/`

代码片段可定义多个。加多个标签就OK了~注意先定义再使用。

## Mybatis配置文件

### dataSource标签（了解）

这里主要来讲一下主配置文件中的dataSource标签。

表示数据源，java体系中，规定实现了`javax.sql.DataSource`接口的都是数据源。

该接口定义了很多抽象方法，封装的实现类来实现它们，就能连接数据库的信息。

![chrome_s7hN5DqI2K.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_s7hN5DqI2K.png)

### 数据库属性配置文件

我们一般在项目中会使用数据库的属性配置文件，

把数据库连接信息放到单独的文件中，和Mybatis的主配置文件分开。目的时便于修改，保存，处理多个数据库的信息。

1. **在resources目录中**<mark><u>定义</u></mark>一个属性配置文件，xxx.properties
   
   其中定义数据格式是：key=value
   
   key:一般使用 . 做多级目录的，例如jdbc.mysql.driver。目的是更有层次性，可读性更高~

2. **在mybatis的主配置文件中**，使用`property`指定文件的位置，在需要使用值的地方：`${key}`就可以了（学到这里是不是发现，这正是我们之前不同库创建连接池模板的东东了呢！原来知识间是互相杂糅，融会贯通的！）

![chrome_33HxT8fc6K.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_33HxT8fc6K.png)

温习一下：编译时文件在类路径的根目录下去找！所以这里我们直接写文件名就行啦~下面使用的时候几句用${key}替换一下就好了。

### 指向多个mapper文件（sql映射文件）位置的方式

有两种方式：多个标签，或直接包名指向下所有mapper文件（还是那两个条件）

![chrome_oORay7gozP.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_oORay7gozP.png)

我记得Spring家族中很多地方的**配置文件中指向文件的位置都可以使用包名的方法**，springmvc里应该是有的，温习的时候回顾总结一下。**用到package属性**。

## 扩展

扩展功能，辅助我们mybatis的使用的。这里我们就讲一个。

### PageHelper

它是做数据分页的。我们在数据库中使用limit 0,3这种方式实现的（意思是先只取1-3行的数据）。而pageHelper就是帮我们在mybatis中实现分页操作这个小功能，它支持多种不同的数据库，不只是mysql的。

需要注意的是：不同数据库，分页语法是不一样的。

用法：

1. （Github有该代码，就是个jar包）
   
   [分页插件](https://github.com/pagehelper/Mybatis-PageHelper)

2. 我们用maven方式，pom.xml文件中**引入依赖**。这样就将github上的jar包加到maven中了。

3. 需要在mybatis主文件中**加入plugin配置**
   
   它就相当于之前讲的过滤器，在里面做分页limit语句的编写

那如何用它呢？在执行分页语句的时候执行PageHelper类中的静态方法`PageHelper.startPage(1,3);`

意思是**获取第1页中的3条内容**。

pom.xml引入依赖：

```xml
<!--PageHelper依赖-->
    <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>5.1.10</version>
    </dependency>
```

mybatis主配置文件**环境environment标签前面**加plugin配置插件：（注意**是有顺序的，不是随便加的！**）

```xml
 <!--配置插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor"/>
    </plugins>
```

谨记，PageHelper分页静态方法是在测试方法中加入的！

（第几页，一页中有多少行数据）

这样sql语句中就帮我们添加了LIMIT语句。顺便还帮我们统计了总共的行数，通过它来计算可以分几页。

注意，它不是Mybatis自带的，是国内一位大神写的小插件。
