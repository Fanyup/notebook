Maven home path:D:/Program Files/apache-maven-3.8.6-bin/apache-maven-3.8.6

user settings files:conf\settings.xml

Local repository:E:\repository

settings.xml配置阿里镜像+jdk版本：[Maven配置阿里云镜像与JDK编译版本-阿里云开发者社区](https://developer.aliyun.com/article/1162679?spm=a2c6h.12873639.article-detail.27.6251667cxQyq3n&scm=20140722.ID_community@@article@@1162679._.ID_community@@article@@1162679-OR_rec-V_1)

3.8.6下载：[Index of /dist/maven/maven-3/3.8.6/binaries](http://archive.apache.org/dist/maven/maven-3/3.8.6/binaries/)

![chrome_P6W80sH56C.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_P6W80sH56C.png)

## Maven常用命令总结

- 编译源代码： mvn compile

- 编译测试代码：mvn test-compile

- 运行测试：mvn test

- 打包生成jar/war文件：mvn package

- 在本地Repository中安装jar：mvn install

- 清除生成的文件：mvn clean

- 生成idea项目：mvn idea:idea

- 编译测试的内容：mvn test-compile

- 只打jar包: mvn jar:jar

- 只测试而不编译，也不测试编译：mvn test -skipping compile -skipping test-compile ( -skipping 的灵活运用，当然也可以用于其他组合命令)

- mvn -version/-v 显示版本信息

- mvn -e 显示详细错误 信息.

- mvn validate 验证工程是否正确，所有需要的资源是否可用。

- mvn integration-test 在集成测试可以运行的环境中处理和发布包。

- mvn verify 运行任何检查，验证包是否有效且达到质量标准。

- mvn generate-sources 产生应用需要的任何额外的源代码，如xdoclet。

- mvn dependency:resolve 打印出已解决依赖的列表

- mvn dependency:tree 打印整个依赖树

- mvn jetty:run 调用 Jetty 插件的 Run 目标在 Jetty Servlet 容器中启动 web 应用
