# search代码理解

## 服务端

地址：10.11.44.52

端口：8091



## 配置环境

+ 安装cmake

官网  [Download | CMake](https://cmake.org/download/)

Source Distribution 和 Binary Distribution，前者是源代码版，你需要自己编译成可执行软件。后者是已经编译好的可执行版，直接可以拿来用的。

+ 安装protobuf   [Ubuntu16.04安装Protobuf - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/113809843)

~~~
protoc --version   // 查看protoc版本
~~~

[关于使用protobuf出现undefined reference 的问题_jaybroker的博客-CSDN博客](https://blog.csdn.net/qinjie6839000/article/details/53573629)

+ 安装tensorTF   下载路径[third_party / TensorRT · GitLab](http://10.11.12.216/third_party/tensorrt)  

解压后移动到`/usr/local`文件下面即可

+ 安装QT    解压文件夹，修改cmake的格式
+ 安装opencv

步骤如下

~~~
cd opencv
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D \
CMAKE_INSTALL_PREFIX=/usr/local ..
~~~

报错   `You should create a separate directory for build files.`

解决   删除opencv文件夹里的CMakeCache.txt



## 编译报错

+ problem-1

~~~
CMake Error at /home/priya/grpc/examples/cpp/cmake/common.cmake:101 (find_package):
  Could not find a package configuration file provided by "Protobuf" with any
  of the following names:

    ProtobufConfig.cmake
    protobuf-config.cmake
~~~

跟`cmakelist.txt`的module和config模式有关系。

**slove**

去掉**CONFIG**配置

```
-find_package(Protobuf CONFIG REQUIRED)
+find_package(Protobuf REQUIRED)
```

+ problem-2

~~~
CMake Error at /usr/local/lib/cmake/grpc/gRPCConfig.cmake:13 (include):
  include could not find load file:

    /usr/local/lib/cmake/grpc/gRPCTargets.cmake
~~~

**slove**

[ubuntu16.04docker中安装gRPC - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/360101346)

grpc要用高版本的cmake创建，所以grpc没有安装成功。



+ 缺少头文件

~~~
include_directories(./src)
~~~

添加头文件，当前project文件

+ 缺少特殊头文件

```
/usr/local/include/grpcpp/impl/codegen/sync.h:35:10: fatal error: absl/synchronization/mutex.h: No such file or directory
 #include "absl/synchronization/mutex.h"
```

slove

~~~
cp -r /data/dev-lxy/grpc/third_party/abseil-cpp/absl /usr/local/include/
~~~

+ protoc的undefined reference问题

解决办法

[关于使用protobuf出现undefined reference 的问题_jaybroker的博客-CSDN博客](https://blog.csdn.net/qinjie6839000/article/details/53573629)

### opencv的依赖项安装

`libopenexr22`    `libdc1394.so.22`   `libIlmImf.so.2`

libIlmImf动态库包含在libopenexr

安装命令

~~~
sudo apt-get install libopenexr22
~~~

~~~
sudo apt-get install libdc1394-22-dev libdc1394-22 libdc1394-utils
~~~











## 待强化的知识点

+ 关于cmake文件的构建，build文件写的是什么
+ 如何通过脚本自动下载和配置相关依赖
+ 静态单例模式
+ grpc接口如何使用

