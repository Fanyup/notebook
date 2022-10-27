# Spring事务管理操作

## 概念

1. 什么是事务？（回顾一下）
   
   数据库操作最基本单元，逻辑上是一组操作，要么都成功，要么都失败。
   
   经典场景：银行转钱

2. 事务四大特性（ACID）：
   
   1. 原子性
      
      过程不可分割，要么都成功，要么都失败。
   
   2. 一致性
      
      转钱总数不变
   
   3. 隔离性
      
      多事务操作是互相间不会产生影响。
   
   4. 持久性
      
      事务提交，表中数据会发生真正变化。

## 搭建事务操作环境

示例：银行转账（少钱、多钱操作）

1. **WEB层**：视图部分相关操作

2. **Service层**（业务逻辑层）：业务操作
   
   转账的方法：1.调用dao类的两个方法

3. **Dao层**：对数据库操作的相关方法，一般不写业务操作。
   
   创建两个方法：1.少钱的方法；2.多钱的方法。

下面来开始具体实现吧~

1. 创建数据库表，添加记录。
   
   我在mydb库下创建了t_account表，注意我这里id类型是id，网课老师用了varchar。这里我们以一共添加了三条属性：id,username和money。
   
   接着手动添加两条记录：lucy和mary各有1000块钱。

2. 在项目中创建service,dao类，完成对象创建和注入关系。
   
   service注入dao，dao注入JT对象（模板），JT对象中注入连接池信息（DataSource)

3. 在dao实现类里创建两个方法：多钱和少钱方法。
   
   ```java
       //多钱
       @Override
       public void addMoney() {
           String sql = "update t_account set money=money+? where username=?";
           jdbcTemplate.update(sql,100,"mary");
   
       }
   
       //少钱
       //Lucy转账100给mary
       @Override
       public void reduceMoney() {
           String sql = "update t_account set money=money-? where username=?";
           jdbcTemplate.update(sql,100,"lucy");
       }
   ```
   
   在service类里创建转账方法。
   
   ```java
       //转账的方法
       public void accountMoney(){
           //lucy少100
           userDao.reduceMoney();
           //mary多100
           userDao.addMoney();
       }
   ```

到目前位置，该示例环境（银行转账的背景）就完成了。

测试类测试一下：

```java
 @Test
    public void testAccount(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.accountMoney();
    }
```

结果：数据库成功-100和+100。

4. 上面代码有问题：中间加个异常（int i = 10/0），发现不一致了。说明使用事务非常有必要！

事务操作基本过程：（原始方法）

1. 开启事务操作

2. 进行业务操作（try..catch捕获异常）
   
   - 出现异常，事务回滚
   
   - 正常执行，提交事务

但我们现在**用spring框架实现事务操作更方便**！基本流程时不变的。

## Spring事务管理

1. <mark>**一般建议把事务加到Service层（业务逻辑层）中**</mark>

2. 在Spring进行事务管理操作有两种方式：
   
   1. 编程式事务管理（很不方便，代码臃肿，一般不用）
   
   2. **声明式事务管理**（一般常用）（通过配置把它做到）

<mark>**底层使用AOP**</mark>

（面向切面)(不改变源代码情况下增强类中的方法)

<mark>**Spring事务管理API**</mark>（相关接口和实现类）

1. 提供一个接口，**代表事务管理器**。
   
   对事务操作进行了封装。
   
   这个接口针对不同框架，提供了不同的实现类。
   
   接口`PlatformTransactionManager`
   
   我们快捷键`Ctrl+H`看一下它的结构
   
   注：transaction“事务”

可以看到针对不同框架提供了不同的事务实现类。

我们这里用到的是Mysql的jdbc。所以用到如图所示：

![ApplicationFrameHost_adx94OLZuc.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_adx94OLZuc.png)

（Mybatis也用到是它）

### 声明式事务管理

1. **<mark>基于注解方式实现（常用）</mark>**

2. 基于xml配置文件方式实现

下面演示注解声明式事务管理

1. 在spring配置文件中**配置事务管理器**。

点开实现类源码发现其有一个set方法，说明需要注入数据源。注入方式和之前一样，其实就是连接池信息。

