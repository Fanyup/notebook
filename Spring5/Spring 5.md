## 大纲

1. Spring基本概念

2. IOC容器

3. Aop

4. JdbcTemplate
   
   （把JDBC做了封装，更方便的操作数据库）

5. 事务管理

6. Sprin5新特性

## Spring概念

1. Spring是一个**轻量级**的开源的JavaEE**框架**
   
   我们用一个框架，得引入里面一些相关依赖（jar包）
   
   Spring里体积小，不需依赖其他的，可以独立进行使用
   
   **框架：它能让我们的开发更方便，代码更简洁。解决企业应用开发的复杂性。**
   
   Spring有**两个核心部分**：IOC与Aop
   
   1. <mark>IOC</mark>
      
      **控制反转/反转控制**
      
      把创建对象的过程交给Spring进行管理，（帮我们创建对象，进行对象实例化）不需要再用new的方式
   
   2. <mark>Aop</mark>
      
      **面向切面**
      
      想在一个程序中扩展加个功能
      
      **在不修改源代码的情况下**，进行功能添加或增强

2. Spring特点
   
   - 方便解耦，简化开发（解耦合）
     
     用IOC之后可以将耦合性降低
   
   - Aop编程支持
     
     不改变源代码，可以增强它的功能
   
   - 方便程序测试
     
     不需要写完整过程，对Junit4支持🔺
     
     - 方便继承各种优秀框架
       
       可以单独用，也可以整合框架(`Struts2``Hibernate```Mybatis`等)
   
   - 降低API开发难度
     
     把很多东西做了封装，方便进行事务的管理（如封装JDBC）
   
   - Java源码中经典代码

### 入门案例

1. 需要下载相关需要jar包（Spring5)    `sping.io`官网
   
   - `GA`稳定版；`SNAPSHOT`快照版
   - `Artifact`是Maven中一个概念
   - [下载地址]([JFrog](https://repo.spring.io/ui/native/release/org/springframework/spring/))Spring的所有版本
   - 解压放在了`D:\Program Files\spring-5.2.6.RELEASE-dist`路径下

2. 创建普通Java工程

3. 导入Spring5相关jar包(依赖)
   
   ![](C:\Users\up\AppData\Roaming\marktext\images\2022-09-29-18-49-13-image.png)

    实现基本功能起码要core（4）

    导入jar包（复制）->**File--Project Structure--Module(模块)**--➕<u>**添加依赖**</u>

4. 开始写代码
   
   想用Spring方式创建一个对象
   
   创建一个普通类，在这个类创建一个普通方法
   
   ```java
   package com.atguigu.spring5;
   
   public class User {
       public void add(){
           System.out.println("add方法");
       }
   }
   ```
   
   - 配置文件√
   
   - 注解√（两种方式都可以，先选方法1）

5. 创建Spring配置文件，在配置文件中配置要创建的对象
   
   1. Spring配置文件使用xml格式
      
      ```xml
      <bean id="user" class="com.atguigu.spring5.User"></bean>
      </beans>
      ```

6. 测试代码编写（实际项目应用中不会写到）
   
   ```java
   package com.atguigu.spring5.testdemo;
   
   import com.atguigu.spring5.User;
   import org.junit.Test;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   public class TestSpring5 {
   
       @Test
       public void testAdd(){
           //1.加载spring配置文件
           ApplicationContext context =
                   new ClassPathXmlApplicationContext("bean1.xml");
           //因为配置文件在类路径src下所以调用ClassPath可以直接写它的名字
   
           //2.获取配置创建的对象,得到对象
           User user = context.getBean("user", User.class);
           //3.调用对象方法
           System.out.println(user);
           user.add();
       }
   }
   ```

出现的一个小问题：爆红异常原因：commons-logging（日志）版本导错，此处必须要1.1.1版本的

其中注意：测试注解@Test需导入Junit4包🔺（我猜应该是JDK自带的，因为可以直接导包，用来测试单元执行）

[commons-logging-1.1.1.jar](https://repo1.maven.org/maven2/commons-logging/commons-logging/1.1.1/commons-logging-1.1.1.jar)

成功执行，测试说明对象创建成功。打印了对象地址，并调用对象方法。

```java
com.atguigu.spring5.User@27f723
add方法
```

#### JUnit补充

JUnit是单元测试框架

[参考W3C基本用法]([JUnit - 基本用法_w3cschool](https://www.w3cschool.cn/junit/2wjx1hvc.html))

##### 注意事项

- **测试方法必须使用 <mark>@Test</mark> 修饰**
- 测试方法**必须使用 <mark>public void</mark> 进行修饰**，不能带参数
- 一般使用单元测试会**新建一个 test 目录存放测试代码**，在生产部署的时候只需要将 test 目录下代码删除即可
- 测试代码的包应该和被测试代码包结构保持一致
- 测试单元中的**每个方法必须可以独立测试**，方法间不能有任何依赖
- 测试类一般使用 Test 作为类名的后缀
- 测试方法使一般用 test 作为方法名的前缀

##### 常用注解

- **@Test:将一个普通方法修饰成一个测试方法**@Test(excepted=xx.class): xx.class 表示异常类，表示测试的方法抛出此异常时，认为是正常的测试通过的 @Test(timeout = 毫秒数) :测试方法执行时间是否符合预期

## IOC容器

1. IOC是什么？能做什么事？底层原理？
   
   不是一门技术，而是**一种设计思想**

2. IOC接口（一个重要接口：BeanFactory)

3. IOC具体操作
   
   什么是Bean管理？（基于xml或注解方式）

### 什么是IOC

控制反转/反转控制：（一种软件设计模式）

<mark>**对象的控制权转移了，从开发人员转移到对象容器了！**</mark>

其遵循软件工程重点依赖倒置原则；

依赖注入（DI,Dependency Injection)时Spring框架实现控制反转的一种方式。（注释）

把对象的创建和对象之间的调用过程，都交给Spring管理

解耦合（降低耦合度）

入门案例就是IOC实现

#### IOC底层原理

1. xml解析

2. 工厂模式

3. 反射

**举例**：

    两个类，想在UserService类中调用UserDao中的方法

    过去做法：

    `UserDao dao = new UserDao();`创建实例对象

    `dao.add();`调用方法

    存在一个问题：耦合度太高了，关联过于紧密，不利于程序扩展。

解决方案：工厂模式--为了解耦合

    注意，工厂模式并不是最终过程，而是演变过程，最终我们会用IOC实现。

1. **<mark>建一个工厂类</mark>**`UserFactory`创建一个方法<mark>**（其实就是接口多实现）**</mark>

2. 该方法返回Dao类对象；
   
   ```java
   class UserFactory{
       public static UserDao getDao(){
           return new UserDao();
   }
   }
   ```

3. 此时UserService不需要再在类里直接new Dao对象
   
   而是调用工厂
   
   ```java
   UserDao dao =UserFactory.getDao();
   dao.add();
   ```

新问题：工厂还是有耦合度

🔺**Java实体书**

最终目的：将耦合度降到最低限度——IOC

#### IOC解耦过程

1. xml配置文件
   
   配置需要创建的对象（创建哪些对象？）
   
   ```html
   <bean id="dao class="全类名"></bean>
   ```

2. 有一个service类和dao类，想在service类中调用dao类。
   
   创建一个工厂类
   
   ```java
   class UserFactory{
       public static UserDao getDao(){
           String classValue = class属性值;
   //xml解析获取类名
   //用反射和类名调用方法
   //dom4j是xml解析的一种
   //通过反射创捷对象，先得到类的字节码文件
           Class c = Class.forName(classValue;)
           return(UserDao)c.newInstance();//创建对象
   //return强转
   
   }
   }
   ```

        注意，此时通过xml解析+反射方式

        现在只需修改配置文件，其他地方不用动——解耦合！进一步降低耦合度

### IOC（两个重要接口）

1. IOC思想基于IOC容器完成，

2. IOC容器底层是一个“对象工厂”

3. Spring提供IOC容器实现的两种方式（**两个重要的接口**）
   
   1. **BeanFactory**：IOC容器最基本实现方式，是Spring内部使用接口，一般不提供给我们开发人员使用。
      
      - 加载配置文件时，不会创建对象，
      
      - 而在获取（使用）对象时，它才去创建这个对象。
   
   2. <mark>**ApplicationContext**</mark>：**BeanFactory接口的子接口**。提供更多、更强大的功能。一般面向开发人员进行使用。（常用）
      
      - 在加载配置文件时，就会把在配置文件中对象进行创建。
   
   功能相似
   
   他们都能实现加载配置文件，通过工厂过程去创建对象。
   
   把耗时耗资源的操作都在（交给）项目（服务器）启动时创建对象更好，而不是什么时候用什么时候启动，所以ApplicationContext接口更好

4. ApplicationContext接口里的主要实现类：
   
   点击类➕`Ctrl+H`打开类的结构

![](C:\Users\up\AppData\Roaming\marktext\images\2022-09-30-13-49-13-image.png)

ClassPath（类路径）

### Bean管理

1. 什么是Bean管理？
   
   指的是两个操作：
   
   1. **Spring创建对象**
   
   2. **Spring注入属性**

2. Bean管理操作有两种实现方式：
   
   1. 基于**xml配置文件方式**实现
   
   2. 基于**注解方式**实现

#### 1.基于xml方式创建对象

```xml
<bean id="dao class="com.atguigu.Spring5.User"></bean>
```

使用bean标签，在标签里添加对应属性，就可以实现对象创建。

Bean标签里有很多属性，最常用属性：

- **id属性**
  
  获取对象的唯一标识

- **class属性**
  
  创建对象所在类的**全路径（含包）**

- name属性
  
  和id相比，可以加特殊符号。早期属性，为Structs1提供，现在很少用。

**<mark>创建对象时，默认也是执行无参的构造方法。</mark>**

若添加有参，此时报错。`Caused by: java.lang.NoSuchMethodException: com.atguigu.spring5.User.<init>()`

#### 2.基于xml方式注入属性

##### 2.1属性注入的两种基本方式

1. **DI：依赖注入**
   
   就是注入属性，是Ioc中一种具体实现，就表示依赖注入。但需要在创建对象的基础之上完成。

**示例**：

快捷键`Alt+Fn+Insert`直接生成set方法、构造函数等等

1. **（原始方式）**（set方式或有参构造注入）
   
   ```java
   package com.atguigu.spring5.testdemo;
   
   public class Book {
       public void setBname(String bname) {
           this.bname = bname;
       }
       private String bname;
   
       public static void main(String[] args) {
           Book book = new Book();
           book.setBname("abc");
       }
   }
   ```

2. 在Spring配置文件配置对象的创建，再配置属性注入
   
   <mark>property属性</mark>（**set方法**）
   
   .xml配置文件下：
   
   ```xml
   <!--创建对象-->
       <bean id="book" class="com.atguigu.spring5.testdemo.Book">
           <!--set方法注入属性,通过对象方法注入属性，要写到标签里面-->
           <!--使用标签property完成属性注入-->
           <property name="bname"value="hi！"></property>
           <!--name是类里指定的属性名称，value是要注入的值-->
       </bean>
   ```
   
   **set方式这里我们还可以作进一步简化**：
   
   不用property属性方式
   
   **p名称空间注入**
   
   可以简化配置方式（了解）（实际中用的并不多）
   
   1. 添加一个p名称空间在配置文件中
      
      `xmlns:p=...p`
   
   2. 进行属性注入，可以在bean标签里进行
      
      `<bean...p:name="bname" value="hi!"></beans>`
      
      用这种方法就不用写property属性了
   
   方式2(**通过有参构造方式**)
   
   在配置文件中进行配置
   
   <mark>constructor-arg属性</mark>

```xml
<!--创建对象-->
    <!--id：获取对象的唯一标识-->
    <bean id="orders" class="com.atguigu.spring5.Orders"></bean>
```

   此时这样写会爆红，为什么呢？因为默认找无参构造方法，而我们定义了有参后，无参构造就没了。

   (注：还可以使用`<constructor-args index="1"><../>`**索引方式**)

   于是我们更改成:

```xml
<bean id="orders" class="com.atguigu.spring5.Orders">
        <constructor-arg name="oname" value="computer"></constructor-arg>
        <constructor-arg name="address" value="China"></constructor-arg>
    </bean>
```

3. 测试一下@Test(具体实现)
   
   （首先原Orders类中给定义一个方法，能输出才能看到测试结果。）
   
   ```java
    @Test
       public void testOrders(){
           //1.加载spring配置文件
           ApplicationContext context =
                   new ClassPathXmlApplicationContext("bean1.xml");
           //因为配置文件在类路径src下所以调用ClassPath可以直接写它的名字
   
           //2.获取配置创建的对象,得到对象
           Orders orders = context.getBean("orders", Orders.class);
   
           //3.调用对象方法
           System.out.println(orders);
           orders.orderTest();
       }
   ```
   
   这里一开始出现了一个小问题：测试爆红；其原因写的是xml第14行写错了。查看发现是value与class属性定义直接少了一个空格。

##### 2.2注入其他形式属性

对象形式..其他形式..

1. **字面量**
   
   设定使用的固定值（写死的）
   
   以下这两种情况该怎么做呢？：
   
   - null值
     
     向属性里面设置控制，可以用`null标签做到`：（不用写value属性）
     
     ```xml
     <property name="bname" >
                 </null>
     </property>
     ```
   
   - **属性值中包含特殊符号**
     
     如我想在value赋值里加入`<>`这样的特殊符号，但此时`property`标签会报错，因为它把它当作一个标签来识别了。
     
     解决方式一：**把特殊符号进行转义**，用`&lt;`和`&gt;`
     
     解决方式二：把特殊符号内容写到**CDATE**
     
     首先要注意，属性是可以分开写的，形式像这样：
     
     ```xml
     <property name="bname" >
                 <value></value>
             </property>
     ```
- - 接下来，我们可以这样写：
    
    ```xml
     <property name="bname" >
                <value><![CDATA[<<南京>>]]]]>
                </value>
            </property>
    ```
    
    `![CDATE[...]]`
    
    此时测试时会输出我们要的内容：`<<南京>>`

###### 2.2.1.注入外部bean

举例演示：web层-service层-Dao层

演示过程：

1. 创建两个类：Service类和Dao类

2. 在Service调用Dao里面的方法
   
   通过xml配置方式完成
   
   接口：
   
   ```java
   package com.atguigu.spring5.dao;
   
   public interface UserDao {
       public void update();
   }
   ```
   
   接口实现类：快捷键`alt+Fn+enter`重写接口方法

```java
package com.atguigu.spring5.dao;

