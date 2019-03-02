---
title: Jetpack 全家桶
date: 2019-02-12 14:20:53
tags: jetpack
categories: 技术
---

### Jetpack 全家桶

#### DataBinding

[单项绑定](https://o0o0oo00.github.io/2019/02/11/databinding/#more)  
双向绑定 我们知道 正常的情况下是数据驱动视图，也就是数据改变了，UI也会跟着更新，但是如果是一个EditText 输入框中的内容改变了，我们的数据是不会跟着改变的，这就引申出了**双向绑定** UI视图的变化也会驱动数据的改变，

#### Lifecycle
**`Lifecycle`** 是一个类，它持有关于组件（如 Activity 或 Fragment）生命周期状态的信息，并且允许其他对象观察此状态。

##### 使用方式：

* 在UIController（Activity/Fragment）中添加**观察者**`getLifecycle().addObserver(mPresenter);//添加LifecycleObserver`
* 在外部的一个类去实现`LifecycleObserver` 进而就可以去监听到UIController 的各个生命周期的状态改变了。

##### 原理以及重要的类

* LifecycleObserver接口（ Lifecycle观察者）：实现该接口的类，通过注解的方式，可以通过被LifecycleOwner类的addObserver(o: LifecycleObserver)方法注册，被注册后，LifecycleObserver便可以观察到LifecycleOwner的生命周期事件。


* LifecycleOwner接口（ Lifecycle持有者）：实现该接口的类持有生命周期(Lifecycle对象)，该接口的生命周期(Lifecycle对象)的改变会被其注册的观察者LifecycleObserver观察到并触发其对应的事件。

`LifecycleOwner` 通常为`Activity`/`Fragment` 而目前AndroidX中已经对其做了封装，我们可以直接使用 持有 的**Lifecycle对象**

* LifecycleRegistry 是 Lifecycle 的一个子类 我们正常通过`getLifecycle()`获取的Lifecycle对象 就是它。

* 当被观察者LifecycleOwner生命周期改变的时候 LifecycleRegistry 就会逐个通知每一个注册的LifecycleObserver ，并执行对应生命周期的方法。

#### ViewModel

**`ViewModel`** 应尽量保证 纯的业务代码，不要持有任何View层(Activity或者Fragment)或Lifecycle的引用

* ViewModel的扩展类会自动保留其数据，如果Activity被重新创建了，它会收到和之前相同的ViewModel实例。当所属Activity终止后，框架调用ViewModel的onCleared()方法释放对应资源。

* ViewModel是有一定的 作用域 的，它不会在指定的作用域内生成更多的实例，从而节省了更多关于 状态维护（数据的存储、序列化和反序列化）的代码。

* 如果ViewModel的作用域为Activity那么 不同的fragment 之间就可以 实现 **数据状态的共享**

* 在对应的作用域内，保正只生产出对应的**唯一实例**，多个Fragment维护相同的数据状态，极大减少了UI组件之间的数据传递的代码成本。


#### LiveData

又是观察者模式，允许我们去观察一个变量，当变量发生改变的时候，我们会接收到改变回调，以便于做出响应。

```
val live = MutableLiveData<String>()
live.observe(this, Observer {
    Toast.makeText(this,it,Toast.LENGTH_SHORT).show()
})
```

* 首先我们在调用LiveData.observer()方法时，传递的第一个参数Acitivity实际被向上抽象成为了 LifecycleOwner，第二个参数Observer实际就是我们的观察后的回调。

* RxJava在使用过程中，避免内存泄漏是一个不可忽视的问题，因此我们一般需要借助三方库比如RxLifecycle、AutoDispose来解决这个问题。

* 而反观LiveData，当它被我们的Activity订阅观察之后 Activity如果finish()掉，LiveData本身会自动“清理”以避免内存泄漏。

* 方法内部实际上将我们传入的2个参数包装成了一个新的 LifecycleBoundObserver对象，它实现了 Lifecycle 组件中的LifecycleObserver接口。

* 为什么LiveData能够 自动解除订阅而避免内存泄漏 了，因为它内部能够感应到Activity或者Fragment的生命周期。

* `livedata.observer()` 与 `livedata.observerForever()`前者 只会接收到 observer.mActive == true 时的回调。而后者可以收到所有状态的回调

* 既然LiveData已经能够实现在onDestroy()的生命周期时自动解除订阅，为什么还要多此一举设置一个Active的状态呢？

>Activity并非只有onDestroy()一种状态的，更多时候，新的Activity运行在栈顶，旧的Activity就会运行在 background——这时旧的Activity会执行对应的onPause()和onStop()方法，我们当然不会关心运行在后台的Activity所观察的LiveData对象（即使数据更新了，我们也无从进行对应UI的更新操作），因此LiveData进入 InActive(待定、非活跃)状态，return并且不去执行对应的回调方法，是 非常缜密的优秀设计 。


#### Paging
**`Paging`** 可以使开发者更轻松在 RecyclerView 中 分页加载数据。

