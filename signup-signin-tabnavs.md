# 知乎登录/注册页之标签切换动画
在知乎登录/注册页面，点击“登录”或“注册”可以分别切换到各自的输入表单，这个切换动画中有一个蓝色小条比较有意思，动图如下：

![效果图](./img/tabnavs-animation.gif)

具体实现包含以下几个部分:
 1. 底部小蓝条
 2. 小蓝条跟随标签
 3. 蓝条跟随动画

## 基本页面元素
html部分：
```html
<div class="tab-navs">
  <div class="navs-slider">
    <a href="#signup" class="active">注册</a>
    <a href="#signin">登录</a>
  </div>
</div>
```
CSS部分
```
.tab-navs {
  text-align: center
}

.tab-navs .navs-slider {
  display: inline-block
}

.tab-navs a {
  float: left;
  font-size: 18px;
  width: 4em;
  line-height: 35px;
  color: #000000;
  text-decoration: none;
}

.tab-navs a.active {
  color: #0f88eb
}
```
注意，在上述的css代码中：
 - 最外层的`div.tab-navs`是一个块级元素(block-level element)，默认宽度自动填满其父元素宽度；
 - `div.tab-navs`通过`text-align: center`控制子元素**居中对齐**显示
 - 中间层的`div.navs-slider`默认是一个块级元素(div)，但是通过`display: inline-block`修改为内联显示，但仍然保留块级元素的特性
 - `a`标签设置不显示下划线
 - `a`标签设置`float: left;`并设置宽度为`width: 4em;`，因为这里两个标签的文本都是两个字，所以加上间距设置宽度为**4em**

js部分
```
function onHashChanged() {
  var $arr = $(".tab-navs a");
  var hash = window.location.hash || $arr.filter(".active").attr("href");
  var $a = $arr.filter(function() {
    return $(this).attr("href") === hash
  });
  $a.addClass("active").siblings().removeClass("active");
}
$(window).on("hashchange", onHashChanged)
$(window).on("load", onHashChanged)
```
js代码中使用了**hashchange事件**监听url中的hash值的变化，并更新当前选中的a标签的css属性。

![效果图](./img/tabnavs-base.gif)

## 底部小蓝条
我最开始以为是通过给a标签添加border之类的方法来实现，看了知乎源码才发现是添加了一个`span`元素结合css来实现。

html代码中添加一个span元素：
```
<div class="tab-navs">
  <div class="navs-slider">
    <a href="#signup" class="active">注册</a>
    <a href="#signin">登录</a>
    <span class="navs-slider-bar"></span>
  </div>
</div>
```

设置该span元素的css属性：
```
.tab-navs .navs-slider .navs-slider-bar {
    position: absolute;
    left: 0;
    bottom: 0;
    margin: 0 .8em;
    width: 2.4em;
    height: 2px;
    background: #0f88eb;
}
```

### 问题1
添加span并设置css属性后，该span元素并没有出现在预定的位置：“注册”标签的下面，而是出现在了浏览器的最底部。相关解释见参考文档。

解决方法是给其父元素设置position属性：
```
.tab-navs .navs-slider {
    position: relative;
    display: inline-block
}
```

### 问题2
span元素出现在了“注册”标签的下面，但是并不居中显示在文字的下面，而是有点偏左。重新看一下该span元素的css属性：
```
.tab-navs .navs-slider .navs-slider-bar {
    position: absolute;
    left: 0;
    bottom: 0;
    margin: 0 .8em;
    width: 2.4em;
    height: 2px;
    background: #0f88eb;
}
```
其中，width的值是**2.4em**，margin的值是**0 .8em**(注意并不是0.8em，而是margin-top和margin-bottom都是0，margin-left和margin-right都是0.8em)。

这样，整体的span宽度就是0.8*2+2.4=4em，和a标签的宽度是一致的。但显示效果却不居中对齐，原因在于，`em`值的大小是相对于`font-size`的值，a标签的`font-size`是18px，span元素的`font-size`是默认值16px，所以同样是`4em`，长度却不同。

解决方法是把`font-size`的设置移到父元素中：
```
.tab-navs .navs-slider {
    font-size: 18px;
    position: relative;
    display: inline-block
}
```

## 小蓝条跟随标签
因为小蓝条是绝对定位的，且宽度和标签一致，那么当第二个标签处于active状态的时候，应该让小蓝条的位置设置到该标签下面，即：
```
    left: 4em
```

具体实现是结合js和css来处理的：
```
$(window).on("hashchange", function() {
  var $arr = $(".tab-navs a");
  var hash = window.location.hash || $arr.filter(".active").attr("href");
  var $a = $arr.filter(function() {
    return $(this).attr("href") === hash
  });
  $a.addClass("active").siblings().removeClass("active");
  $a.parent().attr("data-active-index", $a.index());
})
```

css部分：
```
.tab-navs .navs-slider[data-active-index="1"] .navs-slider-bar {
    left: 4em
}
```

## 蓝条跟随动画
跟随动画就是一个标准的css过渡效果：
```
  -webkit-transition: left .15s ease-in;
  transition: left .15s ease-in
```
三个参数分别表示，指定的css属性、过度效果时间、速度曲线。不过在本例中，span元素移动的范围有限，速度曲线几乎没有意义。

## 总结
登录/注册标签切换动画中使用到了以下知识点：
 - `display: inline-block`和`text-align: center`配合居中对齐
 - 通过`float`来支持行内元素设置宽度
 - 使用`em`配合`font-size`动态设置块级元素的宽度
 - 使用了hashchange事件监听url中的hash值的变化
 - css3的transition过渡效果

## 参考资料
 - [block，inline和inline-block概念和区别](http://www.cnblogs.com/KeithWang/p/3139517.html)
 - [关于水平居中及display:inline-block在ie6,7下的处理方法 ](http://blog.sina.com.cn/s/blog_4c8b519d010150jh.html)
 - [nline-block元素间间隙产生及去除详解](http://demo.doyoe.com/css/inline-block-space/)
 - [为什么行内元素设置float之后才能用width调整宽度？](http://bbs.csdn.net/topics/390045550#post-391278083)
 - [window.onhashchange](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onhashchange)
 - [CSS 中position:absolute的定位到底是相当于body，还是父级元素的问题](http://www.cnblogs.com/zhglhtt/articles/3265372.html)