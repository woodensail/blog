---
title: easyui较低版本中validType无法接收对象参数
id: 84
categories:
  - 未分类
date: 2015-03-11 17:34:08
tags:
---

validatebox即验证框，easyui中所有带验证功能的输入控件都是由此控件派生而成的。验证规则是通过使用 required 和 validType 特性来定义的。其中前者值为true或false，表示是否允许空值。后者则指定一条或多条验证规则，并给出对应验证规则所需要的自定义参数。
validatebox共有三种方式指定validType，分别为字符串，字符串数组和object。官网的介绍和例子如下：
> Defines the field valid type, such as email, url, etc. Possible values are:> 
> 1) a valid type string, apply a single validate rule.> 
> 2) a valid type array, apply multiple validate rules. The multiple validate rules on a field are available since version 1.3.2.> 
> 3) a key/value pairs, the key is the validing type name, the value is an array consisting validating parameters.

[html]
&lt;input class=&quot;easyui-validatebox&quot; data-options=&quot;required:true,validType:'url'&quot;&gt;
&lt;input class=&quot;easyui-validatebox&quot; data-options=&quot;
	required:true,
	validType:['email','length[0,20]']
&quot;&gt;
&lt;input class=&quot;easyui-validatebox&quot; data-options=&quot;
	required:true,
	validType:{
		length:[10,30],
		remote:['http://.../action.do','paramName']
	}
&quot;&gt;
[/html]

其中，第三种方式较前两种有极大的改进，前两种方式只能以字符串方式传递自定义参数，即**[10,30]**部分。而第三种方式可以在参数数组中传递任意对象，包括function对象。因而可以轻松地完成回调而不需要通过toSource或直接以字符串形式写function。同时，可以实现闭包的功能。相较前两种灵活性要高得多。
但是，第三种方式在1.3.5版本中是无效的。我测试了在js中指定validType以及在data-options中指定validType，结果都是无效。接下来查源代码，发现在源码中对validType类型进行判断，如果是string则按第一种处理，反之按第二种处理，根本没有第三种情况。

[javascript]
if (_19.validType) {
    if (typeof _19.validType == &quot;string&quot;) {
        if (!_1c(_19.validType)) {
            return false;
        }
    } else {
        for (var i = 0; i &lt; _19.validType.length; i++) {
            if (!_1c(_19.validType[i])) {
                return false;
            }
        }
    }
}
[/javascript]

<!--more-->
补充，本来以为是bug或者官方忘了实现。后来我尝试去easyui官网下了最新的1.4.2版。然后发现，第三种是可以用的。看来这个功能也是后期版本新实现的，而官方忘了在文档上标注……
新版本中的代码如下：
[javascript]
if ($.isArray(opts.validType)) {
    for (var i = 0; i &lt; opts.validType.length; i++) {
        if (!_45e(opts.validType[i])) {
            return false;
        }
    }
} else {
    if (typeof opts.validType == &quot;string&quot;) {
        if (!_45e(opts.validType)) {
            return false;
        }
    } else {
        for (var _465 in opts.validType) {
            var _466 = opts.validType[_465];
            if (!_45e(_465, _466)) {
                return false;
            }
        }
    }
}
[/javascript]