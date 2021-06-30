大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章 欢迎关注

今天分享的是frida 堆栈打印

## 1.java层堆栈打印相关

java层堆栈打印的例子网上资料比较全，但是在实际应用的时候，因为frida版本的区别,会发现一些不便

所以这里我针对网上的两个例子做了一些改良。

网上的两个例子如下 

### 1.1 java堆栈打印 例子1

```
function showStacks1() 
{
    send(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Exception").$new()));
}  

```

在新版frida 1405版本环境下做测试

效果图如下

![image](https://upload-images.jianshu.io/upload_images/25193798-70f53e0c2ceff859.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2 java堆栈打印 例子2

```
  function printStack(name) 
  {
    Java.perform(function () 
    {
        var Exception = Java.use("java.lang.Exception");
        var ins = Exception.$new("Exception");
        var straces = ins.getStackTrace();
        if (straces != undefined && straces != null) 
        {
            var strace = straces.toString();
            var replaceStr = strace.replace(/,/g, "\\n");
            console.log("=============================" + name + " Stack strat=======================");
            console.log(replaceStr);
            console.log("=============================" + name + " Stack end=======================\r\n");
            Exception.$dispose();
        }
    });
}

```

在新版frida 1405版本环境下做测试

效果图如下

![image](https://upload-images.jianshu.io/upload_images/25193798-2733951709a4113d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3 java堆栈打印 例子3 改良版

在用了网上两个在新版frida不能换行的例子后 我这里做了改良 代码如下

```
 function showStacks3(str_tag) 
 {
    var Exception=  Java.use("java.lang.Exception");
    var ins = Exception.$new("Exception");
    var straces = ins.getStackTrace();
    
    if (undefined == straces || null  == straces) 
    {
        return;
    }

    console.log("=============================" + str_tag + " Stack strat=======================");

    console.log("");

    for (var i = 0; i < straces.length; i++)
    {
        var str = "   " + straces[i].toString();
        console.log(str);
    }

    console.log("");
    console.log("=============================" + str_tag + " Stack end=======================\r\n");
    Exception.$dispose();
);
```

在新版frida 1289 1405版本环境下做测试

效果图如下

![image](https://upload-images.jianshu.io/upload_images/25193798-6c336a309395f8bd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.c层堆栈打印相关

c层堆栈打印比较简单 网上的代码用起来效果还可以

##2.1网上打印c层堆栈的代码

```
Interceptor.attach(f, {
  onEnter: function (args) {
    console.log('RegisterNatives called from:\n' +
        Thread.backtrace(this.context, Backtracer.ACCURATE)
        .map(DebugSymbol.fromAddress).join('\n') + '\n');
  }
});

```

### 2.2真正有用的一行

上面写了那么多 其实真正起作用的是这一行代码

```

Thread.backtrace(this.context, Backtracer.ACCURATE)
        .map(DebugSymbol.fromAddress).join('\n') + '\n');

```

### 2.3 封装后的代码

```

//context 这里传入this.context
//str_arg 这里传入堆栈显示时展示的标签

function print_c_stack(context, str_tag)
{
    console.log('');

    console.log("=============================" + str_tag + " Stack strat=======================");       

    console.log(Thread.backtrace(context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n'));

    console.log("=============================" + str_tag + " Stack end  =======================");

}

```

### 2.4 例子代码

```
Java.perform(function () 
{
    console.log("hook c start");

    var str_name_so = "libc.so";
    var str_name_func = "fgets";

    //********************************hook native*********************************//

    //hook export function
    var func_ptr = Module.findExportByName(str_name_so , str_name_func);

    if (null == func_ptr)
    {
        console.log(str_name_func + " point is null");
        return;
    }

    Interceptor.attach(func_ptr, 
    {
        onEnter: function(args) 
        {
            print_c_stack(this.context, str_name_func);
        },

        onLeave:function(retval)
        {

        }
    });   
});

function print_c_stack(context, str_tag)
{
    console.log('');

    console.log("=============================" + str_tag + " Stack strat=======================");       
    console.log(Thread.backtrace(context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n'));

    console.log("=============================" + str_tag + " Stack end  =======================");
}
```

效果如下

![image](https://upload-images.jianshu.io/upload_images/25193798-7672456af5832c7d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

