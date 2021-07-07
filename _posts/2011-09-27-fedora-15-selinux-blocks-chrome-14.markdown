---
layout: post
title: '从Fedora 15 SELinux阻止Chrome 14说起'
date: 2011-09-27 07:34:48 +0000
categories:
  - 业界评论
  - 小小技术
---

前些日子由于课程需要我拥入了 Fedora 15 的怀抱。在经过了半个多小时的安装后，一个又一个 Linux 系统在我机器上诞生了。Fedora 在众多的 Linux 发行版里算是比较贴近初级用户了，默认提供了图形化系统管理工具。因为以前用过 Ubuntu，所以除了 GNOME 3 用起来不太习惯之外，一切还算顺利。

正当我欢快地为系统做一些配置，装一些常用软件的时候，麻烦来了。我想装个 Chrome 玩玩，于是去 Google 那下载了程序的 RPM 包安装。软件在安装过程中很是 OK，在安装结束后，习惯性地打开 Chrome 想做一下账户同步。结果按下启动图标后桌面毫无反应，然后就看到下面这行东西。

![](/images/2011-09-avc-alert.png)

Oops，SELinux 这货是什么东西？AVC 又是啥玩意？这让我完全摸不着头脑。以前在 Ubuntu 下完全没遇到过这种问题。于是我立刻条件反射般地 Google 了一番。

这是维基百科关于[SELinux](http://zh.wikipedia.org/wiki/SELinux)的条目解释。更详细的英文条目在[这里](http://en.wikipedia.org/wiki/Security-Enhanced_Linux)

> 安全增强式 Linux（SELinux, Security-Enhanced Linux）是一种强制访问控制（mandatory access control）的实现。它的作法是以最小权限原则（principle of least privilege）为基础，在 Linux 核心中使用 Linux 安全模块（Linux Security Modules）。它并非一个 Linux 发行版，而是一组可以套用在类 Unix 操作系统（如 Linux、BSD 等）的修改。

原来是访问控制程序，还是美国国家安全局弄的，被集成到了部分 Linux 的核心版本中。譬如我手上的 Fedora，Red Hat 的操作系统上默认都有这玩意。那么这个问题到底怎么解决呢？

既然是 SELinux 是访问控制程序，那么**基本思路就是让它允许 Chrome 通过**就行了。然而自己以前从未接触过 SELinux 怎么办？本能反应是看看它有没有提供一些解决问题的提示。

![](/images/2011-09-selinux-alert-1.png 909 537)

在 root 权限下，输入了以下命令：

```
semanage fcontext -a -t textrel_shlib_t '/opt/google/chrome/chrome'
restorecon -v '/opt/google/chrome/chrome'
```

`man semanage` ，了解这两条命令的大意是让 Chrome 可以访问 `/opt/google/chrome/chrome` 下的文件。然而这样做依然没能让 Chrome 跑起来，我试着从双击图标和终端运行得到这样的错误

![](/images/2011-09-selinux-alert-2.png)

![](/images/2011-09-selinux-alert-3.png)

坑爹啊！既然这样只能去看看别人怎么做了。搜索“chrome selinux”立刻得到了大量的结果。PS:中文互联网上关于这个问题似乎没半点影子。

原来这个问题早在几个月前就有人在 SELinux 的维护组和 chromium 的 BUG 反馈处提交了。在 chromium 的[issue](http://code.google.com/p/chromium/issues/detail?id=87704)回帖里，有不少临时的解决方案。

有人提议：直接把 SELinux enforcingmode 禁了！很显然，这种做法虽然能解决问题，但却是杀鸡取卵的做法，相当不靠谱。于是，立刻有人指出这个方案是一个 BAD IDEA，并提供了其他解决方案。输入以下命令：

```
chcon -t usr_t  /opt/google/chrome/chrome-sandbox
```

这可以暂时让 Chrome 工作，但是它不是一个永久的解决方案。一旦 Chrome 升级（这里指这个 bug 没有被修复的升级）,那么它将失效。你会看到如前所示的错误。又有人提出了一个方案：

```
semanage fcontext -a  -t usr_t /opt/google/chrome/chrome-sandbox
restorecon -v /opt/google/chrome/chrome-sandbox
```

这和 SELinux 的提示差不多，能比较好的让 Chrome 跑起来。

虽然网友们给出的解决方案形形色色，但是稍微仔细观察一下就能发现，**它们都是通过修改 SELinux 的 policy 做的 workaround**。问题并没有从更本上解决，而且大家理所当然的认为 \_It's a Fedora 15 bug or it's a SELinux issue. chromium 的一个程序员在回帖里这样提到：

> I filed another bug with Fedora:
> [https://bugzilla.redhat.com/show_bug.cgi?id=730449](https://bugzilla.redhat.com/show_bug.cgi?id=730449)
> We've had breakages before due to Fedora SELinux policy. If it were in our control to fix, we would have fixed it (see the other bugs mentioned on that bug). If there's anything you can do to help out on the Fedora bug I'd appreciate it.
> It is frustrating that users come to us when Fedora changes break us. :(

呵呵，如果你是一个程序员，当别人向你提交一个 BUG 的时候，是不是第一时间也这么想呢？

接着有人立刻反驳了这位开发人员。我目前正在用 Fedora 12，并安装了 Google Chrome 14，也同样遇到了这个问题。显而易见，这并不是 Fedora 或者 SELinux 升级带来的错误。所以，这是你的错误，而非 Fedora 的。他还补充到：**这不是 Chrome 发现了 SELinux 的错误，而是 SELinux 暴露了 Chrome 的 bug。**同样，有人反应\_为什么我用 Chrome 13 的时候就没这问题呢，反而是升级后就被阻止了。被这么一说，先前的那个开发人员也承认了自己的不对之处。但在括号里来了一句：

> I've lately been wondering if it'd be better for us to stop saying we support Fedora, given these sorts of problems coming up so often.

除了 chromium 这边，Red Hat 那边也有关于这个 bug 的 issue

- [https://bugzilla.redhat.com/show_bug.cgi?id=730449](https://bugzilla.redhat.com/show_bug.cgi?id=730449)
- [https://bugzilla.redhat.com/show_bug.cgi?id=730179](https://bugzilla.redhat.com/show_bug.cgi?id=730449)

当然所讨论内容大抵也是这个 bug 归咎于谁，问题出在哪，怎么修复。有人提供了一个 quik fix:修改 SELinux policy...又是 workaround...按我看来，不管错误归咎于谁，**彻底地解决问题才是最重要的**。从错误报告来看，

> chrome is doing text-relocations which is NOT good, and SHOULD be blocked by SElinux.

而**让人疑惑的是**：

> The chrome application attempted to load /opt/google/chrome/chrome which requires text relocation. This is a potential security problem. Most libraries do not need this permission.

但是 Chrome 不是一个 library。截至我写这篇文章的时，在[comment 65#](http://code.google.com/p/chromium/issues/detail?id=87704#c65)，chromium 的管理员已着手开始修复。到底什么会怎样，让我们拭目以待。

## 后记：

像 SELinux 阻止 Chrome 14 一样，类似的事情每天都在发生。即使再牛逼的程序员也会避免不了一些陋习。“这不是我的错！”你是不是也这样呢？:-)，我觉得不管怎样，写程序就是为了从根本上解决问题，只要有更高的解决方案，那么就应该不停得往上靠。即使有时你看不见，但它就在那里。相反的，除非万不得已，丑陋的方案不应该存在。
