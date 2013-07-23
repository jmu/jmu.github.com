---
layout: post
title: "Scala in Impatient 习题解答3"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}


1. 编写一段代码，将a设置为一个n个随机整数的数组，要求随机数介于0（含）和n（不含
）之间

    ```scala
    scala> val n = 10
    n: Int = 10

    scala> val a = (for (i <- 0 until n) yield scala.util.Random.nextInt(n)).toArray
    a: Array[Int] = Array(6, 7, 1, 7, 2, 0, 7, 9, 0, 6)
    ```

2. 交换相邻元素

    ```scala
    scala> val a = Array(1,2,3,4,5)
    a: Array[Int] = Array(1, 2, 3, 4, 5)

    for (i <- 0 until (a.length, 2)) {
        if (i + 1 < a.length) {
            var tmp = a(i)
            a(i) = a(i+1)
            a(i+1) = tmp
        }
    }

    scala> a
    res3: Array[Int] = Array(2, 1, 4, 3, 6, 5)
    ```

3. 用`yield`生成新数组

    ```scala
    scala> val a = Array(1,2,3,4,5)
    a: Array[Int] = Array(1, 2, 3, 4, 5)

    for (i <- 0 until a.length) yield {
        if (i % 2 == 0) {
            if (i + 1 < a.length) a(i+1) else a(i)
        } else {
            if (i - 1 >= 0) a(i-1) else a(i)
        }
    }

    res2: scala.collection.immutable.IndexedSeq[Int] = Vector(2, 1, 4, 3, 5)
    ```

4. 给定一个数组，产出新的数组，正值在前;0和负数在后，两者都保持原顺序不变。

    ```scala
    scala> val a = Array(0,1,-2,-3,4,5,-6)
    a: Array[Int] = Array(0, 1, -2, -3, 4, 5, -6)

    scala> a.filter(_ > 0) ++ a.filter(_ <= 0)
    res5: Array[Int] = Array(1, 4, 5, 0, -2, -3, -6)
    ```

  (好吧，有点投机去巧。)

5. 求平均值

    ```scala
    scala> val b = Array(0,1.44,-2.0,-3.1,4,5.99,-6)
    b: Array[Double] = Array(0.0, 1.44, -2.0, -3.1, 4.0, 5.99, -6.0)

    scala> b.sum / b.length
    res7: Double = 0.04714285714285715
    ```

6. `Array[Int]`和`ArrayBuffer[Int]`都可以用`reverse`方法反序排列。此题的用意是神马？难道Scala 2.8版本有什么特别吗？

7. 数组排重用`ArrayOps.distinct`

    ```scala
    scala> a
    res17: Array[Int] = Array(1, 2, 4, 3, 4, 5)

    scala> a.distinct
    res18: Array[Int] = Array(1, 2, 4, 3, 5)
    ```

8. 收集负值下标，反序，去掉最后一个。

    ```scala
    val indexes = for (i <- 0 until a.length if a(i) < 0) yield i
    for (i <- indexes.reverse.init) a.remove(i) 
    ```

  与3.4的第1和第2两个代码比,`remove`是开销最大的，而本历两次循环，又没有避免使用
  `remove`，连第1种都不如。效率由高到低是2>1>本例。

9. 

        scala> import java.util.TimeZone
        import java.util.TimeZone

        val a = TimeZone.getAvailableIDs()
        a.filter(_.startsWith("America/")).map(_.drop("America/".length))

10. 

        scala> import java.awt.datatransfer._
        import java.awt.datatransfer._

        scala> val flavors =
        SystemFlavorMap.getDefaultFlavorMap().asInstanceOf[SystemFlavorMap]
        flavors: java.awt.datatransfer.SystemFlavorMap =
        java.awt.datatransfer.SystemFlavorMap@1365301

        scala> import scala.collection.mutable.Buffer
        import scala.collection.mutable.Buffer

        scala> import scala.collection.JavaConversions.asScalaBuffer
        import scala.collection.JavaConversions.asScalaBuffer

        scala> val b: Buffer[java.lang.String] = flavors.getNativesForFlavor(DataFlavor.imageFlavor)
        b: scala.collection.mutable.Buffer[java.lang.String] = Buffer(image/jpeg,
        image/png, image/x-png, image/gif, PNG, JFIF)

 其中`import scala.collection.JavaConversions.asScalaBuffer`的声明要写，否则会报
 type mismatch 的错误。

----
<div align="right">use Scala 2.9.1</div>
