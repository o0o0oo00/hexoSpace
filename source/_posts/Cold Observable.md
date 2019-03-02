RxJava tips

## Cold Observable
我们常见的工厂方法提供的都是ColdObservable,包括`just(),fromXX,create(),interval(),defer()`。 他们的共同点是当你有多个Subscriber的时候，他们的事件是独立的，就是说发给每个订阅者的时间都是新的，不共用且互不干扰。

## Hot Observable

Hot Observable是共享数据的，订阅者在同一时刻收到相同的数据。

## Observable.create

* 返回的是**`ObservableCreate`** 对象
* ObservableCreate本身就是Observable的子类，因此Observable.create()实际上就是创建并返回一个ObservableCreate对象
* 在ObservableCreate的构造函数中，实际上是将create()方法中的参数ObservableOnSubscribe依赖注入ObservableCreate中并保存 **`this.source = source;`**
* subscribeActual()方法是在Observable.subscribe()**订阅后**才执行的

### 实现订阅


