- Servlet是Java程序，所以在Servlet中完全可以编写JDBC代码连接数据库

- 在一个webapp中去连接数据库，需要将去报jar包放到`WEB-INF/lib`目录下

（`com.mysql.cj.jdbc.Driver`这个类九载驱动jar包当中）

### 在集成开发环境中开发Servlet

1. New Project(直接新建非空的Project)（示例：新建空项目Empty Project，再项目里添加Maven)

2. 新建模块：`File->New->Module`
   
   新建一个普通的JavaSE模块（先不要新建Java Enterprise模块，因为我们Maven还没学）
   
   模块会自动建在Javaweb的project下面

3. 让Module模块变成javaEE的模块(让Module变成webapp的模块，符合webapp规范。符合Servlet规范)
   
   再Module上右键：`Add Framework Support..`(添加框架支持)
   
   在弹出的窗口中，选择`Web Application`(选择的是webapp的支持)
   
   IDEA会自动生成一个符合Webapp的目录结构
   
   **注意：在IDEA工具生成的目录中优一个web目录，其代表webapp的根（例之前的：crm）**

4. 编写Servlet(StudentServlet)
   
   - `class StudentServlet implements Servlet`
   
   - 这个时候发现Servlet.class文件没有，怎么办？
   
   - 从`tomcat/lib/servlet-api.jar`和`jsp-api.jar`包添加到`classpath`（这里说的是IDEA中的）当中
     
     `File->Project Structure...->Modules`下的Dependencies(依赖)，这里可以添加库(Library)，也可以添加jar包。这里我们选择添加jar包，因为用过不到tomcat中库中其他包
     
     加入后点Apply(应用)

5. 点中`Servelt`，键盘点击`alt+Enter`后`implements method`（StudentServlet类去实现Servlet接口中5给方法）选中

6. 在Servelt当中的service方法中编写业务代码（我们这里连接数据库了）
   
   ```java
   //设置响应的内容类型
           response.setContentType("text/html");
           PrintWriter out = response.getWriter();
   ```
   
   后【处理结果集】打印out

        `out.print();`

7. WEB-INF下加一个目录lib（第三方库）
   
   将连接数据库的jar包(驱动）加到lib目录下

8. 在web.xml文件中完成StudentServlet类的注册
   
   请求路径和Servlet之间对应起来
   
   ```xml
     <servlet>
           <servlet-name>studentServlet</servlet-name>
           <servlet-class>com.bjpowernode.javaweb.servlet.StudentServlet</servlet-class>
       </servlet>
       <servlet-mapping>
           <servlet-name>studentServlet</servlet-name>
           <url-pattern>/servlet/student</url-pattern>
       </servlet-mapping>
   ```

9. 给个html页面，在HTML页面中编写一个超链接
   
   用户点击超链接，发送请求，Tomcat执行后台的StudentServlet
   
   （在WEB-INF外面创建哦！）
   
   student.html只能放到WEB-INF外面
   
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>student page</title>
   </head>
   <body>
       <a href="/xmm/servlet/student">student list</a>
       <!--别忘了加项目名，不是jsp，没办法动态获取-->
       <!--这里的项目名用的是/xmm，先写死-->
   </body>
   </html>
   ```

10. 让IDEA工具去关联Tomcat服务器。
    
    关联的过程当中将webapp部署到Tomcat服务器当中
    
    ![ApplicationFrameHost_QiVTrOPnoK.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_QiVTrOPnoK.png)
    
    左上角➕点击`Tomcat Server`->`local`
    
    选完后点`Deployment`(部署)->`Artifact`
    
    点击这个来部署webapp
    
    ![ApplicationFrameHost_845Greda8g.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_845Greda8g.png)

    应用的上下文（根），这里改一下
    
    
    
    ![ApplicationFrameHost_PZHDK2VPQf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_PZHDK2VPQf.png)

    部署完成。

11. 启动Tomcat服务器
    
    ![ApplicationFrameHost_dSo8vkNrYE.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_dSo8vkNrYE.png)
    
    上方绿色小瓢虫，以debug的模式启动服务器
    
    我们开发中建议使用debug模式启动tomcat

12. 打开浏览器，在地址栏上输入：
    
    `http://localhost:8080/xmm/student.html`
