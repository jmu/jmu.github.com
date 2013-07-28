---
layout: post
title: "Scala in Impatient 习题解答4 映射和元组A1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

本章中文版有一小错误。
4.5小节 包名`collection`是没有s的，参考了下英文原版，是正确的。
翻译后反而多了个s，比较奇怪。

    val scores = scala.collections.immutable.SortedMap("Alice" ->10,
                                 ^ 正确的应该是没有s

1. 

        scala> val items = Map("iphone" -> 4999, "mx2" -> 2499, "pad" -> 3800)
        items: scala.collection.immutable.Map[java.lang.String,Int] = Map(iphone ->
        4999, mx2 -> 2499, pad -> 3800)

        scala> val items2 = for ((k,v) <- items) yield (k, v * 0.9)
        items2: scala.collection.immutable.Map[java.lang.String,Double] = Map(iphone ->
        4499.1, mx2 -> 2249.1, pad -> 3420.0)

2. 

    ```scala
    import scala.collection.mutable.HashMap

    val in = new java.util.Scanner(new java.io.File("README.md"))
    val result = new HashMap[String,Long]
    while (in.hasNext()) {
        val str = in.next()
        if (result.contains(str)) result(str) += 1 
        else result += (str -> 1)
    }
    in.close()

    print(result)
    ```

3. 重复前一个练习，这次用不可变的映射

    ```scala
    val in = new java.util.Scanner(new java.io.File("README.md"))
    var result = Map[String,Long]()
    while (in.hasNext()) {
        val str = in.next()
        if (result.contains(str)) {
            val c_v = result(str)
            result = result - str
            result = result + (str -> (c_v + 1))
        } else
            result = result + (str -> 1)
    }
    in.close()

    print(result)
    ```

4. 

    ```scala
    import scala.collection.immutable.SortedMap

    val in = new java.util.Scanner(new java.io.File("README.md"))
    var result = SortedMap[String,Long]()
    while (in.hasNext()) {
        val str = in.next()
        if (result.contains(str)) {
            val c_v = result(str)
            result = result - str
            result = result + (str -> (c_v + 1))
        } else
            result = result + (str -> 1)
    }
    in.close()

    print(result)
    ```

5. use TreeMap in scala

    ```scala
    import scala.collection.JavaConversions.mapAsScalaMap

    val in = new java.util.Scanner(new java.io.File("README.md"))
    val result: scala.collection.mutable.Map[String, Long] = 
        new java.util.TreeMap[String,Long]
    while (in.hasNext()) {
        val str = in.next()
        if (result.contains(str)) result(str) += 1 
        else result += (str -> 1)
    }
    in.close()

    print(result)
    ```

6. 

        scala> val months = scala.collection.mutable.LinkedHashMap("Monday" ->
        java.util.Calendar.MONDAY)
        months: scala.collection.mutable.LinkedHashMap[java.lang.String,Int] =
        Map(Monday -> 2)

        scala> months += "Sunday" -> java.util.Calendar.SUNDAY
        res5: months.type = Map(Monday -> 2, Sunday -> 1)

        scala> months += "Friday" -> java.util.Calendar.FRIDAY
        res6: months.type = Map(Monday -> 2, Sunday -> 1, Friday -> 6)

7. 

        scala> import scala.collection.JavaConversions.propertiesAsScalaMap
        import scala.collection.JavaConversions.propertiesAsScalaMap

        scala> val props: scala.collection.Map[String, String] = System.getProperties()

        scala> var maxLen = 0; for (i <- props.keys) if (i.length > maxLen) maxLen =
        i.length
        maxLen: Int = 29

        scala> for ((k,v) <- props) println(k + " " * (maxLen+2-k.length) + "|" + v)
        env.emacs                      |
        java.runtime.name              |Java(TM) SE Runtime Environment
        sun.boot.library.path          |/usr/lib/jvm/java-7-sun/jre/lib/i386
        java.vm.version                |23.21-b01
        java.vm.vendor                 |Oracle Corporation
        java.vendor.url                |http://java.oracle.com/
        path.separator                 |:
        java.vm.name                   |Java HotSpot(TM) Server VM
        file.encoding.pkg              |sun.io
        user.country                   |US
        sun.java.launcher              |SUN_STANDARD

8. 

        def minmax(values: Array[Int]) = {
            (values.min, values.max)
        }

        scala> val a = minmax(Array(1,2,3,4,-5))
        a: (Int, Int) = (-5,4)

9. 

    ```scala
    def lteqgt(values: Array[Int], v: Int) = {
        val lt = values.filter(_ < v).length
        val eq = values.filter(_ == v).length
        (lt, eq, (values.length - lt - eq))
    }
    ```

        scala> val (lt, eq, gt) = lteqgt(Array(1,2,3,4,5,6), 4)
        lt: Int = 3
        eq: Int = 1
        gt: Int = 2

10. 执行`"Hello".zip("World")`的结果是，生成了字母相对的tuple。一个现实中可能的
用途？捆绑一起然后随机排列，以便能保持相同顺序？

        scala> "Hello".zip("World")
        res18: scala.collection.immutable.IndexedSeq[(Char, Char)] = Vector((H,W),
        (e,o), (l,r), (l,l), (o,d))


----
<div align="right">use Scala 2.9.1</div>