public class UserDaoImpl implements UserDao{

    @Override
    public void update() {
        System.out.println("dao的update方法");
    }
}
```

**<mark>通过UserService中调用UserDao</mark>**（原始方式）（但我们这里

```java
public class UserService {

    public void add(){
        System.out.println("service add...");
        //原始方式
        //创建UserDao对象
        UserDao user = new UserDaoImpl();
        //多态
        user.update();
    }
}
```

<mark>用Spring方式实现</mark>：

在spring配置文件中进行配置

1. 把userdao注入到userservice中就可以（set方法或有参构造）现在**注入属性**：**对象类型**
   
   通过生成set方法完成注入（快捷键`alt+Fn+Insert`）
   
   ```java
   public class UserService {
   
       //创建UserDao类型属性(对象类型），生成对应set方法
       private UserDao userDao;
       public void setUserDao(UserDao userDao) {
           this.userDao = userDao;
       }
   
       public void add(){
           System.out.println("service add...");
           userDao.update();
       }
   }
   ```
   
   注意：二周目我在这里做了一个测试：将set方法去掉再进行测试会怎样？结果是爆红！
   
   我理解这里的含义是：首先创建一个UserDao类的对象类型，此时需要通过**创建Setter方法**，将此时的形参对象指向UserDao类的**当前**对象这个对象；（因为this本身作用就是指向当前对象），这样就将UserService类中创建的这个对象类型属性，成功指向userDao类中对象了。（即**将UserDao类的实例成功注入到UserService类中**），这样就可以调用UserDao类中方法了。

2. 在**配置文件中进行配置**(在配置文件中把两个类的对象创建)

```xml
 <!--创建service和dao对象-->
    <!--id名可以随便起，但作用是唯一指向-->
    <bean id="userService" class="com.atguigu.spring5.service.UserService">
        <!--注入userDao对象-->
        <!--name是类里面的属性名称，这里指上文对象类型属性名称-->
        <!--因为注入的是对象，所以此时不写value，而是ref-->
        <!--ref属性：指向创建的实现类对象的bean标签的id值（唯一）-->
        <!--ref就完成了外部bean注入，注入了其他对象-->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>
    <bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
