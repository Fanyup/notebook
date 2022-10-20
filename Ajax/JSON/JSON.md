# JSON

JavaScript Object Notation“**JS对象标记**(记号，符号)”(所以注解才叫"annotation")

它是一种标准的数据交换格式。涉及到不同系统之间交换数据。（另一个是XML）（目前非常流行）

**轻量级，体积小，易解析**。

XML体积较大，解析麻烦，但是尤其优点是：语法严谨。（通常银行相关的系统之间数据交换用它）

XML是W3C万维网联盟定义的规范。

```json
//创建JSON对象
var studentObj = {
"sno" : "110",
"sname" " "张三",
“sex" : "男"
};
//访问JSON对象的属性
alert(studentObj.sno + "," +studentObj.name);
```

不像JS得去定义一个类，然后new创建对象，访问对象属性。区别在于JSON没有类型。所以**它也被称为无类型对象**。

也可以定义数组

```json
var student = [{"son":"110",...}];
```

访问遍历它很简单，挨个访问每个元素的属性。

Java能字符串拼接，发给浏览器，浏览器能解析数据。这样java就与浏览器之间传数据了。

## 语法格式

没有类型

```json
var jsonObj = {
    "属性名" : 属性值，
    "属性名" : 属性值，
    "属性名" : 属性值
}；
```

长得有点像map集合。是一种标准数据交换格式，易解析，体积小。

**JSON属性的值可以是任何类型**

属性值当然也可以是一个JSON对象（嵌套），也可以是数组。

```json
 <script type="text/javascript">
        var user = {
            "usercode" : 110,
            "username" : "张三",
            "sex" : true,
            "address" : {
                "city" : "北京",
                "street" : "大兴区"
                "zipcode" : "12345"
            },
            "aihao" : ["smoke","drink","tt"]
        };
        //访问人名以及居住的城市
        alert(user.address + "居住在" +user.address.city);
    </script>
```

## eval函数

作用是：将字符串当作一段JS代码，解释并执行。

```json
windows.eval("var i = 100;");
alert("i = " + i); //i = 100
```

注意，java程序发过来的json格式的仅仅是一个JSON格式的字符串，还不是一个JSON对象。**使用eval函数，可以将json格式的字符串转换成json对象。**
