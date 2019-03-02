---
title: DataBinding & MVVM
date: 2019-02-11 16:18:01
tags: jetpack
categories: 技术
---

### DataBinding


因为DataBinding相关的Gradle默认是不开启的，但是确实存在系统中，所以手动去打开它。  
在app的目录下面 **`android{}`**里面加上这句开启命令。

```
dataBinding {
    enabled = true
}
```
<!--more-->
#### 简单的例子

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="person"
            type="com.zcy.nidavellir.mydatabinding.Person" />
    </data>
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{person.name}" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{person.age}" />
    </LinearLayout>
</layout>
```

定义了一个形式变量**`person`** 然后可以在view中直接引用。

```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = DataBindingUtil.setContentView<com.zcy.nidavellir.mydatabinding.databinding.ActivityMainBinding>(this, R.layout.activity_main)
        binding.person = Person("只是","123")
    }
}
```
这里使用了**`DataBindingUtil.setContentView`**来代替 `setContentView`

而泛型中的`ActivityMainBinding` 则为xml文件名的大驼峰命名转换加上Binding组成。

进而通过`binding`来赋值xml文件中的`person`变量

非常的简洁 ヾ(=･ω･=)o  

如果`person`有一个属性`val like: ObservableInt` 那么更改这个属性，会自动更新UI。

```
fun click(v: View) {
	person.like.set(person.like.get() + 1)
}
```
**注意！** 如果含有`ImageView`且引用了`R.drawable.xxx` 那么还需要在`data`标签中加上
**`<import type="com.zcy.nidavellir.mydatabinding.R"/>`**


### MVVM

#### Model层
负责数据的存储、网络请求、数据库以及I/O操作
#### View层
View层做的仅仅和UI相关的工作，View层不做任何业务逻辑、不涉及操作数据、不处理数据、UI和数据严格的分开。
#### ViewModel层
ViewModel 只做和业务逻辑和业务数据相关的事，不做任何和UI、控件相关的事


#### 举栗子

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable
            name="person"
            type="com.zcy.nidavellir.mydatabinding.PersonViewModel"/>
    </data>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:id="@+id/asdf"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{person.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{Integer.toString(person.likes)}" />

        <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{() -> person.onLike()}"
            android:text="+LIke" />
    </LinearLayout>
</layout>
```

我们在这里面绑定了View与Model

```
val viewModel = ViewModelProvider.NewInstanceFactory().create(PersonViewModel::class.java)
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val binding =
        DataBindingUtil.setContentView<com.zcy.nidavellir.mydatabinding.databinding.ActivityMainBinding>(this, R.layout.activity_main)
    binding.person = viewModel
}
```
在Activity中去赋值xml中的参数变量

***这里 AndroidX 中的ViewModelProvider 与 Support 中的ViewModelProvider 不太一样 ***

```
class PersonViewModel : ViewModel() {
    val name = ObservableField("Ada")
    val likes = ObservableInt(0)

    fun onLike() {
        likes.set(likes.get() + 1)
    }
}
```
最后在ViewModel中去处理点击事件