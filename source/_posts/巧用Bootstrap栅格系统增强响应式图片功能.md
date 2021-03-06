---
title: 巧用Bootstrap栅格系统增强响应式图片功能
date: 2018-12-01 13:19:39
categories:
- 前端
tags:
- bootstrap
---
在Bootstrap中，通过为图片添加 `.img-responsive` 类可以让图片支持响应式布局。然而这么做有时会让图片出现在移动端显示正常，桌面端却缩放过大的情况。其实，利用栅格系统可以解决这个问题。

<!--more-->

## 问题

让我们先来看看如果只设置 `.img-responsive` 类会出现什么异常。

这是在移动端显示的效果，假定它是我们所需的：

{% asset_img img 1.png 预期效果 %}

到了桌面端，就会变成这样：

{% asset_img img 2.png 太大了！ %}

问题显而易见：在桌面端图片会放缩到原来的大小，而这导致了显示异常。然而，如果为了顾及桌面端而直接减小图片尺寸，移动端的显示又会受到影响。网上的资料大多是让我们利用JS做UA判定，可是既然用着Bootstrap，何不利用栅格系统达到相同的效果呢？

## 解决

将图片包进一个设置了 `.col-md-*` 类的容器中，像这样：

```html
<div class="row">
    <div class="col-md-2">
        <img src="/1s.png" class="img-responsive">
    </div>
</div>
```

然后桌面端显示效果就和移动端相似了，到移动端上 `.col-md-*` 类导致的缩放效果会自动消失。桌面端效果：

{% asset_img img 3.png 解决 %}

当然，这样做似乎有点土味，诸君若有更好的方法，欢迎在评论区交流。