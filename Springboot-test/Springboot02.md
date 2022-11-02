# ORM操作Mysql

ORM指的是Mybatis框架

Object-relational mapping

java对象数据库表的映射关系，我们用Mybatis框架操作数据。springboot集成Mybatis~

## 使用步骤

1. mybatis起步以来，完成mybatis对象自动配置，对象放在容器中。

2. pom.xml指定src/main/java目录中的xml文件包含到classpath类路径中，编译得有它才行。

3. 创建实体类Student

4. 创建Dao接口StudentDao，创建一个查询学生的方法

5. 创建Dao接口对应的Mapper文件，xml文件，写sql语句

6. 创建Service对象，创建StudentService接口和它的实现类。调用dao对象的方法，完全数据库的操作。

7. 创建Controller对象，访问Service

8. 写application.properties文件，配置数据库的连接信息。

新建一个项目，在勾选导入Web框架的基础上加上MySQL驱动（依赖）和mybatis框架

![idea64_B0fUw7Yl9u.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_B0fUw7Yl9u.png)

# 自动配置

- 启动器：说白了就是springboot的启动场景

- 比如spring-boot-starter-web,，它会就会帮我们自动导入web环境所有的依赖！

- springboot会将所有的功自芴景，都变成一个个的启动器

- 我们要使用什么功能，就只需要找到对应的启动器就可以了
