---
layout: post
title: "Scala in Impatient 习题解答21  隐式转换和隐式参数L3"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 实际上2.11版本已经不使用`Predef.any2ArrowAssoc`了。取而代之的是`ArrowAssoc`。它将任意对象上`->`的调用隐式转换成`ArrowAssoc`上的`->`方法调用，进而转换成Tuple2 的类型。

    ```scala
    implicit final class ArrowAssoc[A](private val self: A) extends AnyVal {
        @inline def -> [B](y: B): Tuple2[A, B] = Tuple2(self, y)
        def →[B](y: B): Tuple2[A, B] = ->(y)
    }
    ```

2. 

    ```scala
    class PercentPlus(val number: Int) {
         def +%(p: Int): Double = number * (1 + p/100D) 
    }

    implicit def int2PercentPlus(number: Int) = new PercentPlus(number)
    ```

          scala> 120 +% 10
          res0: Double = 132.0

  在scala2.11版本中可以直接声明隐式类的简化方式来达到同样效果

    ```scala
    implicit class PercentPlus(val number: Int) {
         def +%(p: Int): Double = number * (1 + p/100D) 
    }
    ```

3. 虽然可以如题中提示那样用一个常规类和一个隐式转换声明。使用隐式类更简洁一些，如下所示

    ```scala
    implicit class FactorialAssoc(private val self: Int) {
      def ! = (1 to self).product
    }

    5!
    ```
4. 

----
<div align="right">use Scala 2.11.1</div>
