先把官方文档放在这里[pipenv](https://pipenv.readthedocs.io/en/latest/)

这里有中文的介绍

[Pipenv & 虚拟环境](https://pythonguidecn.readthedocs.io/zh/latest/dev/virtualenvs.html)

[Pipenv 上手指南](http://foofish.net/Pipenv_tutorial.html)

解释一下这个是什么，pipenv 就是管理项目依赖的工具，在写前端的时候，有个东西叫做 npm，它和 npm 很像，因为不同的项目要依赖不同的包，不可能将所有的包都存放在一个地方，所以需要 pipenv 为每个项目单独安装包，便于管理。

### 1. 安装

```
brew install pipenv
```

### 2. 检查安装是否成功

![](/madao.github.io/database/images/articles/python/pipenv/image.png)

### 3. 使用

- 创建一个项目文件夹 my_project，并进入这个文件夹

```
mkdir my_project
cd my_project
```

- 给这个项目安装一个依赖包

```
pipenv install requests
```

安装完成之后会自动的有两个文件

![](/madao.github.io/database/images/articles/python/pipenv/image1.png)

Pipfile 这个文件记录了包的信息：

```
  [[source]]
  name = "pypi"
  url = "https://pypi.org/simple"
  verify_ssl = true

  [dev-packages]

  [packages]
  request = "*"

  [requires]
  python_version = "3.7"

```

1. source 代表包的来源
2. dev-packages 代表开发阶段使用的包。
3. packages 就是项目依赖的包，\*代表始终下载最新版本
4. requires 表示项目 python 的版本

### 4. 常用的命令

- pipenv --venv

  查看虚拟环境安装的位置

- pipenv --rm

  删除虚拟环境

- pipenv uninstall packageName

  删除包

- pipenv shell

  激活虚拟环境，激活了之后，就可以在当前的环境中使用项目中安装的包了，例子：

  ![](/madao.github.io/database/images/articles/python/pipenv/image2.png)

  上图中的(my_project) 就代表你当前所在项目的虚拟环境。要退出当前虚拟环境只要输入
  exit 即可

- pipenv run python3 file_name.py

  在不用激活虚拟环境的情况下执行 Python 文件
