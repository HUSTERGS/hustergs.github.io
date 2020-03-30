---
title: Core Java Chapter6
tags: Java
categories:
  - Programming Language
  - Java
abbrlink: a5c9fab8
date: 2020-03-30 15:31:22
---

## Interfaces, Lambda Expressions, and InnerClasses
> 每次读一本书读不完或者读完了又由于没有实践都忘记是真的很烦，作为这个系列的第一篇，希望可以开一个好头。主要的计划是将原本一本书一个md文件改为一个章节一个文件，也许会有帮助？ 冲鸭
### 6.1 Interfaces
1. 从Java5以来对于`Comparable`有了一些改动，提供了另外一个可选的泛型接口接口的`compareTo`函数由原来的接收一个`Object`作为参数，变为接收一个泛型`T`作为参数，在这种情况下可以免去强制类型转换以及类型判定的繁琐操作
   ```Java
   public interface Comparable<T>
   {
       int compareTo(T other);// 参数有了类型T
   }

   class Employee implements Comparable<Employee> {
       public int compareTo(Employee other) {
           return Double.compare(salary, other.salary);
           // 没有cast
       }
   }
   ```
   当然这也只是可选项，并没有强制要求
   > 对于Comparable，如果比较的是int，double等可能发生溢出的类型的时候(相减的时候)，需要使用对应的包装类(boxed class)的`compare`方法来避免溢出。

   > 同时理论上，对于大部分的类来说，当`equals`方法返回`true`的时候，`compareTo`方法也应该返回0，两者在这种情况下应该保持一致性，其中一个特例为`BigDecimal`，由于`equals`判定的时候考虑了精度

   > 理论上Comparable应该实现反向对称性(antisymmetry)，即`X.compareTo(Y) = - Y.compareTo(X)`，而继承有时候会破坏这一点(如果使用泛型的`Comparable`接口的话)
<!--more-->

2. `interface`所有的方法自动设置为`public`所以没有必要加上访问修饰符(Java 9似乎可以定义`private`方法，静态或者类方法，没有深入了解)。同时在实现(`implement`)一个接口的时候，所有的对应的方法都必须为`public`，与接口对应
3. `interface`可以定义方法以及常量(`public static final`)，但是**不能**定义成员(instance field)。同时Java8以后为接口内编写函数体提供了可能
4. `interface`可以被`extends`。与继承不同，一个类可以实现多个接口(implement multiple interfaces)，这也是接口与抽象类最大的不同(之一？)。不知道多个接口如果有同样的函数会发生什么。
5. `interface`中可以加入`static`方法，但是不会继承到实现这个接口的类中。换句话就是接口的static方法只能通过接口名进行调用
6. 通过使用`default`关键字可以给接口所定义的函数加上函数体，函数体中可以调用其他的函数
   ```Java
   public interface Collection
   {
       int size();
       default boolean isEmpty() {return size() == 0;}
   }
   ```
   问题：实现一个抽象类的时候一定要实现它所定义的所有方法吗？

   并不是，分情况讨论，抽象类实现接口可以不实现所有的方法，同时由`default`修饰的方法也可以不实现(会自动继承)，其他情况似乎都要实现(大概)
   `default`的另外一个作用就是提供可维护性，原来写的代码不会因为接口更新之后添加了非`default`的方法而无法编译
7. 当`dafault`函数之间出现冲突或者与父类的函数发生冲突的时候,两条原则
   - 父类强于接口
   - 接口之间发生冲突必须`Override`,事实上,即使两个接口之间一个通过`default`提供了实现,而另一个仅仅只是进行了生命也会发生冲突

8. 有一些类的比较函数我们无法改变,如`String`,这时就需要使用`Comparator`，比较的时候实例化一个`Comparator`并调用`compare`方法即可，也可以传入`sort`函数来自定义比较方法(其实有点奇怪`compare`为什么不设计成静态方法)，当然也可以使用`lambda`函数传入`sort`
9. 一旦为一个类制定了相应的clone策略,那么他的所有子类都必须制定相应的策略,否则有可能出现成员是一个可变对象(*mutable*)而导致错误的情况(深拷贝).所以有些人建议尽可能少的使用clone,事实上只要不到5%的Java类实现了Cloneable接口.
10. 在Java1.4之前 clone方法只能返回Object对象,但是现在我们可以自定义返回类型从而和类相符合

### 6.2 Lambda Expressions
1. 例子
   ```Java
   (String first, String second) -> {
       if (first.length() < second.length()) return -1;
       else if (first.length() > second.length()) return 1;
       else return 0;
   }
   // 省略了类型
   Comparator<String> comp = (first, second) -> first.length() - second.length(); 
   // 允许函数在某些情况下返回值而其他情况不返回
   (int x) -> {if (x >= 0) return 1;}
   ```

2. 关于接口中的抽象函数和非抽象函数
3. 在Java中lambda表达式只能作为interface(functional interface)的替代,甚至都不能赋值给一个`Object`对象(由于`Object`不是一个`functional interface`),只能赋值给functional interface
4. 函数引用(function reference)
   ```Java
   var timer = new Timer(1000, event->System.out.println(event));
   // 等价于
   var timer = new Timer(1000, System.out::println);
   ```
   其中`System.out::println`即为函数引用,函数引用在使用的时候实际上会被编译器转化为对应的interface,并重写(Override)其中**唯一**的抽象方法(abstract method).可能出现函数本身有重载的情况,由于在使用函数引用的时候并没给出函数签名,而只是给出了函数名. 如此处的`println`方法有10个重载方法.此时编译器会根据其要转换的interface中抽象方法的签名来选择合适的函数.(上述例子会选择println(Object))
   ```Java
   Runnable task = System.out::println;
   // 其中Runnable接口中有唯一的方法void run(),所以选中的是println()方法
   task.run(); // 相当于System.out.println();
   // 其他的例子
   list.removeIf(Objects::isNull);
   // 等价于 list.removeIf(e -> e == null);
   ```
5. 对于构造函数的引用是`ClassName::new`,对于构造函数的引用可以解决Java本身存在的一些缺陷,如没有泛型数组.`new T[n]`会被自动变为`new Object[n]`.使用构造函数的引用可以在某种程度上解决这个问题
   ```Java
   Object[] people = stream.toArray();// 不满足要求
   Person[] people = stream.toArray(Person[]:new);
   ```

