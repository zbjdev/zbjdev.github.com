---
layout: post
date: 2013-10-12 14:17:16 +0800
title: 移动页面默认放大的bug修复
---

###背景
今天早上接到无线部门求助：[移动注册页面](http://m.zhubajie.com/webApp/login-register.html)  在部分Android机型上默认会放大，需要用户手动缩小才能正常浏览，原作者时间精力有限，无暇顾及。

![bug截图](/assets/images/2013-10/20131012_142442.png "bug截图")

###排查

检查HTML代码发现头部有以下meta：

```css
	<meta content="width=device-width,initial-scale=0.5,user-scalable=yes,densitydpi=device-dpi" name="viewport"/>
```	
按理说都已经 `initial-scale=0.5` 了，默认应该缩小才对啊，怎么会反而放大了？这不科学啊！

进一步检查CSS代码，发现一段关键代码：

```css
	.loginPage,.registerPage{ 
	width: 100%;
	min-width:640px; 
	background: #e8e8e8;
	/*min-height:960px;	*/ 
	font-family: "microsoft yahei";position: relative;
	}
```

猜测是由于`min-width`影响了`initial-scale=0.5`的正常缩放，尝试注释掉`min-width`，页面比例当即恢复正常，但是页面中的部分区域文字换行，布局错乱

![bug截图](/assets/images/2013-10/Snip20131012_17.png "bug截图")

第三方登录按钮挤在了一起，甚至在部分机型下会换行

![bug截图](/assets/images/2013-10/Snip20131012_18.png "bug截图")

页面字体大小比例不协调

这个时候怎么办呢？如果只是字体大小还好办，调整字体大小就行了，第三方登录的图标大小是固定的，怎么办呢，这个时候我们脑海里灵光闪现，想起了在IE时代经常使用的`zoom` 属性，虽然大家平时都只`zoom:1`用来触发haslayout以修复各种bug，但是这个属性其实是可以缩放元素大小的，而且据说webkit内核浏览器也已经支持这个属性了，当即决定尝试一下，结果很悲催，不起作用。

好在我们是做的移动开发，而且这个页面是会内嵌到微信应用中去使用的，那么借助CSS3的解决办法就顺理成章啦：
针对我们的小屏幕，我们直接使用CSS3的transform进行缩放：

```css
/* 480 以内的小屏幕*/
@media screen and (max-width: 480px) {
    .loginFormIpt{font-size: 22px}
    .anyOther{font-size: 24px}
    .condition {font-size: 16px}
    .otherUser .cut span{font-size: 14px}
    .webUser {text-align: center;}
    .webUser span {
      -moz-transform:scale(0.5);
      -o-transform:scale(0.5);
      -ms-transform:scale(0.5);
      -webkit-transform:scale(0.5);
      transform:scale(0.5);
    }
  }
```

![修复后截图](/assets/images/2013-10/Snip20131012_19.png "修复后截图")

问题完满解决，提交无线部同事在各种机型上测试，过了一阵儿之后反馈说没问题了。

但是真正放到开发环境中去测试的时候却始终有问题，经过反复尝试，将CSS的引用地址临时指向我的开发版本就没问题，确认是移动开发同事服务器端的同步或缓存的问题。

---
团队交流后更新:

* 老谭提出可以使用background-size来解决这个问题，也是个不错的思路
* 这种第三方登录的icon比较扁平化，使用iconfont技术也能实现自由缩放

###总结
当页面CSS设置的 `min-width` 值超过 `device-width x initial-scale` 值时, 部分Android机型下`initial-scale` 将失效，iOS普遍正常。

