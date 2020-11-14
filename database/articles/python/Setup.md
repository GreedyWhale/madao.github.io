系统：macOS

编辑器：VScode

Python 版本：3.7.1

macOS 系统自带 Python，不过是 2.7.10 的，所以需要自己安装 python3 的版本，推荐使用 Homebrew 安装。

我是根据这个教程安装的[在 Mac OS X 上安装 Python 3](https://pythonguidecn.readthedocs.io/zh/latest/starting/install3/osx.html)

书：《Python 编程从入门到实践》

### 1. Python 版本查看

在终端输入

```
python --version
```

![](/caisr.github.io/database/images/articles/python/setup/image.png)

这个版本是系统自带的

```
python3 --version
```

![](/caisr.github.io/database/images/articles/python/setup/image1.png)

这个就是自己安装的版本

### 2. exit

如果直接在终端输入`python3`，那么就会进入 Python 的环境，可以直接写 Python 的代码。退出则要输入 exit()

![](/caisr.github.io/database/images/articles/python/setup/image2.png)

### 3. VScode 配置

因为目前完全不了解 Python，所以先简单的配置两个，一个是 flake8，一个是 yapf。

- flake8，是代码语法检查工具
- yapf，是代码格式化工具

安装命令是：

```
pip3 install flake8
pip3 install yapf
```

pip 类似于 npm，用来下载 Python 的包，还没有深入研究。

VScode 的配置添加为：

![](/caisr.github.io/database/images/articles/python/setup/image3.png)

pylint 也是代码语法检查工具，是 VScode 默认的检查工具。

格式化代码的快捷键是 alt + shift + f

还有一点是当用 VScode 打开一个 Python 文件的时候，如果没有指定 Python 的版本，需要手动的指定一下。

![](/caisr.github.io/database/images/articles/python/setup/image4.png)

选择 python3.7.1 就好了。里面有两个，我选的是第二个，也就是 3.7 的那个。

好了，简单的配置就完成了

试着写出第一个 Python 程序

```
print('Hello Python world!')
```

![](/caisr.github.io/database/images/articles/python/setup/image5.png)
