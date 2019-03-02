---
title: RxJava2 线程调度
date: 2018-10-22 16:08:04
categories: 技术
tag: kotlin
---
### RxJava2 线程调度
既然是异步就要涉及到线程控制  
#### RxJava提供的线程

1. `Schedulers.trampoline()`当前线程
2. `Schedulers.newThread()`启动新的线程，在新的线程中执行操作
3. `Schedulers.io()`io读写线程，区别于`newThread`在于`io`内部的是用一个无数量上限的线程池，可以重用空闲线程，推荐使用。
4. `Schedulers.computation()`这个Sheduler使用的固定的线程池，大小为cpu核数，有些操作符或者涉及到运算的情况下使用。
5. `Schedulers.single()`任务都在这个单例线程中执行，按**队列**顺序
<!--more-->

#### 指定线程
指定**观察者**线程：`observerOn(Schedulers)`  
指定**被观察者**线程：`subScribeOn(Schedulers)`  

恕老衲直言，这个名字起的是真的瘪嘴。

看例子：

```
private fun testThreadWhere() {
    Observable.create(ObservableOnSubscribe<Long>() {
        Log.e(TAG, "发射 线程为：${Thread.currentThread().name} ")
        it.onNext(12L)
    })
        .filter {
            Log.e(TAG, "filter 线程为：${Thread.currentThread().name} $it ")
            return@filter it > 10
        }
        .subscribeOn(Schedulers.io())// 1 指定filter线程 
        .observeOn(Schedulers.newThread()) // 2 指定map线程
        .map(Function<Long, String>() {
            Log.e(TAG, "map 线程为：${Thread.currentThread().name} $it ")
            return@Function it.toString()
        })
        .observeOn(AndroidSchedulers.mainThread())// 3
        .subscribe {
            Log.e(TAG, "处理 线程为：${Thread.currentThread().name} $it ")
        }
}
```
看一下结果：
其中  
`RxCachedThreadScheduler `为`io`线程因为其内部采用的是线程池处理机制。  
`RxNewThreadScheduler `为`newThread`线程。

```
10-22 16:02:16.199 17797-17855/com.zcy.nidavellir.javaworld E/MainActivity: 发射 线程为：RxCachedThreadScheduler-1 
10-22 16:02:16.199 17797-17855/com.zcy.nidavellir.javaworld E/MainActivity: filter 线程为：RxCachedThreadScheduler-1 12 
10-22 16:02:16.200 17797-17856/com.zcy.nidavellir.javaworld E/MainActivity: map 线程为：RxNewThreadScheduler-1 12 
10-22 16:02:16.251 17797-17797/com.zcy.nidavellir.javaworld E/MainActivity: 处理 线程为：main 12 
```

* 如果注释掉 注释中的1 相当于是不指定**被观察者**线程，那么**被观察者**会在当前的**默认线程**也就是**主线程**执行任务。
* 如果注释掉 注释中的2 那么filter 和 map 都会在`io`线程中执行，直到遇到了`observerOn`
* observerOn的下游也就是**观察者**因为指定了`AndroidSchedulers.mainThread()`所以在`mainThread`中运行。
* 多次指定`subscribeOn`不生效，以最先指定的线程为准。多次`observerOn`生效，指定一次对下游直到下一个`observerOn`中间生效。




