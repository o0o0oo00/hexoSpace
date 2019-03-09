---
title: 两种方式写一个EventBus或者叫EventDidi
tags: rxjava
categories: 技术
date: 2019-03-09 15:24:52
top: true
cover: true
---

## 用RxJava写一个EventBus

骚话不说。现在我们用RxJava来**优雅的**实现它。RxJava我们都知道，[观察者模式](https://o0o0oo00.github.io/2018/10/09/Observer/)！

EventBus，直接翻译为事件的公交车，我们现在写一个事件的专车。

首先写一个用于观察的ObservableMap作为订单池，每当Map发生改变的时候，都会去通知观察者。

ObservableMap的Key和value 就相当于是订单和人，订单池里面有很多的订单（key），而这些订单与人绑定，

我们Map的key作为订单，来确定谁是乘车人，然后司机只需要接单，就可以去接乘客出发去目的地啦。

```kotlin
class ObservableMap<T, K> : HashMap<T, K>() {

    private var onChange: ((T) -> Unit)? = null

    override fun put(key: T, value: K): K? {
        val put = super.put(key, value)
        onChange?.invoke(key)
        return put
    }

    fun registerOnChange(change: (T) -> Unit) {
        this.onChange = change
    }

    fun unregisterOnChange() {
        this.onChange = null
    }
}
```



1、继承自HashMap<T,K> 然后重写它的put方法，每当有put操作发生的时候，也就是下单的时候，我们就去通知我们的滴滴平台，

用一个高阶函数，来将这个key传给Observable，让Observable来发射给观察者。

2、我们写一个滴滴平台用来分发专车订单：

每次发生put操作也就会生成一个订单(如果之前没有订单的话)，

定义一个Observable好让观察者（司机）来观察我（订单），也就是一个滴滴平台管理的订单池，司机可以通过订单id来找到我。

```kotlin
private val onBooleanChange = Observable.create<String> { emitter ->
        val onChange: (String) -> Unit = {
            emitter.onNext(it)
        }
        emitter.setCancellable {
            map.unregisterOnChange()
        }
        map.registerOnChange(onChange)
    }.share()
```



我们都知道当Observable在发生订阅关系的时候才会发射数据，所以我们写一个发生订阅的方法。我们将他作为司机。

```kotlin
fun observeBoolean(key: String): Observable<Boolean> = onBooleanChange
        .filter { it == key }
        .map { map[key] }
```

根据传进来的key作为唯一标识，也就是司机要知道是你打的专车，然后将对应的value发送给Observer观察者。司机拿着这个订单Key来匹配滴滴平台中的诸多订单**filter { it == key }**，当发现有匹配订单的时候，司机可以根据这订单来拿到乘客**.map { map[key] }**，一旦找到匹配的订单，司机就可以去接乘客啦。



我们看下面，这里为map进行了一次put操作，前面说到map进行操作put，然后高阶函数将key发射给Observable，这里我们可以看做是乘客去下单啦，下的订单下到订单池中去，由Observable来发射给各个观察者(司机)

 **map[key] = value**    就是   **map.put(key,value)**

```kotlin
fun setBoolean(key: String,  value: Boolean) {
        map[key] = value
    }
```



既然已经将Observable写好了，那么理所当然我们马上就要去**subscribe**发生订阅关系了啦！

司机拿到了真正的订单**The KEY**，油门已经踩得嗡嗡响了，一旦乘客下单 订单号为The KEY，那么司机马上就可以接到乘客出发啦，最终目的地是sout。岂不美哉

```
 ObserverBooleanStatus.observeBoolean(The KEY)
            .subscribe { boolean -> 
               sout(boolean = $boolean)
            }
            
```

现在我要下单啦，一个订单**The KEY**和一个我**value**，先别管我为什么是一个true，我也只是一个数据信息而已。

```kotlin
ObserverBooleanStatus.setBoolean(The KEY, true)
```

已经发送滴滴平台啦。司机大哥一看到我下单了，立刻就来接我啦。

## 用LiveData写一个EventBus

上面的那些比喻理解起来可能有些迷糊，再来梳理一下，然后用LiveData再写一个。

其实只要理解了，事件通知这个东西的本质是观察者模式这件事，就能触类旁通。

### we need

- 一个订单池：订单池负责存储订单(**String**)和乘客信息(**MutableLiveData<Any>**)
- 一个滴滴平台：当对应订单的乘客来的时候，平台会通知司机
- 一个司机端：司机端也就是我们所说观察者，一直观察订单，当这个订单有人来的时候，就出发。
- 一个乘客端：就是把我的订单(key)，和我(value)发送给订单池，然后由滴滴平台来通知司机

```kotlin
object LiveDataStatus {
	// 订单池
    private val map = HashMap<String, MutableLiveData<Any>>()
	// 滴滴平台：分发订单
    fun <T : Any> observeData(key: String): MutableLiveData<T> {
        if (!map.containsKey(key)) {
            map[key] = MutableLiveData()
        }
        return map[key]!! as MutableLiveData<T>
    }
	// 司机，焦灼的等待着订单
    fun observeData(key: String): Any = observeData(key)
	// 乘客端，一个订单对应一个人
    fun setObserveData(key: String, value: Any) {
        map[key]?.postValue(value)
    }

}
```

前提是乘客要先下单，司机端才能去接乘客。

现在乘客要下单啦，他的订单号是key，它的信息是一句诗

```kotlin
 LiveDataStatus.setObserveData("key", "会须一饮三百杯")
```



此时急不可待的司机师傅已经坐不住啦，油门踩的嗡嗡响，就等着乘客下单呢。正好上面的乘客下单了，司机立马出发，到达了 **println("it = $it")**这个地方。皆大欢喜！

```kotlin
LiveDataStatus.observeData<Boolean>("key")
    .observe(activity!!,android.arch.lifecycle.Observer {
        println("it = $it")
    })
```









