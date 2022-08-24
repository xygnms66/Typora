# Opencv



## Mat



### 格式

|    属性     |                             说明                             |
| :---------: | :----------------------------------------------------------: |
|    data     | uchar型的指针。Mat类分为了两个部分:矩阵头和指向矩阵数据部分的指针，data就是指向矩阵数据的指针。 |
|    dims     | 矩阵的维度，例如5*6矩阵是二维矩阵，则dims=2，三维矩阵dims=3. |
|    rows     |                          矩阵的行数                          |
|    cols     |                          矩阵的列数                          |
|    size     |                 矩阵的大小，size(cols,rows)                  |
|  channels   |  矩阵元素拥有的通道数，例如常见的彩色图像，每一个像素由RGB   |
|   type()    | 表示了矩阵中元素的类型以及矩阵的通道个数，它是一系列的预定义的常量，其命名规则为CV_(位数）+（数据类型）+（通道数）如：,CV_8UC3 |
|   depth()   | 矩阵中元素的一个通道的数据类型，这个值和type是相关的。例如 type为 CV_8UC3，一个3通道的16位的有符号整数。那么，depth则是CV_8UC |
| elemSize()  |    矩阵一个元素占用的字节数(不区分通道，即多个通道的总和)    |
| elemSize1() |  矩阵一个元素每个通道占用的字节数(区分通道，单个个通道的值)  |
|    flags    | 一个int型数字，保存了许多有用的信息，[flags说明](http://blog.csdn.net/yiyuehuan/article/details/43701797)； |



#### example

1. Mat M(2,2,CV_8UC3,Scalar(0,0,1));
   2*2大小的矩阵，每个元素为3通道8位无符号整数，如下：
   0 0 1 0 0 1
   0 0 1 0 0 1
2. Mat M(2,2,CV_8UC1,Scalar(10));
   与上面类似：
   10 10
   10 10
3. Mat M(2,2,CV_8UC1,Scalar::all(0));
   数据全为0；
4. Mat M = (Mat_(3,3) << 1,2,3,4,5,6,7,8,9);

注意问题：Mat可以通过row(),col()函数来访问每一行每一列，Mat结构可以通过clone(),copyTo()函数进行复制，直接赋值的话只会把新Mat的data指向原来Mat的data，而不会重新创建一个数据域；

除此之外，Mat还有一些常见的函数用来创建和初始化矩阵：
Mat E = Mat::eye(4,4,CV_64F);
Mat Z = Mat::zeros(4,4,CV_32F);
Mat O = Mat::ones(4,4,CV_8UC1);

注：CV_64F：64位浮点数



## 函数

~~~
cv::imread    // 读jpeg图片
cv::imwrite(path.toStdString(), frame);  // 将cv::mat存成jpeg图像

// 裁剪某个cv::mat
Rect rc = Rect(box.left_top_x, box.left_top_y, box.w, box.h);
cv::Mat roi = frame(rc);    // frame为原图的cv::mat

// 打开某个cv::mat数据并
 im = Image.open(IMG)
 im = im.resize((WIDTH,HEIGHT), Image.NEAREST)

~~~

## 基础

### imread

~~~
import numpy as np
fn="baboon.jpg"
if __name__ == '__main__':
    print 'load %s as ...' % fn
    img = cv2.imread(fn)
    sp = img.shape
    print sp
    sz1 = sp[0]#height(rows) of image
    sz2 = sp[1]#width(colums) of image
    sz3 = sp[2]#the pixels value is made up of three primary colors
~~~

### getRotationMatrix2D

~~~
 Point center = Point( warp_dst.cols/2, warp_dst.rows/2 );   #(width,height)
 rot_mat = getRotationMatrix2D( center, angle, scale );
~~~







# gRPC

[gRPC](https://github.com/grpc/grpc) 是 Google 开源的 RPC 框架和库，已支持主流计算机语言。底层通信采用 gRPC 协议，比较适合互联网场景。gRPC 在设计上考虑了跟 ProtoBuf 的配合使用。

两者分别解决的不同问题，可以配合使用，也可以分开。

典型的配合使用场景是，写好 .proto 描述文件定义 RPC 的接口，然后用 protoc（带 gRPC 插件）基于 .proto 模板自动生成客户端和服务端的接口代码。

[ProtoBuf 与 gRPC - 简书 (jianshu.com)](https://www.jianshu.com/p/8856b9408bbf)

RPC 的全称是 Remote Procedure Call，即远程过程调用。

RPC 一般默认采用 TCP 来传输

![acf53138659f4982bbef02acdd30f1fa](E:\Document\Typora\img\acf53138659f4982bbef02acdd30f1fa.webp)

![nowig0ylpq](E:\Document\Typora\img\nowig0ylpq.jpeg)

![Snipaste_2021-08-09_15-30-13](E:\Document\Typora\img\Snipaste_2021-08-09_15-30-13.png)

1. 交换机在开启gRPC功能后充当gRPC客户端的角色，采集服务器充当gRPC服务器角色；
2. 交换机会根据订阅的事件构建对应数据的格式（GPB/JSON），通过Protocol Buffers进行编写proto文件，交换机与服务器建立gRPC通道，通过gRPC协议向服务器发送请求消息；
3. 服务器收到请求消息后，服务器会通过Protocol Buffers解译proto文件，还原出最先定义好格式的数据结构，进行业务处理；
4. 数据梳理完后，服务器需要使用Protocol Buffers重编译应答数据，通过gRPC协议向交换机发送应答消息；、
5. 交换机收到应答消息后，结束本次的gRPC交互。

上图展示的是gRPC交互过程的具体流程，这也是Telemetry触发方式其中之一，称为Dial-out模式。简单地说，gRPC就是在客户端和服务器端开启gRPC功能后建立连接，将设备上配置的订阅数据推送给服务器端。我们可以看到整个过程是需要用到Protocol Buffers将所需要处理数据的结构化数据在proto文件中进行定义。

## 参考

[gRPC 基础概念详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/389328756)

[gRPC 官方文档中文版_V1.0 (oschina.net)](http://doc.oschina.net/grpc?t=57966)



## 基础术语

+ NameResolver

NameResolver 是可拔插的组件，用于解析目标 URI 并返回地址给调用者。

NameResolver 使用 URI 的 scheme 来检测是否可以解析它， 再使用 scheme 后面的组件来做实际处理。

目标的地址和属性可能随着时间的过去发生修改，因此调用者注册 Listener 来接收持续更新。



### Channel层

`Channel` 到概念上的端点的虚拟连接，用于执行RPC。 通道可以根据配置，负载等自由地实现与端点零或多个实际连接。通道也可以自由地确定要使用的实际端点，并且可以在每次 RPC 上进行更改，从而允许客户端负载平衡。应用程序通常期望使用存根(stub)，而不是直接调用这个类。

应用可以通过使用 ClientInterceptor 装饰 Channel 实现来为 stub 添加常见的切面行为。预计大多数应用代码不会直接使用此类，而是使用已绑定到在应用初始化期间装饰好的 Channel 上的存根。



### stub

grpc 中给服务端取名叫做grpc server，但是客户端不叫grpc client叫做grpc stub，作用就是起到了客户端的作用，用来发起proto请求和接收proto响应。





## 服务类型

定义一个服务，你必须在你的 .proto 文件中指定 `service`：

~~~
service RouteGuide {
   ...
}
~~~

gRPC允 许你定义4种类型的 service 方法，在 `RouteGuide` 服务中都有使用：

+ 一个 *简单 RPC* ， 客户端使用存根发送请求到服务器并等待响应返回，就像平常的函数调用一样。

~~~
 // Obtains the feature at a given position.
   rpc GetFeature(Point) returns (Feature) {}
~~~

- 一个 *服务器端流式 RPC* ， 客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流，直到里面没有任何消息。从例子中可以看出，通过在 *响应* 类型前插入 `stream` 关键字，可以指定一个服务器端的流方法。

~~~
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
~~~

- 一个 *客户端流式 RPC* ， 客户端写入一个消息序列并将其发送到服务器，同样也是使用流。一旦客户端完成写入消息，它等待服务器完成读取返回它的响应。通过在 *请求* 类型前指定 `stream` 关键字来指定一个客户端的流方法。

~~~
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
~~~

- 一个 *双向流式 RPC* 是双方使用读写流去发送一个消息序列。两个流独立操作，因此客户端和服务器可以以任意喜欢的顺序读写：比如， 服务器可以在写入响应前等待接收所有的客户端消息，或者可以交替的读取和写入消息，或者其他读写的组合。 每个流中的消息顺序被预留。你可以通过在请求和响应前加 `stream` 关键字去制定方法的类型。

~~~
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
~~~

## 生成客户端和服务器端代码

从 .proto 的服务定义中生成 gRPC 客户端和服务器端的接口。我们通过 protocol buffer 的编译器 `protoc` 以及一个特殊的 gRPC C++ 插件来完成。

脚本代码

~~~
$ protoc -I ../../protos --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` ../../protos/route_guide.proto
$ protoc -I ../../protos --cpp_out=. ../../protos/route_guide.proto
~~~



运行这个命令可以在当前目录中生成下面的文件：

- `route_guide.pb.h`， 声明生成的消息类的头文件
- `route_guide.pb.cc`， 包含消息类的实现
- `route_guide.grpc.pb.h`， 声明你生成的服务类的头文件
- `route_guide.grpc.pb.cc`， 包含服务类的实现

这些包括：

- 所有的填充，序列化和获取我们请求和响应消息类型的 protocol buffer 代码
- 名为`RouteGuide`的类，包含
  - 为了客户端去调用定义在 `RouteGuide` 服务的远程接口类型(或者 *存根* )
  - 让服务器去实现的两个抽象接口，同时包括定义在 `RouteGuide` 中的方法。

### 结合本地代码理解

#### 对应关系

`packeg`对应`namespace`

`service ReidServiceRpc`对应`class ReidServiceRpc`



#### 服务类：xxxx.grpc.pb.h

![image-20210816153913990](E:\Document\Typora\img\image-20210816153913990.png)

生成的服务类头文件

![image-20210816154045899](E:\Document\Typora\img\image-20210816154045899.png)

![image-20210816154345656](E:\Document\Typora\img\image-20210816154345656.png)

![image-20210816154423617](E:\Document\Typora\img\image-20210816154423617.png)

调用接口

![image-20210816154224651](E:\Document\Typora\img\image-20210816154224651.png)



#### 服务类和消息类

定义的服务如下

![image-20210816155736088](E:\Document\Typora\img\image-20210816155736088.png)

`TaskType`类型参数是入参，`Task`类型是输出参数

函数调用

![image-20210816161314604](E:\Document\Typora\img\image-20210816161314604.png)

对应的服务类函数，分别对应入参为引用，输出参数为指针

![image-20210816161407964](E:\Document\Typora\img\image-20210816161407964.png)

#### 消息类：xxxx.pb.h



![image-20210816160813069](E:\Document\Typora\img\image-20210816160813069.png)

对应的类

![image-20210816161810846](E:\Document\Typora\img\image-20210816161810846.png)

其中的私有变量

![image-20210816161942129](E:\Document\Typora\img\image-20210816161942129.png)



### 创建客户端

在这部分，我们将尝试为`RouteGuide`服务创建一个C++的客户端。你可以从`grpc/examples/cpp/route_guide/route_guide_client.cc`看到我们完整的客户端例子代码.

#### 创建一个存根

首先需要为我们的存根创建一个gRPC *channel*，指定我们想连接的服务器地址和端口，以及 channel 相关的参数——在本例中我们使用了缺省的 `ChannelArguments` 并且没有使用SSL：

~~~
grpc::CreateChannel("localhost:50051", grpc::InsecureCredentials(), ChannelArguments());
~~~

现在我们可以利用channel，使用从.proto中生成的`RouteGuide`类提供的`NewStub`方法去创建存根。

~~~
 public:
  RouteGuideClient(std::shared_ptr<ChannelInterface> channel,
                   const std::string& db)
      : stub_(RouteGuide::NewStub(channel)) {
    ...
  }
~~~



## 内存管理部分





## grpc源码阅读

### 服务器端

让 `RouteGuide` 服务工作有两个部分：

- 实现我们服务定义的生成的服务接口：做我们的服务的实际的“工作”。
- 运行一个 gRPC 服务器，监听来自客户端的请求并返回服务的响应。



#### 同步双向流

~~~
examples\cpp\route_guide\route_guide_server.cc
~~~



#### 异步单步

~~~
examples\cpp\helloworld\greeter_async_server.cc
~~~

[gRPC 官方文档中文版_V1.0 (oschina.net)](http://doc.oschina.net/grpc?t=61534)

~~~
  void Run() {
    std::string server_address("0.0.0.0:50051");

    ServerBuilder builder;
    // 无认证对地址进行监听
    builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
    // 在和客户端沟通时注册异步服务作为即时
    builder.RegisterService(&service_);
    // gRPC运行时获取异步完整队列
    cq_ = builder.AddCompletionQueue();
    // Finally assemble the server.
    server_ = builder.BuildAndStart();
    std::cout << "Server listening on " << server_address << std::endl;

    // Proceed to the server's main loop.
    HandleRpcs();
  }
~~~

### 客户端

#### 双向流式

`examples\cpp\route_guide\route_guide_client.cc`

起一个线程用于写入，其他用于读

~~~
  void RouteChat() {
    ClientContext context;

    std::shared_ptr<ClientReaderWriter<RouteNote, RouteNote> > stream(
        stub_->RouteChat(&context));

    std::thread writer([stream]() {
      std::vector<RouteNote> notes{
        MakeRouteNote("First message", 0, 0),
        MakeRouteNote("Second message", 0, 1),
        MakeRouteNote("Third message", 1, 0),
        MakeRouteNote("Fourth message", 0, 0)};
      for (const RouteNote& note : notes) {
        std::cout << "Sending message " << note.message()
                  << " at " << note.location().latitude() << ", "
                  << note.location().longitude() << std::endl;
        stream->Write(note);
      }
      stream->WritesDone();
    });

    RouteNote server_note;
    while (stream->Read(&server_note)) {
      std::cout << "Got message " << server_note.message()
                << " at " << server_note.location().latitude() << ", "
                << server_note.location().longitude() << std::endl;
    }
    writer.join();
    Status status = stream->Finish();
    if (!status.ok()) {
      std::cout << "RouteChat rpc failed." << std::endl;
    }
  }
~~~







# Protocol Buffers

[ProtoBuf](https://github.com/google/protobuf) 是一套接口描述语言（[IDL](https://en.wikipedia.org/wiki/Interface_description_language)）和相关工具集（主要是 [protoc](https://github.com/google/protobuf)，基于 C++ 实现），类似 Apache 的 [Thrift](https://thrift.apache.org/)）。用户写好 .proto 描述文件，之后使用 protoc 可以很容易编译成众多计算机语言（C++、Java、Python、C#、Golang 等）的接口代码。这些代码可以支持 gRPC，也可以不支持。

Protocol Buffers是一种更加灵活、高效的数据格式，与XML、JSON类似，在一些高性能且对响应速度有要求的数据传输场景非常适用。

Protoco Buffers在gRPC的框架中主要有三个作用：

1. 定义数据结构

![Snipaste_2021-08-09_15-34-58](E:\Document\Typora\img\Snipaste_2021-08-09_15-34-58.png)

2. 定义服务接口

![Snipaste_2021-08-09_15-35-41](E:\Document\Typora\img\Snipaste_2021-08-09_15-35-41.png)

3. 通过序列化和反序列化，提升传输效率

