---
title: Java反射
date: 2018-10-12 17:50:17
categories: 技术
---
### Java反射

#### Class的获取
* Class是一个类，不同于小写的class是关键字，不过它没有公开的构造方法，所以只能用`getClass()`方法获取相应的Class对象。

```
Class aClass = new car().getClass();
```
<!--more-->
**Attention~**

* 这种方法不适合基本类型如 int、float 等等，也可以用
`.class`来获取

```
Class aClass = new car().getClass();
Class aClass1 = car.class;
Class aClass2 = int.class;
Class aClass3 = String.class;
```
* `Class.forName(包名+类名)`例如某些类加上了@hide注解，表示这个类不会出现在SDK中，那么我们没办法获得Car，索性就直接传入它的名字好了。

#### Class的名字
1. `getName()`获取到的名字包含包名
2. `getSimpleName()` 相较于`getName()`去掉了包名，注意，匿名内部类获取的simpleName是一个空的字符串
3. ` getCanonicalName()`官方名字，如果没有返回null

#### Class获取修饰符
Java中对于一个类的修饰符有很多
我们可以这样获取他们

```
System.out.println(Modifier.toString(testModeify.class.getModifiers()));
```
结果为：public abstract  
还提供了一系列的判断方法如：
isInterface、isAbstract、isSynchronized、isFinal等等
```
System.out.println(Modifier.isPrivate(testModeify.class.getModifiers()));
```
结果为false

#### 获取Class成员
##### 获取属性
* `getDeclaredField(String name) `获取的是Class中的属性，不包含父类中的属性。
* `getField() `获取Class中的Public属性，并且如果获取不到的话，会向其父类获取。 
*  `public Field[] getDeclaredFields()`获取所有的属性，但不包括从父类继承下来的属性
*  `public Field[] getFields()`获取自身的所有的 public 属性，包括从父类继承下来的。
Look at these 注释掉的是不合法的。留下的就一目了然了。

```
Class son =  Son.class;
try {
System.out.println(son.getDeclaredField("sonStrDef"));
System.out.println(son.getDeclaredField("sonPub"));
System.out.println(son.getDeclaredField("sonPro"));
System.out.println(son.getDeclaredField("sonPri"));
//            System.out.println(son.getDeclaredField("fatherStr"));//error
//            System.out.println(son.getDeclaredField("fatherPub"));
//            System.out.println(son.getDeclaredField("fatherPri"));
//            System.out.println(son.getDeclaredField("fatherProt"));

//            System.out.println(son.getField("sonStrDef"));
System.out.println(son.getField("sonPub"));
//            System.out.println(son.getField("sonPro"));
//            System.out.println(son.getField("sonPri"));
//            System.out.println(son.getField("fatherStr"));
System.out.println(son.getField("fatherPub"));
//            System.out.println(son.getField("fatherPri"));
//            System.out.println(son.getField("fatherProt"));

} catch (NoSuchFieldException e) {
e.printStackTrace();
}
```

##### 获取方法
与获取属性一样

* `getDeclaredMethod`
* `getMethod `
* `getDeclaredMethods `
* `getMethod `

##### 获取构造方法
一样

#### Field类型的获取
* `public Type getGenericType() {}`
* `public Class<?> getType() {} `

不同的是两者**返回的类型**不一样，`getGenericType()` 方法能够获取到**泛型**类型。

看一个例子：

* 定义一个公开list集合

```
public List<String> strs = new ArrayList<>();
```

* 然后通过反射获取这个属性

```
for (Field field : son.getFields()) {
System.out.println(field.getName());
System.out.println(field.getType());
System.out.println(field.getGenericType());
}

```
输出为：

```
strs
interface java.util.List
java.util.List<java.lang.String>
```
#### Field读取与赋值
Field这个类定义了一系列的`set/get`方法来获取不同类型的值。  
例如：  
`public Object get(Object obj);`
`public void set(Object obj, Object value);`   
Object参数的作用是为了精确到到底修改的是哪一个对象
>Class 本身不对成员进行储存，它只提供检索，所以需要用 Field、Method、Constructor 对象来承载这些成员，所以，针对成员的操作时，一般需要为成员指定类的实例引用。如果难于理解的话，可以这样理解，班级这个概念是一个类，一个班级有几十名学生，现在有A、B、C 3 个班级，将所有班级的学生抽出来集合到一个场地来考试，但是学生在试卷上写上自己名字的时候，还要指定自己的班级，这里涉及到的 Object 其实就是类似的作用，表示这个成员是具体属于哪个 Object。这个是为了精确定位。  

###### 例子
修改一个类中的Public属性值

```
Class son =  Son.class;//Class
Son s = new Son();// Object 用于精确修改的位置
Son s2 = new Son();// Object 用于精确修改的位置
s.sonInt = 111;
s2.sonInt = 222;
try {
Field field = son.getField("sonInt");
int a = field.getInt(s);
int a1 = field.getInt(s2);
System.out.println("field a = " + a);
System.out.println("field a1 = " + a1);
field.setInt(s,1111);
field.setInt(s2,2222);
System.out.println("field s.sonInt = " + s.sonInt);
System.out.println("field s2.sonInt = " + s2.sonInt);
} catch (NoSuchFieldException e) {
e.printStackTrace();
} catch (IllegalAccessException e) {
e.printStackTrace();
}
```

```
field a = 111
field a1 = 222
field s.sonInt = 1111
field s2.sonInt = 2222
```
可以看到，一个field可以根据Object的参数不同，获取的值也不同，同样修改的值也不同。

