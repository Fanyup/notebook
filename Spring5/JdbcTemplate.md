# JdbcTemplate

## 概念和准备工作

1. 什么是JdbcTemplate
   
   Spring框架对JDBC进行封装，使用JdbcTemplate方便实现对数据库操作。

准备工作：

1. 引入相关jar包
   
   `mysql-connector-java-版本号.jar`和`druid-1.1.9.jar`连接池（这个之前引入过了）这里我引入了`8.0.27`版本
   
   还需引入spring里的依赖
   
   1. `spring-jdbc-版本号.jar`
   
   2. `spring-tx-版本号.jar`（针对事务的相关操作依赖）
   
   3. `spring-orm-版本号.jar`如果spring整合其他框架（Mybatis等其他时），这里用Mysql，所以可加可不加。（？）

2. 在spring配置文件中**配置数据库的连接池**（可用下面模板）
   
   ```xml
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
           <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
           <property name="url" value="jdbc:mysql://localhost:3306/mydb?characterEncoding=utf8&useUnicode=true&useSSL=false&autoReconnect=true"/>
           <property name="username" value="root" />
           <property name="password" value="root" />
       </bean>
       <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
           <property name="dataSource" ref="dataSource" />
       </bean>
   ```

注意：&报错，修改方式：将其转义。

改为`&amp;`

3. 配置JdbcTemplate对象，注入DataSource对象。即创建它的bean的示例（**上面代码已经写好了**）

注入DataSource源信息（父类有**对象类型属性**叫DataSource，其自带set方法，可用它来注入，而不必用有参构造方法注入。）

ref指的是我们创建类属性对象，添加引用上面创建的bean连接池（**它就是数据源信息**）。

简单来讲就是**把数据源信息注入属性对象中**。

4. 创建service类，dao类
   
   service类里注入dao类
   
   dao类里注入JdbcTemplate对象。
   
   用之前学的注解方式做到~
   
   （开启组件扫描

xml文件中加了：

```xml
 <!--开启组件扫描-->
    <context:component-scan base-package="com.atguigu"></context:component-scan>
```

Service类：

```java
@Service
public class BookService {
    //注入dao类对象
    //体现多态
    @Autowired
    private BookDao bookDao;
}
```

Dao类：

```java
public class BookDaoImpl implements BookDao{

    //注入JdbcTemplate对象（这个对象是在配置文件中创建）
    //按照类型注入哦~
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

再回顾一次：无需再像之前bean配置那样对象类型属性注入时在Service类中创建它的对象的同时**写它的set方法**。因为注解已经帮我们封装好了！

## 添加功能

首先来清晰一下上面的思路：

- BookDaoImpl实现类传入了JdbcTemplate对象。（简称JT对象）
  
  使用改对象可以调用对象的**添加update()方法**。

- 我们在bean配置文件中进行了利用注解方式注入了服务类和实现类的对象。并创建了JT对象，将连接池信息（我理解为连接信息）通过`<property..>`的方式传到了JT对象里。

接下来的步骤有：

- 创建一个**Book实例类**（entity"实体"），并创建它的set和get方法。来**模拟从页面获取数据**。
  
  get方法就是return得到，set方法就是设置填入。
  
  ```java
  public class Book {
      private String userId;
      private String username;
      private String ustatus;
  
      public String getUserId() {
          return userId;
      }
  
      public String getUsername() {
          return username;
      }
  
      public String getUstatus() {
          return ustatus;
      }
  
      public void setUserId(String userId) {
          this.userId = userId;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      public void setUstatus(String ustatus) {
          this.ustatus = ustatus;
      }
  }
  ```

- 数据库创建一个表名为t_book的表，表里有三个参数：id值，姓名，和状态。对应上面实体获取的三个参数。

- 在实现类中加一个add添加方法。这个方法的具体实现是调用JT对象的update方法（已经给我们封装好了）
  
  调用该方法需要传入实体对象哦。
  
  ```java
  public class BookDaoImpl implements BookDao{
  
      //注入JdbcTemplate对象（这个对象是在配置文件中创建）
      //按照类型注入哦~
      @Autowired
      private JdbcTemplate jdbcTemplate;
  
      //添加方法的实现
      @Override
      public void add(Book book) {
          //创建sql语句
          String sql = "insert into t_book values(?,?,?)";
          //调用方法实现
          Object[] args = {book.getUserId(), book.getUsername(), book.getUstatus()};
          int update = jdbcTemplate.update(sql, args);
          System.out.println(update);
      }
  }
  ```
  
  相应的，接口也要修改一下。

- 服务类中调用实现类创建的方法（相当于调用实现类封装好的东西，这样可以做到解耦合）
  
  因为之前引入了bookDao对象（接口类型体现多态），那我们这里可以加一个方法：
  
  接收book实体示例，然后调用bookDao封装好的add方法。
  
  ```java
  
  ```

## 修改和删除功能

## 查询功能

## 批量添加功能

## 批量修改和删除功能
