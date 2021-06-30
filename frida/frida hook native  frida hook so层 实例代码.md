# frida hook native

大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章 

今天分享的是 frida hook native 也就是 frida hook so层函数

frida是一个轻便好用的工具，不仅支持java层的hook,同样支持so层的hook. 那么,frida hook so 是如何实现的哪？

这里，根据场景的不同 分为有导出和无导出

## 原理 通过地址进行hook

有导出：函数名可以在导出表找到 通过导出表的数据结构 用函数名称进行函数的定位
无导出：函数名在导出表找不到。 这里需要根据函数特征 比如字符串等 手动搜索关键字符串定位函数地址

## 1 有导出so层hook

原理：通过导出表的结构找到函数名对应的汇编代码的地址。

![frida hook native frida hook so层](https://upload-images.jianshu.io/upload_images/25193798-1a06f2374dd910d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.1 有导出适用场景

一般情况下,要hook的函数名可以在导出表找到
找不到只有下面两种情况

1.  写代码时使用__attribute__((visibility("hidden")))关键字隐藏导出
2.  编译后 被开发者, 加固壳或者第三方框架修改elf格式 被加密抹去相关信息,

下面先讨论正常可以在导出表看到的情况
这里我写了两个函数

```c++
extern "C" void func_exp()
{
    LOGD("exp");
}

//这里没有extern "C"关键字 默认是c++风格导出的
//hook时要注意名称粉碎 
void func_exp_cpp()
{
    LOGD("exp_cpp");
}

```

这里只要不做啥骚操作 导出表绝对是可以看到的 如下图
![frida hook native](https://upload-images.jianshu.io/upload_images/25193798-f2c0622122e4f815.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.2 frida hook so层有导出代码

这里 func_exp 函数是c风格导出 所以函数名直接填写就可以了
但是 func_exp_cpp 函数这里要进入ida里面看具体函数名 如图
![frida hook so层](https://upload-images.jianshu.io/upload_images/25193798-1194eefa6f9bfc42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```javascript
var str_name_so = "libnative-lib.so";    //要hook的so名
var str_name_func = "func_exp";          //要hook的函数名
//var str_name_func = "_Z12func_exp_cppv";    //这里注意名称粉碎

var n_addr_func = Module.findExportByName(str_name_so , str_name_func);
console.log("func addr is ---" + n_addr_func);

Interceptor.attach(n_addr_func, {
    //在hook函数之前执行的语句
    onEnter: function(args) 
    {
        console.log("hook on enter")
    },
    //在hook函数之后执行的语句
    onLeave:function(retval)
    {
        console.log("hook on leave")
    }
});

```

## 1.3 hook效果图

![frida hook so frida hook native](https://upload-images.jianshu.io/upload_images/25193798-5bfe39986cd154e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2 无导出

无导出这里要使用一个关键字才能达到无导出的效果

```c++
__attribute__((visibility("hidden")))

```

这里写一个例子：

```c++
//extern "C" c语言格式导出
__attribute__((visibility("hidden"))) void func_no_exp()
{
    LOGD("hidden");
}

```

生成so文件后,导出表是看不到这个函数的:
如下图。
![frida hook so层](https://upload-images.jianshu.io/upload_images/25193798-5082f8810226953a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为无导出的函数无法通过函数名去定位地址：
所以这里只能通过手动定位去找到函数对应的偏移 这里可以根据情况用字符串或者看上下级调用定位到偏移 这里的函数偏移是0x7078 还有一点 确定偏移的时候要注意使用的so是v7 arm32 还是 v8 arm64
函数偏移如下 如图 这里 函数 func_no_exp的偏移是 0x7078
![image](https://upload-images.jianshu.io/upload_images/25193798-c16787a4a2bbb3d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种函数在ida里面一般是 sub_xxx xxx是16进制的地址

## 2.2 frida hook so层无导出代码

```javascript
var str_name_so = "libnative-lib.so";    //要hook的so名
var n_addr_func_offset = 0x7078;         //要hook的函数在函数里面的偏移

//加载到内存后 函数地址 = so地址 + 函数偏移
var n_addr_so = Module.findBaseAddress(str_name_so);
var n_addr_func = parseInt(n_addr_so, 16) + n_addr_func_offset;

var ptr_func = new NativePointer(n_addr_func);
Interceptor.attach(ptr_func, 
{
    onEnter: function(args) 
    {
        console.log("hook on enter no exp");
    },
    onLeave:function(retval)
    {
        console.log("hook on Leave no exp");
    }
});

```

## 2.3 frida hook so 无导出效果图

![image](https://upload-images.jianshu.io/upload_images/25193798-942e33125230c3c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

持续更新移动安全，iot安全，编译原理相关原创视频文章
