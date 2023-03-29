## 尚硅谷全套笔记

👉[戳我](https://24kcs.github.io/vue3_study/chapter1/03_HelloWorld.html#%E7%BC%96%E5%86%99-ts-%E7%A8%8B%E5%BA%8F)

## 什么是TS

JS的超集（包含JS内容），只能通过TS的编译器，把TS编译成JS才行，现在谷歌浏览器支持可以直接运行了。

`npm install -g typescript`cmd命令行终端全局安装

`tsc- v`查看版本号，我的是👉**5.0.2**

如果ts文件中只有单纯的js代码在浏览器可以正常使用不会报错。

get-ExecutionPolicy

## PowerShell管理员身份

Win+R输入`PowerShell` 后输入`start-process PowerShell -verb runas`启动管理员身份。

## 报错： Root file specified for compilation

始终没有解决问题，再尝试了[P5](1NR4y1x7Ab?p=5&spm_id_from=pageDriver&vd_source=baf5a4288b31a243832175ddb5cbd481)的vscode自动编译ts代码后可以运行，这个问题目前先暂时搁置了。相关配置也请看视频。

```html
(()=>{
    function sayHi(str){
    return '嗨'+str
}
    let text = '小妹'
    console.log(sayHi(text))
})();
```

### vscode自动编译小问题：类型注解

类型注解是一种轻量级的，为函数或变量添加的约束。

js是弱类型语言，再ts中编译报错的事情在js中有时是可以的。

ts提供了静态分析的能力。

（其实就是定义了string类型的对象，而不能传成数组这个意思）

## 接口

可以当成一种约束。

```ts
//接口：是一种能力，一种约束而已
(()=>{
    //定义一个接口
    interface IPerson{
    firstName:string
    lastName:string
    }
    //函数👇
    function showFullName(person:IPerson){
        return person.firstName+ '__'+person.lastName
    }
    //对象👇
    const person ={
        firstName:"东方",
        lastName:"不败"
    }
    console.log(showFullName(person))
})()
```

index.html文件中引入👇:

```html
<script src="./js/02_接口.js"></script>
```

（注意这个是在P5的自动编译中手动修改路径后生成的目录地址）

## 类（js和ts可一致）

```ts
//ts中书写js中的类👇
(()=>{
    //定义一个接口
    interface IPerson{
        firstName:string
        lastName:string 
    }
    //定义一个类型
    class Person{
    //定义公共字段（属性）
    firstName:string
    lastName:string
    fullName:string
    //定义一个构造器函数
    constructor (firstName:string,lastName:string){
        this.firstName = firstName
        this.lastName = lastName
        this.fullName = this.firstName + '_' + this.lastName
    }
    }
    //定义个函数
    function showFullName(person:IPerson){
    return person.firstName + '__' + person.lastName
    }
    //实例化对象
    const person = new Person('斋宫','宗')
    console.log(showFullName(person))

})()
```

## 使用webpack打包TS

跟着视频进行相关配置。

在**该文件夹** 下开启终端，先后输入`npm init -y`和`tsc --init`

（前者产生pacakage.json文件，后者生成tsconfig.json文件）

下载依赖👇（此处采用npm install方式）

手动修改web版本，不用最新的，而改成4.41.5如下：

![chrome_qkeaXdvbeF.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_qkeaXdvbeF.png)

现在安装html-webpack-plugin clean-webpack-plugin 默认是最新的，但是最新的会和webpack版本冲突。解决方法是安装版本一致的 。下面代码👇  
`npm install -D html-webpack-plugin@4.5.0 clean-webpack-plugin@3.0.0`

`npm install -D ts-loader@9.1`

报错的安装 npm install ts-loader@~8.2.0(还是不行)

npm run dev运行不起来

![chrome_tlqx3qsXKg.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_tlqx3qsXKg.png)

网课版本配置（在package.json中查看）

还是失败，暂时不打包运行了。

### 成功方法：

换成手脚架cli了。根据[视频](https://www.bilibili.com/video/BV18K4y1T7Nc/?spm_id_from=333.999.0.0&vd_source=baf5a4288b31a243832175ddb5cbd481)创了一个vue3demo案例，放在E:\Vue3Demo，每次启动运行时，我会在文件输入框里打上cmd回车（这样是在该目录下用管理员身份执行），再`cd vue3-demo/`，切换到该目录下的直径子目录。

此时可以使用`yarn serve`命令（我没装环境啊好像，这个是运行不了的）或者`npm run build`或`npm run serve`（这里用了最后一种）去启动它。得到效果是一样的~

此时你的vue3运行成功了。
