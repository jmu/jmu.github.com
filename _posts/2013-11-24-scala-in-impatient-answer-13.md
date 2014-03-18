---
layout: post
title: "Scala in Impatient 习题解答13 集合A2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}


1. 编写一个函数，给定字符串，产出一个包含所有字符的下标的映射。举例来说， indexs("Mississippi")应返回一个映射，让`M`对应集{0},`i`对应集{1,4,7,10},以此类推。使用字符到可变集的映射。另外，你如何保证集是经过排序的？

    ```scala
    import collection.mutable.Map
    import collection.immutable.SortedSet

    def indexes(str: String): Map[Char,Set[Int]] = {
      val map = Map[Char,Set[Int]]()
      for(i <- 0 until str.length) {
        map(str(i)) = map.getOrElse(str(i), SortedSet[Int]()) + i
      }
      map
    }
    ```
此时使用可变Map，并且使用`SortedSet`来保证排序。

2. 重复前一个练习，这次用字符到列表的不可变映射。

    ```scala
    import collection.immutable.SortedSet

    def indexes(str: String): Map[Char,Set[Int]] = {
      (Map[Char,Set[Int]]() /: (0 until str.length)) {
        (m, i) => m + (str(i) -> (m.getOrElse(str(i), SortedSet[Int]()) + i))
      }
    }
    ```

3. 编写一个函数，从一个整型链表中去除所有零值。

    ```scala
    import collection.mutable.LinkedList

    def removeZero(list: LinkedList[Int]): LinkedList[Int] = {
      list.collect {case i: Int if (i != 0) => i}
    }
    ```

4. 编写一个函数，接受一个字符串的集合，以及一个从字符串到整数值的映射。返回整型
的集合，其值为能和集合中某个字符串相对应的映射的值。举例来说，给定`Array("Tom",
"Fred", "Harry")`和`Map("Tom" -> 3, "Dick" -> 4, "Harry" -> 5)`，返回`Array(3,
5)`。提示：用flatMap将get返回的Option值组合在一起。

    ```scala
    def getNumber(arry: Array[String], m: Map[String, Int]) = {
      arry.flatMap(m.get(_))
    }
    ```
5. 

    ```scala
    def mkString(it: Iterable[Any]): String = {
      it.reduceLeft{ (a,b) => 
        val re:Any = a.toString + b.toString
        re
      }.toString
    }
    ```

6. 

    ```
    scala> (lst :\ List[Int]())(_ :: _)
    res39: List[Int] = List(1, 2, 3, 5)

    scala> (List[Int]() /: lst)( _:+ _)
    res40: List[Int] = List(1, 2, 3, 5)
    ```

   都是他们的本身，反序应该使用

    ```scala
    (lst :\ List[Int]())((a,b) => b :+ a)
    (List[Int]() /: lst)((a,b) => b :: a)
    ```

7. 

    ```scala
    (prices zip quantities).map(Function.tupled(_*_))
    ```

8. 

    ```scala
    def groupit(arry: Array[Double], column: Int) = {
      arry.grouped(column).toArray.map(_.toArray)
    }
    ```




未完待续...

----
<div align="right">use Scala 2.9.1</div>
