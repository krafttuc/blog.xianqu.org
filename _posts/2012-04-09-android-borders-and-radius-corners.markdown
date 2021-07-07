---
layout: post
title: 'Android开发笔记——圆角和边框们'
date: 2012-04-09 13:48:56 +0000
categories: 小小技术
---

在做 Android 界面开发时，我们往往希望它尽可能优美，尽可能显得专业。于是你看了看其他应用，哇，好多边框和圆角啊。你是不是也想给自己的应用加上边框和圆角效果？呃……那怎么做呢？如果你是从 web 前端跑到 Android 来的，那么我想你一定想到了不下三种解决方案。如用图片替代，用 CSS3 定义，用 JS 画。在 Android 中，其实也有类似的用法，本文将简单介绍两种 Android 圆角和边框的实现。

![](/images/2012-04-twitter-android.jpg 'Twitter for Android的圆角和边框设计')

## 1 图片

在 Android 中，给一个控件（或 View）设置背景主要是通过`background:xxx`属性来完成。background 的参数一般来说是一个 drawable 资源。
drawable 可以是一张普通的图片，也可以是 9 patch 图片，还可以是一个 xml 文件。
给控件设置边框最简单的方式就是把 background 设置成你预先设计好的带圆角和边框的背景图。比如下面这张图：

![](/images/2012-04-android-layer-list-1.jpg)

但是，你很快会发现一个缺点：**灵活性很差**！是的，固定大小的图片很难根据控件里的内容而调整大小。它在被做出来的那天就已经被确定了！换句话说，你很难只用这一张图来应付拥有相同风格却大小各异的控件。为了给所有控件加上圆角和边框，你必须小心翼翼地计算他们的大小，然后一个一个得制作背景图片！天哪，这简直太愚蠢了。一旦遇到大小不定的控件，这方法就歇菜了。而且，大量的背景图片会让你的安装包迅速膨胀。呃……还有，你怎么应对拥有各式各样分辨率的 Android 设备呢？

所以，你需要……换个方法。

比较为大众采用一种解决方案是[NinePatch](http://developer.android.com/guide/developing/tools/draw9patch.html)。可以毫不夸张得说，9 patch 是 Android 中解决自适应问题的利器。介绍和使用你可以看看[这里](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch)还有[这里](http://radleymarx.com/blog/simple-guide-to-9-patch/)。

使用 9 patch 图片有很多好处，如减轻美工压力，减少 UI 代码量，减少内存使用……总结起来就是：省时省力，屌爆了。

所以在给圆角和边框时，你或许会这么做。

![](/images/2012-04-android-9-patch.jpg)

当然，9 patch 能做的是远远比这多，如做一个自适应的对话框什么的。

## 2 XML 定义

我想大多数程序员都喜欢用代码解决问题。原因如下：

1. 用代码更加 cool。
2. 我美工不行，我会说出去吗？

OK，好东西在这里。

### 2.1 基本的圆角、边框

Android 除了支持原始的图片资源外，比较棒的一点就是可以用 XML 文件定义一些简单的图形。这有点像 web 的 CSS，不过相比 CSS3，Android 的 xml 实现还没那么强大，例如，边框要么四周都有，要么四周都没有（我们将在后面讨论这事）。xml drawable 的传送门在[这里](http://developer.android.com/guide/topics/resources/drawable-resource.html)。

要画一个带灰色边框和圆角的图形很容易，在 drawable 资源目录下添加一个 xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- shape如果不声明形状则默认为正方形 -->
<shape xmlns:android="http://schemas.android.com/apk/res/android" >

    <corners android:radius="5.0dp" />
    <!-- 圆角，你也可以对不同的角设置不同的数值 -->

    <solid android:color="#FFFFFF" />
    <!-- 形状的填充色 -->

    <stroke
        android:width="1dp"
        android:color="#CCCCCC" />
    <!-- 边框宽度和颜色 -->

</shape>
```

在你需要用到这东西的地方如某个 View 下，设置 background 就行了。

### 2.2 “自由的边框“

当前版本的 Android SDK 并没有给 stroke 提供 bottom、left、right 之类的属性，也就是说你无法通过它来让长方形的边框少于 4 条。啊，真是太遗憾了。怎么办呢？有人想到了对[Layer List](http://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList) hack。 在 StackOverflow 上有不少这样的[把戏](http://stackoverflow.com/questions/1598119/is-there-an-easy-way-to-add-a-border-to-the-top-and-bottom-of-an-android-view)。

为了实现只有 left，right 和 top 边框，我们可以这么写：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item>
        <shape android:shape="rectangle" >
            <stroke
                android:width="1dp"
                android:color="@color/card_stroke" />
        </shape>
    </item>

    <item
        android:left="2dp"
        android:right="2dp"
        android:top="2dp">
        <!-- 在实际使用中我发现1dp达不到显示效果，而2dp正好可以显示边框 -->

        <shape android:shape="rectangle" >
            <solid android:color="@color/solid_white" />
        </shape>
    </item>
</layer-list>
```

原理差不多是这样：

![](/images/2012-04-android-layer-list-2.jpg)

诡异的是理论上只要偏移量只要 1dp 就能显示 1dp 宽带边框了，但我在 listview 里实验了一下发现不行，换成 2dp 方可。有同学能解释解释么？

如果要给图形加上圆角，只需要给每个 shape 加上

```xml
<corners
	android:topLeftRadius="5.0dip"
	android:topRightRadius="5.0dip" />
```

值得注意的是，两个 shape 的 radius 在设置的时候请确保前面的图层不会把后面的挡住。

## 3 小结

要在 Android 中实现圆角和边框，比较简单的方法：图片、XML 差不多就是这么用的啦。此外还有用 Java 代码调用 draw 方法画出来的，不过我没有研究过。
他们各有各的优点啦。用图片，能控制的东西更多，用代码修改起来比较另过。
最后要说的是两个方法的效率。在这个问题上，我留有疑问，没有做过专门的比较。但直观的感受是……好吧，没什么感受。
