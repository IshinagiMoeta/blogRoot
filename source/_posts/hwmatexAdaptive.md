---
title: 华为MateX适配先行指北(折叠屏它来了！)
date: 2019-03-13 15:34:34
tags: [Android,适配]
categories: Android

---
去年11月8号，三星先是在发布会上发布了新款的折叠屏的手机。紧接着今年2月，华为也祭出了MateX，虽然发布会上已经看到过实体机了，但是商城迟迟还是没有上架，作为一个开发者，确先是收到了华为发的折叠屏适配的通知。脑壳疼  

我全面屏还没全部整完呢，你们真会玩  

先瞄了眼[华为Mate X显示适配指导](https://developer.huawei.com/consumer/cn/devservice/doc/90101)从上到下都是UI适配，从4.2开始才是开发适配。细看一下，大致是要我们去根据屏幕的变动来自适应布局，嗯嗯，我完全搞懂了呢...你个大头鬼哦！

按照文档内的意思是要我们去监听onConfigurationChanged()方法，根据方法传回来的config来决定怎么去修改界面，达到不重启切换的目的。好的，看起来没什么问题，已经给出了几种屏幕比例，所以我们看看config里面的宽高比，然后再去适配，就可以了对吧。

| 状态 | 比例 | 分辨率 |  
| :------: | :------: | :------: |  
| 展开态 | 8：7.1 | 2480x2200 |  
| 折叠态正屏 | 19.5:9 | 2480x1148 |  
| 展开态 | 25:9 | 2480x892 |  


但是，这只是华为的情况，目前有三种类型比例的屏幕，展开8：7.1，折叠态正面19.5：9，折叠态背面25：9，以后如果还有其他厂商出了折叠屏呢，我们还要一个个的去算长宽比再去适配不成？UX小姐姐要提着刀杀了我的哟。

明显不能这么弄，该怎么办呢，我又去找了下Google爸爸，果然谷歌爸爸还是很贴心的，在三星发布会后几天，就推出了foldable技术。

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1g11ay1p9jvg20hs0a07wh.gif)

相关博客在这里~  
[https://android-developers.googleblog.com/2018/11/get-your-app-ready-for-foldable-phones.html](https://android-developers.googleblog.com/2018/11/get-your-app-ready-for-foldable-phones.html)

谷歌爸爸提出了一个屏幕连续性的概念，就是说你的应用应该可以自动从一个屏幕转到另一个屏幕上，每个屏幕都应该有自己的屏幕ID,用来通知开发者做相应的适配。这个ID在Android P中有新添加，同时，Google团队正在针对这种行为进行兼容性优化。

这样就好办了，三星的折叠屏明显是2个屏幕，会对应两个不同的屏幕ID，我们只要得到对应ID和对应分辨率就能进行适配，但是华为那边我看起来还是有点毛毛的，因为，目前看起来，华为的屏幕是一整个屏幕，只不过折叠的之后可以只用一半而已。emmmm，会不会有不同的屏幕ID呢？这是个问题。

然后是代码部分的处理，不管怎么说，我们需要先在xml里面添加以下内容：

```xml  
<meta-data android:name = "android.allow_multiple_resumed_activities" android:value = "true"/>  
//Android7.1及以下版本，添加  
<meta-data android:name = "android.max_aspect" android:value = "2.4"/>  
<meta-data android:name = "android.min_aspect" android:value = "1.0"/>  
//Android8.0及以上版本，在GameAct标签内添加  
<activity  
	android:configChanges = "orientation|keyboardHidden|screenSize|smallestScreenSize|screenLayout"  
	android:resizeableActivity = "true"  
	android:minAspectRatio = "1.0"  
	android:maxAspectRatio = "2.4">  
```

然后再Activity中我们需要复写onConfigurationChanged()，这时我们的程序就会自动捕捉屏幕的变动，分辨率的变动，来调用该方法，并传入新的屏幕属性，我们做对应处理就可以了。

刚刚上面说的Google爸爸提供的屏幕ID接口为，ActivityOptions.setLaunchDisplayId()和ActivityOptions.getLaunchDisplayId()，其中set方法是用来在你的应用启动新Activity的时候，决定在哪个Activity上面启动的方法。

目前网上能找到的资料基本就是这些了，而且设备也没摸到，姑且先这样准备着，到时候拿到了设备我们再进行细节调整。

