最近在做小程序的时候遇到了一个列表底部的排版问题，简单地需求描述一下：就是不管列表内容多还是少，footer会一直显示在页面的最下面。类似这样：

内容少的时候：
![内容少的时候](https://s1.ax2x.com/2018/02/27/arAA6.png)

内容多的时候：
![artwp.png](https://s1.ax2x.com/2018/02/27/artwp.png)

当在实现的时候，因为要把footer定位到底部，通常会使用绝对定位，这时候当内容少的时候会出现这种情况：
![ar1uB.png](https://s1.ax2x.com/2018/02/27/ar1uB.png)

如果换成fixed定位，当内容多的时候则会出现下面的问题：
![arKFJ.png](https://s1.ax2x.com/2018/02/27/arKFJ.png)

于是我上网查了一下发现了这篇文章，非常有用就翻译了mark下来。英文好的可以直接看[原文](http://matthewjamestaylor.com/blog/keeping-footers-at-the-bottom-of-the-page
)。

# 怎么才能让footer一直呆在页面的底部

![](http://matthewjamestaylor.com/img/illustrations/large/keeping-footers-at-the-bottom-of-the-page.jpg)

如果一个页面所包含的内容很少的时候，footer通常会显示在页面的中部并在下面留下好大一块空白，这常常令人觉得非常难看，特别是在大屏幕上。这时前端工程师一般会被要求把footer弄下去，但明显这并不是一时半刻就可以搞掂的。

![](http://matthewjamestaylor.com/img/post/bottom-footer-the-problem.gif)

之前我曾尝试用纯css来代替用表格布局来让footer停留在底部，但是发现无法实现。经过了几年的实践，现在我终于找出了一个整洁的方法来实现它。这个方法100%使用Css并且可以正常地显示在主流的浏览器上。此外它还优雅地降级到旧浏览器也可以正常地显示，所以这个方法是非常安全的。

[Demo地址：](http://jsrun.net/YDqKp)

### 主要特性

- 可以在所有现代的，标准的浏览器上兼容运行

> 兼容浏览器：Firefox (Mac & PC), Safari (Mac & PC), Internet Explorer 7, 6 & 5.5, Opera and Netscape 8

- 优雅地降级到旧的浏览器

> 让footer在旧的、非标准的浏览器可以像预期地置于底部。如果用户用的是一个过时的浏览，那我们也没有办法，我们能做的只是提醒他们去更新一个性能更好的，可以支持更多特性的浏览器。

- footer在长内容页面也能保持在底部

> 在一个有很多内容的页面中，footer会保留在页面的最底部。就像正常的网站一样，当你往下滚动到页面底部的时候footer才会出现。这意味着footer不会像fixed那样一直占用着宝贵的阅读空间

- 100%的css代码并且不有使用hack

> 这个demo是使用纯css代码实现并且没有使用任何的hack特性

- 没有使用javascript

> 因为是纯Css所以没有使用JavaScript

- 兼容iphone

> 这个方法也可以在iphone、ipod touch 里的 Safari 浏览器上正常显示

- 开源代码

> 你只要在我的Demo页上简单地下载源代码就可以使用

### 有一个限制

你必须在充当footer的div上把height属性设置为除auto外的值。你可以随意地给一个值，但确保值是指定的并且单位是px或者ems。这并不是一个很大的限制，但却是很让这个方法正常工作的关键部分。

如果你的footer有很多的文字，一个好的解决方法是给底部更多的空间从而让footer能够包含这些文字。这是专门为了一些人会默认地在浏览器设置大字体而做的。另外也可以把footer的单位设置为em，这样footer的大小就会随着字体的大小而适应变化。如果你只有图片在你的footer里，你可以什么都不用担心，放心去把你的footer高度设置为像素值。

### 怎样实现的？

其实这并不复杂。这里把它分成两个部分，HTML 和 CSS。

**HTML 结构**

```JavaScript
<div id="container">
   <div id="header"></div>
   <div id="body"></div>
   <div id="footer"></div>
</div>
```

这里只需要有4个divs就可以实现。第一个容器div会包含所以东西。里面可以包含3个或者更多的div：header, body, footer等等。就这样，所以神奇的地方都在CSS里。

### CSS部分

```CSS
html,
body {
   margin:0;
   padding:0;
   height:100%;
}
#container {
   min-height:100%;
   position:relative;
}
#header {
   background:#ff0;
   padding:10px;
}
#body {
   padding:10px;
   padding-bottom:60px;   /* Height of the footer */
}
#footer {
   position:absolute;
   bottom:0;
   width:100%;
   height:60px;   /* Height of the footer */
   background:#6cf;
}
```

为了可以兼容IE6和IE 5.5，需要写另外一个简单的CSS规则

```CSS
#container {
   height:100%;
}
```


#### html 和 body 标签

html 和 body 标签的height属性一定要设置为100%，这样才能例我们接下来给container div 设置的百分值有效。同时也要去除html,body的margin和padding值，这样就没有多余的空间在页面的周围。

#### conainter div

在container div 必须要设置一个min-heihgt并设置为100%, 这样才能在页面很少内容的情况下确保让footer保留在屏幕的底部。有很多老的浏览器不支持`min-height`属性，有很关于这个问题的javascript或其它解决方法，不过这都超出了本文章的讨论范围。container div 还要设置一个position: relative 规则，这可以使之后的元素根据它来定位。

#### header div

就像普通的header一样，你可以给它任何的颜色或大小。并不影响footer。

#### body div

就像普通的body一样，不过有一个重要的东西就是它必须有一个和footer一样高度或者稍微高一点的内底边距（padding-bottom）。你也可以用底边框（border-bottom）代替，但是不能用外边距（margin）。

#### footer div

footer会有自己的一个高度。而且div会进行绝对定位并置于底部（bottom: 0），这样footer就会在container div的底部。当页面内容很少的时候，因为设置了`min-height: 100%`，container div的高度会和浏览器的视口（viewport）高度一样，并且footer会紧贴着屏幕的底部。当页面内容变多的时候，container div将会向视口的底部廷伸，footer仍然会在container div的底部，不过你需要滚动到页面的底部才能看到它。

#### IE 6 & IE 5.5 CSS

旧版本的IE浏览器不认识min-height属性，不过幸好，正常的height属性和min-height在旧IE上的显示是一样的，它也会把100%视口的高度铺满，如果内容变多它也会相应地覆盖很远的地方。我们使用IE条件注释来让这个CSS规则只对IE浏览器起效。你可以去看Demo的源码来了解它是怎么样实现的。

就这些代码你就可以把footer弄下去。希望这对你会有帮助（@_@)

