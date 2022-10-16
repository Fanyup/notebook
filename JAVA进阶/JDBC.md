# JDBC

IDEA中`Shift+Shift`非常强大，搜索。

## 定义

是SUN公司指定的一套接口（interface)

    java.sql.*;（这个软件包下有很多接口）

面向接口调用、写实现类都属于面向接口编程

- **为什么要面向接口编程？**
  
  <mark>**解耦合**</mark>：降低程序的耦合度，<mark>提高程序的扩展力</mark>
  
  多态机制就是非常典型的：面向抽象编程（不要面向具体的编程）：
  
  - **父类型引用指向子类型对象**
    
    ```java
     Animal a = new Cat()；//建议
     Animal a = new Dog()；//建议
    
       //喂养的方法
    
    public void feed(Animal a){
    
     }//面向抽象（父类型、接口）编程
    
        不建议：
    
     Dog d = new Dog();
     Cat c = new Cat();
    ```

组件与组件间要扩展得有个接口，这个接口就是一种抽象的协议：螺栓螺母之间 3mm,2mm（这就是抽象的接口）

**调用者（SUN公司）和实现者（Oracle..）都得遵守。**

为什么SUN定制一套JDBC接口呢？

    因为每个数据库底层的实现原理都不一样

    Oracle数据库有自己的原理（他们写的JDBC实现类得去官网下载）**（专业术语名词：驱动/就是一个jar包）**

    MySQL数据库也有自己的原理

    MS SqlServer数据库也有自己的原理...

接口的实现方是数据库厂家实现的，接口的实现类不用我们写，我们只需要调用这个接口。

- 驱动：所有的数据库驱动都是以jar包的形式存在，
  
  这些jar包中有很多.class文件，这些.class文件就是对JDBC的实现
  
  驱动不是SUN公司提供的，是各大数据库厂家负责提供，下载驱动jar包要去数据库官网加载。
  
  **<mark>只有接口，没有这个接口的实现类，程序是永远无法运行的！！！</mark>**

JDBC编程六部（需要背会）

1. **注册驱动**
   
   告诉Java程序，即将要连接的是哪个品牌的数据库

2. **获取连接**
   
   表示JVM的进程和数据库进程之间的通道打开了
   
   属于进程之间的通信（重量级的）使用完之后一定要关闭通道

3. **获取数据库操作对象（专门执行sql语句的对象）**

4. **执行SQL语句**
   
   DQL,DML..

5. **处理查询结果集**
   
   只有当第四步执行的是select语句，才有第五步结果集（若4执行DML则直接到第六步）

6. **释放资源**
   
   使用完资源后一定要关闭资源

## 什么是通信协议？有什么用

通信协议是通信之前就提前定好的**数据传送格式**

数据包具体怎么传数据，格式提前定好的

## 流程演示

```java
import java.sql.Driver;//这是一个接口
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Connection;

public class JDBCTest01{
    public static void main(String[] args){
    try{
        //1、注册驱动
        //多态，父类型引用指向子类型对象
        Driver driver = new com.mysql.jdbc.Driver();
        DriverManager.registerDriver(driver);
        //MYSQL厂家实现类Driver.class

        //2、获取连接
        //url：统一资源定位符（网络中某个资源的绝对路径）
        //URL包括哪几部分？（协议、IP\PORT端口、资源名）
        //http://通信协议
        //182.61.200.7 服务器IP地址（计算机的代号）
        //80 服务器上软件的端口（计算机上某个软件的代号）
        //index.html是服务器上某个资源名
        String url = "jdbc:mysql://127.0.0.1:3306/bjpowernode";
        //jdbc:mysql://协议
        //哪个机器（IP地址）？127.0.0.1
        //数据库端口号？3306
        //具体数据库是零秒？bjpowernode

        //localhost和127.0.0.1都是本机IP地址
        String user = "root";
        String password = "333";
        //多态
        Connection conn = DriverManager.getConnnection(url,user,password);
        //调的方法永远都是这个接口（Connection）里的方法————面向接口写代码
    }catch(SQLException e){
        e.printStackTrace();
    }    
    }

}
```

  

```java
//3、获取数据库操作对象（Statement对象，专门执行SQL语句的）
        //我们已经拿到Connection接口，可以调用它的相关方法
            Statement stmt  = conn.createStatement();
```

