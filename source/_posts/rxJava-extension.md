---
title: RxJava 的一些奇技淫巧
tags: rxjava
categories: 技术
date: 2019-03-11 14:16:00
---

# RxJava 的一些奇技淫巧

## 防止双击

网上很多例子都是写RxBinding的防止抖动。但是我现在不想用它，那么如何去封装一个点击事件呢？

分析： 我们每次点击View都发射一个onNext，然后运用操作符**throttleFirst**它的意思是在指定时间内只发射第一条事件，正好符合我们的需要。

```kotlin
fun View.notDoubleClick() : Observable<View> {
    var emitter: ObservableEmitter<View>? = null
    val observable = Observable.create<View> {
        emitter = it
    }
    setOnClickListener {
        emitter?.onNext(this)
    }
    return observable.throttleFirst(1000,TimeUnit.MILLISECONDS)
}
```

当我们需要使得这个控件是防止双击的时候我们这样使用

```kotlin
titleText.notDoubleClick().subscribe {
// do something
}.autoDispose(activity!!)
```

## 请求Loading

我们想要在请求开始的时候加一个Loading，结束的时候将Loading取消掉

包括onComplete、onError、dispose

经查阅得知在这三个之后都回调用doFinally()，索性我们就写一个扩展方法

```kotlin
fun <T> Observable<T>.loading(activity: AppBaseActivity, cancel: Boolean = false): Observable<T> {
    val wake = WeakReference(activity)
    return doOnSubscribe {
        wake.get()?.loading(true, cancel)
    }.doFinally {
        wake.get()?.loading(false, cancel)
    }
}
```

我们的Response数据也会随着扩展方法一并返回去了，而且扩展方法，不同于使用操作符会产生额外的Observable对象。

使用的话直接加到我们调用链里面，非常完美！