这里介绍一下，Python 中使用 JSON 格式存储数据。

[JSON](https://zh.wikipedia.org/wiki/JSON) --- 维基百科

使用 json 格式存储数据，需要用到 json 模块：

- json.load

  json.load 可以读取 json 文件中的数据：

  ![](/madao.github.io/database/images/articles/python/data_store/image.png)

  会把 json 格式的数据转换为 dict 类型

- json.dump

  dump 可以将 dict 类型的数据转换为字符串，并写入到 json 格式的文件中：

  ```
  user_info = {
      'name': 'Jack',
      'age': 100
  }
  with open('user.json', 'w') as user_file:
      json.dump({'jack': user_info}, user_file, indent=2, ensure_ascii=False)
  ```

  可以看到 dump 多传了两个参数，indent 是 json 数据的缩进，如果不传它默认就是一行，ensure_ascii 这个默认是 True，如果你希望传入的数据不被转换成 ASCII 码那么就传入 ensure_ascii=True

  结果：

  ![](/madao.github.io/database/images/articles/python/data_store/image1.png)

- json.loads

  loads 它不是从 json 文件中的读取数据，而是把一个 JSON 编码的字符串转换回一个 Python 数据结构例子：

  ![](/madao.github.io/database/images/articles/python/data_store/image2.png)

  注意这里的字符串格式，必须是符合 json 格式的才行。

- json.dumps

  json.dumps 可以将 dict 类型转换成符合 json 格式的字符串，例子：

  ![](/madao.github.io/database/images/articles/python/data_store/image3.png)

  同样也可以传入 indent 参数规定字符串的缩进格式

  最后总结一下：

  - load 和 dump，是会进行数据转换并且操作文件
  - loads 和 dumps，仅仅是数据操作
