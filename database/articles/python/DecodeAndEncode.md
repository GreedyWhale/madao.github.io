我们需要一个字典让计算机能读懂我们的语言，这个字典就叫做 - 编码表

编码：人类语言 -> 编码表 -> 机器语言
解码：机器语言 -> 编码表 -> 人类语言

因为最早的计算机由美国人发明，所以最开始只有英文的编码表 - ASCII，只支持：大写字母，小写字母，数字，和一些常用符号。

但是世界上有那么多语言，ASCII 编码表不能满足其他语言的需求，于是就诞生了很多编码表，常见的有：
| 编码表 | 适用性 | 特点 |
| :--: | :--: | :--: |
| ASCII | 支持英文大小写，常用的符合，数字，不支持中文 | 占用空间小 |
| GB1312，GBK | 支持中文 | GBK 码是 GB1312 的升级版 |
| Unicode | 支持国际语言 | 占用空间大，适用性强。|
| UTF-8 | 支持国际语言 | UTF-8 是 Unicode 的实现方式之一。也可以认为是 Unicode 的升级版，占用空间小。UTF-8 码包含 ASCII 码

数据在计算机的内存中，使用的是 Unicode 码，这是统一标准。

Python3 中使用 input 方法输入的内容也是用 Unicode 码进行编码。

在 Python 中编码和解码的方法是：

```
('你要编码的内容').encode('编码表名字')
('你要解码的内容').decode('编码表名字')
```

例子：

```
print('你好'.encode('utf-8'))
print('你好'.encode('gbk'))
print('abc'.encode('ASCII'))
print(b'\xe4\xbd\xa0\xe5\xa5\xbd'.decode('utf-8'))
print(b'\xc4\xe3\xba\xc3'.decode('gbk'))
print(b'abc'.decode('ASCII'))
```

结果：

![](/madao.github.io/database/images/articles/python/decode_and_encode/image.png)

编码结果最开始有一个`b`，它表示数据是 bytes（字节）类型

- ord

  单个字符的十进制整数编码。

- chr

  用一个范围在 0 ～ 255 整数作参数，返回一个对应的字符。

例子：

![](/madao.github.io/database/images/articles/python/decode_and_encode/image1.png)
