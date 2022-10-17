# AOP

## 概念

1. 什么是AOP？
   
   它是“**面向切面编程**”的缩写，或者说“面向方面编程”
   
   软件里有各个功能（业务逻辑），把它们分离开来，可以降低耦合度，提高程序的可重用性。

举个例子：

我们想做一个最基本的**登录功能**，做好后我想在登录功能的基础上添加（完善）功能：权限判断（管理员用户、普通用户..）

1. 原视方式：修改源代码实现

2. 现在：**不通过修改源代码方式**，在原有基础之上添加新的功能。（<mark>美国宪法</mark>）
   
   可以把独立出一个权限判断新模块，给它配置到主干里区。

这样大大降低了耦合度，某天不用到这个判断，只需要去掉这个模块。

## 底层原理

AOP是怎么实现的呢？

1. AOP底层使用**动态代理**的方式实现
   
   有两种情况的动态代理：
   
   1. **有接口**情况
      
      使用<mark>**JDK动态代理**</mark>
      
      - 🔺<mark>**创建接口实现类代理对象**</mark>，增强类的方法。
   
   2. **没有接口**情况
      
      使用<mark>**CGLIB动态代理**</mark>
      
      - <mark>**创建当前类子类代理对象**</mark>，同样不是new出来的。

通过代理对象，把要增加的功能做到。

### 动态代理：

#### 有接口情况

有接口就有实现类，现在想要在登录基础之上加个新功能（判断），这就叫**增强一个类中的方法**

如何用AOP方式实现呢？这就需要JDK动态代理实现。

需要**再创建一个接口实现类的代理对象**，但不是new出来的。而是代理对象。

<mark>有它之前的功能</mark>，但与new出来对象是不一样的，他是代理对象。

#### 没有接口情况

比如我想增强User类中的方法，以前我们是创建User类的子类，继承父类。（原视方式）但现在我们要通过动态代理做到：CGLIB动态代理。

**要创建当前类子类的代理对象**，同样不是new出来的。

通过代码，写一下JDK动态代理代码编写，加深印象。Spring5已经帮我们封装好了。

### 使用JDK动态代理

使用Proxy类里面的方法，来创建出代理对象。

