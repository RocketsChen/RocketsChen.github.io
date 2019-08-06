---
layout:     post
title:      iOS 群头像框架
subtitle:   CDDGroupAvatar
date:       2019-08-06
author:     RocketsChen
header-img: img/post-bg-group_avatar.jpg
catalog: true
tags:
- 框架
- 群头像
- CDDGroupAvatar
- iOS
---
# iOS 群头像框架 — CDDGroupAvatar

## 前言：

#### 谈到APP里面群头像功能的实现，很容易想到获取展示位数头像，并根据UI绘制图片。从开发角度上讲，为了更好实现这个功能，还需要考虑缓存，占位，以及线程和图片的绘制。为了可以更优雅的实现这个功能和减少侵入性，我设计了这个框架。并希望能让群头像的实现尽可能的能像`SDWebImage`加载一张图片那么简单。

#### 接下来我会结合Demo来谈谈这个框架的设计。

## CDDGroupAvatar框架：

* 1：使用
* 2：设计初衷
* 3：实现分析
* 4：缓存与更新机制
* 5：集成方式
* 6：不足或后续优化



### 一.使用

为什么要把使用放在第一个来讲，因为我想展现出`CDDGroupAvatar`的简单和方便，由于功能自身的难度性并不是很大，如果封装出来的轮子还是很繁琐，那还不如自己写个方法类来的轻巧。

#### 1：调用


```
[self.avImageViewW8 dc_setImageAvatarWithGroupId:@"avImageViewW8" Source:_groupNum8];
```
分类方法调用传入群头像id和群头像数据源数组，完事！

来看看展示

