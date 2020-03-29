---
title: Core Java Chapter6
tags: Java
categories:
  - Programming Language
  - Java
---
## Interfaces, Lambda Expressions, and InnerClasses
> 每次读一本书读不完或者读完了又由于没有实践都忘记是真的很烦，作为这个系列的第一篇，希望可以开一个好头。主要的计划是将原本一本书一个md文件改为一个章节一个文件，也许会有帮助？ 冲鸭

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
