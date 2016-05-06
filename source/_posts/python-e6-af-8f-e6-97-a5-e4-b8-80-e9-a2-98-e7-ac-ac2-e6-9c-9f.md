---
title: Python每日一题 第2期
tags:
  - python
  - 每日一题
  - 练习
id: 41
categories:
  - python
date: 2014-12-11 12:25:45
---

[【我们一起学Python吧】每日一题 第2期](http://www.pythonla.com/read-17.html "【我们一起学Python吧】每日一题 第2期")
题目：企业发放的奖金根据利润提成。利润(I)低于或等于10万元时，奖金可提10%；利润高于10万元，低于20万元时，低于10万元的部分按10%提成，高于10万元的部分，可可提成7.5%；20万到40万之间时，高于20万元的部分，可提成5%；40万到60万之间时高于40万元的部分，可提成3%；60万到100万之间时，高于60万元的部分，可提成1.5%，高于100万元时，超过100万元的部分按1%提成，从键盘输入当月利润I，求应发放奖金总数？ 

[python]
ratios = ((0, 0.1), (10, 0.075), (20, 0.05), (40, 0.03), (60, 0.015), (100, 0.01))
while 1:
    i_str = input('input a number:')
    if '' == i_str:
        exit(0)
    i = int(i_str)
    bonus = 0
    last = 0
    for ratio in ratios:
        if ratio[0] &lt; i:
            bonus += (i - ratio[0]) * (ratio[1] - last)
            last = ratio[1]
    print(round(bonus, 2))
[/python]

群里的解法，利用字典实现了switch，虽然有滥用eval的嫌疑，但是实现方式还是值得借鉴的
[python]
a = int(input('input a number:'))
item = {'a&lt;=10': 'a*(1+0.1)', 'a&gt;10 and a&lt;=20': '10*(1+0.1)+(a-10)*(1+0.075)', 'a&gt;20 and a&lt;40': '(a-20)*(1+0.05)','40&lt;a&lt;=60': '(a-40)*(1+0.03)', '60&lt;a&lt;=100': '(a-60)*(1+0.15)', 'a&gt;100': '(a-100)*(1+0.01)'}
for i in item.keys():
    if eval(i):
        print(eval(item[i]))
[/python]