[Java Platform SE 8英文版](https://docs.oracle.com/javase/8/docs/api/)

[Java 8 中文版](https://www.matools.com/api/java8)

java.lang(工具类).reflect包下有类叫Proxy类，其提供的方法能帮我们创建出代理对象。

<img title="" src="file:///C:/Users/up/AppData/Roaming/marktext/images/2022-10-15-10-57-27-image.png" alt="" width="661" data-align="inline">

返回**指定接口的代理类**的实例（对象）。

`newProxyInstance`方法，里面有三个参数：

1. `ClassLoader`**类加载器**

2. 我们针对的是有接口情况，所以我们写接口的class。
   
   即<u>增强方法所在的类的</u>**这个类实现的接口**；支持多个接口，所以返回Class类型的数组形式。

3. 最重要`InvocationHandler`写增强的部分：它单独是一个接口！
   
   <mark>**实现这个接口，创建代理对象，写增强的方法**</mark>

但这里返回的是`InvocationHandler h`实例对象（即我们创建的代理对象），返回指定接口的代理类的实例。（两种方法：1.直接创建匿名内部类，生成对象；2.创建一个`InvocationHandler`接口的实现类，new它的实例对象。

现在我们来写一下吧~

基本背景：

1. 创建接口，定义方法。
   
   别忘了接口里的方法都是抽象方法，没有方法体的。

2. 创建接口的实现类，实现方法

下面重点：增强功能

3. 使用**Proxy类创建接口代理对象**

#### 增强接口有多种写法：

1. 写个匿名内部类，直接new

```java
public class JDKProxy {
    public static void main(String[] args) {
        //用newProxyInstance方法创建接口实现类代理对象
        //1.类加载器；2.要实现的接口（需要数组类型）
        //3.增强接口
        Class[] interfaces = {UserDao.class};
        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return null;
            }
        });
    }
}
```

2. 单独写个类来实现接口

```java
public class JDKProxy {
    public static void main(String[] args) {
        //用newProxyInstance方法创建接口实现类代理对象
        //1.类加载器；2.要实现的接口（需要数组类型）
        //3.增强接口
        Class[] interfaces = {UserDao.class};
        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),interfaces,new UserDaoProxy());
    }


}
//创建代理对象代码
//实现类实现接口
class UserDaoProxy implements InvocationHandler{
    //实现接口里的方法
    //增强功能的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
}
```

现在我们想要增强的是UserDaoImpl类的方法，所以要创建它的代理对象，因此要把对象传到UserDaoProxy里来。

相当于副本。

```java
class UserDaoProxy implements InvocationHandler{
    //1 把创建的是谁的代理对象，把谁传递过来
    //有参数构造传递
    private Object obj;
    public UserDaoProxy(Object obj){
        this.obj = obj;
    }
```

这里为了通用用了Object类型，当然这个实例我们可以直接用UserDaoImpl类

```java
//创建代理对象代码
//实现类实现接口
class UserDaoProxy implements InvocationHandler{
    //1 把创建的是谁的代理对象，把谁传递过来
    //有参数构造传递
    private Object obj;
    public UserDaoProxy(Object obj){
        this.obj = obj;
    }


    //实现接口里的方法
    //增强功能的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //方法前执行
        System.out.println("方法之前执行..."+method.getName()+"：传递的参数"+ Arrays.toString(args));

        //被增强的方法执行(对象，参数）
        //对象是obj，因为我们已经将它传过来了
        Object res = method.invoke(obj,args);

        //方法之后
        System.out.println("方法之后执行.."+obj);
        return res;
    }
}
```

invoke有“提及，唤起”之意。

此时之前new的UserDaoProxy类对象报红，因为我们上面为它增加了有参构造方法，传入了需要增强功能类的对象。

这样的话，我们修改一下：

```java
public class JDKProxy {
    public static void main(String[] args) {
        //用newProxyInstance方法创建接口实现类代理对象
        //1.类加载器；2.要实现的接口（需要数组类型）
        //3.增强接口
        Class[] interfaces = {UserDao.class};
        UserDaoImpl userDao = new UserDaoImpl();
        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),interfaces,new UserDaoProxy(userDao));
    }
}
```

上面`Proxy.newProxyInstance`方法会返回我们的代理对象。这里有一个快捷键小技巧：后面加`.var`**帮助生成属性（成员对象）。**

这里做一个强转`UserDao`，因为`UserDaoImpl`实现类实现接口是它。<mark>**强转体现多态**</mark>。

```java
UserDao dao =(UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));.
```

输出一下：

```java
public class JDKProxy {
    public static void main(String[] args) {
        //用newProxyInstance方法创建接口实现类代理对象
        //1.类加载器；2.要实现的接口（需要数组类型）
        //3.增强接口
        Class[] interfaces = {UserDao.class};
        UserDaoImpl userDao = new UserDaoImpl();
        UserDao dao =(UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
        int result = dao.add(1, 2);
        System.out.println("result" + result);

    }
```

得到结果：

```java
方法之前执行...add：传递的参数[1, 2]
add方法执行了
方法之后执行..com.atguigu.spring5.UserDaoImpl@2b193f2d
result3
```

总结一下JDK手动底层实现动态代理：

#### 总结

1. 写一个UserDao接口及它的实现类UserDaoImpl

2. 实现类UserDaoImpl中实现两个基础方法（add增加，update更新）

3. 使用JDK动态代理方式
   
   1. 创建一个JDKProxy类模拟JDK动态代理，里面包含main方法
   
   2. 使用newProxyInstance创建接口实现类的代理对象
   
   3. newProxyInstance里传入参数有三个：
      
      - 类加载器：
        
        `JDKProxy.class.getClassLoader()`
      
      - 要实现的接口（可能有很多，所以定义为数组形式，返回类型Class)
        
        本案例中只有一个，就是UserDao接口
      
      - `InvocationHandler`接口返回实现类的实例对象
      
      进一步：
      
      创建其实现类，该实现类通过有参构造函数，传递需要增强方法的类的实例对象。
      
      在这个里面再写增强方法（即创建副本，在这基础上修改副本），我理解为两层嵌套，复制包裹最初的方法。
   
   4. 写好后，调用newProxyInstance创建的所谓的”代理对象“，这一步就是说这个代理对象可以用啦！

## 相关术语

### 1.连接点

举例：创建一个User类，里面有四个方法，实现增删改查功能。

类里面的哪些方法可以被增强，那这些方法就被成为连接点。

### 2.切入点

**实际真正<mark>被增强</mark>的方法**。四个方法理论上都可以被增强（连接点），但实际上我只增强”增方法“，那么只有”增方法“被成为切入点。

### 3.通知（增强）

1. 实际增强的逻辑的部分被成为通知（增强）
   
   即**实际新增的功能**。

2. 通知有很多种类型：
   
   - **前置@Before**通知
     
     add方法（示例中的切入点）之前执行
   
   - **后置（返回）@AfterReturning**通知
     
     add方法之后执行
     
     返回结果之后执行。
   
   - **环绕@Around**通知
     
     方法前后都执行
     
     里面有一个参数：`ProceedingJoinPoint proceedingJoinPoint`
     
     调里面的方法：（执行被增强方法）
     
     `proceedingJoinPoint.proceed()`
   
   - **异常@AfterThrowing**通知
     
     add方法出现异常之后才会执行，未发生不执行。
     
     但异常未处理，所以若有@AfterReturning也不会执行了。
   
   - **最终@After**通知
     
     类似finally，不管怎样都会执行。有异常也会执行。
     
     After..
     
     AfterThrowing..

### 4.切面

是一个动作。

1. <mark>**把通知应用到切入点的过程**</mark>
   
   把权限判断加入到登录方法**的过程**

## 准备工作

Spring中有多种方式可以做到AOP实现。

Spring框架中一般<mark>**基于AspectJ**</mark>实现针对AOP相关的操作

1. 什么是AspectJ
   
   它并不是Spring组成部分，而是独立的AOP框架。
   
   即不需要Spring也能单独使用。
   
   一般把AspectJ和Spring框架一起使用，进行AOP操作。
   
   这样在开发中更方便。

2. 基于它主要是有两种方式实现AOP操作：
   
   1. 基于xml配置文件实现（少用）
   
   2. 基于注解方式实现（一般经常使用，更方便）

3. 在项目的工程里面引入AOP相关依赖。
   
   之前已经引入了：
   
   ![](C:\Users\up\AppData\Roaming\marktext\images\2022-10-16-13-12-41-image.png)

现在我们还要在之前下好的Spring框架：

`D:\Program Files\spring-5.2.6.RELEASE-dist\`中引入：

`spring-aspects-5.2.6.RELEASE.jar`

我们之前提到AspectJ本身并不是Spring中一部分，所以还需另外引入它的依赖。

![](C:\Users\up\AppData\Roaming\marktext\images\2022-10-16-15-01-43-image.png)

接下来配置中需要用到一个东西：切入点的表达式

### 切入点表达式

1. 切入点表达式的作用：
   
   知道我们要对哪个类里面的哪个方法进行增强

2. 语法结构：
   
   `execution([访问权限修饰符][返回类型][全类名][方法名称]([参数列表])`

”执行，实施“

举例一：对com.atguigu.dao.BookDao类里面的add方法进行增强

`execution(public com.atguigu.dao.BookDao.add(..))`

**<mark>修饰符可以省略不写，返回类型可以用*</mark>**

举例二：相对类里所有方法进行增强

`execution(public com.atguigu.dao.BookDao.*(..)`

举例三：对com.atguigu.dao包里所有类，类里面所有方法进行增强

`execution(public com.atguigu.dao.*.*(..)`

这上面三个示例是实际中比较常用的例子。

## AspectJ注解方式

1. 创建**被增强类**，在类里面创建方法

2. 创建**增强类**（编写增强的逻辑）
   
   创建方法，让不同方法代表不同的**通知类型**（执行前呀，执行后呀..）

3. 进行通知的配置
   
   1. 在spring中的配置文件中，**开启注解的扫描**
      
      可以写配置类（完全注解开发），也可以写配置文件
      
      我们这次用后者
   
   2. 使用注解**创建User类和UserProxy类的实例对象**
   
   3. 在**增强的类上面添加注解**`@Aspect`
      
      它就表示让这个类生成代理对象
   
   4. 在spring配置文件中写一段配置，**开启生成代理对象**

我们需要用到相关的名称空间：context。添加一下

![](C:\Users\up\AppData\Roaming\marktext\images\2022-10-16-15-43-30-image.png)

**除此之外，为了我们能开启生成代理对象，我们还要加个名称空间：叫aop**

component"组成部分"

我们想做个前置通知的效果:所以第四步

4. 配置不同类型对象通知
   
   1. 在增强类的里面，在作为通知的方法上面
      
      **添加想要添加想要通知类型注解**
      
      并且**使用切入点表达式来配置**
   
   2. 前置通知用到是`@Before(切入点表达式)`注解
      
      value可以省略

增强类：

```java
//增强的类
@Component
//生成代理对象
@Aspect
public class UserProxy {

    @Before(value="execution(* com.atguigu.spring5.aopanno.User.add())")
    public void before(){
        System.out.println("之前执行");
    }
}
```

被增强类：

```java
//被增强类
@Component
public class User {
    public void add(){
        System.out.println("add..");
    }
}
```

**<mark>配置文件</mark>**：

- **开启注解扫描**

- **生成代理对象**
  
  （主要代码，不包含名称空间的引入）

```xml
 <!--开启注解方式的【组件扫描】-->
    <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>
    <!--开启Aspect生成代理对象-->
    <!--找有没有@Aspect，找到，就生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

测试一下效果：

```java
    @Test
    public void testAopAnno(){
        //加载配置文件，获取对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        //这里获取User类实例对象
        User user = context.getBean("user", User.class);
        user.add();

    }
```

成功返回结果：

```java
之前执行
add..
```

**注意：我们这里明明调用的是User类实例对象而不是代理对象，怎么会调用了它更改后的”副本“呢？**

返回的还是User类，但这个类中间被代理过了！

下面说一些细节问题：

### 提取相同的切入点

把相同的切入点进行抽取。

写一个方法pointdemo()

在方法上<mark>**用@Pointcut(把相同切入点放进来)注解**</mark>

以后就可以在不同注解类型里写一样的东西（即封装工具）

示例：**@Before(value="porintdemo()")**

pointdemo()是我们定义的方法名，该方法使用@Pointcut提取切入点注解。

### 增强类设置优先级

对一个方法有多个增强类。

即多个增强类对同一个方法进行增强，

我们可以**设置它们的优先级**。**谁先执行，谁后执行**。

1. **在增强类上<mark>添加注解@Order(数字类型)</mark>**
   
   **<mark>数字越小，优先级越高</mark>**

## AspectJ配置文件方式

一般很少用到，这部分仅作了解。

1. 创建两个类，增强类和被增强类，创建方法。

2. 在spring配置文件中创建两个类对象

3. **在spring配置文件中配置切入点。**
   
   即对内个类做增强（Aop增强）
   
   标签`<aop:config>`里：
   
   1. 切入点`<aop:pointcut id="p" expression"切入点表达式"/>`
   
   2. 配置切面：把before方法配置到类之前执行:
      
      `<aop:aspect ref="bookProxy">`
      
      - 增强作用在具体的方法上
        
        `<aop:before method="现在增强的方法" point-ref="p"/>`把新增增强方法作用到这个p需被增强方法（切入点，即实际被增强的方法）通知（实际被增强功能）（名词）。

## 完全注解方式

**不需要写配置文件，完全使用注解开发**

创建配置类，不需要创建xml文件

```java
@Configuration
//开启组件扫描
@ComponentScan(basePackages = {"com.atguigu"})
//👇生成代理对象,不写默认是false
//等同于在xml中配置aspectj-autoproxy,表示开启对注解AOP的支持
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```

# 注：CGLIB网课漏讲了
