---
title: RxJava - 异步
date: 2018-10-15 17:27:52
categories: 技术
tag : 技术
---
# RxJava - 异步

### Observable(被观察者)的创建
它没有公开的构造方法，是通过内部的Create方法来创建一个

```
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
@Override
public void call(Subscriber<? super String> subscriber) {}
});
```
我们可以看到，call方法的参数是是一个观察者，也就是持有**观察者**的引用，**被观察者**做了一些操作以后，通过`subscriber.onNext/onComplete` 来实现**响应式**
<!--more-->
### Subscriber/Observer(观察者)的创建

Subscriber 对 Observer 接口进行了一些扩展，但他们的基本使用方式是完全一样的

```
Subscriber<String> subscriber = new Subscriber<String>() {
@Override
public void onCompleted() {

}

@Override
public void onError(Throwable e) {

}

@Override
public void onNext(String s) {
System.out.println("subscriber" + s);
}
};

Observer<String> observer = new Observer<String>() {
@Override
public void onCompleted() {

}

@Override
public void onError(Throwable e) {

}

@Override
public void onNext(String s) {
System.out.println("observer"+s);
}
};
```
### 观察者响应被观察者
我们知道[观察者模式](https://o0o0oo00.github.io/2018/10/09/Observer/#more)，是**被观察者持有观察者**的引用，进而遍历所有观察者，调用（通知）其方法来达到响应的目的。
所以我们看RxJava中的关联方式是通过**被观察者订阅观察者**observable.subscribe(observer/subscriber)来实现的。这个地方可能与概念有些不贴切，因为概念是观察者要响应的是被观察者，所以这里通过**被观察者订阅观察者**来达到**观察者响应被观察者**的目的

1. 创建被观察者`Observable`与观察者`Observer`对象
2. 观察者订阅被观察者，实际是**观察者**当做参数传入**被观察者** `observable.subscribe(observer);`
3. **被观察者**`Observable`的重写方法`call()`中调用**观察者**`Observer`的`onNext`方法。然后`Observer`**重写**的`onNext`进行相应的处理

### Observable的其他创建方式
#### Observable.just(t : T) 具有多个参数的重载方法，源码中就有多达10个参数的重载方法。

例如：相当于执行`call`方法中的`subscriber.``onNext("123")``onNext("234")``onCompleted()`

```
Observable.just("123","234").subscribe(new Observer<String>() {
@Override
public void onCompleted() {

}

@Override
public void onError(Throwable e) {

}

@Override
public void onNext(String s) {
System.out.println(s);
}
});

```
#### Observable.from(Iterable<? extends T> iterable)

`from(Iterable<? extends T> iterable)`**支持从数组或者是实现了Iterator接口的集合中接收参数**  
例如：这两种方式

```
Observable.from(new String[]{"345","456"}).subscribe(new Observer<String>() {
@Override
public void onCompleted() {

}

@Override
public void onError(Throwable e) {

}

@Override
public void onNext(String s) {
System.out.println(s);
}
});
```
```
List<String> list = new ArrayList<>();
list.add("567");
list.add("678");
Observable.from(list).subscribe(new Observer<String>() {
@Override
public void onCompleted() {
System.out.println("onCompleted");
}

@Override
public void onError(Throwable e) {

}

@Override
public void onNext(String s) {
System.out.println(s);
}
});
```
**PostScript**：当just参数在2个以上时，实际上内部调用的也是from(T[] array)方法。

#### `Action0`、`Action1<T>` ... `Action9<T1, T2, T3, T4, T5, T6, T7, T8, T9> `
0个参数的Action,1个参数的Action  
Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。由于 onNext(T obj) 和 onError(Throwable error) 也是单参数无返回值的，因此 Action1 可以将 onNext(obj) 和onError(error) 打包起来传入 subscribe() 以实现不完整定义的回调。

```
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
@Override
public void call(Subscriber<? super String> subscriber) {
subscriber.onNext("123");
subscriber.onNext("234");
subscriber.onCompleted();
}
});

observable.subscribe(new Action1<String>() {
@Override
public void call(String s) {
System.out.println("onNext"+s);
}
}, new Action1<Throwable>() {
@Override
public void call(Throwable throwable) {
System.out.println("onError");
}
}, new Action0() {
@Override
public void call() {
System.out.println("onComplete");
}
});
```

来看看源码中定义的使用方式

```
public final Subscription subscribe(final Action1<? super T> onNext) {
if (onNext == null) {
throw new IllegalArgumentException("onNext can not be null");
}

Action1<Throwable> onError = InternalObservableUtils.ERROR_NOT_IMPLEMENTED;
Action0 onCompleted = Actions.empty();
return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
}

public final Subscription subscribe(final Action1<? super T> onNext, final Action1<Throwable> onError) {
if (onNext == null) {
throw new IllegalArgumentException("onNext can not be null");
}
if (onError == null) {
throw new IllegalArgumentException("onError can not be null");
}

Action0 onCompleted = Actions.empty();
return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
}

public final Subscription subscribe(final Action1<? super T> onNext, final Action1<Throwable> onError, final Action0 onCompleted) {
if (onNext == null) {
throw new IllegalArgumentException("onNext can not be null");
}
if (onError == null) {
throw new IllegalArgumentException("onError can not be null");

if (onCompleted == null) {
throw new IllegalArgumentException("onComplete can not be null");
}

return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
}
```

无论传入几个Action，最后都会返回`subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));`  
哎呀，听人家讲不如自己去看看源码，也没几行 (●ﾟωﾟ●)


上面这些都只是皮毛啦，RxJava真正的意义在于**异步**，『后台处理，前台回调』，而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler 。

### 线程控制Scheduler
用于指定每一段代码运行在什么样的线程之中。RxJava内置的Scheduler：

* `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
* `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
* `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
* 另外， Android 还有一个专用的 `AndroidSchedulers.mainThread()`，它指定的操作将在 Android 主线程运行。

有了这些个SchedulerJ就可以用`subscribeOn ``observeOn `这两个方法来对线程进行控制了。分别指定**观察者在主线程**用于更新UI，**被观察者位于io线程**执行以下耗时的操作。  
例子：

```
Observable.just(123, 123, 123)
.subscribeOn(AndroidSchedulers.mainThread())
.observeOn(Schedulers.io())
.subscribe { integer -> println(integer!!.toString()) }

```