![-w200](http://ww1.sinaimg.cn/large/006tNc79ly1g5mknwvluuj30gm08awkz.jpg)

#### 2：？不对还缺点什么

在CDDGroupAvatar中提供了`DCAvatarManager`类。可以统一设置，baseURL、头像类型、占位图等等。这些属性都或多或少的可以帮助你简化调用。一般来说加载头像后台返回的数据是xxxx.jpg/.png，并不是一个完整的头像地址需要我们自行去拼接，baseUrl属性就可以很好地处理，在初始化的时候设置好统一的拼接头，接下来你只需这样。

```
// 初始化
[DCAvatarManager sharedAvatar].baseUrl = @"http://xxxxxx";

// 数组的格式
_groupNum2 = @[@"006tNc79gy1g56or92vvmj30u00u048a.jpg",@"006tNc79gy1g56mcmorgrj30rk0nm0ze.jpg"];

// 加载头像
baseUrl + _groupNum2[index] 
```

不用担心不同拼接头的情况，头像本身就是一个完整的链接的话，默认就不会再拼接baseUrl属性。

下面还有一些其他的参数

```
#pragma mark - DCAvatarManager属性

/* 请求的baseURL。这应该只包含URL的共体部分，例如http://www.example.com。 */
@property (strong , nonatomic)NSString *baseUrl;

/* 头像类型枚举(默认微信样式) */
@property (assign , nonatomic) DCGroupAvatarType groupAvatarType;


/* 头像背景(默认微信背景色)  */
@property (nonatomic, strong) UIColor *avatarBgColor;


/* 一次性设置小头像加载失败的占位图 ： 权重低于类方法中的placeholder属性 placeholderImage < (id)placeholder  */
@property (nonatomic, strong) UIImage *placeholderImage;


/* 微信和QQ群内小头像间距（默认值：2）  */
@property (nonatomic, assign) CGFloat distanceBetweenAvatar;


/* 微博外边圈宽度（默认：10）  */
@property (nonatomic, assign) CGFloat bordWidth;


#pragma mark - SDWebImage依赖框架属性

/* sd加载群内单个小图片属性 (默认：SDWebImageRetryFailed | SDWebImageLowPriority ) */
@property (assign , nonatomic) SDWebImageOptions sdImageOptions;
```

#### 3：如果你不想设置统一的占位图，例如根据性别显示不同占位你也可以这样

placeholder：占位图属性 你可像这样去传值@[@"p1",@"p2"] 或者 @[image1,image2] 当然@[@"p1",image2]也行。在框架内部会在合适的位置显示。

```
// 传入无效地址测试 Placeholder属性
[self.avImageViewW2 dc_setImageAvatarWithGroupId:@"avImageViewW2" Source:@[@"1",@"2"] itemPlaceholder:@[@"man",@"woman"] options:0 completed:nil];
```
![-w200](http://ww1.sinaimg.cn/large/006tNc79ly1g5ml1fklr1j30cg08ajup.jpg)

注！权重大于 统一设置的placeholderImage属性


#### 4：可选和回调

就像图片加载框架`SDWebImage`一样我同样提供了加载可选和数据的回调，方便使用者自行选择。
CDDGroupAvatar提供两种加载模式`DCGroupAvatarCacheType`：

* DCGroupAvatarCachedDefault (默认) ： 走缓存 取到返回 没有则获取最新
* DCGroupAvatarCachedRefresh  ： 先读缓存再获取最新

```
[self.avImageViewW4 dc_setImageAvatarWithGroupId:@"avImageViewW4" Source:_groupNum4 itemPlaceholder:@"place" options:DCGroupAvatarCachedRefresh completed:nil];
```

CDDGroupAvatar提供`GroupImageBlock`回调：

```
[self.avImageViewW9 dc_setImageAvatarWithGroupId:@"avImageViewW9" Source:_groupNum9 completed:^(NSString *groupId, UIImage *groupImage, NSArray<UIImage *> *itemImageArray, NSString *cacheId) {
   NSLog(@"groupId：%@ -- 群头像：%@ 群内拼接小头像数组：%@ -- cacheId：%@",groupId, groupImage, itemImageArray , cacheId);
}];
```


#### 5：同样`CDDGroupAvatar`也支持Button，调用方式和ImageView一样，还可以根据UIControlState属性去展示。

```
// Image
[self.avaButton dc_setImageAvatarWithGroupId:@"avaButton" Source:_groupNum9 forState:0];
// Background
[self.avaBgButton dc_setBackgroundImageAvatarWithGroupId:@"avaBgButton" Source:_groupNum4 forState:0];
```

#### 6：支持什么格式的群头像

目前`CDDGroupAvatar`支持三种样式的群头像：微信（默认）、QQ、微博，你可以通过`groupAvatarType`去选择

来张全家福：

![-w200](http://ww4.sinaimg.cn/large/006tNc79ly1g5mlql5ti4j30ds0u416w.jpg)


## 二：设计初衷：

为了减少浸入性，功能内聚，可以使群头像的实现尽可能的能像`SDWebImage`加载一张图片那么简单。同时更进一步的去学习SDWebImage的源码。

## 三：微信样式分析：

接下来，我以微信样式分析一下整个框架的运行流程

![](http://ww1.sinaimg.cn/large/006tNc79gy1g5msrr7v4wj30ub0kadgz.jpg)

在如上流程中，主要有图片的批量下载，尺寸和位置计算，图片的绘制，裁剪，缓存和回调处理，以及如何优雅的兼顾Image和Button的调用。特别是一开始cache为空的时候批量加载群内成员的头像和完成后回调，以及这个过程中占位图和缓存的处理，还要有一些挑战的。

从上面最全的一张展示图来看，微信和QQ这两种群头像图片的绘制还不算难，中规中矩，计算好小头像的大小，和位置绘制。在设计过程中，微博样式稍微让我有些头疼。


#### 1：在拿到群内所有成员头像后，首先需要先确定的就是小头像的尺寸

#### 2：其次是确定每个头像所在的位置（x,y）坐标

#### 3：根据他们的frame进行群头像的绘制


#### 4：图片的属性处理

为了避免绘制的图片和地址图片的属性不等，我在等图片绘制成功后会有一次关于图片属性的调整，默认以群成员地址加载出来的属性为准。

#### 5：缓存和回调

在批量加载图片的的统一回调中我新增了一个属性`BOOL succeed`，加载的成功性。用来判断是否进行存储。如果群内的所有头像都加载失败，那就不用存储了，直接返回上次的缓存图片或者默认占位图片。这样既可以避免重复存储，也能一定意义上的优化显示。

### 四.缓存与更新机制

说到缓存，这是在网络加载过程中必不可少的一步，针对绘制成功的群头像会做一次缓存和磁盘的存储。缓存方面利用的事 SDWebImage 框架中的Cache。既然 SDWebImage 本身的加载就自带缓存，为什么还需要再做一遍呢？节约性能这样可以避免重复计算和绘制。重复的计算和绘制都是会损耗一定的性能。

缓存key是如何定义的？又如何能保证及时更新？

CDDGroupAvatar 中的cacheId是一段字符属性的拼接然后进行MD5。其中以群头像id，群内头像的数量（以最大截取数），群头像数据源的最后一个对象（监听变动），和一些边距，背景颜色等属性进行拼接。

```
id%@_num%zd_lastObj%@_distance%.f_bordWidth%.f_bgColor%@
```

这样可以在兼顾群内成员变动，主题切换，甚至自定义边距改变的同时能后及时的对已经绘制的好的缓存中群头像进行更新。


### 五.Demo的演示

clean操作：清除所有内存缓存和磁盘缓存

![GIF演示](http://ww2.sinaimg.cn/large/006tNc79gy1g5nkb2qnyqg30hs0b4e86.gif)

如上截图中的样式在demo演示项目中均有实现。

### 六.集成方式

* CocoaPods

1：在 Podfile 中添加 pod 'CDDGroupAvatar'，执行 pod install 或 pod update。

* 手动导入

1：将demo项目中的 CDDGroupAvatar 文件夹所有内容拖入你的工程中。
2：集成 SDWebImage 框架。

* 用法

1：导入`#import <CDDGroupAvatar/DCAvatar.h>`可以拥有全部功能。
2：调用对应控件的类方法。
3：如果有使用上的疑问，可以下载演示demo进行查看。

### 七.不足或后续优化

1：项目重度依赖`SDWebImage`框架。不管是图片的加载和缓存都是用的此框架，为什么不去减少对外部框架的自己实现，成本太高，简易的自己去实现一个加载图片和缓存功能，还不见得能比依赖此框架来的好。

2：微博类型的绘制模式不够完美。我之所以把此类型的实现拿出来分析，就是因为我觉得现在的处理方式不够好，有优化的空间。特别是在相交的处理方式上，我更愿意一次绘制的时候设置相切路径。


后续：我重点会把第二个优化一下。

![](http://ww3.sinaimg.cn/large/006tNc79gy1g5nki5cskfj302k02ha9z.jpg)

最后还是建议有需要去下载源码。

## 工具：

在设计demo和测试的过程中用到如下两款工具：

*  [iPic](https://toolinbox.net/iPic/)

iPic - Markdown 图床、文件上传工具，一个很方便的软件一直在使用，非常推荐，App Store就可以下载还是免费的

* [JPFPSStatus](https://github.com/joggerplus/JPFPSStatus)

JPFPSStatus：记录当前屏幕的帧数（FPS），joggerplus大神的开源，GitHub就有源码，集成和使用非常方便。

## 参考：

[SDWebImage](https://github.com/SDWebImage/SDWebImage)


## 地址：

Github : [CDDGroupAvatar](https://github.com/RocketsChen/CDDGroupAvatar)

