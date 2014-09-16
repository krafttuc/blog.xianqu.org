---
layout: post
title: "Android应用开发笔记——打造自己的标签栏"
date: 2011-08-14 15:02:46 +0000
categories:
- 小小技术
comments: true
---

注：本文为“[第二届 Google 暑期大学生博客分享大赛 - 2011 Android 成长篇](http://www.google.com/daxue/blog)”参赛博文。

今年暑假我和一群朋友参加了某金融软件开发比赛。我们团队打算制定一个基于条形码的移动支付解决方案（点子诞生于5月份的一天，后来我们发现想做的东西已经有了——Google Wallet。囧），软件系统为C/S架构，我负责手机客户端部分。考虑到开放性和技术门槛我们选择了Android平台。

虽然以前也写过一点Android程序，但绝大多数都是所谓的“玩具”，没什么实质性用途。因此，这些玩具们无不例外地拥有着极其简陋的界面。几乎所有的控件都是原生态，纯纯的Android~

两年前，当Android刚出道不久，当各种入门书籍还在漫天飞舞的时候，一款软件穿着原生控件走出去还稍显神气，毕竟那时成型的应用很少。但是，两年后，在这个绿色机器人摧城拔寨之时，如果一款Android应用没有好用、漂亮的UI，你都不好意思说自己在做Android应用开发。我觉得如果不在原有控件的基础上做一定的拓展，想要写出优美的界面是难以想象的。

我们团队的软件很大程度上参考了Twitter，淘宝一类的客户端。我们想在屏幕底部弄一个标签栏（Tabs），这样用户就可以很方便地在不同的页面间切换了。

<!--more-->

作为Android新手，我当时只知道一种方法：使用Android自带的TabHost容器。

传统的TabHost的使用效果大概是这样的：

![](/images/2011-08-dianping.jpg)

![](/images/2011-08-twidroyd.jpg)


这是大众点评网早期的Android客户端，标签栏使用了系统自带的控件。然而这个效果和我们看到的很多底部标签栏相差甚远（右图，twitter客户端Twidroyd截图）。

他们到底是怎么实现的呢？为此，我在网上查了不少资料，大致有以下种方案：

* Button Bar实现；
* 修改TabWidget。

## 1 Button Bar

**基本思路**：用一个Layout来呈现整个标签栏，在上面添加若干按钮作为标签，整个Layout作为一个部件include到各个界面的布局里去。当用户点击按钮时，切换到相应的界面（多个Activity间的切换）。

在Android 3.0以前的版本里，这种方案多用于制作动作栏。3.0后出现了ActionBar控件，为开发者省了不少事。希望手机版的Android也早点加入。


![](/images/2011-08-tabs-1.png)

![](/images/2011-08-tabs-2.png)

**优点**：实现了你想要的效果，外观很容易定制。

**缺点**：标签栏实际不是一个，有多个，他们分别属于不同的界面（从属关系）。这导致了在界面切换的时候，标签栏会随着他的界面（或者说Activity）“动”，而非常驻在底部。

**改进**：有一种改进方式是，用一个Activity来控制整个主界面，标签栏include在这个主界面的布局里，点击标签时显示相应的View（实际上是在一个Activity上多个View的切换）。这样就保证了标签栏只有一个，而且不会随着标签切换而动来动去。当然，缺点也是显而易见的，当标签过多的时候，这个Activity会变得相当复杂。呵呵~万一有个变动，你懂的。

## 2 修改TabWidget

**基本思路**：在于既然Android是开源的，我们就能大改特该，让它听话。我们把TabWidget移到底部，然后修改它的外观。

**优点**：达到了目的，继承了TabWidget的基本功能，可复用，使用简单；

**缺点**：需要研究不少Android资料，查references 查到眼花，费九牛二虎之力去掉分割线，底部白线之类的东西，完成外观定制。

这个方法貌似有不少人用，这里有几篇中文资料（其实都差不多啦，中文圈你抄我我抄你的。。。唉）：

* [http://www.youmi.net/bbs/thread-102-1-4.html](http://www.youmi.net/bbs/thread-102-1-4.html)
* [http://sillydong.com/myjava/android-tabwidget.html](http://sillydong.com/myjava/android-tabwidget.html)

## 3 奇技淫巧

如你所见，前面说过的方法都能达到目的。但是，Button Bar的方式看上去太dirty了，修改TabWidget又太麻烦了。有什么方式能避免这些不必要的麻烦，把两者优点结合起来呢？

我想到了Android控件的一个属性： `android:visibility="gone"`。这简直就是救命稻草！它可以让你的控件不显示，而且还不占地方。这有什么用呢？这意味着，我们可以照样用TabHost，可以照样放TabWidget，只是，只是我们不把TabWidget显示出来。它现在是隐形的标签栏，the real tabs。我们所要做的是：安置一个“傀儡”标签栏，让它来和用户接触。这是不是有点像Design Pattern里的“Proxy”呢？

这个方案算是投机取巧之作，很可惜，我不是第一个想到的，你可以看[这里](http://blog.csdn.net/sdhjob/article/details/6312512)。历史总是惊人的相似啊！

标签容器布局，大致代码如下：

``` xml
<?xml version="1.0"; encoding="utf-8"?>

<!-- TabHost -->
<TabHost
	xmlns:"android=http://schemas.android.com/apk/res/android"
	android:layout_width="fill_parent"
	android:layout_height="fill_parent"
	android:id="@android:id/tabhost">

	<LinearLayout
		android:orientation="vertical"
    	android:layout_width="fill_parent"
    	android:layout_height="fill_parent">

    <!-- 隐藏的TabWidget,visibility="gone" -->
		<TabWidget
			android:id="@android:id/tabs"
			android:layout_width="fill_parent"
			android:layout_height="wrap_content"
			android:visibility="gone"	/>

  		<!-- 标签内容 -->
		<FrameLayout
			android:id="@android:id/tabcontent"
			android:layout_width="fill_parent"
			android:layout_height="0.0dip"
			android:layout_weight="1.0"/>

		<!-- 用户可见的“tabs”实际上是一组按钮-->
		<!-- 注意：checkedButton=“@+id/tab1“，这使得默认选中第一个按钮 -->
		<RadioGroup
			android:id="@+id/tab_group"
			android:layout_width="fill_parent"
			android:layout_height="50dp"
			android:gravity="center_vertical"
			android:layout_gravity="bottom"
			android:orientation="horizontal"
			android:background="@drawable/tab_bar_bg"
			android:checkedButton="@+id/tab1"></p>

      <!-- 第一个标签，注意button属性设置成null，以此去掉自带的radio button -->
			<!-- 注意：id="@id/tab1"，为什么不是+id呢？这个和加载先后有关系,Google一下吧 -->
			<RadioButton
				android:id="@id/tab1"
				android:tag="tab1"
				android:layout_width="fill_parent"
	  			android:layout_height="fill_parent"
	  			android:layout_weight="1.0"
				android:layout_marginTop="1.0dip"
				android:text="Tab 1"
				android:button="@null"
				android:gravity="center"
				android:background="@drawable/tab_bg_selector"/>

			<RadioButton
				android:id="@+id/tab2"
				android:layout_width="fill_parent"
	  			android:layout_height="fill_parent"
	  			android:layout_weight="1.0"
				android:layout_marginTop="1.0dip"
				android:text="Tab 2"
				android:button="@null"
				android:gravity="center"
				android:background="@drawable/tab_bg_selector"/>

			<RadioButton
				android:id="@+id/tab3"
				android:layout_width="fill_parent"
	  			android:layout_height="fill_parent"
	  			android:layout_weight="1.0"
				android:layout_marginTop="1.0dip"
				android:text="Tab 3"
				android:button="@null"
				android:gravity="center"
				android:background="@drawable/tab_bg_selector"/>

		</RadioGroup>
	</LinearLayout>
</TabHost>
```

RadioGroup就是底部的标签栏，背景进行了自定义。我在这边demo里用了9.png格式的图，嗯，你会经常用它的。
此外，标签被选中和未被选中的关键在于`android:background="@drawable/tab_bg_selector"`这句。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<selector
  xmlns:android="http://schemas.android.com/apk/res/android"></p>

		<item
    	android:state_focused="true"
    	android:drawable="@drawable/tab_bg_focused" />
    <item
    	android:state_pressed="true"
    	android:drawable="@drawable/tab_bg_selected" />
    <item
    	android:state_checked="true"
    	android:drawable="@drawable/tab_bg_selected" />

</selector>
```


下面列出加载它的Activity代码：

``` java
public class TabDemoActivity extends TabActivity {

	private TabHost tabhost;
	private RadioGroup tabGroup;

	@Override
	public void onCreate(Bundle savedInstanceState) {

		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);

		tabhost = getTabHost();
		tabGroup = (RadioGroup) findViewById(R.id.tab_group);

		// 这里新建3个的Intent用于Activity的切换
		Intent tab1 = new Intent(this, TabOneActivity.class);
		Intent tab2 = new Intent(this, TabTwoActivity.class);
		Intent tab3 = new Intent(this, TabThreeActivity.class);

		// 向tabhost里添加tab
		tabhost.addTab(tabhost.newTabSpec("TAB1").setIndicator("Tab 1")
				.setContent(tab1));
		tabhost.addTab(tabhost.newTabSpec("TAB2").setIndicator("Tab 2")
				.setContent(tab2));
		tabhost.addTab(tabhost.newTabSpec("TAB3").setIndicator("Tab 3")
				.setContent(tab3));

		// 给各个按钮设置监听
		tabGroup.setOnCheckedChangeListener(new OnTabChangeListener());

	}

	private class OnTabChangeListener implements OnCheckedChangeListener {

		@Override
		public void onCheckedChanged(RadioGroup group, int id) {
			// TODO Auto-generated method stub

			//尤其需要注意这里，setCurrentTabByTag方法是纽带
			switch (id) {
			case R.id.tab1:
				tabhost.setCurrentTabByTag("TAB1");
				break;
			case R.id.tab2:
				tabhost.setCurrentTabByTag("TAB2");
				break;
			case R.id.tab3:
				tabhost.setCurrentTabByTag("TAB3");
				break;
			}
		}
	}
}
```


最后，为演示起见我们写3个装入容器的Activity。`TabOneActivity`，`TabTwoActivity`，`TabThreeActivity`。

比如

``` java
public class TabOneActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.content);
		TextView text=(TextView)findViewById(R.id.text);
		text.setText("This is tab 1.");

	}
}
```

他们的layout很简单，在LinearLayout里就放了个TextView。。。这里就不贴了。

最终效果图：

![](/images/2011-08-tabs-demo.png)

当然这只是一个Demo，要做得更漂亮你需要在细节处好好作文章，譬如，美化文字，添加图标等等。

**题外话**：我觉得美工是你绝对不能忽视的问题，因为没有优秀的图片，你做出来的东西看上去要多渣就有多渣。你弄不出高光效果，你弄不好优雅的渐变，更做不出漂亮的图标。程序猿们，还在觉得自己比美工高级么？呵呵。
