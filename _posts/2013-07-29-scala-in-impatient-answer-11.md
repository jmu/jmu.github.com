---
layout: post
title: "Scala in Impatient 习题解答11 操作符"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. Scala中+和->是平级的，中置操作符高于后置，也可认为是左结合。所以`3 + 4 -> 5`
相当于`(3 + 4) -> 5`，`3 -> 4 + 5`相当于`(3 -> 4) + 5`

2. 因为Scala中\*和^都是合法的标识符，使用这些定义过多的操作符，会引起混淆。
  而且`^`已作他用


3. 

    ```scala
    class Fraction(n: Int, d: Int) {
      private val num: Int = if (d == 0) 1 else n * sign(d) / gcd(n, d)
      private val den: Int = if (d == 0) 0 else d * sign(d) / gcd(n, d)

      override def toString = num + "/" + den
      def sign(a: Int) = if (a > 0) 1 else if (a < 0) -1 else 0
      def gcd(a: Int, b:Int): Int = if (b == 0) math.abs(a) else gcd(b, a % b)
      def +(f: Fraction) = new Fraction(num * f.den + f.num * den, den * f.den)
      def -(f: Fraction) = new Fraction(num * f.den - f.num * den, den * f.den)
      def *(f: Fraction) = new Fraction(num * f.num, den * f.den)
      def /(f: Fraction) = this.*(new Fraction(f.den, f.num))
    }
    ```
  执行结果

        scala> new Fraction(1,2) + new Fraction(1, 5)
        res31: Fraction = 7/10

        scala> new Fraction(1,2) - new Fraction(1, 5)
        res32: Fraction = 3/10

        scala> new Fraction(1,2) * new Fraction(1, 5)
        res33: Fraction = 1/10

        scala> new Fraction(1,2) / new Fraction(1, 5)
        res34: Fraction = 5/2

4. 技术上提供`*`和`/`应该是没问题的，问题是逻辑上说不通。

    ```scala
    object Money {
      def apply(buck: Int, cent: Int) = new Money(buck, cent)
    }
    class Money(buck: Int, cent: Int) {
      private val d = buck * 100 + cent

      def +(m: Money) = {
        val total = d + m.d
        new Money(total / 100, total % 100)
      }

      def -(m: Money) = {
        val total = d - m.d
        new Money(total / 100, total % 100)
      }

      def ==(m: Money): Boolean = { d == m.d }
      def <(m: Money): Boolean = { d < m.d }
    }
    ```

5. 

    ```scala
    object Table {
      def apply() = new Table("<table></table>")
    }

    class Table(val html: String) {
      def |(td: String): Table = {
        operate("</td>", "<td>" + td + "</td>")
      }

      def ||(tr: String): Table = {
        operate("</tr>", "<tr><td>" + tr + "</td></tr>")
      }

      private def operate(after: String, str: String): Table = {
        val lastFind = html.lastIndexOf(after)

        if (lastFind == -1) {
          val index = html.indexOf("</table>")
          new Table(insert(index, str))
        } else {
          new Table(insert(lastFind + after.size, str))
        }
      }

      private def insert(index: Int, str: String) = {
        html.take(index) + str + html.takeRight(html.length - index)
      }

      override def toString = html
    }
    ```
6. 


----
<div align="right">use Scala 2.9.1</div>
