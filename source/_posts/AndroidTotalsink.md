---
title: 我的Android学习路径
date: 2018-09-29 15:23:45
tags: [Android,总结]
categories: Android

---
这篇博客主要是为了总结自己在学习Android过程中得到的经验，同时也算是自己对自己学习的知识的一个系统化梳理，因为博主的技术能力有限，而且时常脑抽，如果这篇文章能够帮到什么人的话，那我要提醒你小心文章中的坑23333。

# 概要 #
------



本篇文章所说的Android开发大部分为**应用层**所涉及到的知识，关于Android系统全貌，由于博主也没有研究透彻，所以也没法系统性的整理，但是文中可能会出现一些底层知识。  
本文主要涉及的是下图中Application的知识，同时可能会含有一些博主对Framework层的理解。
![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fvqjji5kh4j20k70go131.jpg)

如果有读者对Android全貌有兴趣，所以这里推荐一篇博主认为写的比较详细的说明Android系统的文章  
[Android系统开篇](http://gityuan.com/android/#syscall--jni "http://gityuan.com/android/#syscall--jni")  

# 前期准备 #
------


工欲善其事，必先利其器  
## 一、硬件 ##
- **电脑**：没有推荐  
博主从入门到现在一直使用的是Windows系统开发，Mac系统虽然用过，但是一段时间后又换回来了。并不是说Mac系统不好，但是博主除了Android之外，还喜欢捣鼓一些其他的黑科技，收集一些看似能够提升效率的工具，Mac在这个上面相比Windows有些劣势，但是不得不说，Mac的开发效率是真的高，Google的员工都在用Mac。不过这个主要是看人吧，毕竟真正的程序员网吧也能敲代码，就和真正的作家网吧也能码字一样。  
- **测试用手机**：有条件的强烈推荐小米  
Android开发必然会需要调试自己开发的程序，在入门的时候，可以使用模拟器啥的（也推荐一下模拟器好了[Genymotion](https://juejin.im/post/5aec3fa96fb9a07aca7a0961 "Genymotion")），但是开发的越久.....  
各位，模拟器的能力是有极限的  
我从短暂的开发过程当中学到一件事......  
越是各种测试,就越会发现模拟器的能力是有极限的......  
除非超越模拟器。  
所以这里我推荐大家有条件的买个真机，几百块的就行，Android系统最好是6.0的，这个涉及到版本适配问题，之后会详细说明。  
还有就是博主是米粉，不是其他原因，单纯的是因为在开发过程中，遇到过各种机型适配问题后发现，国产机中，VO两家问题最多，华为其次，小米最少。(顺便三星是真的垃圾，三星的拍照接口和大家都不一样，问题很多。  
 
## 二、开发环境 ##
主流的Android开发环境主要有三种  

- Android Studio  
Android Studio(以下简称AS)，AS是Google推出，专门为Android“量身订做”的，是Google大力支持的一款基于IntelliJ idea改造的IDE，google的工程师团队一直在不断完善，现在AS相比其他Android开发软件（Eclipse我并不是针对你，我只是想说：）具有的优点有：速度更快，更加智能，整合了Gradle，具有强大的UI编辑器，内置终端模拟器，海量第三方插件，完美整合各个版本控制，本身UI更美观。  
- Eclipse+ADT：  
先说明一点，Android团队已经停止了对Eclipse的ADT的更新，现在ADT的更新是由Eclipse团队完成的，所以，你如果使用Eclipse开发的话，没法第一时间体验最新Android版本，同时Eclipse会逐渐被淘汰，不过现在任有一定数量的公司在使用Eclipse开发。
- IntelliJ IDEA：  
虽然说AS是基于IntelliJ idea改造的，但是现在AS和IntelliJ idea在Android开发上可以说是天差地别，AS是专门针对Android开发的IDE，而IntelliJ idea属于多语言开发的IDE。Google团队将NDK(Native development kit)从Android Support Plugin中抽出了，如果使用IntelliJ idea则无法使用NDK。并且之后可能会出现更多的工具只能在AS中使用，而不能在其他应用中使用。  

综上所述，力推Android Studio作为Android入门开发软件。  

## 三、翻墙 ##
国内的网络环境对Android开发及其不友好，谷歌被墙在门外，导致我们如果不翻墙的话，就根本没法获取官方提供的资源和文档，同时AS下载的也是墙外资源。  
作为一名程序员，翻墙并习惯使用Google解决问题，会发现一个新世界。~~（面向 Google/Stack Overflow 编程）~~  
这里推荐使用Shadowsocks/ShadowsocksR来翻墙，教程如下 [使用shadowsocks科学上网](https://www.textarea.com/ExpectoPatronum/shiyong-shadowsocks-kexue-shangwang-265/ "使用shadowsocks科学上网") ，这个教程中需要花钱的地方只有买服务器，自建翻墙服务，如果懒得去自建服务器的话，可以使用Shadowsocks自带的免费服务器爬墙，也可以选择购买Shadowsocks的付费服务，几百块钱换一年可以翻墙的机会，绝对不亏。   

# Google官方教程 #
------
[谷歌官方教程](https://developer.android.com/guide/)  
[国内汉化教程](http://hukai.me/android-training-course-in-chinese/index.html)  
官方提供的东西虽然全面，但是因为Android系统本身比较复杂，所以刚入门的新手可能看起来反而比较迷茫，可以作为工具书来查阅，入门学习的话，并不是很推荐。  

# Android基础知识 #  
------
以下会列出博主自己在学习过程中遇到的，认为比较好的其他博客  

首先要致敬**stormzhang**大神的[Android学习之路](http://stormzhang.com/android/2014/07/07/learn-android-from-rookie/)。

以下干货：  
## 零、语言基础 ##
目前Android开发最热门的语言是**Java**，同时Google团队也推出了**kotlin**，如果你是个想要在国内找一份用来谋生工作的，没有工作经验，没有JAVA经验的人。不要使用kotlin入门，就我所知，目前国内招Android开发的公司，90%使用的是Java作为开发语言~~（而且就算你Android入门失败了，你还可以用Java去找工作）~~。如果是想要提升自己，或者想要学点新知识来充电的话，kotlin其实是一个很不错的语言。  
所以学好Java基础很重要  

- Java知识：  
博主入门的参考书：[大话Java](https://book.douban.com/subject/3640322/)  
- Java设计模式：  
[大话设计模式](https://book.douban.com/subject/2334288/)

本文并不推荐直接买书，这两本书在网上资源已经流传了很久了，大家可以下载看看，等有能力之后可以再补票，如果你比较习惯看实体书，那直接买也无所谓，反正这两本书都很经典，值得购买。

## 一、Android基本知识 ##
写给出工具贴，本帖只用来查找你在阅读下文中遇到的不理解的知识点的扩展，还有对博主说的不完全的知识点的补充。[Android 知识梳理](https://juejin.im/post/587dbaf9570c3522010e400e)  

这里会列出Android开发的基本知识点，和对应博客   

- [Android四大基本组件](https://www.cnblogs.com/bravestarrhu/archive/2012/05/02/2479461.html)
	- [Activity](https://www.jianshu.com/p/d4d677727a0a)
	- [Service](https://www.jianshu.com/p/b44e2620167f)
	- [Content Provider](http://www.cnblogs.com/linjiqin/archive/2011/05/28/2061396.html)
	- [Broadcast Receiver](https://www.jianshu.com/p/5459c653b34a)
- [Intent](https://www.jianshu.com/p/19147a69e970)
- [Fragment](https://www.jianshu.com/p/dedbae8ffca1)
- [数据存储](https://ishinagimoeta.github.io/2018/09/30/AndroidStrageWays/)
	- SharedPreference
	- Sqlite
	- 文件存储
	- 网络存储
- [网络请求](https://www.jianshu.com/p/3141d4e46240) 
- [常用View与优化](https://ishinagimoeta.github.io/2018/10/10/ViewDetailed/)
- [Android屏幕适配](https://juejin.im/post/5ae32bac518825671a638405)  

如果学完了上面的基础，基本上开发一个简单的APP就没什么问题了，不过想要熟练应用，还需要多练习，多谷歌。可以随便找一个简单点的APP复刻一下，用自己的方式还原出市面上的简单APP。  
或者自己有些想法的话，也可以自己做一个原创APP，不过我推荐大家最好去做一些和网络请求有关的APP，这里推荐一个有较多免费API的平台[MobAPI](http://api.mob.com/#/)。




















[Android资源推荐](https://github.com/Freelander/Android_Data)  
[Android开发技巧](https://github.com/cctanfujun/android-tips-tricks-cn)