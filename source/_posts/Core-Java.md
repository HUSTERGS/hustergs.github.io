---
title: Core Java
date: 2019-5-16 14:15:57
tags: Java
categories: 
    - Programming Language
    - Java
---

# Core Java

## Chapter 3

### ***Fundamental Programming Structures in Java***

1. use `javac filename.java` to compile java files and use `java filename` to run it. First step will generate a `filename.class` file.

2. keyword `public` is a `modifier` which controls the level of access

3. everything in Java program lives inside a class.

4. the length of java class is essentially ***unlimited***

5. the filename must be the same as the classname defined in the `.class` file

6. the `main method` must be decleared `public`. or more specificly like this 

   ``` Java
   public class ClassNmae
   {
       public static void main(String [] args)
       {
           program statements
       }
   }
   ```

7. To terminate the program with a different exit code, use `System.exit` method

<!--more-->


8. Java is a strongly typed language. this meas that every variable must have a decleared type. there are eight primitive types in Java.

   * Integer Types    

     * int 
     * short 
     * long
     * byte

     > You can write long integer, Hexadecima, Octal and binary number with `-L`, `0x-`, `0-` and `0b-`

   * Floating-Point Types

     * float
     * double

     > 1. the limitrf precision of `float` is simply not sufficient for many situantions. Use `float` values ***only*** when you work with a library that requires them, or when you need to store a very large number of them 
     > 2. use `Double.isNaN()` instead of compare with `Double.NaN` to test whether `x` is `NaN`
     > 3. use `BigDecimal` class, if you need precise numerical computations without roundoff errors like `2.0 - 1.1 == 0.8999999`

   * char

   > unicode escape sequences are processed before the code is parsed. so `public static void main(String\u0058\u005D args)` is perfectly legal

   * boolean

     > different from C/C++. expressions like `x = 0` can not be used in Java as a condition because you can not convert between `integers` and `boolean` values

9. the letter and digit you can use in a variable name is much more wider than other languages especally C/C++

10. use `final` to denote a constant, `final double PI = 3.14`. `final` indicates that you can assign to the variable once, and then its value is set once and for all. And use `static final` to create a `class constants` which is available to multiple methods inside a single class. 

    > `const` is a reserved keyword, but not currently used for angthing

    ``` Java
    public class Constants
    {
        public static final double CM_PER_INCH = 2.54;
    
        // ......
    }
    ```


11. Occasionally need Mathmatical Functions and Constants

    * `Math.sqrt()`
    * `Math.pow(x,a)`
    * `Math.florrMod()`
    * `Math.sin()`
    * `Math.exp()`
    * `Math.log()`
    * `Math.log10()`
    * `Math.PI`
    * `Math.E`

12. ``` java
    double x = 9.997;
    int nx = (int) x; //x = 9
    
    int nx2 = (int) Math.round(x); // x = 10
    ```

13. Sometime, a variable should only hold a restricted set of values. Use `enum`

    ```java
    enum Size {SMALL, MEDIUM, LARGE, EXTRA_LARGE};
    Size s = Size.MEDIUM;
    ```

14. * `string.substring(startPosition, endPosition)`

    * `string3 = string1 + string2`

    * `stringnum = string + num`

    * `String.join()`

    * `string.equals(t)`

    * `string.equalsIngnoreCase(t)`

      > java strings behave like pointers, do not use `==` to test whether two strings are equal

    * use `string.charAt(int index)` instead of `[]`

15. `Strings` Are Immutable

16. Building Strings

    > It will be inefficient to use string concatenation for this purpose  ( `+` ). Every time you concatenate strings, a new string object is constructed which consums time and wastes memory. Using the `StringBuilder` class to avoid this problem

    ``` java
    StringBuilder builder = new StringBuilder();
    builder.append(ch);
    builder.append(string);
    // when done
    String completedString = builder.toString();
    ```

17. Reading Input

    ``` java
    // System.in is not quite as simple as System.out
    import java.util.*;
    // Scanner is part of java.util
    public class input {
        public static void main(String[] args) {
            Scanner in = new Scanner(System.in);
    
            System.out.print("What is   your name? ");
            String name = in.nextLine();
            // Here we use the nextLine method, because the input might contain spaces
            // To read a single word, call
            // String firstName = in.next();
    
            // To read a integer, call
            // int age = in.nextInt();
    
        }
    }
    ```

18. There is no `goto` in Java, while there is a "labeled" version of `break` that you can use to break out of a nested loop. There is also a labeled version of `continue`

19. You can not redefine a variable inside a nested block hoping the inner definition will shadows the outer one. Java would rather not compile 

20. A `case` label can be

    * a constant expression of type `char`, `bytr`, `short`, or `int`
    * An enumerated constant
    * Starting with Java SE 7, a string literal

21. Unlike C++, Java has no programmable operator overloading

22. Array

      

     ```java
     int [] a;
     // initialize a array
     // array type is specified by `[]`
     int [] a = new int[100]
     // use the `new` operator to create the array 
     
     // The array length need not be a constant:
     new int[n]
     
     // And there are actually two way to define a array
     //1. 
     int [] a;
     //or 
     int a[];
     // But most Java programmer prefer the former style, because it neatly separates the type int [] from the variable name
     ```

 When you create an array of numbers, all elements are initialized with zero. Arrays of `boolean` are initialized with `false`. Array of objects are initialized with the special value `null`
    

    ```java
    String[] names = new String[10];
    // all the strings are null
    ```
    
    index out of range will cause a `array index of bounds` exception.

