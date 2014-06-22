---
layout: post
title: "Scala in Impatient 习题解答18 高级类型L2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

    ```scala
    class Bug {
      var steps = 0

      def move(step: Int): this.type = { steps += step; this }
      def turn(): this.type = { steps = 0; this}
      def show(): this.type = { print(steps + " "); this}
    }
    ```

        scala> val bug= new Bug
        bug: Bug = Bug@196ed85

        scala> bug.move(4).show().move(6).show().turn().move(5).show()
        4 10 5 res45: Bug = Bug@196ed85

2. 

    ```scala
    object around
    object then
    object show

    class Bug {
      var steps = 0

      def move(step: Int): this.type = { steps += step; this }
      def and(obj: then.type): this.type = this
      def and(obj: show.type): this.type = { print(steps + " "); this}
      def turn(obj: around.type): this.type = { steps = 0; this}
    }
    ```

3. 

    ```scala
    class Document {
      var title: String = ""
      var author: String = ""
      def setTitle(title: String):this.type = {this.title = title; this}
      def setAuthor(author: String):this.type = {this.author= author; this}
    }
    object Title
    object Author

    class Book extends Document {
      private var useNextArgAs: Any = null
      def set(obj: Any): this.type = {useNextArgAs = obj; this}
      def to(arg: String): this.type = {
        useNextArgAs match {
          case Title => title = arg
          case Author => author = arg
          case _ =>
        }
        this
      }
    }

    object Main extends App {
      val book = new Book
      book set Title to "Scala for the Impatient" set Author to "Cay Horstmann"
      println(book.title)
      println(book.author)
    }
    ```

4. 

    ```scala
    import scala.collection.mutable.ArrayBuffer
    class Network {
      class Member(val name: String) {
        val contacts = new ArrayBuffer[Member]

        def canEqual(other: Any): Boolean = other.isInstanceOf[Member]

        override def equals(other: Any): Boolean = other match {
          case that: Member =>
            (that canEqual this) &&
              name == that.name
          case _ => false
        }
      }
      private val members = new ArrayBuffer[Member]

      def join(name: String) = {
        val m = new Member(name)
        members += m
        m
      }

    }
    ```

5. 

    ```scala
    object NetworkMain extends App {
      type NetworkMember = n.Member forSome { val n: Network }
      def process(m1: NetworkMember, m2: NetworkMember) = (m1, m2)

      val chatter = new Network
      val myFace = new Network

      val fred = chatter.join("Fred")
      val barney = myFace.join("Barney")

      val nm: NetworkMember = fred
      val nm2: NetworkMember = barney

      val result = process(nm, nm2)
      println(result)
    }
    ```

  运行结果如下, 正常运行。说明与18.8不同，是允许不同网络的。

        scala NetworkMain
        (Network$Member@8ce525,Network$Member@1810a1)
6. 

    ```scala
    def getIndex(arr: Seq[Int], v: Int): Int Either Int = {
      if (arr.contains(v)) {
        Left(arr.indexOf(v))
      } else {
        Right(arr.indexOf(arr.reduce((a,b) => if (math.abs(v - a) > math.abs(v - b)) b else a)))
      }
    }
    ```

  返回值的声明使用了中置类型，等同于`Either[Int, Int]`。另外，使用reduce来查找接近值不是最好的算法，它会reduce数组的所有值，而实际上一旦找到最接近值，由于数组是有序的，就没必要继续查找了。 

7. 

    ```scala
    def doSomething[E <: {def close(): Unit}](obj: E, func: E => Unit) {
      try {
        func(obj)
      } finally {
        obj.close()
      }
    }
    ```

  然后就可以这么用

        scala> class MyClose { def close() {println("closed")}; def say =
        println("run")}
        defined class MyClose

        scala> doSomething(new MyClose(), (f: MyClose) => f.say)
        run
        closed

8. 

    ```scala
    def printValues(f: {def apply(a: Int): Int}, from: Int, to: Int) {
      for(i <- from to to) {
        print(f.apply(i) + " ")
      }

    }
    ```

        scala> printValues((x:Int) => x *x , 3,6)
        9 16 25 36 
        scala> printValues(Array(1,1,2,3,5,8,13,21,34,55) , 3,6)
        3 5 8 13 

9. 

  为了避免Seconds和Meters相加，只需要增加`this: T`的自身类型就可以了。
  它用来限制调用其修饰范围内的3个方法this类型必须都是T，即一致。

    ```scala
    abstract class Dim[T](val value: Double, val name: String) {
      this: T =>
        protected def create(v: Double): T
        def +(other: Dim[T]) = create(value + other.value)
        override def toString() = value + " " + name
    }

    class Seconds(v: Double) extends Dim[Seconds](v, "s") {
      override def create(v: Double) = new Seconds(v)
    }

    // compile error
    class Meters(v: Double) extends Dim[Seconds](v, "m") {
      override def create(v: Double) = new Seconds(v)
    }
    ```

 原本可以通过编译的代码，在增加`this: T =>`之后报以下错误。

        <console>:10: error: illegal inheritance;
         self-type Meters does not conform to Dim[Seconds]'s selftype Dim[Seconds] with
         Seconds
                class Meters(v: Double) extends Dim[Seconds](v, "m") {
                                                ^

10. 

    ```scala

    trait A {
      def sing() = "from a"
    }

    trait C {
      this: A =>
      val w = sing + "from c"

    }

    class B {
      this: C =>
      val k = w
    }

    object a extends App {
      val b = new B with C with A
      println(b.k)

    }
    ```
 
  b.k的输出是`null`，并不是期望的值。主要是因为初始化k在初始化trait C之前就完成
  了。

----
<div align="right">use Scala 2.11.1</div>

