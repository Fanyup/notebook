# 文件上传

文件上传时，对页面的form表单有如下要求：（必须）

- method=**post**

- enctype="multipart/form-data"

- type="file"
  采用post方式提交数据
  采用multipa格式上传文件
  使用input的file控件上传

我们一般使用前端组件库，其实底层原理还是基于ofrm表单的文件上传。

如Element-UI提供的组件等。

**服务端要接收客户端页面上传的文件，通常都会使用Apache的两个组件**：

- commons-fileupload

- commons-io（本质上对流操作）

但用这个api相对代码比较繁琐，所以在Spring框架中的spring-web包里对文件上传进行了封装，大大简化了服务端代码，但底层还是基于apache提供的这两个lib包。

我们只需要在ControlIer的方法中**声明一个MultipartFile类型的参数**即可接收上传的文件。（固定的）

`Fn+Alt+F9`热键静态页面刷新。

![idea64_wbkrKrRT1s.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_wbkrKrRT1s.png)

注意：我们之前加了过滤器LoginCheckFilter,里面自定义了，如果我们未登录，则会返回NOTHING给页面（未登录结果）

```java
//文件上传和下载
@Slf4j
@RestController
@RequestMapping("/common")
public class CommonController {
    @PostMapping("/upload")
    //这里前端写死了，name必须保持一致，所以这里参数名只能是file
    public R<String> upload(MultipartFile file){
        //file是一个临时文件，需要转存到指定位置，否则请求完成后临时文件会删除
        log.info(file.toString());
        return null;
    }
```

所以这里我们想要访问到文件上传这个Controller处理器方法，就**必须要先用户登录**。**登录后用户信息就写到Session会话当中了**。（一次浏览器对话）

**注意，此时我们上传的文件是在一个临时目录下面**，所以我们需要把这个文件转存到指定位置下。否则等这次请求结束之后文件就不存在了。

![idea64_uJjCoXUmr8.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_uJjCoXUmr8.png)

找一下文件所在目录：

![chrome_iKrM96VE0e.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_iKrM96VE0e.png)

## 临时文件转存：配置路径

配置文件(application.yaml）方式动态指定转存位置。

```yaml
reggie:
  path: D:\uploadImg(D盘下的uploadImg文件夹下)
```

![idea64_w4wi97gCCA.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_w4wi97gCCA.png)

测试时几个小错误，yaml路径指定时目录分隔符应左上到右下\（这个）（windows路径是这样），还有，如果指定文件夹目录不存在会报错。一开始（👆这张图）有个错误：@Value注解下漏了个右括号，字符串中东西IDE是不会检测报错的，需要自己回看代码查找。

但是这样还是有不足：文件名也要改成动态的。

## 代码优化

1. 使用UUID（java.util)重新生成文件名，防止文件名称重复造成文件覆盖。

2. 动态截取原始文件名的后缀
   
   ```java
   //原始文件名
   String originalFilename = file.getOriginalFilename();
   String suffix = originalFilename.substring(originalFilename.lastIndexOf("."));
   //UUID
   String fileName = UUID.randomUUID().toString() + suffix;
   
   
   ```

3. 创建目录对象（存放时若不存在该文件夹会报错，那就我们手动创建一个）

```java
//创建一个目录对象
        File dir = new File(basePath);//D:\img\
        if (!dir.exists()) {
            //目录不存在，需要创建
            dir.mkdirs();
        }
```

**File既可以代表目录也可以代表文件。**

return文件名：

```java
return R.success(fileName);
```

# 文件下载

通过浏览器进行文件下载，通常由两种表现形式：

- **以附件形式下载**，将文件保存到指定磁盘目录

- 直接在浏览器中打开

本质上是对流的操作。输出流方式将文件写回到页面。

```java
//文件下载
    @GetMapping("/download")
    public void download(String name, HttpServletResponse response){
            //输入流，读取文件内容
            FileInputStream fileInputStream = null;
            try {
            fileInputStream = new FileInputStream(new File(basePath + name));
            //输出流，通过输出流将文件写回浏览器
            ServletOutputStream outputStream = response.getOutputStream();

            //设置响应类型
            response.setContentType("image/jpeg");

            int len = 0;
            byte[] bytes = new byte[1024];
            while((len = fileInputStream.read(bytes))!= -1){
                outputStream.write(bytes,0,len);
                outputStream.flush();
            }
            //关闭资源
            outputStream.close();
            fileInputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
```

**通过输入流读取文件，再通过输出流向客户端往回写**（类型是img/jepg）

之后的菜品文件上传下载就可以复用这两个。

# 新增菜品

数据表：菜品表dish| 菜品口味表dis_flavor

在开发业务功能前，先将需要用到的类和接口基本结构创建好：

- 实体类DishFlavor（直接从课程资料中导入即可，Dish实体前面课程中已经导入过了

- Mapper接口DishFIavorMapper

- 业务层接口DishFIavorService

- 业务层实现类DishFIavorServiceImpI

- 控制层DishContr011er



## 下拉框实现

CategoryController类中👇：

