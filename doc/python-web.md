# flask

flask 是一个轻量级的 Web 应用框架, 使用 Python 编写。基于 [Werkzeug WSGI](http://werkzeug.pocoo.org/) 工具箱和 [Jinja2](http://jinja.pocoo.org/) 模板引擎。使用 BSD 授权。Flask 也被称为 microframework ，因为它使用简单的核心，用 extension 增加其它功能。Flask 没有默认使用的数据库、窗体验证工具。然而，Flask 保留了扩增的弹性，可以用 Flask-extension 加入这些功能：ORM、窗体验证工具、文件上传、各种开放式身份验证技术。

当安装 Flask 时，以下组件也会自动安装：

- [Werkzeug](http://werkzeug.pocoo.org/): WSGI（web 服务器网关接口）工具，是介于应用和服务器之间标准的接口工具。
- [Jinja](http://jinja.pocoo.org/): web 前端页面中使用的模板语言。
- [MarkupSafe](https://pypi.org/project/MarkupSafe/): 与 Jinja 配合使用，当表单页面跳转时会进行验证从而避免遭遇不信任的输入带来的攻击。
- [ItsDangerous](https://pythonhosted.org/itsdangerous/): 安全地注入数据以确保数据的完整性，通常用于保护 Flask 的 session cookie。
- [Click](https://click.palletsprojects.com/en/7.x/): 一个解析命令行的应用，它支持在 Flask 中自定义管理命令。



## 蓝图

[认识flask 蓝图（blueprint）技术 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/357444025)





## 基础使用

连接ip: 10.101.5.14

代码运行

~~~
export FLASK_APP=hello.py
flask run -h 0.0.0.0 -p 80
~~~

网页登录

~~~
http://10.101.5.14:8081
~~~

开发设置

~~~
export FLASK_ENV=development
export FLASK_DEBUG=1
flask run -h 0.0.0.0 -p 8080
~~~

## 创建应用

一个最小的应用看起来像这样，在 `/home/project` 目录下新建 `hello.py` 文件，可以在命令行中使用`touch hello.py`新建，或者在侧边栏中点击右键 `New File` 创建文件并向其中写入如下代码：

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

那么这段代码做了什么？

1. 首先我们导入了类 Flask。这个类的实例化将会是我们的 WSGI 应用。
2. 接着，我们创建一个该类的实例。第一个参数是应用模块或包的名称，这样 Flask 才会知道去哪里寻找模板、静态文件等等。如果你使用的是单一的模块（就如本例），第一个参数应该使用 `__name__`。
3. 我们使用装饰器 `route()` 告诉 Flask 哪个 `URL` 才能触发我们的函数。
4. 定义一个函数，该函数名也是用来给特定函数生成 URLs，并且返回我们想要显示在用户浏览器上的信息。

使用 Python 解释器运行这个文件，注意这个文件不能取名为 `flask.py` ，因为这会与 Flask 本身冲突。

运行这个应用既可以使用 flask 命令也可以使用 Python 的 `-m` 调用 flask，在运行之前你需要设置 `FLASK_APP` 的环境变量来告诉终端需要运行哪个应用。

在终端执行如下命令：

```bash
export FLASK_APP=hello.py
flask run
```

也可以使用如下命令启动应用：

```bash
export FLASK_APP=hello.py
python3 -m flask run
```

以上的命令启动了一个非常简单的 flask 内置服务器，用于测试已经足够了但可能你并不想用于生产环境。更多配置可以参考[开发者选项](http://flask.pocoo.org/docs/1.0/deploying/#deployment)。

实验楼在 WebIDE 环境中提供了一个 “Web 服务” 功能，可以直接进入到实验环境中 8080 端口运行的网站。所以在启动应用的时候需要提供端口号为 8080 的选项 `-p 8080` ，此外还需要将 IP 地址设置为 0.0.0.0 以支持实验环境的随机 IP 地址被外部环境访问。

在实验环境中执行如下命令：

```bash
export FLASK_APP=hello.py
flask run -h 0.0.0.0 -p 8080
```

使用 flask 命令行可以非常方便的启动一个本地开发服务器，但是每次修改代码后你都需要手动重启服务器。通过前面的启动后输出显示可以发现 `Environment` 为 `production`，同时调试模式未开启 `Debug mode: off`。

这样做并不好，Flask 能做得更好。如果启用了调试支持，在代码修改后服务器能够自动重载，并且如果发生错误，它会提供一个有用的调试器。

为了让所有的开发者特征可用（包括调试模式），在运行服务器之前可以设置 `FLASK_ENV` 环境变量为 `development`：

```bash
export FLASK_ENV=development
export FLASK_DEBUG=1
flask run -h 0.0.0.0 -p 8080
```

上述命令做了以下几件事：

1. 使调试器（debugger）可用
2. 启动了代码改变自动的热加载
3. 在 flask 应用中开启了 debug 模式

### URL变量规则

为了给 URL 增加变量的部分，你需要把一些特定的字段标记成 `<variable_name>` 。这些特定的字段将作为参数传入到你的函数中。当然也可以指定一个可选的转换器通过规则 `<converter:variable_name>` 将变量值转换为特定的数据类型。 在 `/home/project/hello.py` 文件中添加如下的代码：

```python
@app.route('/user/<username>')
def show_user_profile(username):
    # 显示用户名
    return 'User {}'.format(username)

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # 显示提交整型的用户"id"的结果，注意"int"是将输入的字符串形式转换为整型数据
    return 'Post {}'.format(post_id)

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # 显示 /path/ 之后的路径名
    return 'Subpath {}'.format(subpath)
```

按照前面的方式启动应用，逐个访问地址：

当访问 `https://xxx/user/shiyanlou` 时，页面显示为 `User shiyanlou`。

当访问 `https://xxx/post/3` 时，页面显示为 `Post 3`。用户在浏览器地址栏上输入的都是字符串，但是在传递给 `show_post` 函数处理时已经被转换为了整型。

当访问 `https://xxx/path/file/A/a.txt` 时，页面显示为 `Subpath file/A/a.txt`。

**注意：上面几个链接中的`https://xxx`是指你点击右侧工具栏“Web 服务”自动生成的链接**

转换器的主要类型如下：

| 类型   | 含义                                        |
| ------ | ------------------------------------------- |
| string | 默认的数据类型，接受没有任何斜杠“/”的字符串 |
| int    | 接受整型                                    |
| float  | 接受浮点类型                                |
| path   | 和 string 类似，但是接受斜杠“/”             |
| uuid   | 只接受 uuid 字符串                          |

### 唯一URLs/重定向行为

Flask 的 URL 规则是基于 Werkzeug 的 routing 模块。该模块背后的思路是基于 Apache 和早期的 HTTP 服务器定下先例确保优雅和唯一的 URL。

以这两个规则为例，在 `/home/project/hello.py` 文件中添加如下的代码：

```python
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```

虽然它们看起来确实相似，但它们结尾斜线的使用在 URL 定义中不同。

第一种情况中，规范的 URL 指向 projects 尾端有一个斜线 `/` 。这种感觉很像在文件系统中的文件夹。访问一个结尾不带斜线的 URL 会被 Flask 重定向到带斜线的规范 URL 去。当访问 `https://xxx/projects/` 时，页面会显示 `The project page` 。

然而，第二种情况的 URL 结尾不带斜线，类似 UNIX-like 系统下的文件的路径名。此时如果访问结尾带斜线的 URL 会产生一个 `404 “Not Found”` 错误。当访问 `https://xxx/about` 时，页面会显示 `The about page` ；但是当访问 `https://xxx/about/` 时，页面就会报错 `Not Found` 。

当用户访问页面忘记结尾斜线时，这个行为允许关联的 URL 继续工作，并且与 Apache 和其它的服务器的行为一致，反之则不行，因此在代码的 URL 设置时斜线只可多写不可少写；另外，URL 会保持唯一，有助于避免搜索引擎索引同一个页面两次。



### 构建URL

去构建一个 URL 来匹配一个特定的函数可以使用 `url_for()` 方法。它接受函数名作为第一个参数，以及一些关键字参数，每一个关键字参数对应于 URL 规则的变量部分。未知变量部分被插入到 URL 中作为查询参数。

为什么你要构建 URLs 而不是在模版中[硬编码](http://baike.baidu.com/link?url=n_z_TKp-zXEZf8H-D9-5YMLbNnhbezKXe-FsiWMwMjLMz2bl3bAJBL1s40yHihwnc0T1EXTYMmoZUVvrgf6yV_)呢？这里有几个理由：

1. 反向构建通常比硬编码更具备描述性。
2. 它允许你一次性修改 URL，而不是到处找 URL 修改。
3. 构建 URL 能够显式地处理特殊字符和 `Unicode` 转义，因此你不必去处理这些。
4. 如果你的应用不在 URL 根目录下(比如，在 `/myapplication` 而不在 `/`)，`url_for()`将会适当地替你处理好。

在 Python shell 交互式命令行下运行如下代码：

![图片描述](https://doc.shiyanlou.com/courses/3817/810810/b2f7c84a713accce1618e822c9e8ff3e-0)

```python
>>> from flask import Flask, url_for
>>> app = Flask(__name__)
>>> @app.route('/')
... def index():
...     return 'index'
...
>>> @app.route('/login')
... def login():
...     return 'login'
...
>>> @app.route('/user/<username>')
... def profile(username):
...     return '{}\'s profile'.format(username)
...
>>> with app.test_request_context():
...     print(url_for('index'))
...     print(url_for('login'))
...     print(url_for('login', next='/'))
...     print(url_for('profile', username='John Doe'))
...
/
/login
/login?next=%2F   # 对字符串进行了转义，未知变量部分被当做查询参数
/user/John%20Doe
```

`test_request_context()` 方法告诉 Flask 表现得像是在处理一个请求，即使我们正在通过 Python shell 交互。大家可以仔细分析一下该函数的打印结果。

### HTTP方法

HTTP (也就是 Web 应用协议) 有不同的方法来访问 URLs 。默认情况下，路由只会响应 GET 请求，但是能够通过给 `route()` 装饰器提供 `methods` 参数来改变。这里是一个例子:

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()   # 如果是 POST 方法就执行登录操作
    else:
        show_the_login_form()   # 如果是 GET 方法就展示登录表单
```

如果使用 `GET` 方法，`HEAD` 方法将会自动添加进来。你不必处理它们。也能确保 `HEAD` 请求会按照 HTTP RFC (文档在 HTTP 协议里面描述) 要求来处理，因此你完全可以忽略这部分 HTTP 规范。同样地，自从 Flask 0.6 后，`OPTIONS` 方法也能自动为你处理。

也许你并不清楚 HTTP 方法是什么？别担心，这里有一个 HTTP 方法的快速入门以及为什么它们重要：

`HTTP`方法（通常也称为“谓词”）告诉服务器客户端想要对请求的页面做什么。

下面这些方法是比较常见的：

- GET：浏览器通知服务器只获取页面上的信息并且发送回来。这可能是最常用的方法。
- HEAD：浏览器告诉服务器获取信息，但是只对头信息感兴趣，不需要整个页面的内容。应用应该处理起来像接收到一个 GET 请求但是不传递实际内容。在 Flask 中你完全不需要处理它，底层的 Werkzeug 库会为你处理的。
- POST：浏览器通知服务器它要在 URL 上提交一些信息，服务器必须保证数据被存储且只存储一次。这是 HTML 表单通常发送数据到服务器的方法。
- PUT：同 POST 类似，但是服务器可能触发了多次存储过程，多次覆盖掉旧值。现在你就会问这有什么用，有许多理由需要如此去做。考虑下在传输过程中连接丢失：在这种情况下浏览器和服务器之间的系统可能安全地第二次接收请求，而不破坏其它东西。该过程操作 POST 方法是不可能实现的，因为它只会被触发一次。
- DELETE：移除给定位置的信息。
- OPTIONS：给客户端提供一个快速的途径来指出这个 URL 支持哪些 HTTP 方法。从 Flask 0.6 开始，自动实现了该功能。

现在在 HTML4 和 XHTML1 中，表单只能以 GET 和 POST 方法来提交到服务器。在 JavaScript 和以后的 HTML 标准中也能使用其它的方法。同时，HTTP 最近变得十分流行，浏览器不再是唯一使用 HTTP 的客户端。比如许多版本控制系统使用 HTTP。



## 静态文件

动态的 web 应用同样需要静态文件。`CSS` 和 `JavaScript` 文件通常来源于此。理想情况下，你的 web 服务器已经配置好为它们服务，然而在开发过程中 Flask 就能够做到。只要在你的包中或模块旁边创建一个名为 `static` 的文件夹，在应用中使用 `/static` 即可访问。

给静态文件生成 URL ，使用特殊的 `static` 端点名：

```python
url_for('static', filename='style.css')
```

这个文件是应该存储在文件系统上的 `static/style.css` 。



## 请求对象

### 局部上下文

注意：如果你想要了解上下文作用域是如何工作的以及如何使用它进行测试，就可以读这一部分，如果暂时不需要的话，可以直接跳过这部分。

Flask 中的某些对象是全局对象，但不是通常的类型。这些对象实际上是给定上下文的局部对象的代理。虽然很拗口，但实际上很容易理解。

想象下线程处理的上下文。一个请求传入，web 服务器决定产生一个新线程(或者其它东西，底层对象比线程更有能力处理并发系统)。当 Flask 开始它内部请求处理时，它认定当前线程是活动的上下文并绑定当前的应用和 WSGI 环境到那个上下文（线程）。它以一种智能的方法来实现，以致一个应用可以调用另一个应用而不会中断。

所以这对你意味着什么呢？除非你是在做一些类似单元测试的事情，否则基本上你可以完全忽略这种情况。你会发现依赖于请求对象的代码会突然中断，因为没有请求对象。解决方案就是自己创建一个请求并把它跟上下文绑定。

针对单元测试最早的解决方案是使用 `test_request_context()` 上下文管理器。结合 `with` 声明，它将绑定一个测试请求来进行交互。这里是一个例子：

```python
from flask import request

with app.test_request_context('/hello', method='POST'):
    # 现在你可以做出请求，比如基本的断言
    assert request.path == '/hello'
    assert request.method == 'POST'
```

另一个可能性就是传入整个 WSGI 环境到`request_context()`方法：

```python
from flask import request

with app.request_context(environ):
    assert request.method == 'POST'
```

### request

[python-flask之request的属性 - 百度文库 (baidu.com)](https://wenku.baidu.com/view/177ef4ae87868762caaedd3383c4bb4cf7ecb77f.html)

首先你需要从 flask 模块中导入 `request` ：

```python
from flask import request
```

当前请求的方法可以用`method`属性来访问。你可以用`form`属性来访问表单数据 (数据在 `POST` 或者`PUT`中传输)。这里是上面提及到的两种属性的完整的例子：

~~~
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # 当请求形式为“GET”或者认证失败则执行以下代码
    return render_template('login.html', error=error)
~~~

如果在`form`属性中不存在上述键值会发生些什么？在这种情况下会触发一个特别的 `KeyError`。你可以像捕获标准的`KeyError`一样来捕获它，如果你不这样去做，会显示一个`HTTP 400 Bad Request`错误页面。所以很多情况下你不需要处理这个问题。

你可以用`args`属性来接收在`URL ( ?key=value )`中提交的参数：

```python
searchword = request.args.get('key', '')
```

我们推荐使用`get`来访问 URL 参数或捕获`KeyError`，因为用户可能会修改 URL，向他们显示一个`400 bad request`页面不是用户友好的。

### 文件上传

你能够很容易地用 Flask 处理文件上传。只要确保在你的 HTML 表单中不要忘记设置属性 `enctype="multipart/form-data"` ，否则浏览器将不会传送文件。

上传的文件是存储在内存或者文件系统上一个临时位置。你可以通过请求对象中`files`属性访问这些文件。每个上传的文件都会存储在这个属性字典里。它表现得像一个标准的 Python `file`对象，但是它同样具有`save()`方法，该方法允许你存储文件在服务器的文件系统上。

下面是一个简单的例子用来演示提交文件到服务器上：

~~~
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...
~~~

如果你想要知道在上传到你的应用之前在客户端的文件名称，你可以访问`filename`属性。但请记住永远不要信任这个值，因为这个值可以伪造。如果你想要使用客户端的文件名来在服务器上存储文件，把它传递到 `Werkzeug` 提供给你的 `secure_filename()` 函数：

```python
from flask import request
from werkzeug import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
    ...
```

### cookies

你可以用 `cookies` 属性来访问 [Cookies](http://baike.baidu.com/link?url=qqNODHYRG7RhXXhQtFZUJpYUE8kX2vd3kkG-s641Obx9cwa8nHspmzUYpZJRfHXo5YKKY6jsEH9Yxa9da8BcP_) 。你能够用响应对象的 `set_cookie` 来设置 `cookies`。请求对象中的 `cookies` 属性是一个客户端发送所有的 `cookies` 的字典。

如果你要使用会话(`sessions`)，请不要直接使用 `cookies`，相反，请用 Flask 中的会话，Flask 已经在`cookies` 上增加了一些安全细节；关于更多 `seesions` 和 `cookies` 的区别与联系，请参见[施杨出品的博客](http://www.cnblogs.com/shiyangxt/archive/2008/10/07/1305506.html)。

读取 cookies ：

```python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # 注意这里引用cookies字典的键值对是使用cookies.get(key)
    # 而不是cookies[key]，这是防止该字典不存在时报错"keyerror"
```

存储 cookies ：

```python
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```

注意`cookies`是在响应对象中被设置。由于通常只是从视图函数返回字符串，Flask 会将其转换为响应对象。如果你要显式地这么做，可以使用 `make_response()` 函数接着修改它。

有时候你可能要在响应对象不存在的地方设置`cookie`。利用[延迟请求回调模式](http://flask.pocoo.org/docs/1.0/patterns/deferredcallbacks/#deferred-callbacks)使得这种情况成为可能。

+ 练习

请实现一个上传图片到服务器的功能。具体要求如下：

- 在 `/home/project` 目录下新建 `upload_file.py` 文件并在其中写入本练习的代码。
- 要求上传的文件保存位置为 `/home/project `目录下。
- 当访问首页 `https://xxx` 时出现表单可以上传文件，当上传成功后在浏览器可以看到 `<上传的文件名> 上传成功!`

~~~
import os
from flask import Flask, request
from werkzeug.utils import secure_filename   # 获取上传文件的文件名

UPLOAD_FOLDER = '/home/project'   # 上传路径
ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])   # 允许上传的文件类型

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

def allowed_file(filename):   # 验证上传的文件名是否符合要求，文件名必须带点并且符合允许上传的文件类型要求，两者都满足则返回 true
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':   # 如果是 POST 请求方式
        file = request.files['file']   # 获取上传的文件
        if file and allowed_file(file.filename):   # 如果文件存在并且符合要求则为 true
            filename = secure_filename(file.filename)   # 获取上传文件的文件名
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))   # 保存文件
            return '{} 上传成功!'.format(filename)   # 返回保存成功的信息
    # 使用 GET 方式请求页面时或是上传文件失败时返回上传文件的表单页面
    return '''
    <!doctype html>
    <title>上传文件</title>
    <h1>上传文件</h1>
    <form action="" method=post enctype=multipart/form-data>
      <p><input type=file name=file>
         <input type=submit value=上传></p>
    </form>
    '''

if __name__ == '__main__':
    app.run('0.0.0.0', 8081)
~~~

## 重定向

你能够用 `redirect()` 函数重定向用户到其它地方。能够用 `abort()` 函数提前中断一个请求并带有一个错误代码。

~~~
from flask import Flask
from flask import abort, redirect, url_for

app = Flask(__name__)

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
~~~

按照之前的方式运行应用，这是一个相当无意义的例子因为用户会从主页`/`重定向到一个不能访问的页面`/login`（ 401 意味着禁止访问），但是它说明了重定向如何工作。

默认情况下，每个错误代码会显示一个黑白错误页面。比如上面的页面会显示 `401 Unauthorized`。如果你想定制错误页面，可以使用`errorhandler()`装饰器，向 `/home/project/hello.py` 文件中添加如下代码:

~~~
from flask import render_template

@app.errorhandler(401)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
~~~

## 响应（404）

一个视图函数的返回值会被自动转换为一个响应对象。如果返回值是一个字符串，它被转换成一个响应主体是该字符串，错误代码为 `200 OK` ，媒体类型为`text/html`的响应对象。Flask 把返回值转换成响应对象的逻辑如下：

1. 如果返回的是一个合法的响应对象，它会直接从视图返回。
2. 如果返回的是一个字符串，响应对象会用字符串数据和默认参数创建。
3. 如果返回的是一个元组而且元组中元素能够提供额外的信息。这样的元组必须是(`response, status, headers`) 形式且至少含有其中的一个元素。`status`值将会覆盖状态代码，`headers`可以是一个列表或额外的消息头值字典。
4. 如果上述条件均不满足，Flask 会假设返回值是一个合法的 WSGI 应用程序，并转换为一个请求对象。

如果你想要获取在视图中得到的响应对象，你可以用函数`make_response()`。

想象你有这样一个视图:

~~~
@app.errorhandler(404)
def not_found(error):
    return render_template('error.html'), 404
~~~

你只需要用`make_response()`封装返回表达式，获取结果对象并修改，然后返回它：

~~~
@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp
~~~

## 会话

除了请求对象，还有第二个称为`session`对象允许你在不同请求间存储特定用户的信息。这是在 cookies 的基础上实现的，并且在 cookies 中使用加密的签名。这意味着用户可以查看 cookie 的内容，但是不能修改它，除非知道签名的密钥。

要使用会话，你需要设置一个密钥。这里介绍会话如何工作，在 `/home/project` 目录下新建 `test.py` 文件并写入如下代码：

~~~
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

# 设置密钥，保证会话安全
app.secret_key = '_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # 如果用户名存在，则从会话中移除该用户名
    session.pop('username', None)
    return redirect(url_for('index')
~~~

这里提到的`escape()`可以在你不使用模板引擎的时候做转义（如同本例）。其中，`login`函数中返回的网页源代码可以单独存储在`templates`文件夹中作为模板文件`html`，然后使用 return `render_template()`更方便。

当访问首页 `https://xxx/` 时会显示 `You are not logged in`；

当访问登录页面 `https://xxx/login` 时会出现一个输入框，在输入框中输入用户名 shiyanlou，然后点击 `Login` 按钮，这时 URL 会重定向到首页上，首页显示 `Logged in as shiyanlou`；

最后再访问登出页面 `https://xxx/logout`，这时从 session 中移除了用户名，URL 重定向到首页显示 `You are not logged in`；

**怎样产生一个好的密钥：**

随机的问题在于很难判断什么是真随机。一个密钥应该足够随机。你的操作系统可以基于一个密码随机生成器来生成漂亮的随机值，这个值可以用来做密钥:

~~~
$ python3 -c 'import os; print(os.urandom(16))'
b'm \xf8>]?\x86\xcf/y\x0e\xc5\xc7j\xc5/'
~~~

把这个值复制粘贴到你的代码，你就搞定了密钥。

使用基于 cookie 的会话需注意: Flask 会将你放进会话（session）对象的值序列化到 cookie 。如果你试图寻找一个跨请求不能存留的值，cookies 确实是启用的，并且你不会获得明确的错误信息，检查你页面请求中 cookie 的大小，并与 web 浏览器所支持的大小对比。

## 消息闪烁

好的应用和用户界面全部是关于反馈。如果用户得不到足够的反馈，他们可能会变得讨厌这个应用。Flask 提供了一个真正的简单的方式来通过消息闪现系统给用户反馈。消息闪现系统基本上使得在请求结束时记录信息并在下一个 （且仅在下一个）请求中访问。通常结合模板布局来显示消息。

使用`flash()`方法来闪现一个消息，使用`get_flashed_messages()`能够获取消息，`get_flashed_messages()`也能用于模板中。

下面来看一个简单的例子，在 `/home/project` 目录下新建 `flashTest.py` 文件，并向其中写入如下代码：

```python
from flask import Flask, flash, redirect, render_template, \
     request, url_for

app = Flask(__name__)
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != 'admin' or \
                request.form['password'] != 'secret':
            error = 'Invalid credentials'
        else:
            flash('You were successfully logged in')
            return redirect(url_for('index'))
    return render_template('login.html', error=error)
```

然后在 `/home/project/templates` 目录下新建 `base.html` 页面，其中写入基本的模板代码，代码主要是从后端获取 flash 消息以及错误信息。

```html
<!DOCTYPE html>
<title>My Application</title>
{% with messages = get_flashed_messages() %} {% if messages %}
<ul class="flashes">
  {% for message in messages %}
  <li>{{ message }}</li>
  {% endfor %}
</ul>
{% endif %} {% endwith %} {% block body %}{% endblock %}
```

在该目录下新建 `index.html` 页面，这个页面继承于 `base.html` 页面：

```html
{% extends "base.html" %} {% block body %}
<h1>Overview</h1>
<p>
  Do you want to <a href="{{ url_for('login') }}">log in?</a> {% endblock %}
</p>
```

在该目录下新建 `login.html` 页面，这个页面也继承于 `base.html` 页面：

```html
{% extends "base.html" %} {% block body %}
<h1>Login</h1>
{% if error %}
<p class="error"><strong>Error:</strong> {{ error }} {% endif %}</p>

<form method="post">
  <dl>
    <dt>Username:</dt>
    <dd>
      <input
        type="text"
        name="username"
        value="{{
          request.form.username }}"
      />
    </dd>

    <dt>Password:</dt>
    <dd><input type="password" name="password" /></dd>
  </dl>

  <p><input type="submit" value="Login" /></p>
</form>

{% endblock %}
```

按照前面的方式运行程序：

```bash
cd /home/project
source venv/bin/activate
export FLASK_APP=flashTest.py
export FLASK_ENV=development
export FLASK_DEBUG=1
flask run -h 0.0.0.0 -p 8080
```

当访问首页 `https://xxx`，会提示 `Do you want to log in?`，点击链接跳转到登录页面。

在登录页面 `https://xxx/login`，输入用户名和密码，如果输入错误的信息比如两个都为 shiyanlou，点击 Login，就会出现错误提示 `Error: Invalid credentials`。如果用户名输入 admin、密码输入 secret，点击 Login，就会跳转到首页，同时在首页会显示 flash 消息 `You were successfully logged in`。

![图片描述](https://doc.shiyanlou.com/courses/3817/810810/4d9e836ff6b6e8c3804d94b24b25a8c4-0)

![图片描述](https://doc.shiyanlou.com/courses/3817/810810/4d9e836ff6b6e8c3804d94b24b25a8c4-0)

## 日志和整合WSGL中间件

### 日志

有时候你会遇到一种情况：理论上来说你处理的数据应该是正确的，然而实际上并不正确。比如你可能有一些客户端代码，代码向服务器发送一个 HTTP 请求，但是显然它是错误的。这可能是由于用户篡改数据，或客户端代码失败。大部分时候针对这一情况返回`400 Bad Request`就可以了，但是有时候不能这样做，代码必须继续工作。

你也有可能想要记录一些发生的不正常事情。这时候日志就派上用处。从 Flask 0.3 开始日志记录是预先配置好的。

这里有一些日志调用的例子:

```python
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```

附带的 logger 是一个标准的日志类 Logger ，因此更多的信息请查阅官方文档 [logging documentation](http://docs.python.org/library/logging.html)。

### 整合 WSGI 中间件

如果你想给你的应用添加 WSGI 中间件，你可以封装内部 WSGI 应用。例如如果你想使用 Werkzeug 包中的某个中间件来应付 lighttpd 中的 bugs，你可以这样做:

```python
from werkzeug.contrib.fixers import LighttpdCGIRootFix
app.wsgi_app = LighttpdCGIRootFix(app.wsgi_app)
```





## 返回值

[flask return返回值的类型要求_gxz987的博客-CSDN博客_flask return](https://blog.csdn.net/gxz987/article/details/90611601)

+ 元组的内容包括三个参数

`response`（响应体）status_code（状态码）headers（响应头）





## 博客项目

~~~
# 创建应用
app = Flask(__name__)
# app.config 的 from_object 方法通常用于获取对象的属性以增加配置项
# 此处使用的参数为 __name__ ，也就是当前所在文件
# 结果就是读取当前所在文件中的所有变量，选择其中全大写的变量作为配置项
app.config.from_object(__name__)
~~~

`from_object()` 将会寻找给定的对象(如果它是一个字符串，则会导入它)，搜寻里面定义的全部大写的变量。在我们的这种情况中，配置文件就是我们上面写的几行代码。 你也可以将他们分别存储到多个文件。

通常从配置文件中加载配置是一个好的主意。这时可以使用`from_envvar()`来实现，可以用它替换上面的`from_object()`：

~~~
# 这一行代码无需写入文件
app.config.from_envvar('FLASKR_SETTINGS', silent=True)
~~~

这种方法我们可以设置一个名为`FLASKR_SETTINGS`的环境变量来设定一个配置文件载入后是否覆盖默认值。静默开关`silent=True`告诉 Flask 不去关心这个环境变量键值是否存在。

`SECRET_KEY` 是为了保持客户端的会话安全。明智地选择该键，使得它难以猜测，最好是尽可能复杂。

DEBUG 调试标志启用或禁用交互式调试。决不让调试模式在生产系统中启动，因为它将允许用户在服务器上执行代码！

我们还添加了一个轻松地连接到指定数据库的方法，这个方法用于在请求时打开一个连接，并且在交互式 Python shell 和脚本中也能使用，这将方便后面的使用。









## token机制

[Python 生成 JWT(json web token) 及 解析方式 - lowmanisbusy - 博客园 (cnblogs.com)](https://www.cnblogs.com/lowmanisbusy/p/10930856.html)

[flask 实现token机制 - 简书 (jianshu.com)](https://www.jianshu.com/p/7bff6e3d869b)





# 自动化特征工程

![image-20220602152957022](E:\Document\Typora\img\image-20220602152957022.png)

![image-20220602153013156](E:\Document\Typora\img\image-20220602153013156.png)

![image-20220602153232003](E:\Document\Typora\img\image-20220602153232003.png)

![image-20220602153531088](E:\Document\Typora\img\image-20220602153531088.png)

![image-20220602154451899](E:\Document\Typora\img\image-20220602154451899.png)

![image-20220602154613523](E:\Document\Typora\img\image-20220602154613523.png)



# Django

