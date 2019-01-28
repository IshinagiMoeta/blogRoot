---
title: 华为手机 Register too many Broadcast Receivers 问题解决方法   
date: 2018-10-15 14:48:08  
tags: [Android,适配]  
categories: Android  

---
最近在集成Vungle视频广告的过程中遇到了一个蛮奇怪的问题，是一个会导致崩溃的Bug，当App运行到一定的时间后，突然会自动闪退。毫无预兆，也没有打印我设置的任何可能导致crash的log。很是奇怪，只能去翻25M的日志报告。最后找到了这么一段东西。  

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fw8ww0w36rj214j0lcjv7.jpg)  

关键字段应该是 `Register too many Broadcast Receivers` 这个没错了，谷歌了一下，发现还蛮多人遇到这个问题的。  
导致这个问题的原因是华为5.1和5.1.1上，有一个针对广播注册的限制，HuaWei自家定制的ROM系统中有一个白名单机制，只有加入了白名单的APP才允许注册超过500个BroadcastReceiver，否则就会抛出Register too many Broadcast Receivers的异常。也就是说没有加入该白名单机制的APP最多只能注册500个BroadcastReceiver。  

既然HuaWei定制的ROM系统做了限制，那么肯定是在哪一个类中做了校验操作，根据crash信息可以知道崩溃是发生在LoadedApk类的checkRecevierRegisteredLeakLocked()方法中，由于我们没法拿到HuaWei手机ROM系统中LoadedApk类的具体代码，只能通过反射对比在HuaWei手机和Google Pixel手机上这俩LoadedAPK究竟有和不同，功夫不负有心人，最终发现在HuaWei的ROM的LoadedApk类中定义了一个叫mReceiverResource的成员变量，根据名称我们猜测该属性就是和注册BroadcastReceiver的数量相关的类。在HuaWei手机中的Debug模式下，LoadedApk信息如下所示：  

mReceiverResource是ReceiverResource类型，ReceiverResource内部定义了一个ArrayList类型的成员变量mWhiteList，直接看名字就知道是白名单的意思，mWhiteList中默认添加了"com.tencent.mm"字符串，该字符串就是微信的包名。由此可知HuaWei定制的ROM系统把微信加入了白名单里从而允许微信可以注册超过500个BroadcastReceiver。为了验证mReceiverResource是用来控制注册BroadcastReceiver的数量的，我把HuaWei手机上的微信卸载了，然后把刚刚创建的工程包名改为com.tencent.mm，紧接着再运行工程，这时候果然可以注册超过500个BroadcastReceiver了，打印日志如下所示：  

从实验结果来看，果真的如我们前边猜测的那样，mReceiverResource就是用来控制非白名单中的APP最多只能注册500个BraodcastReceiver的开关，只要把我们APP的packageName添加到白名单中，也就跳过了HuaWei手机的限制。  

下边就是通过反射把我们APP的packageName添加到mWhiteList中的操作，转载于[这里](https://github.com/llew2011/HuaWeiVerifier)。  
  

```java
public class LoadedApkHuaWei {

    public static void hookHuaWeiVerifier(Context baseContext) {
        try {
            if (null != baseContext && "ContextImpl".equals(baseContext.getClass().getSimpleName())) {
                IMPL.verifier(baseContext);
            } else {
                Log.w(LoadedApkHuaWei.class.getSimpleName(), "baseContext is't instance of ContextImpl");
            }
        } catch (Throwable ignored) {
            // ignore it
        }
    }


    private static final HuaWeiVerifier IMPL;

    static {
        final int version = android.os.Build.VERSION.SDK_INT;
        if (version >= 26) {
            IMPL = new V26VerifierImpl();
        } else if (version >= 24) {
            IMPL = new V24VerifierImpl();
        } else {
            IMPL = new BaseVerifierImpl();
        }
    }

    private static class V26VerifierImpl extends BaseVerifierImpl {

        private static final String WHITE_LIST = "mWhiteListMap";

        @Override
        public void verifier(Context baseContext) throws Throwable {
            Object whiteListMapObject = getWhiteListObject(baseContext, WHITE_LIST);
            if (whiteListMapObject instanceof Map) {
                Map whiteListMap = (Map) whiteListMapObject;
                List whiteList = (List) whiteListMap.get(0);
                if (null == whiteList) {
                    whiteList = new ArrayList<>();
                    whiteListMap.put(0, whiteList);
                }
                whiteList.add(baseContext.getPackageName());
            }
        }
    }

    private static class V24VerifierImpl extends BaseVerifierImpl {

        private static final String WHITE_LIST = "mWhiteList";

        @Override
        public void verifier(Context baseContext) throws Throwable {
            Object whiteListObject = getWhiteListObject(baseContext, WHITE_LIST);
            if (whiteListObject instanceof List) {
                List whiteList = (List) whiteListObject;
                whiteList.add(baseContext.getPackageName());
            }
        }
    }

    private static class BaseVerifierImpl implements HuaWeiVerifier {

        private static final String WHITE_LIST = "mWhiteList";

        @Override
        public void verifier(Context baseContext) throws Throwable {
            Object receiverResourceObject = getWhiteListObject(baseContext, WHITE_LIST);
            if (receiverResourceObject instanceof String[]) {
                String[] whiteList = (String[]) receiverResourceObject;
                List<String> newWhiteList = new ArrayList<>();
                newWhiteList.add(baseContext.getPackageName());
                Collections.addAll(newWhiteList, whiteList);
                FieldUtils.writeField(receiverResourceObject, WHITE_LIST, newWhiteList.toArray(new String[newWhiteList.size()]));
            }
        }

        Object getWhiteListObject(Context baseContext, String whiteList) throws Throwable {
            Field receiverResourceField = FieldUtils.getDeclaredField("android.app.LoadedApk", "mReceiverResource", true);
            if (null != receiverResourceField) {
                Field packageInfoField = FieldUtils.getDeclaredField("android.app.ContextImpl", "mPackageInfo", true);
                if (null != packageInfoField) {
                    Object packageInfoObject = FieldUtils.readField(packageInfoField, baseContext);
                    if (null != packageInfoObject) {
                        Object receivedResource = FieldUtils.readField(receiverResourceField, packageInfoObject, true);
                        if (null != receivedResource) {
                            return FieldUtils.readField(receivedResource, whiteList);
                        }
                    }
                }
            }
            return null;
        }
    }

    private interface HuaWeiVerifier {
        void verifier(Context baseContext) throws Throwable;
    }
}

```

LoadedApkHuaWei只对外暴露一个hookHuaWeiVerifier()方法，该方法内部实现是根据HuaWei手机不同的版本做了不同的Hook操作。该API使用非常简单，需要在自己APP中显示的定义一个Application，然后在Application的onCreate()方法中调用LoadedApkHuaWei的hookHuaWeiVerifier()方法就行了，如下所示：

```java
public class SimpleApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        LoadedApkHuaWei.hookHuaWeiVerifier(getBaseContext());
    }
}
```

以上操作就可以把我们APP添加进HuaWei ROM中的白名单里了。