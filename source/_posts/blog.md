---
title: Meet Kotlin
categories: "技术"
---
1. 函数参数可以添加默认值，这样可以不穿这个参数，相当于是重载的一种方式
2. 如果没有任何指定，类的属性会默认的使用getter和setter方法，当然也可以自定义setter方法与getter方法， 
<!--more-->
```
var name: String = ""
        get() = field.toUpperCase()
        set(value){
            field = "Name: $value"
        }
```
那么既然直接使用点来代替setter方法，那么需要访问属性的时候需要使用，filed来作为属性本来的值。

覆盖方法总是使用与基类型相同的默认值

调用混位置参数时候，命名参数要放在默认位置参数之后，例如：允许：（1，x = 1）不允许 （x = 1 , 1）  

单个表达式函数，可以省略花括号，直接在等号后面接表达式，作为函数体，当返回值类型编译器可以推断出来的时候，可以不同指定函数的返回值类型  

具有代码块的函数需要显示的指定返回值类型， 

可变数量的参数Varargs

```
fun <T> asList(vararg ts:T): List<T> {
    val result = ArrayList<T>()
    for(t in ts)//ts is an array
        result.add(t)
    return result
}
```


#### 扩展函数
```
fun Context.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}
```
扩展了Context，直接使用this，来代表context的实例

** 映射对象到变量中**
```
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val (date, temperature, details) = f1

for ((key, value) in map) {
    Log.d("map", "key:$key, value:$value")
}
```
### 扩展函数
扩展函数是指在一个类上面增加一种新的行为，它就表现的像是属于这个类一样，而且可以使用this关键字调用类的所有public方法.它可以被任何类和类的扩展子类调用。  

扩展函数是静态解析的，即使是在运行时参数传入其他类型，得出的结果仍然是声明时候的参数类型  
**扩展函数中的操作符**：  
```
operator fun ViewGroup.get(position: Int): View = getChildAt(position)  

val container: ViewGroup = find(R.id.container)
val view = container[2]
```

### Lambdas
定义一个匿名函数，
在箭头的左边指定参数，在箭头的右边返回函数执行的结果。  
一直简写，可以写成这样
```
view.setOnClickListener { toast("Click") }
```

```
fun main(args: Array<String>) {
   val result = {msg : String -> print(msg)}
   result("hello")
}
```
**语法糖**   

* 当参数只有一个的时候，声明中可以不用显示的声明参数，在使用参数的时候可以使用it来代替这个唯一的参数
* 当有多个参数用户到的时候，可以使用下划线来代替参数名，但是如果已经使用下划线来省略参数时候，是不能用it来代替当前参数的
* Lambda 最后一条语句的执行结果，表示这个表达式的返回值


### 高阶函数
当函数的参数为闭包时，称这个函数为高阶函数  

```
val printMsg = {str : String -> print(str)}
val log = { str: String , log:(String)-> Unit -> log(str) }

fun main(args: Array<String>) {
 log("hello",printMsg)
}
```
printMsg 是一个 lambda表达式 参数为 String  
log是一个lambda表达式 参数为 String和一个函数  

***TODO***

### 单例模式
object 可以定义在全局也可以在类的内部使用  
object 就是单例模式的化身  
object 可以实现 Java 中的匿名类  
companion object 就是 Java 中的 static 变量  
companion object 只能定义在对应的类中 

### 委托属性
Kotlin 的属性是天生再带set、get方法的，如果要重写他们的时候需要用到field这个东西，field就是该属性本身的意思。  
***TODO***

### 集合和函数操作符
#### 总数操作符：  
any，至少有一个符合要求返回rue  
all，全部都要符合要求返回true  
count，返回条件判断的总数  
fold，从第一项，到底i项  
foldRight，从最后一项，到第n-i项  
forEach，遍历所有元素  
forEachIndexed，  
#### 过滤操作符：
* drop，去掉前n个元素 
* dropWhile从第一项开始去掉指定元素的列表 
* dropLastWhile 从最后一项开始
* filter 过滤掉所有符合函数条件的元素
* filterNot 过滤掉所有不符合条件的元素
* slice 过滤一个list中指定index的元素
* take 返回从第一开元素开始的n个元素
* takeLast 返回从最后一个元素开始的n个元素
* takeWhile 返回从第一个元素开始符合条件的元素
#### 映射操作符：
* flatMap 为每一个元素通过函数省城新的元素集合
* groupBy 返回一个根据给定函数分组后的map
* map 返回每一个元素根据给定的函数所转换成的list
* mapIndexed 返回一个每一个
* mapNotNull 返回每一个飞null元素根据指定的函数所转换成的list
#### 元素操作符
* contains 如果指定元素可以在集合中找到返回true
* elementAt 返回给定index对应的元素，注意不能越界
* elementAtOrElse 如果越界返回默认值
* elementAtOrNull 
* first 返回符合给定函数的第一个元素
* firstOrNull 没有返回null
* indexOf 返回指定元素的第一个index 不存在-1
* indexOfFirst  返回符合条件的第一个元素的index 没有-1
* indexOfLast 返回最后一个符合给定函数的元素的index 没有-1
* last 返回符合给定函数的最后一个元素
* single 返回符合非定函数的单个元素，


#### 生产操作符
* merge 把两个集合合并成一个新的，相同的index的元素通过给定的函数进行合并成新的元素，作为新的元素几个的一个元素，返回这个新的集合，新的集合的大小有最小的那个集合大小决定。
* partition 通过非定的函数条件，把一个给定的集合分割成两个
* plus + 两个集合相加

#### 顺序操作符
* reverse 返回一个翻转原list的list
* sort 排序
* sortBy 根据指定函数排序
* sortDescending 降序排列
* sortDescendingBy 。。

### 空安全
```
val a : Int? = null
```
* 通过在类型的后面增加一个问号，就表示允许这个变量为null  
* 一个可null的变量，在没有检查之前就去使用它，是不能通过编译的。
* 还有一个特性就是，当我们检查了变量是否为null之后，就可以自动推导出变量不可为null，进而可以使用，编译通过。  

  
这里使用**安全访问操作符（？）**，来达到判断并推导出可null类型。  
如果我们确定正在使用一个非null得变量，但是他的类型却是可null类型的，我们可以强制使用！！来使编译器执行可null类型，来跳过限制检查。

### 委托模式
他可以用来从类中抽取出主要负责的部分。
委托者只需要指定实现的接口的实例，我们可以使用接口指示鸟可飞行，但是鸟的飞行方式呗定义在一个委托中，这个委托定义在构造函数中，所以我们可以针对不同的鸟使用不同的飞行方式，动物使用翅膀飞行的方式被定义在另一个类中，  
***TODO***


## 泛型

可以使用任何类型初始化
```
class TypedClass<T>(parameter: T) {  
    val value: T = parameter
}
```
如果在初始化的时候可以推断出参数的类型，甚至可以不需要去指定泛型
```
val t1 = TypedClass<String>("Hello World!")
val t2 = TypedClass<Int>(25)  
val t2 = TypedClass<Int>(25)
```
如果是一个null，则不能推断出他的类型，所以必须要加上？  
函数中也可以使用泛型  

### 变体



s
