### 一. requests.Session()

上一篇总结爬虫的文章里，有说道一个 cookie 的问题。当时我都是手动解析，手动添加的。后来才知道 requests 模块居然有一个 Session 功能，可以保持 cookie。这里记录一下：

```
import requests

base_url = 'https://www.zhihu.com'

# 未登录装态
r_session = requests.Session()
r_session.headers['User-Agent'] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'
zhihu = r_session.get(base_url)
print(zhihu.request.headers)

# 登录
data = {'username': 'username', 'password': 'password'}
zhihu_sign_in = r_session.post('https://www.zhihu.com/api/v3/oauth/sign_in', data=data)
print(zhihu_sign_in.request.headers)
```

结果：

![](/caisr.github.io/database/images/articles/python/download_images/image.png)

用了 Session 就会自动带上 cookie 了。

### 二. 如何下载一个图片

- urlretrieve

  在 Python 自带的模块 urllib 中提供了一个方法 urlretrieve，可以用这个方法下载一个图片：

  ```
  from urllib.request import urlretrieve
  import os

  img_url = 'https://wx4.sinaimg.cn/mw690/70396e5agy1fxlqvsv93sj20ku0yu42u.jpg'
  urlretrieve(img_url, './image.jpg')
  ```

  传入图片地址和图片要存放的地址即可。

- requests 模块

  ```
  import requests

  img_url = 'https://wx4.sinaimg.cn/mw690/70396e5agy1fxlqvsv93sj20ku0yu42u.jpg'
  img = requests.get(img_url)
  with open('./image2.jpg', 'wb') as f:
    f.write(img.content)
  ```

  使用 requests 模块也可以下载图片，通过 open 方法将下载下来的图片通过二进制方式写入文件中

  如果下载的图片（除了图片其他资源也一样，比如视频）特别大，那么还可以用

  ```
  import requests

  img_url = 'https://wx4.sinaimg.cn/mw690/70396e5agy1fxlqvsv93sj20ku0yu42u.jpg'

  img = requests.get(img_url, stream=True)
  with open('./image3.jpg', 'wb') as f:
      for chunk in img.iter_content(chunk_size=10):
          f.write(chunk)
  ```

  这种方式，加上 steam=True 参数，可以让 requests 下载一点保存一点，而不是等全部下载完成再进行保存，然后通过 chunk_size 控制 chunk 的大小

- 实现一个下载壁纸的爬虫

  ```

  import requests

  import time

  import random

  from bs4 import BeautifulSoup

  import os

  from functools import reduce





  base_url = 'https://alpha.wallhaven.cc/' # 壁纸网址

  req_session = requests.Session()

  req_session.headers['user-agent'] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'





  def get_store_path(dir_name):

      '''

      创建存放壁纸文件目录

      '''

      current_path = os.path.abspath('.')

      target_path = os.path.join(current_path, 'wallpaper/%s' % dir_name)

      folder = os.path.exists(target_path)

      if not folder:

          os.makedirs(target_path)



      return target_path





  def get_img_html(page_size):

      '''

      获取壁纸的html资源

      '''

      htmls = []

      for i in range(1, page_size):

          params = {

              'q': keyword,

          }

          # 由于该网站page是从2开始的，如果带了小于2的值就会找不到壁纸

          if i >= 2:

              params['page'] = i

          html = req_session.get('%s/search' % base_url, params=params)

          htmls.append(html)

          time.sleep(random.random())

      return htmls





  def parse_html(html):

      '''

      从html中将壁纸地址解析出来

      '''

      imgs_src = []

      bs_html = BeautifulSoup(html.text, 'lxml')

      img_tags = bs_html.find_all('img', {'class': 'lazyload'})

      if len(img_tags):

          for img in img_tags:

              imgs_src.append(img['data-src'])

      return imgs_src





  def download_imgs(imgs_src):

      '''

      下载壁纸

      '''

      if len(imgs_src):

          for index, src in enumerate(imgs_src):

              img = req_session.get(src, stream=True)

              img_name = src.split('/')[-1]

              store_path = '%s/%s' % (get_store_path(keyword), img_name)

              with open(store_path, 'wb') as f:

                  for chunk in img.iter_content(chunk_size=128):

                      f.write(chunk)

                  print('下载进度：%d/%d' % (index + 1, len(imgs_src)))

          print('下载完成，图片已存放在：%s' % store_path)

      else:

          print('没有找到相关主题的壁纸')





  keyword = input('请输入你想要下载的壁纸主题：')

  page_size = int(input('请输入下载壁纸的页数，每页24张：'))

  print('开始查找壁纸....')

  html_list = get_img_html(page_size)

  img_src_list = []

  if len(html_list):

      print('查找完成')

      print('开始爬取壁纸地址...')

      for html in html_list:

          img_src_list.append(parse_html(html))

      img_src_list = reduce(lambda x, y: x+y, img_src_list)

      print('爬取完成')

      print('开始下载壁纸...')

      download_imgs(img_src_list)

  else:

      print('没有找到相关主题的壁纸')



  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/download_images/image1.png)

  ![](/caisr.github.io/database/images/articles/python/download_images/image2.png)
