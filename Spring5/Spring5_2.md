# IOC操作

## Bean管理

### XML方式

#### XML自动装配

1. 什么是自动装配
   
   之前在配置文件中通过加入name,value这些属性的值。这种方式叫做手动装配。
   
   根据属性名称、类型，spring自动完成属性注入。
   
   即根据指定的装配规则（**属性名称/类型**），Spring自动将匹配的属性值进行注入。

2. 演示自动装配过程（部门，员工类例子）
   
   部门类：
   
   员工类：
   
   完成注入关系，建个配置文件
   
   bean标签中属性`autowire`，配置自动装配
   
   而不需要再用property属性注入（外部/内部)bean类的属性了！
   
   `autowire`属性常用两个值：
   
   1. **byName**
      
      根据属性名称注入
      
      **前提条件：<mark>（外部/内部）注入bean的id值</mark>和<mark>类属性名称</mark>一样**
   
   2. **byType**
      
      根据属性类型注入
      
      有一个问题：如果有两个相同类型的外部bean，此时就无法判断了，只能用byName方式注入。
   
   但实际中用到很少！**<mark>一般都用注解方式自动装配。</mark>**

#### 引用外部属性文件

按以前方法，写死了，每次修改都要改xml配置文件，很不方便。

所以我们把固定、相关值放到一个其他文件中，如property文件

再把property文件引入到xml文件中。

比方说数据库值放到property文件中，再引入到xml文件。

1. 直接配置数据库信息（分布式不是很好用）
   
   1. 配置Druid连接池（缓冲池）
   
   2. 引入Druid连接池依赖，jar包`druid-1.1.9.jar`
      
      复制到lib包中，再File->projectStructure引入到当前项目中

2. 引入外部属性文件，配置数据库的连接池
   
   ```xml
   <!--创建druid连接池对象-->
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
           <!--驱动名称，8.0后驱动名记得加cj-->
           <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
           <property name="url" value="jdbc:mysql://localhost:3306/bjpowernode"></property>
           <property name="username" value="root"></property>
           <property name="password" value="密码"></property>
   
       </bean>
   ```

现在注入值都是固定的，可以写在property配置文件，在引入到xml

#### 引用外部属性文件：正文

创建property配置文件`jdbc(文件名).properties`

等号左边可以随便写，最好加个前缀，保证不会冲突

```properties
prop.driverClass=com.mysql.cj.jdbc.Driver
prop.url=jdbc:mysql://localhost:3306/bjpowernode
prop.userName=root
prop.password=密码
```

把外部properties属性文件引入到spring配置文件中，在进行读取。

**先引入context名称空间**（之前有p,util)

按照之前方式快速修改xmlns和xsi

`xmlns:context="http://www.springframework.org/schema/context"`

![ApplicationFrameHost_qqHxou5V2E.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_qqHxou5V2E.png)

在spring配置文件中

表达式`value="${properties中配置等号左边的东西}"`

```xml
   <!--引入外部的属性文件-->
    <!--加一个context标签 用属性location-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!--驱动名称，8.0后驱动名记得加cj-->
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>

    </bean>
```

**经常用到数据库配置中，之后会用到**

## 注解方式

基于注解方式，实现Spring对象创建及注入属性

1. 什么是注解？
   
   1. 代码特殊标记，格式：`@注解名称（属性名称=属性值,属性名称=属性值..`
   
   2. 可以作用在类、方法、属性等等上
   
   3. 使用目的：简化xml配置，把配置使用更优雅、简介方式实现

2. Spring针对Bean管理中**创建对象**提供了以下**四个**注解：
   
   1. `@Component`（组成部分，成分，组件）
   
   2. `@Service`一般用在业务逻辑层/serivce层
   
   3. `@Controller`一般用在web层
   
   4. `Repository`一般用在Dao层

但实际中可以互通使用，只是为了我们开发更加方便，建议而已。

**<mark>功能是一样的，都可以用来创建bean的实例(即对象)</mark>**

### 对象创建

（准备工作：复制一份Project项目，用Open打开，方便阅览）

