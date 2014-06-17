---
layout: post
title: "Scala in Impatient 习题解答17 类型参数L2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 定义一个不可变类`Pair[T, S]` ，带一个swap方法，返回组件交换过位置的新对偶。

    ```scala
    class Pair[T, S](val first: T, val second: S) {
      def swap() = new Pair(second, first)
      }
    ```

2. 定义一个可变类`Pair[T]`，带一个swap方法，交换对偶中组件的位置。

    ```scala
    class Pair[T](var first:T, var second: T) {
      def swap() = new Pair(second, first)
    }
    ```

3. 给定类`Pair[T, S]` ，编写一个泛型方法swap，接受对偶作为参数并返回组件交换过位置的新对偶。

    ```scala
    class Pair[T, S](val first: T, val second: S) {
      def swap(p: Pair[T, S]) = new Pair(p.second, p.first)
    }
    ```

4. 因为Student是Person的子类，是可以转成T类型的，不必定义下界
5. `T <% Comparable[T]` ,T是Int的时候将自动调用RichInt中的`Comparable[Int]`,所以是实现`Comparable[Int]`而不是`Comparable[RichInt]`

6. 

    ```scala
    def middle[T](it: Iterable[T]) = {
      it.toSeq match {
        case seq => seq((seq.size / 2).toInt)
      }
    }
    ```

7. 很多方法都使用了A， 比如min, max, last 等等。这些方法都会产出A类型，所以位于协变点 

8. 

    ```scala
    class Pair[+T](var first: T, var second: T) {
      def replaceFirst[R >: T](newFirst: R) {first = newFirst}
    }
    ```
 
  报以下错误

        <console>:8: error: type mismatch;
         found   : newFirst.type (with underlying type R)
         required: T
                 def replaceFirst[R >: T](newFirst: R) {first = newFirst}
                                                                ^
  只是因为R是父类，不能赋给子类。

9. 

    ```scala
    class Pair[+T](val first: T, val second: T) {
      def replaceFirst(newFirst: T){} //编译不过
    }

    class NastyDoublePair(a: Double, b: Double) extends Pair[Double](a, b){
      override def replaceFirst(newFirst: Double) = {
        new Pair(math.sqrt(newFirst), b)
      }
    }

    object a extends App {
      val p: Pair[Any] = new NastyDoublePair(1.0, 2.0)
      p.replaceFirst("Hello")
    }
    ```

 虽然假定以上代码可以编译通过，但是，如果方法被重写导致允许传入不恰当的类型。

10.  

    ```scala
    class Pair[S, T](var first:S, var second: T) {
      def swap(implicit ev: S =:= T) = new Pair(second, first)
    }
    ```

        scala> val a = new Pair(1,2)
        a: Pair[Int,Int] = Pair@9981eb

        scala> a.swap
        res33: Pair[Int,Int] = Pair@1f650e6

        scala> res33.first
        res34: Int = 2

        scala> val b = new Pair(1,"2")
        b: Pair[Int,String] = Pair@d3e2d4

        scala> b.swap
        <console>:10: error: Cannot prove that Int =:= String.
                      b.swap
                        ^

  可以看到，如果类型不同，调用了`类型约束`了的函数会报以上错误。

----
<div align="right">use Scala 2.11.1</div>
