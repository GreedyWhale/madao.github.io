上一个笔记总结了 asyncio 的一些知识点，这次就来应用一下。如果使用协程的方式来写爬虫，网络相关的请求就要将 requests 库替换成 aiohttp 这个库。

- [官方文档](https://aiohttp.readthedocs.io/en/stable/index.html)
- [中文文档](https://hubertroy.gitbooks.io/aiohttp-chinese-documentation/content/SREADME.html)

### 一. 效率对比

1. 用上次写的爬虫，[先爬一些壁纸链接](#/article/PythonDownloadImages)

   ```
   urls = [
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-729560.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-724055.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716644.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716643.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716645.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686220.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686212.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-652608.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639894.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639893.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639892.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639890.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639888.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-468197.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467016.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467012.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467009.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467007.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467005.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466997.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466998.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466993.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466994.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466995.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-729560.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-724055.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716644.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716643.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716645.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686220.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686212.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-652608.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639894.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639893.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639892.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639890.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639888.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-468197.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467016.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467012.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467009.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467007.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467005.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466997.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466998.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466993.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466994.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466995.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466992.jpg",
       "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466989.jpg"
   ]
   ```

2. 用协程的方式下载这些图片

   - 安装 aiohttp 库
   - 根据文档，aiohttp 库发起请求使用的是 aiohttp.ClientSession()，文档建议不用每次请求都创建一个 session 会话，所以这里就只创建一个：

     ```
     async def main():
         async with aiohttp.ClientSession() as session:
             pass
     ```

   - 定义下载图片的协程函数和创建储存路径的函数：

     ```
     # 下载图片
     async def download_img(session, url):
         image_name = url.split('/')[-1]
         async with session.get(url, headers=headers) as response:
             with open('%s/%s' % (get_store_path('city'), image_name), 'wb') as fd:
                 while True:
                     chunk = await response.content.read(200)
                     if not chunk:
                         break
                     fd.write(chunk)
     ```

     ```
     # 获取图片储存路径，如果没有则创建
     def get_store_path(dir_name):
         current_path = os.path.abspath('.')
         target_path = os.path.join(current_path, 'wallpaper/%s' % dir_name)
         folder = os.path.exists(target_path)
         if not folder:
             os.makedirs(target_path)

         return target_path
     ```

   - 补全 main 函数
     ```
     async def main(loop):
         async with aiohttp.ClientSession() as session:
             tasks = [loop.create_task(download_img(session, url)) for url in urls]
             await asyncio.wait(tasks)

     loop = asyncio.get_event_loop()
     loop.run_until_complete(main(loop))
     loop.close()
     ```
   - 计算下载耗时：

     ```
     if __name__ == '__main__':
         t1 = time.time()
         loop = asyncio.get_event_loop()
         loop.run_until_complete(main(loop))
         loop.close()
         print('耗时：%fs' % (time.time() - t1))
     ```

   - 结果：

     ![](/caisr.github.io/database/images/articles/python/aiohttp/image.png)

     26 张图片用时 1.476328s

3. 使用多进程方式下载

   ```
    import requests
    import multiprocessing
    import os
    import time


    urls = [
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-729560.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-724055.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716644.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716643.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716645.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686220.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686212.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-652608.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639894.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639893.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639892.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639890.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639888.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-468197.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467016.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467012.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467009.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467007.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467005.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466997.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466998.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466993.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466994.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466995.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-729560.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-724055.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716644.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716643.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-716645.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686220.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-686212.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-652608.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639894.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639893.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639892.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639890.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-639888.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-468197.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467016.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467012.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467009.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467007.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-467005.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466997.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466998.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466993.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466994.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466995.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466992.jpg",
        "https://alpha.wallhaven.cc/wallpapers/thumb/small/th-466989.jpg"
    ]

    req_session = requests.Session()
    req_session.headers['user-agent'] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'


    def get_store_path(dir_name):
        current_path = os.path.abspath('.')
        target_path = os.path.join(current_path, 'wallpaper/%s' % dir_name)
        folder = os.path.exists(target_path)
        if not folder:
            os.makedirs(target_path)

        return target_path


    def download_img(url):
        img = req_session.get(url, stream=True)
        image_name = url.split('/')[-1]
        with open('%s/%s' % (get_store_path('city'), image_name), 'wb') as fd:
            for chunk in img.iter_content(chunk_size=128):
                fd.write(chunk)


    def main():
        p = multiprocessing.Pool()
        [p.apply_async(download_img, args=(url,)) for url in urls]
        p.close()
        p.join()

    if __name__ == '__main__':
        t1 = time.time()
        main()
        print('耗时：%fs' % (time.time() - t1))

   ```

   结果

   ![](/caisr.github.io/database/images/articles/python/aiohttp/image1.png)

两种我都执行了多次，差不多都在 1.4~2.7s 之间，可以看到协程还是很强大的，仅仅用单线程就做到了类似多进程的效果。