```java
//4、执行sql
        String sql = "insert into dept(deptno,dname,loc)values(50,'人事部','北京')";//执行个插入
        int count = stmt.executeUpdate(sql);//专门执行DML语句(insert,delete,update)
        //返回值是“影响数据库中的记录条数”
```

6. 释放资源

```java
Statement stmt = null;
        Connection conn = null;
    try{
```

```java
finally{
        //6、释放资源
        //遵循从小到大一次关闭
        //为了保证资源一定释放（比方说先stmt再conn）（把它挪到try外边）
        //分别对其try..catch，不能一起try哦！
    }
    }
```

先搭建总框架

jdbc的sql语句不需要写分号

### 注册驱动的方法2（常用）

Driver.class类里有个静态代码块，自带注册驱动，不用我们写了
要执行静态代码块，就要加载这个类
如何加载类？————反射机制

```java
Class.forName("com.mysql.jdbc.Driver");//类加载。这里无需接收返回值
```

为啥常用？因为参数是一个字符串，字符串可以写到配置文件（如xxx.properties)当中

从属性资源文件中读取数据库信息

将连接数据库的所有信息配置到配置文件中

```java
//使用资源绑定器绑定配置文件（IO+Properties）（util包下方法）
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");//jdbc.properties的后缀不用写
        String driver = bundle.getString("driver");//调用Bundle相关方法；
        String driver = bundle.getString("url");
        String driver = bundle.getString("user");
        String driver = bundle.getString("password");
        //跟它的Key值，直接从配置文件拷贝OK的👆
```

实际开发中不建议把数据库的信息写死到java程序中。

### 处理查询结果集

```java
        Connection conn = null;
        Statement stmt = null;
        //新增一个接口（结果集）
        Resultset rs = null;//查询之后的结果，自动封装到这里面了
        //关闭顺序，由下到上
```

```java
//执行sql语句
String sql = "select no,ename,sal from emp";
rs = stmt.executeQuery(sql);
//专门DQL语句的方法
```

返回Int                **<mark>executeUpdate</mark>(insert/delete/update)**

返回ResultSet(接口，里面有很多方法)  

                            **<mark>executueQuery</mark>(select)**

遍历取值：

.next()方法，光标处若有数据，则返回true

```java
//遍历结果集
boolean flag1 = re.next();
if(flag1){
    //光标指向的行有数据
    //取数据，怎么取？通过结果集一个get方法
    //getString()方法，特点：不管数据库中数据类型是什么
    //都以String类型取出
    String no = rs.getString(1);//这个是下标（第几列）
    String ename = rs.getString(2);//第二列
    String sal = rs.getString(3);
    //这是取一行的代码

    //注意！！！JDBC中所有下标从1开始！
}
```

**<mark>注意！！！JDBC中所有下标从1开始！</mark>**

```java
while(rs.next()){
    //循环遍历
    String no = rs.getString(1);//这个是下标（第几列）
    String ename = rs.getString(2);//第二列
    String sal = rs.getString("sal");//还可以跟列名，更健壮
}
```

**建议不用下标，用列名，更健壮！**

重点注意：列名称不是表重点列名称，是查询结果集的列名称。

除了可以以String类型取出之外，还可以以特定类型取出。

如getInt()..不过适可而止哦！

## IDEA

1. 导包
   
   模块（文件夹）上右键-Open Module Settings-Libraries(库）)
   
   快捷键Alt+Ctrl+Shift+S
   
   打开Project Structures
   
   新建-Apply-OK

### PowerDesigner

设计阶段

我建立了课程需要的表格在mydb数据库下，表名叫t_user；

## 实践

意识往抽象方法上转，不要老想着main()方法

注意！：

    MySQL5用的驱动url是com.mysql.jdbc.Driver

    **MySQL6以后用的是<mark><u>com.mysql.cj.jdbc.Driver</u></mark>**

把变量拼到一个字符串（口诀）：

‘**<mark>“++”</mark>**’再把变量贴到++的中级

（有风险）

