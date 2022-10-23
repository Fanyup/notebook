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
