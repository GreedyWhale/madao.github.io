爬虫简单来讲就是，让程序通过一些设置好的规则去网页上查找我们想要的内容，我还没有爬过有验证码，登录等一系列需要鉴权的网站。所以这里仅仅记录一下我自己写的最简单的爬虫实现。

1. 找到想要爬取的网页
2. 确定数据源

   在网页上的数据源，有两种：

   - 通过 ajax 请求接口获取。
   - 数据就在页面上，并没有单独的 ajax 请求获取数据。

   这两种数据源，都可以通过浏览器自带的开发工具查看，也可以通过抓包工具，比如 charles 抓取网络请求查看。

   这里说下第一种，因为抓包工具配起来挺麻烦的。

   这里用 chrome 举例，随便打开一个页面，

   ![](/madao.github.io/database/images/articles/python/crawler/image.png)

   也可以直接双击页面，点击检查

   ![](/madao.github.io/database/images/articles/python/crawler/image1.png)

   然后会得到这样一个界面，点击 NetWork 这个选项，刷新页面

   ![](/madao.github.io/database/images/articles/python/crawler/image2.png)

   然后就可以看到各种请求

   ![](/madao.github.io/database/images/articles/python/crawler/image3.png)

   这里可以根据请求的类型查看，一般通过 ajax 请求接口的数据都在 XHR 中，页面上的数据，也就是直接返回一个渲染好的页面在 Doc 中。

   现在先看第一种数据，通过接口获取的数据：

   ![](/madao.github.io/database/images/articles/python/crawler/image4.png)

   上图中，左边部分就是请求列表，右边就是接口返回的数据，一个一个看找到页面上对应的内容就行。

   还有一种是直接返回渲染好的页面或者是直接返回一段 html，还是一样的也是这样找。例子：

   ![](/madao.github.io/database/images/articles/python/crawler/image5.png)

   像豆瓣这种，它会直接返回渲染好的 html。这时候找就要在 Doc 选项下面找

   还有一种像携程评论的这种，

   ![](/madao.github.io/database/images/articles/python/crawler/image6.png)

   点击左边的 response 选项，可以看到它也是一段 html

   ![](/madao.github.io/database/images/articles/python/crawler/image7.png)

   幸运的是，Python 都有对应的模块，来处理这些情况。

