Python 标准库中的 unittest 模块，它提供了代码测试工具。

- 单元测试，单元测试用于核实某个函数的某个方面没有问题。
- 测试用例，测试用例是一组单元测试，这些单元测试一起核实函数在各个情形下的行为都符合要求。

### 1. 使用 unittest 模块测试代码

```

# 1. 引入unittest模块

# 2. 引入需要测试的函数

# 3. 创建一个继承unittest.Testcase的类

# 4. 给这个类添加方法用于对函数行为的测试

# name.py

# 在name.py文件创建一个函数，用于生成整洁的姓名。

def get_formatted_name(first_name, last_name):

    full_name = first_name + ' ' + last_name

    return full_name.title()

# test_name.py，在test_name.py文件中写测试代码

import unittest

from name import get_formatted_name

class NameFunctionTestCase(unittest.TestCase):

    def test_first_last_name(self):

        fotmatted_name = get_formatted_name('allen', 'walker')

        self.assertEqual(fotmatted_name, 'Allen Walker')

unittest.main()

```

结果是

![](/madao.github.io/database/images/articles/python/test_code/image.png)

测试通过了，看下代码：

1. 首先创建一个继承 unittest.TestCase 的类，这个类必须继承 unittest.TestCase，Python 才能知道如何运行编写的测试用例。
2. self.assertEqual(fotmatted_name, 'Allen Walker')，这一句。这就是一个断言，就是说我期待 fotmatted_name 这个变量的值等于 Allen Walker。
3. unittest.main()，是让 Python 运行文件中的测试。

unittest 模块中常用的断言方法：

| 方法                    | 用途                   |
| :---------------------- | :--------------------- |
| assertEqual(a, b)       | 核实 a == b            |
| assertNotEqual(a, b)    | 核实 a != b            |
| assertTrue(a)           | 核实 a 为 True         |
| assertFalse(a)          | 核实 a 为 False        |
| assertIn(item, list)    | 核实 item 在 list 中   |
| assertNotIn(item, list) | 核实 item 不在 list 中 |

再来看下没有通过的情况：

```
# 将上面例子中的  self.assertEqual(fotmatted_name, 'Allen Walker') 改为
#  self.assertEqual(fotmatted_name, 'Allen Balker')
```

结果：

![](/madao.github.io/database/images/articles/python/test_code/image1.png)

如果测试失败，Python 会告诉你原因，上图中 Python 说断言错误，Allen Walker != Allen balke。这时候我们就可以根据测试结果检查函数写错了还是测试用例写错了。

注意：测试方法必须以 test 开头，否则测试的时候是不会执行的。

```

import unittest

from name import get_formatted_name

class NameFunctionTestCase(unittest.TestCase):

    def test_first_last_name(self):  # 这个方法必须以test开头，才会自动执行

        fotmatted_name = get_formatted_name('allen', 'walker')

        self.assertEqual(fotmatted_name, 'Allen Walker')

unittest.main()

```

### 2. 测试一个类

测试类其实和测试一个函数差不多，看例子：

```

# survey.py

# 在survey.py文件中创建一个问卷调查类

class survey():

    def __init__(self, questions):

        self.questions = questions

        self.questionIndex = 0

        self.responses = []

    # 展示问题

    def show_question(self):

        if self.questionIndex < len(self.questions):

            return self.questions[self.questionIndex]

        else:

            print('感谢填写问卷\n')

            self.show_all_results()

            exit()

    # 存储用户输入

    def store_response(self, new_response):

        self.responses.append(new_response)

        self.questionIndex += 1

    # 展示所有用户输入

    def show_all_results(self):

        print('问卷结果：')

        for response in self.responses:

            print('- %s' % response)

# user_info_surevy.py

# 在user_info_surevy.py文件中编写一个调查用户信息的程序

from survey import survey

questions = [

  '你最喜欢的颜色是什么：',

  '你最喜欢的动物是什么：',

  '你最喜欢的数字是什么：'

]

user_info_survey = survey(questions)

print('输入q退出\n')

while True:

    response = input('%s' % user_info_survey.show_question())

    if response == 'q':

        break

    user_info_survey.store_response(response)

```

结果：

![](/madao.github.io/database/images/articles/python/test_code/image2.png)

测试 survey 类

```

import unittest

from survey import survey

class TestSurvey(unittest.TestCase):

    def test_store_single_response(self):

        question = ['你今天吃了吗？']

        my_survey = survey(question)

        my_survey.store_response('吃了')

        self.assertIn('吃了', my_survey.responses)

unittest.main()

```

结果：

![](/madao.github.io/database/images/articles/python/test_code/image3.png)

再添加一个测试多个答案的方法

```

import unittest

from survey import survey
class TestSurvey(unittest.TestCase):

    def test_store_single_response(self):

        question = ['你今天吃了吗？']

        my_survey = survey(question)

        my_survey.store_response('吃了')

        self.assertIn('吃了', my_survey.responses)

    # 新增

    def test_store_three_response(self):

        question = ['你今天吃了吗？']

        my_survey = survey(question)

        responses = ['吃了', '没有', '不喜欢']

        for response in responses:

            my_survey.store_response(response)

        for response in responses:

            self.assertIn(response, my_survey.responses)

unittest.main()
```

结果：

![](/madao.github.io/database/images/articles/python/test_code/image4.png)

可以看到测试通过了，但是代码有重复的地方，可以看到测试 survey 类的时候，每新增一个单元测试，都要创建一个 survey 的实例。这一部分可以简化。

- setUp()

  setUp 方法，会在运行每个测试方法调用前执行

- tearDown()

  tearDown 方法，会在运行每个测试方法调用后执行

所以将重复的部分写到 setUp 方法中：

```

class TestSurvey(unittest.TestCase):

    def setUp(self):

        question = ['你今天吃了吗？']

        self.my_survey = survey(question)

        self.responses = ['吃了', '没有', '不喜欢']

        for response in self.responses:

            self.my_survey.store_response(response)

    def test_store_single_response(self):

        self.assertIn('吃了', self.my_survey.responses)

    def test_store_three_response(self):

        for response in self.responses:

            self.assertIn(response, self.my_survey.responses)
unittest.main()

```

结果：

![](/madao.github.io/database/images/articles/python/test_code/image5.png)

Python 运行测试用例的时候，每完成一个单元测试，就会打印一个字符，通过打印一个`.`，引发错误打印一个`E`，断言失败打印一个`F`。

![](/madao.github.io/database/images/articles/python/test_code/image6.png)
