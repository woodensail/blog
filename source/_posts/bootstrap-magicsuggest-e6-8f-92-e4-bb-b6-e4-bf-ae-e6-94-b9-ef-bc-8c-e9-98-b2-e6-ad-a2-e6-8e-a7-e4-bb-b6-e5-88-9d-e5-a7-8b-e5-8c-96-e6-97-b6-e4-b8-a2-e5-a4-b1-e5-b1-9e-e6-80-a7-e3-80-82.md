---
title: bootstrap-MagicSuggest插件修改，防止控件初始化时丢失属性。
tags:
  - bootstrap
  - javaScript
id: 113
categories:
  - JavaScript
date: 2015-04-08 17:26:10
---

与之前常用的easyui不同，MagicSuggest没有将原始标签设为hidden，并作为访问入口的方式。而是选择用新标签替换原始表签，同时将原标签的id和style复制过来，这样带来的缺点是，在对象初始化时，原标签所绑定的属性和事件会丢失。
下面就是对MagicSuggest进行修改，将原始标签所有属性复制入新标签中。此处没有处理事件，因为我暂时不需要。
第一处是遍历原标签属性，其中2,4行为原有，我增加了1,3行，将所有属性并入attrivutes中，而不是直接置入def，防止混淆。
第二处是通过extend方法将原来的属性和cfg.attributes合并后赋给新标签。
[javascript]
def.attributes = {};
$.each(this.attributes, function (i, att) {
    def.attributes[att.name] = att.value;
    def[att.name] = att.name === 'value' &amp;&amp; att.value !== '' ? JSON.parse(att.value) : att.value;
});

ms.container = $('&lt;div/&gt;', $.extend(true, {}, cfg.attributes, /* 原代码 */));
[/javascript]