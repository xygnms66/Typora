# 2021/8/4 入门

# docker使用

安装软件包，以允许apt通过HTTPS使用存储库

sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

首先，我们添加阿里云提供的镜像源以便于加快国内安装速度，先添加相应的密钥：

```
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

### curl扩展

curl 是常用的命令行工具，用来请求 Web 服务器。

  `-f`参数用来向服务器上传二进制文件。

  `-s`参数将不输出错误和进度信息。

  `-S`参数指定只输出错误信息，通常与`-s`一起使用。

  `-L`参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

### 使用阿里云镜像源

国内拉取 Docker Hub 的速度非常慢，好在阿里云提供了镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件【可跳过步骤】：

```bash
sudo vim /etc/docker/daemon.json
```

然后加入如下内容【可跳过步骤】：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}
```

修改之后，需要重启 Docker 服务，让修改生效。使用如下命令【可跳过步骤】：

```bash
sudo service docker restart
```

### docker命令

~~~
docker version          #显示docker版本信息
docker info             #显示docker的系统信息，包括镜像和容器数量
docker --help           #帮助文档 
~~~

#### docker镜像命令

~~~python
docker imager           #查看本地主机的所有镜像
docker search           #搜镜镜像
docker pull             #下载镜像
docker rmi              #删除镜像

docker  rmi -f 镜像Id 删除指定镜像
docker  rmi -f  镜像Id  镜像Id  镜像Id 删除多个镜像
docker rmi -f $(docker images -qa) 删除所有镜像

docker system info      #查看系统相关信息
~~~

![image-20210804172826405](E:\Document\Typora\img\image-20210804172826405.png)



~~~
# 使用 Commands 分组中的命令创建一个新的容器
docker create
# 使用 Management Commands 分组中的命令创建一个新的容器
docker container create

# 显示容器列表
docker ps
docker container ls

# 在一个新的容器中运行一个命令
docker run
docker container run
~~~

### docker run

~~~
# Management Commands
docker container run [OPTIONS] IMAGE [COMMAND [ARGS...]]

# 旧命令格式如下：
docker run [OPTIONS] IMAGE [COMMAND [ARGS...]]
~~~

一些常用的配置项为：

- `-i` 或 `--interactive`， 交互模式。
- `-t` 或 `--tty`， 分配一个 `pseudo-TTY`，即伪终端。
- `-d` 或 `--detach`，在后台运行容器并输出容器的 `ID` 到终端。
- `--rm` 在容器退出后自动移除。
- `-p` 将容器的端口映射到主机。
- `-v` 或 `--volume`， 指定数据卷。

这些配置项对于上述的两个命令（Management Commands 和旧命令）都是有效的，在后面的内容不会再特殊说明。关于该命令的详细参数较多，并且大多数参数在很多命令中的意义是相同的，将在后面的内容中使用到时进行相应的介绍。

我们指定 `busybox` 镜像，然后运行命令 `echo "hello shiyanlou"` 命令，

~~~
docker container run \
    busybox echo "hello shiyanlou"
~~~

如下所示：

![uid977658-20190426-1556273902835](E:\Document\Typora\img\uid977658-20190426-1556273902835.jpg)

在上图中，我们可以看到该命令执行的过程：

- 对于指定镜像而言，首先会从本地查找，找不到时将会从镜像仓库中下载该镜像
- 镜像下载完成后，通过镜像启动容器，并运行 `echo "hello shiyanlou"` 命令，输出运行结果之后退出。

在执行命令之后，容器就会退出，如果我们需要一个保持运行的容器，最简单的方法就是给这个容器一个可以保持运行的命令或者应用，比如 `bash`，例如我们在 `ubuntu` 容器中运行 `/bin/bash` 命令：

~~~
docker container run -it \
    ubuntu /bin/bash
~~~

对于交互式的进程而言（例如这里的 bash），必须将 `-i` 和 `-t` 参数一起使用，才能为容器进程分配一个伪终端，通常我们会直接使用 `-it`。

如上所示，我们已经进入到分配的终端中了，这时如果我们需要退出 `bash`，可以使用以下两种方式，它们的效果完全不同：

- 直接使用 `exit` 命令，这时候 `bash` 程序终止，容器进入到停止状态。
- 使用组合键退出，容器仍然保持运行的状态，可以再次连接到这个 `bash` 中，组合键是 `ctrl + p` 和 `ctrl + q`。即先同时按下 `ctrl` 和 `p` 键，再同时按 `ctrl` 和 `q` 键，就可以让容器在后台运行。

对于刚刚创建创建的容器，我们输入 `exit` 退出容器，再使用 `docker container ls -a` 查看容器的状态，结果如下图所示：

![2](E:\Document\Typora\img\2.jpg)

然后我们新建一个容器，使用第二种方式退出，创建的方式和刚刚相同。使用 `docker container ls -a` 命令查看容器的状态，可以看到该容器仍然处于运行中：

![3](E:\Document\Typora\img\3.jpg)

