---
title: 提取并修改class文件中的字符串常量
id: 80
categories:
  - 未分类
date: 2015-02-07 15:37:51
tags:
---

这两天在网上看到某个汉化组在汉化jar包过程中遇到开发者加扰，将部分class采取超长命名以及相同名称不同大小写的方式，导致windows下无法解压缩且专用汉化工具无法支持的情况。

所以我就做了脚本，用于将指定jar包或zip文件里所有class文件中的字符串常量提取出来，以JSON格式存入txt文件中。修改该文件中字符串的值，然后在利用该脚本重新导入，即可完成对jar包中class文件字符串常量的修改。

&nbsp;

脚本通过zipfile库遍历jar包读取所有class文件并解析，class文件解析的原理在我之前转载的「Class文件内容及常量池」中。之后通过json库保存信息。

汉化完成后再将汉化信息以json格式读入，并根据文件名已经常量编号对原字符串常量进行替换。

&nbsp;

项目地址：[https://github.com/woodensail/ExtractString](https://github.com/woodensail/ExtractString)