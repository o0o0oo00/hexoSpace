---
title: RxJava操作符举例
date: 2018-10-16 15:59:39
categories: 技术
tag : kotlin
---
### 变换与操作符
**所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。**

#### map()操作符
func1有两个泛型，一个是输入泛型，一个是输出泛型，可以看到下面这个例子，输入的是Integer类型，在map的重写方法中进行一系列操作例如`integer.toString()`来将输入类型转换为输出类型。于是观察者收到的类型就变成了String

```
observable.map(new Func1<Integer, String>() {
@Override
public String call(Integer integer) {
return integer.toString();
}
}).subscribe(new Action1<String>() {
@Override
public void call(String s) {
System.out.println(s);
}
});
```
<!--more-->
### 操作符举例
#### first()/last()
只发射第一个数据项，或者是满足条件的第一个数据项。 
附带一个`onCompleted`  

```
Observable.just(1,2,3,4,5)
.first()
.subscribe(new Action1<Integer>() {
@Override
public void call(Integer integer) {
System.out.println(integer.toString());
}
});
}
```
结果 1
#### merge()
合并多个Observeable的发射数据。 可能交错发送

```
Observable<Integer> just = Observable.just(1, 2, 3);
Observable<Integer> just1 = Observable.just(4, 5, 6);
Observable.merge(just1, just).subscribe(new Action1<Integer>() {
@Override
public void call(Integer integer) {
System.out.println(integer.toString());
}
});
```
结果 456123
#### concat()
不同于merge,他是有序发送的
#### zip()
将两个发射项合并一起

```
Observable<Integer> just = Observable.just(1, 2, 3);
Observable<String> just1 = Observable.just("a", "b", "c");
Observable.zip(just1, just, new Func2<String, Integer, String>() {
@Override
public String call(String s, Integer integer) {
return s + integer;//合并操作
}
}).subscribe(new Action1<String>() {
@Override
public void call(String s) {
System.out.println(s);
}
});
```

结果：a1 b2 c3

#### repeat(times)
#### range(start , count)

```
Observable.range(10,2).repeat(2).subscribe(new Action1<Integer>() {
@Override
public void call(Integer integer) {
System.out.println(integer);
}
});
```
结果为： 10 11 10 11

#### flatMap(Func1)
不同于map的一对一。他是一对多的关系。  
A中有一个list，我们现在要发送一个A，但是观察者接收的要是其中的list,

```
class A {
List<String> list = new ArrayList<>();
public A() {
for (int i = 0; i < 10; i++) {
list.add("index " + i);
}
}
}
Observable
.just(new A())
.flatMap(new Func1<A, Observable<String>>() {
@Override
public Observable<String> call(A a) {
return Observable.from(a.list);
}
})
.subscribe(new Action1<String>() {
@Override
public void call(String s) {
System.out.println(s);
}
});

```

结果为：  
index 0 index 1 index 2 index 3 index 4 index 5 index 6 index 7 index 8 index 9

#### timer / interval
timer():  
delay延迟后发送一个值为 long型的0 内部通过onSubscribeTimerOnce工作

```
Observable.timer(2, TimeUnit.SECONDS)
.subscribe(new Action1<Long>() {
@Override
public void call(Long aLong) {
System.out.println("收到一个为" + aLong + "的值");
}
});
```

interval():
间隔interval的时间发送一个从0开始的递增,内部通过OnSubscribeTimerPeriodically工作。

```
Observable.interval(2, TimeUnit.SECONDS).subscribe(new Action1<Long>() {
@Override
public void call(Long aLong) {
System.out.println("收到一个" + aLong + "的值");
}
});
```
#### startWith
startWith： 在数据序列的开头增加一项数据。startWith的内部也是调用了concat
#### ofType
`ofType(final Class<R> klass)`
与filter类似
#### take(N) / takeLast(N)
只发射前/后N个数据
#### skip（N）/ skipLast
跳过前/后N项
#### ignoreElements
丢弃所有数据，只发射错误或正常终止的通知。内部通过OperatorIgnoreElements实现。
#### distinct / distinctUntilChanged
过滤重复数据 / 过滤掉连续重复的数据
#### reduce 
对数据进行聚合操作，只返回最终结果

```
Observable.just(1, 2, 3, 4, 5)
.reduce(new Func2<Integer, Integer, Integer>() {
@Override
public Integer call(Integer integer, Integer integer2) {
return (integer + integer2);
}
}).subscribe(new Action1<Integer>() {
@Override
public void call(Integer integer) {
System.out.println(integer);
}
});
```
#### collect

```
Observable.just(1, 2, 3, 4, 5)
.collect(new Func0<List<Integer>>() {
@Override
public List<Integer> call() {
return new ArrayList<>();
}
}, new Action2<List<Integer>, Integer>() {
@Override
public void call(List<Integer> o, Integer integer) {
o.add(integer);
}
})
.subscribe(new Action1<List<Integer>>() {
@Override
public void call(List<Integer> o) {
System.out.println(o.toString());
}
});
```
结果：[1, 2, 3, 4, 5]

#### toList / toSortedList / toMap

```
Observable.just(1,3,4,5,6).toList().subscribe(new Action1<List<Integer>>() {
@Override
public void call(List<Integer> integers) {
System.out.println(integers.toString());
}
});
```


### 最后来一个例子

```
final String memoryCache = null;
final String diskCache = "从磁盘缓存中获取数据";
Observable<String> memory = Observable.create(new Observable.OnSubscribe<String>() {
@Override
public void call(Subscriber<? super String> subscriber) {
if (memoryCache == null) {
subscriber.onCompleted();
} else {
subscriber.onNext(memoryCache);
}
}
});

Observable<String> disk = Observable.create(new Observable.OnSubscribe<String>() {
@Override
public void call(Subscriber<? super String> subscriber) {
if (diskCache == null){
subscriber.onCompleted();
}else{
subscriber.onNext(diskCache);
}
}
});

Observable<String> net = Observable.just("从NET中获取数据");

Observable.concat(memory,disk,net)
//                .first()
.subscribe(new Observer<String>() {
@Override
public void onCompleted() {
System.out.println("onCompleted");
}

@Override
public void onError(Throwable e) {
System.out.println("onError");
}

@Override
public void onNext(String s) {
System.out.println("最终是 " + s);
}
});

```
**Attention ！！** 

concat 就是如果前一个不发送onComplete那么后一个是不会执行的。