```java
//根据条件查询分类数据（下拉框）
    @GetMapping("/list")
    public R<List<Category>> list(Category category){
        //条件构造器
        LambdaQueryWrapper<Category> queryWrapper = new LambdaQueryWrapper<>();
        //添加条件
        queryWrapper.eq(category.getType() != null,Category::getType,category.getType());
        //添加排序条件
        queryWrapper.orderByAsc(Category::getSort).orderByDesc(Category::getUpdateTime);

        List<Category> list = categoryService.list(queryWrapper);
        return R.success(list);
    }
```

实现效果： 

![chrome_cjhthLwN9P.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_cjhthLwN9P.gif)

## 问题：又双叒叕找不到导入的实体类

重启IDE。

## 上传并加载图片

复用之前写的代码。也就是说，可以直接使用了。

## 数据以JSON格式保存传入

导入实体类DishDto，它的作用是封装页面提交的数据。

**保存数据传入到菜品表和菜品口味表。**

DTO（Data Transfer Object)数据传输对象。之前我们用的前端页面传入属性和实体类中属性是一一对应的，所以用不上它。

但这里我们需要传入Dish_Flavor字段，也就是数据表中有而实体类中没有。那么这时就不能再用Dish的Java实体类进行数据接收了。

一般dto会单独创建一个新的包（目录）

```java
@Data
public class DishDto extends Dish {

    private List<DishFlavor> flavors = new ArrayList<>();

    private String categoryName;

    private Integer copies;
}
```

![idea64_ziZ1oTKjBt.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_ziZ1oTKjBt.png)

有多个“口味”flavor，所以用list集合。

![idea64_KSzTDhe2fM.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_KSzTDhe2fM.gif)

**再DishService接口新增抽象方法：新增菜品，同时插入菜品对应的口味数据，需要操作两张表:dish和dish_flavor。**

```java
public interface DishService extends IService<Dish> {
    //新增菜品，同时插入菜品对应的口味数据，需要操作两张表:dish和dish_flavor
    public void saveWithFlavor(DishDto dishDto);
}
```

这里为什么不用增强for?**Stream流效率高，数据量大的时候自动多线程**

涉及到多张表的操作：**需要事务控制**，别忘了在主程序上开启对事务的支持 **@EnableTransactionManagement**。保证数据一致性。

DishServiceImpl实现类👇：

```java
@Service
@Slf4j
@Transactional
public class DishServicelImpl extends ServiceImpl<DishMapper, Dish> implements DishService {

    @Autowired
    private DishFlavorService dishFlavorService;//没用上

    //新增菜品，同时保存对应的口味数据
    @Override
    public void saveWithFlavor(DishDto dishDto) {
        //保存菜品的基本信息到菜品表dish
        this.save(dishDto);
        Long dishId = dishDto.getId();//菜品id
        //菜品口味
        List<DishFlavor> flavors = dishDto.getFlavors();
        flavors = flavors.stream().map((item)->{
            item.setDishId(dishId);
            return item;
        }).collect(Collectors.toList());

        //保存菜品口味数据到菜品口味表dish_flavor
        dishFlavorService.saveBatch(dishDto.getFlavors());批量保存它；它是一个集合
    }
}
```

DishController表👇：

```java
//菜品管理
@RestController
@Slf4j
@RequestMapping("/dish")
public class DishController {
    @Autowired
    private DishService dishService;
    @Autowired
    private DishFlavorService dishFlavorService;//暂时没用上

    //新增菜品
    @PostMapping
    public R<String> save(@RequestBody DishDto dishDto){
        log.info(dishDto.toString());
        dishService.saveWithFlavor(dishDto);
        return null;
    }
}
```

## 问题：为添加口味数据到表里

debug检查后发现flavors未被调用，即未被保存到。网课练习中添加了注释后去掉，但我忘记取消注释//符号了。

# 分页查询（难点）

难点：菜品表里未保存分类名称，只保存了typeid。而我们要在页面展示以文字形式的名称。

与以往分页另一个不同：页面发送请求，请求服务端进行图片下载，用与页面图片展示。

当前代码（过去）：

```java
//菜品信息的分页查询
    @GetMapping("/page")
    public R<Page> page(int page,int pageSize, String name){
        //构造分页构造器(封装信息）
        Page<Dish> pageInfo = new Page<>(page,pageSize);
        //条件构造器
        LambdaQueryWrapper<Dish> queryWrapper = new LambdaQueryWrapper<>();
        //添加过滤条件(模糊查询)
        queryWrapper.like(name != null,Dish::getName,name);
        //添加排序条件
        queryWrapper.orderByDesc(Dish::getUpdateTime);
        //执行分页查询
        dishService.page(pageInfo,queryWrapper);

        return R.success(pageInfo);
    }
}
```

效果：

**没展示的把菜品图片复制到d盘的img即可**，刚才在那里做了上传下载的功能。

![chrome_oz1efoaGnJ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_oz1efoaGnJ.png)

你发现了吗？这样写菜品分类是无法展示的，虽然它并没有报错。原因是返回的数据没有与之对应的。此时需要用到之前那个DishDto中新加入的属性，与前端页面对应。

![idea64_P14OWhRQiD.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_P14OWhRQiD.png)

