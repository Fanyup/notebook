# Mybatis框架

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


