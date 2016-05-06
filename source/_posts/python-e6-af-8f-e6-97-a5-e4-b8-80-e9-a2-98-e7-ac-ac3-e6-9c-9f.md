---
title: Python每日一题 第3期
tags:
  - python
  - 每日一题
  - 练习
id: 50
categories:
  - python
date: 2014-12-11 22:28:06
---

[题目：判断今天是2014年的第多少天？ ](http://www.pythonla.com/read-20.html "题目：判断今天是2014年的第多少天？ ")

time.mktime(tuple) 可以将一个时间对象转化为浮点型表示的时间戳，需要传入一个时间对象「实质上是一个长度为9的元组」，其返回值为传入从1970年1月1日8时整到现在过去的秒数。
另外，struct_time对象共9个元素元素，分别为，年，月，日，时，分，秒，星期几，今年的第几天，是否夏令时。因此也可以这样调用mktime： 
[python]
time.mktime((2014, 12, 11 ,21 ,50 ,30, 0, 0, 0))
[/python]

下面第一种是我一开始的写法，其基本思路是用当前时间的时间戳减去2014年1月1日0点整的时间戳得到今年过去的秒数。再除以86400后取整加1就可以得到当前是第几天。
后两种写法是在研究struct_time对象时发现的，可以直接获取struct_time格式的gmt时间和本地时间，其中第8项就是当前是本年的第几天。
[python]
import time
print(int((time.time() - time.mktime(time.strptime('2014', '%Y'))) / (60 * 60 * 24)) + 1)
print(time.localtime()[7])
print(time.gmtime()[7])
[/python]

有关python中time和datatime的更多用法可参见help(time)及help(datatime)