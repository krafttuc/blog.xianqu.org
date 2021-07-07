---
layout: post
title: 'Android应用开发笔记——打造自己的标签栏'
date: 2011-08-14 15:02:46 +0000
categories:
  - 小小技术
---

注：本文为“[第二届 Google 暑期大学生博客分享大赛 - 2011 Android 成长篇](http://www.google.com/daxue/blog)”参赛博文。

今年暑假我和一群朋友参加了某金融软件开发比赛。我们团队打算制定一个基于条形码的移动支付解决方案（点子诞生于 5 月份的一天，后来我们发现想做的东西已经有了——Google Wallet。囧），软件系统为 C/S 架构，我负责手机客户端部分。考虑到开放性和技术门槛我们选择了 Android 平台。

虽然以前也写过一点 Android 程序，但绝大多数都是所谓的“玩具”，没什么实质性用途。因此，这些玩具们无不例外地拥有着极其简陋的界面。几乎所有的控件都是原生态，纯纯的 Android~

两年前，当 Android 刚出道不久，当各种入门书籍还在漫天飞舞的时候，一款软件穿着原生控件走出去还稍显神气，毕竟那时成型的应用很少。但是，两年后，在这个绿色机器人摧城拔寨之时，如果一款 Android 应用没有好用、漂亮的 UI，你都不好意思说自己在做 Android 应用开发。我觉得如果不在原有控件的基础上做一定的拓展，想要写出优美的界面是难以想象的。

我们团队的软件很大程度上参考了 Twitter，淘宝一类的客户端。我们想在屏幕底部弄一个标签栏（Tabs），这样用户就可以很方便地在不同的页面间切换了。

<!--more-->

作为 Android 新手，我当时只知道一种方法：使用 Android 自带的 TabHost 容器。

传统的 TabHost 的使用效果大概是这样的：

![](/images/2011-08-dianping.jpg)

![](/images/2011-08-twidroyd.jpg)

这是大众点评网早期的 Android 客户端，标签栏使用了系统自带的控件。然而这个效果和我们看到的很多底部标签栏相差甚远（右图，twitter 客户端 Twidroyd 截图）。

他们到底是怎么实现的呢？为此，我在网上查了不少资料，大致有以下种方案：

- Button Bar 实现；
- 修改 TabWidget。

## 1 Button Bar

**基本思路**：用一个 Layout 来呈现整个标签栏，在上面添加若干按钮作为标签，整个 Layout 作为一个部件 include 到各个界面的布局里去。当用户点击按钮时，切换到相应的界面（多个 Activity 间的切换）。

在 Android 3.0 以前的版本里，这种方案多用于制作动作栏。3.0 后出现了 ActionBar 控件，为开发者省了不少事。希望手机版的 Android 也早点加入。

![](/images/2011-08-tabs-1.png)

![](/images/2011-08-tabs-2.png)

**优点**：实现了你想要的效果，外观很容易定制。

**缺点**：标签栏实际不是一个，有多个，他们分别属于不同的界面（从属关系）。这导致了在界面切换的时候，标签栏会随着他的界面（或者说 Activity）“动”，而非常驻在底部。

**改进**：有一种改进方式是，用一个 Activity 来控制整个主界面，标签栏 include 在这个主界面的布局里，点击标签时显示相应的 View（实际上是在一个 Activity 上多个 View 的切换）。这样就保证了标签栏只有一个，而且不会随着标签切换而动来动去。当然，缺点也是显而易见的，当标签过多的时候，这个 Activity 会变得相当复杂。呵呵~万一有个变动，你懂的。

## 2 修改 TabWidget

**基本思路**：在于既然 Android 是开源的，我们就能大改特该，让它听话。我们把 TabWidget 移到底部，然后修改它的外观。

**优点**：达到了目的，继承了 TabWidget 的基本功能，可复用，使用简单；

**缺点**：需要研究不少 Android 资料，查 references 查到眼花，费九牛二虎之力去掉分割线，底部白线之类的东西，完成外观定制。

这个方法貌似有不少人用，这里有几篇中文资料（其实都差不多啦，中文圈你抄我我抄你的。。。唉）：

- [http://www.youmi.net/bbs/thread-102-1-4.html](http://www.youmi.net/bbs/thread-102-1-4.html)
- [http://sillydong.com/myjava/android-tabwidget.html](http://sillydong.com/myjava/android-tabwidget.html)

## 3 奇技淫巧

如你所见，前面说过的方法都能达到目的。但是，Button Bar 的方式看上去太 dirty 了，修改 TabWidget 又太麻烦了。有什么方式能避免这些不必要的麻烦，把两者优点结合起来呢？

我想到了 Android 控件的一个属性： `android:visibility="gone"`。这简直就是救命稻草！它可以让你的控件不显示，而且还不占地方。这有什么用呢？这意味着，我们可以照样用 TabHost，可以照样放 TabWidget，只是，只是我们不把 TabWidget 显示出来。它现在是隐形的标签栏，the real tabs。我们所要做的是：安置一个“傀儡”标签栏，让它来和用户接触。这是不是有点像 Design Pattern 里的“Proxy”呢？

这个方案算是投机取巧之作，很可惜，我不是第一个想到的，你可以看[这里](http://blog.csdn.net/sdhjob/article/details/6312512)。历史总是惊人的相似啊！

标签容器布局，大致代码如下：

```xml
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

RadioGroup 就是底部的标签栏，背景进行了自定义。我在这边 demo 里用了 9.png 格式的图，嗯，你会经常用它的。
此外，标签被选中和未被选中的关键在于`android:background="@drawable/tab_bg_selector"`这句。

```xml
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

下面列出加载它的 Activity 代码：

```java
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

最后，为演示起见我们写 3 个装入容器的 Activity。`TabOneActivity`，`TabTwoActivity`，`TabThreeActivity`。

比如

```java
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

他们的 layout 很简单，在 LinearLayout 里就放了个 TextView。。。这里就不贴了。

最终效果图：

![](/images/2011-08-tabs-demo.png)

当然这只是一个 Demo，要做得更漂亮你需要在细节处好好作文章，譬如，美化文字，添加图标等等。

**题外话**：我觉得美工是你绝对不能忽视的问题，因为没有优秀的图片，你做出来的东西看上去要多渣就有多渣。你弄不出高光效果，你弄不好优雅的渐变，更做不出漂亮的图标。程序猿们，还在觉得自己比美工高级么？呵呵。