实际使用中，我们不必创建容器之后使用组合键 `ctrl + p` 和 `ctrl + q` 来让容器进入后台运行，通常以 `-d` 参数指定容器以后台模式运行：

~~~
# 以后台模式创建并运行一个容器
docker container run \
    -i -t -d \
    ubuntu /bin/bash
~~~

使用 `docker container ls -a` 命令查看容器的状态，可以看到这个容器已经在后台运行。

![4](E:\Document\Typora\img\4.jpg)



### docker create

严格意义上来讲，`docker run` 命令的作用并不是创建一个容器，而是在一个新的容器中运行一个命令。而用于创建一个新容器的命令为

~~~
# Management Commands
docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]

# 旧的命令格式如下：
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
~~~

该命令会在指定的镜像 IMAGE 上创建一个可写容器层，并 **准备** 运行指定的命令。需要着重强调的是，这里是准备运行，并不是立即运行。即该命令只创建容器，并不会运行容器。

一些常见的配置项如下所示：

- `--name` 指定一个容器名称，未指定时，会随机产生一个名字
- `--hostname` 设置容器的主机名
- `--mac-address` 设置 `MAC` 地址
- `--ulimit` 设置 `ulimit` 选项

关于上述提到的 `ulimit`，我们可以通过其对容器运行时的一些资源进行限制。`ulimit` 是一种 Linux 系统的内建功能，一些简单的描述，可以参考 [通过 ulimit 改善系统性能](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/) ，而对于在下面我们将要设置的部分值的含义，可以参考 [How to set ulimit values](https://access.redhat.com/solutions/61334)。

除此之外，关于创建容器，我们还可以设置有关存储和网络的详细内容，将会在下一节的内容中进行介绍。

下面来看一个实例，我们指定容器的名字为 `shiyanlou01`，主机名为 `shiyanlou01`，设置相应的 `MAC` 地址，并通过 `ulimit` 设置最大进程数（`1024:2048` 分别代表软硬资源限制，详细内容可以参考上面的链接），使用 `ubuntu` 的镜像，并运行 `bash`：

```
docker container create \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash
```

![5](E:\Document\Typora\img\5.jpg)

此时，容器创建成功后，会打印该容器的 `ID`，这里需要简单说明一下，在 Docker 中，容器的标识有三种比较常见的标识方式：

- `UUID` 长标识符，例如 `1f6789f885029dbdd4a6426d7b950996a5bcc1ccec9f8185240313aa1badeaff`。
- `UUID` 短标识符，从长标识符开始，只要不与其它标识符冲突，可以从头开始，任意选用位数，例如针对上面的长标识符，可以使用 `1f`，`1f678` 等。
- `Name` 最后一种方式即是使用容器的名字。

在容器创建成功后，我们可以查看其运行状态，使用如下命令：

```
# 此时该容器并未运行，需要使用 -a 参数
docker container ls -a
```

![6](E:\Document\Typora\img\6.jpg)

新创建的容器的状态 (`STATUS`) 为 `Created`，并且其容器名被设置为对应的值，而之前没有指定名字的容器都是随机生成的名字。

### docker启动

~~~
# Management Commands
docker container start [OPTIONS] CONTAINER [CONTAINER...]

# 旧的命令格式如下：
docker start [OPTIONS] CONTAINER [CONTAINER...]
~~~

对于上面我们创建的容器 `shiyanlou01` 而言，此时处于 `Created` 状态，需要使用如下命令启动它：

~~~
docker container start shiyanlou01
~~~

![7](E:\Document\Typora\img\7.jpg)

此时，运行一个容器我们分成了两个步骤，即创建和启动，使用的命令如下【可跳过步骤】：

~~~
# 创建
docker container create \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash

# 启动
docker container start shiyanlou01
~~~

上述的两个命令如果我们使用 `docker container run` 只需要一步即可，即此时 `run` 命令同时完成了 `create` 及 `start` 操作【可跳过步骤】：

~~~
docker container run \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash
~~~







---



# 容器的理解

~~~
$ docker run -it busybox /bin/sh
~~~

这个命令是 Docker 项目最重要的一个操作，即大名鼎鼎的 docker run。

而 -it 参数告诉了 Docker 项目在启动容器后，需要给我们分配一个文本输入 / 输出环境，也就是 TTY，跟容器的标准输入相关联，这样我们就可以和这个 Docker 容器进行交互了。

而 /bin/sh 就是我们要在 Docker 容器里运行的程序。所以，上面这条指令翻译成人类的语言就是：请帮我启动一个容器，在容器里执行 /bin/sh，并且给我分配一个命令行终端跟这个容器交互。

这样，我的 Ubuntu 16.04 机器就变成了一个宿主机，而一个运行着 /bin/sh 的容器，就跑在了这个宿主机里面。

上面的例子和原理，如果你已经玩过 Docker，一定不会感到陌生。

此时，如果我们在容器里执行一下 ps 指令，就会发现一些更有趣的事情：

~~~
/ # ps
PID  USER   TIME COMMAND
  1 root   0:00 /bin/sh
  10 root   0:00 ps
~~~

可以看到，我们在 Docker 里最开始执行的 /bin/sh，就是这个容器内部的第 1 号进程（PID=1），而这个容器里一共只有两个进程在运行。这就意味着，前面执行的 /bin/sh，以及我们刚刚执行的 ps，已经被 Docker 隔离在了一个跟宿主机完全不同的世界当中。这究竟是怎么做到的呢？

本来，每当我们在宿主机上运行了一个 /bin/sh 程序，操作系统都会给它分配一个进程编号，比如 PID=100。这个编号是进程的唯一标识，就像员工的工牌一样。所以 PID=100，可以粗略地理解为这个 /bin/sh 是我们公司里的第 100 号员工，而第 1 号员工就自然是比尔 · 盖茨这样统领全局的人物。

而现在，我们要通过 Docker 把这个 /bin/sh 程序运行在一个容器当中。

这时候，Docker 就会在这个第 100 号员工入职时给他施一个“障眼法”，让他永远看不到前面的其他 99 个员工，更看不到比尔 · 盖茨。这样，他就会错误地以为自己就是公司里的第 1 号员工。这种机制，其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程。

这种技术，就是 Linux 里面的 Namespace 机制。而 Namespace 的使用方式也非常有意思：它其实只是 Linux 创建新进程的一个可选参数。我们知道，在 Linux 系统中创建线程的系统调用是 clone()，（注：clone()是线程操作，但linux 的线程是用进程实现的）比如：

~~~
int pid = clone(main_function, stack_size, SIGCHLD, NULL); 
~~~

这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如

~~~
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
~~~

这时，新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

当然，我们还可以多次执行上面的 clone() 调用，这样就会创建多个 PID Namespace，而每个 Namespace 里的应用进程，都会认为自己是当前容器里的第 1 号进程，它们既看不到宿主机里真正的进程空间，也看不到其他 PID Namespace 里的具体情况。

而除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。

比如，

Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；

Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

这就是 Linux 容器最基本的实现原理了。所以，Docker 容器这个听起来玄而又玄的概念，实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace 所限定的**资源、文件、设备、状态，或者配置**。而对于宿主机以及其他不相关的程序，它就完全看不到了。所以说，容器，其实是一种特殊的进程而已。

五个命名空间：**进程** **网络** **挂载** **宿主** **共享内存**



## 虚拟机与容器

![1](E:\Document\Typora\img\1.webp)

这幅图的左边，画出了虚拟机的工作原理。其中，名为 Hypervisor 的软件是虚拟机最主要的部分。它通过硬件虚拟化功能，模拟出了运行一个操作系统需要的各种硬件，比如 CPU、内存、I/O 设备等等。然后，它在这些虚拟的硬件上安装了一个新的操作系统，即 Guest OS。

这样，用户的应用进程就可以运行在这个虚拟的机器中，它能看到的自然也只有 Guest OS 的文件和目录，以及这个机器里的虚拟设备。这就是为什么虚拟机也能起到将不同的应用进程相互隔离的作用。

而这幅图的右边，则用一个名为 Docker Engine 的软件替换了 Hypervisor。这也是为什么，很多人会把 Docker 项目称为“轻量级”虚拟化技术的原因，实际上就是把虚拟机的概念套在了容器上。



# gitlab基础(内网使用)

公钥匹配        [GitLab使用教程，看这一篇就够了 - 简书 (jianshu.com)](https://www.jianshu.com/p/bf7b09e234c8)

git基础命令   [使用gitlab下载代码（附常用命令）_String_HelloWorld的博客-CSDN博客](https://blog.csdn.net/weixin_44348787/article/details/109643994)

  `apt-get update `     软件更新

 `apt-get install`    git安装

`git clone [path]`   git下载代码



# CMake巩固

### cmake构建规范

~~~
├── bin
│   └── main
├── build
├── CMakeLists.txt
├── include
├── lib
│   ├── cJSON
│   │   ├── CMakeLists.txt
│   │   ├── inc
│   │   │   └── cJSON.h
│   │   ├── lib
│   │   │   ├── libcJSON.a
│   │   │   └── libcJSON.so
│   │   └── src
│   │       └── cJSON.c
│   └── printfhellocmake
│       ├── CMakeLists.txt
│       ├── inc
│       │   └── hellocmake1.h
│       ├── lib
│       │   ├── libprintfhellocmake.a
│       │   └── libprintfhellocmake.so
│       └── src
│           └── hellocmake1.c
└── src
    ├── CMakeLists.txt
    └── main.c
~~~

+ bin : 存放src中编译后的可执行文件

+ build ：构建项目的地方,存放编译后的缓存文件

+ include : 存放src中的头文件

+ lib： 存放库文件

  按照库的名字命名文件夹

  + /lib/库名/src : 存放库的源文件
  + /lib/库名/inc : 存放库的源文件对应的头文件
  + /lib/库名/lib ：编译生成的库文件
  + src：存放项目源文件



### cmake的使用

+ 内部编译：在源码目录编译 结果与源码混在一起，不好区分

cd 源码目录
cmake .                      生成makefile等文件（需要确认cmakelists在此路径中）
make                          生成可执行文件
sudo make install     安装到系统中

内部编译：cmakelists 中的 BINARY__DIR 与 SOURCE_DIR相同

+ 外部编译

cd 源码目录
mkdir build
cd build
cmake …      将上层目录的源码文件编译 编译结果放在build中
（需要确认cmakelists在上层目录路径中）
make 可执行文件生成 在build中

外部编译：cmakelists 中的 BINARY__DIR 是 SOURCE_DIR/build

图形界面：
使用cmake的图形界面也是相同原理，设置source源码路径、build要生成可执行文件的路径，之后configure、generate。



### cmake常见命令

* 指定cmake版本号

```
#指定CMAKE最低版本要求
cmake_minimum_required(VERSION 3.0)
```

+ 指定cmake版本号

~~~
#设定项目名称
project(xxx_ProjectName)
~~~

+ 添加编译命令

~~~
#方式1：查找指定目录下源文件到指定变量中
aux_source_directory(文件夹名称 源文件存放变量名)
#例子1,将driver目录下源文件（*.c或者*.cpp装载到SRC_LIST中）
aux_source_directory(driver SRC_LIST)
#例子2,将当前CMakeLists.txt目录下源文件（*.c或者*.cpp装载到SRC_LIST中）
aux_source_directory(. SRC_LIST)

#方式2：直接设置变量包含的源文件
set(变量名 变量需装载的内容)
#例子1,将原SRC_LIST变量中的内容和1.c 2.c 3.c源文件装载到SRC_LIST变量中去
set(SRC_LIST "${SRC_LIST} 1.c 2.c 3.c")

#方式3：批量添加源文件
#设置源文件目录集合
set(SRC_DIRS "src" "port" "plugins/flash" "plugins/file")
#轮询目录加入源文件到SRC_LIST(局部变量)
foreach(srcdir ${SRC_DIRS})
    aux_source_directory(${srcdir} SRC_LIST)
endforeach()
message("源文件有：${SRC_LIST}")

#方式4：通配符查找
file(GLOB_RECURSE SRC_LIST ${YOUR_PROJECT_SRC_DIR}/ *.c)
~~~

+ 添加编译变量

~~~
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
~~~

+ 配置的内容（待补充）

~~~
CMAKE_INSTALL_PREFIX        #指定安装路径，相当于 --prefix=
CMAKE_CXX_FLAGS            	#指定 CXX 编译器，相当于 CXX=
~~~

#### set

~~~
set(CAFFE_TARGET_VERSION "1.0.0" CACHE STRING "Caffe logical version")
set(CAFFE_TARGET_SOVERSION "1.0.0" CACHE STRING "Caffe soname version")
~~~

`set()`指令是设定变量的名字和取值，`CACHE`意思是缓存类型，是说在外部执行CMake时可以临时指定这一变量的新取值来覆盖cmake脚本中它的取值：`CMAKE -Dvar_name=var_value`。

而最后面的双引号包起来的取值可以认为是”注释“。`STRING`是类型，不过据我目前看到和了解到的，CMake的变量99.9%是字符串类型，而且这个字符串类型变量和字符串数组类型毫无区分。



#### include_directories

添加依赖的头文件



#### 添加动态库

~~~
link_directories(${PROJECT_SOURCE_DIR}/lib)    #添加动态连接库的路径
target_link_libraries(project_name -lmxnet )   #添加libmxnet.so
~~~

#### 添加静态库

~~~
add_library(mxnet STATIC IMPORTED)
set_property(TARGET mxnet PROPERTY IMPORTED_LOCATION /path/to/libmxnet.a)
target_link_libraries(project_name mxnet ) #添加libmxnet.a
~~~

#### CMAKE_PREFIX_PATH

~~~
list(APPEND CMAKE_PREFIX_PATH "/home/zj/work/opencv")
~~~

[c++ - CMakeList 设置 CMAKE_PREFIX_PATH - IT工具网 (coder.work)](https://www.coder.work/article/1709642)

#### set_property

源文件属性通常用来控制文件如何被构建。 一个必定存在的属性是LOCATION。

~~~
set_property(<GLOBAL                            |
                DIRECTORY [dir]                   |
                TARGET    [target1 [target2 ...]] |
                SOURCE    [src1 [src2 ...]]       |
                TEST      [test1 [test2 ...]]     |
                CACHE     [entry1 [entry2 ...]]>
               [APPEND][APPEND_STRING]
               PROPERTY <name>[value1 [value2 ...]])
~~~

GLOBAL域是唯一的，并且不接特殊的任何名字。

DIRECTORY域默认为当前目录，但也可以用全路径或相对路径指定其他的目录（前提是该目录已经被CMake处理）。

TARGET域可命名零或多个已经存在的目标。

SOURCE域可命名零或多个源文件。注意：源文件属性只对在相同目录下的目标是可见的(CMakeLists.txt)。

TEST域可命名零或多个已存在的测试。

CACHE域必须命名零或多个已存在条目的cache.


#### add_definitions()

~~~
add_definitions(-DFOO -DBAR ...)
example:
	add_definitions("-Wall -g")
~~~

Adds -D define flags to the compilation of source files.

命令通常用来添加C/C++中的宏，例如：

+ `add_defitions(-DCPY_ONLY)`给编译器传递了预定义的宏`CPU_ONLY`，相当于代码中增加了一句`#define CPU_ONLY`
+ `add_defitions(-DMAX_PATH_LEN=256)`，则相当于`#define MAX_PATH_LEN 256`
   根据文档，实际上`add_definitions()`可以添加任意的编译器flags，只不过像添加头文件搜索路径等flags被交给`include_directory()`等命令了。



#### list

```cmake
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules)
```

`list(APPEND VAR_NAME VAR_VALUE)`这一用法，表示给变量`VAR_NAME`追加一个元素`VAR_VALUE`。

这里是把项目根目录(CMakeLists.txt在项目根目录，`${PROJECT_SOURCE_DIR}`表示CMakeLists.txt所在目录）下的`cmake/Modules`子目录对应的路径值，追加到`CMAKE_MODULE_PATH`中；`CMAKE_MODULE_PATH`后续可能被`include()`和`find_package()`等命令所使用。



#### include

命令的作用：

- 包含文件，
- 或者，包含模块

所谓包含文件，例如`include(utils.cmake)`，把当前路径下的`utils.cmake`包含进来，基本等同于C/C++中的`#include`指令。通常，`include`文件的话文件应该是带有后缀名的。
 所谓包含模块，比如`include(xxx)`，是说在`CMAKE_MODULE_PATH`变量对应的目录，或者CMake安装包自带的Modules目录（比如mac下brew装的cmake对应的是`/usr/local/share/cmake/Modules`)里面寻找`xxx.cmake`文件。注意，此时不需要写".cmake"这一后缀。

具体的说，这里是把CMake安装包提供的`ExternalProject.cmake`(例如我的是`/usr/local/share/cmake/Modules/ExternalProject.cmake`）文件包含进来。ExternalProject，顾名思义，引入外部工程，各种第三方库什么的都可以考虑用它来弄;



#### aux_source_directory

~~~
aux_source_directory(<dir> <variable>)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
~~~

**搜集所有在指定路径下的源文件的文件名，**将输出结果列表储存在指定的变量中。 



### 特定名称

~~~
${PROJECT_SOURCE_DIR}    表示CMakeLists.txt所在目录
${CMAKE_MODULE_PATH}     后续可能被`include()`和`find_package()`等命令所使用。
~~~



### find_package

[Cmake之深入理解find_package()的用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97369704)

[深入理解CMake(4)：find_package寻找系统Protobuf（apt）的过程分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/2946b0e5c45b)

[“轻松搞定CMake”系列之find_package用法详解_zhanghm1995的博客-CSDN博客](https://blog.csdn.net/zhanghm1995/article/details/105466372)



**每一个依赖库库都直接（在Dependencies.cmake中）或间接（在各自的cmake脚本文件中）使用`find_package()`命令来查找包**。

~~~
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])
~~~

+ 版本选择

如果使用了[version]选项，将会搜索与指定的版本相匹配的包(版本格式为major[.minor[.patch[.tweak]]])。如果使用了EXACT选项，则会搜索与指定版本准确匹配的包。

+ REQUIRED

如果没有使用REQUIRED选项，即使包没有被找到，也不会显示任何信息。使用REQUIRED选项后，如果包没有被找到，就会产生一个错误信息，中断处理。

+ 工作模式

find_package指令有两种查找包的模式：一种是模块(Module)模式，一种是配置(Config)模式。默认情况下，首先使用模块(Module)模式，如果没有找到对应的模块(Module)，就会使用配置(Config)模式。如果使用了MODULE选项，使用模块模式失败后，不会继续使用配置(Config)模式。

这两种工作模式的不同决定了其搜包路径的不同：

+ Module模式
  find_package命令基础工作模式(Basic Signature)，也是默认工作模式。

+ Config模式
  find_package命令高级工作模式(Full Signature)。 只有在find_package()中指定CONFIG、NO_MODULE等关键字，或者Module模式查找失败后才会进入到Config模式。

#### Module模式

**Module**模式下是要查找到名为`Find<PackageName>.cmake`的配置文件。

Module模式只有两个查找路径：**CMAKE_MODULE_PATH**和**CMake**安装路径下的**Modules**目录，
**搜包路径依次为：**

~~~
CMAKE_MODULE_PATH
CMAKE_ROOT
~~~

#### Config模式

**Config**模式下是要查找名为`<PackageName>Config.cmake`或`<lower-case-package-name>-config.cmake`的模块文件。

具体查找顺序为：
1、名为`<PackageName>_DIR`的CMake变量或环境变量路径
默认为空。
这个路径是非根目录路径，需要指定到`<PackageName>Config.cmake`或`<lower-case-package-name>-config.cmake`文件所在目录才能找到。
2、名为`CMAKE_PREFIX_PATH`、`CMAKE_FRAMEWORK_PATH`、`CMAKE_APPBUNDLE_PATH`的CMake变量或环境变量路径。
3、`PATH`环境变量路径
**根目录**，默认为系统环境`PATH`环境变量值。



#### 查找指定包建议

上面的查找规则整体看起来好像很复杂，但其实我们在安装库的时候都会自动配置安装到对的位置，一般都不会出现问题。如果我们需要指定特定的库，我们也只需要设置优先级最高的几个变量名即可。包括下面两种情况：

1、如果你明确知道想要查找的库`<PackageName>Config.cmake`或`<lower-case-package-name>-config.cmake`文件所在路径，为了能够准确定位到这个包，可以直接设置变量`<PackageName>_DIR`为具体路径，如：

```
set(OpenCV_DIR "/home/zhanghm/Softwares/enviroment_config/opencv3_4_4/opencv/build")
1
```

就可以明确需要查找的OpenCV包的路径了。

2、如果你有多个包的配置文件需要查找，可以将这些配置文件都统一放在一个命名为cmake的文件夹下，然后设置变量`CMAKE_PREFIX_PATH`变量指向这个cmake文件夹路径，需要注意根据上述的匹配规则，此时每个包的配置文件需要单独放置在命名为包名的文件夹下（文件夹名不区分大小写），否则会提示找不到。

---

问题

在用cmake编译项目的时候，很多时候需要用find_package来导入一些库，比如opencv，cuda等。但是有时候，下载了预编译好的项目时，怎么手动指定路径呢？

解决方案
通过设定一个project_DIR变量来指定路径，该路径是projectConfig.cmake文件所在的路径，比如下载预编译好的llvm。

~~~
set(LLVM_DIR yourpath/llvm-7.0/lib/cmake/llvm)
find_package(LLVM CONFIG REQUIRED)
~~~

---

### CMake的自定义模块

[CMake教程——如何编写FindXXX.cmake - 简书 (jianshu.com)](https://www.jianshu.com/p/51a483fe58dc)















# Linux精进

### 常忘命令

+ **tar -zxvf 压缩文件名.tar.gz**

+ **设置LD_LIBRARY_PATH**

```
在 ~/.bashrc 或者 ~/.bash_profile 中加入 export 语句，前者在每次登陆和每次打开 shell 都读取一次，后者只在登陆时读取一次。
```

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/Qt5.14.2/5.14.2/gcc_64/lib:/data/dev-lxy/searcher/lib
```





### 不理解的命令

+ **sudo ldconfig**

ldconfig是一个**动态链接库**管理命令

安装完成某个工程后生成许多动态库，为了让这些动态链接库**为系统所共享**，还需运行动态链接库的管理命令–ldconfig。



### 常见报错

#### Unable to locate package

**问题:**  Unable to locate package XXX

方法1： `sudo apt-get update`

方法2：`sudo apt-get upgrade`



# Git使用

## 相关材料

[Git 基础手册](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/)

[Git 里面的 origin 到底代表啥意思? - 知乎 (zhihu.com)](https://www.zhihu.com/question/27712995)

## 背景

版本控制系统（VCS：Version Control System）在软件开发领域有着悠久的历史，由于发源于软件工程师在对源代码的管理，所以也称为源码管理系统（SCM：Source Control System）。



## SVN和GIT

+ svn 是集中式管控：所有库（repo）的内容都在 server 上，离了 server 连 svn log 都看不到，更别说提交代码了。
+ git 是分布式管控：每个 git 项目里面的 `.git` 文件夹中都包括了所有的库（repo）内容，可以看 log、提交代码、创建分支、打 tag 等，两台电脑的 git 库之间是**同步**（sync）的概念，大家都是平等的。可以断网工作，话说当年 Torvalds 创建 git 就是在一列行进多天、没有网络的火车上写出来的



## 版本控制相关软件

1. 2008.2：[Github](http://github.com/)
2. 2009.3：[sourceforge](http://sourceforge.net/)
3. 2011.7：[Google Code](http://code.google.com/) —— 2015 年底已关闭服务
4. 2012.3：[CodePlex](http://www.codeplex.com/)
5. 2013：[gitee（码云/OSChina）](http://gitee.com/) —— 前身是 OSChina Code
6. 2013.10：[CodeChina（CSDN）](http://codechina.csdn.net/) —— 前身是 CSDN Code
7. 2014.2：[gitlab](http://gitlab.com/)
8. 2014：[Coding](https://coding.net/)

Github 在你方便的 git 之外，也增加了权限、Fork、PullRequest（PR）、项目列表、趋势等，在 git 协议的基础上创建了一个庞大的系统。



## github两大工具

+ Github REST API

[GitHub REST API - GitHub Docs](https://docs.github.com/en/rest)

+ Github CLI

CLI：Command-Line Interface，在 Terminal 终端中使用命令来连接、登录 Github，并获取项目的信息、管理 issue、PR 等操作，有了 CLI，用户可以在不使用浏览器登录 Github 网站的情况下，全面管理自己的账号及项目。这为打造独立的 DevOps 工作流、CI/CD 环境提供了非常便捷的手段。

如果说 API 大多是获取 Github 上公开信息的话，CLI 方式则是登录授权后，开发工作的一部分。

[Github CLI 官网](https://github.com/cli/cli)、[Github CLI 源码](https://cli.github.com/)。

~~~
gh <command> <subcommand> [flags]
~~~

5个核心 command

![11](E:\Document\Typora\img\11.png)

6个一般command

![22](E:\Document\Typora\img\22.png)





## 基础命令

### 本地命令整理

![Snipaste_2021-08-13_11-09-41](E:\Document\Typora\img\Snipaste_2021-08-13_11-09-41.png)

+ 配置账号

~~~
git config --global user.name wk**in
git config --global user.emal wk**in27@gmail.com
~~~

![Snipaste_2021-08-13_10-47-16](E:\Document\Typora\img\Snipaste_2021-08-13_10-47-16.png)

+ 初始化

~~~
git init
~~~

+ 显示git状态

~~~
git status [opt]
  -s : short模式显示，即只显示最精简的内容
  -b : 显示分支(branch)，也是git status的参数默认之一
~~~

![Snipaste_2021-08-13_10-39-16](E:\Document\Typora\img\Snipaste_2021-08-13_10-39-16.png)

+ 文件纳入管理

~~~
git add <filename>
~~~

`git add` 将文件添加到暂存区（git 官方又称为索引（index）区），暂存区就像回收站（删除前给你一个检视的机会，多次删除操作放入回收站的文件可以一次清空），多次 add 操作放入暂存区的文件，最后考虑成熟了，就可以提交入本地库了：执行 `git commit` 命令来完成。

+ git commit

~~~
git commit            // 提交且弹出log框
git commit -m "add"   // 提交且log为add
git commit --amend    // 修改log信息
~~~

+ git log

~~~
$ git log
commit a225befe027f2dcaa24ab567ff2c87796aa5fc01 (HEAD -> master)
Author: wkevin <wk**in27@gmail.com>
Date:   Sat Oct 22 19:00:45 2020 +0800

    创建我的日记本
~~~

这是 log 默认的显示方式，包含：

~~~
- hash 码：git 和 svn 不同，没有一个数字递增的节点号，而是一串 40Bytes 的哈希字符，这个 hash 码后面还有大用途，我们这里暂且略过。
- 作者
- 时间
- log 的具体内容
~~~

| `git log`                   | 解释                                                      | 举例                           |
| --------------------------- | --------------------------------------------------------- | ------------------------------ |
| `git log <hash>`            | 从某个提交开始，hash 为该次提交的 hash 码的头 4(或 6)位   | `git log a225`                 |
| `git log <file/folder>`     | **仅查看某个文件/文件夹相关的 log**                       | `git log diary.md`             |
| `git log --stat`            | 把修改了哪些文件也列出来                                  |                                |
| `git log --date=short`      | 日期只显示年月日，可与部分参数配合使用                    |                                |
| `git log --author=<name>`   | 查看某人的 log，name 支持统配符                           | `git log --author=shi.*lou`    |
| `git log --pretty=<format>` | 用 format 美化：oneline、short、fuller、fortmat:string 等 | `git log --pretty=oneline`     |
| `git log --oneline`         | 对上面 oneline pretty 的简写                              |                                |
| `git log --format=<string>` | 对上面 pretty=format:string 的简写                        | `git log --format="%h %ai %s"` |

+ alias

别名（alias）是 linux 系统的基本概念，git 也用它来放权给用户自由定制自己的命令行。

~~~
git config --global alias.st  "status"
~~~

~~~
git config --global --replace-all alias.lg "log --format=format:'%C(auto)%h | %ai %Cred%an%C(auto) | %ci %Cred%cn%C(auto) | %Cgreen %s'"
git config --global --replace-all alias.ls "log --format=format:'%C(auto)%h | %ai %Cred%an%C(auto) | %ci %Cred%cn%C(auto) | %Cgreen %s' --stat"
git config --global --replace-all alias.st "status -sb"
git config --global --replace-all alias.ci commit
git config --global --replace-all alias.co checkout
git config --global --replace-all alias.br "branch -avv"
git config --global --replace-all alias.rt "remote -vv"
git config --global --replace-all alias.cl clone
~~~



### 远程命令整理

+ git clone

~~~
git clone <origin.repo> <local.repo> 
~~~

如果 `<local.repo>` 省略不写，则创建 `<origin.repo>` 的同名文件夹。



+ git pull

~~~
git pull origin(第一个remote默认命名为origin)
git pull <any-remote>     // 多个remote存在时
~~~

从版本库中拉取最新版本代码。



+ 添加remote

~~~
git remote add <name> <url>
~~~

更改remote的名字

~~~
git remote rename origin any-name
~~~



### 创建裸版本库

得到一个裸版本库有几种办法：

1. `git init --bare ...`：直接从零创建一个裸版本库，它是没有 remote 的。
2. `git clone --bare ...`：从远程 clone 一个库到本地，但存储为裸库，无法 checkout 出工作文件，但它是有 remote 的
3. 删除工作文件，只保留 `.git` 文件夹，并修改 `.git/config` 文件中的 `bare = true` —— 纯手工操作，不推荐



### git diff

![Snipaste_2021-08-13_11-26-39](E:\Document\Typora\img\Snipaste_2021-08-13_11-26-39.png)

- `git diff`：已修改，但尚未缓存（即：没有 `git add`）的文件与索引区文件比较差异
- `git diff --cached`：已经缓存，但尚未提交（即：没有 `git commit`）的文件差异
- `git diff <ref.id>`：当前未提交（即上述两种情况的合集）与指定版本`ref.id`之间的差异，其中 ref.id 可以是
  - 某次提交的 hash 值，如：`git diff 6eff`
  - `HEAD`：未提交与版本库里最新版本之间对比差异
  - `HEAD^`：与 HEAD 的前一个版本之间对比差异
- `git diff <ref.id1> <ref.id2>`：两个版本之间的差异



### 分支

git 的分支有以下几类最常用操作：

- 查询：`git branch [-a|v]`：`-a` 会显示 remote 上的分支， `-v` 会显示该分支最新的一次提交
- 创建：`git branch <new.branch>`
- 检出：`git checkout <another.branch>`
- 创建并检出：`git checkout -b <another.branch>`
- 删除：`git branch -d <one.branch>`
- 重命名：`git branch -m <one.branch>`
- 合并：`git merge`

![Snipaste_2021-08-13_11-35-30](E:\Document\Typora\img\Snipaste_2021-08-13_11-35-30.png)









## 创建一个分支代码并进行合并

1. 创建并切换新的分支

~~~
git branch -d dev
~~~

2. **第一次将本地代码更新到远程分支**

~~~
git push --set-upstream origin [分支名]
~~~

3. 将本地代码更新到远程分支

~~~
git push origin master:dev-linux
~~~

origin表示的是本地的仓库名称，如果是`git clone`表示默认创建origin作为本地仓库名称。

以下表示建立仓库的名称为`origin`和`aMao`。

~~~
git remote add origin git@github.com:imki911/myProject.git
git remote add aMao git@github.com:imki911/myProject.git
~~~







# vscode配置

### c_cpp_properties.json

[笔记-vscode下的c_cpp_properties.json文件配置_fdfdsds的博客-CSDN博客_c_cpp_properties.json](https://blog.csdn.net/fdfdsds/article/details/102248876?utm_medium=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-2.test_version_3&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-2.test_version_)

+ env

用户自定义变量,可用于在其他配置属性中进行替换,在本例中myDefaultIncludePath即为用户自定义变量,在configurations.includePath字段下被引用

+ configurations

一组配置对象，向智能感知引擎提供有关你的项目和首选项的信息。默认情况下，扩展插件会根据操作系统自动创建相关信息,我们自定义配置主要就是修改这里

+ version

建议不要编辑这个字段,它跟踪c_cpp_properties.json文件的当前版本,以便扩展插件知道应该显示什么属性和设置，以及如何将该文件升级到最新版本。



### Configuration字段

+ **name**

用来标识配置文件,一般是内核的名字就可以了,如"Linux"。

+ **compilerPath**

用于构建项目的编译器的完整路径，例如/usr/bin/gcc，在交叉编译时,将该字段设置为编译器的绝对路径就行了,例如/root/esp8266/xtensa-lx106-elf/bin/xtensa-lx106-elf-gcc。

+ **compilerArgs**

编译选项,之所以不重要,是因为defines的问题可以用browse解决,而include问题可以用includePath字段解决,该字段可以不写。

+ **intelliSenseMode**

智能感知模式,有msvc-x64.gcc-x64和clang-x64,根据编译器的前端选择就行。

+ **includePath**

添加头文件目录

+ **defines**

用于智能感知引擎在解析文件时使用的预处理程序定义的列表。可以选择使用=设置一个值，例如VERSION=1。

+ **cStandard**

用于智能感知的C语言标准版本,根据实际情况确定。

+ **cppStandard**

用于智能感知的c++语言标准的版本,根据实际情况确定。

+ **browse**

1. path
2. limitSymbolsToIncludedHeaders
3. databaseFilename



# draw.io tips

+ 编辑样式

![1](E:\Document\Typora\img\1.png)

+ 漫画风格

打开编辑样式，输出文本中添加`comic=1` 就是漫画风格了。

+ 拷贝样式

`shift+ctrl+c`拷贝样式，`shift+ctrl+v`粘贴样式



# Typora

表格模板

| 1    | 2    | 3    |
| ---- | ---- | ---- |
| 1    | 2    | 3    |
| 1    | 2    | 3    |





# Question



![2](E:\Document\Typora\img\2.png)

cmake的module的配置在哪里？

能否给个拷贝的内容？

