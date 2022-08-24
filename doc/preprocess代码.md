# preprocess代码

## 调试遇到的问题

+ Device not open

![image-20210825172916374](E:\Document\Typora\img\image-20210825172916374.png)

解决：

因为代码指定了输出路径，所以端口占用，无法输出文件

~~~
  logFilePath = qApp->applicationDirPath() + "/log";
  qInstallMessageHandler(logMsgHandler);
~~~

+ 没有debug模式

---> 使用release模式进行调试

[(VS2015在release模式下进行调试_liuwb博客-CSDN博客](https://blog.csdn.net/liuzhezhe111/article/details/82155306)



+ 调试过程字符串显示异常

![image-20210825173253988](E:\Document\Typora\img\image-20210825173253988.png)

解决：

可以看到字符串应该是正常的，但是debug调试异常。。。忽视吧~

可能跟不是debug模式有关，这个先凑合看，周末调一版debug的

+ TensorRT版本不对

![image-20210827165850181](E:\Document\Typora\img\image-20210827165850181.png)

engine和当前环境的tensorRT版本不同

。。。无语，我居然忘记安装tensorRT了，服了自己，TensorRT居然是运行时才调用，应该是preprocess本身不依赖TensorRT，是底层代码调用。

...安装了tensorRT还是一样的报错，好吧是文件没替换，然后又报错，是因为之前装的cnn库版本不对没有换掉

[windows安装tensorrt - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/339753895)

这个cudnn的版本是8.1，我本人安装的是8.2

![image-20210830101257911](E:\Document\Typora\img\image-20210830101257911.png)

涉及到了Qt/grpc/opencv 

+ tensorrt的部署问题

![image-20210830101944451](E:\Document\Typora\img\image-20210830101944451.png)



[tensorrt的部署和问题](https://blog.csdn.net/weixin_42476942/article/details/113753334)

cuda 版本  10.2.1

cudnn版本  8.1.0.77

tensorRT版本   7.2.3.4

心累，原来我不能跑是因为显卡版本不对，模型可运行的环境是2080Ti，我的显卡是3070。。。

![image-20210830134119532](E:\Document\Typora\img\image-20210830134119532.png)

## 代码理解

### 1. QT的线程启动逻辑



`main()`函数最后，调用`app.exec()`，开启事件循环。

![image-20210827144719963](E:\Document\Typora\img\image-20210827144719963.png)

创建线程等待启动

![perprocess1](E:\Document\draw.io\perprocess1.png)



![preprocess4](E:\Document\draw.io\preprocess4.png)

~~~
connect(sender,signal,receiver,slot);
~~~

每100ms触发一次定时器，执行`schedule()`函数。

### 2. 类调用逻辑

![preprocess2](E:\Document\draw.io\preprocess2.png)

### 3. 内部线程逻辑









### 4. RPC逻辑

#### 4.1 服务类：xxxx.grpc.pb.h

![image-20210816153913990](E:\Document\Typora\img\image-20210816153913990.png)

生成的服务类头文件

![image-20210816154045899](E:\Document\Typora\img\image-20210816154045899.png)

![image-20210816154345656](E:\Document\Typora\img\image-20210816154345656.png)

![image-20210816154423617](E:\Document\Typora\img\image-20210816154423617.png)

调用接口

![image-20210816154224651](E:\Document\Typora\img\image-20210816154224651.png)



#### 4.2 服务类和消息类

定义的服务如下

![image-20210816155736088](E:\Document\Typora\img\image-20210816155736088.png)

`TaskType`类型参数是入参，`Task`类型是输出参数

注意这里的`task`是临时变量。

函数调用

![image-20210816161314604](E:\Document\Typora\img\image-20210816161314604.png)

对应的服务类函数，分别对应入参为引用，输出参数为指针

![image-20210816161407964](E:\Document\Typora\img\image-20210816161407964.png)

#### 4.3 消息类：xxxx.pb.h



![image-20210816160813069](E:\Document\Typora\img\image-20210816160813069.png)

对应的类

![image-20210816161810846](E:\Document\Typora\img\image-20210816161810846.png)

其中的私有变量

![image-20210816161942129](E:\Document\Typora\img\image-20210816161942129.png)





## 底层调用代码理解

### Detect

ZJLabCV_Detect_win.h

![image-20210830112340584](E:\Document\Typora\img\image-20210830112340584.png)

**BATCH_SIZE:即一次训练所抓取的数据样本数量；**

+ 对图片大小进行重采样





**resize : 缩小或者放大函数至某一个大小**

~~~
resize(InputArray src, OutputArray dst, Size dsize, 
        double fx=0, double fy=0, int interpolation=INTER_LINEAR 
~~~

**INTER_CUBIC** - 4x4像素邻域内的双立方插值

~~~
cv::Mat out(INPUT_H, INPUT_W, CV_8UC3, cv::Scalar(128, 128, 128));
~~~

` CV_8UC3`的意思是3通道8位， Scalar(0,0, 0)是黑色字体，cv::Scalar(128, 128, 128)相当于b=128，g=128，r=128。

**image.copyTo(imageROI)。作用是把image的内容复制粘贴到imageROI上；**

`cv::Rect`指定区域范围。
