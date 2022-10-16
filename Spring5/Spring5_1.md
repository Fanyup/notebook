# IOC操作

## Bean管理

### XML方式

#### 1.1注入集合类型属性

之前演示了注入字符串、对象类型属性。

现在来学注入数组形式，List集合，Set集合..

数组集合里能**放多个值**

1. 注入数组类型属性

2. 注入List集合类型属性

3. 注入Map集合类型属性

4. ```java
   package com.atguigu.spring5.collectiontype;
   
   import java.util.List;
   import java.util.Map;
   import java.util.Set;
   
   public class Stu {
       //1.数组类型属性
       private String[] courses;
       //2.List集合类型属性
       private List<String> list;
       //3.Map集合类型属性
       private Map<String,String> maps;
       //4.Set集合类型属性
       private Set<String> sets;
   
       //set方法，注入数组的值
       public void setCourses(String[] courses) {
           this.courses = courses;
       }
       public void setList(List<String> list) {
           this.list = list;
       }
       public void setMaps(Map<String, String> maps) {
           this.maps = maps;
       }
   
       public void setSets(Set<String> sets) {
           this.sets = sets;
       }
   }
   ```

创建类，定义集合类型属性，并且生成它们对应的set方法

2.在spring配置文件中进行配置

以前我们用value，但现在数组多了需要注入多个值。

所以我们可以用`array`或`list`标签进行数据注入，里面可以加多个value值。

- 数组类型属性注入
  
  ```xml
     <!--集合类型属性注入-->
      <!--首先，创建对象，property注入属性-->
      <bean id="stud" class="com.atguigu.spring5.collectiontype.Stu">
          <!--数组类型的属性注入-->
          <property name="courses">
              <array>
                  <value>java课程</value>
                  <value>数据库课程</value>
              </array>
          </property>
      </bean>
  ```

- list类型属性注入
  
  ```xml
   <!--list类型-->
          <property name="list">
              <list>
                  <value>张三</value>
                  <value>王五</value>
              </list>
          </property>
  ```

- map类型属性注入
  
  因为map集合是key-value键值对形式，
  
  所以这里用到了`map`,`entry`标签注入值
  
  ```xml
    <!--map类型-->
          <property name="maps">
              <map>
                  <entry key="JAVA" value="java"></entry>
                  <entry key="PHP" value="php"></entry>
              </map>
          </property>
  ```

- set类型属性诸如
  
  用到了`set`标签
  
  ```xml
          <!--set类型-->
          <property name="sets">
              <set>
                  <value>MySQL</value>
                  <value>Redis</value>
              </set>
          </property>
  ```

3.测试一下（Stu类里写一个测试方法）

```java
 @Test
    public void testCollection(){
        //加载配置文件，获取对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        Stu stu = context.getBean("stu", Stu.class);
        //调用test方法
        stu.test();

    }
```

注意，**数组是不能直接输出的**，而需要重写toString方法将数组的值转换成字符串类型输出。

这里出现了一个小插曲，小问题：我把xml里对象id记错了，改为stu后可以使用，一开始手滑写成了stud。

最终测试输出成功：

```html
[java课程, 数据库课程]
[张三, 王五]
{JAVA=java, PHP=php}
[MySQL, Redis]
```

问题：

1. 如果我想放一个user对象呢？而不是单纯的String字符串类型。

2. 我在stu里放了list集合，但在不同类中不能用。能不能提取处它，让所有类都能读取调用这个集合呢？

下面来实例：

#### 1.2设置对象类型的值

我想在集合里设置一个对象类型值

新创建一个课程类：

```java
//课程类
public class Course {
    private String cname;//课程名
    //生成它对应的set方法
    public void setCname(String cname) {
        this.cname = cname;
    }
}
```

在对象类中新增属性，表示学生上的所有课程。

```java
    //创建属性：学生所学多门课程
    private List<Course> courseList;
```

```java
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }
```

spring配置文件配置

```xml
 <!--注入list集合类型，但值是对象-->
        <property name="courseList">
            <list>
                <!--注入第一个对象-->
                <ref bean="course1"></ref>
                <!--注入第二个对象-->
                <ref bean="course2"></ref>
            </list>
        </property>
```

同时**在外面**创建多个course对象

**在前面我们外部Bean**注入对象属性时用到了`<property name="类中属性名" ref="引用类创建对象的名字">`，而这里我们同样注入对象，不过这里是注入一系列对象，即以数组形式，所以把`ref`这个标签拿出来（之前说过，可以拿出来的）再以注入数组形式注入。

```xml
    <!--创建多个course对象-->
    <!--这就是一个对象-->
    <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <!--再创建第二个对象-->
    <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Mybatis框架"></property>
    </bean>
```

课程类的toString方法重写一下，重写输出测试

得到结果：

```html
[Course{cname='Spring5框架'}, Course{cname='Mybatis框架'}]
```

#### 1.3提取集合注入使通用

把集合注入部分提取出来

**不止在当前bean类中，通用。**

