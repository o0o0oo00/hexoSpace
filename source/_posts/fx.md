---
title: 泛型简述
date: 2018-10-11 16:48:12
categories: 技术
tag: kotlin
---
### 泛型
泛型还有一种较为准确的说法就是为了参数化类型，或者说可以将类型当作参数传递给一个类或者是方法。  
看一个例子，证明为什么要有泛型
<!--more-->

``` 
class Cache {
var value: Any? = null
}
```

我们操作数据的时候是酱紫的

```        
Cache cache = new Cache();
cache.setValue("123");
String str = (String) cache.getValue();

```
显然需要一个强制类型转换才能取出理想的数据。
我们来看使用泛型的情况

```
class Cache<T> {
var value: T? = null
}

Cache<String> cache = new Cache<String>();
cache.setValue("123");
String str =cache.getValue();
```
你看，不用强制类型转换了吧~而且泛型可以传入任何一种想要的类型
泛型可以将类型参数化，但是参数一旦确定好，如果类似不匹配，编译器就不通过。  
**好处：**  
1. 它提供了一种扩展能力  
2. 它是一种类型安全检测机制  
3. 泛型提高了程序代码的可读性  

#### 按照使用方式
1. 泛型类。 
2. 泛型方法。 
3. 泛型接口。

**定义一个泛型类,理论上这个ZCY可以是任何**

```
class Cache<ZCY> {
var value: ZCY? = null
}
```

```
class Cache<ZCY1,ZCY2> {
var value: ZCY1? = null
var value2: ZCY2? = null
}
```

但出于规范的目的，Java 还是建议我们用单个大写字母来代表类型参数。常见的如：

1. T 代表一般的任何类。 
2. E 代表 Element 的意思，或者 Exception 异常的意思。 
3. K 代表 Key 的意思。 
4. V 代表 Value 的意思，通常与 K 一起配合使用。 
5. S 代表 Subtype 的意思

**泛型方法**  
泛型方法所定义的泛型与其所在类的泛型互相不冲突，类的ZCY可以是String类型，而方法的ZCY可以是Int类型，它只是一泛泛的类型而已

```
fun someThing(some: ZCY2): ZCY1? {
return null
}

或者
@Throws(IllegalAccessException::class, InstantiationException::class)
fun <T> fun1(t: Class<T>): T {
return t.newInstance()
}

```
#### 通配符
除了用` <T> `表示泛型外，还有 `<?>` 这种形式。`？` 被称为通配符。  
有需求说希望泛型可以处理某一范围内的数据类型，比如某个类和他的子类  
##### 三种形式的通配符

如果存在了通配符，那么它就丧失了“写”的能力，因为它属于未知类型。
下面的编译是不会通过的

```
List<?> test1 = new ArrayList<>();
test1.add("123"));
test1.add(123);

```
于是乎，通配符用来指明泛型**范围**、**边界**

1. <?> 被称作无限定的通配符。
2. <? extends T> 被称作有上限的通配符。
3. <? super T> 被称作有下限的通配符。 

时常（哦不，应该是一直）搞不懂他俩的区别  
For Example:

```
Plate<Fruit> plate = new Plate<Apple>(new Apple());
```
这样子是不能通过编译的，**因为一个装苹果的盘子不能变成一个装水果的盘子**。沙雕玩意 ┐(ﾟ～ﾟ)┌ 
就算容器里装的东西之间有继承关系，但容器之间是没有继承关系的。所以我们不能像`Fruit fruit = new Apple();`这么样写。

**有上限通配符**：   
`Plate<？ extends Fruit>`
就是说水果以及水果的子类可以放进盘子里面。

```
Plate<? extends Fruit> plate = new Plate<Apple>(new Apple());
```
此时我们发现set方法失效了，原因是编辑器只知道可以set进去Fruit和它的子类，但是具体的类型不知道，可能是Apple，可能是Banana、Orange。所以就不允许。（可是为什么构造方法可以啊？摊手┑(￣Д ￣)┍）  
**那么我们就可以得出结论 `？`代表我知道它是一个东西，具体是什么我不知道**  

**有下限通配符**：  
`Plate<? super Fruit>`就是水果以及水果的父类（食物）可以放进盘子里面

```
Plate<? super Fruit> plate = new Plate<Fruit>(new Fruit());
plate.set(new Apple());
plate.set(new Fruit());
plate.set(new Food());//报错
Object o = plate.get();
```
我们可以看到，水果（Apple 或者 Fruit）可以放进盘子里面，但是Food这一行会报错的，**因为食物Food不一定是水果**，也可能是肉、蔬菜，所以它不能放盘子里面。  
下限通配符，放开了存储，但是取出来的话就会失效一部分，因为取出来的不知道到底是什么类型，Apple? Fruit? Banana? 不确定 **所以只能用Object来接收**，这样就把元素的类型信息丢掉了。 (●ﾟωﾟ●)

##### PECS:  

* 频繁往外读取内容的，适合用上界Extends。
* 经常往里插入的，适合用下界Super。

#### 类型擦除

java中的泛型，是相对于**编辑器**阶段的，在**编译**期间java的字节码中是不包含泛型信息的。

看一段代码，这段代码的结果是true，是因为在进入JVM之前，与泛型相关的信息，T 会被类型擦除，无论 T 是String，还是Integer都被替换成了Object，如果指定了上限<? extends String>那么就会被擦除成上限String。

```
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();

System.out.println(l1.getClass() == l2.getClass());
```

**Attention**  
泛型类或者泛型方法中，不接受 8 种基本数据类型。要接受也是他们的包装类。  
