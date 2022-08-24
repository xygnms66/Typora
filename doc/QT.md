## 参考资料

[《Qt 学习之路 2》目录 (devbean.net)](https://www.devbean.net/2012/08/qt-study-road-2-catelog/)

[Qt 5.15官方文档](https://doc.qt.io/qt-5/index.html)

## QT安装

报错

~~~
./qt-opensource-linux-x64-5.14.2.run: error while loading shared libraries: libfontconfig.so.1: cannot open shared object file: No such file or directory
~~~

安装  `sudo apt-get install libfontconfig`

## QT学习之路

### 1 Hello, world!

`main()`函数中第一句是创建一个`QApplication`类的实例。对于 Qt 程序来说，`main()`函数一般以创建 application 对象（GUI 程序是`QApplication`，非 GUI 程序是`QCoreApplication`。

`main()`函数最后，调用`app.exec()`，开启事件循环。我们现在可以简单地将事件循环理解成一段无限循环。

~~~
#include <QApplication>
#include <QLabel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QLabel *label = new QLabel("Hello, world");
    label->show();

    return app.exec();
}
~~~



### 2 信号槽

在 Qt 5 中，`QObject::connect()`有五个重载：

~~~
QMetaObject::Connection connect(const QObject *, const char *,
                                const QObject *, const char *,
                                Qt::ConnectionType);

QMetaObject::Connection connect(const QObject *, const QMetaMethod &,
                                const QObject *, const QMetaMethod &,
                                Qt::ConnectionType);

QMetaObject::Connection connect(const QObject *, const char *,
                                const char *,
                                Qt::ConnectionType) const;

QMetaObject::Connection connect(const QObject *, PointerToMemberFunction,
                                const QObject *, PointerToMemberFunction,
                                Qt::ConnectionType)

QMetaObject::Connection connect(const QObject *, PointerToMemberFunction,
                                Functor);
~~~

这五个重载的返回值都是`QMetaObject::Connection`，**`connect()`函数最常用的一般形式：**

```
// !!! Qt 5
connect(sender,signal,receiver,slot);
```

`connect()`一般会使用前面**四个参数，第一个是发出信号的对象，第二个是发送对象发出的信号，第三个是接收信号的对象，第四个是接收对象在接收到信号之后所需要调用的函数。**也就是说，当 sender 发出了 signal 信号之后，会自动调用 receiver 的 slot 函数。

这是最常用的形式，我们可以套用这个形式去分析上面给出的五个重载。第一个，sender 类型是`const QObject *`，signal 的类型是`const char *`，receiver 类型是`const QObject *`，slot 类型是`const char *`。这个函数将 signal 和 slot 作为字符串处理。第二个，sender 和 receiver 同样是`const QObject *`，但是 signal 和 slot 都是`const QMetaMethod &`。我们可以将每个函数看做是`QMetaMethod`的子类。因此，这种写法可以使用`QMetaMethod`进行类型比对。第三个，sender 同样是`const QObject *`，signal 和 slot 同样是`const char *`，但是却缺少了 receiver。这个函数其实是将 this 指针作为 receiver。第四个，sender 和 receiver 也都存在，都是`const QObject *`，但是 signal 和 slot 类型则是`PointerToMemberFunction`。看这个名字就应该知道，这是指向成员函数的指针。第五个，前面两个参数没有什么不同，最后一个参数是`Functor`类型。这个类型可以接受 static 函数、全局函数以及 Lambda 表达式。

由此我们可以看出，`connect()`函数，sender 和 receiver 没有什么区别，都是`QObject`指针；主要是 signal 和 slot 形式的区别。具体到我们的示例，我们的`connect()`函数显然是使用的第五个重载，最后一个参数是`QApplication`的 static 函数`quit()`。也就是说，当我们的 button 发出了`clicked()`信号时，会调用`QApplication`的`quit()`函数，使程序退出。

信号槽要求信号和槽的参数一致，所谓一致，是参数类型一致。如果不一致，允许的情况是，槽函数的参数可以比信号的少，即便如此，槽函数存在的那些参数的顺序也必须和信号的前面几个一致起来。这是因为，你可以在槽函数中选择忽略信号传来的数据（也就是槽函数的参数比信号的少），但是不能说信号根本没有这个数据，你就要在槽函数中使用（就是槽函数的参数比信号的多，这是不允许的）。

+ 信号槽的Lambda表达式

```
// !!! Qt 5
#include <QApplication>
#include <QPushButton>
#include <QDebug>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QPushButton button("Quit");
    QObject::connect(&button, &QPushButton::clicked, [](bool) {
        qDebug() << "You clicked me!";
    });
    button.show();

    return app.exec();
}
```

+ 使用stdc++11以上的版本配置编译信息

~~~
QMAKE_CXXFLAGS += -std=c++0x
~~~

+ 三种类型的信号槽连接

1. 直接连接，当signal发射时，slot立即调用。此slot在发射signal的那个线程中被执行(不一定是接收对象生存的那个线程)
2. 队列连接，**当控制权回到对象属于的那个线程的事件循环时，slot被调用**。此slot在接收对象生存的那个线程中被执行
3. 自动连接(缺省)，假如信号发射与接收者在同一个线程中，其行为如直接连接，否则，其行为如队列连接。

+ connect(sender,SIGNAL(signal()),receiver,SLOT(slot())); 

SIGNAL() 和SLOT()；通过connect声明可以知道**这两个宏最后倒是得到一个const char\*类型**。

\#ifndef QT_NO_DEBUG 
\# define QLOCATION "\0"__FILE__":"QTOSTRING(__LINE__) 
\# define METHOD(a)  qFlagLocation("0"#a QLOCATION) 
\# define SLOT(a)   qFlagLocation("1"#a QLOCATION) 
\# define SIGNAL(a)  qFlagLocation("2"#a QLOCATION) 
\#else 
\# define METHOD(a)  "0"#a 
\# define SLOT(a)   "1"#a 
\# define SIGNAL(a)  "2"#a 
\#endif 
所以这两个宏的作用就是**把函数名转换为字符串并且在前面加上标识符**。

#### 关键字emit，slots，signals

[QT 中 关键字讲解（emit,signal,slot） - FelixWang - 博客园 (cnblogs.com)](https://www.cnblogs.com/felix-wang/p/6212197.html)

[基于 Qt QThread 的同步任务队列和异步任务队列_Jimmy的小黑屋-CSDN博客_qt异步任务](https://blog.csdn.net/u011546766/article/details/80975266)



### 3 自定义信号槽

只有继承了`QObject`类的类，才具有信号槽的能力。所以，为了使用信号槽，必须继承`QObject`。凡是`QObject`类（不管是直接子类还是间接子类），都应该在第一行代码写上`Q_OBJECT`。不管是不是使用信号槽，都应该添加这个宏。这个宏的展开将为我们的类提供信号槽机制、国际化机制以及 Qt 提供的不基于 C++ RTTI 的反射能力。因此，如果你觉得你的类不需要使用信号槽，就不添加这个宏，就是错误的。其它很多操作都会依赖于这个宏。注意，这个宏将由 moc（我们会在后面章节中介绍 moc。这里你可以将其理解为一种预处理器，是比 C++ 预处理器更早执行的预处理器。） 做特殊处理，不仅仅是宏展开这么简单。moc 会读取标记了 Q_OBJECT 的**头文件**，生成以 moc_ 为前缀的文件，比如 newspaper.h 将生成 moc_newspaper.cpp。你可以到构建目录查看这个文件，看看到底增加了什么内容。注意，由于 moc 只处理头文件中的标记了`Q_OBJECT`的类声明，不会处理 cpp 文件中的类似声明。因此，如果我们的`Newspaper`和`Reader`类位于 main.cpp 中，是无法得到 moc 的处理的。解决方法是，我们手动调用 moc 工具处理 main.cpp，并且将 main.cpp 中的`#include "newspaper.h"`改为`#include "moc_newspaper.h"`就可以了。不过，这是相当繁琐的步骤，为了避免这样修改，我们还是将其放在头文件中。许多初学者会遇到莫名其妙的错误，一加上`Q_OBJECT`就出错，很大一部分是因为没有注意到这个宏应该放在头文件中。

下面总结一下自定义信号槽需要注意的事项：

- 发送者和接收者都需要是`QObject`的子类（当然，槽函数是全局函数、Lambda 表达式等无需接收者的时候除外）；
- 使用 signals 标记信号函数，信号是一个函数声明，返回 void，不需要实现函数代码；
- 槽函数是普通的成员函数，作为成员函数，会受到 public、private、protected 的影响；
- 使用 emit 在恰当的位置发送信号；
- 使用`QObject::connect()`函数连接信号和槽。

### 4 线程简介



### 5 多线程

**Qt 提供了 QThreadPool 和 QRunnable 这两个类来对线程进行重复的使用**。



## isInterruptionRequested

isInterruptionRequested()来判断是否请求终止线程，如果没有，则一直运行；当希望终止线程的时候，调用requestInterruption()即可。

如果一个应该停止的任务在此线程上运行，则返回true。 requestInterruption（）可以请求中断。



## **QQueue**

它的父类是QList,是个模板类

~~~
QQueue<int> Q;           　　　　  //定义一个int型队列

Q.isEmpty();                      //返回队列是否为空

Q.size();                        //返回队列元素个数

Q.clear();                        //清空队列

Q.enqueue();                      //在队列尾部添加一个元素, 比如插入数字5: Q.enqueue(5)

Q.dequeue();                     //删除当前队列第一个元素,并返回这个元素

Q.head();                        //返回当前队列第一个元素

Q.last();                        //返回当前队列尾部的元素

T &  operator[]( int i );        //以数组形式访问队列元素
~~~



## QCoreApplication

QCoreApplication表示Qt控制台程序，QApplication 和 QGuiApplication 表示GUI程序。

它们之间的关系为QCoreApplication 继承自最顶层的QObject，QGuiApplication 又继承自QCoreApplication，QApplication又继承自QGuiApplication。

![QT1.0](E:\Document\draw.io\QT1.0.png)

QCoreApplication类表示**Qt控制台程序**，它**承载了应用程序的事件循环**，<u>通过这个事件循环对所以来自操作系统和其他事件源，如网络，数据库，的事件进行处理和分发</u>。同时，它还实现了应用程序的初始化和退出清理工作，以及系统级和应用级的常用设置。所以，Qt在生成main函数时就自动为我们定义了已应用程序对象，其在程序运行的过程中，就代表了整个应用程序本身。但由于该对象是main函数的局部对象，不方便我们在其他地方使用，特别是对于GUI程序来说，我们可能需要在某个窗口类中使用到这个应用程序对象，所以，鉴于此，Qt又在全局空间为我们提供了已应用程序指针，即qApp，该全局指针指向的就是main函数中定义的应用程序对象，我们可以在任何地方使用这个指针。当然，我们还可以**使用QCoreApplication类中的一个静态成员函数instance()，来获得代表该应用程序的指针**，该函数声明如下：

~~~
QCoreApplication *QCoreApplication::instance()
~~~

+ 事件循环和事件处理

定义了一个QCoreApplication类的对象，紧接着就调用了exec() 成员函数，而Qt的事件循环就是开始于此函数的调用。



exec() 会一直运行到事件循环结束才返回，比如quit()或exit() 函数被调用时，而exec() 的返回的值即为调用exit()时所传入的值，如果是quit() 函数，则相当于exit(0)。



QCoreApplication还为我们提供了几个方便的静态程序函数。比如我们上面已经说过的instance() 函数，它可以返回应用程序对象的指针；**sendEvent()**、**postEvent()** 和**sendPostedEvent()** 用于发送或寄送事件；事件队列中未处理的事件可以使用**removePostedEvents()** 或者 **flush()** 进行处理。









## QTimerEvent使用

timerEvent 是QObejct所内置的事件，所有继承自QObject的类都可以使用。

要产生timerEvent，就需要startTimer(delaytime) 方法，startTimer方法返回该对象的这个计时器的id号，int类型。

killTimer(timerid) 停止该对象的id号为timerid的计时器。 对于多个定时器，timerEvent(QTimerEvent *e)   可以通过

e->timerId()来区分哪个timer出发了事件。



### 函数

1. void setExpiryTimeout(int expiryTimeout)//设置线程回收的等待时间

## qInfo

Qt提供了5个全局函数用于输出调试或警告信息。

- qDebug：调试信息
- qWarning：警告信息
- qCritical：严重错误
- qFatal：致命错误
- qInfo : 正常信息





## Qt startTimer

`int QObject::startTimer(int interval, Qt::TimerType timerType = Qt::CoarseTimer)`

调用 startTimer启动一个定时器，并返回定时器id。如果启动失败，返回0.
定时器每隔interval 毫秒就会启动一次，直到调用killTimer(). 如果interval=0,当没有其他系统时间发生时，会调用一次。
当定时器发生时，会调用timerEvent(QTimerEvent *event).如果多个定时器在运行，可用通过timeId()来区分。

~~~
startTimer(50);     // 50-millisecond timer
      startTimer(1000);   // 1-second timer
      startTimer(60000);  // 1-minute timer

      using namespace std::chrono;
      startTimer(milliseconds(50));
      startTimer(seconds(1));
      startTimer(minutes(1));

      // since C++14 we can use std::chrono::duration literals, e.g.:
      startTimer(100ms);
      startTimer(5s);
      startTimer(2min);
      startTimer(1h);
~~~

## qreal

qreal是Qt的数据类型，在桌面操作系统中（比如Windows， XNix等）qreal其实就是**double类型**；而在嵌入设备系统中，qreal则等同于float 类型。





## QT的线程使用

1. 继承 ***QThread***，重写 ***run()*** 接口；
2. 使用 ***moveToThread()*** 方法将 ***QObject*** 子类移至线程中，内部的所有使用信号槽的槽函数均在线程中执行；
3. 使用 ***QThreadPool*** 线程池，搭配 ***QRunnable***；
4. 使用 ***QtConcurrent***；





## QThreadPool 和QRunnable

[QT 多线程之线程池QThreadPool(深入理解)_断点-CSDN博客](https://blog.csdn.net/qq_37529913/article/details/115536799)

[Qt之QThreadPool和QRunnable-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/119875)



### 描述

`QRunnable`是所有runnable对象的基类，而`QThreadPool`类用于管理`QThreads`集合

`QRunnable`类是一个接口，用于表示一个任务要执行的代码，需要重新实现`run()`函数。

`QThreadPool `管理和循环使用单独的 QThread 对象，以帮助程序减少创建线程的成本。每个 Qt 应用程序都有一个全局 QThreadPool 对象，可以通过调用 globalInstance() 访问。



QThreadPool 类管理 QThread 的集合。

QThreadPool 管理和回收单独的 QThread 对象，以减少使用线程的程序中的线程创建成本。

每个 Qt 应用程序都有一个全局 QThreadPool 对象，可以通过调用 globalInstance() 来访问它。

要使用 QThreadPool，需要子类化 QRunnable 并实现 run() 虚函数。然后创建该类的对象并将其传递给 QThreadPool::start()。QThreadPool 默认自动删除 QRunnable。

QThreadPool 是管理线程的低级类，Qt Concurrent 模块是更高级的方案。



### QThreadPool

#### 成员变量

+ activeThreadCount : const int

表示线程池中的活动线程数。

+ expiryTimeout : int

在 expiryTimeout 毫秒内未使用的线程被认为已过期并将退出。此类线程将根据需要重新启动。 默认的expiryTimeout 是30000 毫秒（30 秒）。

设置 expiryTimeout 对已经运行的线程没有影响。

建议在创建线程池之后调用start()之前设置 expiryTimeout。

+ maxThreadCount : int

该属性表示线程池使用的最大线程数。默认值是 QThread::idealThreadCount()。

注意：线程池将始终使用至少 1 个线程，即使 maxThreadCount 设置为零或负数。

+ stackSize : uint

此属性包含线程池工作线程的堆栈大小。默认值为 0，表示 QThread 使用操作系统默认堆栈大小。

更改它对已创建或正在运行的线程没有影响。



#### 成员函数

~~~
void QThreadPool::start(QRunnable * runnable, int priority = 0);
bool QThreadPool::tryStart(QRunnable * runnable);
~~~

- ***start()*** 预定一个线程用于执行QRunnable接口，当预定的线程数量超出线程池的最大线程数后，QRunnable接口将会进入队列，等有空闲线程后，再执行；
- priority指定优先级
- ***tryStart()*** 和 ***start()*** 的不同之处在于，当没有空闲线程后，不进入队列，返回false、



+ 启动函数对QRunnable的影响

对于start，传入的是QRunnable对象指针，传入后线程池会调用QRunnable的**[autoDelete](http://doc.qt.io/qt-5/qrunnable.html#autoDelete)**() 函数，若返回true，则当此运算完成后自动释放内容，不需要后续主动判断是否运算完成并释放空间。

对于tryTake，若返回成功，不会自动释放内容，而是需要调用方主动释放，无论autodelete返回值是什么。返回false自然也不会自动delete



+ **其他成员函数**

1. int activeThreadCount() const //当前的活动线程数量
2. void clear()//清除所有当前排队但未开始运行的任务
3. int expiryTimeout() const//线程长时间未使用将会自动退出节约资源，此函数返回等待时间
4. int maxThreadCount() const//线程池可维护的最大线程数量
5. void releaseThread()//释放被保留的线程
6. void reserveThread()//保留线程，此线程将不会占用最大线程数量，从而可能会引起当前活动线程数量大于最大线程数量的情况
7. void setExpiryTimeout(int expiryTimeout)//设置线程回收的等待时间
8. void setMaxThreadCount(int maxThreadCount)//设置最大线程数量
9. void setStackSize(uint stackSize)//此属性包含线程池工作线程的堆栈大小。
10. uint stackSize() const//堆大小
11. void start(QRunnable *runnable, int priority = 0)//加入一个运算到队列，注意start不一定立刻启动，只是插入到队列，排到了才会开始运行。需要传入QRunnable ，后续介绍
12. bool tryStart(QRunnable *runnable)//尝试启动一个
13. bool tryTake(QRunnable *runnable)//删除队列中的一个QRunnable，若当前QRunnable 未启动则返回成功，正在运行则返回失败
14. bool waitForDone(int?<i>msecs</i>?=?-1)//等待所有线程运行结束并退出，参数为等待时间-1表示一直等待到最后一个线程退出



#### 全局线程池

QThreadPool提供了一个静态函数，**[globalInstance](http://doc.qt.io/qt-5/qthreadpool.html#globalInstance)**()，使用此方法可获取一个当前进程的全局线程池，可在多个类中共同使用一个线程池。

#### 局部线程池

和常规类的使用相同，可以通过 `QThreadPool pool`的方式建立一个局部线程池，并由当前类维护，可保证此线程池仅供当前类应用



### QRunnable

QRunnable 类是一个接口，用于表示需要执行的任务或代码段，由重新实现的 run() 函数表示。
可以使用 QThreadPool 在单独的线程中执行代码。

#### 成员函数

+ [**static**] QRunnable *create(std::function<void ()> functionToRun)

创建一个在 run() 中调用 functionToRun 的 QRunnable。默认情况下启用自动删除。

+ void run()

线程中要执行的代码，重写需要运算的内容。

+ void setAutoDelete(bool autoDelete)

设置是否启用自动删除。

~~~~
bool QRunnable::autoDelete() const;
void QRunnable::setAutoDelete(bool autoDelete)；
~~~~

**QRunnable**对象的`autoDelete()`设为`true`的话，`QThreadPool`会在run（）运行结束后**自动删除该对象**。



## QList







## QVariantMap

**QVariant**类型的放入和取出必须是相对应的，你放入一个int就必须按int取出，不能用toString(), Qt不会帮你自动转换。

**数据核心无非就是一个 union，和一个标记类型的type：传递的是整数 123，那么它union存储整数123，同时type标志Int；如果传递字符串，union存储字符串的指针，同时type标志QString**。



## QSqlDatabase

QSqlDatabase类实现了数据库连接的操作
QSqlQuery类执行SQL语句
QSqlRecord类封装数据库所有记录



QSqlDatabase 类用于处理与数据库的连接，提供用于通过连接访问数据库的接口，一个 QSqlDatabase 实例就代表了一个连接

QSqlDatabase类

~~~
//! testMYSQLs
void MainWindow::on_testMYSQL_clicked()
{
 
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL", "localhost@3306");
    db.setHostName("localhost");    //数据库主机名
    db.setDatabaseName("databasex");    //数据库名
    db.setUserName("root");        //数据库用户名
    db.setPassword("123456");        //数据库密码
    bool bisOpenn = db.open();          //打开数据库连接
    qDebug()<<"bisOpenn="<<bisOpenn;
    //db.close();         //释放数据库连接
}
~~~



## QFile

构造一个以name为文件名的QFile对象。

~~~
QFile file(_configFile);
~~~

文件读操作

~~~
QByteArray QIODevice::read(qint64 maxSize);//读取maxSize个字节，内部位置指针后移maxSize，并返回一个QByteArray对象。
QByteArray QIODevice::readLine(qint64 maxSize = 0) //读取一行，但是这一行不能超过maxSize字节，maxSize = 0代表不限制行字节数。
QByteArray QIODevice::readAll()
~~~





## JSON处理类

| `QJsonArray`            | 封装 JSON 数组                                    |
| ----------------------- | ------------------------------------------------- |
| `QJsonDocument`         | 读写 JSON 文档                                    |
| `QJsonObject`           | 封装 JSON 对象                                    |
| `QJsonObject::iterator` | 用于遍历`QJsonObject`的 STL 风格的非 const 遍历器 |
| `QJsonParseError`       | 报告 JSON 处理过程中出现的错误                    |
| `QJsonValue`            | 封装 JSON 值                                      |

`QJsonDocument::fromJson()`可以由`QByteArray`对象构造一个`QJsonDocument`对象，用于我们的读写操作。这里我们传入一个`QJsonParseError`对象的指针作为第二个参数，用于获取解析的结果。如果`QJsonParseError::error()`的返回值为`QJsonParseError::NoError`，说明一切正常，则继续以`QVariant`的格式进行解析（由于我们知道这是一个 JSON 对象，因此只判断了`isObject()`。当处理未知的 JSON 时，或许应当将所有的情况都考虑一边，包括`isObject()`、`isArray()`以及`isEmpty()`）。

## QJsonObject

QJsonObject类用于封装JSON对象。JSON对象是包含键值对的链表，其中键是唯一的字符串，其值由QJsonValue对象。

~~~
QJsonObject jo = QJsonDocument::fromJson(file.readAll()).object();
_detectConfigFile = jo["DetectConfigFile"].toString();
~~~

[QJson总结 - 海物chinono - 博客园 (cnblogs.com)](https://www.cnblogs.com/chinono/p/13973056.html)

## QString

QString 是 [Qt](http://c.biancheng.net/qt/) 编程中常用的类，除了用作数字量的输入输出之外，QString 还有很多其他功能，熟悉这些常见的功能，有助于灵活地实现字符串处理功能。

QString 存储字符串釆用的是 Unicode 码，每一个字符是一个 16 位的 QChar，而不是 8 位的 char，所以 QString 处理中文字符没有问题，而且一个汉字算作是一个字符。

[Qt QString类及常用函数功能详解 (biancheng.net)](http://c.biancheng.net/view/1844.html)

### arg使用



~~~
        _channel = grpc::CreateChannel(QString("%1:%2").arg(serverAddress).arg(port).toStdString(),
                                        grpc::InsecureChannelCredentials());
~~~

+ toStdString()

~~~
// 1
QString str = "Hello, world!";
char *cStr = str.toStdString().c_str();

// 2
std::string sstr = str.toStdString();
char *cStr2 = sstr.c_str();
~~~



# 界面学习



## 实验楼布局学习

在自定义导航按钮中规划按钮布局，即图标、文字、三角显示的位置，图标与文字的间距，文字的边距等等。

自定义按钮中的布局表示将按钮拆分为小块，然后进行绘制。例如背景、文字两个是需要分开单独绘制的。

### 文字

- 最简单的一种情况是，一个导航按钮只有文字，没有图标。如下图所示，需要考虑到文字到四边的边距、颜色、背景颜色等。
- 文字距四边的边距分别使用一个变量来保存，在绘制文本时需要将边距计算在内。

![2-1-button-text](https://doc.shiyanlou.com/courses/2623/939394/59cdaa2cc255c1558d6110d3796a29db-0)

### 图标 + 文字

- 当在文字的基础上加上图标时需要增加属性，并且在计算边距时需要将图标考虑在内。
- 在计算图标与文字的相对位置时需要注意图标的边距与文字的同边边距是重叠的。如下图中图标边距是距离左侧的边距，文字的左边距是包含了图标边距。
- 此时需要计算图标的大小，同时还需要考虑图标与文字的相对位置，即图标在左文字在右、图标在右文字在左、图标在上文字在下、图标在下文字在上。
- 在以上的可能性中，图标在左文字在右、图标在上文字在下这两种方式是常用的。

图标与文字左右结构：

![2-2-button-icon-text](https://doc.shiyanlou.com/courses/2623/939394/b1bc9695550cb509dcbd539ee6b073da-0)

图标与文字上下结构：

![2-3-button-icon-text](https://doc.shiyanlou.com/courses/2623/939394/7fe9ea0b0a05900fc8ef9890a6708165-0)

### 图标 + 文字 + 三角

- 在 图标 + 文字的基础上增加三角，多个导航按钮组合时可能会达到更好的效果。
- 在画三角时可以不考虑边距的文字，只需要考虑三角的方向、颜色。

![2-4-button-icon-text-tri](https://doc.shiyanlou.com/courses/2623/939394/cc6a427d836012363b9a632ff5c15e0d-0)

## Q_PROPRERTY属性

Q_PROPERTY 是 Qt 中的一个宏，是用类中声明属性。如果需要使用该宏，必须要继承 QObject 类或者其子类。QPushButton 则是 QObject 的间接子类，所以继承 QPushButton 类后同样可以使用 Q_PROPERTY 宏。

Q_PROPERTY 属性自带了一些属性，同样程序可以自定义。本实验中只讲解 Q_PROPERTY 自带的属性。

在自定义导航按钮的程序中同样使用了 Q_PROPERTY，且程序中只使用了 Q_PROPERTY 的 READ 属性和 WRITE 属性。

```C++
// jnavibutton.h
class JNaviButton : public QPushButton
{
    Q_PROPERTY(int m_paddingLeft READ paddingLeft WRITE setPaddingLeft)
    Q_PROPERTY(int m_paddingTop READ paddingTop WRITE setPaddingTop)
    Q_PROPERTY(int m_paddingRight READ paddingRight WRITE setPaddingRight)
    Q_PROPERTY(int m_paddingBottom READ paddingBottom WRITE setPaddingBottom)
    // ...
}
```

Q_PROPERTY 自带属性：

```C++
Q_PROPERTY(type name
           READ getFunction
           [WRITE setFunction]
           [RESET resetFunction]
           [NOTIFY notifySignal]
           [DESIGNABLE bool]
           [SCRIPTABLE bool]
           [STORED bool]
           [USER bool]
           [CONSTANT]
           [FINAL])
```

- 在上面的代码中，方括号 [] 中的内容属性可选。
- 必选 `READ` 属性：用来读取属性值，因此使用 const 限制，返回值类型必须为属性类型或者属性类型的引用或者指针。
- 可选 `WRITE` 属性：用来设置属性值，返回值必须为 void 类型，需要一个参数。
- 可选 `RESET` 属性：能够将值设置为默认状态，返回值为 void 类型且不带参数。
- 可选 `NOTIFY` 属性：提供一个信号，当值发送改变是该信号会自动被触发。
- 可选 `DESIGNABLE` 属性：是否在界面设计器的属性编辑器中出现。大多数属性是可见的，除了可以为变量传入 true 或 false 还可以执行一个 bool 行的成员函数。
- 可选 `SCRIPTABLE` 属性：是够可以被脚本引擎操作（默认为 true）。可以赋予 true 或者 false 或 bool 类型的函数。
- 可选 `STORED` 属性：是否被认为是独立存在还是依赖于其他的值而存在，也可以表明是否在保存对象状态时保存此属性的值。大多数属性都是需要保存的，但也有例外，例如 `QWidget::minimumWidth()` 就是不被保存的，因为它的值是从另一个属性 `QWidget::minimumSize()` 得来的。
- 可选 `USER` 属性：是否被设计为面向用户的或用户可修改的类属性。通常，每个类只有一个 USER 属性。例如 `QAbstractButton::checked` 是按钮类的用户可修改属性。注意 QItemDelegate 获取和设置 widget 的 USER 属性。
- 可选 `CONSTANT` 属性：表示属性的值是不变的。
- 可选 `FINAL` 属性：表示属性不能被派生类所重写。

## 信号槽

#### 介绍

在 Qt 中，信号槽是用来响应事件的一种方式，其实质是函数回调，是 Qt 的一大特点，也是 Qt 中一种常用的通信方式。

对于只想学会如何使用信号槽的同学来说，可以先不用细究其实现原理，直接看使用方法小节即可。

#### 原理

Q_OBJECT 是 Qt 中常用且很重要的一个宏，通常情况下如果新建一个类并继承了 QObject，那么程序将会指定在类的最前方使用 Q_OBJECT 宏。这个宏展开后其实声明了 5 个函数，直接转到 Q_OBJECT 可以看到。而在编译时会自动生成一个 moc_xxx.cpp，在 moc_xxx.cpp 中实现了 Q_OBJECT 声明的函数。

- Qt 的信号槽实质是函数回调，不同的是直连直接回调，队列连接使用 Qt 的事件循环隔离了一次达到异步，最终还是使用函数回调。
- moc 预编译构建了信号槽回调的开头（信号函数体）和结尾（qt_static_metacall 回调函数），中间的回调过程 Qt 已经在 QObject 函数中实现。
- 信号和槽本质上是一样的，但对于使用者来说，信号只需要声明，在 moc 预编译中自动实现，而槽函数声明和定义都需要自己编写。

```C++
// Q_OBJECT 定义源码
/* qmake ignore Q_OBJECT */
#define Q_OBJECT \
public: \
    Q_OBJECT_CHECK \
    QT_WARNING_PUSH \
    Q_OBJECT_NO_OVERRIDE_WARNING \
    // 构造一个 QMetaObject 对象，传入当前 moc 文件的动态信息
    static const QMetaObject staticMetaObject; \
    // 返回当前 QMetaObject，一般而言，需要数 metaObject() 仅返回类的 staticMetaObject 对象
    virtual const QMetaObject *metaObject() const; \
     // 是否可以进行类型转换，被 QObject::inherits 直接调用，用于判断继承自某个类。判断时需要传入父类的字符串名称
    virtual void *qt_metacast(const char *); \
    // 调用函数回调，内部调用了 qt_static_metacall 函数，该函数被异步处理信号时调用
    virtual int qt_metacall(QMetaObject::Call, int, void **); \
    QT_WARNING_POP \
    QT_TR_FUNCTIONS \
private: \
    // 改函数将根据函数索引调用槽函数，从下面导航按钮的生成源码可以看出，信号和槽函数都是可以被回调
    Q_DECL_HIDDEN_STATIC_METACALL static void qt_static_metacall(QObject *, QMetaObject::Call, int, void **); \
    struct QPrivateSignal {};
// moc_widget.cpp
// 截取自自定义导航按钮实验中生成的 moc_widget.cpp 中部分源码
void Widget::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        Widget *_t = static_cast<Widget *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->btns1_clicked(); break;
        case 1: _t->btns2_clicked(); break;
        case 2: _t->btns3_clicked(); break;
        case 3: _t->btns4_clicked(); break;
        case 4: _t->btns5_clicked(); break;
        case 5: _t->btns6_clicked(); break;
        default: ;
        }
    }
    Q_UNUSED(_a);
}
```

#### 连接

- connect 方法就是把发送者、信号、接受者和槽存储、绑定起来，在执行时查找。
- 信号槽有 5 种连接方式，前 4 种是互斥的，可以表示为异步和同步；而第 5 种唯一连接是配合前 4 种方式使用。

connect 的 5 种方式：

1. `Qt::AutoConnection`：自动连接，根据 sender 和 receiver 是否在一个线程来决定使用哪种方式，同一线程直连，不同线程队列连接。
2. `Qt::DirectConnection`：直连。
3. `Qt::QueuedConnection`：队列连接。
4. `Qt::BlockingQueuedConnection`：阻塞队列连接，即虽然是跨线程，但同时希望一个槽函数执行完毕之后才能执行信号的下一步代码。
5. `Qt::UniqueConnection`：唯一连接。

#### 使用方法

在自定义导航按钮的示例程序中有使用到信号槽，在示例 1 和示例 2 中说明，槽函数是自定义槽函数，而信号是按钮对象自带的信号。在实际编程中同样可以自定义信号，并且信号和槽函数都是可以带参数的。同时也自定义了信号与槽函数，用于使用说明。

1. 自定义导航按钮中示例。

```C++
// Widget.h
class Widget : public QWidget
{
    Q_OBJECT
public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

    // 注意使用 slots
public slots: // 或者写成 Q_SLOTS
    // 槽函数
    void btns1_clicked();
    // ...
};
// Widget.cpp
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    btns1 << ui->pushButton << ui->pushButton_2 << ui->pushButton_3 <<
             ui->pushButton_4 << ui->pushButton_5;
    ui->pushButton->setChecked(true);
    for(int i = 0; i < btns1.size(); i++) {
        // 使用 connect 连接信号与槽函数
        connect(btns1.at(i),                 // 按钮对象
                &JNaviButton::clicked,         // 按钮的信号函数
                this,                         // this 指针
                &Widget::btns1_clicked);    // 自定义槽函数

        btns1.at(i)->setShowIcon(true);
        btns1.at(i)->setNormalIcon(QPixmap(":/image/image/right.png"));
        btns1.at(i)->setHoverIcon(QPixmap(":/image/image/right-hover.png"));
        btns1.at(i)->setCheckedIcon(QPixmap(":/image/image/right-pressed.png"));
        btns1.at(i)->setStyleSheet("JNaviButton{font-size:14px;}");
    }
    // ...
}

// 自定义槽函数
void Widget::btns1_clicked()
{
    JNaviButton *btn = dynamic_cast<JNaviButton*>(sender());
    QMessageBox::information(this, "btns1", btn->text());
}
```

1. 自定义信号并发送信号示例。

```C++
// Widget.h
class Widget : public QWidget
{
    Q_OBJECT
public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

signals：// 或者 Q_SIGNALS
    void custom_signal(const QString &msg);

    // 注意使用 slots
public slots: // 或者写成 Q_SLOTS
    // 槽函数
    void custom_slot(const QString &msg);
    // ...
};
// Widget.cpp
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    // 连接自定义信号与槽函数
    connect(this, &Widget::custom_signal, this,  &Widget::custom_slot);
    // 或者 connect(this, SIGNAL(custom_signal()), this,  SLOT(custom_slot()));
    // ...
}

// 自定义槽函数
void Widget::btns1_clicked()
{
    JNaviButton *btn = dynamic_cast<JNaviButton*>(sender());
    emit custom_signal(btn->text());
}

void Widget::custom_slot(const QString &msg)
{
    QMessageBox::information(this, "info", msg);
}
```



## 命名规则

在实验介绍时将布局分为拖拽布局和代码布局。

拖拽布局时拖拽的所有控件都将放在 UI 文件中，这相当于天然将所有的控件类变量进行了分类，在拖拽控件后我们同样需要为其命名。在代码布局时我们将所有的控件变量加上前缀 “ui_”，这样才能区分于不同的变量，而拖拽的控件不需要添加该前缀，因为使用控件时使用 “ui->变量名” 的方式，可以很方便的区分。

除此之外，在命名控件变量名称时还需要加上控件类型的缩写作为前缀，这是为了区分控件类型。以代码布局按钮类型为例，需要命名一个开始按钮：“ui*btnStart”，其中 “ui*” 为控件类型前缀，“btn” 为按钮类型的缩写前缀，“Start” 为变量的名称。

下表是常用控件的缩写，是我自己总结，如果在编码时有更好的想法欢迎提出，缩写表如下：

| 控件类型                 | 缩写名称 | 使用示例       |
| ------------------------ | -------- | -------------- |
| QPushButton、QToolButton | btn      | ui_btnStart    |
| JNaviButton              | nbtn     | ui_nbtnStart   |
| QRadioButton             | rbtn     | ui_rbtnStart   |
| QCheckBox                | check    | ui_checkStart  |
| QComboBox                | cBox     | ui_cBoxStart   |
| QLineEdit                | edit     | ui_editStart   |
| QTextEdit                | text     | ui_textStart   |
| QTimeEdit                | time     | ui_timeStart   |
| QDateEdit                | date     | ui_dateStart   |
| QDateTimeEdit            | dtEdit   | ui_dtEditStart |
| QTableWidget、QTableView | table    | ui_tableStart  |
| QTabWidget               | tab      | ui_tabStart    |
| QGroupBox                | gBox     | ui_gBoxStart   |
| QScrollArea              | sArea    | ui_sAreaStart  |
| QWidget                  | w        | ui_wStart      |
| QStackedWidget           | wStack   | ui_wStackStart |



# 界面基本使用

## 基础函数

`setWindowTitle`设置窗口标题

`setText`设置对话框内容

`setCentralWidget`设置中心窗口

`QHBoxLayout ui_hlayWindow->addStretch();`加弹簧可以保持不同控件的相对位置不变。

## 常用布局器

Qt 布局是很方便的，原因就是有以下 4 种布局类，让我们在布局软件界面时可以很容易达到想要的效果。4 种布局器的属性是类似的，而在用法上 QHBoxLayout 与 QVBoxLayout 几乎相同，QHBoxLayout 与 QVBoxLayout 的不同之处在于 QHBoxLayout 可以设置为垂直方向显示，而 QVBoxLayout 不能设置水平方向显示。

每种布局器都有其特殊的使用场景，在对应的场景下使用起来会显得简单方便、开发快捷。

除了以下 4 种布局方式之外，开发者还能通过继承 QLayout 来自定义布局方式，在课程中以下 4 种布局方式已经能够满足我们的需求，所以就不再单独讲解自定义布局器。

- QHBoxLayout
- QVBoxLayout
- QFormLayout
- QGridLayout

### QHBoxLayout

水平布局，在水平方向上排列控件，即左右排列。

在导航按钮的示例中，右侧的 4 排导航按钮使用的就是水平布局（QHBoxLayout）的方式，且是使用拖拽的方式实现的。下面通过代码布局的方式使用水平布局并说明其相关属性。

在使用代码布局时按钮和布局器都可以声明临时变量，将按钮放在布局器中，最后将布局器设置到窗口中；也可以将按钮和布局器声明为成员变量，后期如果想要使用时同样可以使用。在下面的示例中直接使用临时变量。

1. QHBoxLayout 使用代码布局，在默认情况下，布局器的边距和控件间隔为 6，并且控件居中显示。

```C++
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    JNaviButton *ui_btn1 = new JNaviButton;
    JNaviButton *ui_btn2 = new JNaviButton;
    JNaviButton *ui_btn3 = new JNaviButton;
    JNaviButton *ui_btn4 = new JNaviButton;
    JNaviButton *ui_btn5 = new JNaviButton;

    ui_btn1->setText("button1");
    ui_btn2->setText("button2");
    ui_btn3->setText("button3");
    ui_btn4->setText("button4");
    ui_btn5->setText("button5");

    QHBoxLayout *ui_hLayMain = new QHBoxLayout;
    ui_hLayMain->addWidget(ui_btn1);
    ui_hLayMain->addWidget(ui_btn2);
    ui_hLayMain->addWidget(ui_btn3);
    ui_hLayMain->addWidget(ui_btn4);
    ui_hLayMain->addWidget(ui_btn5);

    setLayout(ui_hLayMain);
}
```

![3-1-QHBoxLayout](https://doc.shiyanlou.com/courses/2623/939394/755c0c86ae83ff0be6b7603261d5b09a-0)

1. 常常在开发时是需要设置边距，设置边距一共有三个重载的函数。setMargin 可以设置左、上、右、下的外边距，设置之后，它们的外边距是相同的；setContentsMargins 与其功能相同，但是可以将左、上、右、下的外边距设置为不同的值。在示例代码中我们使用 setMargin 来设置边距，再对比上面的效果可以看出。

   ```C++
   void setMargin(int)
   void setContentsMargins(int left, int top, int right, int bottom);
   void setContentsMargins(const QMargins &margins);
   ```

![3-2-setMargin](https://doc.shiyanlou.com/courses/2623/939394/2961102f0d1d6815728b44de5eb2a34b-0)

1. 边距同时也要配合控件间隔来使用，导航按钮的例子就是如此，下面示例程序中我们将控件间距设置为 0 后看效果。

```C++
void setSpacing(int);
```

![3-3-setSpacing](https://doc.shiyanlou.com/courses/2623/939394/d709ec51a1e56eaa886ec0dd731187b5-0)

1. 还有一个比较重要的属性可以称之为弹簧，意思是在某个位置添加这个属性后可以将控件伸缩开，就像弹簧一样。如下面示例中在布局器的最后添加一个弹簧，在窗口适配时也是一个重要的属性。有横向弹簧和纵向弹簧，并且可以添加在布局器的任意一个位置。

   ```C++
   ui_hLayMain->addStretch();
   ```

![3-4-addStretch](https://doc.shiyanlou.com/courses/2623/939394/2e86f412e2cd60432a6a3079918ef022-0)

1. 布局器添加控件时默认是垂直和水平方向居中，但也可以设置其它显示方式。同时布局器中也可以添加布局器。

   ```C++
   void addWidget(QWidget *, int stretch = 0, Qt::Alignment alignment = 0);
   void addLayout(QLayout *layout, int stretch = 0);
   ```

   ```C++
   ui_hLayMain->addWidget(ui_btn1, 0, Qt::AlignLeft | Qt::AlignTop);
   ui_hLayMain->addWidget(ui_btn2, 0, Qt::AlignLeft | Qt::AlignTop);
   ui_hLayMain->addWidget(ui_btn3);
   ui_hLayMain->addWidget(ui_btn4, 0, Qt::AlignLeft | Qt::AlignBottom);
   ui_hLayMain->addWidget(ui_btn5, 0, Qt::AlignLeft | Qt::AlignBottom);
   ```

![3-5-Align](https://doc.shiyanlou.com/courses/2623/939394/05bff9233fff1daa59adff89a72b6471-0)

1. 在布局时可以设置控件占据的空间比例。示例中设置 ui_btn1 的拉伸系数为 1，ui_btn2 拉伸系数为 2，当窗体变大时，会优先将 ui_btn2 进行拉伸，当达到一定程度时，再拉伸 ui_btn1，ui_btn2 与 ui_btn2 的宽度比例为 1:2。

   ```C++
   bool setStretchFactor(QWidget *w, int stretch);
   bool setStretchFactor(QLayout *l, int stretch);
   void setStretch(int index, int stretch);
   ```

![3-6-setStretchFactor](https://doc.shiyanlou.com/courses/2623/939394/d019a9b189f2454a16bcd2d12a304c46-0)

1. QHBoxLayout 还有一个可以将水平布局改变为垂直布局，由于已经有 QVBoxLayout 垂直布局器，所以也不需要再使用这个属性。

   ```C++
   void setDirection(Direction);
   ```

### QVBoxLayout

垂直布局，在垂直方向上排列控件，即：上下排列。

在使用方法上与 QHBoxLayout 水平布局器是相同的，除了不能设置显示方向外，故不再举例演示。

### QFormLayout

表单布局，只有两列，但可以有多行，通常情况下左侧添加文本，右侧添加控件或另外的布局器。

1. 表单布局器在布局时一般使用 addRow 方法来添加控件，同时也可以使用 insertRow 来添加，insertRow 多一个行参数，且可以插入到表单的任意位置。

```C++
void addRow(QWidget *label, QWidget *field);
void addRow(QWidget *label, QLayout *field);
void addRow(const QString &labelText, QWidget *field);
void addRow(const QString &labelText, QLayout *field);
void addRow(QWidget *widget);
void addRow(QLayout *layout);
// 对应 3-2-QFormLayout 实验代码
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    QLineEdit *ui_editServer = new QLineEdit;
    QLineEdit *ui_editUser = new QLineEdit;
    QLineEdit *ui_editPasswd = new QLineEdit;
    QComboBox *ui_cBoxDb = new QComboBox;

    QFormLayout *ui_fLayout = new QFormLayout;
    ui_fLayout->addRow("服务器：", ui_editServer);
    ui_fLayout->addRow("用户名：", ui_editUser);
    ui_fLayout->addRow("密   码：", ui_editPasswd);
    ui_fLayout->addRow("数据库：", ui_cBoxDb);

    setLayout(ui_fLayout);
}
```

![3-7-QFormLayout](https://doc.shiyanlou.com/courses/2623/939394/0af9ec8f1c2e956af9bdb499b256f02e-0)

1. 接下来看 QFormLayout 的 RowWrapPolicy 属性，该属性控制表单布局器的显示策略，RowWrapPolicy 是一个枚举类型，有三个属性值，下面直接使用示例来说明属性的作用。

   | 属性名                    | 值   | 描述                                                         |
   | ------------------------- | ---- | ------------------------------------------------------------ |
   | QFormLayout::DontWrapRows | 0    | 输入框始终在标签旁边。在本实验环境下，默认为该属性。         |
   | QFormLayout::WrapLongRows | 1    | 标签有足够的空间适应，如果最小大小比可用空间大，输入框会被换到下一行 |
   | QFormLayout::WrapAllRows  | 2    | 输入框始终在标签下边                                         |

   1. `QFormLayout::DontWrapRows` 属性的例子，我们将第一行中服务器的文本输入框设置为固定长度，文本框始终在文旁边。

      ```C++
      ui_editServer->setFixedWidth(70);
      ```

      ![3-8-DontWrapRows](https://doc.shiyanlou.com/courses/2623/939394/7db43b6b83b3018df8627368e2b12c4b-0)

   2. `QFormLayout::WrapLongRows` 属性的例子。

      ```C++
      ui_fLayout->setRowWrapPolicy(QFormLayout::WrapLongRows);
      ```

      ![3-9-WrapLongRows](https://doc.shiyanlou.com/courses/2623/939394/4460cf4003c4a4aaff1a431a65eccc21-0)

   3. `QFormLayout::WrapAllRows` 属性的例子。

      ![3-10-WrapAllRows](https://doc.shiyanlou.com/courses/2623/939394/ffc9da346829bed8143a43e892ddaef8-0)

2. 设置行对应的控件，使用 QFormLayout 的 ItemRole 属性，ItemRole 时枚举类型，该属性也有三个值。

   ```C++
   void setWidget(int row, ItemRole role, QWidget *widget);
   ```

   | 属性名                    | 值   | 描述                   |
   | ------------------------- | ---- | ---------------------- |
   | QFormLayout::LabelRole    | 0    | 标签                   |
   | QFormLayout::FieldRole    | 1    | 输入框                 |
   | QFormLayout::SpanningRole | 2    | 跨越标签和输入框的控件 |

3. 设置边距，设置边距时与 QHBoxLayout 设置边距相同。窗口和控件。

   ```C++
   void setMargin(int);
   void setContentsMargins(int left, int top, int right, int bottom);
   void setContentsMargins(const QMargins &margins);
   ```

4. 设置间距，由于是表单，可以单独设置水平间距和垂直间距。控件和控件。

   ```C++
   void setSpacing(int spacing);
   void setHorizontalSpacing(int spacing);
   void setVerticalSpacing(int spacing);
   ```

### QGridLayout

栅格布局，也称为网格布局，可像表格一样有多行多列。

栅格布局将位于其中的窗口控件放入网状的栅格之中，将提供给它的空间划分为行和列，并将每个窗口控件插入并管理到正确的单元格。当界面布局比较复杂时使用栅格布局器会使得布局变得简单。

栅格布局器与其他类型的布局器的使用方法有所不同，栅格布局器比其他布局器相对复杂，主要体现在栅格布局添加窗口控件的方式。

下面就一个简单的登录界面作为示例。

```C++
// 声明控件
QLabel *ui_labelLogo = new QLabel(this);
QLineEdit *ui_editUser = new QLineEdit(this);
QLineEdit *ui_editPasswd = new QLineEdit(this);
QCheckBox *ui_checkRember = new QCheckBox(this);
QCheckBox *ui_checkAutoLogin = new QCheckBox(this);
QPushButton *ui_btnLogin = new QPushButton(this);
QPushButton *ui_btnRegister = new QPushButton(this);
QPushButton *ui_btnForgot = new QPushButton(this);

// 设置控件文本信息
ui_editUser->setPlaceholderText("账号");
ui_editPasswd->setPlaceholderText("密码");
ui_editPasswd->setEchoMode(QLineEdit::Password);
ui_checkRember->setText(QStringLiteral("记住密码"));
ui_checkAutoLogin->setText(QStringLiteral("自动登录"));
ui_btnLogin->setText(QStringLiteral("登录"));
ui_btnRegister->setText(QStringLiteral("注册账号"));
ui_btnForgot->setText(QStringLiteral("找回密码"));

// 布局方式
QGridLayout *ui_gLayoutMain = new QGridLayout();
ui_gLayoutMain->addWidget(ui_labelLogo, 0, 0, 3, 1);
ui_gLayoutMain->addWidget(ui_editUser, 0, 1, 1, 2);
ui_gLayoutMain->addWidget(ui_btnRegister, 0, 4, Qt::AlignLeft);
ui_gLayoutMain->addWidget(ui_editPasswd, 1, 1, 1, 2);
ui_gLayoutMain->addWidget(ui_btnForgot, 1, 4, Qt::AlignLeft);
ui_gLayoutMain->addWidget(ui_checkRember, 2, 1, 1, 1);
ui_gLayoutMain->addWidget(ui_checkAutoLogin, 2, 2, 1, 1);
ui_gLayoutMain->addWidget(ui_btnLogin, 3, 1, 1, 2);
ui_gLayoutMain->setHorizontalSpacing(10);
ui_gLayoutMain->setVerticalSpacing(10);
ui_gLayoutMain->setAlignment(Qt::AlignCenter);
ui_labelLogo->setMinimumSize(100, 100);

// 设置对象名称
ui_labelLogo->setObjectName("labelLogo");
ui_btnLogin->setObjectName("btnLogin");
ui_btnRegister->setObjectName("btnRegister");
ui_btnForgot->setObjectName("btnForgot");
// 设置样式
QString style = "QPushButton#btnLogin{background-color:rgb(0, 165, 253);color:white;height:20px;}"
    "QPushButton#btnRegister,#btnForgot{border:none;color:blue}"
    "QLabel#labelLogo{image:url(:/image/header.png)}";
qApp->setStyleSheet(style);

setLayout(ui_gLayoutMain);
```

![3-11-QGridLayout](https://doc.shiyanlou.com/courses/2623/939394/ee4441e0e17bc24e33840ac95997e2d3-0)

1. 添加窗口控件或其他布局器至栅格局器中。当前单元将从 row 和 column 开始，扩展到 rowSpan 和 columnSpan 指定的倍数的行和列。如果 rowSpan 或 columnSpan 的值为 -1，则窗口控件将扩展到布局的底部或者右边边缘处，`Qt::Alignment` 为对齐方式。

   ```C++
   void addWidget(QWidget *, int row, int column, Qt::Alignment = 0);
   void addWidget(QWidget *, int row, int column, int rowSpan, int columnSpan, Qt::Alignment = 0);
   void addLayout(QLayout *, int row, int column, Qt::Alignment = 0);
   void addLayout(QLayout *, int row, int column, int rowSpan, int columnSpan, Qt::Alignment = 0);
   ```

2. 设置行、列的伸缩空间。

   ```C++
   // 设置行、列的比例
   void setRowStretch(int row, int stretch);
   void setColumnStretch(int column, int stretch);
   ```

3. 设置间距。

   ```C++
   void setSpacing(int spacing);            // 可以同时设置水平、垂直间距，设置之后，水平、垂直间距相同。
   void setHorizontalSpacing(int spacing);    // 设置水平间距
   void setVerticalSpacing(int spacing);    // 垂直间距
   ```

4. 设置最小行高、最小列宽。

   ```C++
   void setRowMinimumHeight(int row, int minSize);
   void setColumnMinimumWidth(int column, int minSize);
   ```

5. 获取行数、列数。

   ```C++
   int columnCount();
   int rowCount();
   ```

6. 设置对齐方式。

   ```C++
   void setAlignment(Qt::Alignment a);
   ```

## 分栏布局

分栏布局是将软件界面分为两个或两个以上的区域，可以在水平或者垂直方向对窗口分栏布局。QSplitter 是 Qt 专门用于分栏的一个类，QSplitter 类似于布局器，但 QSplitter 只能添加 QWidget 或继承自 QWidget 的窗口控件。同时 QSplitter 对象也可以放在其他窗口控件或布局器中，因为 QSplitter 也是继承自 QFrame。

在 Qt 中所有的窗口控件都直接或间接的继承了 QWidget 类。例如 QTextEdit 控件，QTextEdit 继承自 QAbstractScrollArea，QAbstractScrollArea 继承自 QFrame，QFrame 继承自 QWidget。

为了演示效果更加明显，下面示例程序中使用了 QTextEdit 富文本窗口控件，示例中包含了 QSplitter 的常见用法。

```C++
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    // 添加窗口控件
    QSplitter *ui_hSplitter = new QSplitter(Qt::Horizontal, 0);
    QSplitter *ui_vSplitter = new QSplitter(Qt::Vertical);
    QTextEdit *ui_text1 = new QTextEdit;
    QTextEdit *ui_text2 = new QTextEdit;
    QTextEdit *ui_text3 = new QTextEdit;
    ui_hSplitter->addWidget(ui_text1);
    ui_vSplitter->addWidget(ui_text2);
    ui_vSplitter->addWidget(ui_text3);
    ui_hSplitter->addWidget(ui_vSplitter);
    // 设置窗口最小尺寸
    ui_hSplitter->setMinimumSize(500, 400);
    ui_hSplitter->setWindowTitle(QStringLiteral("3-4-QSplitter"));
    // 设置伸缩比例
    ui_vSplitter->setStretchFactor(0, 1);
    ui_vSplitter->setStretchFactor(1, 2);
    ui_hSplitter->setStretchFactor(0, 1);
    ui_hSplitter->setStretchFactor(1, 1);
    // 设置分栏条宽度
    ui_hSplitter->setHandleWidth(1);
    // 设置拖动风格
    ui_vSplitter->setOpaqueResize(false);
    ui_hSplitter->show();
    // 设置关闭窗口时释放内存
    ui_hSplitter->setAttribute(Qt::WA_DeleteOnClose);

    return a.exec();
}
```

![3-12-QSplitter](https://doc.shiyanlou.com/courses/2623/939394/ff3b661b411bae3a2ed4cc0cab37413d-0)

#### 添加控件

QSplitter 添加其他窗口控件有两种方式：一种是在实例化其他窗口控件时将 QSplitter 的对象作为父类传入；第二种是通过 addWidget 方法将其他窗口控件添加至分栏中。

```C++
// 第一种方式
QSplitter *ui_splitter = new QSpliiter(Qt::Horizontal);
QWidget *ui_w1 = new QWidget(ui_spliiter);

// 第二种方式
QSplitter *ui_splitter = new QSpliiter;
QWidget *ui_w1 = new QWidget;
ui_splitter.addWidget(ui_w1);
```

#### 设置最小尺寸

示例程序中使用 QTextEdit，这是因为 QWidget 的背景色与 QSplitter 背景色相同，演示不明显。而设置 QSplitter 窗口最小尺寸是为了保证窗口在缩小至最小时不至于看不见，也是软件开始的体验效果。

```C++
void setMinimumSize(const QSize &s);
void setMinimumSize(int w, int h);
```

#### 设置样式

在项目中，如果为了软件的样式，需要设置分栏拖动条的样式同样也是可以做到的。

```C++
// 设置颜色，可以使用 QSS 给分栏拖动条设置颜色甚至是渐变色
QString style = "QSplitter{background-color:red;}";
ui_hSplitter->setStyleSheet(style);
```

#### 设置宽度

设置分栏拖动条宽度有两种方式：一种是使用接口；另一种是使用 QSS 样式。

```C++
// 程序接口设置分栏拖动条宽度
ui_hSplitter->setHandleWidth(1);

// QSS 设置分栏拖动条宽度
QString style = "QSplitter{height:1px;}";
ui_hSplitter->setStyleSheet(style);
```

#### 设置拖动方式

拖动方式即实时刷新拖动和拖动完成后刷新，默认实时拖动刷新。

```C++
void setOpaqueResize(bool opaque = true);
```

#### 设置窗口比例

在初始化情况窗口显示比例可以通过接口设置。其中 index 为插入窗口控件序号，stretch 为伸缩比例。

```C++
void setStretchFactor(int index, int stretch);
```

#### 注意事项

- 在 main 函数中，如果在堆上实例化一个窗口类，且没有指定父类，需要设置关闭窗口是释放内存。

  ```C++
  ui_splitter->setAttribute(Qt::WA_DeleteOnClose);
  ```

- 在使用 QSplitter 时设置最小窗口尺寸。如果 QWidget 窗口控件中没有其他控件且不设置最小尺寸时，整个窗口控件将被收缩起来，并且拉伸后的窗口会重叠。

  ```C++
  ui_splitter->setMinimumSize(500, 400);
  ```

- 不能使用拖拽的方式来使用 QSplitter。



## 代码示例

在布局时使用拖拽的方式添加对应的子控件，而将控件加入布局器的部分使用代码布局实现，再通过设置布局器和窗口控件的样式、布局等最终完成自定义标题栏的布局。

1. 向布局器中添加窗口控件或其他布局器，按照顺序添加即可。

   ```C++
   // jtitlebar.cpp 构造函数中
   QHBoxLayout *ui_hlayWindow = new QHBoxLayout;
   ui_hlayWindow->addWidget(ui->btnLogo);            // 添加 LOGO 按钮
   ui_hlayWindow->addLayout(ui_hLayNaviButton);    // 添加导航按钮所在布局器
   ui_hlayWindow->addStretch();                    // 添加一个弹簧
   ui_hlayWindow->addLayout(ui_hlayToolButton);    // 添加工具按钮所在布局器
   ui_hlayWindow->addWidget(ui->frameLine);        // 添加竖线
   ui_hlayWindow->addLayout(ui_hlayWinButton);        // 添加窗口功能按钮所在布局器
   
   setLayout(ui_hlayWindow);
   ```

2. 设置布局器属性。展示的代码中是最外层布局器，所以讲边距和间距属性都设置为 0，如果里面的控件有布局需要在去设置内层布局器属性。

   ```C++
   ui_hlayWindow->setMargin(0);
   ui_hlayWindow->setSpacing(0);
   ```

3. 设置控件属性。

   ```C++
   ui->btnLogo->setMinimumWidth(180);                // 为 LOGO 按钮设置最小宽度
   ui->frameLine->setFixedSize(1, 20);                // 将竖线的尺寸设为固定值
   ```

4. 导航按钮属性需要单独设置。导航按钮默认为左右布局的方式显示，需要对显示的属性进行设置。将同一个小区域内同类窗口控件放在一个容器中，可方便为一组按钮设置同一属性。如 m_listNaviButtons 是 QList 类型链表，用来单独存放导航按钮。

   ```C++
   for(auto btn : m_listNaviButtons) {
       btn->setShowIcon(true);                        // 设置导航按钮显示图标
       btn->setIconSpace(20);                        // 设置图标左侧边距
       btn->setPaddingLeft(30);                    // 设置文字左侧边距
   }
   ```









### 文本编辑

文本编辑功能，通常会用到 Qt 类的 `QTextEdit` 、 `QTextDocument` 、 `QTextBlock` 、 `QTextList` 、 `QTextFrame` 、 `QTextTable` 、 `QTextCharFormat` 、 `QTextBlockFormat` 、 `QTextListFormat` 、 `QTextFrameFormat` 以及 `QTextTableFormat` 等等。

![image-20220214102100581](E:\Document\Typora\img\image-20220214102100581.png)

在本实例中的文本编辑中，主要采用的是对 `QTextDocument` 和 `QTextCharFormat` 的操作，主要操作对象是新建显示 `ShowWidget` 类中的文本编辑框控件 `text` 。

+ 字体控件

~~~
// 文本操作相关控件，并没有放置到菜单栏中
QLabel *fontLabel1;    			// 字体型号标签
QFontComboBox *fontComboBox;    // 字体型号选择控件
QLabel *fontLabel2;        		// 字体大小标签
QComboBox *sizeComboBox;    	// 字体大小选择控件
QToolButton *boldBtn;    		// 字体加粗工具栏按钮控件
QToolButton *italicBtn;    		// 字体斜体工具栏按钮控件
QToolButton *underlineBtn;    	// 下划线工具栏按钮控件
QToolButton *colorBtn;        	// 字体颜色设置工具栏按钮控件
QToolBar *fontToolBar;        	// 字体工具栏控件声明
~~~

+ 设置字体格式

~~~
//字体修改相关操作
QTextCharFormat fmt;
//设置文本格式
fmt.setFontFamily(comboStr);
mergeFormat(fmt);
~~~

+ 光标选择文本部分操作

~~~
//光标选择文本部分操作
//获取当前showWidget窗口下 textEdit 控件中的光标对象
QTextCursor cursor = showWidget->text->textCursor();	// 继承QWidget的文本显示类
if(!cursor.hasSelection())
{
cursor.select(QTextCursor::WordUnderCursor);
}
//被光标选中的文本进行格式变换
cursor.mergeCharFormat(format);
showWidget->text->mergeCurrentCharFormat(format);
~~~

+ 设置字体加粗

~~~
void TextProcess::ShowBoldBtn()
{
    QTextCharFormat fmt;
    //设置字体加粗
    fmt.setFontWeight(boldBtn->isChecked() ? QFont::Bold : QFont::Normal);
    showWidget->text->mergeCurrentCharFormat(fmt);
}
~~~

+ 设置字体斜体

~~~
void TextProcess::ShowItalicBtn()
{
    QTextCharFormat fmt;
    //设置字体斜体
    fmt.setFontItalic(italicBtn->isChecked());
    showWidget->text->mergeCurrentCharFormat(fmt);
}
~~~

+ 设置下划线：`setFontUnderline`
+ 设置字体颜色`setForeground`

+ 字体改变

`showWidget->text->mergeCurrentCharFormat(fmt);`造成`(currentCharFormatChanged(const QTextCharFormat &)`

```
connect(showWidget->text, SIGNAL(currentCharFormatChanged(const QTextCharFormat &)), this, SLOT(ShowCurrentFormatChanged(const QTextCharFormat &)));
```

子控件`showWidget`字体格式一旦变化，更新设置界面的状态。

~~~
fontComboBox->setCurrentIndex(fontComboBox->findText(fmt.fontFamily()));// QFontComboBox *fontComboBox
sizeComboBox->setCurrentIndex(sizeComboBox->findText(QString::number(fmt.fontPointSize()))); // QComboBox *sizeComboBox
boldBtn->setChecked(fmt.font().bold()); // QToolButton *boldBtn
italicBtn->setChecked(fmt.font().italic()); // QToolButton *italicBtn
underlineBtn->setChecked(fmt.fontUnderline()); // QToolButton *underlineBtn
~~~





## 样式



SS 的语法跟 CSS 语法很类似，可以说 QSS 源自于 CSS，具体介绍、语法和使用方式将在下一实验《QSS 样式》详细讲解，本次实验就使用到的样式内容做一个简单了解。

设置的 QSS 样式其实质是一个带有特殊格式的字符串，设置样式后会自动解析样式。

```C++
QString style = "QWidget#JTitleBar{background: qlineargradient(x1:0,y1:0,x2:1,y2:0,stop:0 #313947,stop:1 #34375E);}"
                "QPushButton{border: none;}"
                "QPushButton#btnLogo{color:white;}"
                "QPushButton#btnProject{image: url(:/icon/icon/project.png); color:white;}"
                "QPushButton#btnProject:hover{image: url(:/icon/icon/project-hover.png);}"
                "QPushButton#btnEdit{image: url(:/icon/icon/analysis.png); color:white;}"
                "QPushButton#btnEdit:hover{image: url(:/icon/icon/analysis-hover.png);}"
                "QPushButton#btnSetting{image: url(:/icon/icon/setting.png); color:white;}"
                "QPushButton#btnSetting:hover{image: url(:/icon/icon/setting-hover.png);}"
                "QPushButton#btnAbout{image: url(:/icon/icon/about.png);}"
                "QPushButton#btnAbout:hover{image: url(:/icon/icon/about-hover.png);}"
                "QPushButton#btnMin{image: url(:/icon/icon/min.png);}"
                "QPushButton#btnMin:hover{image: url(:/icon/icon/min-hover.png);"
                "background-color: red;}"
                "QPushButton#btnMax{image: url(:/icon/icon/max.png);}"
                "QPushButton#btnMax:hover{image: url(:/icon/icon/max-hover.png);"
                "background-color: red;}"
                "QPushButton#btnClose{image: url(:/icon/icon/close.png);}"
                "QPushButton#btnClose:hover{image: url(:/icon/icon/close-hover.png);"
                "background-color: red;}"
                "QFrame#frameLine{background-color: black;}";
setStyleSheet(style);
```

### QSS 样式

1. 设置样式。qApp 是一个全局的对象，是 Qt 窗口类自带的。通过向 qApp 设置样式可以样式设置。也可以单独为某一窗口控件编写 QSS，然后单独设置到该控件中。在使用时可以放在 main 函数中，也可以直接放在标题栏类中。考虑到是在整个项目中开发，后期需要考虑到换肤功能，所以在 main 函数中设置为全局样式。

   ```C++
   // 设置窗口控件样式，标题栏类中
   this->setStyleSheet(style);
   
   // 设置全局样式
   qApp->setStyleSheet(style);
   ```

2. 设置共同样式。在标题栏中所有的按钮都不需要边框，所以将所有按钮设置为无边框，这样可以避免重复编写同类多控件共同的样式。

   ```
   "QPushButton{border: none;}"
   ```

3. 设置指定某一控件样式。#btnProject 是指定了该按钮控件的名称，其中 btnProject 是控件的对象名称。拖拽控件时系统默认将控件的变量名称设置为对象名称；而代码布局时由于代码添加控件，默认控件对象名称为空，需要单独设置，一般情况下设置为控件变量名，保持名称统一。

   ```C++
   "QPushButton#btnProject{image: url(:/icon/icon/project.png); color:white;}"
   ```

4. 指定属性样式。:hover 表示鼠标上划是设置的样式。

   ```C++
   "QPushButton#btnProject:hover{image: url(:/icon/icon/project-hover.png);}"
   ```

5. 设置背景图片。由于标题栏中按钮只显示图标，所以讲整个图片设置到按钮中，使用了 `{image: url(:/icon/icon/project.png); color:white;}` 。

6. 设置渐变色。

   ```C++
   "QWidget#JTitleBar{background: qlineargradient(x1:0,y1:0,x2:1,y2:0,stop:0 #313947,stop:1 #34375E);}"
   ```

### 样式加载

加载方式的本质都是向类中设置 QSS 样式。

1. 控件直接设置样式。

   ```C++
   myPushButton->setStyleSheet("QPushButton { color: red; background-color: white; }");
   ```

2. 整体设置样式。

   ```C++
   qApp->setStyleSheet("QPushButton { color: red; background-color: white; }");
   ```

3. 加载 qss 文件外部路径，在编写 QSS 文件时主要 qss 文件的编码需要存为 utf-8，其他的编码格式设置样式后无效。如果使用这种方式加载 qss 文件，在生成安装包时需要将 qss 文件添加到软件目录下。

   ```C++
   QFile file("./style.qss");
   //只读方式打开文件
   file.open(QFile::ReadOnly);
   
   QString style = QLatin1String(file.readAll());
   
   qApp->setStyleSheet(style);
   ```

4. 加载 qss 文件资源路径，需要先将 qss 文件加载到资源文件中，再使用的时候可以使用资源路径。该方式在生成安装包后不需要 qss 文件。

   ```C++
   QFile file(":/qss/style.qss");
   //只读方式打开文件
   file.open(QFile::ReadOnly);
   
   QString style = QLatin1String(file.readAll());
   
   qApp->setStyleSheet(style);
   ```

### 按钮样式

QSS 设置按钮样式可以将按钮打造成自己想要的样式，下面的示例使用图标 + 文字为按钮设置样式，在示例中左侧一列没有添加样式，只是为图标添加了图标和文字，右侧一列为按钮添加了 QSS 样式，图标是设置了背景图片。

![5-2-styleButton](https://doc.shiyanlou.com/courses/2623/939394/cf8f3afcd4f9379ca73832e7c8aef087-0)

1. 设置按钮边框，将按钮边框宽度、实线、颜色。

   ```C++
   border: 1px solid rgb(50, 56, 82);
   ```

2. 设置按钮圆角。

   ```C++
   // 设置 4 个角都为圆角
   border-radius: 5px;
   
   // 单独设置一个角为圆角
   border-top-left-radius: 5px;
   border-top-right-radius: 5px;
   border-bottom-right-radius: 5px;
   border-bottom-left-radius: 5px;
   ```

3. 背景颜色，在设置颜色属性时可以添加透明属性。

   ```C++
   background-color: rgba(161, 170, 207, 30);
   ```

4. 前景色，前景色就是字体的颜色。

   ```C++
   color: rgb(50, 56, 82);
   ```

5. 字体大小，除了设置字体大小还可以设置字体、字号等。

   ```C++
   // 字体大小
   font-size: 14px;
   
   // 字体
   font-family: "Microsoft YaHei";
   
   // 字体样式，斜体
   font-style: italic
   ```

6. 背景图片，设置背景图片有两种属性，但实际这两种是不一样的。

   ```C++
   // 这种方式是在子控件的内容矩形中绘制的图像
   image: url(:/icon/icon/lsave.png);
   
   // background-image 属性属于 background 属性的一个子属性
   background-image: url(:/icon/icon/lsave.png);
   ```

### 日历控件

日历控件是开发中比较常用的控件，在界面设计中经常结合 QDateEdit 使用，如下示例中所示，第一个日历控件使用 QCalendarWidget 类，控件默认使用自带样式。第二个控件使用了 QDateEdit 类，并设置为弹出日历控件选择日期的方式，对弹出的日历控件使用了部分 QSS 设置样式。

![5-3-styleCalendarWidget](https://doc.shiyanlou.com/courses/2623/939394/a7d77c3080f0f29701b9cf0e3712b15b-0)

1. 日历控件布局。

   ![5-4-calendarWidgetLayout](https://doc.shiyanlou.com/courses/2623/939394/9cba5d33a90a1633fcdd5d1f85965b69-0)

2. 子控件对象名，下面是列出了常用涉及的日历子控件。如果要获取日历控件或其他控件的子对象，可以将所有的对象类型和名称都遍历出来，可参考下面附有代码。

   - `qt_calendar_navigationbar`：日历控件导航栏，即上个月、下个月、月选择、年选择按钮所在栏目。
   - `qt_calendar_prevmonth`：日历控件上个月按钮。
   - `qt_calendar_monthbutton`：日历控件月选择按钮。
   - `qt_calendar_yearbutton`：日历控件年选择按钮。
   - `qt_calendar_nextmonth`：日历控件下个月按钮。
   - `qt_calendar_calendarview`：日历控件日历区域。

   ```C++
   // 遍历一个类的所有子类
   void dumpStructure(const QObject *obj, int spaceCount)
   {
       qDebug() << QString("%1%2 : %3")
                   .arg("", spaceCount)
                   .arg(obj->metaObject()->className())    // 对象类型
                   .arg(obj->objectName());                // 对象名称
   
       QObjectList list = obj->children();                    // 获取当前类的子类对象
   
       foreach (QObject * child, list) {
           dumpStructure(child, spaceCount + 4);
       }
   }
   ```

3. 设置 QDateEdit 样式，可以设置边框、颜色、图标等样式。

   ```C++
   QDateEdit { border: 1px solid rgb(50, 56, 82); border-radius: 3px; color: rgb(50, 56, 82); }
   QDateEdit:hover { border: 1px solid rgb(18, 150, 219); }
   QDateEdit::drop-down { width: 18px; margin-right: 3px; image: url(:/image/calendar.png); }
   ```

4. 设置日历控件导航栏背景色为渐变色。

   ```C++
   QCalendarWidget QWidget#qt_calendar_navigationbar { "background-color: qlineargradient(x1: 0, y1: 0, x2: 1, y2: 0, stop: 0 #313947, stop: 1 #34375E); }
   ```

5. 设置日历控件全部按钮样式。

   ```C++
   QCalendarWidget QToolButton { color: white; border: none; }
   ```

6. 日历控件设置样式，添加图图标。

   ```C++
   QCalendarWidget QToolButton#qt_calendar_monthbutton { width: 40px; }
   QCalendarWidget QToolButton#qt_calendar_prevmonth { qproperty-icon: url(:/image/calendar_pre_month.png); }
   QCalendarWidget QToolButton#qt_calendar_nextmonth { qproperty-icon: url(:/image/calendar_next_month.png); }
   ```

7. 设置步长调节器样式。

   ```C++
   QCalendarWidget QSpinBox::up-button { image: url(:/image/calendar_pre_year.png); background-color: rgba(255, 255, 255, 100); width:15px; }
   QCalendarWidget QSpinBox::up-button:hover { background-color: rgba(215, 218, 223, 100); }
   QCalendarWidget QSpinBox::down-button { image: url(:/image/calendar_next_year.png); background-color: rgba(255, 255, 255, 100); width:15px; }
   QCalendarWidget QSpinBox::down-button:hover { background-color: rgba(215, 218, 223, 100); }
   ```

### 滚动条样式

带有滚动条的控件比较多，如 QTextEdit、QTableWidget、QTableView、QStackedWidget 等等，而滚动条的功能就是让控件在有限的空间内显示更多的内容时可以滚动窗口查看更多的内容。这些滚动条本身也是控件类，可以对滚动条的样式进行设置。如下图示例中，对 QTableWidget 类的滚动条样式做了简单的设置。

![5-5-qssTableWidget](https://doc.shiyanlou.com/courses/2623/939394/ec51a3ba6d518444ba9576b3978ed729-0)

1. 滚动条一本都分为垂直滚动条和水平滚动条，分别设置滚动条的背景色。

   ```C++
   QScrollBar:horizontal { background-color: transparent;                /*设置垂直背景色继承父控件*/
   border: none; height: 8px; }                                        /*设置垂直滚动条宽度*/
   QScrollBar::vertical { background-color: rgba(255, 255, 255, 0);    /*设置水平背景色透明*/
   border: none; width: 8px; }                                            /*设置水平滚动条宽度*/
   ```

2. 设置滚动条颜色。

   ```C++
   QScrollBar::handle:vertical, QScrollBar::handle:horizontal {
   background: rgb(204, 204, 204);                                     /*同时设置垂直、水平滚动条颜色*/
   width: 8px;                                                         /*同时设置垂直、水平滚动条宽度*/
   border-radius: 2px; }                                                /*同时设置垂直、水平滚动条圆角*/
   ```

3. 设置进度控制按钮样式，示例中只是简单的设置，对设置其他样式，如添加图标、背景色等。

   ```C++
   QScrollBar::add-line:vertical, QScrollBar::add-line:horizontal {
   background: rgb(204, 204, 204); }                                    /*同时设置垂直、水平滚动条进度加按钮样式*/
   QScrollBar::sub-line:vertical, QScrollBar::sub-line:horizontal {
   border: none; }"                                                    /*同时设置垂直、水平滚动条进度减按钮样式*/
   ```

### QT换肤

换肤的实质是重新加载 QSS 样式，示例中通过点击按钮来切换窗口的颜色，示例中是最简单的原理，掌握了之后对复杂的布局也能实现换肤功能。

![5-6-changeSkin](https://doc.shiyanlou.com/courses/2623/939394/2aa39be3983175f21ebdb0b10b46f7eb-0)

1. 准备工作，首先需要创建 3 个 .qss 文件，并将 3 个 .qss 文件加载到工程中。

   ![5-7-skin_qss](https://doc.shiyanlou.com/courses/2623/939394/1261c923c70b6097f748ade618616461-0)

2. 编写 qss 文件，在示例中简单对窗口设置了背景色。

   ```C++
   QWidget#Widget {
       background-color: red;
   }
   ```

3. 实现换肤类。示例中将该类放在 widget.h 文件中，并且需要添加头文件 `#include <QFile>` 和 `#include <QApplication>`。

   ```C++
   class CommonHelper
   {
   public:
       static void setStyle(const QString &style) {
           QFile qss(style);
           qss.open(QFile::ReadOnly);
           qApp->setStyleSheet(qss.readAll());
           qss.close();
       }
   };
   ```

4. 初始化时加载皮肤。

   ```C++
   // m_styleList 为 QStringList 类型成员变量
   m_styleList.append(":/qss/style1.qss");
   m_styleList.append(":/qss/style2.qss");
   m_styleList.append(":/qss/style3.qss");
   
   // 换肤类的使用方式
   on_pushButton_clicked();
   ```

5. 点击按钮实现切换皮肤。

   ```C++
   // 在按钮事件中实现切换皮肤
   void Widget::on_pushButton_clicked()
   {
       static int i = 1;
   
       if(i == m_styleList.size())
           i = 0;
       CommonHelper::setStyle(m_styleList.at(i));
       ++i;
   }
   ```

### tips

+ 渐变

渐变有三种：QLinearGradient、QConicalGradient 、 QRadialGradient

**QLinearGradient线性渐变**

构造函数函数如下:

```
QLinearGradient ( qreal x1, qreal y1, qreal x2, qreal y2 )
//其中x1,y1表示渐变起始坐标, x2,y2表示渐变终点坐标
//如果只有x相等,则表示垂直线性渐变,如果只有y相等,则表示平行线性渐变,否则就是斜角线性渐变
```

![image-20220216060008613](E:\Document\Typora\img\image-20220216060008613.png)

+ 放大缩小，右上角图标移动



### 注意事项

1. 当样式中指定相同的属性具有不同的值时，就会出现冲突。在例子中将所有按钮的前景色都设置为红色，而后面又将单个一个按钮前景色设置为蓝色，出现这种情况时，编译器会考虑到的选择器的特殊性，QPushButton#okButton 是指定了具体某个对象，所有会优先为将这个控件前景色设置为蓝色。

   ```C++
   QPushButton { color: red; }
   QPushButton#okButton { color: blue; }
   ```

2. 多个状态的冲突。例如下面第一种情况下，hover 和 enabled 两种状态是一种并行关系，这个时候就和设置顺序有关。默认情况下按钮时处于 enabled 状态，表示按钮只要在 enabled 状态前景色都是红色，hover 状态是鼠标上划时前景色为蓝色，这种情况下由于 enabled 状态后设置，所以按钮不区分鼠标上划状态，前景色都为红色。当将设置的顺序改变后，hover 状态设置的前景色将会起作用，鼠标上划时蓝色。

   ```C++
   // 2. enabled 和 hover 状态，前景色都为红色
   QPushButton:hover { color: blue; }
   QPushButton:enabled { color: red; }
   
   // 2. enabled 和 hover 状态，正常状态前景色为红色，鼠标上划状态前景色为蓝色
   QPushButton:enabled { color: red; }
   QPushButton:hover { color: blue; }
   
   // 3. enabled 和 hover 状态，将和第 2 种情况达到相同效果
   QPushButton:hover:enabled { color: blue; }
   QPushButton:enabled { color: red; }
   ```

3. 优先选择器，QPushButton 继承自 QAbstractButton，并且有 color 属性的冲突。QPushButton 比 QAbstractButton 更具体，本应该按钮文本颜色为红色。然而，对于 QSS 的计算，所有的类型选择具有相同的特殊性，最后出现的规则优先，所以实际上按钮文本颜色为蓝色。

   ```C++
   // 后设置覆盖前设置的属性，交换顺序即可改变
   QPushButton { color: red }
   QAbstractButton { color: blue }
   ```

4. 级联效应。QSS 可以在 QApplication、父部件、子部件中设置。冲突发生时，不论冲突规则的特殊性，部件自身的样式表总优先于任何继承样式表。如下示例，控件前景色将是蓝色。

   ```C++
   // 设置至 qApp 中
   qApp->setStyleSheet("QPushButton { color: white; }");
   
   // 设置控件样式
   myPushButton->setStyleSheet("color: blue;");
   ```

5. 继承性。假设 QGroupBox 中包含 QPushButton，QPushButton 不会继承其父 QGroupBox 的颜色，而是显示系统的颜色。

   ```C++
   qApp->setStyleSheet("QGroupBox { color: red; } ");
   ```

6. 只能使用 /_ 注释内容 _/ 注释 QSS 内容。

## 控件提升

1. 鼠标右击工程，选中添加新文件。

   ![2-10-add_class](https://doc.shiyanlou.com/courses/2623/939394/56851c49944d0226fe28fc9fb178e68e-0)

2. 选中添加 C++ 类。

   ![2-11-add_class](https://doc.shiyanlou.com/courses/2623/939394/e8689e23bcb43e0a7c91637082331a7f-0)

3. 定义类名称 `JNaviButton` 并选择继承的基类 `QPushButton`。

   ![图片描述](https://doc.shiyanlou.com/courses/2623/600404/88dfb88bd277f69fcacde49224bdfaee-0)

4. 选择工程管理并完成添加。

   ![2-13-add_class](https://doc.shiyanlou.com/courses/2623/939394/25fcf444b76e8675f67746cda18f29bc-0)

### 提升控件类

自定义按钮时如果还想使用 Qt Designer 来拖拽控件布局，需要多一个步骤，将原来的 QPushButton 类提升为 JNaviButton 类，在使用时还是正常像 QPushButton 控件一样使用，但其已经变成了我们自定义的 JNaviButton 类了。具体方法如下：

1. 拖拽一个 QPushButton 按钮到界面中。

2. 右击按钮，选择提升为...

   ![2-14-promote_control](https://doc.shiyanlou.com/courses/2623/939394/138d5807a06c874480b9a5a7ac4ad948-0)

3. 选择 JNaviButton 类。

   ![2-15-promote_control](https://doc.shiyanlou.com/courses/2623/939394/ae49db7d9b1e5cadf174cac771fdcdaa-0)

## 鼠标事件

鼠标事件是软件的标题栏的必要部分，需要将鼠标事件分为几个功能点。

- 窗口正常状态时双击标题栏窗口切换至最大化状态，最大化状态时双击标题栏窗口切换至正常状态。
- 鼠标拖动标题栏可移动软件窗口，移动至屏幕顶部切换至最大化状态，移动离开屏幕顶部切换至正常状态。
- 鼠标点击最小化、最大化和关闭按钮触发窗口执行最大化、最小化和关闭事件。

#### 按钮事件

在窗口功能区域的最小化、最大化和关闭按钮需要实现接受按钮的点击事件在实现相关的功能。

1. 最小化，直接在连接信号槽时使用了匿名函数。

   ```C++
   connect(ui->btnMin, &QPushButton::clicked, this, [=](){ parentWidget()->showMinimized(); });
   ```

2. 最大化状态和正常状态切换功能。同样使用了匿名函数， `winStatus_changed();` 函数中实现了最大化状态和正常状态切换的功能。

   ```C++
   connect(ui->btnMax, &QPushButton::clicked, [=](){ winStatus_changed(); });
   ```

3. 关闭按钮功能。

   ```C++
   connect(ui->btnClose, &QPushButton::clicked, [=](){ parentWidget()->close(); });
   ```

4. 获取父类窗口指针。在本次实验中，标题栏作为一个单独的窗体实现，然后将标题栏放置到主窗口中，所以在实现标题栏的事件时其实质应该是主窗口中功能，而当标题栏放置到主窗口布局中后可以直接获取到主窗口的指针，这样就可以直接实现其相应的窗口功能。

   ```C++
   QWidget *parentWidget();
   ```

#### 鼠标功能事件

1. 重写鼠标事件。首先需要包含鼠标事件头文件，在 QWidget 类已经实现鼠标事件，只需要重写事件即可。

   ```C++
   #include <QMouseEvent>
   
   class JTitleBar : public QWidget
   {
   public Q_SLOTS:
       virtual void mousePressEvent(QMouseEvent *event) override;
       virtual void mouseReleaseEvent(QMouseEvent *event) override;
       virtual void mouseMoveEvent(QMouseEvent *event) override;
       virtual void mouseDoubleClickEvent(QMouseEvent *event) override;
       // ...
   }
   ```

2. 鼠标双击事件。在实现鼠标双击事件时需要使用变量保存上一次状态，在切换状态时需要状态判断。

   ```C++
   void JTitleBar::mouseDoubleClickEvent(QMouseEvent *event)
   {
       // 判断是鼠标左键
       if(event->button() == Qt::LeftButton) {
           if(NORMAL == m_winStatus) {            // 当前处于正常状态则切换至最大化状态
               parentWidget()->setWindowState(Qt::WindowMaximized);
               m_winStatus = MAXIMIZED;
               return;
           }
           if(MAXIMIZED == m_winStatus){        // 当前处于最大化状态则切换至正常状态
               parentWidget()->setWindowState(Qt::WindowNoState);
               m_winStatus = NORMAL;
               return;
           }
       }
   }
   ```

3. 鼠标移动事件。在鼠标移动事件中不能通过鼠标事件指针获取到鼠标的键值，也就无法判断是鼠标左键、中间或右键，而在实现鼠标拖动窗口一般使用的是鼠标左键，所以需要配合鼠标按下、抬起事件来实现。

   ```C++
   void JTitleBar::mouseMoveEvent(QMouseEvent *event)
   {
       if(m_bLeftButtonPressed) {
           if(m_bDragMax) {
               return;
           }
           if(MAXIMIZED == m_winStatus){        // 当前处于最大化状态时切换至正常状态
               parentWidget()->setWindowState(Qt::WindowNoState);
               m_winStatus = NORMAL;
           }
           else {
               if(event->globalY() <= 2) {        // 判断距离屏幕顶部，距离小于 2 时切换至最大化状态
                   parentWidget()->setWindowState(Qt::WindowMaximized);
                   m_winStatus = MAXIMIZED;
                   m_bDragMax = true;
                   return;
               }
               else {                            // 正常移动窗口
                   QPoint p = event->pos() - m_startPos;
                   parentWidget()->move(parentWidget()->x() + p.x(), parentWidget()->y() + p.y());
               }
           }
       }
   ```









# QT函数调用

## 对话框类

### 标准文件对话框类

Qt5 中标准文件对话框的使用主要依赖 `QFileDialog` 类，其中关于打开并显示标准文件对话框是使用该类的 `getOpenFileName()` 静态成员函数，该函数返回用户选择的文件名，当用户选择文件时，返回被选中的文件的文件名，当选择 `取消(Cancel)` 按键时返回一个空串。具体形参描述说明如下：

~~~
QString QFileDialog::getOpenFileName
(
    QWidget *parent = 0,    //标准文件对话框的父窗口
    const QString &caption = QString(),    //标准文件对话框的标题名
    const QString &dir = QString(),        //指定打开时的默认目录路径
    const QString &fileter = QString(),    //参数选择对文件进行过滤显示，多种过滤器之间使用 “;;” 隔开
    QString *selectedFileter = 0,        //用户选择的过滤器通过此参数返回
    Options    options = 0,        //选择显示文件名的格式，默认是同时显示目录和文件名
);
~~~

在实例中需要实现点击 `fileBtn` 按钮后弹出标准文件对话框并完成文件名称的选择获取，将返回的结果显示到对应的文件名称显示控件 `fileLineEdit` 中。

### 标准颜色对话框类

在 Qt5 中的颜色类 `QColorDialog` 提供了 Qt 对颜色选择对话框的相关支持，通过该类的静态成员函数 `getColor()` 可以打开标准颜色对话框并进行颜色的选择，返回一个 `QColor` 类型的被选中的颜色值。

具体的函数参数说明如下：

```C++
QColor getColor
(
    const QColor &initial = Qt::white,        //指定默认选中的颜色为白色
    QWideget *parent = 0                    //指定标准颜色对话框的父窗口
);
```

在使用过程中，可以通过 `QColor::isValid()` 函数判断用户选择的颜色是否有效，如果用户在颜色选择界面点击 `Cancel` 则返回 `false` 。

### 标准字体对话框类

在 Qt5 中的颜色类 `QFontDialog` 提供了 Qt 对字体选择对话框的相关支持，通过该类的静态成员函数 `getFont()` 可以打开标准字体对话框并进行字体相关的参数设定，返回一个 `QFont` 类型的被选中的字体。

具体的函数参数说明如下：

```C++
QFont getFont
(
    bool *ok,        //若用户选择 OK， 则该参数将设为 true 并返回被设定的字体，否则设为 false ，此时返回默认字体。
    QWidget *parent = 0,    //标准字体对话框的父窗口
);
```

### 输入对话框类

，实现标准字符输入对话框是通过 `QInputDialog类` 的静态成员函数 `getText()` 完成，具体参数形式如下：

```C++
QString getText(
    QWidget *parent,        //标准输入对话框的父窗口
    const QString &title,    //标准输入对话框的标题名
    QLineEdit::EchoMode mode = QLineEdit::Normal,    //标准输入对话框中QLineEdit的输入模式
    const QString &text = QString(),    //标准字符对话框弹出时QLineEdit控件中默认显示文件
    bool *ok = 0,    //提示标准输入对话框的按钮触发，如果为 true ，则表示用户单击 OK ， 若为 false 则表示选择了 Cancel
    Qt::WindowFlags flags = 0    //指明标准输入对话框的窗体标识
);
```

根据该函数的参数，在 `inputdlg.cpp` 文件中添加对应槽函数 `ChangeName()` 实现如下：

```C++
void InputDlg::ChangeName()
{
    bool ok;
    QString text = QInputDialog::getText(this, tr("Input String Dialog"), tr("Input Name:"), QLineEdit::Normal, nameLabel_2->text(), &ok);
    if(ok && !text.isEmpty())
    {
        nameLabel_2->setText(text);
    }
}
```

### 消息对话框

#### Question 消息框

Question 消息框使用 `QMessageBox::question()` 函数实现，具体函数形式如下：

```C++
StandardButton QMessageBox::question(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showQuestionMsg()` ，具体代码如下：

```C++
void MsgBoxDlg::showQuestionMsg()
{
    //Question消息对话框
    label->setText(tr("Question Message Box"));
    switch (QMessageBox::question(this, tr("Question Message Dialog"), tr("End the Program ?"), QMessageBox::Ok | QMessageBox::Cancel, QMessageBox::Ok)) {
    case QMessageBox::Ok:
        label->setText(tr("Question button/OK"));
        break;
    case QMessageBox::Cancel:
        label->setText(tr("Question button/Cancel"));
        break;
    default:
        break;
    }
    return;
}
```

构建项目，运行程序，如下图：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/569ef11a87c5d16d7f8ce439b26eeed4-0)

单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/e0f49f8d9bd1f907f50cbc7918c0ce9d-0)

点击 `QuestionMsg` 按钮，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/39e82f2f391fe73149418cc72509a2e8-0)

#### Information 消息框

Information 消息框使用 `QMessageBox::information()` 函数完成，具体函数形式如下：

```C++
StandardButton QMessageBox::information(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showInformationMsg()` ，具体代码如下：

```C++
void MsgBoxDlg::showInformationMsg()
{
    label->setText(tr("Information Message Box"));
    QMessageBox::information(this, tr("Information Message Dialog"), tr("This is Information Dialog Test, Welcome !"));
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 消息对话框实例按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/0b426a4a67f638526909290135178047-0)

点击 `InformationMsg` 按钮，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/d52bcc61c064b71762b7b5b9b230bfdb-0)

#### Warning 消息框

Warning 消息框使用 `QMessageBox::warning()` 函数完成，具体函数形式如下：

```C++
StandardButton QMessageBox::warning(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showWarningMsg()` ，具体代码如下：

```C++
void MsgBoxDlg::showWarningMsg()
{
    label->setText(tr("Warning Message Box"));
    switch (QMessageBox::warning(this, tr("Warning Message Dialog"), tr("Data not saved , Whether to save ?"), QMessageBox::Save | QMessageBox::Discard | QMessageBox::Cancel, QMessageBox::Save)) {
    case QMessageBox::Save:
        label->setText(tr("Warning button/Save"));
        break;
    case QMessageBox::Discard:
        label->setText(tr("Warning button/Discard"));
        break;
    case QMessageBox::Cancel:
        label->setText(tr("Warning button/Cancel"));
        break;
    default:
        break;
    }
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/c94da2dd4e3769b8e4439b1f1573f633-0)

点击 `WarningMsg` 按钮，弹出如下界面：

![图片描述](https://doc.shiyanlou.com/courses/2595/600404/c64613da19f8758d292b4c212cf84bce-0)

#### Critical 消息框

Critical 消息框使用 `QMessageBox::critical()` 函数完成，具体函数形式如下：

```C++
StandardButton QMessageBox::critical(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showCriticalMsg()`，具体代码如下：

```C++
void MsgBoxDlg::showCriticalMsg()
{
    label->setText(tr("Critical Message Box"));
    QMessageBox::critical(this, tr("Critical Message Dialog"), tr("This is Critical Dialog Test !"));
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/4ea317220e71c156d692b8907adb73cc-0)

点击 `CriticalMsg` 按钮，弹出如下界面：

![图片描述](https://doc.shiyanlou.com/courses/2595/600404/5798c0ba758f754696162760d10ab326-0)

#### About 消息框

About 消息框使用 `QMessageBox::about()` 函数完成，具体函数形式如下：

```C++
StandardButton QMessageBox::about(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showAboutMsg()`，具体代码如下：

```C++
void MsgBoxDlg::showAboutMsg()
{
    label->setText(tr("About Message Box"));
    QMessageBox::about(this, tr("About Message Dialog"), tr("This is About Dialog Test !"));
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/9a56f6e5c54c0d0182a4325175b2524b-0)

点击 `AboutMsg` 按钮，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/194b1f8aeb9a98656ba42b7773383db3-0)

#### AboutQt 消息框

AboutQt 消息框使用 `QMessageBox::aboutQt()` 函数完成，具体函数形式如下：

```C++
StandardButton QMessageBox::aboutQt(
    QWidget *parent,    //消息框的父窗口
    const QString &title,    //消息框的标题名
    const QString &text,    //消息框的文字提示信息
    StandardButtons buttons = ok,    //显示按键定义
    StandardButton defaulltButton = NoButton,    //默认按钮，即弹出对话框时默认选择显示按钮。
);
```

完成文件 `msgboxdlg.cpp` 中槽函数 `showAboutQtMsg()`，具体代码如下：

```C++
void MsgBoxDlg::showAboutQtMsg()
{
    label->setText(tr("About Qt Message Box"));
    QMessageBox::aboutQt(this, tr("About Qt Message Dialog"));
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/9a56f6e5c54c0d0182a4325175b2524b-0)

点击 `AboutQtMsg` 按钮，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/a1c0a047deffd44833759b67586978a4-0)

#### Custom 自定义消息框

当前面标准消息框都满足不了需求时，Qt5 还允许自定义消息框，自定义消息框可以定义图标、按钮和内容等。

在 `msgboxdlg.cpp` 中进行对应控件的初始化和事件关联以及槽函数实现，具体如下：

```C++
MsgBoxDlg::MsgBoxDlg(QWidget *parent)
{
    ...
    ...
    CustomBtn = new QPushButton;
    CustomBtn->setText(tr("Custom"));
    customLabel = new QLabel;
    customLabel->setFrameStyle(QFrame::Panel | QFrame::Sunken);
    ...
    mainLayout->addWidget(CustomBtn, 4, 0);
    mainLayout->addWidget(customLabel, 4, 1);
    ...
    connect(CustomBtn, SIGNAL(clicked()), this, SLOT(showCustomDlg()));
}

void MsgBoxDlg::showCustomDlg()
{
    customLabel->setText(tr("Custom Message Box"));
    QMessageBox customMsgBox;
    customMsgBox.setWindowTitle(tr("User Custom Message Box"));
    QPushButton *yesBtn = customMsgBox.addButton(tr("Yes"), QMessageBox::ActionRole);
    QPushButton *noBtn = customMsgBox.addButton(tr("No"), QMessageBox::ActionRole);
    QPushButton *cancelBtn = customMsgBox.addButton(QMessageBox::Cancel);
    customMsgBox.setText(tr("This is user custom message box!"));
    customMsgBox.setIconPixmap(QPixmap("Qt.png"));
    customMsgBox.exec();

    if(customMsgBox.clickedButton() == yesBtn)
    {
        customLabel->setText(tr("Custom Message Box/Yes!"));
    }
    if(customMsgBox.clickedButton() == noBtn)
    {
        customLabel->setText(tr("Custom Message Box/No!"));
    }
    if(customMsgBox.clickedButton() == cancelBtn)
    {
        customLabel->setText(tr("Custom Message Box/Cancel!"));
    }
    return;
}
```

构建项目，运行程序，单击 `Message Box demo` 按钮控件，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/97de4b0f14212332436d085d43de5a29-0)

点击 `Custom` 按钮，弹出如下界面：

![avatar](https://doc.shiyanlou.com/courses/2595/1398198/bc85b9b0166474e1e571a179254bcf17-0)

## 创建行为

```
QAction *exitAction; 
exitAction = new QAction(QIcon("exit.png"), tr("Exit"), this);
exitAction->setShortcut(tr("Ctrl+Q")); // 设置快捷键，等价于setShortcut （QKeySequence(Qt::CTRL + Qt::Key_Q))）
exitAction->setStatusTip(tr("exit this function."));
connect(exitAction, SIGNAL(triggered()), this, SLOT(close()));
```

+ 工具栏对应动作添加图标

在工具栏添加图标

~~~
openFileAction = new QAction(QIcon("open.png"), tr("Open"), this);  // 创建带图片的动作
~~~

创建工具条

~~~
QToolBar *fileTool;        //文件操作工具栏控件
//文件工具条
fileTool = addToolBar("File");
fileTool->addAction(openFileAction);
fileTool->addAction(NewFileAction);
fileTool->addAction(PrintTextAction);
fileTool->addAction(PrintImageAction);
~~~

图片放在编译路径下。

## 创建菜单栏

~~~
QMenu *fileMenu;    //文件菜单栏控件
//文件菜单
fileMenu = menuBar()->addMenu(tr("File"));
fileMenu->addAction(openFileAction);
fileMenu->addAction(NewFileAction);
fileMenu->addAction(PrintTextAction);
fileMenu->addAction(PrintImageAction);
fileMenu->addSeparator();			// 分割线
fileMenu->addAction(exitAction);    //文件菜单
~~~



## 创建工具条

~~~
QToolBar *fileTool;        //文件操作工具栏控件
fileTool = addToolBar("File");
fileTool->addAction(openFileAction);
fileTool->addAction(NewFileAction);
fileTool->addAction(PrintTextAction);
fileTool->addAction(PrintImageAction);
~~~