1. 在spring配置文件中引入**名称空间**
   
   之前我们提到过p名称空间，`xmlns:p="http://www.springframework.org/schema/p"`
   
   现在我们引入交**util名称空间**
   
   不需要记，写法用之前的该就可以了（就是复制黏贴，修修改改：把bean全部换成util》

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
```

⚠：这里发现了一个小细节，顺序必须是按照`xmlns`的在一起，而`xis:`添加的是在里面添加，顺序不能乱，不能插入，否则会报错！

```xml
    <!--1,[提取]list集合类型属性注入-->
    <!--帮它起个名字-->
    <util:list id="bookList">
        <!--里面就类似了-->
        <!--如果是对象就用ref的方式，这里我们注入字符串，所以用value就好了-->
        <value>九阳玄功</value>
        <value>三昧真火</value>
    </util:list>
    <!--提取出集合后，我们的目的是注入到Book类的list集合中-->
    <!--首先创建对象-->
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book">
        <property name="list" ref="bookList">
            <!--这里不再写list标签了，因为我们已经把它提取出来了，而是用ref指向bookList-->
        </property>
    </bean>
```

- 这里我们再一次用到了引用`ref`，这里不在指向对象了，而是指向一个提取出的集合，我们用id帮它独一无二化叫bookList，就像创建类的对象时为它们起独一无二的id名字一样。又因为它还是属性，而我们调用它，所以自然还是用`<property>`标签啦~

测试一下

```java
    @Test
    public void testCollection2(){
        //加载配置文件，获取对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        Book book = context.getBean("book", Book.class);
        //调用test方法
        book.test();

    }
```

得到结果：

```html
[九阳玄功, 三昧真火]
```

### 工厂Bean

和我们之前说到那个工厂不是同一个东西。

**Spring有两种类型bean**，我们现在自己创建的就是一种普通bean

另外还有一种，叫工厂（Factory）Bean。两者之间有不同的区别。

<mark>Spring里内置的一种Bean类型。</mark>

#### 区别：普通Bean

特点是：

返回的bean实例就是Book类

在bean标签里定义定义的是什么类型，返回的就是什么类型（写死的）。

而工厂Bean就是你定义的bean类型和你返回的类型可以是不一样的。

#### 演示工厂Bean

1. 创建一个类，让这个类称为工厂bean。
   
   让他实现一个接口就可以了，接口就叫FactoryBean。
   
   接口需要实现哦！

2. 实现接口里的方法，在实现方法中定义返回的bean类型

```java
public class MyBean implements FactoryBean {

    //判断是否是单例（以后会提到
    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }

    //返回bean实例的类型
    //默认Object，我们要在里面加一个泛型类的写法
    @Override
    public Object getObject() throws Exception {
        return null;
    }

    //返回类类型（？吧
    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
