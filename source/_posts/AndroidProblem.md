---
title: Android疑难问题总结
date: 2019-01-03 16:46:29
tags: 技术
categories : 技术
---

### Android疑难问题总结
***

使用WindowManager的时候 我们在`removeView(view)`的时候 会事先判断`view.isAttachedToWindow`  而我遇到的问题是，我再`removeView(view)`之后再去判断`view.isAttachedToWindow` 其值仍然为**true** ,这就令我很是费解。  
##### 解决方法：
`windowManager.removeViewImmediate(view)`之后我们再去判断`view.isAttachedToWindow` 其值就变成了**false**

***

持续更新...
<!--more-->