---
title: autodispose
tags: rxjava
categories: 技术
date: 2019-03-04 16:03:33

---
## [译]AutoDispose 未完



**AutoDispose**是RxJava2的一个工具，在一定的生命周期内，自动解除订阅，以防止因为生命周期的不同导致的内存泄漏。



### 概览

通常情况下（尤其是在移动应用开发中），比如在Android中，Activity执行到onStop的时候，Rx 的订阅需要被取消掉，以防止内存泄漏。于是就有了**AutoDispose**

看起来就想这样简单

```kotlin
myObservable
    .doStuff()
    .as(autoDisposable(this))   // The magic
    .subscribe(s -> ...);
```

如果超出了给定的scope周期，订阅将不再生效，例如界面已经被销毁了，但是网络请求刚刚拿到

### autoDisposable()

我们已经发现了最关键的就是这句话了。它是AutoDispose类中的一个静态的工厂方法。

它有两个重载方法，一个参数是`ScopeProvider`一个参数是`Completable`，他们都返回一个`AutoDisposeConverter<T>`对象，它实现了RxJava Converter 的接口以用于RxJava中的`as()`运算符

### Completable(as a scope) 作用范围

它的作用类似于`takeUntil()`操作符的第二个**Observable**，该运算符接收一个**Observable**，其第一个发射的信号作为完成的通知，因为第二个Observable发射的类型无关紧要，所以简化为发送一个**Completable**，其中的**scope-end**只是发射一个完成事件。AutoDispose中的所有作用域最终都会解析为发出作用域结束通知的**Completable**。

### ScopeProvider

```kotlin
public interface ScopeProvider {
  CompletableSource requestScope() throws Exception;
}
```

他是一个抽象类，继承它的子类对象可以提供一个专属的作用域。一般作用于自定义作用域，“我停止的时候你也必须停止”，此处可以抛出异常，会发送到`onError`中去。

### AutoDisposePlugins

以插件方式自定义AutoDispose行为

#### OutsideScopeHandler

当超过一个scope(作用域)的时候，AutoDispose会发射一个`OutsideScopeException` error事件，如果想自己定义一种行为，那么就可以使用AutoDisposePlugins#setOutsideScopeHandler，来拦截这些异常并自己进行处理，或者发射或不发射一些东西。

```kotlin
AutoDisposePlugins.setOutsideScopeHandler(t -> {
    // Swallow the exception, or rethrow it, or throw your own!
})
```

### AutoDisposeAndroidPlugins

类似于上面的**AutoDisposePlugins**，允许我们

