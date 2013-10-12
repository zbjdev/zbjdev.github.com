---
layout: post
date: 2013-10-12 15:58:31 +0800
title: jQuery-AJAX-post-null-value
---
昨天下午QQ上收到如下消息：

<blockquote>
<p>
Date:2013-10-11
<p>
水  17:00:34
<p>
兄弟伙，问一哈也。
<p>jquery 1.8.3 ajax的时候，如果我postdata里有字段是""或null，是不是会被自动转为空
</blockquote>

询问jQuery 1.8.3 版本的ajax方法会怎样对待postdata字段里面的“”或null字段，是否会被自动转为空，实践是检验真理的唯一标准，我们随便打开一个有jQuery的页面，在console里输入：

```js
	$.ajax('/get/',{type:"post",data:{a:"",b:null,c:123,d:"abs"}})
```

然后观察Network面板即可知道答案。

更进一步的做法，是深入理解jQuery的 [相应源码](http://james.padolsey.com/jquery/#v=1.8.3&fn=jQuery.param):

```js

	function (a, traditional) {
    var prefix, s = [],
        add = function (key, value) {
        // If value is a function, invoke it and return its value
        value = jQuery.isFunction(value) ? value() : (value == null ? "" : value);
        s[s.length] = encodeURIComponent(key) + "=" + encodeURIComponent(value);
    };

    // Set traditional to true for jQuery <= 1.3.2 behavior.
    if (traditional === undefined) {
        traditional = jQuery.ajaxSettings && jQuery.ajaxSettings.traditional;
    }

    // If an array was passed in, assume that it is an array of form elements.
    if (jQuery.isArray(a) || (a.jquery && !jQuery.isPlainObject(a))) {
        // Serialize the form elements
        jQuery.each(a, function () {
            add(this.name, this.value);
        });

    } else {
        // If traditional, encode the "old" way (the way 1.3.2 or older
        // did it), otherwise encode params recursively.
        for (prefix in a) {
            buildParams(prefix, a[prefix], traditional, add);
        }
    }

    // Return the resulting serialization
    return s.join("&").replace(r20, "+");
	}

```

可以了解到jQuery在发送数据之前对数据进行了怎样的处理，进而就可以毫无悬念地知道结果了。