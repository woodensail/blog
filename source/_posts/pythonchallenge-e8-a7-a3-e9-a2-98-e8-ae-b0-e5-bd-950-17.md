---
title: PythonChallenge解题记录(0-17)
tags:
  - python
  - PythonChallenge
  - 练习
id: 56
categories:
  - python
date: 2014-12-24 10:15:11
---

群里的每日一题计划似乎进行了没几次就被群主遗忘了，我在闲极无聊之下找到了PythonChallenge。pc的玩法基本上就是根据每一关的提示找出下一关的密码，和闯关游戏类似。不过pc的特点是解析下一关地址的过程极其复杂，无法人工完成，只能借助计算机编程实现。pc主要涉及了网络爬虫，图像处理和文件操作，虽说没有涉及python高级技巧，但是对于python的常用功能也算是基本都覆盖到了，因此非常适合新手在缺少练手机会是拿来熟练python。最后，我的解题过程基本上市先看网上的攻略找到题目中谜题的含义，然后自己编写代码来解题，我建议大家尽量也采用这种方法，可以减少在谜面上浪费的脑细胞。

**0**
http://www.pythonchallenge.com/pc/def/0.html
根据提示2^38得到274877906944
[python]
2 ** 38
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/274877906944.html

**1**
http://www.pythonchallenge.com/pc/def/map.html
页面包含一段乱码：g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj. 
文字提示：everybody thinks twice before solving this.
图片提示：k->m,o->q,e->g
根据图片提示得出是凯撒密码，将字母后移两位得到"i hope you didnt translate it by hand. thats what computers are for. doing it in by hand is inefficient and that's why this text is so long. using string. maketrans() is recommended. now apply on the url."
用同样的方式处理url"map"得到"ocr"
此处使用正则表达式匹配所有字母以避免影响到其他字符，匹配成功后使用lambda表达式完成字符替换
[python]
 re.compile('[a-z]').sub(lambda m: chr((ord(m.group(0)) - 97 + 2) % 26 + 97), hint)
 re.compile('[a-z]').sub(lambda m: chr((ord(m.group(0)) - 97 + 2) % 26 + 97), 'map')
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/ocr.html

**2**
http://www.pythonchallenge.com/pc/def/ocr.html
根据提示在源代码中找到需要统计的字符串，统计完成后获得出现最少的次数min_times，筛选出所有出现次数为mint_times的字符拼接后得到equality
[python]
for i in level2_str:
    if None == count.get(i):
        count[i] = 1
        char.append(i)
    else:
        count[i] += 1
min_times = min(count.items(), key=lambda x: x[1])[1]
char = filter(lambda x: min_times == count[x], char)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/equality.html

**3**
http://www.pythonchallenge.com/pc/def/equality.html
老样子在源代码中找到字符串，然后正则匹配出左右刚好各有三个大写字母的小写字母，最后拼接得到linkedlist
[python]
re.compile('[a-z][A-Z]{3}([a-z])[A-Z]{3}[a-z]').findall(level3_str)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/linkedlist.html

**4**
http://www.pythonchallenge.com/pc/def/linkedlist.php
根据页面提示打开下一个页面，只不过这关页面太多，必须要借助爬虫才行。
需要注意的是这关有两处陷阱，第一处提示"Yes. Divide by two and keep going."，需要将之前的数字直接除二得到下一个页面。第二处在中间有一个假数字，会导向一个错误页面。
[python]
pattern = '(?&lt;=[^0-9])\d+$' pattern = '(?&lt;=[^0-9])\d+$'
while 1:
        page = request.urlopen(base + nothing).read().decode()
        match = re.search(pattern, page)
        if not match:
            if 'Yes. Divide by two and keep going.' == page:
                nothing = str(int(nothing) / 2)
            else:
                break
        else:
            nothing = match.group(0)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/peak.html

**5**
http://www.pythonchallenge.com/pc/def/peak.html
这一关涉及到利用pickle模块进行反序列化，爬取banner.p，然后反序列化可以得到类似[[(' ', 95)], [(' ', 14), ('#', 5), (' ', 70), ('#', 5), (' ', 1)]……]的数据，列表解析后输出可以得到用井号和空格拼成的channel单词
[python]
banner = pickle.load(request.urlopen('http://www.pythonchallenge.com/pc/def/banner.p'))
return ''.join(''.join([pair[0] * pair[1] for pair in line]) + '\n' for line in banner)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/channel.html

**6**
http://www.pythonchallenge.com/pc/def/channel.html
这关和第4关类似，也是循环读取，只不过这回变成了读取压缩文件。按第四关的方法读到最后提示需要压缩文件的注释，于是把所有注释拼接起来就可以得到类似上一关的字符画，内容为hockey。进入hockey.html后提示"it's in the air. look at the letters. "，观察组成字符画所用的字母分别是o,x,y,g,e,n。于是输入oxygen.html，成功到达下一关。
[python]
# 只展示与第4关不同的部分
channel = zipfile.ZipFile('channel.zip')
comments.append(channel.getinfo(nothing+'.txt').comment.decode())
channel.read(nothing+'.txt').decode()
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/oxygen.html

