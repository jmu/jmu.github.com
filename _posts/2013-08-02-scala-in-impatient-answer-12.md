---
layout: post
title: "Scala in Impatient 习题解答12 高阶函数L1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

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

----
<div align="right">use Scala 2.9.1</div>
