记录一下，这两天写的一个日历实现的思路

### 1. 思路

一般的日历都是这样的

![](/madao.github.io/database/images/articles/gadgets/calendar/image.png)

主要的几点：

1.  当前月有多少天
2.  当前月的第一天是星期几
3.  当前月的最后一天是星期几

### 2. 实现

1.  获取月的天数

    ```

    function getOneMonthDays (year, month) {

        return new Date(year, month, 0).getDate()

    }

    ```

2.  获取某天的星期数

    ```

    function getDayOfWeek (year, month, day) {

        return new Date(year, month - 1, day).getDay()

    }

    ```

3.  生成日期数组

    有了这两个方法就可以生成每个月的日期数组了

    ```

    // 这个用来填充日期列表

    function setDateListItem ({ year, month, date, dateTableIndex }) {

        return {

            dateStamp: `${year}/${month}/${date}`,

            date,

            cnWeek: dateTable.cnWeek[dateTableIndex],

            cnWeekShort: dateTable.cnWeekShort[dateTableIndex],

            enWeek: dateTable.enWeek[dateTableIndex],

            enMonth: dateTable.enMonth[month - 1]

        }

    }

    // 日期数组中每一项的内容

    const dateTable = {

        cnWeek: ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'],

        cnWeekShort: ['日', '一', '二', '三', '四', '五', '六'],

        enWeek: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],

        enMonth: [

            'January', 'February', 'March', 'April', 'May',

            'June', 'July', 'August', 'September', 'October',

            'November', 'December'

        ]

    }



    // 设置日期数组

    function setDateList (year, month) {

        const currentMonthDays = getOneMonthDays(year, month)

        const firstDayInCurrentMonth = getDayOfWeek(year, month, 1)

        const dateList = []



        /*

          如果当前月不是从星期日开始，从上个月补全，因为

          我是从周日开始，getDay也是从0开始到6，所以这里判断

          当前月的第一天不是0，也就是不是周日，那么它前面就需要

          补上个月的日期。

        */

        if (firstDayInCurrentMonth !== 0) {

            // 这里要判断1月的情况，1月的上个月就成去年的12月了

            const isPrveYear = (month - 1) === 0

            const prevMonth = isPrveYear ? 12 : month - 1

            const currentYear = isPrveYear ? year - 1 : year

            const prveMonthDays = getOneMonthDays(currentYear, prevMonth)

            // 这里是补上个月的，所以从上个月的最后开始补

            for (let i = firstDayInCurrentMonth - 1; i >= 0; i -= 1) {

                const index = getDayOfWeek(currentYear, prevMonth, prveMonthDays - i)

                dateList.push(

                    setDateListItem({

                        year: currentYear,

                        month: prevMonth,

                        date: prveMonthDays - i,

                        dateTableIndex: index

                    })

                )

            }

        }



        // 当月日期添加，这里就没什么好说的，直接循环就行了

        for (let i = 0; i < currentMonthDays; i += 1) {

            const index = getDayOfWeek(year, month, 1 + i)

            dateList.push(

                setDateListItem({

                    year,

                    month,

                    date: i + 1,

                    dateTableIndex: index

                })

            )

        }



        const currentDateListLen = dateList.length

        /*

        当月结束时未填满整个行, 这里用了余数来判断，因为有时候是35个，

        有时候是42个，所以只要前面的日期数组余7不等于0，那么证明没有

        填满，需要下个月的天数来补

        */

        if (currentDateListLen % 7) {

            const total = currentDateListLen + (7 - (currentDateListLen % 7))

            const diff = total - currentDateListLen

            const isNextYear = (month + 1) > 12

            const nextMonth = isNextYear ? 1 : month + 1

            const currentYear = isNextYear ? year + 1 : year

            for (let i = 0; i < diff; i += 1) {

                const index = getDayOfWeek(currentYear, nextMonth, 1 + i)

                dateList.push(

                    setDateListItem({

                        year: currentYear,

                        month: nextMonth,

                        date: i + 1,

                        dateTableIndex: index

                    })

                )

            }

        }



        return dateList

    }

    ```

测试一下

![](/madao.github.io/database/images/articles/gadgets/calendar/image1.png)

应该是没啥问题的，这就可以拿到每个月的日期数组，然后就是细节了，样式啊之类的。

这里就做了个小 demo。具体长这样

![](/madao.github.io/database/images/articles/gadgets/calendar/image2.png)

代码在这里

[github](https://github.com/GreedyWhale/otaku-date/tree/dev)

后面一段时间，要开启 node.js 之旅了。感觉学的东西好多啊。
