---
layout: wiki
title: Android 远程调试 WebView
categories: Android
description: Android 远程调试 WebView
keywords: Android, debug, WebView
---

使用 Chrome 开发者工具在您的原生 Android 应用中调试 WebView。

在 Android 4.4 (KitKat) 或更高版本中，使用 DevTools 可以在原生 Android 应用中调试 WebView 内容。

TL;DR

- 在您的原生 Android 应用中启用 WebView 调试；在 Chrome DevTools 中调试 WebView。

- 通过 **chrome://inspect** 访问已启用调试的 WebView 列表。

- 调试 WebView 与通过远程调试调试网页相同。


## 配置 WebViews 进行调试

必须从您的应用中启用 WebView 调试。要启用 WebView 调试，请在 WebView 类上调用静态方法 setWebContentsDebuggingEnabled。

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WebView.setWebContentsDebuggingEnabled(true);
}
```

此设置适用于应用的所有 WebView。

**提示：**WebView 调试不会受应用清单中 debuggable 标志的状态的影响。如果您希望仅在 debuggable 为 true 时启用 WebView 调试，请在运行时测试标志。

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    if (0 != (getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE))
    { WebView.setWebContentsDebuggingEnabled(true); }
}
```

在 DevTools 中打开 WebView

**chrome://inspect** 页面将显示您的设备上已启用调试的 WebView 列表。

要开始调试，请点击您想要调试的 WebView 下方的 inspect。像使用远程浏览器标签一样使用 DevTools。

故障排除

在 `chrome://inspect page` 上无法看到您的 WebView？

验证已为您的应用启用 WebView 调试。
在设备上，打开应用以及您想要调试的 WebView。然后，刷新 `chrome://inspect` 页面。
