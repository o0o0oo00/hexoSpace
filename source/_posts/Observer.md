---
title: 观察者模式简述
date: 2018-10-09 17:33:19
categories: 技术
---
### 观察者模式简述
点击查看理论知识…（⊙＿⊙；）…
<details>
观察者模式是对象的行为模式，又叫发布-订阅(Publish/Subscribe)模式、模型-视图(Model/View)模式、源-监听器(Source/Listener)模式或从属者(Dependents)模式。
观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态上发生变化时，会通知所有观察者对象，使它们能够自动更新自己。
**观察者模式所涉及的角色有**  
**抽象主题角色：**抽象主题角色吧所有对观察者对象的引用保存在一个聚集里面。每个主题都可以有任何数量的观察者。抽象主题提供一个借口，可以增加和删除观察者对象，抽象主题角色又叫做抽象被观察者（Observable）角色。  
**具体主题角色**：将有关状态存入具体观察者对象；在具体主题的内部状态改变时，给所有登记过的观察者发出通知。具体主题角色又叫做具体被观察者(Concrete Observable)角色。  
**抽象观察者：**为所有的具体观察者定义一个接口，在得到主题的通知时更新自己，这个接口叫做更新接口。  
**具体观察者：**存储与主题的状态自恰的状态。具体观察者角色实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题的状态 像协调。如果需要，具体观察者角色可以保持一个指向具体主题对象的引用。  
</details>
<!--more-->
#### 简易的例子
上面的全都是理论，读起来绕嘴都 。。。

1. 创建观察者
2. 创建被观察者
3. 被观察者的状态改变，进而通知观察者改变状态。所以，被观察者里面有一个装有所有观察者的集合，被观察者状态改变的时候，遍历被观察者所拥有的这个集合，通知（调用）观察者中的更新状态方法。

```
/**
* @author:       zhaochunyu
* @description:  被观察者的抽象
* @date:         2018/10/9
*/
abstract class Subject {
private val list = mutableListOf<MyObserver>()


fun attach(observer: MyObserver) {
list.add(observer)
println("Attached an observer")
}

fun detach(observer: MyObserver) {
list.remove(observer)
println("detach an observer")
}

fun notifyObservers(newState: String) {
list.forEach {
it.update(newState)
}
}

}
```
```
/**
* @author:       zhaochunyu
* @description:  实际被观察者
* @date:         2018/10/9
*/
class ActualSubject : Subject() {

fun change(newString: String) {
println("被观察者的状态改变为：" + newString)
notifyObservers(newString)
}
}
```

```
/**
* @author:       zhaochunyu
* @description:  观察者的抽象
* @date:         2018/10/9
*/
interface MyObserver {

fun update(newState: String)

}
```
```
/**
* @author:       zhaochunyu
* @description:  观察者
* @date:         2018/10/9
*/
class ActualObserver : MyObserver {

override fun update(newState: String) {
println("观察者状态改变为："+ newState)
}
}
```
我们得到在创建了实际观察者与实际被观察者对象之后，进行Attach(也就是将观察者注册到被观察者的集合中)，然后改变被观察者状态。验证观察者更新状态。

```
class MainActivity : AppCompatActivity() {

override fun onCreate(savedInstanceState: Bundle?) {
super.onCreate(savedInstanceState)
setContentView(R.layout.activity_main)

val actualSubject = ActualSubject()
val actualObserver = ActualObserver()
// 注册
actualSubject.attach(actualObserver)

actualSubject.change("new state")

}
}
```
#### 推模型VS拉模型
<details>
在观察者模式中，又分为推模型和拉模型两种方式。  
**推模型**  
主题对象向观察者推送主题的详细信息，不管观察者是否需要，推送的信息通常是主题对象的全部或部分数据。  
**拉模型**  
主题对象在通知观察者的时候，只传递少量信息。如果观察者需要更具体的信息，由观察者主动到主题对象中获取，相当于是观察者从主题对象中拉数据。一般这种模型的实现中，会把主题对象自身通过update()方法传递给观察者，这样在观察者需要获取数据的时候，就可以通过这个引用来获取了。  
**两种模式的比较**  
　　■　　推模型是假定主题对象知道观察者需要的数据；而拉模型是主题对象不知道观察者具体需要什么数据，没有办法的情况下，干脆把自身传递给观察者，让观察者自己去按需要取值。
　　■　　推模型可能会使得观察者对象难以复用，因为观察者的update()方法是按需要定义的参数，可能无法兼顾没有考虑到的使用情况。这就意味着出现新情况的时候，就可能提供新的update()方法，或者是干脆重新实现观察者；而拉模型就不会造成这样的情况，因为拉模型下，update()方法的参数是主题对象本身，这基本上是主题对象能传递的最大数据集合了，基本上可以适应各种情况的需要。
</details>
实际就是传一个具体的类型，例如String,还是传入一个被观察者对象的区别。方便使用吧o((⊙﹏⊙))o.
