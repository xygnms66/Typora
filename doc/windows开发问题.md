# puzzle

## 运行reId的客户端

![image-20210818104711780](E:\Document\Typora\img\image-20210818104711780.png)

安装**vcruntime140_app.dll**

regsvr32

![image-20210818104639116](E:\Document\Typora\img\image-20210818104639116.png)

安装相关运行时库



![image-20210818134825277](E:\Document\Typora\img\image-20210818134825277.png)

安装`Visual C++ Redistributable for Visual Studio 2015`

旧版本没办法覆盖新的版本

试下安装`Microsoft Visual C++ Redistributable for Visual Studio 2017`

曹卫强给了库，可以运行了



 git checkout -b opencv_cuda10.2 origin/opencv_3.4.14_vs2017_cuda10.2



## cmake生成protobuf库

+ 教程

[protobuf的编译和使用，在windows平台上_斧冰-CSDN博客_protobuf 编译](https://blog.csdn.net/hp_cpp/article/details/81561310)

+ 报错

![image-20210818154957976](E:\Document\Typora\img\image-20210818154957976.png)

cmake的build失败

vs2017少了配置

![image-20210818162803234](E:\Document\Typora\img\image-20210818162803234.png)

+ 生成的protobuf

~~~
D:\dev\code\protobuf_build_64\Release
~~~

+ 安装-->运行`INSTALL`

报错，原因是缺少zlib

+ 安装报错--->编译后的命令不执行

以管理员身份启动vs2017，启动install安装protobuf

+ 最后的安装位置

![image-20210819143811524](E:\Document\Typora\img\image-20210819143811524.png)





## 安装zlib

[vs编译zlib生成动态链接库_pxy7896的博客-CSDN博客](https://blog.csdn.net/pxy7896/article/details/107359719?utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-6.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-6.control)

zlib生成的库居然叫zlibwap.dll，草，wap是什么鬼

## 编译grpc

+ 编译报错

+ 重新下载子模块

+ 下载子模块报错

## windows下的preprocess项目

### preprocess项目生成报错

+ 生成报错找不到CUDA

![image-20210819151153189](E:\Document\Typora\img\image-20210819151153189.png)

**解决：**

在配置时候点高级模式，添加`CUDA_TOOLKIT_ROOT_DIR`变量作为CUDA路径

CUDA_TOOLKIT_ROOT_DIR:

~~~
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2
~~~

+ 报错

![image-20210823103630243](E:\Document\Typora\img\image-20210823103630243.png)



+ 生成报错

![image-20210819110714161](E:\Document\Typora\img\image-20210819110714161.png)

**解决：**

去掉cmakelist里面的set的`CUDA_TOOLKIT_ROOT_DIR`

+ 生成报错

![image-20210819151931003](E:\Document\Typora\img\image-20210819151931003.png)

+ 编译报错

protoc不一致，因为用的protoc.exe根据脚本文件生成对应服务类和消息类，当前问题出在msys2安装了protoc。



### 生成问题最终解决方案

gitlab下载最新的三方文件，protobuf使用，preprocess不采用find package方法链接。

有以下报错

![image-20210824094842129](E:\Document\Typora\img\image-20210824094842129.png)

### preprocess项目编译报错



+ 预定义

配置预处理器定义_WIN32_WINNT=0x0A00，否则会报“Please compile grpc with _WIN32_WINNT of at least 0x600 (aka Windows Vista)”的错误。



+ 大量未定义文件

因为grpc是release的，所以debug无法使用



+ 链接问题

![image-20210820123644964](E:\Document\Typora\img\image-20210820123644964.png)

1. LNK2019

缺少链接的文件

这个问题在于，接口类本身没有定义内容，然后继承了QThread，因为有OBJECT的原因，通过moc可以生成对应的OBJ文件，但是继承纯虚接口的类反而没有生成对应的OBJ文件，缺少中间件，造成的编译失败。



2. LNK2001

该问题由Qt的moc造成

[ VS+Qt5生成moc文件_Bryan要加油-CSDN博客](https://blog.csdn.net/sjt19910311/article/details/49794631)

[vs2010+qt4编译出现error LNK2001: 无法解析的外部符号 "public: virtual struct QMetaObject等错误_sxj的专栏-CSDN博客](https://blog.csdn.net/sunxiaoju/article/details/48316271)

运行

moc "D:\dev\code\preprocess\src\preprocess\ZJReidPreprocessBase.h" -o "D:\dev\code\preprocess\build_vs2017\preprocess_autogen\include_Release\moc_ZJReidPreprocessBase.cpp"

moc "D:\dev\code\preprocess\src\task\ZJReidTaskInterface.h" -o "D:\dev\code\preprocess\build_vs2017\preprocess_autogen\include_Release\moc_ZJReidTaskInterface.cpp"



moc "D:\dev\preprocess\src\preprocess\ZJReidParallelPreprocessor.h" -o "D:\dev\preprocess\build_vs2017\preprocess_autogen\include_Release\moc_ZJReidParallelPreprocessor.cpp"



+ 链接问题

![image-20210824104355579](E:\Document\Typora\img\image-20210824104355579.png)



### preprocess项目运行报错

![image-20210823141332636](E:\Document\Typora\img\image-20210823141332636.png)

![image-20210823141631039](E:\Document\Typora\img\image-20210823141631039.png)

全局搜路径，找到对应文件位置



## windows下的grpc项目

### 生成



![image-20210823143932496](E:\Document\Typora\img\image-20210823143932496.png)

![image-20210823144103956](E:\Document\Typora\img\image-20210823144103956.png)

缺少golang的安装

[Windows平台C++ 使用VS2015 编译gRPC(总结)_atceedsun的专栏-CSDN博客](https://blog.csdn.net/atceedsun/article/details/102967088)



![image-20210823172715038](E:\Document\Typora\img\image-20210823172715038.png)

[CMake安装grpc生成gRPCTargets.cmake文件 - 从此寂静无声 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jason1990/p/10381320.html)

### 编译

+ 缺少zlib的库

放文件进去

+ 编译失败的项目加include和链接库路径

~~~
..\third_party\zlib\lib\Debug\zlibwapi.lib
~~~



+ 链接问题

![image-20210823155917152](E:\Document\Typora\img\image-20210823155917152.png)

[error LNK2019: 无法解析的外部符号 _deflate_friendan的专栏-CSDN博客](https://blog.csdn.net/friendan/article/details/40042885)



### 运行

+ 启动项

  ![image-20210825111951263](E:\Document\Typora\img\image-20210825111951263.png)

  

  解决

![image-20210825111855683](E:\Document\Typora\img\image-20210825111855683.png)





# 查看dll文件和lib文件的依赖库

+ problem

~~~
bash: dumpbin: command not found
~~~

需要使用VS2017提供的“选择适用于VS2017的x86_x64兼容工具命令提示”



[Linux/Windows 实用工具简记 - 浩月星空 - 博客园 (cnblogs.com)](https://www.cnblogs.com/haomiao/p/6442507.html)

[VS2017利用dumpbin工具查看exe和dll的依赖项或提供的API_mengmeng0510的博客-CSDN博客](https://blog.csdn.net/mengmeng0510/article/details/89957151?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.essearch_pc_relevant&spm=1001.2101.3001.4242)



+ 查看模块依赖于哪些DLL文件

~~~
dumpbin /exports a.dll
~~~

+ 查看b.exe中加载了哪些动态库，可以使用

~~~
dumpbin /imports b.exe
~~~

+ 如果查看c.lib中包含哪些函数

~~~
dumpbin /all /rawdata:none c.lib
~~~

+ 查询位数

在查询结果中如果32BITREQ的值为0表示dll是64位，1表示dll是32位。 通过VS命令提示符中的dumpbin命令可以查看.net和非.net的dll的位数，查询命令是：

~~~
dumpbin /headers C:\Temp\Oracle.DataAccess.dll
~~~



# Visual Studio



## c语言头文件最后一行必须是空行



## gRPC的Debug和Release

+ 区别

1. bin文件里面的exe大小不一样，debug的大一些，release小一点
2. cmake文件的target多了debug和relese的区别

![image-20210826112536903](E:\Document\Typora\img\image-20210826112536903.png)

+ lib文件有的是lib大小不一样，有的是名称不同

![image-20210826112721402](E:\Document\Typora\img\image-20210826112721402.png)





## 快捷键

ctrl + -  回退上一层



D:\dev\code\grpc\third_party\zlib\include

## 运行时库版本

![image-20210823151648612](E:\Document\Typora\img\image-20210823151648612.png)



## VS和运行时库的对应关系

| VS   | 14   | 2015 |
| ---- | ---- | ---- |
| VS   | 15   | 2017 |
| VS   | 16   | 2019 |

## 项目宏定义

`$(Platform)` ----  x64/win32

`$(Configuration)` ----  Debug/Release

`$(ProjectDir)`----项目路径

设置输出目录

~~~
$(ProjectDir)..\windows\$(Platform)\$(Configuration)
~~~





### 设置输出目录



## stdcall和_cdecl

cdecl(C declaration，即C声明)是源起[C语言](https://zh.wikipedia.org/wiki/C语言)的一种调用约定，也是C语言的事实上的标准。

**stdcall是由微软创建的调用约定，是Windows API的标准调用约定。**





## 内存泄露检测工具（VLD）

[Visual Leak Detector for Visual C++ 2008-2015 - CodePlex Archive](https://archive.codeplex.com/?p=vld)

[ VS2017使用Visual Leak Detector（VLD）查找内存泄露_u014077038的博客-CSDN博客](https://blog.csdn.net/u014077038/article/details/100057512)





# msys2交叉编译

msys2安装完成后，开始菜单会有三个启动方式：

- MSYS2 MSYS
- MSYS2 MinGW 64bit
- MSYS2 MINGW 32bit

三种启动方式区别主要在于编译环境软件包的不同，如gcc，clang等版本不同。通用的工具如：grep,git,vim,emacs等等在三种方式内都是一样的。











# cmake生成vsproject



# windows环境

## 转义字符

c++中\\是一种转义字符，他表示一个\，就像\n表示回车一样。

所以C++中的路径名：

D:\matcom45\doc\users\_themes\m.dat

应为：

CString filename=_T("D:\\matcom45\\doc\\users\\_themes\\m.dat");或

CStringfilename=_T("D:/matcom45/doc/users/_themes/m.dat");

在Unix/Linux中，路径的分隔采用正斜杠"/"，比如"/home/hutaow"；而在Windows中，路径分隔采用反斜杠"\"，比如"C:\Windows\System"。



## CUDA安装

![image-20210818190200583](E:\Document\Typora\img\image-20210818190200583.png)





#
