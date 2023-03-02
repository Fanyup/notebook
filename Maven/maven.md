Maven home path:D:/Program Files/apache-maven-3.8.6-bin/apache-maven-3.8.6

user settings files:conf\settings.xml

Local repository:E:\repository

settings.xml配置阿里镜像+jdk版本：[Maven配置阿里云镜像与JDK编译版本-阿里云开发者社区](https://developer.aliyun.com/article/1162679?spm=a2c6h.12873639.article-detail.27.6251667cxQyq3n&scm=20140722.ID_community@@article@@1162679._.ID_community@@article@@1162679-OR_rec-V_1)

3.8.6下载：[Index of /dist/maven/maven-3/3.8.6/binaries](http://archive.apache.org/dist/maven/maven-3/3.8.6/binaries/)

![chrome_P6W80sH56C.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_P6W80sH56C.png)

## Maven常用命令总结

常用maven命令总结：
mvn -v //查看版本
mvn archetype:create //创建 Maven 项目
mvn compile //编译源代码
mvn test-compile //编译测试代码
mvn test //运行应用程序中的单元测试
mvn site //生成项目相关信息的网站
mvn package //依据项目生成 jar 文件
mvn install //在本地 Repository 中安装 jar
mvn -Dmaven.test.skip=true //忽略测试文档编译
mvn clean //清除目标目录中的生成结果
mvn clean compile //将.java类编译为.class文件
mvn clean package //进行打包
mvn clean test //执行单元测试
mvn clean deploy //部署到版本仓库
mvn clean install //使其他项目使用这个jar,会安装到maven本地仓库中
mvn archetype:generate //创建项目架构
mvn dependency:list //查看已解析依赖
mvn dependency:tree //看到依赖树
mvn dependency:analyze //查看依赖的工具
mvn help:system //从中央仓库下载文件至本地仓库
mvn help:active-profiles //查看当前激活的profiles
mvn help:all-profiles //查看所有profiles
mvn help:effective -pom //查看完整的pom信息