```xml
    <!--创建事务管理器-->
    <!--其实就是创建它的实现类的对象-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源，方法跟之前一样-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

2. 在spring配置文件中，**<mark>开启事务注解</mark>**
   
   老样子，<mark>**引入名词空间tx**</mark>，专门用于处理事务的。
   
   用一个标签开启事务注解。<mark>指定使用哪个事务管理器来开启注解</mark>。（记得这个不是写在上面里面的，而是另开一行，写在外面）
   
   `<tx:annotation-driven..`
   
   ```xml
     <!--开启事务注解-->
       <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
   ```

3. 在service类上面（或里面的类方法上面）**<mark>添加事务注解</mark>**
   
   <mark>**@Transactional**</mark>可以加到类上，也可以加到方法上面。
   
   如果在类上：表示这个类里的所有方法都添加事务
   
   如果在方法：为这个方法添加事务。
   
   一般加到类上，方便。
   
   测试后，发现现在即使发现异常，也不会改变值。<u>体现了事务的回滚</u>。

#### 参数配置

刚才完成了声明式事务管理，这里面有一些参数配置。现在来讲一下。

##### 1.propagation传播行为

**<mark>事务传播行为</mark>；“传播”**

现在有事务A和事务B，当A的事务方法被B事务方法调用时，事务该如何进行处理。

**传播行为可由传播属性指定**。

###### 传播属性

事务A中有方法a，事务B中有方法b

1. **REQUIRED**
   
   方法2开启事务A，调用事务B的方法b
   
   <u>方法b加入到当前事务A里面</u>
   
   也就是说，不会开启事务B而是把方法b加入到事务A中。
   
   出错，一起回滚

2. **REQUIRED_NEW**
   
   A调B，A叫外层事务，B叫内层事务。他俩之间互不影响。
   
   A调方法b会开启事务B，两个事务间互不影响
   
   谁出错谁回滚。

3. **SUPPORTS**
   
   支持在事务中运行，当然也可以在非事务中运行。

##### 2.isolation隔离级别

事务隔离级别；“隔离”

多事务操作之间不会产生影响

###### 三个读问题

并发时会有三个读问题：**脏读、不可重复读、幻（虚）读**

1. 脏读（致命问题）
   
   一个**未提交事务**读取到**另一个未提交事务**的数据。
   
   小王和小李分别开启事务A和B，小王和小李两人同时改了同一条数据。
   
   小李读了小王正在修改的数据。
   
   如果此时小王事务回滚，
   
   小李的数据就不对了！

2. 不可重复读（现象，允许产生，不算是问题）
   
   一个**未提交事务**读取到**另一个已提交事务**的数据。
   
   小王和小李都获取到了源数据。
   
   小王又修改了数据，修改后马上提交了事务。——表中数据发生变化。
   
   小李未提交，但他此时却读取到了小王修改后的数据！

3. 幻读
   
   一个**未提交事务**读取到**另一个已提交事务**的**添加数据**。

通过<u>设置事务隔离级别</u>可以解决这三个读问题。

###### 隔离级别

| 隔离级别                   | 脏读  | 幻读  | 不可重复读 |
| ---------------------- | --- | --- | ----- |
| READ UNCOMMITTED（读未提交） | T   | T   | T     |
| READ COMMITTED（读已提交）   | F   | T   | T     |
| REPEATABLE READ（可重复读）  | F   | F   | T     |
| SERIALIZABLE(串行化)      | F   | F   | F     |

可以这样写：

###### 设置演示

```java
@Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.READ_COMMITTED)
```

**<mark>mysql默认可重复读</mark>**。

#### 3.timeout超时时间

1. 设置在一定事件内进行事务提交，超时回滚。

2. **<mark>spring里默认值是-1(即不超时)</mark>**。
   
   以**秒为单位**。

#### 4.readOnly是否只读

1. 读：查询操作
   
   写：添加修改删除操作

2. **readOnly默认值：false;**
   
   读写都可以。

3. 设置成true后，只能查询了，即只读。

#### 5.rollbackFor回滚

1. 设置出些哪些异常进行事务回滚

#### 6.noRollbackFor不回滚

5的反义。

### XML声明式事务管理

1. 配置事务管理器（创建对象）
   
   同之前一样。
   
   ```xml
       <!--创建事务管理器-->
       <!--其实就是创建它的实现类的对象-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <!--注入数据源，方法跟之前一样-->
           <property name="dataSource" ref="dataSource"></property>
       </bean>
   ```

2. 配置通知（增强的具体部分：事务）
   
   attribute"属性"
   
   ```xml
       <!--配置通知-->
       <tx:advice id="txadvice">
           <!--配置事务相关参数-->
           <tx:attributes>
               <!--指定哪种规则的方法上添加事务-->
               <tx:method name="accountMoney" propagation="REQUIRED"/>
               <tx:method name="account*"></tx:method>
           </tx:attributes>
       </tx:advice>
   ```

3. 配置切入点和切面（加入到哪个类/方法上；把事务加到方法的过程）
   
   ```xml
       <!--配置切入点和切面-->
       <aop:config>
           <!--配置切入点（切入点表达式）-->
           <!--切入点名字随便起-->
           <!--UserService类中所有方法*.(..)-->
           <aop:pointcut id="pt" expression="execution(* com.atguigu.spring5.svice.UserService.*(..))"/>
           <!--配置切面-->
           <!--将【事务】的通知，设置到【切入点】上-->
           <aop:advisor advice-ref="txadvice" pointcut-ref="pt"></aop:advisor>
       </aop:config>
   ```

用这种方式就**很好体现出来了事务管理底层是AOP**。

### 完全注解声明式事务管理

之前说过，要这么实现，需要创建配置类，代替xml配置文件。

```java
@Configuration//配置类
//组件扫描
@ComponentScan(basePackages = {"com.atguigu"})
//开启事务
@EnableTransactionManagement
public class TxConfig {
    //创建数据库的连接池（注入连接池属性）
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        //set方法注入
        dataSource.setDriverClassName("com.alibaba.druid.pool.DruidDataSource");
        //其他三个一样，就不写了
        //返回对象
        return dataSource;
    }
    //创建JT对象
    @Bean
    //里面参数意思：根据类型，在Ioc容器中找到对象，把它注入
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    //最后，创建事务管理器（里面开启事务）
    //同样的，形参那传入entity实体对象
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;

    }
}
```

c测试一下：（需要修改的地方）

（加载配置类）

```java
ApplicationContext context = new AnnotationConfigApplicationContext(TxConfig.class);
```

# Spring5新功能没看
