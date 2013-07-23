---
layout: post
title: "Scala in Impatient 习题解答2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 第一题

    ```scala
    def signum(i:Int) = {
        if (i > 0) 1 else if (i < 0) -1 else 0
    }
    ```

2. 空的块`{}`值写作`()`, 类型是`Unit`

        scala> def a = {}
        a: Unit

        scala> print(a)
        ()

        scala> val p = a
        p: Unit = ()

3. 与Java/C++不同，Scala的赋值语句的返回值是`Unit`，只要x和y是`Unit`类型，这么
x=y=1就是合法的。

4. 查阅 `RichInt`的 Scaladoc 可知，to还有一个带step的重载方法`def to(end: Int, step: Int): Inclusive ` 

    ```scala
    for (i <- 10.to(0, -1)) println(i)
    ```

5. countdown(n:Int)

    ```scala
    def countdown(n:Int) {
        for (i <- n.to(0,-1)) println(i)
    }
    ```

6. 算了半天也不对，后来发现默认的Int不够长。用Long就可以了。

        scala> var sum: Long = 1;for (i <- "Hello") sum *= i.toInt; print(sum)
        9415087488sum: Long = 9415087488

7. 找了半天找到了`StringOps.product`。貌似输出是16位，如何返回`Long`型还有待研究。
   `product[B >: Char](implicit num: Numeric[B]): B` 这个怎么用现在还不清楚。

8. 编写product(s: String)函数

    ```scala
    def product(s: String) = {
        var t: Long = 1;
        for (ch <- s) t *= ch
        t
    }
    ```

        scala> print(product("Hello"))
        9415087488

9. 将上一题写成递归

    ```scala
    def product2(s: String) :Long = {
        if (s.length <= 0) 0
        else if (s.length == 1) s.head.toLong
        else s.head * product2(s.tail)
    }
    ```

        scala> print(product2("Hello"))
        9415087488

10. 其中一个答案

    ```scala
    def pow(x:Int, n:Int):Double = {
        if (n == 0) 1
        else if (n > 0) {
            if (n % 2 == 0) {
                val y = pow(x,n/2)
                y * y
            }
            else x * pow(x,n-1)
        }
        else 1/pow(x,-n)
    }

    println(pow(readLine.toInt, readLine.toInt))
    ```

----
<div align='right'>use Scala 2.9.1</div>
