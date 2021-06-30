大家好，我是王铁头 一个乙方安全公司搬砖的菜鸡
持续更新移动安全，iot安全，编译原理相关原创视频文章

**写文章初衷**：
做完 frida rpc远程调用的视频后。
很多大佬表示  搭建arida环境的时候 遇到了很多坑 没有解决
![frida arida安装报错](https://upload-images.jianshu.io/upload_images/25193798-7afbdf8e034f025b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所以我就水一个视频

---

arida是一个好用的用于frida rpc的项目
项目地址：[https://github.com/lateautumn4lin/arida](https://github.com/lateautumn4lin/arida)
原项目相关：
原贴对于arida的安装描述十分简单，以至于普通群众，第一次按照文档大概率掉到坑里。一次就安装成功的几率还是比较小
![原项目 arida安装](https://upload-images.jianshu.io/upload_images/25193798-6f11c7c2d83eb9da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我这里在 windows 和 ubuntu上都安装了一波。
这里详细演示下ubuntu下安装 arida 相关过程。

这里梳理下ubuntu下arida环境安装的流程
1. ubuntu安装miniconda  (conda虚拟环境的精简版)
2. 配置conda软件源
3. 下载arida代码  
4. 创建conda环境 安装arida需要的python支持库
5. 安装arida环境需要的nodejs相关支持库

# ubuntu安装 arida
## 1.ubuntu安装conda虚拟环境  
这里为啥要安装conda?
这里可以把conda理解成一个虚拟机一样的东西，是一个虚拟环境，在里面安装软件后，是这个环境所私有的，对主机环境和别的虚拟环境没有影响。

比如我们要用两个版本的frida，12.8.0 和 14.11.17
因为12.8.0比较稳定，14.11.17比较新，有一些新的功能，
你两个frida都要用咋办，用的时候反复卸载安装frida不合适吧。
这个时候就可以创建两个conda环境去解决。一个虚拟环境安装frida 的12.8.0版本，一个虚拟环境安装fruda的14.11.17版本。这样多香。

这里也是考虑到arida会安装一波软件怕给主机环境造成污染，环境冲突，这里就安装miniconda，方便创建一个虚拟环境。
conda下载地址：
[https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)
翻到页面下面 选择这个版本：

```cmd
sh Miniconda3-py38_4.9.2-Linux-x86_64.sh
```
![ubuntu安装 miniconda](https://upload-images.jianshu.io/upload_images/25193798-6f8d67c76612e5dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
按回车键继续安装
![image-20210623214321428](https://upload-images.jianshu.io/upload_images/25193798-c786ed379fb06de0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

阅读完条款 选择 yes 接受软件声明

![ubuntu安装conda接受协议](https://upload-images.jianshu.io/upload_images/25193798-7290787543448b22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

确认安装路径这里直接回车

![miniconda确认安装路径](https://upload-images.jianshu.io/upload_images/25193798-3259a81764676490.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

conda init 这里也输入 yes

![conda init yes](https://upload-images.jianshu.io/upload_images/25193798-353c2748015665d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装完成

![ubuntu安装conda完成](https://upload-images.jianshu.io/upload_images/25193798-b0e7c0b3493e959f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候关掉当前的命令行 打开一个新的命令行
查看是否安装成功

```cmd
conda --version
```
![conda 查看版本](https://upload-images.jianshu.io/upload_images/25193798-9bcedf976140d576.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)安装成功
这里安装完成后  打开新的cmd窗口 在用户名之前 总是有个 （base)阴魂不散
这里是默认进入了 conda的环境

![conda 默认进入base](https://upload-images.jianshu.io/upload_images/25193798-46e0b68ce15efe04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果想关闭 直接执行一行命令 然后重新打开命令行就好

```cmd
conda config --set auto_activate_base false
```
![conda设置不进入base](https://upload-images.jianshu.io/upload_images/25193798-bef162d4f437e592.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
执行命令  重新打开命令行后 恢复正常

![conda ubutnu](https://upload-images.jianshu.io/upload_images/25193798-093eff30903d8d1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. 配置conda 软件源

这里的专业术语是 channels  意思是频道，其实通俗点讲就是下载软件的网址，我这里直接叫软件源
这里设置成北京外国语学院的软件源  因为用清华源的人实在是太多了  让那些用清华源的内卷去吧 我们猥琐发育就可
复制粘贴一波 执行下面的命令就好

```shell
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/main/
```
## 3. 下载arida相关代码
![image.png](https://upload-images.jianshu.io/upload_images/25193798-482cdff7ffadccd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
自己选一个目录  把arida目录下载下来
我这里直接用 git clone
```cmd
git clone https://github.com/lateautumn4lin/arida.git
```
如果网络不好的同学直接执行下面的命令
```cmd
git clone --depth=1 https://github.com/lateautumn4lin/arida.git
```
上面的语句表示浅复制  只下载代码最新的版本 之前的版本一概不管
![image.png](https://upload-images.jianshu.io/upload_images/25193798-6bbf3277e9429979.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好了 成功下载
## 4.创建conda环境 安装arida需要的python支持库
这里要先创建一个虚拟环境，然后在虚拟环境里安装相关库
创建虚拟环境的命令格式是
```shell
conda create -n (环境名) python=(python版本)
```
这里环境名 就叫 wtt_arida  python版本是3.8
```
conda create -n wtt_arida python=3.8
```
![image.png](https://upload-images.jianshu.io/upload_images/25193798-194d483411db0b17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```cmd
conda activate wtt_arida
```
![image.png](https://upload-images.jianshu.io/upload_images/25193798-700f3ab363ada98c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先进入我们自己的创建的 wtt_arida环境
然后安装所需要的库  这样安装的库 都在这个虚拟环境里面  跟主系统隔离

这里安装软件库 最好和主系统隔离，要不然各种软件冲突有的你难受的。
把下面的文本内容保存成一个txt  然后安装
```cmd
autopep8=1.5.4
ca-certificates=2020.7.22
certifi=2020.6.20
click=7.1.2
colorama=0.4.3
fastapi=0.61.1
h11=0.9.0
loguru=0.5.3
openssl=1.1.1g
pip=20.2.2
pycodestyle=2.6.0
pydantic=1.6.1
python=3.8.0
setuptools=49.6.0
six=1.15.0
sqlite=3.33.0
starlette=0.13.6
toml=0.10.1
websockets=8.1
wheel=0.35.1
zlib=1.2.11
```
![image.png](https://upload-images.jianshu.io/upload_images/25193798-8b362451f880dae3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里保存成了 install.txt   保存成txt之后 执行安装命令
```
conda install --yes --file install.txt
```
![image.png](https://upload-images.jianshu.io/upload_images/25193798-18752a86cf35e7d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
出现下面的提示就是安装完成了 
![image.png](https://upload-images.jianshu.io/upload_images/25193798-eb5ad82e9b316bbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有一部分的库 conda的软件源没有 这里用 pip安装
这里最好用 conda环境里的pip安装 这样安装后 软件在conda的虚拟环境里 主环境访问不到 
 如果用主机环境里的pip 那就不是虚拟环境私有的了  后面安装别的软件 容易出现版本冲突
![image.png](https://upload-images.jianshu.io/upload_images/25193798-01902039eb792328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把下面的文本保存成 txt 这里我保存的文件名是 pipinstall.txt
```
 uvicorn==0.11.8
 pyexecjs==1.5.1
 wincertstore==0.2
 win32-setctime==1.0.2
 frida==12.11.17
```
pip install -r pipinstall.txt
![image.png](https://upload-images.jianshu.io/upload_images/25193798-fcc7b47cdf50b2f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ## 5. 安装arida环境需要的nodejs相关支持库
arida执行需要nodejs的环境  所以这里要安装nodejs 和 npm
安装命令
```
sudo apt-get install nodejs npm
```
安装完查看版本 可以看到版本是比较低的 
![image.png](https://upload-images.jianshu.io/upload_images/25193798-e7380e56256f0b41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
arida要求nodejs 必须大于12（作者的git没写 这是我自己踩坑后总结的）  这里是10 显然是不能用的
这里执行2行命令就可以 同时完成 nodejs和npm的更新了
```
sudo npm install n -g  
sudo n stable
```
两条命令主要做的事是  安装一个叫n的工具 
然后 n这个工具 执行命令就可以同时更新  nodejs 和 npm
![image.png](https://upload-images.jianshu.io/upload_images/25193798-93a7f0a095f4a3b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打开一个新的shell 查看版本 
![image.png](https://upload-images.jianshu.io/upload_images/25193798-09ea87231629b6e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里可以看到 nodejs和npm已经成功安装了
更新完之后  安装两个软件库  babel/core babel-loader 
就大功告成了
```shell
npm install @babel/core babel-loader
```
![image.png](https://upload-images.jianshu.io/upload_images/25193798-add81916d4278ac3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到 这里成功安装

##ubuntu启动arida
好的 我们尝试启动一下
这里分为2个步骤
1. 启动frida_server 
![image.png](https://upload-images.jianshu.io/upload_images/25193798-27283ed4b0c16806.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 启动被注入的apk
![image.png](https://upload-images.jianshu.io/upload_images/25193798-f31d3645b219eb72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 进入arida安装目录 启动arida 
```shell
conda activate wtt_arida
uvicorn main:app --reload
```

![image.png](https://upload-images.jianshu.io/upload_images/25193798-78d7dfa26a5e2a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功启动

持续更新移动安全，iot安全，编译原理相关原创视频文章
都看到这里了  确定不点个关注吗？