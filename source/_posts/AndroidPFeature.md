---
title: Android P 版本适配
date: 2018-10-11 16:22:46
tags: [Android,Android-P]
categories: Android

---
Android P 这次有很多行为变更，其中不乏一些需要注意的变更。

# 一、全面屏检测 #

于2016年10月25日小米就推出了第一款真正意义上的全面屏手机 — 小米MIX，可谓是震惊四座，然后2017年9月13日发布的的IPhoneX将全面屏推向了世界，由于其独特的结构，和极具科技感的外形十分受消费者们的推崇，各大厂商开始纷纷生产各种各样的全面屏手机。  
2018年是全面屏井喷的一年，而P版本之前，Android 官方并未支持到该功能，所以各个厂商都各自实现了一套全面屏判断逻辑，对于开发者来说甚是麻烦。终于在 Android P 里官方收归了该功能的判断逻辑，Android P 和之后的版本完全可以使用官方 API 来判断全面屏，当然前提是第三方厂商按照 google 官方接口去实现。  
Android P 版本判断全面屏代码很简单，但是在适配过程中你可能会在网上发现如下判断代码：  
**错误适配案列**  
```java  
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    decorView.setOnApplyWindowInsetsListener(new View.OnApplyWindowInsetsListener() {
        @RequiresApi(api = 28)
        @Override
        public WindowInsets onApplyWindowInsets(View view, WindowInsets windowInsets) {
            if (windowInsets != null) {
                DisplayCutout cutout = windowInsets.getDisplayCutout();
                if (cutout != null) {
                    List<Rect> rects = cutout.getBoundingRects();
                    //通过判断是否存在rects来确定是否全面屏手机
                    if (rects != null && rects.size() > 0) {
                        isNotchScreen = true;
                    }
                }
            }
            return windowInsets;
        }
    });
}  
```