```java
package com.bjpowernode.jdbc;

import java.sql.*;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

/*
* 实现功能：
*    1.需求：模拟用户登录功能的实现
*    2. 提供一个输入的入口，输入用户名，密码
*    输入之后，提交信息，java程序手机到用户信息
*    java程序连接数据库验证用户名和密码是否合法
*    合法：登录成功；不合法：登陆失败
*    3.数据的准备：
*       在实际开发中，表的设计会使用专业的建模工具：PowerDesigner
*       来实现数据库表的设计（设计阶段）（参见user-login.sql脚本）
* 
* */
public class JDBCTest00 {
    public static void main(String[] args) {

        Map<String,String> userLoginInfo = initUI();
        //验证用户名和密码
        boolean loginSuccess = login(userLoginInfo);
        //最后输出结果
        System.out.println(loginSuccess ? "登陆成功" : "登陆失败");
    }


    /*
     *用户登录
     * @param userLoginInfo用户登录信息
     * @return false表示失败,true表示成功
     * */

    private static boolean login(Map<String, String> userLoginInfo) {
        //打标记的意识
        boolean loginSuccess = false;

        //JDBC代码（6）
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        try {
            //1.注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //2.获取连接
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC","root","999887723");
            //3.获取数据库操作对象
            stmt = conn.createStatement();
            //4.执行sql
            String sql = "select * from t_user where loginName = '"+userLoginInfo.get("loginName")
                    +"' and loginPwd = '"+userLoginInfo.get("loginPwd")+"'";
            rs = stmt.executeQuery(sql);//返回到结果集
            //5.处理结果集
            if(rs.next()){
                //登陆成功
                loginSuccess = true;
            }

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //6.释放资源
            if (rs !=null){
                //若结果集不等于空
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (stmt !=null){
                //若结果集不等于空
                try {
                    stmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn !=null){
                //若结果集不等于空
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }

        return loginSuccess;
    }

    //初始化一个界面
    /*
     * @return 用户输入的用户名和密码等登录信息
     * */
    private static Map<String, String> initUI() {
        Scanner s = new Scanner(System.in);

        System.out.println("用户名：");
        String loginName = s.nextLine();

        System.out.println("密码");
        String loginPwd = s.nextLine();
        //返回数据
        Map<String,String> userLoginInfo = new HashMap<>();
        userLoginInfo.put("loginName", loginName);
        userLoginInfo.put("loginPwd", loginPwd);
        return null;
    }

}
```

### 问题：出现SQL注入现象

黑客经常使用（安全隐患）

导致SQL注入的根本原因是什么？

    打断点-debug执行测试

    正好完成了sql语句的拼接，之后发送sql语句给DBMS，DBMS进行sql编译

    导致原SQL语句扭曲了

- 用户输入的信息中含有sql语句的关键字
  
  并且这些关键字参与了sql语句的编译过程
  
  导致sql语句的愿意被扭曲，进而达到sql注入

### 解决SQL注入问题

如何解决SQL注入问题？

Statement接口下有很多子接口：**PreparedStatement**(**预编译的**数据库造作对象)

只要用户提供的信息不参与SQL语句的编译过程，问题就解决了。

- 要想用户信息不参与SQL语句的编译，那么必须使用`java.sql.PreparedStatement`接口
  
  **预编译**的数据库操作对象
  
  原理：**预先对SQL语句的框架（给DBMS）进行编译**
  
              然后再给SQL语句传“值”
  
  修改部分：
  
  ```java
    //JDBC代码（6）
          Connection conn = null;
          PreparedStatement stmt = null;//这里使用预编译的数据库操作对象
          ResultSet rs = null;
  ```

```java
 //3.获取【预编译的】数据库操作对象
            String sql = "select * from t_user where loginName = ? and loginPwd = ?";
            stmt = conn.prepareStatement(sql);
                //给占位符传值
            stmt.setString(1,loginName);
            stmt.setString(2,loginPwd);
```

SQL语句的框子

要把传入值改成`？`哦！它是一个**占位符**，只能填充**值**。千万别写成`'？'`，不让它会认为你这个只是一个简单的问号字符串！！

一个`？`将来就是一个**值**

占位符？传值（第一个问号下标是1）

这里因为密码和用户名都是String类型，所以调用预编译对象方法`setString()`,那么占位符`？`将返回一个单引号`'我是内容`'的东西

```java
//4.执行sql
            rs = stmt.executeQuery(sql);//返回到结果集
```

sql语句不用再传给它了！不然会再编译一次（上为修改前），直接调对象的`executeQuery()`方法

修改如下：

```java
//4.执行sql
            rs = stmt.executeQuery();//返回到结果集
```

### 对比Statement和PreparedStatement

- Statement存在SQL注入问题，PreparedStatement解决了这个问题