上面是修改的Public的属性值，如果要修改private属性的话，直接用上面的方法会报错。`can not access a member`
应该加上**`field.setAccessible(true);`**就万事大吉了。

#### 获取方法参数
* `public Parameter[] getParameters() {}`
* Parameter.java
* `public String getName() {}`// 获取参数名字
* `public Class<?> getType() {}`// 获取参数类型
* `public int getModifiers() {}`// 获取参数的修饰符
* Method.java
* `public Class<?>[] getParameterTypes() {}`// 获取所有的参数类型
* `public Type[] getGenericParameterTypes() {}`// 获取所有的参数类型，包括泛型

看一组例子：  
Son.java

```
public class Son extends Father {

void normalMethod(String name, int num) {
}

public void test(String[] paraa, List<String> b, HashMap<Integer, Son> maps) {
}

public void exe() {
System.out.println("exexexeexexeexeexeexeee");
}
}
```
```
Class son = Son.class;

Method[] declaredMethods = son.getDeclaredMethods();
for (Method method : declaredMethods) {
System.out.println(method.getName());
// 参数类型
Parameter[] parameters = method.getParameters();
for (Parameter parameter : parameters) {
System.out.println("parameter name = " + parameter.getName() + " " + parameter.getType().getName());
}

Class<?>[] parameterTypes = method.getParameterTypes();
System.out.println("method para types:");
for (Class<?> type : parameterTypes) {
System.out.print(" " + type.getName());
}
System.out.println();

Type[] genericParameterTypes = method.getGenericParameterTypes();
System.out.println("method para generic types:");
for (Type type : genericParameterTypes) {
System.out.print(" " + type.getTypeName());
}
System.out.println();
System.out.println("==========================================");

}
```
来看一下输出，很清晰

```
test
parameter name = arg0 [Ljava.lang.String;
parameter name = arg1 java.util.List
parameter name = arg2 java.util.HashMap
method para types:
[Ljava.lang.String; java.util.List java.util.HashMap
method para generic types:
java.lang.String[] java.util.List<java.lang.String> java.util.HashMap<java.lang.Integer, com.zcy.nidavellir.javaworld.Reflection.Son>
==========================================
normalMethod
parameter name = arg0 java.lang.String
parameter name = arg1 int
method para types:
java.lang.String int
method para generic types:
java.lang.String int
==========================================
exe
method para types:

method para generic types:

==========================================
```
#### Method 获取返回值类型
* `public Class<?> getReturnType() {}`// 获取返回值类型
* `public Type getGenericReturnType() {}`// 获取返回值类型包括泛型

#### Method 获取修饰符
* `public int getModifiers() {}`

#### Method 获取异常类型
* `public Class<?>[] getExceptionTypes() {}`
* `public Type[] getGenericExceptionTypes() {}`

#### Method 方法的执行
* `public Object invoke(Object obj, Object... args) {}`

说明一下:

* 第一个参数是反射Class对应类的一个实例。如果是静态方法，那么传`null`。后面的参数对应方法Method的参数
* `invoke()` 返回的对象是 Object，所以实际上执行的时候要进行强制转换。
* 在对 Method 调用 `invoke()` 的时候，如果方法本身会抛出异常，那么这个异常就会经过包装，由 Method 统一抛出 `InvocationTargetException`。而通过 `InvocationTargetException.getCause()` 可以获取真正的异常。 
* 
例子：

```
public class testInvoke {
public static void staticMethod(){
System.out.println("我是个静态方法，我被调用啦");
}

private  int add (int a,int b ) {
return a + b;
}

public void testException () throws IllegalAccessException {
throw new IllegalAccessException("You have some problem.");
}
}

```

我们来验证一下：

```
Class cls = testInvoke.class;
try {

Method method = cls.getMethod("staticMethod", null);// 参数类型的Class，没有参数所以null
method.invoke(null, null);// 静态方法，没有参数，所以都是null

testInvoke testInvoke = new testInvoke();
Method method1 = cls.getDeclaredMethod("add", int.class, int.class);
method1.setAccessible(true);
int result = (int) method1.invoke(testInvoke, 2, 3);
System.out.println("result = " + result);

Method method2 = cls.getDeclaredMethod("testException", null);
method2.invoke(testInvoke, null);

} catch (NoSuchMethodException e) {
e.printStackTrace();
} catch (SecurityException e) {
e.printStackTrace();
} catch (IllegalAccessException e) {
e.printStackTrace();
} catch (IllegalArgumentException e) {
e.printStackTrace();
} catch (InvocationTargetException e) {
// 通过 InvocationTargetException.getCause() 获取被包装的异常
System.out.println("testException occur some error,Error type is :" + e.getCause().getClass().getName());
System.out.println("Error message is :" + e.getCause().getMessage());
e.printStackTrace();
}
```
输出：

```
我是个静态方法，我被调用啦
我是add方法，我被调用啦
result = 5
我是testException方法，我被调用啦
testException occur some error,Error type is :java.lang.IllegalAccessException
Error message is :You have some problem.
java.lang.reflect.InvocationTargetException
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at com.zcy.nidavellir.javaworld.Reflection.main.main(main.java:110)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at com.intellij.rt.execution.application.AppMainV2.main(AppMainV2.java:131)
Caused by: java.lang.IllegalAccessException: You have some problem.
at com.zcy.nidavellir.javaworld.Reflection.testInvoke.testException(testInvoke.java:20)
... 10 more


```
