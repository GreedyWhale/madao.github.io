- ### time 模块

  time 模块提供了各种与时间相关的功能。该模块有很多方法，这里整理一下常用的几个：

  - time.time

    返回 1970 年 1 月 1 日 00:00:00 到现在的秒数。

    ```
    import time

    print(time.time()) # 1547704127.963818
    ```

  - time.localtime

    浮点数的时间戳转换为时间元组

    ```
    import time

    print(time.localtime())

    '''
    time.struct_time(tm_year=2019, tm_mon=1, tm_mday=17, tm_hour=13, tm_min=51, tm_sec=38, tm_wday=3, tm_yday=17, tm_isdst=0)
    '''
    ```

    时间元组的值的含义：

    | 下标 |           属性            |         值          |
    | :--: | :-----------------------: | :-----------------: |
    |  0   |       tm_year（年）       |      比如 2019      |
    |  1   |       tm_mon（月）        |       1 - 12        |
    |  2   |       tm_mday（日）       |       1 - 31        |
    |  3   |       tm_hour（时）       |       0 - 23        |
    |  4   |       tm_min（分）        |       0 - 59        |
    |  5   |       tm_sec（秒）        |       0 - 61        |
    |  6   |    tm_wday（weekday）     | 0 - 6（0 表示周日） |
    |  7   | tm_yday（一年中的第几天） |       1 - 366       |
    |  8   | tm_isdst（是否是夏令时）  |      默认为-1       |

  - time.strftime

    格式化时间

    ```
    import time

    print(time.strftime('%Y-%m-%d'))
    ```

    | 符号 | 含义                                           |
    | :--: | :--------------------------------------------- |
    |  %a  | 本地（locale）简化星期名称                     |
    |  %A  | 本地完整星期名称                               |
    |  %b  | 本地简化月份名称                               |
    |  %B  | 本地完整月份名称                               |
    |  %c  | 本地相应的日期和时间表示                       |
    |  %d  | 一个月中的第几天（01 - 31）                    |
    |  %H  | 一天中的第几个小时（24 小时制，00 - 23）       |
    |  %I  | 第几个小时（12 小时制，01 - 12）               |
    |  %j  | 一年中的第几天（001 - 366）                    |
    |  %m  | 月份（01 - 12）                                |
    |  %M  | 分钟数（00 - 59）                              |
    |  %p  | 本地 am 或者 pm 的相应符                       |
    |  %S  | 秒（01 - 61）                                  |
    |  %U  | 一年中的星期数。（00 - 53 星期天为星期的开始） |
    |  %w  | 一个星期中的第几天（0 - 6，0 是星期天）        |
    |  %W  | 一年中的星期数。（00 - 53 星期一为星期的开始） |
    |  %x  | 本地相应日期                                   |
    |  %X  | 本地相应时间                                   |
    |  %y  | 去掉世纪的年份（00 - 99）                      |
    |  %Y  | 完整的年份                                     |
    |  %Z  | 时区的名字（如果不存在为空字符）               |
    |  %%  | ‘%’字符                                        |

  - time.sleep

    暂停执行调用线程达到给定的秒数。参数可以是浮点数。（直白一点说就是让程序等待指定的秒数，再继续执行）

- ### datetime

  datetime 模块提供了日期操作相关的方法。

  - datetime.datetime.now

    返回当前的本地日期和时间。

    ```
    import datetime

    print(datetime.datetime.now()) # 2019-01-17 14:13:28.696311
    ```

  - datetime.timedelta

    设置时间的偏移量，比如想要获取 2012 年 12 月 1 日的 20 天之后的日期就可以这样写：

    ```
    import datetime

    start_date = datetime.datetime(2012,12,1)
    offset_date = datetime.timedelta(days=20)
    print(start_date + offset_date) # 2012-12-21 00:00:00
    ```

    datetime.timedelta 接受的参数用：

    |    参数名    |  含义  | 默认值 |
    | :----------: | :----: | :----: |
    |     days     |  天数  |   0    |
    |   seconds    |  秒数  |   0    |
    | microseconds | 微秒数 |   0    |
    | milliseconds | 毫秒数 |   0    |
    |   minutes    | 分钟数 |   0    |
    |    hours     | 小时数 |   0    |
    |    weeks     |  周数  |   0    |
