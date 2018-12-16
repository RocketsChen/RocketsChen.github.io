---
layout:     post
title:      Taptic-Engine 微震动
subtitle:   触感
date:       2018-12-17
author:     RocketsChen
header-img: img/post-bg-touch.jpg
catalog: true
tags:
- iOS
- Taptic-Engine
- 旧博客迁移
---

> 体验过Apple Watch 的小伙伴肯定知道，很多操作产生微震动（短震）是一个非常好的用户体验，所以就想是否能用到手机上。答案是肯定的，而且很多优秀的APP已经拥有类似的体验。

##### 谷歌，百度下来发现关于Taptic-Engine文档少的可怜🤕，不过还是找到一篇很好的参考博客：[iOS——关于-Taptic-Engine-震动反馈](http://zhoulingyu.com/2017/01/16/iOS%E2%80%94%E2%80%94%E5%85%B3%E4%BA%8E-Taptic-Engine-%E9%9C%87%E5%8A%A8%E5%8F%8D%E9%A6%88/)

#### Taptic-Engine是什么？
> Taptic Engine 是苹果推出的全新震动模块,目前支持Apple Watch，iPhone 6s以上的手机。

#### Taptic-Engine调用
*  导入 #import <AudioToolbox/AudioToolbox.h>
* ###### 调用长震
  > AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
   `__OSX_AVAILABLE_STARTING(__MAC_10_5,__IPHONE_2_0);` (版本)

* ###### 调用普通短震，3D Touch 中 Peek 震动触感
  > AudioServicesPlaySystemSound(1519);

* ###### 调用普通短震，3D Touch 按压弹出触感
  > AudioServicesPlaySystemSound(1520);

* ###### 调用三次短震
  > AudioServicesPlaySystemSound(1521);

##### 但以上方法均未在 Apple 的 Documents 中描述。显然，这是调用了一些私有API 。


> iOS10 引入了一种新的、产生触觉反馈的方式， ***UIImpactFeedbackGenerator：用来给用户反馈当UI元素之间发生产生影响*** 。Apple 对于 UIImpactFeedbackGenerato 有一篇[介绍文档](https://developer.apple.com/reference/uikit/uifeedbackgenerator#2555399)UIFeedbackGenerator 可以帮助你实现 haptic feedback。它的要求是：支持 Taptic Engine 机型 (iPhone 7 以及 iPhone 7 Plus).app 需要在前台运行系统 Haptics setting 需要开启。

````
   UIKIT_CLASS_AVAILABLE_IOS_ONLY(10_0)

    if (@available(iOS 10.0, *)) {
        UIImpactFeedbackGenerator *generator = [[UIImpactFeedbackGenerator alloc] initWithStyle: UIImpactFeedbackStyleHeavy];
        [generator prepare];
        [generator impactOccurred];
    }
````

#### UIImpactFeedbackGenerator提供了三种Style：`Light`、`Medium`、`Heavy`

````
typedef NS_ENUM(NSInteger, UIImpactFeedbackStyle) {
    UIImpactFeedbackStyleLight,
    UIImpactFeedbackStyleMedium,
    UIImpactFeedbackStyleHeavy
};
````

  #### 总结一下，希望同样的代码能在更多的机型上实现短振，建议使用 AudioServicesPlaySystemSound(1519)。不过可能会涉及到调用私有 API。安全起见，希望使用 UIImpactFeedbackGenerator。

> 在这里对`Taptic-Engine`只做简单的使用介绍，如若想更为详细的了解，请前去文中博客地址查看。



#### 源文档链接：[Taptic-Engine 微震动](https://www.jianshu.com/p/5c990900d056)