这段代码确实可以判断出全面屏与否，但是会造成一个很严重的后果，就是在某些手机（pixel 和 vivo x21 均出现该情况）上底部导航栏会透明，导致应用内容会透到导航栏从而被遮挡，大大影响内容展示。最后经过仔细排查发现仅仅因为在上面那段代码中调用了 `setOnApplyWindowInsetsListener`函数，该函数在 Android 官网有详细介绍，是用来在 Android 21 版本之后代替 fitSystemWindows 函数，目的是让 View 根据 Window 的缩进进行相应处理，调用后会影响系统状态栏和导航栏对应用内容的展示，就不赘述了。真正完美判断全面屏的代码如下：  
**正确适配案例**  
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    WindowInsets windowInsets = decorView.getRootWindowInsets();
    if (windowInsets != null) {
        DisplayCutout displayCutout = windowInsets.getDisplayCutout();
        if (displayCutout != null) {
            List<Rect> rects = displayCutout.getBoundingRects();
            //通过判断是否存在rects来确定是否刘海屏手机
            if (rects != null && rects.size() > 0) {
                isNotchScreen = true;
            }
        }
    }
}
```

# 二、非 SDK API 适配 #
Android P 版本最大最严格的特性变更应该非 SDK 接口限制莫属了。对于非 SDK API 里面的部分名单来说，就算在不修改 targetSdkVersion 的前提下，不管是直接、反射还是通过 JNI 调用都会造成调用失败、抛出 NoSuchFieldException或 NoSuchMethodException 等严重后果，该行为影响范围波及所有调用此接口的应用。
## 2.1 非 SDK API 名单 ##
非 SDK API 名单总共分为三类：

- light grey list （浅灰名单）
- ark grey list （深灰名单）
- dark list（黑名单）

详情：
![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fw4bmvwpwkj20ll0ctgve.jpg)
## 2.2非 SDK API 名单扫描 ##  
所以对于我们应用开发者来说，当前首要任务是适配深灰名单和黑名单。目前 google 官方提供了一个可以实时查询三个名单里面 API 列表的网站：  [查询黑名单API](https://android.googlesource.com/platform/frameworks/base/+/master/config/)，DP 版本时开发者如果遇到了不得不使用的黑名单或者深灰名单 API，需要向 google 官方及时提出反馈([反馈URL](https://issuetracker.google.com/issues/new?component=328403&template=1027267))  
详细了解了非 SDK API 之后，下一步当然是将应用代码里面的深灰名单和黑名单 API 调用找出来一一修改。目前官方提供了一个非常实用的扫描工具，该工具可以把应用里面三个类型名单的 API 调用都扫描出来（但是可能会有遗漏），使用方法也很简单：

1. 打包一个应用 APK，建议使用 release 包，排除一些未使用到的单元测试类或者其他因素的影响，将 APK 放到工具指定目录下；
2. 执行命令 `./appcompat.sh --dex-file=test.apk`，在终端上会输出三个名单每个 API 的详细调用处:  #1: Linking dark greylist Landroid/os/SystemProperties;->get(Ljava/lang/String;)Ljava/lang/String; use(s):         Ltmsdkobf/gv;->a(Ljava/lang/String;)Ljava/lang/String;   #2: Linking dark greylist Landroid/os/SystemProperties;->get(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String; use(s):         Ltmsdkobf/gp;->b(Landroid/content/Context;)Ljava/lang/String;    ....  

## 2.3 非 SDK API 适配 ##
经过上一步扫描出应用内非 SDK API 调用之后，接下来就可以直接开始适配。适配的原则是优先黑名单和深灰名单，浅灰名单在官方未有替代 API 之前可以暂时不适配，在 Android P 上运行也不会有任何问题。扫描完成之后，不出意外大家应该会有三类需要适配的 API 调用：
1. 应用代码本身调用到了非 SDK API 接口； 针对应用代码本身调用到了非 SDK API 接口，用的比较频繁的例如 `SystemProperties.get`，就需要去寻找另外一个可以替代的合法 API，如果找不到就只能认为该 API 调用失败从而走失败逻辑，如果实在必须要用到该 API 就尽早去向 google 申请移动到浅灰名单中。  
2. 第三方库调用到了非 SDK API 接口； 针对第三方库调用到了非 SDK API 接口，解决办法当然是直接查询相关资料或者联系库提供方，确认是否有适配 Android P 新版本的 SDK。还有需要提到的一点，就算更换适配完成的第三方 SDK 后，仍然可能会在同一地方扫描出非 SDK API 的调用，这是因为适配工程师只是在调用处加了一个 try-catch 保护逻辑，虽然这样也勉强叫做适配完成，但是还是强烈建议大家使用如下的适配方式：  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {  // Android P or above  } else {  // below Android P  } 严格按照上面的适配方案，扫描工具就不会再扫描出此处的非 SDK API 调用，我们也无需每次都去确认所有非 SDK API 调用处都加了保护逻辑。  当然如果第三方库没有适配也没有近期适配的意向，目前有两种方法：第一种是屏蔽入口；第二种是反编译 SDK，在关键地方加上适配代码；  
3. Android 官方库调用到了非 SDK API 接口； 没错！Android 官方库也会被扫描出非 SDK API 调用，针对这种情况，需要分情况讨论：
![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fw4bw99gwjj20tz00twep.jpg)  
该 API 调用查看 v7 support 包源码可以发现已经被 try-catch 住了，测试了相关类也可以正常运行，而且在适配过程中升级 rc 版本的 support-v7 包会导致应用编译不过，所以目前 QQ 音乐暂时认定无需升级到最新版本的 support-v7。除上面介绍的特殊情况之外还是建议更换最新版本的官方 SDK。

# 三、电源管理改进 #
Android P 上对电源管理又做了一系列的改进措施，不管应用 targetApi 版本是否已经升级到 P，系统都会依据应用最近的使用时间和频率来给应用进行待机分组，然后根据应用所属群组限制应用可以访问的资源。
## 3.1 应用待机群组 ##
目前总共有五类分组：  

1. 活跃：  一般为正在使用或者在前台运行的应用，例如：
 - 应用启动一个 Activity；
 - 应用正在运行前台 Service；
 - 应用的同步适配器关联上了一个前台应用；
 - 用户点击了应用的一个通知； 系统不会对该类应用有任何的限制；
2. 工作集：  应用经常运行，但是当前未属于活跃状态就会被归属于工作集，该群组的应用在运行作业和触发闹钟方面会被施加轻度的限制；
3. 常用：  应用如果被定期使用，但不是每天的话就会被归到该工作群组。该群组的应用在运行作业和触发闹钟方面会被施加较强的限制，FCM 消息数量也会有相关限制；
4. 极少使用：  应用如果不经常使用就会被归到该工作群组，系统会对该群组应用运行作业、触发闹钟和接收高优先级别 FCM 的消息能力方面有严格的限制；
5. 从未使用：  安装但从未被使用过的应用会被归到该工作群组，该工作群组的应用会被施加极其严格的限制；  

更加详细的表述可以参考官网：[App Standby ](https://developer.android.com/about/versions/pie/power)，`UsageStatsManager.getAppStandbyBucket()` 函数来获取当前所属的应用群组，借助这个结果来更好的提升自己的打开频率，同时可以借助此来模拟处于不同群组能否正常工作。另外，位于低电耗模式白名单中的应用不适用基于应用待机群组的限制。  
## 3.2 省电模式改进 ##
Android 9 对省电模式又做了很多改进，开启省电模式之后会有如下限制：

1. 系统会更加积极的将应用置于待机模式，不管应用是否空闲；
2. 后台执行限制将适用于所有应用，无论他们的 targetApi 是多少；
3. 屏幕关闭时，位置服务可能被停用；
4. 后台应用没有网络访问权限；

这里需要重点介绍一下后台执行限制，该限制于 Android O 版本引入，主要是为了优化 Android 在多应用多服务运行时，系统负载过大会杀死后台音乐播放等服务导致用户体验下降的问题，它默认只对 targetApi 大于等于 26 的应用生效。目前用户可以通过设置页面对任意应用施加后台执行限制，后台执行限制会对应用有两方面的影响：

1. 后台服务限制：  处于前台（可见、具有前台服务或者关联到前台应用）或临时白名单（处理高优先级 FCM、接收短信等广播或者执行通知的 `PendingIntent`）时，应用可以自由创建和运行前台与后台服务。 进入后台时，在一个持续数分钟的时间窗内，应用仍可以创建和使用服务，但是超过该时间之后再通过 `startService` 去启动一个服务就会抛出 `java.lang.IllegalStateException: Not allowed to start service Intent` 的错误，解决办法是使用 `startForegroundService` 或者 `JobIntentService`；
2. 广播限制:  针对 Android O 和之上的应用无法继续在其清单中为隐式广播注册广播接收器。  

# 四、Apache HTTP client 相关类找不到 #
将 compileSdkVersion 升级到 28 之后，如果在项目中用到了 Apache HTTP client 的相关类，就会抛出找不到这些类的错误。这是因为官方已经在 Android P 的启动类加载器中将其移除，如果仍然需要使用 Apache HTTP client，可以在 Manifest 文件中加入：
```xml
<uses-library android:name="org.apache.http.legacy" android:required="false"/>
```
或者也可以直接将 Apache HTTP client 的相关类打包进 APK 中。
除上面两种适配方式外，QQ 音乐目前采用了另外一种方式。在音乐项目中，我们已经将使用 Apache HTTP client 的模块单独抽离到了一个 module 中，所以暂时只需要保持 module 中的 compileSdkVersion 在 28 以下即可正常编译运行。

# 五、其余适配 #
## 5.1 前台 Service ##
在 Android P 中，如果 targeSdkVersion 升级到 28，使用前台 Service 必须要申请 FOREGROUND_SERVICE 权限，如果没有申请该权限，系统会抛出 SecurityException，该权限为普通权限，申请自动授予应用。

## 5.2 隐私安全保护 ##
1. Build.SERIAL 标识修改：在 Android P 中，对隐私保护又做了更加严格的要求。在某些应用中为了识别手机的唯一性可能会用到 Build.SERIAL 这个标识，但这个标识在 Android P 中已经被设置成了 UNKNOWN，所以会直接导致该功能出现异常。
2. 多进程 webview 信息访问限制：在 Android P 中为了提升系统的安全性，用户无法在多进程的 webview 中共享数据目录，该目录下存储的是一些 cookies、Http 缓存和其他一些永久、临时的缓存。当下不少应用会把 webview 放在另一个进程中打开以避免内存泄漏，但是他们 cookies 的设置往往还是在主进程中，所以开发者需要仔细排查自己的应用是否有这么使用，webview 相关运行是否正常等。

## 5.3 com.android.internal 包下某些类找不到 ##
升级到 28 之后，应用编译后抛出 com.android.internal 包下面有些类找不到的异常，经过查找发现这些类已经从 SDK 中移除。针对这种情况目前有两种处理办法：

1. 移除该类的调用逻辑；
2. 在应用中新建一个同名类，将被移除类的所有代码逻辑复制到新建类中（必要时可能需要将被移除类相关类同时拷贝一份到应用中），然后将应用中所有相关 import 引用直接修改成新建类的包名引用即可；