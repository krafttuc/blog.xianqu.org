---
layout: post
title: '给 Pod 添加资源文件'
date: 2015-08-09 16:24:11 +0800
categories:
tags:
---

![](/images/2015-08-cocoapods.jpg)

**注：本文假定读者对 CocoaPods 的使用已经相当熟练，创建过 Pod 或有此打算。**

[CocoaPods](https://cocoapods.org) 是当前 Swift 和 Objective-C 工程中较为流行的依赖管理工具。它拥有超过 10,000 个程序库，通过一份 Podfile 和几条基本命令就能帮助开发者优雅地管理工程依赖。

虽然人们绝大多数时候只是利用 CocoaPods 安装或更新特定版本的库，但是在应用开发过程中，难免会遇到需要自己创建 pod 的情况。CocoaPods 规定每个 pod 库都必须要有一份 `podspec` 文件。在这份文件里，你要填写作者信息、功能简介、版权信息等基本内容。具体语法可以参看[官方文档](https://guides.cocoapods.org/syntax/podspec.html)。

在 `podspec` 中，利用 `source_files` 你可以指定要编译的源代码文件。可是，当你需要把图片、音频、NIB 等资源打包进 Pod 时该怎么办呢？我“有幸”在做 pod 时踩过几个和资源文件有关的坑，在此和大家分享。

## 1. resources or resource_bundles

有经验的同学在我提出怎么在 pod 中打包资源这个问题时，肯定会告诉我 `resources` 这个属性。示例用法如下：

```ruby
spec.resources = ["Images/*.png", "Sounds/*"]
```

我们在 Cocoa 社区见到的绝大多数库都在用它。利用 `resources` 属性可以指定 pod 要使用的资源文件。这些资源文件在 build 时会被直接拷贝到 client target 的 mainBundle 里。这样就实现了把图片、音频、NIB 等资源打包进最终应用程序的目的。

但是，这就带来了一个问题，那就是 client target 的资源和各种 pod 所带来的资源都在同一 bundle 的同一层目录下，很容易产生命名冲突。例如，我的 app 里有张按钮图片叫 "button.png"，而你的 pod 里也有张图片叫 "button.png"，拷贝资源时，我很担心 pod 里的文件会不会把我 app 里的同名文件给覆盖掉？即使没覆盖掉，程序运行时到底用哪张？很显然，我们不希望上述事情发生。

为了解决这一问题，CocoaPods 在 0.23.0 加入了一个新属性 `resource_bundles`。示例用法如下：

```ruby
spec.resource_bundles = {
  'MyLibrary' => ['Resources/*.png'],
  'OtherResources' => ['OtherResources/*.png']
}
```

可见， `resources` 和 `resource_bundles` 的差别是在于后者用字典替换了数组。相较之前所有资源都平铺开来的做法，新属性显式地做了 bundle 层面的分组。有组织、有纪律！CocoaPods 官方显然更推荐 `resource_bundles`。原因有二：

1. 如前所述，用 `resources` 属性容易引起资源的命名冲突。诚然， `resource_bundles` 也有极小的可能在 bundle 名上起冲突，可那也比前者好处理。
2. 用 `resources` 属性指定的资源直接被拷贝到 client target（事实上 CocoaPods 会先运行脚本对 NIB，Asset Catalog，Core Data Model 等进行编译），这些资源无法享受 Xcode 的优化。这是官方文档的说法，但不清楚所指的优化是哪些（图片压缩？）

即便如此，那为什么还有很多开源产品依然在用 `resources` 属性呢？

1. 历史遗留问题。早些年用了这属性，现在运行也好好的，就不动了吧。
2. 随大流。貌似大家都这么用，应该是正确的，我也这么弄吧，懒得去看文档，抄抄改改呗。
3. 咱有奇技淫巧。

很多库在 `podspec` 里其实是这么写的。

```ruby
spec.resource = "Resources/MYLibrary.bundle"
```

把资源加到形如 `MYLibrary.bundle` 的 bundle 里。这样就使得 client target 资源在 mainBundle 根目录下，而各个 Pod 的自带资源则在外面套了个 bundle 后再被拷贝到 mainBundle 里，机智地解决了冲突 √。

还有一种解决方案，那就是在 Objective-C 开发中常见的加前缀法，即对每项资源加前缀。跟我一起说：NS 大法好！

当然，我还是建议大家照着 CocoaPods 推荐的来。

## 2. 访问 bundle

在 CocoaPods 0.36 以前，pod 资源最后都会被直接拷贝到 client target 的 `[NSBundle mainBundle]` 里。你可以用访问 mainBundle 里资源的方式访问它们。比如用 `+ (UIImage *)imageNamed:(NSString *)name` 来访问 pod 的图片。

但是在 CocoaPods 0.36 之后，这件事情发生了一些变化。由于 iOS 8 Dynamic Frameworks 特性的引入，CocoaPods 能帮你打包 framework 了（撒花）。[0.36 版的 release note](http://blog.cocoapods.org/CocoaPods-0.36/)很详细地说明了加入 framework 特性所带来的变化。一个显著区别就是当你的 pod 库以 framework 形式被使用时，你的资源不是被拷贝到 mainBundle 下，而是被放到 pod 的最终产物—— `framework` 里。此时，你必须保证自己在访问这个 framework 的 bundle，而不是 client target 的。

```objc
[NSBundle bundleForClass:<#ClassFromPodspec#>]
```

上面这段代码可以返回某个 class 对应的 bundle 对象。具体的，

- 如果你的 pod 以 framework 形式被链接，那么返回这个 framework 的 bundle。
- 如果以静态库（`.a`）的形式被链接，那么返回 client target 的 bundle，即 mainBundle。

但无论以哪种形式链接，在这个方法返回的 bundle 下都有你的 pod 资源。接下来要做就是去访问他们。我写了个简单的 category[^nsbundle]来获取 `MyLibrary` 的 bundle 对象。

```ruby
spec.resource_bundles = {
  'MyLibrary' => ['your/path/to/resources/*.png'],
}
```

```objc
@implementation NSBundle (MyLibrary)

+ (NSBundle *)my_myLibraryBundle {
    return [self bundleWithURL:[self my_myLibraryBundleURL]];
}


+ (NSURL *)my_myLibraryBundleURL {
    NSBundle *bundle = [NSBundle bundleForClass:[MYSomeClass class]];
    return [bundle URLForResource:@"MyLibrary" withExtension:@"bundle"];
}

@end
```

逻辑很简单：先拿到最外面的 bundle。 对 framework 链接方式来说就是 framework 的 bundle 根目录，对静态库链接方式来说就是 target client 的 main bundle，然后再去找下面名为 `MyLibrary` 的 bundle 对象。

题外话，新的 bundle 策略利用 framework 的命名空间，有效防止了资源冲突。同时对 client 来说，他不需要为 framework 里的资源设置 build rules，如 storyboard，xib 一类需要编译的东西，缩短了编译时间，毕竟 framework 里的资源不需要 client 每次都编译了。我觉得很不错。

## 3. 图片资源

前面我们已经讨论过资源被放到不同 bundle 所带来的访问方式的不同。这次说一下图片的坑，毕竟它们是最常见的资源之一。

### 一般的图片访问

还是针对 `resource_bundles`。

```ruby
spec.resource_bundles = {
  'MyLibrary' => ['your/path/to/resources/*.png'],
}
```

写了一个方便访问 pod 图片的 category[^uiimage]。

```objc
#import "UIImage+MyLibrary.h"
#import "NSBundle+MyLibrary.h"

@implementation UIImage (MyLibrary)

+ (UIImage *)my_bundleImageNamed:(NSString *)name {
    return [self my_imageNamed:name inBundle:[NSBundle my_myLibraryBundle]];
}


+ (UIImage *)my_imageNamed:(NSString *)name inBundle:(NSBundle *)bundle {
#if __IPHONE_OS_VERSION_MIN_REQUIRED >= __IPHONE_8_0
    return [UIImage imageNamed:name inBundle:bundle compatibleWithTraitCollection:nil];
#elif __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_8_0
    return [UIImage imageWithContentsOfFile:[bundle pathForResource:name ofType:@"png"]];
#else
    if ([UIImage respondsToSelector:@selector(imageNamed:inBundle:compatibleWithTraitCollection:)]) {
        return [UIImage imageNamed:name inBundle:bundle compatibleWithTraitCollection:nil];
    } else {
        return [UIImage imageWithContentsOfFile:[bundle pathForResource:name ofType:@"png"]];
    }
#endif
}

@end
```

`+ imageNamed:inBundle:compatibleWithTraitCollection:` 这个方法 iOS 8 才加入的，所以做了条件编译。`+ imageWithContentsOfFile:` 没有缓存机制不开心。当然，如果你坚持用 `resources` 把自己的资源被拷贝到 main bundle 下，然后直接用 `+ imageNamed:`的话，我敬你是条汉子。

### Asset Catalog

理论上一个 bundle 里可以有一个 asset catalog。Xcode 最后会把它们编译成 `Assets.car` 文件。

我觉得把 pod 的图片扔到 asset catalog 里，然后把 `MyLibraryImages.xcassets` 放到 `resource_bundles` 里，也能在代码里通过一定办法访问到图片对象。然而我尝试了各种姿势，依然无法解锁该成就。反倒是用 `resources` 属性可以成功，匪夷所思。

Google 一圈后在 CocoaPods 的 issue 里发现了[#2292](https://github.com/CocoaPods/CocoaPods/issues/2292)，然后 StackOverflow 上又[有人说不行](http://stackoverflow.com/questions/23835052/load-image-from-cocoapods-resource-bundle)。我也犯迷糊了。

现在想到的临时解决方案是，如果你用 `resources` 属性指定资源，那么把图片放到 asset catalog 后，用下列方式是可以拿到图片的。

```objc
NSBundle *bundle = [NSBundle bundleForClass:[MYClass class]];
UIImage *image = [UIImage imageNamed:name inBundle:bundle compatibleWithTraitCollection:nil];
```

而如果你用 `resource_bundles` 属性指定资源，请把图片从 asset catalog 里拿出来裸奔。

希望大家能指点一二，拜谢！

## 4. 国际化和本地化

### 怎么做

做过国际化和本地化的同学都知道在代码里用 `NSLocalizedString(key, comment)` 来代替一般的字符串做国际化。通过在各种以`.lproj`结尾的目录下创建 `Localizable.strings` 文件提供字符串键值对来做本地化。

对一个 pod 来说，如果要提供给 client 本地化资源，其流程也是类似的。不同的地方在于，你最好以 pod 名来命名字符串文件，比如 `MyLibrary.strings`。此外，在做国际化时要把 `NSLocalizedString(key, comment)` 换成 `NSLocalizedStringFromTableInBundle(key, tbl, bundle, comment)`，毕竟你的这些本地化资源也要跟着 bundle 走。其中 `tbl` 就是你 pod 的本地化字符串文件名，如例子是 `MyLibrary`。

我们可以用一个宏定义来简化工作。

```objc
#define MYLibraryLocalizedString(key, comment) \
NSLocalizedStringFromTableInBundle((key), @"MyLibrary", [NSBundle my_myLibraryBundle], (comment))
```

把原先的 `NSLocalizedString(key, comment)` 替换成 `MYLibraryLocalizedString(key, comment)` 就行了。好奇心强的同学不妨深入看看这些宏定义最后到底是些什么东西。

剧透一下，`NSLocalizedString(key, comment)` 的本质就是 NSBundle 的 `- localizedStringForKey:value:table:` 方法。

### 为什么没效果

当你满心欢喜地在代码里完成了国际化，把本地化字符串也翻译好了，运行程序，发现里面语言愣是只有英文，你怎么想？key 写错了？没有。文件名没弄对？也不是。停停停，请先检查以下几点：

1. 请检查你的设备是否已经设置为你要测的语言；
2. 请确保你在工程层面也声明了要做本地化。具体的，点开 Project 设置，点到 Info 一栏，看看 Localization 那块有没有加对应的语言。
3. 请确保你在添加新文件后运行了 `pod update` 命令。（写 pod 遇到的各种二逼问题多半因为没跑这条命令）

## 5. 结语

坑我已经踩过了，谁再踩谁二逼。倘若你也在开发过程中踩过奇奇怪怪的坑，欢迎告诉我。至少……能让这世界少点二逼吧……你说呢？

[^nsbundle]: [NSBundle+MyLibrary](https://gist.github.com/yimingtang/08e2e48969e42935b42c)
[^uiimage]: [UIImage+MyLibrary](https://gist.github.com/yimingtang/04ac9ff730e8ee82c1ee)