```

ref即为reference，有“**引文，引证**，参考，参照”之意。

4. 测试效果（`import org.junit.Test;`）

```java
public class testBean {
    @Test
    public void testAdd(){
        //1.加载spring配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        //2.获取配置创建的对象
        UserService userService = context.getBean("userService",UserService.class);
        userService.add();//因为我们将add方法里调用了UserDao类实例的update方法
    }
```

成功输出：

```html
service add...
dao的update方法
```

###### 2.2.2注入内部bean和级联赋值

1. 一对多关系：典型关系：部门和员工
   
   一个部门里有多个员工，一个员工属于一个部门
   
   部门是一，员工是多
   
   **我们要在实体类之间表示一对多的关系**

```java
//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式进行标识
    //表示员工所属的部门（对象类型属性）
    private Dept dept;

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
}
```

```java
//部门类
public class Dept {
    private String dname;

    public void setDname(String dname) {
        this.dname = dname;
    }
}
```

2. 写spring配置文件，通过配置文件，完成内部bean的操作。向里面注入属性。

```xml
    <!--内部bean-->
    <!--创建员工类实例对象-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置属性（3个）普通基本属性和对象属性-->
        <property name="ename" value="Jack"></property>
        <property name="gender" value="男"></property>
        <!--对象类型属性-->
        <property name="dept" 【这里该如何写？】
    </bean>
