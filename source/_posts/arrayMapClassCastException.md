---
title: 一个来自于ArrayMap的ClassCastException  
date: 2019-01-28 10:01:52  
tags: [Android,Exception]  
categories: Android  

---
最近一段时间比较忙，年关将至，公司突然蹦出来了各种各样的问题，脑壳疼。  
这篇博客主要来记录一下最近遇到的一个比较奇葩的问题。一个关于ArrayMap的奇怪异常。  


这个问题我们在开发和测试过程中一次都没有遇到过，真正抓住是在Bugly上发现的，上下文很奇怪，乍一看好像是一个布局问题，像是RelativeLayout的子View的new和remove冲突导致了这个异常。但是我们是在是找不到工程里面哪个地方有这种冲突，实在是找不到一丝头绪，最后还是通过面向Stack Overflow编程的方式来找到了这个问题的根本原因。  

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fzm2uonbkbj20vh0mcwgn.jpg)  

其实这个错误可能发生在任何使用ArrayMap的任何地方,比如Fragment,Intent,Bundle，由于Android Framework层中大量使用ArrayMap存储,所以就算你自己不使用也仍有可能遇到此问题。  

目前已经确认是Android系统本身的一个Bug,相关issue可以参考：  
[https://issuetracker.google.com/issues/37114373](https://issuetracker.google.com/issues/37114373)  

如果你无论如何都想要复现这个问题或者你想要知道引起BUG的最根本原因的话，可以看看这个：  
[https://www.jianshu.com/p/02890898ee68](https://www.jianshu.com/p/02890898ee68)  

其实简单来说，就是ArrayMap是非线程安全的，如果同时在多个线程中调用ArrayMap，在一定量级的用户数基础下，这个bug很有可能会给你的bugly刷榜。  

因为这是一个系统级的bug，google团队貌似在Android Pie版本中才对他修复，而据我所知目前我们的targetSDK版本基本集中在23~26之间，所以这个问题只能规避，或者你修改底层源码，但是由于我只是一个一般的APK开发者，对底层不熟，所以这里就不献丑了。  

规避方法：  
1、不要主动在子线程中调用ArrayMap，确保ArrayMap只出现在主线程中，就不会出现这个问题。  

2、虽然你能管得住自己的代码，但是你管不住三方SDK的，我们这个问题就是出在了友盟分享模块上。所以针对这种情况，我们首先要定位问题出现的原因。这里有个AS的黑科技可以教大家一下。 

首先找到问题出现的位置，在该处断点，右键断点，调出该弹窗。

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fzm3iezbkjj20ue0ktq58.jpg)  

然后在condition中填入你想要打印的数据，我这里主要是想看看有多少线程调用了ArrayMap,所以打印的是线程信息。  

> Log.e("######",Thread.currentThread().toString())==0

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fzm3ju4222j20bk085dfx.jpg) 

然后开启Debug模式，查看打印  

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fzm3mxmik9j20nl0aegms.jpg)  

emmm，看来问题就是出在这里了，有两个线程机会同时调用了ArrayMap，现在要追踪这个线程是什么东西调用的，3.2以下的AS可以使用DDMS中的Threads查看：[android性能优化之DDMS中查看thread信息](https://www.jianshu.com/p/8fa300effa59)，3.2以上的可以从Profiler中查看，还是比较方便的。

然后剩下的就要看各位的需求来调整了，这个bug的触发概率其实比较低，所以如果不会导致特别的后果，一般可以不太用管，或者错开ArrayMap的多线程调用即可。