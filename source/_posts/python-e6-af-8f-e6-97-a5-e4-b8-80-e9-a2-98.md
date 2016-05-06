---
title: Python每日一题 第1期
tags:
  - python
  - 每日一题
  - 练习
id: 37
categories:
  - python
date: 2014-12-11 12:20:12
---

这两天参加的一个群里开展了每日一题的活动，虽然题目简单了点，但是拿来练练手也挺不错的。

[【我们一起学Python吧】每日一题 第1期](http://www.pythonla.com/read-13.html "【我们一起学Python吧】每日一题 第1期")
题目：有1、2、3、4个数字，能组成多少个互不相同且无重复数字的三位数？都是多少？[](http://www.pythonla.com/read-13.html "【我们一起学Python吧】每日一题 第1期")
[python]
count = 0
for i in range(1, 4 ** 3):
    a = i % 4
    b = int(i / 4) % 4
    c = int(i / 16)
    if (a != b) &amp; (a != c) &amp; (b != c):
        count += 1
        print(str(c + 1) + str(b + 1) + str(a + 1))
print('total:%d' % count)
[/python]

这是群中高手的解法，用到了列表解析。
[python]
lis = {1, 2, 3, 4}
l = [x * 100 + y * 10 + z for x in lis for y in lis - {x} for z in lis - {x} - {y}]
print(len(l), l)
[/python]