- PreparedStatement编译在执行之前，所以只需要编译一次
  
  以后传值都不需要再编译了.编译一次，执行n次。效率较高一些

- PreparedStatement会在编译阶段做类型的安全检查

- 以后80％都基本用PreparedStatement

- 业务方面要求必须支持SQL注入的时候，必须使用Statement。
  
  例如：用到SQL语句拼接而非单纯传值。
  
  如升序、降序..

### PreparedStatement完成增删改

增：（dept是表名）

```java
 String sql = "insert into dept(depono,dname,loc) values(?,?,?)";
       stmt = conn.prepareStatement(sql);
              stmt.setInt(1,60);
```

(改)更新：`update dept set dname = ?,loc = ? where deptno = ?`

删除：    `delete from dept where deptno = ?`

**其他不变**

### 一周目问题

1. 连接失败
   
   已解决：url改成`"jdbc:mysql://localhost:3306/数据库名?serverTimezone=UTC"`+密码输入错误·；

2. 执行登录语句报错：空指针异常`java.lang.NullPointerException`
   
   为解决，估计rs接收数据一步写错了

3. 静态方法返回值类型？

## 事务自动提交机制演示

JDBC事务机制：

1. JDBC中的事务时自动提交的
   
   只要执行任意一条DML语句，则自动提交一次。
   
   这是JDBC默认的事务行为。（不符合原子性）
   
   但是在实际业务当中，通常都是N条DML语句共同联合才能完成的。

        必须保证他们在同一事务中同时成功或同时失败（原子性）

2. 需要**改成手动提交**
   
   java.sql.Connection接口下的<mark>setAutoCommit()方法</mark>
   
   **获取连接第二步时添加**`conn.setAutoCommit(false);//开启事务`

      **事务结束，手动提交数据**：`conn.commit();//提交事务`

        为保证数据安全性，**遇到异常，必须回滚**:

```java
 catch (Exception e) {
            if (conn != null) {
                try {
                    conn.rollback();//回滚事务
            }catch (SQLException e1){
                    e1.printStackTrace();
                }
            }
```

3. 以上事务为**单机事务**（非分布式事务）
4. 分布式事务是说：数据库是以集群形式的，以分布式的，不是一个。必须保证两个数据库都成功，才能成功。

## JDBC工具类的封装

代码爆红，暂时不知道问题出在哪里👇

```java
/*
* JDBC工具类，简化JDBC编程
* 工具库可以不处理异常，往外抛出（扔）throws
* */

public class DBUtil {

    /*
    *工具类中的构造方法都是私有的
    * 因为工具类当中的方法都是静态的，不需要new对象，直接采用类名调用
    * Arrays类..Collections类..
    * 防止别人new对象
    *  */
    private DBUtil(){
        //静态代码块在类加载器时执行，并且只执行一次
        static {
            try {
                Class.forName("com.mysql.cj.jdbc.Driver");
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        public static Connection getConnection () throws SQLException {
            return DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb?serverTimezone=UTC", "root", "999887723");
        }
        public static void close(Connection conn,Statement ps, ResultSet rs){
            if (rs !=null){
                //若结果集不等于空
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (stmt !=null){
                //若结果集不等于空
                try {
                    stmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn !=null){
                //若结果集不等于空
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        }
    }
```

连接getConnection()步骤因为可能接下来步骤还有需要连接，所以异常直接抛出，无需处理。

## 乐观锁与悲观锁

- 行级锁
  
  - 悲观琐
    
    SQL语句后+ `for update` = 这一行🔒住了。
    
    **这个事务没结束前，其他事务无法对其中数据进行操作。**
    
    事务必须排队执行。数据锁住了，不允许并发。
  
  - 乐观锁
    
    多线程并发，都可以对这条记录修改。不过后面有一个记录的版本号。
    
    支持并发，事务不需要排队。
    
    - 事务1-->读取到版本号1.1
      
      事务2-->读取到版本号1.1
      
      事务1先修改了，将版本号修改为1.2
      
      事务2后修改，准备提交时发现版本号变成了1.2！！和最初的不一致，**于是回滚，这个操作放弃**。
      
      （<mark>**辨：脏读，不可重复读，幻读**</mark>）

## 演示行级锁机制

略

debug结束是三角形那个（resume：中断后继续，重新开始，恢复...;简历，摘要；）
