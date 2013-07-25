---
layout: post
title: "Scala in Impatient 习题解答8"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. Extend the following BankAccount class to a CheckingAccount class that charges $1 for every deposit and withdrawal.

    ```scala
    class BankAccount(initialBalance: Double) {
        private var balance = initialBalance
            def deposit(amount: Double) = { balance += amount; balance }
        def withdraw(amount: Double) = { balance -= amount; balance }
    }
    // 一种实现
    class CheckingAccount(initialBalance: Double) extends BankAccount(initialBalance) {
      override def deposit(amount: Double) = { super.deposit(amount - 1) }
      override def withdraw(amount: Double) = { super.withdraw(amount + 1) }
    }
    ```
2. 用笨方法写的。

    ```scala
    class SavingsAccount(initialBalance: Double) extends BankAccount(initialBalance) {
      private var freeCount = 3
      private val interestRate = 0.03

      def earnMonthlyInterest() {
        freeCount = 3
        super.deposit(super.deposit(0) * interestRate)
      }

      override def deposit(amount: Double) = { 
        if (freeCount > 0) {
          freeCount -= 1
          super.deposit(amount)
        } else {
          freeCount -= 1
          super.deposit(amount - 1)
        }
      }

      override def withdraw(amount: Double) = {
        if (freeCount > 0) {
          freeCount -= 1
          super.withdraw(amount)
        } else {
          freeCount -= 1
          super.withdraw(amount + 1)
        }
      }
      ```
3. 

    ```scala
    abstract class Animal {
      def run
    }

    class Cat extends Animal {
      override def run = println("I can run, miao")
    }

    class Dog extends Animal {
      override def run = println("I can run, wang")
    }
    ```

4. Define an abstract class Item with methods price and description. A SimpleItem is an item whose price and description are specified in the constructor. Take advantage of the fact that a val can override a def. A Bundle is an item that contains other items. Its price is the sum of the prices in the bundle. Also provide a mechanism for adding items to the bundle and a suitable description method.

    ```scala
    abstract class Item {
      def price: Double
      def description: String
    }

    class SimpleItem(override val price: Double, override val description: String) extends Item {
    }

    class Bundle(val item: Item) extends Item {
      val itemList = scala.collection.mutable.ArrayBuffer[Item]()
      addItem(item)

      def addItem(item: Item) {
        itemList += item
      }

      override def price = {
        var p: Double = 0
        itemList.foreach(i => p = p + i.price) 
        p
      }

      override def description = {
        var des = ""
        itemList.foreach(i => des += i.description + " ")
        des
      }
    }
    ```
5. 

    ```scala
    class Point(val x: Double, val y: Double){}
    class LabeledPoint(val label: String, override val x: Double, override val y: Double) extends Point(x, y) {}
    ```

6. 

    ```scala
    abstract class Shape {
      abstract def centerPoint: Point
    }

    class Rectangle(p1: Point, p2: Point, p3: Point) extends Shape {
      override def centerPoint = {
        //略
      }
    }

    class Circle(p1: Point, p2: Point, p3: Point) extends Shape {
      override def centerPoint = {
        //略
      }
    }
    ```
7. 主构造器貌似不能定义，但是可以在class中直接写语句，以便被默认主构造器调用

    ```scala
    import java.awt.Point
    import java.awt.Rectangle

    class Square extends Rectangle {
      height = 0
      width = 0
      x = 0
      y = 0
      def this(p: Point, w: Int) {
        this()
        height = w
        width = w
        x = p.x
        y = p.y
      }

      def this(width: Int) {
        this(new Point(0, 0), width)
      }

    }
    ```

8. 可以看到两个类中都有name()方法，但是子类覆写了父类的。javap加-c可以看到详细的操作指令。 再加上-v可以看到常量池。

        $javap -p Person.class
        public class Person implements scala.ScalaObject {
          private final java.lang.String name;
          public java.lang.String name();
          public java.lang.String toString();
          public Person(java.lang.String);
        }

  这个name()读取的值是
  `#11 = Fieldref           #9.#10         // Person.name:Ljava/lang/String;`
  而这个#11的值是在构造函数中设置的。

        $javap -p SecretAgent.class
        public class SecretAgent extends Person implements scala.ScalaObject {
          private final java.lang.String name;
          private final java.lang.String toString;
          public java.lang.String name();
          public java.lang.String toString();
          public SecretAgent(java.lang.String);
        }

  SecretAgent和Person不一样的是name设置了默认值，用-v查看，name的secrect实际上是
  在构造函数中设置的

9. ref覆写ref，子类的env可以正确初始化，而用val覆写def，env会被初始化成0长度。  
  这个跟val覆写val的道理是一样的。父类和子类同时存在私有的同名变量`range`和相同
  的`range`的getter,但是父类构造函数先被调用，却在其中调用子类的getter（因为父类
  的getter以被子类覆写。子类的`range`因为此时还没初始化，所以返回了0。父类构造函
  数错误地使用0来初始化了`env`。

  这种行为本身就是个坑，但是也提供了非常大的灵活性。面向对象的Template设计模式就
  依赖这种行为实现的。所以还是多多善用为妙。

10. 前一个`protected`是指主构造器的权限， 即默认情况下，是不能已传入`elems`的方式创建Stack对象的，elems的`protected`指的是这个参数只有子类才能访问。

----
<div align="right">use Scala 2.9.1</div>