1. 先**引入依赖**
   
   需要一个额外的依赖`spring-aop-5.2.6.RELEASE.jar`
   
   Maven仓库：[https://mvnrepository.com/]()

2. **开启组件扫描**
   
   告诉Spring容器，我现在在哪个类里要加上注解，然后区扫描这些类。看哪个位置中有注解，有就创建对象。
   
   <mark>**先引入名称空间context**</mark>
   
   用`context:component-scan`组件
   
   **多包扫描**：
- 方法一：用逗号隔开，但是都是包全名

- 方法二：扫描包上层目录（它们都在com.atguigu包下）

```xml
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

1. 创建类，在类上面添加创建对象的注解（**四个中的任何一个都可以**）
   
   **注意：在类上面加注解**，因为我们要创建对象！

```java
package com.atguigu.spring5.service;

import org.springframework.stereotype.Component;

//里面的value值和<bean id="userService" class".."/>的id值等价
//value属性值可以省略不写，默认值就是类名称，其首字母小写
@Component(value = "userService")
public class UserService {

    public void add(){
        System.out.println("servic add...");
    }
}
```

- 里面的**value值**和`<bean id="userService" class".."/>`的id值等价  

- **value属性值可以省略不写**，默认值就是类名称，其首字母小写

测试一下：成功

执行流程：

1. 加载配置文件；
   
   （xml中我们刚刚写的）**开启注解扫描**，到包中所有类，
   
   若有相关注解，则把相关对象创建！

下面来讲一个细节问题：

#### 组件扫描配置

如果按照上面扫描包，会把包里所有类都扫描到。

我们可以约定哪些扫描，哪些不扫（细化）

示例一（最常见方法）（先能看懂）

![ApplicationFrameHost_vU2jXvINf3.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_vU2jXvINf3.png)

这里添加了属性`use-default-filters=false`

默认过滤：表示现在不适用默认filter，

即不扫描所有东西，我们自己配置规则。

`context:include-filter`筛选：设置**扫描**哪些内容扫描

在com.atguigu包中只扫描带**这个**的类（这里他没讲清楚，应是筛选条件）

示例二：

扫描包里所有内容

`context:exclude-filter`：设置哪些内容**不进行**扫描

### 属性注入

1. `@Autowired`
   
   表示根据**属性类型**进行注入，跟xml中`byType`,`byName`意思差不多

2. `@Qualifier`
   
   根据**属性名称**进行注入

3. `@Resource`
   
   可以根据类型注入，也可以根据名称注入

<mark>注意</mark>：这三个注解都是**针对对象类型**注入，普通类型用

`@Value`注入

#### 对象类型属性注入

##### @Autowired

示例：想在UserService类中注入Dao对象，调用Dao中的方法

1. 把service和dao类对象创建，分别在他们类上面添加创建对象的注解（上面刚学习的）

2. 在service类里注入dao对象：
   
   定义dao对象类型属性
   
   在属性上面使用【属性注入】注解

UserDaoImpl接口实现类：

```java
package com.atguigu.spring5.dao;


import org.springframework.stereotype.Repository;

@Repository
public class UserDaoImpl implements  UserDao{

    @Override
    public void add() {
        System.out.println("dao add...");
    }
}
```

```java
package com.atguigu.spring5.dao;

public interface UserDao {
    public void add();
}
//接口
```

UserService类（<mark>重点在这里</mark>）

```java
//里面的value值和<bean id="userService" class".."/>的id值等价
//value属性值可以省略不写，默认值就是类名称，其首字母小写
@Service
public class UserService {

    //因为它是接口实现类，接口类型，体现多态
    //不需要添加set方法，注解里面已经帮我们封装好了
    //添加【属性注入】注解

    @Autowired //根据类型【这里是UserDao，找到实现类】
    private UserDao userDao;

    public void add(){
        System.out.println("servic add...");
        userDao.add();
    }
}
```

测试一下：

```java
    @Test
    public void testService(){
        ApplicationContext context
                = new ClassPathXmlApplicationContext("bean1.xml");
        UserService userService = context.getBean("userService", UserService.class);
        System.out.println(userService);
        userService.add();
 }
```

成功输出：

```java
com.atguigu.spring5.service.UserService@7fc2413d
servic add...
dao add...
```

这是最常使用的一种方式。

##### @Qualifier

根据**属性名称**进行注入，<mark>注意</mark>：要和上面的`@Autowired`一起使用。

之前说过，**一个接口有多个实现类**，则像`byType`就找不到了，这样就只能用**属性名称**注入。

value不写默认是【类名小写】

- 这里的属性名称值什么？
  
  即**<mark>id值</mark>**(即`<bean id="",class=""`创建bean类对象实例)
  
  ```java
  @Repository(value = "userDaoImpl1")
  public class UserDaoImpl implements  UserDao{
  ```

- UserService类中
  
  ```java
      @Autowired
      //根据类型【这里是UserDao，找到实现类】
  
      //根据对象属性的id名称进行注入
      @Qualifier(value = "userDaoImpl1")
      //具体指定用的是哪个实现类的对象
      private UserDao userDao;
  ```

`@Qualifier`中的<mark>value值与id名对应</mark>**，指向调用/引用的的（外部）bean类对象实例，然后再通过`<property name="定义对象名" ref="外部bean类id名"`来引用（注入对象属性），这里**与xml不同的是，

1. <mark>无需</mark>再在UserService类中写入`private UserDao userDao;`的**同时创建它的set方法**，因为`@Autowired`或和`Qualifiery`已经帮我们封装好了！

2. <mark>无需</mark>再像xml在bean中通过创建外部/内部bean对象实例方式调用实例，只需用`@Autowired`查找同类名属性类型，或加用`@Qualifier`寻找对应id名的（需要调用的）（也就是我们用外部方式写的）类就好了！

实现属性注入。

##### @Resource

既可根据类型注入，也可根据名称注入

```java
    //根据类型进行对象属性注入
    @Resource
    private UserDao userDao;
```

或

```java
    //根据名称进行对象属性注入
    @Resource(name = "userDaoImpl1")
    private UserDao userDao;
```

这里不是`value`而是`name`了

注意：其实Resource是java扩展包javax中的。官方建议我们用上面两种。

### 普通类型属性注入

如字符串等

用注解方式注入 也不需要我们写xml配置文件

```java
    @Value(value="abc")
    private String name;
```

## 完全（纯）注解开发

<mark>可以没有配置文件！</mark>而用纯注解方式。

1. **创建配置类**，代替xml配置文件

config”配置“的意思

我们单创建普通SpringConfig类说他是配置类，Spring它是不认的。

所以需要在类上加上注解`@Configuration`，把当前类作为配置类，代替xml配置文件。

之前我们在配置文件中**开启组件扫描配置**，这里我们同样可以用注解方式写扫描，即加入注解`@ComponentScan(basePackages = {"com.atguigu")`

属性`basePackages`实际上是一种注解形式

```java
@Configuration //"配置“，定义它是配置类，代替xml文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
}
```

这种写法在**测试方法中**和之前**有点不一样**：

不是xml配置了，所以应该改为**加载配置类**：

```java
ApplicationContext context
                = new AnnotationConfigApplicationContext(SpringConfig.class);
```

Annotation是”注解“的意思

后面的步骤获得对象和输出是一样的。

实际中我们一般用SpringBoot，本质上是Spring，只是做了高度集成。
