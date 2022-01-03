### 从overflow消除浮动原理

在项目中我们经常会用到overflow来消除浮动，但是我相信有很多人不知道它是根据什么来达到消除浮动的目的的，下面来讨论一下。

#### overflow属性

overflow有以下几个属性：

- visible: 不对超出的内容作处理；
- hidden: 剪切超出的内容；
- scroll: 不管内容是否超出，始终显示滚动条；
- auto: 超出的内容以拉滚动条方式显示；

#### overflow 消除浮动原理

先看下面一个例子：

```JavaScript
<style>
.box {
    overflow: hidden;
}
.item{
    float: left;
    width: 100px;
    height: 100px;
    background-color: pink;
}
</style>
<div class="box">
    <div class="item">hello overflow</div>
</div>
```

因为.item元素设置了浮动从而脱离了正常的文档流，所以导致了父元素无法自适应子元素的高度，造成**浮动塌陷**现象，所以给父元素添加一个Overflow属性来消除浮动。

那么overflow到底做了什么？

#### overflow的秘密

overflow 是对超出内容做处理，如果一个元素没有设置高，那么就将盒子里所有元素加起来的高度作为父盒子的高度。

但其实这又会导致一个现象，就是把所以子元素的高度加起来作为父元素的高度，那么父元素内容永远都不可能被超出。也就是说只要设置了overflow属性，就可以让没有设置高度的父元素重新计算高度。

#### 总结

所以其实也不能直接说用overflow来清除浮动，它只不过是让父元素根据子元素来重新计算了一下高度，解决了**浮动塌陷**的问题，和clear属性还是不一样的。