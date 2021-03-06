---
layout: post
title: "frame与Autolayout（Masonry）的几点结论"
description: ""
category: UI
tags: [frame，autolayout，masonry]
---

#### 在做开发的时候，UI适配方面老代码都是使用`frame`做UI布局，新的代码却是用的`Autolayout`，我们主要是用的三方库`masonry`写约束完成布局。在这个过程中。我一直有个错误的认识，`Autolayout`不会改变`frame`。网上看到很多blog也这么认为。这么认为多是由于设置完约束之后，打印`frame`的值为`CGRectZero`或者一个错误值造成的。但是在我调试一段代码时，情况却超出了我原来（错误）的认识。为什么打印的`frame`完全正确？

#### 答案是设置约束后`frame`的更新并不是实时的，需要等到根据约束计算好`view`的位置和尺寸之后才会刷新`frame`。 `Autolayout`的本质是通过一套约束规则去确定一个`view`在`superview`中的位置和尺寸。既是位置和尺寸那不就是`frame`嘛。所以我通过文档和实验得出了以下几个结论：
- `Autolayout`是一套关于`view`位置和尺寸的约束规则
- `Autolayout`改变可以改变`view`的`frame`，但是`frame`的改变并不能改变`Autolayout`的约束规则（当`translatesAutoresizingMaskIntoConstraints = YES`时，会根据`autoresizingMask`的相应设置更新相应的约束，但是这很可能会***与既有的约束冲突***，iOS7很有可能crash。设置`translatesAutoresizingMaskIntoConstraints = NO`是推荐的做法，所以我们可以认为更改`frame`不会改变约束），约束应该通过约束更新的方式改变。想要在`Autolayout`改变后立即看到`frame`的变化，请调用以下代码来立即刷新UI的布局
{% highlight objective-c %}
- (void)setNeedsLayout;
- (void)layoutIfNeeded;
{% endhighlight %}

- `Autolayout`和`frame`能否混用，答案是一定程度上可以，牢记上面一条两者之间的相互影响关系。在约束布局的UI视图中，混用`frame`布局，一定要先调用上面的这段代码，这样就能得到`superView`和相关`view`准确的`frame`。但是修改`frame`不会修改约束使得混用的情况的变得复杂很容易出错，所以还是大众化的建议：最好不要混用。
- 在一些业务处理时，在约束布局后，需要获取`view`的宽高尺寸等信息，不用说，先调用上面的代码。这也可以说是一种混用的情况吧。
- 一些人认为在`viewDidLayoutSubviews` 和 `viewDidAppear` 中布局已经确定，可以获取实际的`frame`，诚然如此。这样也比每次调通上面的代码效率高。但是这只适用于进入页面后布局不在变化的情况，假如在当前页面一些操作触发了约束变化，那么这种做法就不适用了。上面这段代码虽然效率低，但是适用范围更大。