**7**
这一关用到了图像处理，需要借助pillow库完成。
图片中央有一条灰阶，读取灰阶中每一格的RGB任一通道值，得到数字，转化为字符串后可以得到下一关的提示。再用正则将提示中的数字取出后再次转化就可以得到下一关的地址integrity。
http://www.pythonchallenge.com/pc/def/oxygen.html
[python]
img = Image.open('oxygen.png')
hint = ''.join([chr(img.getpixel((i, 50))[0]) for i in range(0, 609, 7)])
return ''.join([chr(int(i)) for i in re.findall('\d+', hint)])
[/python]
下一关地址：http://www.pythonchallenge.com/pc/def/integrity.html

**8**
这一关是用bz2解压缩，在源代码中有两个字符串，分别解压后得到'huge'和'file'，点击蜜蜂提示输入账号密码，将量单词分别输入跳转至下一个页面。
http://www.pythonchallenge.com/pc/def/integrity.html
[python]
bz2.decompress(un).decode()
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/good.html

**9**
这题搞得跟脑筋急转弯似的，但仍然是图像处理。源代码中有两个数组first和second，分别将其中的数字两两一组做为坐标在图上画点。完成后可以得到两幅画，分别是牛的侧身和牛头，所以答案是bull。
http://www.pythonchallenge.com/pc/return/good.html
[python]
im = Image.new(&quot;RGB&quot;, (500, 500))
    for i in range(0, len(level9_first), 2):
        im.putpixel((level9_first[i], level9_first[i + 1]), (255, 255, 255))
    im.save('level9_1.png')
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/bull.html

**10**
找规律的脑筋急转弯：
1 = 1个1 = 11
11 = 2个1 = 21
21 = 1个2加1个1 = 1211
依次类推。
我采用正则表达式的方式对数字进行分组，如将111221拆分为[111,22,1]。然后用列表解析的方式生成下一个数，[111,22,1]将被解析为[31,22,11],最后用join合并后进行下一轮处理。最后得到第31个数长度为5808
http://www.pythonchallenge.com/pc/return/bull.html
[python]
num = '1'
for i in range(0, 30):
num = ''.join([str(len(x[1]) + 1) + x[0] for x in re.findall(r'(\d)(\1*)', num)])
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/5808.html

**11**
还是图像处理，这次是将原图中所有xy坐标都为奇数和都为偶数的点取出生成单独的图片，生成完后可以明显的看到右上角有evil字样。
http://www.pythonchallenge.com/pc/return/5808.html
[python]
im = Image.open('cave.jpg')
x, y = im.size
im_new = Image.new(im.mode, (int(x / 2), int(y / 2)))
for i in range(0, x, 2):
    for j in range(0, y, 2):
        im_new.putpixel((int(i / 2), int(j / 2)), im.getpixel((i, j)))
im_new.save('level11_1.jpg')
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/evil.html

**12**
又是一道脑筋急转弯的题。第一页中图名字是evil1.jpg，修改为evil2.jpg后看到提示将后缀名修改为gfx。修改完后成功下载evil2.gfx文件。另外根据evil1中分牌的提示可知需要将文件按分牌方式分成五份。我使用的方式是先将文件全部读出然后列表解析生成5个子bytes，最后写入新文件。写入完成后看图可知下一关地址为disproportional
lenge.com/pc/return/evil.html
[python]
source = bytearray(file.read())
s1 = bytes([source[i] for i in range(0, len(source), 5)])
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/disproportional.html

**13**
页面提示"phone that evil"于是在图片中的电话上按键，按到e所在的5键时跳转到了phonebook.php，内容是一个报错的xml。查了一下这里是用到了xml远程调用，需要导入xmlrpc，用xmlrpc连接上phonebook.php后调用listMethods发现有一个phone方法。于是调用phone('evil')，提示"He is not the evil"。再上网查了一下发现上一关中evil4.jpg有提示"Bert is evil"，于是phone('Bert')，返回555-ITALY，下一关为italy
http://www.pythonchallenge.com/pc/return/disproportional.html
[python]
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/italy.html

**14**
根据图片和源代码中的"100*100 = (100+99+99+98) + (..."提示，可知需要将下方10000*1的图从外向里盘绕形成一个100*100的新图片。按要求生成图片后结果是一只猫，进入cat.html提示"and its name is uzi. you'll hear from him later."。下一关地址为uzi
http://www.pythonchallenge.com/pc/return/italy.html
[python]
im = Image.open('pc/wire.png')
im_new = Image.new(im.mode, (100, 100))
count = 0
for i in range(0, 50):
    jump = 99 - 2 * i
    for j in range(i, 99 - i):
        im_new.putpixel((j, i), im.getpixel((count, 0)))
        im_new.putpixel((99 - i, j), im.getpixel((count + jump, 0)))
        im_new.putpixel((99 - j, 99 - i), im.getpixel((count + 2 * jump, 0)))
        im_new.putpixel((i, 99 - j), im.getpixel((count + 3 * jump, 0)))
        count += 1
    count += jump * 3