3. 确定完数据源之后，介绍 2 个模块

   1. requests
      [文档](http://cn.python-requests.org/zh_CN/latest/)
   2. BeautifulSoup
      [文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#attributes)

   这两个模块的文档都非常清晰，这里就不做多介绍了。

4. 分析网页获取数据的规律

   1. 接口一般都有请求参数，请求参数在这里看：

   ![](/madao.github.io/database/images/articles/python/crawler/image8.png)

   参数的类型也有不同的，比如 json 格式的，form-data 等等格式，上图中参数是通过路由带的就是域名问号后面的这些：

   ![](/madao.github.io/database/images/articles/python/crawler/image9.png)

   这个页面是一个搜索电影资源的页面，通过路由上 keyword 可以猜出这个一般都是电影的名字，只不过被编码了，这时候就要判断这个页面用的编码格式是哪一种了，因为是中文网站，所以一般都是 utf-8 或者 gbk 的。直接在终端里试一下，例子中我搜的电影是千与千寻，那么在终端里将千与千寻分别编码试下：

   ![](/madao.github.io/database/images/articles/python/crawler/image10.png)

   可以看到使用 gbk 编码的结果只要将\x 全部换为%那么就和网址中的 keyword 一样了，如何将正常人能看懂的字符串变为网址中的那样呢，这里需要用到 Python 中内置的一个函数 quote。例子：

   ```
   from urllib.parse import quote

   keyword = '千与千寻'.encode('gbk')
   print('http://s.ygdy8.com/plus/so.php?typeid=1&keyword=%s' % quote(keyword))
   ```

   结果：

   ![](/madao.github.io/database/images/articles/python/crawler/image11.png)

   这样就保持一致了。

   那这个页面数据获取的规律就是通过不断变化 keyword 来获取不同电影的资源列表

   还要注意的一点是，请求也是有不同方法的，比如 post，get。具体参考：

   [HTTP 请求方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)

   一般都是 get 或者 post。请求方法在这里看：

   ![](/madao.github.io/database/images/articles/python/crawler/image12.png)

5. 反爬虫策略

   正常的网站一般都不喜欢爬虫，当然除了搜索引擎的爬虫，因为爬虫不像是用户，
   爬虫可以用非常快的频率去请求数据，一般用户是无法做到这么快的，这样会给服务器带来巨大的负担，所以大部分网站都有反爬虫策略，像什么验证码，cookie，token 之类的。目前我学会的破解方法就两种：

   1. 如果可以拿到 cookie 那么请求的时候就带上，cookie 在这里看

   ![](/madao.github.io/database/images/articles/python/crawler/image13.png)

   在请求头中设置 User-Agent 字段，这个字段的值可以在上图中看到。

   用 requests 模块举个例子：

   ```

   # 将cookie字符串变为字典

   import requests

   def set_cookies(cookie_str):

       cookies = {}

       for cookie in cookie_str.split(';'):

           key, value = cookie.split('=', 1)

           cookies[key] = value

       return cookies

   headers = {

       'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'

   }

   data = requests.get('xxx', headers=headers, cookies=cookies)

   ```

   1. 使用 time 模块中的 sleep 方法，增加请求的间隔时间

   更高级的用法还没学会，ip 代理，Selenium 库等等

由于这个电影资源那个网站广告太多，html 结构混乱，爬起来很难受，所以下面用豆瓣电影 top 250 的页面来实现一个简单的爬虫：

```

import requests

import json

import time

from bs4 import BeautifulSoup

movie_list = []

# 设置cookie

def set_cookies(cookies_str):

    cookies = {}

    for cookie in cookies_str.split(';'):

        key, value = cookie.split('=', 1)

        cookies[key] = value

# 获取豆瓣电影top 250的数据

def get_douban_html(start_num):

    # 这里观察豆瓣电影top 250的数据，是通过start这个参数改变来获取的

    url = 'https://movie.douban.com/top250?start=%d&filter=' % start_num

    headers = {

        'User-Agent':

        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'

    }

    # cookie 从控制台的请求中获取

    cookies_str = 'bid=HpoHjdZefY0; douban-fav-remind=1; __utmc=30149280; __utmc=223695111; __yadk_uid=S3jj0SCU2oe6tpu8M5QfYu2q8ABdiwNU; ll="118281"; _vwo_uuid_v2=DB312EBA83E11CE226E8EE97492FAC19A|6ae39156dc22651e7b2877a68d64f226; __utmz=30149280.1545817444.8.8.utmcsr=pypypy.cn|utmccn=(referral)|utmcmd=referral|utmcct=/; __utmz=223695111.1545817444.2.2.utmcsr=pypypy.cn|utmccn=(referral)|utmcmd=referral|utmcct=/; ap_v=0,6.0; _pk_ref.100001.4cf6=%5B%22%22%2C%22%22%2C1545976162%2C%22https%3A%2F%2Fwww.pypypy.cn%2F%22%5D; _pk_ses.100001.4cf6=*; __utma=30149280.184507092.1540456075.1545901908.1545976162.11; __utmb=30149280.0.10.1545976162; __utma=223695111.1027700142.1544019184.1545901908.1545976162.5; __utmt=1; _pk_id.100001.4cf6=8fbf19d431e46f6f.1544019183.5.1545976165.1545901930.; __utmb=223695111.2.10.1545976162'

    cookies = set_cookies(cookies_str)

    douban_html = requests.get(url, headers=headers, cookies=cookies)

    # 改一下拿到的html的编码，这个是在请求中可以找到，在Response Headers的Content-Type中可以看到

    douban_html.encoding = 'utf-8'

    return douban_html

# 拿到html之后，解析html

def set_movie_list(html):

    # 这里拿到的数据是html文档。所以要借助BeautifulSuop这个模块去解析

    douban_html_bs4 = BeautifulSoup(html.text, 'html.parser')

    # BeautifulSuop中有个叫select方法，它可以用css选择器的规则去获取元素。

    movie_list_html = douban_html_bs4.select('.item')

    for movie in movie_list_html:

        movie_list_item = {

            'movie_post': movie.select('.pic img')[0].get('src'),  # 电影海报

            'movie_name': movie.select('.info .hd a')[0].getText(),  # 电影名

            'movie_score':

            movie.select('.info .bd .rating_num')[0].getText(),  # 电影评分

        }

        movie_short_comment = movie.select('.info .bd .quote')

        # 这个是电影短评，我在爬的时候发现有的电影是没有的，所以要做个判断

        if movie_short_comment:

            movie_list_item['movie_short_comment'] = movie_short_comment[

                0].getText()

        movie_list.append(movie_list_item)

print('开始爬取数据')

# 从豆瓣电影top 250的最后一页看出，它最后一页的start是255，每次间隔25个

# 所以这里用range(0, 226, 25)每隔25个请求一次

for index in range(0, 226, 25):

    data = get_douban_html(index)

    set_movie_list(data)

    # 间隔一秒，避免被封

    time.sleep(1)

    print('当前进度%d' % (25 + index))

with open('douban_movie.json', 'w') as file:

    json.dump(movie_list, file, indent=4, ensure_ascii=False)

    print('爬取结束，数据已存放至当前目录下的douban_movie.json中')

```

结果：

![](/madao.github.io/database/images/articles/python/crawler/image14.png)

douban_movie.json
![](/madao.github.io/database/images/articles/python/crawler/image15.png)
