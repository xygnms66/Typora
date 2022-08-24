# Linux精进

## 二进制文件分析

+ 二进制文件类型

1. file程序查看,如:file XXX；
2. readelf分析ELF头信息,如:readelf -h XXX ；其中类型：EXEC DYN REL 分别为可执行文件、共享目标文件、可重定位文件(如.a/.o文件)；
3. objdump分析ELF头信息，同readelf类似，其中类型：EXEC_P DYNAMIC 分别为可执行文件、共享目标文件，对于目标文件没有显示类型信息。

+ 查看导出符号列表信息

1. nm
2. readelf
3. objdump

+ 查看可执行文件或者共享库加载时依赖的共享列表

1. ldd
2. objdump -p / readelf -d





## NEW tips

### diff

Linux/Unix 一直都有一个基础的 shell 命令：`diff`，它能够比较 2 个文件的差异，并用多种记录形式输出差异

~~~
diff log1 log2
~~~

`.patch`保存文件差异

生成 patch 文件的人只需分享此文件，收到者在原始文件上应用 patch，就能将修订更新到自己的文件中，这样就不必传递整个文件。

历史上发展出了 3 种 patch 格式，幸好 diff 命令都给予了兼容，分别是：

1. 正常格式（normal diff）—— 默认
2. 上下文格式（context diff）—— `-c`
3. 合并格式（unified diff）—— `-u`



命令行生成的 diff 一般是给机器看的，人工肉眼观察通常还需要再套上一层 UI，从编辑器之神 vi/vim：`vim -d log1.txt log2.txt`。



编辑器后起之秀 VSCode：`code -d log1.txt log2.txt`



`git difftool` 是在 `git diff` 的基础之上，不改变参与对比的文件，而使用用户配置的 tool 来展示对比结果。请实验如下配置：



肉眼看 diff 毕竟是个异常困难的工作，配置 UI 工具才是我们更常使用的方式。`git difftool` 是在 `git diff` 的基础之上，不改变参与对比的文件，而使用用户配置的 tool 来展示对比结果。请实验如下配置：

```bash
git config --global difftool.meld.cmd 'meld --diff "$LOCAL" "$REMOTE"'
git config --global difftool.bc.cmd 'bcompare "$LOCAL" "$REMOTE"'
git config --global difftool.code.cmd 'code --wait --diff "$LOCAL" "$REMOTE"'
```

上述命令配置了 3 个 UI 工具给 difftool，分别调用 Linux 版本的 Meld、BeyondCompare、VSCode（其中 `bc`,`meld`,`code` 是自己自由定义的名称），所以需要你事先安装这些软件，否则将无法找到和启动相应的软件。Ubuntu 上可以非常方便的安装 Meld：

```bash
sudo apt install meld
```

然后，还有一个可选但推荐的配置：是否在比较时命令行进行提示，false 不提示，否则打开 difftool 时需要进行一次确认。

```bash
git config --global difftool.prompt false
```

最后我们就可以根据需求进行选择，下面依次演示使用 `meld`，`bc`，`code` 3 种软件进行对比的效果，请先对你的 diary 文件做一些修改，然后选择已经正确安装的软件进行实验（原理相同，选择其一即可）。

```bash
git config --global diff.tool meld
git difftool
```

![Snipaste_2021-08-13_11-31-59](E:\Document\Typora\img\Snipaste_2021-08-13_11-31-59.png)



~~~
git config --global diff.tool bc
git difftool
~~~

![Snipaste_2021-08-13_11-32-31](E:\Document\Typora\img\Snipaste_2021-08-13_11-32-31.png)









## 常忘命令

+ **tar -zxvf 压缩文件名.tar.gz**

+ **设置LD_LIBRARY_PATH**

```
在 ~/.bashrc 或者 ~/.bash_profile 中加入 export 语句，前者在每次登陆和每次打开 shell 都读取一次，后者只在登陆时读取一次。
```

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/Qt5.14.2/5.14.2/gcc_64/lib:/data/dev-lxy/searcher/lib
```

+ 脚本开头

**在bash脚本中，第一行的＃！/ bin / bash是什么意思？**

在Linux系统中，我们有解释我们的UNIX命令的shell。现在在Unix系统中有一些shell。其中，有一个叫做bash的shell，它是非常非常常见的Linux，它有着悠久的历史。这是Linux中的默认shell。

当你编写脚本（unix命令等的集合）时，你可以选择指定可以使用哪个shell。一般来说，你可以通过使用Shebang来指定它将使用的shell （是的，它就是它的名字）。

**＃！/ bin / bash和＃！/ bin / sh之间有区别吗？**

当你告诉＃！/ bin / bash时，你告诉你的环境/ os使用bash作为命令解释器。这是硬编码的东西。





## 不理解的命令

+ **sudo ldconfig**

ldconfig是一个**动态链接库**管理命令

安装完成某个工程后生成许多动态库，为了让这些动态链接库**为系统所共享**，还需运行动态链接库的管理命令–ldconfig。

## 常见报错

#### Unable to locate package

**问题:**  Unable to locate package XXX

方法1： `sudo apt-get update`

方法2：`sudo apt-get upgrade`

