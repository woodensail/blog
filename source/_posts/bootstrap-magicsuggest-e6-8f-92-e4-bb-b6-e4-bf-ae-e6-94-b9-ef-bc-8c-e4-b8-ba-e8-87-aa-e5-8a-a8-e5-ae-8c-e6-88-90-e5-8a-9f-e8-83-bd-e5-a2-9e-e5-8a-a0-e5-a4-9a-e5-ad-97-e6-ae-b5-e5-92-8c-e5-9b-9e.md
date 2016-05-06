---
title: bootstrap-MagicSuggest插件修改，为自动完成功能增加多字段和回调支持
tags:
  - bootstrap
  - javaScript
id: 111
categories:
  - JavaScript
date: 2015-04-08 16:15:04
---

MagicSuggest中的自动完成功能只支持单字段搜索，我增加了一个filter字段，并对_sortAndTrim方法进行修改，从而达成多字段以及通过回调实现自动完成功能。
实现时使用了lodash库，如果filter为数组，根据filter数组中所指定的字段进行筛选。否则视为函数，将目前输入内容q和一条记录所对应的对象obj作为参数执行filter。返回true则判断符合，反之不符合。
[javascript]
if(cfg.filter!==null) {
    if (_.isArray(cfg.filter)) {
        if(cfg.strictSuggest) {
            filtered = _.filter(data, function (obj) {
                return _.any(_.pick(obj, cfg.filter), function (v) {
                    return v.indexOf(q) === 0
                });
            });
        }else {
            filtered = _.filter(data, function (obj) {
                return _.any(_.pick(obj, cfg.filter), function (v) {
                    return v.indexOf(q) &gt; -1
                });
            });
        }
    } else {
        $.each(data, function (index, obj) {
            if (cfg.filter(q, obj)) {
                filtered.push(obj);
            }
        });
    }
}else{
    //原来的代码
}
[/javascript]