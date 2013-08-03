---
layout: post
title: "Scala in Impatient 习题解答12 高阶函数L1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

看到现在，直到这一章才讲到函数式的强项啊

1. 

    ```scala
    def values(fun: (Int) => Int, low: Int, high: Int) = {
      low.to(high).map(x => x -> fun(x))
    }
    ```

2. 如何用`reduceLeft`得到数组中的最大元素。
只要知道`reduceLeft`是以参数1和参数2作为入参，结果最为下一次的参数1的特性，就好办了。

        Scala> (1 to 10).reduceLeft(math.max(_,_))
        res25: Int = 10

3. 用`to`和`reduceLeft`实现递归。此题与题2类似

    ```scala
    def factorial(n: Long) = {
      (1L to n).reduceLeft(_ * _)
    }
    ```
4. 

    ```scala
    def factorial(n: Long) = {
      (1L to n).foldLeft(1L)(_ * _)
    }
    ```

5. 

    ```scala
    def largest(fun: (Int) => Int, inputs:Seq[Int]) = inputs.map(fun(_)).max
    ```

6. 

    ```scala
    def largest(fun: (Int) => Int, inputs:Seq[Int]) = {
        inputs.reduceLeft((x, y) => if (fun(x) > fun(y)) x else y)
    }
    ```

7. 

    ```scala
    def adjustToPair(f: (Int,Int) => Int) = {
       x: Product2[Int, Int] => f(x._1,x._2)
    }
    ```
        scala> val pairs = (1 to 10) zip (11 to 20)
        pairs: scala.collection.immutable.IndexedSeq[(Int, Int)] = Vector((1,11),
        (2,12), (3,13), (4,14), (5,15), (6,16), (7,17), (8,18), (9,19), (10,20))

        //Tuple不接受两个两个参数运算
        scala> pairs.map(_ + _)
        <console>:9: error: wrong number of parameters; expected = 1
                      pairs.map(_ + _)
        //通过adjustToPair转换成功
        scala> pairs.map(adjustToPair(_ + _))
        res22: scala.collection.immutable.IndexedSeq[Int] = Vector(12, 14, 16, 18, 20,
        22, 24, 26, 28, 30)

8. 

        scala> val a = Array(1,2,3)
        a: Array[Int] = Array(1, 2, 3)

        scala> val b = Array("j","jm","jmu")
        b: Array[java.lang.String] = Array(j, jm, jmu)

        scala> b.corresponds(a)(_.length == _)
        res23: Boolean = true

9. 

    ```scala
    import scala.collection.mutable.ArraySeq

    trait MyArray[A] extends ArraySeq[A]{
      def corresponds[B] (that: Seq[B], p: (A, B) => Boolean): Boolean = {
        super.zip(that).map(x => if (p(x._1, x._2) != true) return false)
        true
      }
    }
    ```

        scala> val b1 = new ArraySeq[String](3) with MyArray[String]
        b1: scala.collection.mutable.ArraySeq[String] with MyArray[String] = (null,
        null, null)

        scala> b1(0) = "j"

        scala> b1(1) = "jm"

        scala> b1(2) = "jmu"

        scala> b1.corresponds(a,_.length == _)
        <console>:18: error: missing parameter type for expanded function ((x$1: String,
        x$2) => x$1.length.$eq$eq(x$2))
                      b1.corresponds(a,_.length == _)

        scala> b1.corresponds(a,_.length == (_:Int))
        res49: Boolean = true

  使用非柯里化时，遇到如上所示类型无法判断出的错误。与柯里化相比，此时`that`和
  `p`失去了联系。程序无法推断经过运算后的结果的类型。

10. 第一个参数需要换名调用，柯里化后更美观。只是block返回值如何处理。

    ```scala
    def unless(condition: => Boolean)(block: => Unit) {
      if (!condition) block
    }
    ```

----
<div align="right">use Scala 2.9.1</div>
