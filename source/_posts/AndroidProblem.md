---
title: Android疑难问题总结
date: 2019-01-03 16:46:29
tags: android
categories : 技术
---

## Android疑难问题总结

使用WindowManager的时候 我们在`removeView(view)`的时候 会事先判断`view.isAttachedToWindow`  而我遇到的问题是，我再`removeView(view)`之后再去判断`view.isAttachedToWindow` 其值仍然为**true** ,这就令我很是费解。  
##### 解决方法：
`windowManager.removeViewImmediate(view)`之后我们再去判断`view.isAttachedToWindow` 其值就变成了**false**

***
<!--more-->

## 一个topActivity解决了多少问题

很多时候再很多的情况下我们想要去操作我们的 **输入法显示或者隐藏** 
但是有时在`fragment` 或者 `dialog` 等情况下，普通的形式根本不生效。  
这里我的处理方式是，在任情况下，都去调用`topActivity.hideSoft()`方法。
而不是去`dialog.hideSoft()` 或者 `fragment.hideSoft()`

##  git pull 仓库 refusing to merge unrelated histories

远程仓库与本地仓库存在不相关联的历史

```bash
git pull origin master --allow-unrelated-historie
```







持续更新...