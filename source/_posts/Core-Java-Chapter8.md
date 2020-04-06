---
title: Core Java Chapter8
tags: Java
categories:
  - Programming Language
  - Java
abbrlink: 4271d7bf
date: 2020-04-06 14:19:10
---


## Generic Programming
### 8.2 Defining a Simple Generic Class
1. 本节使用`Pair`作为例子,代码如下
   ```Java
   public class Pair<T>
   {
       private T first;
       private T second;
       public Pair() {first = null; second = null;}
       public Pair(T first, T second) {this.first = first; this.second = second;}
       public T getFirst() { return first; }
       public T getSecond() { return second; }
       public void setFirst(T newValue ) {first = newValue; }
       public void setSecond(T newValue ) {second = newValue;}
   }
   ```
2. 泛型类可以定义多个类型变量(常用`T`, `K`, `V`, `U`, `S`, `E`)

### 8.3 Generic Methods
1. 泛型方法
   ```Java
   class ArrayAlg
   {
       public static <T> T getMiddle(T... a)
       {
           return a[a.length / 2];
       }
   }
   // 调用
   String middle = ArrayAlg.<String>getMiddle("John", "Q.", "Public");
   // 或者省去<String>
   String middle = ArrayAlg.getMiddle("John", "Q.", "Public");
   ```
   大部分情况下类型推断都可以正常运作,但当传入的参数不是一致的类型时可能会发生问题,编译器会尝试寻找不同类型的共同父类来解决问题
<!--more-->

### 8.4 Bounds for Type Variables
1. bounding type(不知道怎么翻译)
   `<T extends BoundingType>`表示传入的类型T至少为BoundingType的子类,其中BoundType可以是一个类(Class),也可以是一个接口(Interface).并使用`&`来分隔对某一个类型的多个限制,如`<T extends Comparable & Serializable>`

### 8.5 Generic Code and the Virtual Machine
本节主要说明泛型背后的秘密
1. 类型说明符首先被替换为`Object`,如果有bounding type的话将换为第一个bounding type
   ```Java
   // 原始
    public class Interval<T extends Comparable & Seiralizable> implements Serializable
    {
        private T lower;
        private T upper;
        ...
        public Interval(T first, T second) {...}
    }
    // 被替换为
    public class Interval implements Serializable
    {
        private Comparable lower;
        private Comparable upper;

        public Interval(Comparable first, Comparable second) {...}
    }
   ```
   > 如果将`Serializable`和`Comparable`的顺序替换一下,那么所有的`T`将会被转化为`Serializable`,必要的时候会进行强制类型转换,而这必然会产生一定的消耗,所以建议将`tagging interface`(表示没有方法的接口)放到最后
   每一次访问泛型类或者方法的时候都会自动插入强制类型转换(cast)
2. 按照1所说的,泛型并不是生成了一族类或者函数,而只是生成了一个,必要的时候进行强制类型转换,这与C++中`template`有着本质上的不同
3. 当泛型类被继承的时候，由于上述所谓*erasure*的过程，和多态相作用可能会产生一些其他问题
   ```Java
   class DateInterval extends Pair<LocalDate>
   {
       public void setSecond(LocalDate second)
       {
           ...
       }
   }
   var interval = new DateInterval(...);
   Pair<LocalDate> pair = interval;
   pair.setSecond(aDate);
   ```
   由于`pair`声明类型为`Pair`,而`Pair`类型只有一个以`Object`为参数的`setSecond`方法,所以会调用`Pair`中的方法,而不是`DateInterval`中的方法,函数签名不同,所以不考虑后期绑定.为了解决这个问题,编译器会自动生成桥接方法(*bridge method*) `public void setSecond(Object second) { setSecond((LocalDate) second); }`(有点不懂)
### 8.6 Restrictions and Limitations
> 实际上可以认为泛型`T`在运行时完全不可用,只有类型检查等编译时可用操作的才能使用`T`
1. 运行时的类型检查只能看到原始类型,而不能看到泛型类型,如
   ```Java
   // 类型检查
   if (a instanceof Pair<String>) // ERROR, 只能判定是否是Pair而不能加入<String>
   // 强制类型转换
   Pair<String> p = (Pair<String>) a; // warning -- can only test that a is a Pair
   //getClass方法
   Pair<String> stringPair = ...;
   Pair<Double> doublePair = ...;
   stringPair.getClass() == doublePair.getClass() // 相等,都等于Pair.class
   ```
2. 无法创建泛型类的数组?如new Pair<String>[10];如果非要使用的话请使用`ArrayList`,唯一的特例是变长参数传递时,允许泛型数组
3. 无法创建泛型的实例如`new T()`
4. 在泛型类的静态方法中无法使用泛型,由于泛型的确定是随着泛型类的实例化而发生的.所以静态方法使用泛型
5. ....

### 8.7 Inheritance Rules for Generic Types
1. 数组具有协变性, `Father[]`也是`Child[]`的父类，但是泛型不具有协变性`GenericClass<Father>`却不是`GenericClass<Child>`的父类


### 8.8 Wildcard Types
> 这一节似乎都在讨论泛型容器,而不是普遍的泛型
1. 子类
   `Pair<? extends Employee>`
   类型`Pair<Manager>`是`Pair<? extends Employee>`的子类(某种意义上的协变性),可以用于函数的复用
   子类型的泛型限制会导致泛型容器只能够读取而不能写入
2. 父类
   `Pair<? super Manager>`可写不可读(似乎还是可以读,只是只能用`Object`来接收)

3. 无限制
   `Pair<?>`,返回值只能被复制给`Object`,并且不能写入
   注意到这时没有一个具体类型的替代符号(如`T`),那我们该如何获取具体的类型呢,这时需要另外编写帮助函数,通过另外一个普通泛型函数来获取具体的类型,如
   ```Java
   public static <T> void swapHelper(Pair<T> p)
   {
       T t = p.getFirst();
       ....
   }

   public static void swap(Pair<?> p) 
   {
       swapHelper(p);
   }
   ```
   > 其实有点不懂`<?>`和直接`<T>`的区别

### 8.9 Reflection and Generics
