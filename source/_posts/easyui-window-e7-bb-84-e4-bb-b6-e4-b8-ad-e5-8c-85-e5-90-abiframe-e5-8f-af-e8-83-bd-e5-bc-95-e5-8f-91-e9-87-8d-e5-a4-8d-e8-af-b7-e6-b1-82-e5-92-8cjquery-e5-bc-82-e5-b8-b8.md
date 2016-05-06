---
title: EasyUI-window组件中包含iframe可能引发重复请求和jquery异常
id: 67
categories:
  - 未分类
date: 2015-01-06 18:33:47
tags:
---

准备工作：在A页面中引入jquery，下面放一个easyui的window，在window中包含一个iframe，src指向B页面。B页面只需要引入jquery即可。
[html]
 &lt;div id=&quot;win&quot; class=&quot;easyui-window&quot;&gt;
	&lt;iframe src='……'&gt;&lt;/iframe&gt;
&lt;/div&gt;
[/html]

我分别用Firefox Dev Edition 36.0a2,chrome 39.0.2171.95 m,IE 11.0.15进行了测试。其中ff和chrome正常，ie在jquery初始化过程中发生异常，无法完成初始化，子页面中无法调用jquery。
对dom树进行分析发现包含iframe标签的div已经被移动到新的位置上，因此怀疑easyui在初始化window的过程中对dom树的操作导致iframe初始化异常。于是查看网络请求，发现在ff中对B页面发起了两次请求，并且全部完成。chrome中对B页面发起了三次请求但其中两次被取消，ie中对B页面发起三次请求并且全部成功。可见easyui控件在初始化的过程中是会导致内部的iframe重复加载。
解决方案是将iframe中的src改为data-src，然后在easyui初始化完成后调用$('#frame').attr('src', $('#frame').data('src'))来实现延迟加载即可避免子页面出错。