# Lombok&Postman

`Tab+Alt`快速切屏

解决实体类相关方法set,get，自动生成，提供注解消除java类中大量的样板代码

## 原理

编译时生效，根据lombok注解修改语法树，再到编译.class文件

## 1.引入依赖

maven中引入依赖，可去maven仓库找

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
    <scope>provided</scope>
</dependency>
```

## 2.使用提供的注解

### @Data

用在类上

```java
@Data
public class User {
    private String id;
    private String name;
    private Integer age;
    private Date bir;
}
```

自动给对象提供GET SET toString equald等方法

@Accessors(chain=true)用在类上，用来给类中set方法开启链式调用

@skf4j 用在类上，快去给类中定义一个日志变量  直接加log.info("说点什么")(相当于在编译阶段创建了一个日志对象)log.debug(),log.warn(),log.error()

# Postman
