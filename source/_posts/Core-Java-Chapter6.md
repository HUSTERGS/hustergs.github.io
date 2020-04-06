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
> 又来写只给自己看到笔记了.每次读一本书读不完,或者读完了又由于没有实践就都忘记是真的很烦，作为这个系列的第一篇，希望可以开一个好头。主要的计划是将原本一本书一个md文件改为一个章节一个文件，也许会有帮助？ 
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

2. `interface`所有的方法自动设置为`public abstract`所以没有必要加上访问修饰符(Java 9似乎可以定义`private`方法，静态或者类方法，没有深入了解)。同时在实现(`implement`)一个接口的时候，所有的对应的方法都必须为`public`，与接口对应
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
6. `lambda`表达式由三个部分组成
   - 参数列表
   - 函数体
   - **自由变量**(free variable)

   自由变量即为lambda表达式可以捕获的外界的变量.Java对其有很严格的要求,只能是`effectively final`(An effetively final variable is a variable that is never assigned a new value after it has been initialized)
   ```Java
   public static void repeat(String text, int count) {
      for (int i = 1; i < count; i++) {
         ActionListener listener = event -> {
            System.out.println(i + ": " + text);
            // 出错,由于引用了i, 但是引用text是可以的
         }
      }
   }
   ```
7. `lambda`表达式同样有名称冲突的问题.作为lambda表达式参数的名字不能与`lambda`表达式所在的代码块含有的其他变量同名
8. `this`似乎可以由6来解释,如果在`lambda`表达式中使用`this`变量的话,指的也是环境中的`this`
9. 使用`@FunctionalInterface`是一个好主意
10. `Comparators`的一些其他的静态方法.
    - `comparing`可以用来自定义用于比较的`key`,比如将人(Person)根据名字来比较,此时要求用于比较的`key`本身是可以比较的(也即实现了`Comparable`接口)
      ```Java
      Arrays.sort(people, Comparator.comparing(Person::getName));
      // 参数可以是函数引用也可以是lambda表达式
      Arrays.sort(people, Comparator.comparing(Person::getName).thenComparing(Person::getFirstName)); // 甚至可以链式制定下一级的比较对象
      ```
    - 基本类型没有实现`Comparable`(甚至都不是类),所以`Comparator`有其他的针对基本类型的包装`comparing`函数
      ```Java
      Arrays.sort(people, Comparator.comparingInt(p -> p.getName().length()));
      ```
### 6.3 Inner Classes
1. 为什么需要内部类(`Inner Classes`)
   - 可以向其他的类隐藏内部类的存在
   - 内部类可以访问包含者的数据,甚至是private的数据
2. 本节使用以下例子
   ```Java
   public class TalkingClock
   {
      private int interval;
      private boolean beep;
      private TalkingClock(int interval, boolean beep) {...}
      public void start() {...}
      public class TimePrinter implements ActionListener
      // an inner class
      {
         public void actionPerformed(ActionEvent event)
         {
            System.out.println("At the tone, the time is "
               + Instant.ofEpochMilli(event.getWhen()));
            if (beep) Toolkit.getDefaultToolkit().beep();
         }
      }
   }
   ```
   可以看到内部类`TimerPrinter`访问了外部类中的`beep`成员,并且为`private`,同时我们可将内部类定义为`private`,这样的话就只有外部类可以创建并访问内部类,一般来说一个类都只能是`public`或者包访问权限,内部类是个例外
2. 内部类访问外部类的方法
   内部类访问外部类的正确姿势应该是如下:
   ```Java
   public void actionPerformed(ActionEvent event)
   {
      ...
      if (TalkingClock.this.beep) Toolkit.getDefautlToolkit().beep();
   }
   ```
   也即通过`OuterClass.this`来访问
3. 外部类创建内部类
   在外部类之中,创建内部类使用如下语法`outerObject.new InnerClass(construction parameters)`
   ```Java
   ActionListener listener = this.new TimrPrinter(); // this.可以去掉
   ```
   而当内部类被声明为`public`,并且从外部访问时(指外部类的外部),就需要明确指出`outerObject`
   ```Java
   var jabberer = new TalkingClock(1000, true);
   TalkingClock.TimePrinter listener = jabberer.new TimePrinter();
   ```
   内部类的**静态成员**必须为final,并用常量初始化,同时内部类**不能有静态方法**
   同时需要认识到内部类是在范围内产生影响,对于JVM来说根本不会意识到内部类的存在,编译器在编译阶段会将内部类转化为普通的类. 同时会通过一些方法来使得这个类能够访问外部类的私有属性,总之就是让其具有更高的访问权限.也正是由于这个原因,理论上是有办法通过内部类来破除外部类的访问限制的,但是仅仅从代码层面无法做到,需要对`.class`文件进行修改,所以一般不考虑
4. 本地内部类(`Local Inner Classes`)
   就是在类方法中定义的内部类,这种内部类没有访问权限控制,他们只能够在所属的方法中被访问,但是同时,它不仅能够访问外部类,而且可以访问该方法中的局部变量(只能够是`effectively final`)
   ```Java
   public start()
   {
      class TimePrinter implements ActionListener
      {
         public void actionPerformed(ActionEvent event)
         {
            System.out.println("At the tone, the time is "
               + Instant.ofEpochMilli(event.getWhen()));
            if (beep) Toolkit.getDefaultToolkit().beep();
         }
      }
      var listener = new TimePrinter();
      var timer = new Timer(interval, listener);
      timer.start();
   }
   ```
   实际上在1秒之后,局部变量`beep`已经不存在了,所以在内部类中访问的`beep`实际上是对原有的`beep`进行了复制(是只有基本类型是这样还是类也是这样呢?)


