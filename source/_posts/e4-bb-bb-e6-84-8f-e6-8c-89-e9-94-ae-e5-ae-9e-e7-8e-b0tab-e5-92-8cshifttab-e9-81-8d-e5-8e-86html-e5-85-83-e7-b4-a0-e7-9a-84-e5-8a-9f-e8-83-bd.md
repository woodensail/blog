---
title: 任意按键实现tab和shift+tab遍历html元素的功能
tags:
  - javaScript
  - jQuery
id: 103
categories:
  - JavaScript
date: 2015-04-01 15:49:49
---

##### 初始化：

首先遍历所有有tabindex的标签，普通标签直接将tabindex的值存入data-tabindex中。对于第三方库渲染的特殊控件，默认的tabindex是无效的，在default.init中可以配置针对的初始化方法，将tabindex值绑定到真正的标签的data-tabindex属性上。
预处理完毕后，选择所有含data-tabindex属性的标签并排序后放入tabList中。此后当事件被触发时，事件函数会利用闭包特性取得tabList。
最后完成事件绑定，绑定前先解绑一次，以防止重复绑定。
&nbsp;

##### 事件响应：

触发事件后依次通过判断keyCode是否在forwardKey和backwardKey中来决定是否需要后移或前移。如果需要执行时对于普通标签执行默认的方法，对于包含defaulte.execute中的class的特殊标签则先执行标签中的处理函数，当处理函数返回true时会继续执行默认方法，否则阻止默认方法。

##### 调用方式：

在html中设置tabindex或通过js动态设置tabindex后，执行$.extendTab()即可。
$.extendTab.default定义了特殊控件处理方式及需要响应的按键，可以通过$.extend修改
[javascript]
(function ($) {
    $.extendTab = function (i18nData) {
        $('[tabindex]').each(function (i, v) {
            var _this = $(v);
            var flag = true;
            $.each($.extendTab.default.init, function (key, value) {
                if (_this.hasClass(key)) {
                    value(_this);
                    flag = false;
                    return false;
                }
            });
            if (flag) {
                _this.attr('data-tabindex', _this.attr('tabindex'));
            }
        });
        var tabList = $('[data-tabindex]').sort(function (a, b) {
            return $(a).data('tabindex') &gt; $(b).data('tabindex')
        });
        tabList.off('keydown.extendTab');
        tabList.on('keydown.extendTab', function (e) {
            var target = $(e.currentTarget);
            if ($.extendTab.default.forwardKey.contains(e.keyCode)) {
                var flag = true;
                $.each($.extendTab.default.execute, function (key, value) {
                    if (target.hasClass(key)) {
                        flag = value(target);
                        return false;
                    }
                });
                if (flag) {
                    var _idx = tabList.index(this) + 1;
                    tabList.eq(_idx &gt;= (tabList.length) ? 0 : _idx).focus();
                    e.preventDefault();
                }
            } else if ($.extendTab.default.backwardKey.contains(e.keyCode)) {
                var flag = true;
                $.each($.extendTab.default.execute, function (key, value) {
                    if (target.hasClass(key)) {
                        flag = value(target);
                        return false;
                    }
                });
                if (flag) {
                    tabList.eq(tabList.index(this) - 1).focus();
                    e.preventDefault();
                }
            }
        });
    };

    $.extendTab.default = {
        init       : {
            'combo-f': function (jq) {
                jq.next().find('.combo-text').attr('data-tabindex', jq.attr('tabindex'));
            }
        },
        execute    : {
            'combo-text': function (jq) {
                clearTimeout(jq.parent().prev().data('combo').timer);
                return true;
            }
        },
        forwardKey : [39, 13],
        backwardKey: [37]
    }
})(jQuery);
[/javascript]
https://github.com/woodensail/extendTab