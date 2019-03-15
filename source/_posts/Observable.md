---
layout: rxjava
title: Observable
date: 2019-03-06 17:43:42
tags:
---

### ObservableCreate

* 它是Observable的子类
* 将create中传入的参数传给ObservableCreate
* subscribeActual
* 传入的参数与：source 实现了ObservableOnSubscribe接口，并在接口的subscribe方法中，传入了数据源，这些数据源需要通过ObservableEmitter进行发射

### Observable.subscribe()

* Observable.subscribe(observer)实际上就是调用Observable.subscribeActual(observer)方法
* ObservableEmitter： 数据源发射器，内部存储了observer
* 