这章我们要实现的是分类管理业务

分类管理业务开发

分类类型：菜品分类+套餐分类

# 公共字段自动填充

mybatis-plus为我们提供的特性，不属于我们开发的内容，而是简化开发的一种功能。

对我们之前写的业务进行改造。

### 问题：公共字段统一处理

每次总是要**设置更新时间和更新人**，其他表中也有它们，所以它们是公共字段，也就是重复性工作。我们能否将这些公共字段在某个地方统一处理呢？

它的功能就是**在插入或者更新的时候为指定字段赋予指定的值，避免了重复代码。**

实现：

1. 在**实体类属性上**加入注解`@TableField`，**指定自动填充的策略**
   
   哪些属性是公共字段，就在响应属性上添加注解。
   
   ```java
    @TableField(fill = FieldFill.INSERT)//插入时填充字段
       private LocalDateTime createTime;
   
       @TableField(fill = FieldFill.INSERT_UPDATE)//插入和更新时填充字段
       private LocalDateTime updateTime;
   
       @TableField(fill = FieldFill.INSERT)
       private Long createUser;
   
       @TableField(fill = FieldFill.INSERT_UPDATE)
       private Long updateUser;
   ```

2. 按照Mybatis-Plus框架要求**编写元数据对象处理器（其实就是一个类）**，这个类必须要去**实现接口MetaObjectHandler**(要求，必须的)，在此类中统一为公共字段赋值。（里面写填充字段的内容）
   
   报错是因为我们要实现它里面两个方法（update时Fill和InsertFill)

编写好大框架，打断点测试一下：

```java
@Component
@Slf4j
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("公共字段自动填充[insert...]");
        log.info(metaObject.toString());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("公共字段自动填充[update...]");
        log.info(metaObject.toString());
    }
}
```

现在使用的自动填充功能，就可以完善更新一下以前写的代码。

![idea64_nyLy38FgmO.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_nyLy38FgmO.png)

我们之前是通过调用session会话对象里存储的（HttpServletRequest request)属性获取id的。以前是做了这一步：

```java
 //6.确定登录成功，将用户id放到session里面
        request.getSession().setAttribute("employee",emp.getId());
```

这也就是为什么之前我们能提供属性名叫employee的来获取到我们的属性Id,因为值传给这个键了。

这里我们写给它写死当前用户id👇之后再来解决动态获取员工id的问题：

```java
public class MyMetaObjectHandler implements MetaObjectHandler {
    //插入操作，自动填充
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("公共字段自动填充[insert...]");
        log.info(metaObject.toString());
        metaObject.setValue("createTime", LocalDateTime.now());
        metaObject.setValue("updateTime",LocalDateTime.now());
        metaObject.setValue("createUser",new Long((1)));
        metaObject.setValue("updateUser",new Long((1)));
    }
```

### 问题：8080端口被占用

测试时犯错了。我先跑了springboot主项目，后面又想做debug断点测试，于是点了两次。关闭单独重开后就没有这个问题了。

### 功能完善：获取动态id

在当前自动填充类中是不能获得HttpSession对象的，我们这里可以使用ThreadLocal类来解决这个问题（线程类）。

**客户端发送的每次http请求，对应的在服务端都会分配一个新
的线程来处理**，在处理过程中涉及到下面类中的方法都属于相同的一个线程：|

![chrome_tQl8T0SoON.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_tQl8T0SoON.png)

如何证明？我们来测试一下：

![idea64_INlGhTYLEZ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_INlGhTYLEZ.png)

![idea64_tCpx8cmyhD.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_tCpx8cmyhD.png)

![idea64_Pl8DxqPLlJ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_Pl8DxqPLlJ.png)

测试结果：（多个类的多个方法是同一线程）

![idea64_BR6ggioTAk.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_BR6ggioTAk.png)

调用链上是同一线程。运用这一特性我们就可以是哦那个ThreadLocal进行解决。

#### 什么是ThreadLocal?

**它并不是一个线程（Thread)，而是线程的局部变量。** 当使用它维护变量时，它能为每个使用该变量的线程**提供独立的变量副本**，也就是说每一个线程都可以独立地改变自己的副本，不会影响其他线程所对应的副本。

（实现线程隔离）（**只能在线程内**获取到相应的值，线程外访问不到）

它为每个线程提供一份单独存储空间，也就是说我们刚刚上面提到的三个方法在同一线程内，它们共享这份存储空间。

常用方法：

- `public void set(T value)`设置当前线程的**线程局部变量的值**

- `public T get()`返回当前线程所对应的线程局部变量的值

#### 具体思路

