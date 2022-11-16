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



# 文件下载

通过浏览器进行文件下载，通常由两种表现形式：

- **以附件形式下载**，将文件保存到指定磁盘目录

- 直接在浏览器中打开

本质上是对流的操作。
