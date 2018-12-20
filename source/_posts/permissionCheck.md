---
title: Android权限检查(版本适配)
date: 2018-11-10 09:54:44
tags: [Android,权限]
categories: Android

---
昨天工作中遇到一个需要判断权限来完成的功能，遇到了一些小问题，记录一下。

权限的检查方式是需要区分版本的，在Android 6.0(23)以前使用的是`PermissionChecker.checkSelfPermission`，而在6.0以后用的直接是`ContextCompat.checkSelfPermission`。  
因为在6.0之后有了动态授权，所以不能够直接通过判断`AndroidManifest.xml`里面的权限是否写了来判断，一定要单独做权限检测，以免发生预料之外的BUG。

```java
//权限检测的例子
int permission = 0;
try {
	PackageInfo info = ctx.getPackageManager().getPackageInfo(ctx.getPackageName(), 0);
	int targetSdkVersion = info.applicationInfo.targetSdkVersion;
	if (targetSdkVersion >= 23) {
		permission = ContextCompat.checkSelfPermission(ctx, "android.permission.WRITE_EXTERNAL_STORAGE");
	} else {
		permission = PermissionChecker.checkSelfPermission(ctx, "android.permission.WRITE_EXTERNAL_STORAGE");
	}
} catch (Exception e) {
	e.printStackTrace();
}
if (permission != PackageManager.PERMISSION_GRANTED) {
	log("WRITE_EXTERNAL_STORAGE is not allow");
	return false;
}
```

P.S  
另有一种邪教说法是，使用try-catch来强行捕获需要权限的动作，在catch里面判断是不是没有权限。这种方式貌似存在版本适配问题，慎用

