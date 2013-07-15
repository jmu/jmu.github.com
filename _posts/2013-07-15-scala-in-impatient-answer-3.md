---
layout: post
title: "Scala in Impatient 习题解答3"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}


1\. 编写一段代码，将a设置为一个n个随机整数的数组，要求随机数介于0（含）和n（不含
）之间

    scala> val n = 10
    n: Int = 10

    scala> val a = (for (i <- 0 until n) yield scala.util.Random.nextInt(n)).toArray
    a: Array[Int] = Array(6, 7, 1, 7, 2, 0, 7, 9, 0, 6)

2\. 交换相邻元素

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

3\. 用`yield`生成新数组

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

4\. to be continued. 


5\.
6\.
7\.
8\.
9\.
10\.


----
  <div align="right">use Scala 2.9.1</div>
