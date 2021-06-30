#frida主动调用函数
除了使用frida进行hook, 很多场景我们需要用frida主动调用app的java方法和so方法。

因为hook大多数时候只能被动的等待触发，如果函数一直不触发咋办，你就干等着吗。所以主动调用要灵活的多。

大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章

#frida主动调用方法分类
frida主动调用分为下面几种情况
1. frida 主动调用java类方法 （静态java方法）
2. frida 主动调用native类方法 (静态native方法）
---
3. frida 主动调用对象的java方法
4. frida 主动调用对象的native方法
---
5. frida 主动调用so方法
---
这里把类方法和对象方法区分开是因为。
**类方法可以直接调用 对象方法必须要有对象 
区别在于有没有static这个关键字 
有static关键字就是静态方法  没有static就是对象方法**

这里获取对象的方法有两个
1. 直接获取内存中已经有的对象
2. 自己 创建（new）一个

好 下面演示一下5种方式的具体调用代码
##1. frida主动调用类方法(java静态方法)代码
要调用的函数声明如下
```Java
public static String enc(String str_data, int n_conunt)
```
可以看到这个函数是有static关键字的 所以是个类方法（静态方法）
调用格式：
```
类名引用.方法名（参数，...）
```
主动调用代码
```Java
function call_enc(str_data, n_cnt) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;

  Java.perform(function () 
  {
    //打log方便调试
    console.log("===========>on enc");  

    // 获取类
    var obj = Java.use(str_cls_name);

    //调用类方法 因为这里是静态方法 所以可以直接调用
    str_ret = obj.enc(str_data, n_cnt);
    
    //打印结果 方便调试
    console.log("enc result: " + str_ret);
  });
  return str_ret;
}
```
##2. frida 主动调用native类方法 (静态native方法）代码
要调用的函数声明如下
```Java
public static native String c_enc(String str_data)
```

可以看到这个函数是有static关键字的 所以是个类方法（静态方法）这里有native 关键字 所以是个静态的c方法

调用格式：
```
类名引用.方法名（参数，...）
```
主动调用代码
```Java
function call_c_enc(str_data) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;

  Java.perform(function () 
  {
    //打log方便调试
    console.log("===========>on enc");  

    // 获取类
    var obj = Java.use(str_cls_name);

    //调用类方法 因为这里是静态方法 所以可以直接调用
    str_ret = obj.c_enc(str_data);
    
    //打印结果 方便调试
    console.log("enc result: " + str_ret);
  });
  return str_ret;
}
```
##3.frida 主动调用对象的java方法
要调用的函数声明如下
```Java
public  String enc(String str_data)
```
可以看到这个函数是没有有static关键字的 所以是个对象方法（实例方法）
主动调用有2种方式 
1. 直接获取内存中已存在的对象（建议使用）
2. 自己创建一个新对象

**因为运行过程中对象的成员的值可能已经发生了变化，所以如果重新创建一个对象，新对象的值还是初始值，在调用算法或者函数的时候，可能会产生影响。
所以这里建议直接获取内存中已经有的对象**

举个例子 A对象有个成员 m_a 初始值是3
 在app运行过程中，被设置成了666
在调用 A对象算法的时候 m_a参与了运算 
666这个值可能才是算法需要的合法的值。

如果创建一个新对象A1，那么m_a的值是3
这时候调用算法  那么因为m_a的改变 很可能返回一个错误的值

调用格式

```
1.instance.方法名（参数，...）  //直接获取已有对象
2.类名引用.方法名（参数，...） //新创建的对象
```

###1) 直接获取内存中对象主动调用
这里介绍一个frida的api 
```JavaScript
      //从内存中（堆）直接搜索已存在的对象
      Java.choose('xxx.xxx.xxx '， //这里写类名 
      {
        //onMatch 匹配到对象执行的回调函数
        onMatch: function (instance) 
        {
        },
       //堆中搜索完成后执行的回调函数
        onComplete: function () 
        {
        }
      });
```
主动调用代码
```Java
function call_enc(str_data) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;
  
  Java.perform(function () 
  {
      Java.choose(str_cls_name, 
      {
        onMatch: function (instance) 
        {
            //调试用
            console.log("onMatch ");  
            //直接调用对象的函数 instance是找到的对象
            str_ret = instance.enc(str_data);
        },
        onComplete: function () 
        {
        }
      });
  });
  console.log("enc result: " + str_ret);
  return str_ret;
}
```
###2) 创建一个新对象 主动调用代码
这里要用frida创建对象的语法 $new()
```JavaScript
 //获取类的引用
var cls = Java.use('这里写类名');

//调用构造函数 创建新对象  这里注意参数
var obj = cls.$new();
```
这里$new()是调用类的构造函数 新建一个对象 
这里注意两点
1. 如果没有构造函数 或者 构造函数没有参数 这里直接写$new()
2. 如果构造函数有参数，那么一定要填参数 $new(参数，...)

创建一个对象 主动调用代码
```Java
function call_enc(str_data) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;
  
  Java.perform(function () 
  {
       //获取类的引用
       var cls = Java.use(str_cls_name);

       //调用构造函数 创建新对象  这里注意参数
       var obj = cls.$new();
      
       //调用新对象的对象方法 enc
       str_ret = obj.enc(str_data)；
  });
  console.log("enc result: " + str_ret);
  return str_ret;
}
```
##4.frida 主动调用对象的native方法
要调用的函数声明如下
```Java
public native String enc(String str_data, int n_num)
```
可以看到这个函数是没有有static关键字的 所以是个对象方法（实例方法）
主动调用有2种方式 
1. 直接获取内存中已存在的对象（建议使用）
2. 自己创建一个新对象
**建议直接获取内存中已经有的对象**

调用格式
```
1.instance.方法名（参数，...）  //直接获取已有对象
2.类名引用.方法名（参数，...） //新创建的对象
```
###1) 直接获取内存中对象主动调用
主动调用代码
```Java
function call_enc(str_data,  n_num) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;
  
  Java.perform(function () 
  {
      Java.choose(str_cls_name, 
      {
        onMatch: function (instance) 
        {
            //调试用
            console.log("onMatch ");  
            //直接调用对象的函数 instance是找到的对象
            str_ret = instance.enc(str_data, n_num);
        },
        onComplete: function () 
        {
        }
      });
  });
  console.log("enc result: " + str_ret);
  return str_ret;
}
```
###2) 创建一个新对象 主动调用代码
```Java
function call_enc(str_data, n_num) 
{
 //这里写函数对应的类名
  var str_cls_name = "com.wangtietou.test_rpc_all.Test_Enc_Dec";
  //返回值 
  var str_ret = null;
  
  Java.perform(function () 
  {
       //获取类的引用
       var cls = Java.use(str_cls_name);

       //调用构造函数 创建新对象  这里注意参数
       var obj = cls.$new();
      
       //调用新对象的对象方法 enc
       str_ret = obj.enc(str_data, n_num)；
  });
  console.log("enc result: " + str_ret);
  return str_ret;
}
```

##5 frida 主动调用so方法
so层函数声明
```c++
char* c_enc_2(char* p_str_data,  int n_num)
```

这里介绍一个frida用于调用c层函数的API
```Javascript
NativeFunction(address, returnType, argTypes[, abi])
```
1）address：要hook的函数地址
2）returnType：返回值类型
3）argTypes[, abi]: 参数类型 这里参数可以是多个

 frida NativeFunction支持的类型
![image.png](https://upload-images.jianshu.io/upload_images/25193798-7ef326e051b2bd94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 这里参数有两个 一个是 char* 一个是int
大家可以看上面的图 NativeFunction是支持int这个类型的

但是没有char*  对于指针一系列的参数 我们可以用 pointer表示 所以这里的返回值类型就是 'pointer', 
参数类型是['pointer', 'int']

---
注意 JavaScript 是没有char* 这个类型的，所以要通过frida的api去模拟 这里Memory.allocUtf8String()申请空间 存入支付串 模拟char*

```JavaScript
function test_c(str_data, n_num) 
{
    var str_name_so = "libnative-lib.so";    //要hook的so名
    var str_name_func = "c_enc_2";          //要hook的函数名
    
    //获取函数的地址
    var n_addr_func = Module.findExportByName(str_name_so , str_name_func);
    console.log("func addr is ---" + n_addr_func);

    //定义NativeFunction 等下要调用
    var func_c_enc = new NativeFunction(n_addr_func , 'pointer', ['pointer', 'int']);
    
    //调用frida的api申请空间 填入字符串 模拟char*
    var str_data_arg = Memory.allocUtf8String(str_data);
    
     //调用so层的c函数
    var p_str_ret = func_c_enc(str_data_arg, n_num);
    
    //读取字符串  
    var str_ret = Memory.readCString(p_str_ret);
    return str_ret;
}
```

持续更新移动安全，iot安全，编译原理相关原创视频文章