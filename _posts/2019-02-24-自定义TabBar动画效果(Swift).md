---
layout:     post
title:      自定义TabBar动画效果(Swift)
subtitle:   页面转场
date:       2019-02-24
author:     RocketsChen
header-img: img/post-bg-tabBarS.jpg
catalog: true
tags:
- 转场动画
- TabBar 动画
- iOS
- Swift
---

# 自定义TabBar动画效果 - 页面转场(Swift)

> 前言：之前写过一篇自定义TabBar动画效果的博客[OC版本](http://chendiandian.fun/2018/12/15/%E8%87%AA%E5%AE%9A%E4%B9%89TabBar%E5%8A%A8%E7%94%BB%E6%95%88%E6%9E%9C/)，本篇换成Swift来实现动画。思路大致相同，有需要可以去上篇博客中查看具体的逻辑本篇主要分享一下在Swift中核心代码和与OC中区别。




### 分三步：

#### 1：初始化：

* 初始化TabBar控制器

* 遵守UITabBarControllerDelegate代理协议，实现代理方法

#### 2：点击动画：

##### 核心代码

> 在代理方法tabBarController - didSelect viewController中找到对应选中的tabBarButton并对其做核心动画

##### 核心代码

````
/// 点击动画
func tabBarController(_ tabBarController: UITabBarController, didSelect viewController: UIViewController) {
   
   tabBarButtonClick(tabBarButton: getTabBarButton())
   
}

private func getTabBarButton() -> UIControl {

   var tabBarButtons = Array<UIView>()
   for tabBarButton in tabBar.subviews {
       if (tabBarButton.isKind(of:NSClassFromString("UITabBarButton")!)){
           tabBarButtons.append(tabBarButton)
       }
   }
   return tabBarButtons[selectedIndex] as! UIControl
}
    
private func tabBarButtonClick(tabBarButton : UIControl) {
   
   for imageView in tabBarButton.subviews {
       if (imageView.isKind(of: NSClassFromString("UITabBarSwappableImageView")!)){
           let animation = CAKeyframeAnimation()
           animation.keyPath = "transform.scale"
           animation.duration = 0.3
           animation.calculationMode = CAAnimationCalculationMode.cubicPaced
           animation.values = [1.0,1.1,0.9,1.0]
           imageView.layer.add(animation, forKey: nil)
       }
   }
}
    
````

#### 3：滑动动画：

> 在代理方法tabBarController - animationControllerForTransitionFrom代理方法中做滑动动画。

* 这里可以创建一个类封装tabBarVc的滑动动画，类似如下：

```
/// 滑动动画
func tabBarController(_ tabBarController: UITabBarController, animationControllerForTransitionFrom fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
   
   return DCSlideBarAnimation() //滑动动画
}
```

> 在类DCSlideBarAnimation中去实现具体滑动动画。

##### ❗️DCSlideBarAnimation类中与OC的不同点：期初在设计oc滑动类的时候我设计一个UIRectEdge属性来告诉类我当前选中的控制器Vc和选中前控制器Vc的一个方向偏移作为滑动动画的dx。

##### ❗️在swift中，我取消了这个属性，从而在类的内部去去通过获取当前TabVc的viewControllers的index来比较偏移方向。

##### 相比较更建议采用第二种方法，在上传的代码中我没更改OC原来思路，可以查看源码进行对比。

##### 核心代码

````
// MARK: - 代理方法
extension DCSlideBarAnimation : UIViewControllerAnimatedTransitioning{
    
    /// 设置时间间隔
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
       return 0.15
    }
        
    /// 处理转场
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
       
       guard let fromeVc = transitionContext.viewController(forKey: .from),
             let toVc = transitionContext.viewController(forKey: .to) else { return }
       
        
       guard let tabVc = UIApplication.shared.keyWindow?.rootViewController as? UITabBarController,
             let fromeIndex = tabVc.viewControllers?.index(where: { $0 == fromeVc}),
             let toIndex = tabVc.viewControllers?.index(where: { $0 == toVc}) else { return }
       
       guard let formView = transitionContext.view(forKey: .from),
             let toView = transitionContext.view(forKey: .to) else { return }
       
       let fromFrame : CGRect = formView.frame
       let toFrame : CGRect = toView.frame
    
      
       var offSet : CGVector?
       if toIndex > fromeIndex {
           offSet = CGVector(dx: -1, dy: 0)
       }else{
           offSet = CGVector(dx: 1, dy: 0)
       }
    
       guard let animOffSet = offSet else { return }
       formView.frame = fromFrame
    
       let ofDx : CGFloat = animOffSet.dx
       let ofDy : CGFloat = animOffSet.dy
       toView.frame = CGRect.offsetBy(toFrame)(dx: toFrame.size.width * ofDx * -1, dy: toFrame.size.height * ofDy * -1)
       transitionContext.containerView.addSubview(toView)
    
       let transitionDuration = self.transitionDuration(using: transitionContext)
       UIView.animate(withDuration: transitionDuration, animations: { //动画
           
           formView.frame = CGRect.offsetBy(fromFrame)(dx: fromFrame.size.width * ofDx * 1, dy: fromFrame.size.height * ofDy * 1)
           toView.frame = toFrame;
    
       }) { (_) in
           transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
       }
    }
}

````

#### 效果预览（GIF）
![GIF演示](https://ws4.sinaimg.cn/large/006tKfTcgy1g0hp7o1ym5g30hs0b44qq.gif)

#### OC版地址：[自定义TabBar动画效果 - 页面转场](http://chendiandian.fun/2018/12/15/%E8%87%AA%E5%AE%9A%E4%B9%89TabBar%E5%8A%A8%E7%94%BB%E6%95%88%E6%9E%9C/)

#### 最后两个版本代码的项目地址：[GitHub](https://github.com/RocketsChen/CDDSlideTabBar)

