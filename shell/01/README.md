逆向的时候发现app加了壳, 比如爱加密加固，梆梆加固，或者360之类的 分析个半天,头都秃了还是脱不了怎么办？

这个时候除了回收站, 还可以用youpk。

大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章

## 主流脱壳机对比
下面是我整理的一份常用的脱壳机的对比
![爱加密脱壳实战 梆梆脱壳](https://upload-images.jianshu.io/upload_images/25193798-96d6136bb3afdb9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 实战演示
## 梆梆脱壳实战
这里在网上找了梆梆2020 加固过的样本。
## apk加壳前代码
![梆梆脱壳教程](https://upload-images.jianshu.io/upload_images/25193798-b55bef76d3c48221.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里加壳前的代码比较简单 只有一个类MainActivity  MainActivity 里只有简单的add 和 sub方法。可以看到方法里面的代码也比较简单，返回相加相减的值。

## 梆梆加壳后
![梆梆脱壳](https://upload-images.jianshu.io/upload_images/25193798-6d1422aca2df3768.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里可以看到  加壳后 MainActivity 这个类已经看不到了  

全是梆梆加固的相关类方法

这里 比较厉害的大佬可能直接硬钢逆向了，分析so层，然后一步步dumpdex。

另一些跟我一样懒得大佬，可以直接上工具了。用脱壳机，真香！！！！

好，上youpk.

## youpk
youpk相关介绍：[https://bbs.pediy.com/thread-259854.htm](https://bbs.pediy.com/thread-259854.htm)
youpk github :      [https://github.com/Youlor/Youpk](https://github.com/Youlor/Youpk)

## youpk 实战脱壳梆梆样本
1)  第一步：往配置文件 /data/local/tmp/unpacker.config 写入要脱壳的包名。
    这里脱壳机会读取文件内容执行脱壳

    这里样本的包名是 com.example.test_shell
   往配置文件写入包名 如下图

![梆梆脱壳工具](https://upload-images.jianshu.io/upload_images/25193798-4bdaa540df93bd81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
具体命令如下
```
sailfish:/ # cd /data/local/tmp
sailfish:/data/local/tmp # 
sailfish:/data/local/tmp # echo 'com.example.test_shell' > unpacker.config
sailfish:/data/local/tmp # 
sailfish:/data/local/tmp # cat unpacker.config 
com.example.test_shell
```
2）第二步：启动app
这里测试机已经安装了测试app 点击图标启动

![image.png](https://upload-images.jianshu.io/upload_images/25193798-098073ee5a2f341a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3）第三步：进入脱出dex的目录 一般为 /data/data/包名/unpacker/dex 获取脱壳后的dex

这里目录为
```
/data/data/com.example.test_shell/unpacker/dex
```
进入目录 取出dex 如下图  这里可以看到dex有两个，脱壳机会把外壳dex和真正的dex都脱下来，后续打开一个一个辨别就可以了

**小窍门：带有base.apk这种dex的一般都是外壳dex**
![梆梆脱壳教程](https://upload-images.jianshu.io/upload_images/25193798-537689040b76f5f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好的 成功脱壳。
此刻日志里也看到输出了  unpack end的字样

这里，有些加壳后的apk是没有日志的，因为hook了打印日志的函数，所以看不到，不过不影响脱壳机脱壳

![爱加密破解](https://upload-images.jianshu.io/upload_images/25193798-3bfcc061a5e28daf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把拖出来的dex反编译看下 坐下对比
![梆梆破解](https://upload-images.jianshu.io/upload_images/25193798-2b64a803fec7011f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里可以看到完美dump出dex。
注意 这里只是dump出dex, 还原了三代壳的方法抽取

对于后续四代壳五代壳的 dexvmp sovmp并无相关处理。

## 爱加密脱壳实战
这里爱加密壳找的是网上的一个例子 这里并未找到加固前的样本
 

## 爱加密加固后反编译代码：
![爱加密脱壳工具](https://upload-images.jianshu.io/upload_images/25193798-14448c60358b1d97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 运行后界面
![爱加密实战脱壳](https://upload-images.jianshu.io/upload_images/25193798-fb2e5b4f068ceb80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 按照上面办法脱壳后的dex
这里相关步骤照搬一遍上面的 也就是写入配置文件的包名改一下，基本没啥不同

dump后的dex如下图
![爱加密脱壳i](https://upload-images.jianshu.io/upload_images/25193798-596d30a69ac79338.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里可以看到 dex里面 之前被抽取的方法 已经成功回填

