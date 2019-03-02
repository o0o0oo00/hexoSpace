---
title: 注解知识简述
date: 2018-10-10 16:22:24
categories: 技术
tag: android

---
### 注解知识点
#### 概念
注解是一种元数据形式，可以被添加到java源代码中，类、方法、变量、参数、包都可以被注解，注解对他们操作的代码没有直接的影响。
<!--more-->
#### 作用
1. 标记，用于告诉编辑器一些信息
2. 编译时动态处理，例如动态生成代码
3. 运行时动态处理，例如得到注解信息  

#### 自定义注解
1. 通过@interface定义，注解名即为自定义注解名
2. 注解配置参数名为注解类的方法名  
1. 所有方法没有方法体，没有参数没有修饰符，实际只允许 public & abstract 修饰符，默认为 public，不允许抛异常  
2. 方法返回值只能是基本类型，String, Class, annotation, enumeration 或者是他们的一维数组  
3. 若只有一个默认属性，可直接用 value() 函数。一个属性都没有表示该 Annotation 为 Mark Annotation
3. 可以加 default 表示默认值 

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface MethodInfo {

String author() default "o0o0oo00";

String date();

int version() default 1;
}
```
#### 元注解
它是一种基本注解，用于应用到定义的注解上面

* @Documented 是否会保存到 Javadoc 文档中
* @Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS，SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, SuppressWarnings
* @Target 可以用来修饰哪些程序元素  

>
ElementType.ANNOTATION_TYPE  可以给一个注解进行注解
ElementType.CONSTRUCTOR  可以给构造方法进行注解
ElementType.FIELD  可以给属性进行注解
ElementType.LOCAL_VARIABLE  可以给局部变量进行注解
ElementType.METHOD   可以给方法进行注解
ElementType.PACKAGE  可以给一个包进行注解
ElementType.PARAMETER  可以给一个方法内的参数进行注解
ElementType.TYPE  可以给一个类型进行注解，比如类、接口、枚举

未标注则表示可修饰所有

* @Inherited 是否可以被继承，默认为 false。意思是说，老子是富豪，那么老子的儿子虽然没有明确的指明为是富豪，但他也是富豪。
* @Repeatable 注解会多次应用，通常是注解的值可以同时取多个

```
@interface Persons {
Person[]  value();
}

@Repeatable(Persons.class)
@interface Person{
String role default "goodman";
}

@Person(role="motherfucker")
@Person(role="asshole")
@Person(role="bitch")
public class SuperMan{

}

```
**Attention!**  

1. 如果只有一个名为value的属性时，在使用的时候可以省略命名参数直接填入
2. 如果定义的时候一个属性都没有那么在使用的时候连括号都可以省略

#### 运行时注解解析
运行时 Annotation 指 @Retention 为 RUNTIME 的 Annotation，可手动调用下面常用 API 解析  
> method.getAnnotation(AnnotationName.class);
method.getAnnotations();  
method.isAnnotationPresent(AnnotationName.class);是否应用了某个注解   

```
public static void main (String[] args){
try {
Class cls = Class.forName("com.zcy.nidavellir.javaworld.java.App");
for (Method method : cls.getMethods()) {
MethodInfo methodInfo = method.getAnnotation(MethodInfo.class);
if (methodInfo != null) {
System.out.println("method name:" + method.getName());
System.out.println("method author:" + methodInfo.author());
System.out.println("method version:" + methodInfo.version());
System.out.println("method date:" + methodInfo.date());
}
}
} catch (ClassNotFoundException e) {
e.printStackTrace();
}
}
```
#### 编译时注解解析
编译时 Annotation 指 @Retention 为 CLASS 的 Annotation，由编译器自动解析

1. 自定义类集成自 AbstractProcessor
2. 重写其中的 process 函数  

实际是编译器在编译时自动查找所有继承自 AbstractProcessor 的类，然后调用他们的 process 方法去处理

