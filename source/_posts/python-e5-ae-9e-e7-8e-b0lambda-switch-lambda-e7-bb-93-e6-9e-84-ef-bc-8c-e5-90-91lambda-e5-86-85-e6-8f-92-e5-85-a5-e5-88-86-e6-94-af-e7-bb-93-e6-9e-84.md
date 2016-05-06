---
title: python实现lambda-switch-lambda结构，向lambda内插入分支结构
tags:
  - python
id: 116
categories:
  - python
date: 2015-04-10 20:44:09
---

python没有switch的问题由来已久，大家也用各种方式实现了switch语句。最常用的便是利用字典来实现。
而通过将单行带default功能的switch语句嵌入lambda中，可以实现根据key值不同，执行不同的分支代码的功能。

[python]
# 1.这是这次的实验数据，目标是根据前面的符号对后面的数据进行操作，问号或其他无法识别的符号则视为直接返回该数字，分别执行 2*2, 3**3, -4, 5,得到结果应该是[4, 27, -4, 5]
items = [('*', 2), ('**', 3), ('-', 4), ('?', 5)]

# 2.从简单的做起首先用map遍历item生成['*', '**', '-', '?']的结果「我更喜欢列表解析，但是为了展示lambda-switch-lambda还是用map吧」。
items = list(map(lambda x: x[0], items))

# 3.然后加入带default功能的switch实现，得到将list转为['q', 'w', 'e', 'r']
items = list(map(lambda x: {'*': 'q', '**': 'w', '-': 'e'}.get(x[0], 'r'), items))

# 4.最后将switch中的元素换成lambda并且在get之后执行，即可分别执行乘，幂，负和保持不变等分支，得到[4, 27, -4, 5]
items = list(map(lambda x: {'*': lambda y: y * y, '**': lambda y: y ** y, '-': lambda y: -y}.get(x[0], lambda y:y)(x[1]), items))

# 5.这是另外一种实现方式，相比上一种多加了一个dict可以对key进行一次转化，合并同类作用的key，实现switch中多个case公用一个代码块的功能。
# 下面是把'*'和'**'都视为乘，可以得到[4, 9, -4, 5]
items = list(map(lambda x: {1: lambda y: y * y, 2: lambda y: y ** y, 3: lambda y: -y, None: lambda y: y}[{'*': 1, '**': 1, '-': 3}.get(x[0])](x[1]), items))

# 6.这是将第5条加入的dict改为通过len对key进行预处理，以此类推，在主dict后的[]中可以进行各种复杂操作，甚至再嵌入lambda，以实现更复杂的功能。
# 下面是根据key的长度，长度为1的执行乘操作，长度为2的执行幂操作，得到结果为[4, 27, 16, 25]
items = list(map(lambda x: {1: lambda y: y * y, 2: lambda y: y ** y, 3: lambda y: -y, None: lambda y: y}[len(x[0])](x[1]), items))
[/python]

利用该技巧，可以实现在lambda中加入分支语句，一定程度的改善了python中lambda的可用性。