做了这么久铺垫其实就是为了引入ThreadLocal这个概念。它以每个线程作为作用域。**这个类是由JDK提供的。**

1. 我们可以在LoginCheckFilter的doFilter方法中获取当前登录用户id

2. **调用ThreadLocal的set方法**来设置当前线程的线
   程局部变量的值(用户id)

3. 然后在MyMetaObjectHandler的updateFilI方法中**调用ThreadLocal的get方法**来获得**当前
   线程所对应的线程局部变量的值**（用户id)。

#### 实现步骤

1. 编写BaseContext工具类，它是**基于ThreadLocal封装的工具类**

2. 在LoginCheckFilter的doFilter方法中**调用BaseContext**来设置当前登录用户的id

3. 在MyMetaObjectHandler的方法中调**用BaseContext**获取登录用户的id

工具类放在common（通用）目录下👇:

```java
//基于ThreadLocal封装工具类，用与保存和获取【当前登录用户】的id
public class BaseContext {
    private static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public static void setCurrentId(Long id){
        threadLocal.set(id);
    }
    public static Long getCurrentId(){
        return threadLocal.get();
    }
}
```

![idea64_KzJmSZJKSX.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_KzJmSZJKSX.png)

![idea64_nBHRavKI6J.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_nBHRavKI6J.png)

# 新增菜样/套餐分类

数据表category同样和employee一样，将name字段添加了索引作唯一约束。

## 搭建大框架

在开发业务功能前，先将需要用到的类和接口基本结构创建好：

- **实体类Category**（直接从课程资料中导入即可）

- **Mapper接口CategoryMapper**
  
  - 该接口继承`BaseMapper<Category>`（里面有很多sql语句）

- **业务层接口CategoryService**
  
  - 继承`IService<Category>`（指定泛型）
    
    点开看里面其实是进一步封装了，**每个操作的元对象都是一个xxxWrapper**，我理解是比方说把实体类--(包裹)-->成一个对象，该对象可以执行sql语句。

- **业务层实现类CategoryServicelmpl**
  
  ```java
  @Service //组件声明
  public class CategoryServiceImpl extends ServiceImpl<CategoryMapper, Category> implements CategoryService {
  }
  ```
  
  - 如何将mapper层与service层连接？可以通过service实现类。
  
  - 查询发现继承的这个父类`ServiceImpl`中有获取当前Mapper，事务提交和回滚，执行分支，getOne等方法。（👇第一步就是ID注入对象类型属性，如何做的？创建相应泛型的类实例对象）
    
    ![idea64_RqoicucjGC.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_RqoicucjGC.png)
  
  - 而这里方法实现的IService<泛型>就是我们指定的CategoryService接口（其继承父接口IService所以，直接写它就🆗。

- 控制层CategoryControIIer
  
  ```java
  //分类管理
  @RestController
  @RequestMapping("/catetory")
  public class CategoryController {
      @Autowired //属性注入
      private CategoryService categoryService;
  }
  ```

![explorer_ziocKIgmQw.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/explorer_ziocKIgmQw.png)

## 待解决：currentMapperClass如何实现

这里有点没明白ServiceImpl中currentMapperClass获取当前映射类的走法,this对象指向谁？

```java
 @Autowired
    protected M baseMapper;
    protected Class<T> entityClass = this.currentModelClass();
    protected Class<T> mapperClass = this.currentMapperClass();
```

请求方式POST

提交地址都是/category

```java
//分类管理
@Slf4j
@RestController
@RequestMapping("/category")
public class CategoryController {
    @Autowired //属性注入
    private CategoryService categoryService;

    @PostMapping
    public R<String> save(@RequestBody Category category){
        log.info("category:{}",category);
        categoryService.save(category);
        return R.success("新增分类成功");
    }
}
```

启动测试...

## 问题：主程序启动失败

error:`Unsatisfied dependency expressed through field 'categoryService'`

说是服务层实现类出问题了，检查后发现原来是Mapper接口忘记添加注解@Mapper

```java
@Mapper
public interface CategoryMapper extends BaseMapper<Category> {
}
```

# 分类信息分页查询

```java
 //分页查询
    @GetMapping("/page")
    public R<Page> page(int page,int pageSize){
        //分页构造器
        Page<Category> pageInfo = new Page<>(page,pageSize);
        //条件构造器对象，设置一个排序的条件sort
        LambdaQueryWrapper<Category> queryWrapper = new LambdaQueryWrapper<>();
        //添加排序条件，根据sort进行排序.asc升序
        queryWrapper.orderByAsc(Category::getSort);
        //进行分页插叙
        categoryService.page(pageInfo,queryWrapper);
        return R.success(pageInfo);
    }
```

