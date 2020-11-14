Python 中正则表达式相关的都在 re 模块中。简单整理一下常见的语法和方法

- ### 关于转义字符

  如果执行下面了下面这句代码：

  `print('a\nb')`

  那么会得到这样的结果：

  ![](/caisr.github.io/database/images/articles/python/regexp/image.png)

  并没有打印出\n 只有 a 和 b，而且它们还换行了。如果想要打印`\n`这个字符串有两种办法：

  ```
  print('a\\nb')
  print(r'a\nb')
  ```

  ![](/caisr.github.io/database/images/articles/python/regexp/image1.png)

  第一种就是再加一个转义符号（\）将原本的转义符转义成原本的意思。第二种是使用`r'字符串'`这种格式。推荐用第二种。

- ### re.match 和 re.search

  re.match 和 re.search 很像，它们都用于判断匹配是否成功，如果成功就返回一个对象，不成功就返回 None

  ```
  import re

  res_match = re.match(r'ab', 'ababababab')
  res_search = re.search(r'ab', 'ababababab')

  print(res_match) # <re.Match object; span=(0, 2), match='ab'>
  print(res_search) # <re.Match object; span=(0, 2), match='ab'>
  ```

  它们两的区别在于，如果字符串的开头就匹配失败，match 方法会直接返回 None，search 则会继续匹配下去。例子：

  ```
  import re

  res_match = re.match(r'ab', 'ccccababababab')
  res_search = re.search(r'ab', 'ccccbababababab')

  print(res_match)  # None
  print(res_search) # <re.Match object; span=(5, 7), match='ab'>
  ```

  match 和 search 方法返回的对象中 span 表示匹配成功的字符的位置，match 则是匹配结果，它们返回的对象都有 group 和 groups 方法。

  直接看例子：

  ```
  import re

  res_match = re.match(r'(ab)-(cd)-(ef)', 'ab-cd-ef')


  print('group不带参数：', res_match.group())
  print('group带参数：', res_match.group(1))
  print('group带参数：', res_match.group(2))
  print('group带参数：', res_match.group(3))
  print('group带多个参数：', res_match.group(1, 2, 3))

  print('groups:', res_match.groups())
  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/regexp/image2.png)

  group 和 groups 方法会返回匹配到的字符，如果在正则表达式中用了`()`那么表示匹配括号内的所有内容，一个`()`就是一个组， group 方法接受的参数就是组的序号。组从 1 开始。

- re.compile

  compile 方法用于创建一个正则表达式（ Pattern ）对象

  ```
  prog = re.compile(pattern)
  result = prog.match(string)
  ```

  等价于

  ```
  result = re.match(pattern, string)
  ```

  例子：

  ```
  import re
  reg = re.compile(r'\d')

  res_match = reg.match('12313131')
  res_search = reg.search('abcdefg12313131')

  print(res_match.group()) # 1
  print(res_search.group()) # 1
  ```

  可以看到上面例子中数字有很多个，但是只匹配到一个就不匹配了。如果想要匹配所有符合规则的字符串可以用 findAll 方法，例子：

  ```
  import re
  reg = re.compile(r'\d')

  res_match = reg.match('12313131')
  res_search = reg.search('abcdefg12313131')
  res_findAll = reg.findall('a1h2h1312h31hjg31j3')

  print(res_match.group()) # 1
  print(res_search.group()) # 1
  print(res_findAll)  # ['1', '2', '1', '3', '1', '2', '3', '1', '3', '1', '3']
  ```

- ### re.sub

  re.sub 用于替换字符串中的匹配项。它的语法是：

  `re.sub(pattern, repl, string, count=0, flags=0)`

  - pattern : 正则中的模式字符串。
  - repl : 替换的字符串，也可为一个函数。
  - string : 要被查找替换的原始字符串。
  - count : 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配。

  例子：

  ```
  import re
  test_str = '2012-12-21'

  def square(num):
      return str(int(num.group())**2)


  res = re.sub(r'\D', '/', test_str)
  res_func = re.sub(r'\d', square, test_str)

  print(res) # 2012/12/21
  print(res_func) # 4014-14-41
  ```

- ### 贪婪模式

  在说贪婪模式之前，先介绍几个特殊字符：`*, +, ?`

  - \*: 匹配 0 到任意次
  - +: 匹配 1 到任意次
  - ?: 匹配 0 到 1 次

  例子：

  ```
  import re
  reg = re.compile(r'ab*')
  reg_1 = re.compile(r'ab+')
  reg_2 = re.compile(r'ab?')

  test_str = 'abbbbbbbbbbbbbb'

  res_xinghao = reg.match(test_str)
  res_jiahao = reg_1.match(test_str)
  res_wenhao = reg_2.match(test_str)

  print('*:', res_xinghao.group()) # *: abbbbbbbbbbbbbb
  print('+:', res_jiahao.group())  # +: abbbbbbbbbbbbbb
  print('?:', res_wenhao.group())  # ?: ab
  ```

  `*, +, ?` 它们都是贪婪模式，也是就在匹配成功的情况下，尽可能的多匹配符合规则的字符。

  如果想要非贪婪模式那么只需要在它们后面各自加上`?`即可：`*?, +?, ??`

  ```
  import re
  reg = re.compile(r'ab*?')
  reg_1 = re.compile(r'ab+?')
  reg_2 = re.compile(r'ab??')

  test_str = 'abbbbbbbbbbbbbb'

  res_xinghao = reg.match(test_str)
  res_jiahao = reg_1.match(test_str)
  res_wenhao = reg_2.match(test_str)

  print('*?:', res_xinghao.group())  # *?: a
  print('+?:', res_jiahao.group())   # +?: ab
  print('??:', res_wenhao.group())   # ??: a
  ```

正则表达式还有很多的特殊字符，也叫元字符。官方文档解释的更清楚，这里就不写了。它们的组合很多，这也是我觉得正则难的原因。