```

此时设置对象类型属性有**两种方法**：

1. 用刚刚我们学的外部bean方式：
   
   在xml中创建一个部门类的对象，用`ref`将其引入

2. 这里我们用内部bean写法
   
   嵌套：在里面**把dept的对象创建**出来
   
   再在这里面设置属性值（一个部门总得有个名儿吧！上面的UserDao为什么不用呢，因为这里是两个不同的案例）
   
   **在一个bean里面可以嵌套另一个对象**   
   
   ```xml
    <!--内部bean-->
       <!--创建员工类实例对象-->
       <bean id="emp" class="com.atguigu.spring5.bean.Emp">
           <!--设置属性（3个）普通基本属性和对象属性-->
           <property name="ename" value="Jack"></property>
           <property name="gender" value="男"></property>
           <!--对象类型属性-->
           <property name="dept">
               <bean id="dept" class="com.atguigu.spring5.bean.Dept">
                   <property name="dname" value="安保部"></property>
               </bean>
           </property>
       </bean>
   ```

但实际中还是很多人喜欢用外部bean，因为这样更清晰；

3. 测试一下：
   
   在员工类中加一个add()方法，显示输出值，证明调用成功
   
   为了显示对象的值，我们再部门类中重写一下toString方法
   
   ```java
    @Override
       public String toString() {
           return "Dept{" +
                   "dname='" + dname + '\'' +
                   '}';
       }
   ```

快捷键`Alt+Fn+Insert`

这里想提一点，因为toString是Object类的方法，所以会默认调用。我们在这里重写了它，这样输出dname时即会根据重写方法输出dname的值；如果这里我们不重写toString方法，得到的结果会是：

`Jack男com.atguigu.spring5.bean.Dept@7403c468`即dname的内存地址。因为dname是实例对象，我们在xml文件中为dept对象的属性dname赋了一个值。**该例与userDao类不同的区别是，上文类只有方法而没有成员属性**。如果有，也是一样的用法的。

```java
 @Test
    public void testBean2(){
        //1.加载spring配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");

        //2.获取配置创建的对象
        Emp emp = context.getBean("emp",Emp.class);
        emp.add();
    }
