---
title: Jetpack Navigation
date: 2019-02-13 15:28:38
tags: jetpack
categories: 技术
---

### Navigation

先介绍概念在举一个栗子

#### 优势

* 自动处理fragment transactions
* 正确处理 **up** and **back**
* 动画和过度的默认行为
* Deep linking as a first class operation // 咋翻译啊？
* 仅用少量代码实现 navigation UI patterns (like navigation drawers and bottom nav)
* 导航时安全的传递信息
* 在Android Studio中支持可视化编辑导航

#### 主要由三部分组成

**`NavGraphFragment`** 导航界面的容器，首先要展示一个fragment必须要有一个容器来承载它，之前可能是framelayout或者RelativeLayout 而现在换成了是 NavGraphFragment 这些个fragment都展示在NavGraphFragment上面

* app:defaultNavHost="true"这个属性意味着你的NavGraphFragment将会 拦截系统Back键的点击事件 
* 同时 必须重写 Activity的 onSupportNavigateUp() 方法
* 
```
override fun onSupportNavigateUp()
        = findNavController(R.id.nav_host_fragment).navigateUp()
```

* app:navGraph="@navigation/nav_graph_main"这个属性就很好理解了，它会指向一个navigation_graph的xml文件，NavGraphFragment就会 导航并展示对应的Fragment。

**`Navigation Graph (New XML resource) `** 用于声明导航结构图，包含有fragment的动作，携带的参数等

**`NavController (Kotlin/Java object)`** 发起动作，导航你想导航到的任何位置

#### 获取NavController的三种方式：

* Fragment.findNavController()
* View.findNavController()
* Activity.findNavController(viewId: Int)

#### 导航时添加转场动画
 这里面用的是DSL形式，得意~~

```
val options = navOptions {
    anim {
        enter = R.anim.slide_in_right
        exit = R.anim.slide_out_left
        popEnter = R.anim.slide_in_left
        popExit = R.anim.slide_out_right
    }
}
view.findViewById<Button>(R.id.navigate_destination_button)?.setOnClickListener {
    findNavController().navigate(R.id.flow_step_one_dest, null, options)
}
```
或者写在Action标签里面

```
<action android:id="@+id/next_action"
        app:destination="@+id/flow_step_one"
        app:enterAnim="@anim/slide_in_right"
        app:exitAnim="@anim/slide_out_left"
        app:popEnterAnim="@anim/slide_in_left"
        app:popExitAnim="@anim/slide_out_right" />
```

#### 安全的获取参数

在fragment标签下增加参数

```
<argument
        android:name="flowStepNumber"
        app:argType="integer"
        android:defaultValue="1"/>
```

相比于之前在fragment中获取参数的这种非类型安全（type safe）的形式

```
val username = arguments?.getString("usernameKey")
```

Navigation提供了一种安全形式

如果声明了argument 那么会自动生成一些代码供开发者使用

```
val safeArgs = FlowStepFragmentArgs.fromBundle(arguments)
val flowStepNumber = safeArgs.flowStepNumber
```