23. the ***enhanced*** `for` loop

    `for (variable : collection) statement`

    ```java
    for (int element : a)
        System.out.println(element)
    
    ```

24. ```java
    int[] smallPrimes = {2,3,5,7,11,13}
    // anonymous array
    new int[] {17, 19, 23, 29, 31, 37}
    // 字面量似乎除了初始化之外，好像不能在其他地方使用
    // 如果想使用字面量只能使用上面那种方式
    
    ```

25. ```java
    new elementType[0]
    // 创建一个长度为0的数组，注意长度为0并不代表是 null
    
    ```

26. 使用`Arrays.copyOf(source, length)` 来生成一个数组的拷贝，如果length超过了`source`的长度，多出来的位置填充 `0` / `false`

27. 和C/C++不同，Java中Main 函数的参数之中并不包括程序的名字

    ```
    java Message -h  world
    // args[0] 是 "-h" 而不是 Message
    
    ```

28. `Array.sort(target)` sort target

29. 多维数组

30. 不规则数组，C++好像不支持？

    

    

## Chapter 4

### ***Object and Classes***

1. You can only have one public class in a source file, but you can have any number of nonpublic classes

2. There is an important difference between constructors and other methods. A constructor can only be called in conjunction with the new operator. You can’t apply a constructor to an existing object to reset the instance fields. For example,4.3 Defining Your Own Classes

   `james.Employee("James Bond", 250000, 1950, 1, 1)` // ERROR
   is a compile-time error.

3. keep the following in mind

   * A constructor has the same name as the class.
   * A class can have more than one constructor.
   * A constructor can take zero, one, or more parameters.
   * A constructor has no return value.
   * A constructor is always called with the new operator.

4. 如果你要在对象的方法中返回一个可以改变的对象，比如`Date`类型的员工的入职时间，应该先将其 `clone` 一遍，否则将会破坏对象的包装(`encapsulation`)

   As a rule of thumb, always use clone whenever you need to return a copy of a mutable field.

5. 一个对象的方法可以访问所有该类型的`private` 类型的属性，不管是不是对象本身。C++也可以

   ```java
   public boolean quals(Employee other)
   {
   	return name.equals(other.name); // 可以访问 other 对象的 private 属性 name
   }
   
   ```

6. Final Instance Fields 这种类型的属性必须要在构造函数中被初始化，一旦初始化便不再能更改

7. Static Fields and Methods

   If you define a field as static , then there is only one such field per class. In contrast, each object has its own copy of all instance fields.

   静态方法/字段 不依赖于具体的实例对象，它是 `class` 固有属性

8. Static Constant 比如 `Math.PI`

   > NOTE: If you look at the System class, you will notice a method `setOut` that sets System.out to a different stream. You may wonder how that method can change the value of a final variable. However, the `setOut` method is a native method, not implemented in the Java programming language. Native methods can bypass the access control mechanisms of the Java language. This is a very unusual workaround that you should not emulate in your programs.

9. 使用类来调用静态方法而不是使用对象，虽然后者确实行得通

10. 工厂函数被声明为静态

11. 函数重载

     Java允许你重载所有函数，不仅仅是构造函数，但是需要注意的是，函数的返回值并不作为函数签名的一部分，所以你不能写两个名字和参数都相同但是返回值不同的函数

12. 如果你没有指定任何构造函数，那么就会使用默认值初始化对象，但是一旦你指定了构造函数但是在调用的时候又没有传入应有的参数，就会报错

13. Java 中可以直接为对象的字段指定初始值，C++不行，同时Java也支持在指定初始值的时候不使用常量，可以使用函数调用等等

     ```java
     class Employee
     {
     	private static int nextId;
     	private int id = assignId();
     	. . .
     	private static int assignId()
     	{
     		int r = nextId;
     		nextId++;
     		return r;
     	}
     	. . .
     }
    
     ```

14. 如果一个构造函数的 ***第一个语句*** 是`this(...)`的形式的话，那么该构造函数就会调用对象中的另外一个构造函数，这在编写很多相似的构造函数的时候很有用

     ```java
     public Employee(double s)
     {
     	// calls Employee(String, double)
     	this("Employee #" + nextId, s);
     	nextId++;
     }
    
     ```

15. 初始化一个对象的数据段有3种方法

     * 在构造函数之中设置值
     * 在数据段声明的时候赋值
     * ***使用 `initialization block`*** (不是很常见)

     ```java
     class Employee
     {
     	private static int nextId;
     	private int id;
     	private String name;
     	private double salary;
     	// object initialization block
     	{
     		id = nextId;
     		nextId++;
     	}
     	public Employee(String n, double s)
     	{
     		name = n;
     		salary = s;
     	}
     	public Employee()
     	{
     		name = "";
     		salary = 0;
     	}
     	. . .
     }
    
     ```

     不论哪一个构造函数被使用进而生成一个对象，`initialization block` 总是先执行，然后再执行构造函数(按照在对象中出现的顺序)

16. 使用`finalize`来达到类似于C++之中的析构函数

17. 使用`import static` 来导入静态方法而不只是类本身

18. 使用`package` 来打包一个类

## Chapter 5

### ***Inheritance***

1. 使用 `extends` 关键字来继承一个类

   ```java
   public class Manager extends Employee
   {
   	added methods and fields
   }
   
   ```

   
