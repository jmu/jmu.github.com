---
layout: post
title: "Scala in Impatient 习题解答1 基础A1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

《[Scala for the Impatient][1]》,书如其书名，是给想快速了解Scala语言的人写的。
当然，也不是没有门槛，至少是对Java/C++比较了解。书中假定你已掌握以上语言的一定基
础，并多处与Scala进行了对比。

此书作者还免费提供了A1级别的全部9章电子版内容， 在[TypeSafe网站][2]可以下到PDF版（英文）。
不得不说真慷慨！赞！

在看书之余，将习题也做了一遍,记录下来。

[1]: http://horstmann.com/scala/
[2]: http://typesafe.com/resources/book/scala-for-the-impatient

1. 在REPL模式下输入 `3.` 然后按回车 `ENTER`显示如下：

        scala> 3.
        !=             ##             %              &              *              
        +              -              /              <              <<             
        <=             ==             >              >=             >>             
        >>>            ^              asInstanceOf   equals         getClass       
        hashCode       isInstanceOf   toByte         toChar         toDouble       
        toFloat        toInt          toLong         toShort        toString       
        unary_+        unary_-        unary_~        |              

2. `res4`就是答案

        scala> math.sqrt(3)
        res2: Double = 1.7320508075688772

        scala> math.pow(res2, 2)
        res3: Double = 2.9999999999999996

        scala> 3 - res3
        res4: Double = 4.440892098500626E-16

3. `res` 在REPL中是`val`。像下面这样赋值会得到错误。 

        scala> res2
        res9: Double = 1.7320508075688772
        scala> res2 = 1
        <console>:8: error: reassignment to val
               res2 = 1
                    ^

4. Scala中字符串的包装类是`StringOps`,乘号`*`是它的运算符重载的方法

        scala> "crazy" * 3
        res10: String = crazycrazycrazy

   Scaladoc中查找`StringOps`结果见图1-1右侧


   ![在Scaladoc搜StringOps]({{site.url}}/assets/files/stringops-scala-1.png)
                               <div align="center">图1-1</div>   
5. 10 max 2 相当于调用10.max(2)，max方法存在于`RichInt`中。10的类型是`Int`，调
   用前首先被转换成`RichInt`。

        scala> 10 max 2
        res86: Int = 10

        scala> 10.max(2)
        res87: Int = 10

6. BigInt如下计算即可.

        scala> BigInt(2).pow(1024)
        res19: scala.math.BigInt = 17976931348623159077293051907890247336179
        76978942306572734300811577326758055009631327084773224075360211201138
        79871393357658789768814416622492847430639474124377767893424865485276
        30221960124609411945308295208500576883815068234246288147391311054082
        7237163350510684586298239947245938479716304835356329624224137216

7. `import`引入包

        scala> import scala.math.BigInt
        import scala.math.BigInt

        scala> import scala.util.Random
        import scala.util.Random

        scala> BigInt.probablePrime(100, Random)
        res55: scala.math.BigInt = 1266874521284717122498840798537

8. 只需要调用`BigInt.toString`即可(为了避免出现负数，使用了`abs`)

        scala> BigInt(util.Random.nextLong).abs.toString(36)
        res63: String = 1qaspwgj5qqlf

9. 查询Scaladoc的StringOps可知，分别是`head`和`last`方法。
10. `take` 和 `drop`分别是从左边取得和去掉几个字符。`takeRight`和`dropRight`差不
  多的功
  能，只是从右边开始数。`substring`可以指定从左起的下标，用来截取字符串的中间
  部分很有用。他们用来截取前后字符串很方便，但是截取中间字符就不太合适，即使可以
  联合使用。

        scala> "jmu is using Scala".take(5).drop(1)
        res82: String = mu i
----
  <div align="right">use Scala 2.9.1</div>
