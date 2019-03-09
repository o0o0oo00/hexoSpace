---
title: RxJava笔记
tags: rxjava
categories: 技术
date: 2019-03-05 16:25:59

---

##  RxJava笔记

如果仅仅是用RxJava的观察者模式Observable来代替原来的Callback形式，显然意义是不大的。

也许除了炫技以外，真的不是我们使用RxJava的理由。

也许目前最流行的说法是

- 链式调用
- 异步+简洁
- 解决回调地狱

实际上RxJava不止于此

### Observable在空间维度上重新组织事件

比如说一个获取一组图片的网络请求，需要加入一个缓存（缓存URL），也就是说当需要展示图片的时候，先获取本地的URL集合，同时发起新的网络请求，等待网络请求完成后，更新缓存，同时使用最新的图片集合刷新列表，缓存可以使用**JakeWharton** 的 [DiskLruCache](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FJakeWharton%2FDiskLruCache)

#### 尝试把读取图片的操作封装为一个Observable

```kotlin
Observable<List<Photo>> cachedObservable = Observable.create(emitter -> {
    DiskLruCache.Snapshot snapshot = cache.get("getAllPhotos");
    if (snapshot != null) {
        List<Photo> cachedPhotos = new Gson().fromJson(
            snapshot.getString(VALUE_INDEX),
            new TypeToken<List<Photo>>(){}.getType()
        );
        emitter.onNext(cachedPhotos);
    } 
    emitter.onComplete();
});
```

然后运用`startWith` 操作符，将两个Observable结合放在一起，这时只需要一个观察者，就可以满足需求了，

```kotlin
networkApi.getAllPhotos()
    .doOnNext(photos -> 
        // 更新缓存
        cache.edit("getAllPhotos")
            .set(VALUE_INDEX, new Gson().toJson(photos))
            .commit()
    )
    // 读取现有缓存
    .startWith(cachedObservable)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(photos -> {
        adapter.setData(photos);
        adapter.notifyDataSetChanged();
    });
```

这样的写法相比较于传统的callback回调来说，减少了Callback的数量，并且观察者的职责单一且清晰了起来，不再关心数据从哪里来，经过怎样的变换。

#### 再进一步 比较两个Observable的发送内容

如果两次发射的内容一致，那么久没有必要去刷新界面了，先不说用Callback来怎样去实现。

我们直接看使用Rxjava如何处理这个需求

`distinctUntilChanged`操作符用来确保Observable发射的数据**相邻的两个元素**必须是不相等的。

只需要加一个操作符而已，就可以完美的解决需求。

而我们的观察者至始至终都是比较稳定的，不关心数据从何而来，也并没有发什么变化。



- Observable通过内置的操作符对自身发射的元素在**空间维度上重新组织**，或者与其他的Observable一起在空间维度上进行重新组织，使得观察者的逻辑简单而直接，不需要关心数据从何而来，从而使观察者的逻辑较为**稳定**。
- 传统的Callback形式是直接生产者与消费者通过及回调接口进行联系。而引入RxJava之后在这个生产者-消费者模型中间又多了一层RxJavaOperator处理。
  - 比如说网络请求相关的逻辑，测试就不是很方便，但是如果们我们使用RxJava进行解耦以后，观察者只是耦合Observable这个接口而已，我们可以手动创建用于测试的Observable，进而测试我们的观察者模块。
- 取消订阅：取消订阅下放到了观察者这一层了，观察者在订阅Observable时，会得到一个Disposable，观察者不需要知道时间的产生、源头在哪里，只要通过这个接口通知事件生产者就可以了。

### 时间维度

比如所搜索功能，每次输入新的内容都会出发搜索接口，而用户如果以过快的速度去输入的话，势必会造成请求次数过多，而请求返回的顺序出现差错，先请求的接口在后请求的接口之后返回数据，显然我们是没有必要去接收之前的数据的，应该取消它。

1. 防止用户输入过快，触发过多网络请求，需要对输入事件做一下防抖动。

2. 用户在输入关键词过程中可能触发多次请求，那么，如果后一次请求的结果先返回，前一次请求的结果后返回，这种情况应该保证界面展示的是后一次请求的结果。
3. 用户在输入关键词过程中可能触发多次请求，那么，如果后一次请求的结果返回时，前一次请求的结果尚未返回的情况下，就应该取消前一次请求。



> `switchMap` 这个操作符与 `flatMap` 操作符类似，但是区别是如果原 `Observable` 中的两个元素，通过 `switchMap` 操作符都转为 `Observable` 之后，如果后一个元素对应的 `Observable` 发射元素时，前一个元素对应的 `Observable` 尚未发射完所有元素，那么前一个元素对应的 `Observable` 会被自动取消订阅，尚未发射完的元素也不会体现在 `switchMap` 操作符调用后产生的新的 `Observable` 发射的元素中

> `debounce` 操作符产生一个新的 `Observable`, 这个 `Observable` 只发射原 `Observable`中时间间隔小于指定阈值的最大子序列的最后一个元素

`debounce` 操作符解决了问题 **1**，`switchMap` 操作符解决了问题 **2**、**3**。这个例子可以很好的说明，**RxJava 的 Observable 可以通过一系列操作符从时间的维度上重新组织事件，从而简化观察者的逻辑**。

[原文链接，作者：prototypez](