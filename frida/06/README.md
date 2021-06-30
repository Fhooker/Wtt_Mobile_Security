大家使用frida 进行hook的时候大部分时候是用usb数据线去连接。
可是总是有一些比较坑爹的时候  比如设备根本不能连接数据线 比如一些iot设备 或者 数据线连接时断时续比较感人，这个时候就可以用到frida 远程连接了

大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章

### 主要使用场景：
1) usb无法使用 
2)  usb连接不稳定 
3)  usb同时hook多个设备
### 使用依赖
1. pc和远程设备要安装frida环境
2. 处在同一局域网环境下 （非局域网我自己没测试 测试了我再来改）
3. frida_server要以root权限执行

## frida远程连接的两种方式

## 1. 命令行远程连接：
### 1) 远程设备端启动 frida_server 
这里端口可以加参数去修改  如果不加参数默认是 27042
```cmd
//这里的 6666端口可以自定义  不加的话是 27042
riva:/data/local/tmp # ./fs_1413_a64 -l 0.0.0.0:6666
```
### 2) pc端执行 frida -H  主机IP:端口
这里演示一下 查看进程 和 注入脚本的命令
### 查看进程
```cmd
//查看进程的命令
frida-ps -H 192.168.2.102:6666
```
![frida远程连接手机](https://upload-images.jianshu.io/upload_images/25193798-f75feec8772c7f6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 注入脚本
```cmd
//注入脚本的命令
frida -H 192.168.2.102:6666 -f com.wangtietou.test_activity -l C:\\Users\\wangtietou\\Desktop\\hook_activity.js --no-pause
```
这里的脚本很简单  就是在主界面加载的时候 打印一行日志 
====> on create
脚本代码：
```javascript
var str_name_class = "com.wangtietou.test_activity.MainActivity";

Java.perform(function()
{
    var obj = Java.use(str_name_class);

    obj.onCreate.implementation = function (arg)
    {
        console.log('====> on create');
        return this.onCreate(arg);
    }
});
```
执行结果：
![frida局域网连接手机](https://upload-images.jianshu.io/upload_images/25193798-30154ee46e1c3129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

bingo 成功执行

## 2. python脚本远程连接
这里除了命令行直接执行js脚本  大家还经常使用 Python脚本使用frida
这里的脚本是hook Test类的 t1方法  当调用t1方法的时候 会打印一行日志：
====> t1

###  1) usb连接时候的 python脚本的注入代码
```Python
import frida
import sys

rdev = frida.get_usb_device()
session = rdev.attach("com.wangtietou.test_activity")  #要hook的程序包名

scr = """
var str_name_class = "com.wangtietou.test_activity.Test";

Java.perform(function()
{
    var obj = Java.use(str_name_class);

    obj.t1.implementation = function ()
    {
        console.log('====> t1');
        return this.t1();
    }
});
"""

script = session.create_script(scr)
def on_message(message ,data):
    print (message)
script.on("message" , on_message)
script.load()
sys.stdin.read()
```

### 2) 无线连接时候的 python脚本的注入代码
```Python
import frida
import sys

str_host = '192.168.2.102:6666'
manager = frida.get_device_manager()
remote_device = manager.add_remote_device(str_host)
session = remote_device.attach("com.wangtietou.test_activity")

scr = """
var str_name_class = "com.wangtietou.test_activity.Test";

Java.perform(function()
{
    var obj = Java.use(str_name_class);

    obj.t1.implementation = function ()
    {
        console.log('====> t1');
        return this.t1();
    }
});
"""

script = session.create_script(scr)
def on_message(message ,data):
    print (message)
script.on("message" , on_message)
script.load()
sys.stdin.read()
```
这两个脚本大部分代码都是相同的 不同点在于连接设备的代码部分

### 连接usb的设备代码
```Python
rdev = frida.get_usb_device()
session = rdev.attach("com.wangtietou.test_activity") 
```

### 连接远程设备的代码
```Python
str_host = '192.168.2.102:6666'
manager = frida.get_device_manager()
remote_device = manager.add_remote_device(str_host)
session = remote_device.attach("com.wangtietou.test_activity")
```
这里执行注入的python脚本
先启动设备端的frida_server
![frida远程](https://upload-images.jianshu.io/upload_images/25193798-6d2783b7de2411aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
执行远程注入脚本：
![image.png](https://upload-images.jianshu.io/upload_images/25193798-6d1ed7b4ead1a6a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功执行

持续更新移动安全，iot安全，编译原理相关原创视频文章