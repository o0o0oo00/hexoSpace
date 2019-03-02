---
title: Use Dagger2 With Kotlin
date: 2019-02-26 13:52:21
tags: dagger
categories: 技术

---

## Use Dagger2 With Kotlin


### 添加Dagger依赖

```
dependencies {
    kapt 'com.google.dagger:dagger-compiler:2.16'
    implementation 'com.google.dagger:dagger:2.16'
```
并且 要再文件的头部添加上apply

```
apply plugin: 'kotlin-kapt'
```

这样就会在`/app/build/generated/source/kapt/debug/com/zcy/mydagger`下生成对应的代码来实现注入
<!--more-->
### 被注入对象
#### 手写代码
```
class Car @Inject constructor() {

}
```
#### 生成的代码
仔细阅读代码发现

调用的顺序似乎是 `create(创建 INSTANCE )` -> `get()` -> `provideInstatnce()` -> `new Car()`

来生产出一个 **Car** 对象

```
public final class Car_Factory implements Factory<Car> {
  private static final Car_Factory INSTANCE = new Car_Factory();

  @Override
  public Car get() {
    return provideInstance();
  }

  public static Car provideInstance() {
    return new Car();
  }

  public static Car_Factory create() {
    return INSTANCE;
  }

  public static Car newCar() {
    return new Car();
  }
}
```

### 使用被注入的对象
在kotlin中 需要注入到属性 set 中去

```
class Man {
    lateinit var car:Car
        @Inject set
```

### Component 搭桥
#### 手写代码
```
@Component
interface ManComponent {
    fun inject(man: Man)
}
```
此时build之后会生成一些代码 比如`DaggerManComponent` 约定多了一个`Dagger`前缀
#### 生成代码

```
public final class DaggerManComponent implements ManComponent {
  private DaggerManComponent(Builder builder) {}

  public static Builder builder() {
    return new Builder();
  }

  public static ManComponent create() {
    return new Builder().build();
  }

  @Override
  public void inject(Man man) {
    injectMan(man);
  }

  private Man injectMan(Man instance) {
    Man_MembersInjector.injectSetCar(instance, new Car());
    return instance;
  }

  public static final class Builder {
    private Builder() {}

    public ManComponent build() {
      return new DaggerManComponent(this);
    }
  }
}
```
调用的顺序似乎是 `create()` -> `inject(Man man)`   
其中
**`create() == Builder().build()`**  
而在`build()`方法中返回了一个**DaggerManComponent**实例，通过这个实例，来调用`inject()`  
而 `inject()`里面调用的是  
**`Man_MembersInjector.injectSetCar(instance, new Car());`**  
最后返回**instance** 也就是传进来的**Man**

其他都没啥 ，关键是**`Man_MembersInjector.injectSetCar(instance, new Car());`**   
而这方法应该是Man(使用注入对象的Man所生成的代码)  
猜测是将`new Car()`对象赋值到了`instance`中  
**继续看下面**

### 完成注入
#### 手写代码
最后成功输出了**`com.zcy.mydagger.Car@6ff3c5b5`**

```
class Man {
    init {
        DaggerManComponent.create().inject(this)
    }

    lateinit var car: Car
        @Inject set

    fun a() {
        println(car)
    }


}

fun main(args: Array<String>) {
    println(Man().a())
}
```
#### 生成的代码

```
public final class Man_MembersInjector implements MembersInjector<Man> {
  private final Provider<Car> p0Provider;

  public Man_MembersInjector(Provider<Car> p0Provider) {
    this.p0Provider = p0Provider;
  }

  public static MembersInjector<Man> create(Provider<Car> p0Provider) {
    return new Man_MembersInjector(p0Provider);
  }

  @Override
  public void injectMembers(Man instance) {
    injectSetCar(instance, p0Provider.get());
  }

  public static void injectSetCar(Man instance, Car p0) {
    instance.setCar(p0);
  }
}

```
此处正好证实了上面的猜想，**`injectSetCar`** 方法中 `instance.setCar(p0);`正好是将**car**这个属性进行赋值，恰好就是我们**component**中的`new car()`


简单的代码分析就酱