所以，Page类型里new对象，并且给属性赋值。通过对象拷贝，将上面里面的属性值，对应拷贝给它。

![idea64_bAnIA0akGd.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_bAnIA0akGd.png)

```java
Page<DishDto> dishDtoPage = new Page<>();
```

```java
//别忘了DI注入对象类型属性。
  @Autowired
    private CategoryService categoryService;
```

```java
//对象拷贝
        BeanUtils.copyProperties(pageInfo,dishDtoPage,"records");

        List<Dish> records = pageInfo.getRecords();
        List<DishDto> list = records.stream().map((item)->{
            DishDto dishDto = new DishDto();

            //再使用一次对象的拷贝
            BeanUtils.copyProperties(item,dishDto);

            Long categoryId = item.getCategoryId();//菜品分类id，去查分类名称
            //根据id查询分类对象
            Category category = categoryService.getById(categoryId);
            String categoryName = category.getName();
            dishDto.setCategoryName(categoryName);
            return dishDto;
        }).collect(Collectors.toList());//收集信息以List集合类形式返回。

        dishDtoPage.setRecords(list);

        return R.success(dishDtoPage);
    }
```

## debug断点测试

![chrome_e2SLlqPOfT.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_e2SLlqPOfT.png)

小技巧：可以先瓢虫🐞点了之后，再添加相应断点也是行的。只要在你未执行之前。

![H8UWAfMGir.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/H8UWAfMGir.png)

![idea64_8rCA7cFamQ.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_8rCA7cFamQ.png)

第一次对象拷贝完之后：除了records数据其他都有了👇

![idea64_IkTMTFCmD8.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_IkTMTFCmD8.png)

为什么要单独处理records？因为我们后来所创建的List集合泛型与前面是不一样的，即不是Dish。所以我们基于records，通过stream流进行处理。

此时再来查看list集合，可以发现它已将数据帮我们封装好了：

![idea64_jKSWjymTq3.gif](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_jKSWjymTq3.gif)

（其中属性已从泛型为Dish的List类的records属性中继承过来了，并新增了categoryName这个属性。

放行后查看，此时就由菜品名称了：👇

![chrome_dq9YlfcaeR.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_dq9YlfcaeR.png)

# 修改菜品

它与新增菜品共享一个页面/add.html

在开发代码之前，需要梳理一下修改菜品时前端页面(add.html)和服务端的交互过程：

1. 页面发送ajax请求，请求服务端获取分类数据，用于菜品分类下拉框中数据展示（**前面已实现**）

2. 页面发送ajax请求，请求服务端，**根据id查询当前菜品信息，用于菜品信息回显**

3. 页面发送请求/请求服务端进行图片下载/用于页图片回显（**前面已实现**）

4. 点击保存按钮/页面发送ajax请求，**将修改后的菜品相关数据以json形式提交到服务端**

也就是说，我们要做的，就是去**处理这四次请求**。

![idea64_C6bS6KFSYa.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_C6bS6KFSYa.png)

同样道理，在DishService接口扩展抽象方法👇：

```java
    //根据id查询菜品信息和对应的口味信息
    public DishDto getByIdWithFlavor(Long id);
```

别忘了在实现类中来实现这个抽象方法。

```java
//根据id查询菜品信息和对应的口味信息（分为两步）
    @Override
    public DishDto getByIdWithFlavor(Long id) {
        //查询菜品基本信息（dish表）
        Dish dish = this.getById(id);//顺序别搞错，不然找不着
        //对象拷贝
        DishDto dishDto = new DishDto();
        BeanUtils.copyProperties(dish,dishDto);

        //查询当前菜品对应口味信息（dish_flavor表）
        //构造条件构造器
        LambdaQueryWrapper<DishFlavor> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(DishFlavor::getDishId,dish.getId());
        List<DishFlavor> flavors = dishFlavorService.list(queryWrapper);
        dishDto.setFlavors(flavors);

        return dishDto;
    }
```

Controller对应代码：

```java
//根据id查询菜品信息和对应的口味信息
    @GetMapping("/{id}")
    public R<DishDto> get(@PathVariable Long id){
        DishDto dishDto = dishService.getByIdWithFlavor(id);
        return R.success(dishDto);
    }
}
```

![idea64_wEPUP7gxkn.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_wEPUP7gxkn.png)

删除操作对应sql语句：

`delete from dish_flavor where dish_id = ???`

```java
@Override
    public void updateWithFlavor(DishDto dishDto) {
        //更新dish表基本信息
        this.updateById(dishDto);
        //清理当前菜品对应口味数据（delete操作）
        LambdaQueryWrapper<DishFlavor> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(DishFlavor::getDishId,dishDto.getId());
        dishFlavorService.remove(queryWrapper);
        //添加当前提交过来的口味数据(insert操作)
        List<DishFlavor> flavors = dishDto.getFlavors();
        flavors = flavors.stream().map((item)->{
            item.setDishId(dishDto.getId());
            return item;
        }).collect(Collectors.toList());
        dishFlavorService.saveBatch(flavors);//批量保存
    }
```

别忘了在实现类方法上加事务注解，保证数据一致性。
