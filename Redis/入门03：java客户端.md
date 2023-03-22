大佬博客笔记整理👉[Vz-Blog](https://www.oz6.cn/articles/58)

- Jedis的弊端：**实例时线程不安全**的，也就是说多线程并发运行时有线程安全问题。所以在多线程使用时，必须为每个线程创建独立的Jedis连接，那么就需要使用**链接池**实现了。

- Lettuce（现在官方并不推荐）是基于Netty实现的，Netty是高性能网络编程框架，底层支持同步、异步连接，并且支持响应式编程。线程安全。6之前式单线程。

网课先选的Jedis。

[github-redis](https://github.com/redis/jedis)

## Jedis

1. 新建空maven项目project

2. 引入jedis依赖
   
   ```xml
       <dependencies>
           <dependency>
               <!--jedis依赖-->
               <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
               <version>4.3.0</version>
           </dependency>
           <!--单元测试-->
           <dependency>
               <groupId>org.junit.jupiter</groupId>
               <artifactId>junit-jupiter</artifactId>
               <version>5.7.1</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   ```

3. 建立连接
   
   `@BeforeEach`是单元测试里的一种语法

4. 测试string
   
   ![idea64_S5KkCruvNt.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_S5KkCruvNt.png)
   
   **soutv输入一个值value，快捷键。**

5. 释放资源

```xml
public class JedisTest {
    private Jedis jedis;

    @BeforeEach
    void setUp(){
        // 1.建立连接
        jedis = new Jedis("192.168.233.130",6379);
        // 2.设置密码
        jedis.auth("123321");
        // 3. 选择1库
        jedis.select(0);
    }

    @Test
    void testString(){
        //存入数据
        String result = jedis.set("name", "胡歌");
        //获取数据
        String name = jedis.get("name");
        System.out.println("name = " + name);
    }

    @AfterEach
    void tearDown(){
        if (jedis != null) {
            jedis.close();
        }
    }
```

![idea64_DEneUq85FK.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_DEneUq85FK.png)

### Jedis连接池

因为其本身是线程不安全的，如果在多线程的环境下并发访问有可能出现问题。推荐使用Jedis线程池（也就是连接池）代替直连方式。

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;
    static {
        //配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        //最大连接数
        poolConfig.setMaxTotal(8);
        //最多空闲连接
        poolConfig.setMaxIdle(8);
        //最小空闲连接
        poolConfig.setMinIdle(0);
        //等待时常
        poolConfig.setMaxWaitMillis(1000);
        //创建连接池对象
        jedisPool = new JedisPool(poolConfig, "192.168.233.130",
                6379, 1000, "123321");
    }
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

测试类里需要修改的地方是连接：

`jedis = JedisConnectionFactory.getJedis();`

## SpringDataRedis

整合来了Lettuce和Jedis

提供了RedisTemplate类统一API来操作Redis。

和Spring提供了JDBCTemplate一个意思。

支持发布订阅模型、哨兵、集群...

支持基于JDK,JSON,字符串,Spring对象的数据序列化和反序列化。

支持基于Redis的JDKCollection重新实现，这样就是分布式跨系统的。

封装了一系列的API。对命令进行了分组。

![chrome_dH5Mp06boU.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_dH5Mp06boU.png)

SpringBoot已经提供了对它的支持（自动装配），使用非常简单。

![idea64_3TWhgwx8ZI.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_3TWhgwx8ZI.png)

新建一个Springboot项目project，**连接spring.io的URL需要先将梯子关掉。**

![idea64_yTDs1XmxvO.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_yTDs1XmxvO.png)

springboot启动2.7.9。Maven用的3.6.3。

1. 引入spring-boot-starter-data依赖（前面选项旋律）
   
   ```xml
   <!--redis依赖-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
           <!--common-pool-->
           <dependency>
               <groupId>org.apache.commons</groupId>
               <artifactId>commons-pool2</artifactId>
           </dependency>
   ```

2. 在application.yml配置Redis信息
   
   注意spring默认使用lettuce，如果池要选择jedis的，需要再pom文件里导入相关依赖。
   
   ![idea64_9V6WC2x1Y2.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_9V6WC2x1Y2.png)
   
   ```yaml
   spring:
     redis:
       host: 192.168.233.130
       port: 6379
       password: 123321
       lettuce:
         pool:
           max-active: 8
           max-idle: 8
           min-idle: 0
           max-wait: 100ms
   ```

3. 注解注入RedisTemplate(自动注入)

4. 编写测试

### 问题：导入池依赖时爆红

更换Maven重载以下就OK了。

### 问题：注入依赖redisTemplate爆红

检查了一下能找到实现类，但实例对象爆红，错误是无法找到。看了一下弹幕，将注解`@Autowired`更改成`@Recource`就好了。然后去补习了一下两者的不同👇

**@Autowired能够用在：构造器、方法、参数、成员变量和注解上，而@Resource能用在：类、成员变量和方法上**。

按理来说前者的应用常见应该是多一些的，具体我也没弄明白。帖子暂时贴上👉[Spring中@Autowired和@Resource的区别](https://blog.csdn.net/Weixiaohuai/article/details/120853683#:~:text=%40Autowired%E8%83%BD%E5%A4%9F%E7%94%A8%E5%9C%A8%EF%BC%9A%E6%9E%84%E9%80%A0,%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%E5%92%8C%E6%96%B9%E6%B3%95%E4%B8%8A%E3%80%82&text=%40Autowired%E6%98%AFSpring%E5%AE%9A%E4%B9%89%E7%9A%84,%E4%B8%8E%E5%85%B6%E4%BB%96%E6%A1%86%E6%9E%B6%E4%B8%80%E8%B5%B7%E4%BD%BF%E7%94%A8%E3%80%82)

```java
@SpringBootTest
class RedisDemoApplicationTests {
    @Resource
    private RedisTemplate redisTemplate;
    @Test
    void testString() {
        //写入一条String数据
        redisTemplate.opsForValue().set("name", "胡歌");
        //获取string数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name = " + name);
    }
}
```

### 问题：值的序列化

![Xshell_WwAltpLr44.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/Xshell_WwAltpLr44.png)

下面一串乱码返回的就是刚刚用springredis返回的「胡歌」

因为redistemplate的set方法接收的参数并不是string而是object。这是springdataredis的一个特殊功能，可以接收任何对象。**而其底层默认对其处理方式就是JDK默认的object序列化。** 看一下RedisTemplate的源码👇

![idea64_Nif3ffex2L.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_Nif3ffex2L.gif)

因此暴露问题：

- 可读性差

- 内存占用较大

**快捷键Ctrl+H**可以查看该接口的是实现类细节。

注意：一点要选中该接口再点Ctrl+H才能找到对应的！！

需要去改变redistemplate的serializer序列化，key一般用srting来写，值一般用json来写。

### 解决：自定义RedisTemplate的序列化方式

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
        //创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        //设置连接工厂
        template.setConnectionFactory(connectionFactory);
        //创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //设置Key的序列话
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        //设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        //返回
        return template;
    }
}
```

添加后，返回测试类，给注入的RedisTemplate加一个泛型`<String, Object>`

### 问题：connectionFactory爆红

系统提示说找不到，我在pom.xml中降低了springboot的版本，刷新重载后问题解决。

### 问题：运行出错时学会看报错原因

![idea64_VFzgKR3cDw.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VFzgKR3cDw.png)

causedby告诉我们他缺少一个json的依赖。

`Bean instantiation via factory method failed;`

bean通过工厂方法实例化失败。

```xml
<!--json依赖-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
```

事实上在平时的工作时是不需要我们手动引入的，因为平时我们使用springmvc，会自动帮我们引入它。

测试成功。

### 测试能否传入一个java对象

首先创建一个User类👇

```java
@Data
@NoArgsConstructor  //无参构造
@AllArgsConstructor //有参构造
public class User {
    private String name;
    private Integer age;
}
```

接下来再编写一个单元测试：

```java
@Test
    void testSaveUser(){
        //写入数据
        redisTemplate.opsForValue().set("user:100", new User("虎哥", 21));
        //获取数据(强转Object为User类)
        User o = (User) redisTemplate.opsForValue().get("user:100");
        System.out.println("o = " + o);
    }
```

测试成功👇

![idea64_AXGLPEqkII.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_AXGLPEqkII.png)

我们要求key为string字符串格式，而值的话就随意了。

### 优化问题：解决自动化序列化内存占用问题（推荐使用实践方案）

我们明明没有要求，可是存入时它自动夹入了私货：User的Class字节码也被它加进去了！仔细观察上图可以发现问题：这个字节码本身占用的内存空间，甚直比数据本身还要大！不过不使用它，它就无法帮我们自动反序列化。

优化方法：不适用JSON序列化器来处理value，而是**统一使用String序列化器**，要求只能处理String类型的字符串。若要储存java对象，**则必须手动完成对象的序列化和反序列化。**

String默认提供了一个StringRedisTemplate类，因此不需要我们重新再修改原先的RedisTemplate了！因为它的key和value默认序列化方式就是String方式。

这样，**仅是多加了两行手动代码，就帮我们优化了内存占用的问题**，大大提升了运行效率。

复制一份test测试类，修改注入👇及下面同样需要修改的地方。

```java
@Resource
    private StringRedisTemplate stringRedisTemplate;
```

```java
    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSaveUser() throws JsonProcessingException {
       //创建对象
        User user = new User("胡戈尔", 21);
        //手动序列化
        String json = mapper.writeValueAsString(user);
        //写入数据
        stringRedisTemplate.opsForValue().set("user:200", json);
        //获取数据（强转不了，因为默认取出来一定是一个字符串）
        String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
        //手动反序列化
        User user1 = mapper.readValue(jsonUser, User.class);
        System.out.println("user1 = " + user1);
    }
```

![idea64_YKtawLE4pf.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_YKtawLE4pf.png)

### 补充知识点：测试哈希结构的用法

```java
@Test
    void testHash(){
        stringRedisTemplate.opsForHash().put("user:400", "name", "湖广");
        stringRedisTemplate.opsForHash().put("user:400", "age", "23");
        //取
        Map<Object, Object> entries = stringRedisTemplate.opsForHash().entries("user:400");
        System.out.println("entries = " + entries);
    }
```

![idea64_vwLka9rVCH.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_vwLka9rVCH.png)