```

成功获得结果

```html
Jack男Dept{dname='安保部'}
```

**刚才的过程其实就是一种级联赋值**；

**级联赋值还有其他写法**；

###### 2.2.3级联赋值

1. 方法一：外部bean方式引入
   
   ```xml
       <!--级联赋值-->
       <!--内部bean-->
       <!--创建员工类实例对象-->
       <bean id="emp" class="com.atguigu.spring5.bean.Emp">
           <!--设置属性（3个）普通基本属性和对象属性-->
           <property name="ename" value="Jack"></property>
           <property name="gender" value="男"></property>
           <!--对象类型属性-->
           <!--级联赋值-->
           <property name="dept" ref="dept"></property>
   
       </bean>
       <bean id="dept" class="com.atguigu.spring5.bean.Dept">
           <property name="dname" value="财务部"></property>
       </bean>
   </beans>
   ```

重新测试一下：成功

2. 方法二：

复盘后对于方式二我是这样理解的：上面都是通过创建部门类的对象的方式注入dname属性的值，所以员工部只需要dept对象的set方法就好了，即指向它。而这种形式则不通过此，而是直接在员工部的dept对象中修改，注入dname属性。那么此时会遇到一个问题，就是我们用set方法只能指向而不能返回，即修改值，这是就需要get方法来`return dept`返回对象类型，通过对象类型的牵引找到dname。（结论：为了能找到dname）

```xml
 <!--对象类型属性-->
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        <property name="dept.dname" value="技术部"></property>
```

`dept.dname`：向dept的对象属性中设置dname的值（对象类型属性名.成员属性）

但是报错了，为什么？

因为我们需要向员工类中的dept对象类型属性中的dame设置值，但我们没有得到对象，所以我们需要生成dept的get方法

在员工部修改如下：（添加入get方法）

```java
 private Dept dept;

    public Dept getDept() {
        return dept;
    }

    public void setDept(Dept dept) {
        this.dept = dept;
    }
```

修改后xml如下：

```xml
    <!--级联赋值-->
    <!--内部bean-->
    <!--创建员工类实例对象-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置属性（3个）普通基本属性和对象属性-->
        <property name="ename" value="Jack"></property>
        <property name="gender" value="男"></property>
        <!--对象类型属性-->
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        <property name="dept.dname" value="技术部"></property>

    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <!--此时这句可要可不要，因为最终会被上面覆盖，所以可以删去-->
        <property name="dname" value="财务部"></property>
    </bean>
```