## 问题：表中找不到is_deleted行

error报错提示很清晰：

![idea64_dLYj8iobTU.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_dLYj8iobTU.png)

最后一句也指明了：错误可能发生在创建属性中。

错误问题出现在category实体类里的isDeleted变量在数据库表中没有对应的字段

# 删除分类

我们想做到删除分类类型，在此之前实现判断：该类型目录中是否含有（关联）菜品/套餐，如果有菜，则不能被删除。

先做出大框架👇，再实现判断功能。

```java
 //根据id删除分类
    //前端只需返回res.code=1，所以返回String就好了
    @DeleteMapping
    public R<String> delete(Long id){
        log.info("删除分类:{}",id);
        //先简单处理
        categoryService.removeById(id);
        return R.success("分类信息删除成功");
    }
```

## 功能完善

要完善分类删除功能，需要先准备基础的类和接口：

1. 实体类Dish和Setmeal（从课程资料中复制即可）

2. Mapper接QDishMapper和SetmeaIMapper

3. Service接口DishService和SetmealService

4. Service实现类DishServicelmpl和SetmeaIServiceImpI

自定义一个删除功能，我们在CategogryServiceImpl实现类下面编写：👇

![idea64_sw3lWrOzAV.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_sw3lWrOzAV.png)

大框架：

```java
   @Autowired
    private DishService dishService;
    @Autowired
    private SetmealService setmealService;

    //根据id删除分类，删除之前需要进行判断
    @Override
    public void remove(Long id) {
        //查询当前分类是否关联了菜品，如果已经关联，抛出一个业务异常
        //构造查询条件器
        LambdaQueryWrapper<Dish> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Dish::getCategoryId,id);
        int count = dishService.count(queryWrapper);

        //和categoryId有关
        if (count > 0){
            //已经关联菜品，抛出一个业务异常（自定义）
        }
        //查询当前分类是否关联了套餐，如果已经关联，抛出一个业务异常
        LambdaQueryWrapper<Setmeal> setmealLambdaQueryWrapper = new LambdaQueryWrapper<>();
        setmealLambdaQueryWrapper.eq(Setmeal::getCategoryId,id);
        int count1 = setmealService.count(setmealLambdaQueryWrapper);
        if (count1 > 0) {
            //已经关联套餐，抛出一个业务异常（自定义）

        }

        //正常删除分类,super的是IService类里的方法
        //等价super.removeById(id);
        setmealService.removeById(id);
    }
}
```

下面在common项目下添加一个自定义异常类：

```java
//自定义业务异常
public class CustomException extends RuntimeException{
    public CustomException(String message){
        super(message);
    }
}
```

在Controller中抛出异常：

```java
throw new CustomException("当前分类目录下含菜品，不能删除！");
```

全局异常处理器（类）下捕获异常：就是catch处理它，目的是想让它在前端页面中展示。

```java
@ExceptionHandler(CustomException.class)
    public R<String> exceptionHandler(CustomException ex){
        log.error(ex.getMessage());
        return R.error(ex.getMessage());
    }
```

全都完成后别忘了在Controller中调用这个方法喔。

```java
//根据id删除分类
//只返回id，所以无需用Java类注入属性+@RequestBody以JSON格式
    //前端只需返回res.code=1，所以返回String就好了
    @DeleteMapping
    public R<String> delete(Long id){
        log.info("删除分类:{}",id);
        //先简单处理
        //categoryService.removeById(id)默认;
        categoryService.remove(id);
        return R.success("分类信息删除成功");
    }
```

## 问题：【java: 找不到符号】

主程序启动时报错说找不到我的实体类Dish，检查后明明就在那里。网络上查找方法是清一下IDEA缓存，重启。测试后发现方法可行。

![idea64_Z2rE9dyd8v.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_Z2rE9dyd8v.png)

## 问题：id接收为null

原因：前端传入参数值为ids，前后不一致，controller当然接收不到。

![chrome_UfImY0LK9h.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_UfImY0LK9h.png)

不只是ServiceImpl的要改，Controller层的也要改。

# 修改分类

请求方式put

请求地址还是/category

```java
 //根据id修改分类信息
    @PutMapping
    public R<String> update(@RequestBody Category category){
        categoryService.updateById(category);

        return R.success("修改分类信息成功");
    }
```

温习以下：

我们之前设置过metaObject元数据对象处理器，注解@TableField

它会在我们update更新时自动更新，填入创建人那些

所以这次更新必会更新修改人信息。

![MySQLWorkbench_cRGKhZfNsr.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/MySQLWorkbench_cRGKhZfNsr.png)
