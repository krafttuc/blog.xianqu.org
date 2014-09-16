---
layout: post
title: "My Open Source Cocoa Controls"
date: 2014-02-16 20:10:52 +0800
comments: true
categories: 小小技术
---

I have been writing Objective-C for about 18 months. When I started working on an iOS application in July 2012, I knew little about Objective-C and Cocoa. There're tones of articles and books teaching you how to write Objective-C, but, to me, the most useful resources are real open source projects.

I learned a lot by reading the source code. Beside basic programming languages, lots of best practices are shown in a well-written open source project. Thanks to the version control systems, sometimes I'm even able to see the whole development process. I learned how to make design decisions according to the context. I realized it's more important to get things done than just keeping a perfect idea in my mind. That is to say, to achieve the goals, workarounds and tricks are acceptable and should be taken into consideration. What's more, while reading source code, I was guessing the aims of every piece of code. That's just like you are typing together with the author.

Open source also plays a significant role in modern software development. Note I'm not going to talk this broadly. It may result in tones of topics. I just want to tell you that I can't build my app without the help of so many third-party open source components. I can't even imagine it.

Anyway, I benefit a lot from open source. Open source is wonderful. So, here comes the main point of this post. To give back to the community, I released two cocoa controls days ago. They are [TYMProgressBarView](https://github.com/krafttuc/TYMProgressBarView) and [TYMActivityIndicatorView](https://github.com/krafttuc/TYMActivityIndicatorView).

TYMProgressBarView is a simple progress bar. It provides many appearance options for customizing. You will be happy to play with it and create your own progress bar.

TYMActivityIndicatorView is an activity indicator view with extra features. Most of the behaviors are much like those of `UIActivityIndicatorView`. The killer feature is that it allows you rotating images. That means you can easily implement all kinds of custom activity indicators by replacing images.

Both of them are super simple and easy to use. I bet you'll like them.
