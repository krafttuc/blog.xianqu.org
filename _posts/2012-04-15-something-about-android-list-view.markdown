---
layout: post
title: 'Android开发笔记——ListView的零零碎碎'
date: 2012-04-15 08:32:45 +0000
categories: 小小技术
---

在数据流型的移动应用中，列表在数据展示上作用很大。随便拿个微博或 SNS 应用，你就能发现自己接触的多是列表。列表承担的责任包括数据展示，对特定对象的快捷操作等。我在写 Android 作业是也收集了一些零零碎碎的东西，在此一并分享出来吧。

## 1 下拉刷新

下拉刷新目前已经是数据流 APP 的标配了。Android 没有原生的下拉刷新控件支持，但只要你想没有不可能。某老外放出了一个下拉刷新的实现代码。请[猛击这里](https://github.com/johannilsson/android-pulltorefresh)。还有人为他写了一个简单的[指南](http://sharedstate.net/archives/pull-to-refresh)。

## 2 Quick Action

Quick Action 或许可以称为快捷动作，意思是点击 list 的 item 后出现一个动作栏，提供快捷的操作。唔……其实和 context menu 差不多啦，不过么……这个看上去绚一点。

[教程在这里](http://www.londatiga.net/it/how-to-create-quickaction-dialog-in-android/)。

## 3 Android 控件大杂烩

一直很希望能找到一个介绍各种 Android UI pattern 的网站，现在[这里](http://www.androiduipatterns.com/p/android-ui-pattern-collection.html)有一个。要是再完整一点，整理好一点那就更完美了。

吾记得国人做过一个 iOS 的，有兴趣的同学去查查。

## 4 带圆角的 ListView

圆角流行很久了，给 ListView 加上圆角会很酷。例如下面这个样子。

![](/images/2012-04-android-rounded-corner-list-1.jpg)

实现方法有好几种：

### 4.1 给 ListView 整体设置一个带圆角的 background

在 XML 文件里你或许会这么做

```xml
android:background = "@drawable/bg_list"
```

`bg_list`便是你定义的可以自适应大小的圆角图。具体做法你可以参看我的[Android 开发笔记——圆角和边框们](http://blog.xianqu.org/2012/04/android-borders-and-radius-corners/)。

这是我们最容易想到的一种解决方案。在很多时候它能起到作用。

但是要应对上面的那种样子好像不太容易哦，怎么让 scroll 跑外面去？listview 是占整个屏幕的宽度还是要做一个 margin？

唔，想来想去，好像只能这么干。

![](/images/2012-04-android-rounded-corner-list-2.jpg)

等你实际运行一下你就发现——坑爹了。这个 listview 的 background 就和一个相框一样，而不是我们想要的那种：让整个 list 看上去像带圆角的卡片。此外，你的 scroll 也不是在屏幕的边缘，而是随 listview 过去了。当然，你可以在 listview 外面加一个 ScrollLayout，但请相信我，那样还是起不了什么大作用。

所以，这个解决方案适合什么情况呢？ 当你需要做**一个内部能滑动的带圆角的框体**。

### 4.2 给 ListView 设置 padding，给每一个条目分别设置 background

我曾逆向工程 Twitter for Android 想看看它们是怎么实现这样的卡片 ListView 的。我拿到了 XML 文件，我发现他们用了一个自定义的"CardView"，尼玛这得看 Java 源码。但反编译出来的结果甚是坑爹，我只能放弃。

后来我自己研究了一下，找了个替代方案。

基本思想是在 ListView 的 Adapter 的 getView()函数里，根据 position 动态设置每个 item 的 background。如，如果是第一个 item，就给他一个顶部是圆角的背景。如果是当中的 item，则给一个不带圆角的背景。如果是底部的 item，就给一个底部带圆角的背景。

实现代码片段如下：

```java
public class MyAdapter extends ArrayAdapter<String> {

    // other methods...

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        View view;
        if (convertView != null) {
            view = convertView;
            holder = (ViewHolder) view.getTag();
        } else {
            view = inflater.inflate(R.layout.list_item_view, parent, false);
            holder = new ViewHolder();
            holder.source = (TextView) view.findViewById(R.id.text);
            view.setTag(holder);
        }

        // 加载要显示的数据
        holder.text.setText(getItem(position));

        // 动态设置item的background
        if (position == 0) {
            holder.layout.setBackgroundResource(R.drawable.bg_list_row_top_selector);
        } else if(position == getCount() -1){
            holder.layout.setBackgroundResource(R.drawable.bg_list_row_bottom_selector);
        }else{
            holder.layout.setBackgroundResource(R.drawable.bg_list_row_middle_selector);
        }

        return view;

    }

    private static class ViewHolder {
        RelativeLayout layout;
        TextView text;          //比如我们的item里只有一个TextView
    }

}
```

每个 item 背景的 selector，如顶部 item 的背景

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/bg_list_row_top_pressed" android:state_pressed="true"/>
    <item android:drawable="@drawable/bg_list_row_top_focused" android:state_focused="true"/>
    <item android:drawable="@drawable/bg_list_row_top_focused" android:state_selected="true"/>
    <item android:drawable="@drawable/bg_list_row_top"/>
</selector>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >

    <item>
        <shape android:shape="rectangle" >
            <stroke
                android:width="1dp"
                android:color="#cccccc" />
            <corners
                android:topLeftRadius="10.0dip"
                android:topRightRadius="10.0dip" />
        </shape>
    </item>

    <item
        android:bottom="2dp"
        android:left="2dp"
        android:right="2dp"
        android:top="2dp">

        <shape android:shape="rectangle" >
            <solid android:color="#ffffff" />

            <corners
                android:topLeftRadius="8.0dip"
                android:topRightRadius="8.0dip" />
        </shape>
    </item>

</layer-list>
```

其他的几种状态如 focused, pressed 请自己脑补。

你的 ListView 需要占满屏幕，并且可以做如下属性设置

```xml
<ListView
    android:id="@android:id/list"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:paddingLeft="10.0dp"
    android:paddingRight="10.0dp"
    android:scrollbarStyle="outsideOverlay"
    android:background ="#ffe4e4e4"
    android:fadingEdge ="none"
    android:listSelector="#00000000"
    android:drawSelectorOnTop="false"
    android:cacheColorHint="#ffe4e4e4"
    android:divider="@null"
    android:dividerHeight="0.0px"
/>
<!-- 让scroll块至于list最外面 -->
```

## 5 题外话

上个礼拜我的博客已经搬到新的 VPS 上了。访问起来应该比以前稳定咯。为了充分利用这个 VPS，我也会加大更新频率。

另外，明天微软实习生面试，攒 RP。