```

r如何加泛型类写法呢？`..implements FactoryBean<Course>`

这里用Course类做泛型类示例

```java
    @Override
    public Course getObject() throws Exception {
        //工厂+反射做到，这里我们粗略带过
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
```

此时这里默认的返回类型`Object`会报错，那是当然的，因为我们定义了它的泛型。修改后，这里就提现的工厂Bean的特点！因为我们最初在xml配置文件中定义的是MyBean类型，而最终

再测试：类型不一致，报错，因为现在返回的是Course类型了。

```java
    @Test
    public void testCollection3(){
        //加载配置文件，获取对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
        Course course = context.getBean("myBean", Course.class);
        System.out.println(course);
    }
```

这里从bean3.xml配置文件中获取的对象还是myBean，但返回的是Course类型的。

测试到这里时出现了一个爆红：

```xml
 Bean named 'myBean' is expected to be of type 'com.atguigu.spring5.collectiontype.Course' but was actually of type 'org.springframework.beans.factory.support.NullBean'
```

检查后发现原来是上文忘记该返回值了！默认的是null，那么我们测试类指定要输出course时当然找不到了。

修改后成功输出结果：

```html
Course{cname='abc'}
```

这再次验证证明：我们在配置文件中定义的对象的返回类型可以不同

```xml
    <!--创建对象-->
    <bean id="myBean" class="com.atguigu.spring5.testdemo.MyBean"></bean>
```

在实现接口的getObject方法重写实现。

### bean的作用域

之前可以得到我们创建的类bean实例对象。这个实例可以时单例对象或多例对象。

**默认情况下是单实例对象**。

我们通过测试验证：

重新复用之前的Book类测试：

<u>把对象再来获取一次</u>，第一次叫它book1,第二次重复获取叫它book2。<u>若是单例对象，那么两次获取它的地址是一样的。</u>

```java
    @Test
    public void testCollection2(){
        //加载配置文件，获取对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        Book book1 = context.getBean("book", Book.class);
        Book book2 = context.getBean("book", Book.class);
        //将获取对象book1和book2分别做输出
        System.out.println(book1);
        System.out.println(book2);
```

结果验证——默认情况下是单实例的！（即单个实例对象）

```xml
com.atguigu.spring5.collectiontype.Book@103f852
com.atguigu.spring5.collectiontype.Book@103f852
```

在Spring里面，我们可以设置你创建的bean实例是单还是多实例。

#### 设置多实例对象

1. 在Spring配置文件bean标签里有属性用于设置单实例还是多实例。
   
   它叫`scope`属性，**<mark>本意n有“范围”的意思，就是“域”嘛！</mark>**
   
   scope属性有多个值，常用有两个：
   
   1. **默认值**，`singleton`，“单个的人，**单项物**”表示是一个单实例对象
   
   2. `prototype`，表示是多实例对象
   
   用scope属性修改默认值后再测试——地址变了：
   
   ```java
   com.atguigu.spring5.collectiontype.Book@103f852
   com.atguigu.spring5.collectiontype.Book@587c290d
   ```

2. singleton和prototype其他区别
   
   1. 设置scope值（默认）是singleton时候，**加载spring配置文件时**就会创建单实例对象。
   
   2. 设置scope值时prototype时，并不是在加载spring配置文件时创建对象，而是**调用getBean方法时**创建多实例对象。
   
   3. scope属性中`request`（一次请求）和`session`（一次会话），基本不用。

### bean的生命周期

1. 什么是生命周期？
   
   从对象创建到销毁的过程

2. bean的生命周期
   
   1. 通过构造器创建bean实例（默认找无参构造（构造器）创建，或可以使用工厂bean创建）
   
   2. 为bean里面的属性来设置值，和对其他bean（外部或内部）的引用（调用set方法）
   
   3. **调用bean的初始化的方法**（<u>需要进行配置</u>）
   
   4. bean可以使用了（对象实例已经获取到了）
   
   5. 当容器在关闭时，会调用bean的销毁方法（<u>需要进行配置</u>）

3. 演示一下bean的生命周期

**第一、二步**：

```java
public class Orders {
    private String oname;
    //为了让效果更明显，我们写出无参构造

    public Orders() {
        System.out.println("第一步：创建bean实例，调用无参构造");
    }

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步：调用set方法，注入属性值");
    }
    //创建执行的初始化的方法（需要自行配置）
    //我们这里只是一个普通的方法，默认执行，需要配置，让它“初始化执行”
    public void initMethod(){
        System.out.println("第三步：执行初始化方法");
    }
}
```

（~~这里我发现，如果不配置销毁方法的话测试类是无法执行的，爆红，或与我们这里设置了初始化方法有关~~）没事了，我发现是复制test方法时贴到了test外面...

**第三步**：设置bean的初始化方法

```java
 //创建执行的初始化的方法（需要自行配置）
 //我们这里只是一个普通的方法，默认执行，需要配置，让它“初始化执行”
    public void initMethod(){
        System.out.println("第三步：执行初始化方法");
    }
```

在bean标签中找到属性`init-Method`+初始化方法名字（不用加括号哦）

还需要自己设置销毁方法哦！

```java
    //创建执行的销毁的方法（需要自行配置）
    //我们这里只是一个普通的方法，默认执行，需要配置，让它“初始化执行”
    public void destroyMethod(){
        System.out.println("第五步：执行销毁方法");
    }
```

**第四步**：对象获取到了，test测试一下

```java
    @Test
    public void testBean3(){
        //加载配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
        //第四步：获取到了创建的bean实例对象
        Orders orders = context.getBean("orders", Orders.class);
        System.out.println(orders);
    }
}
```

`Ctrl+H`查看类/接口结构，将上修改为其子实现类，来调用close销毁方法

```java
  @Test
    public void testBean3(){
        //加载配置文件
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
        //第四步：获取到了创建的bean实例对象
        Orders orders = context.getBean("orders", Orders.class);
        System.out.println(orders);

        //需要手动让bean实例销毁，它才会调用销毁方法
        //ApplicationContext接口的实现类中有方法close()
        context.close();
    }
}
```

这里有一个问题，执行看右边，因为接口中没有close方法，所以你用`ApplicationContextcontext`左边声明是必报错的，因为它根本没有。

执行得到结果:

```java
第一步：创建bean实例，调用无参构造
第二步：调用set方法，注入属性值
第三步：执行初始化方法
com.atguigu.spring5.bean.Orders@543c6f6d
第五步：执行销毁方法
```

#### 添加后置处理器效果

其实还有两个步骤:（**在第三步初始化之前与之后**）

加上bean的后置处理器

初始化之前、之后：

**把bean的实例传递给bean后置处理器，传给一个方法**

演示添加后置处理器效果：

1. 创建类，实现接口BeanPostProcessor，创建后置处理器
   
   查看接口快捷键`Ctrl + 左键` 或者`Ctrl + Alt`

初始化之前执行：  

```java
 @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化执行之前");
        return bean;
    }
```

初始化之后执行：

```java
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
```

为了能看到效果，我们在里面做个注入，输出一下。

但这样只是我们创建的一个普通类，spring并不知道他是后置处理器，所以需要我们在xml里配置一下。

（外部bean）

添加后会对我们配置的所有bean都添加上后置处理器。即里面的方法都会执行。

```xml
     <!--配置后置处理器-->
    <bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

执行结果：

```java
第一步：创建bean实例，调用无参构造
第二步：调用set方法，注入属性值
初始化执行之前
第三步：执行初始化方法
初始化执行之后
com.atguigu.spring5.bean.Orders@7403c468
第五步：执行销毁方法
```