im_new.save(&quot;pc/level14.jpg&quot;)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/uzi.html

**15**
日历显示为1xx6年1月26日周一，且右下角显示二月有29天为闰年，找出所有符合该条件的日期，分别为1176，1356，1576，1756，1976年的1月26日。然后根据源文件中的提示"he ain't the youngest, he is the second. buy flowers for tomorrow"。5个年份中第二靠后的是1756，搜索1756年1月27日为莫扎特的生日，符合所有提示。所以，下一关地址为mozart
http://www.pythonchallenge.com/pc/return/uzi.html
[python]
for year in range(1016, 2000, 20):
    t = date(year, 1, 26)
    if (t+timedelta(34)).month == 2 and t.weekday() == 0:
        print(t.isoformat())
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/mozart.html

**16**
依旧是图像处理，干脆改名叫image challenge得了……这题是根据图片中的红色线段对齐每一行像素，对齐完后得到的图中有romance字样。
http://www.pythonchallenge.com/pc/return/mozart.html
[python]
for i in range(0, 480):
    flag = 0
    for j in range(0, 640):
        if 195 == im.getpixel((j, i)):
            flag += 1
        if 5 == flag:
            flag = j
            break
    for j in range(0, 640):
        im_new.putpixel(((j - flag) % 640, i), im.getpixel((j, i)))
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/romance.html

**17**
这一关算是一个总结，涉及了前面关卡中出图像处理外的大部分内容，同时加入了cookie的处理，而且猜谜性质更加突出了。为了完成这一关在网上差了不少的资料，总算是勉强完成了。
这一关的主图是一堆饼干，暗示的是cookie，而左下角的小图是第四关的图片，于是进入第四关,发现第四关的cookie中有提示："info=you+should+have+followed+busynothing..."。尝试将用busynothing替换nothing，结果可行。接下来按照第四关的方式迭代，不过迭代的过程中需要收集cookie信息，每个网页都带有一字节的信息，将这些信息提取出来并且拼接得到一个bytes类型，开头是b'BZh91AY&SY',一看就是第8关的内容bz2，用bz2解压后得到'is it the 26th already? call his father and inform him that "the flowers are on their way". he'll understand.'。顺便一说，在'linkedlist.php?busynothing=19242'中cookie为'+',但实际上应该是空格，我不清楚到底是官方出错还是有什么其他的我没发现的暗示，但是我在这上面几乎浪费了一上午的时间，最后选择了当busynothing为19242时手动替换结果，才算是成功解压出信息。
言归正传，上面的提示可以看出跟之前的莫扎特有关，上网查莫扎特的父亲名字是'Leopold Mozart'。接下来用第13关的方法phone('Leopold'),结果是'no! i mean yes! but ../stuff/violin.php.'，进入该页面后有一张莫扎特的照片，页面标题是'it's me. what do you want?'，总算是找到莫扎特了……接下来通过cookie给莫扎特鲜花，也就是请求时在cookie中带上'the flowers are on their way'的信息，得到结果'oh well, don\'t you dare to forget the balloons.'其中balloons就是下一关的地址
http://www.pythonchallenge.com/pc/return/romance.html
[python]
page = ''
base = 'http://www.pythonchallenge.com/pc/def/linkedlist.php?busynothing='
nothing = '12345'
pattern = '(?&lt;=[^0-9])\d+$'
cookies = []
while 1:
    while 1:
        try:
            page = request.urlopen(base + nothing, timeout=1)
            break
        except:
            pass
    data = page.read().decode()
    match = re.search(pattern, data)
    cookie = page.info().get(&quot;Set-Cookie&quot;)
    cookies.append(parse.unquote_to_bytes(re.search(&quot;(?&lt;==)[^;]+(?=;)&quot;, cookie).group(0)))
    if '19242' == nothing:
        cookies.pop()
        cookies.append(b' ')
    if not match:
        if 'Yes. Divide by two and keep going.' == page:
            nothing = str(int(nothing) / 2)
        else:
            break
    else:
        nothing = match.group(0)

data = bz2.decompress(b''.join(cookies))
server = client.Server(&quot;http://www.pythonchallenge.com/pc/phonebook.php&quot;)
server.phone('Leopold')
http = urllib3.PoolManager().request('GET', 'http://www.pythonchallenge.com/pc/stuff/violin.php',
                                     headers={'Cookie': &quot;info=the flowers are on their way&quot;})
print(http.data)
[/python]
下一关地址：http://www.pythonchallenge.com/pc/return/balloons.html

PythonChallenge做到这里也算是告一段落了，后面还有十多关没做，不过接下来这段时间会很忙，所以只能找机会慢慢完成了。