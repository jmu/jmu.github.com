---
layout: post
title: "Scala in Impatient 习题解答11 操作符L1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. Scala中+和->是平级的，中置操作符高于后置，也可认为是左结合。所以`3 + 4 -> 5`
相当于`(3 + 4) -> 5`，`3 -> 4 + 5`相当于`(3 -> 4) + 5`

2. 因为Scala中\*和\^都是合法的标识符，使用这些定义过多的操作符，会引起混淆。
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
        if (html.indexOf("</tr>") == -1)
          operate("<table>", "<tr><td>" + td + "</td></tr>")
        else
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

        scala> Table() | "jmu" | "is"|| "a" || "scala" | "guy" |"now."
        res6: Table =
        <table><tr><td>jmu</td><td>is</td></tr><tr><td>a</td></tr><tr><td>scala</td><td>guy</td><td>now.</td></tr></table>

6. 

    ```scala
    class ASCIIArt(val paint: Array[String]) {
      def |(other: ASCIIArt): ASCIIArt = {
        val shortLen = math.min(paint.size, other.paint.size)
        val newPaint = new Array[String](shortLen)
        for (i <- 0 until shortLen)
          newPaint(i) = paint(i) + other.paint(i)

        new ASCIIArt(newPaint ++ (if (shortLen > paint.size)
          other.paint.drop(shortLen) 
          else paint.drop(shortLen)))

      }
      def --(other: ASCIIArt): ASCIIArt = {
        new ASCIIArt(paint ++ other.paint)
      }

      override def toString = paint.mkString("\n")
    }

    object Run extends App {
      val a = new ASCIIArt(Array(" /\\_/\\ ","( ' ' )", "(  -  )", " | | | ", "(__|__)"))
      val b = new ASCIIArt(Array("   ----- ", " / Hello \\", "<  Scala |", " \\ Coder / ", "   _____ "))
      println(a | b)
      println(a -- b)
    }
    ```
 打印效果如下

        $ scala Run
         /\_/\    ----- 
        ( ' ' ) / Hello \
        (  -  )<  Scala |
         | | |  \ Coder / 
        (__|__)   _____ 
         /\_/\ 
        ( ' ' )
        (  -  )
         | | | 
        (__|__)
           ----- 
         / Hello \
        <  Scala |
         \ Coder / 
           _____ 

7. 

    ```scala
    class BigSequence(index: Int, bitValue: Boolean) {
      private var v = 0L
      setValue(index, bitValue)

      private def setValue(index: Int, bitValue: Boolean) {
        val value = 1L << index % 64
        v = if (bitValue) v | value else v & ~(value)
      }

      def update(index: Int, bitValue: Boolean) {
        setValue(index, bitValue)
      }

      override def toString = v.toString
    }

    object BigSequence {
      def apply(index: Int, bitValue: Boolean)  = {
        new BigSequence(index, bitValue) 
      }
    }
    ```

        scala> val a = BigSequence(3,true)
        a: BigSequence = 8

        scala> a(1) = true
        scala> a
        res145: BigSequence = 10

8. 

    ```scala
    class Matrix(row: Int, col: Int) {
      val r = row
      val c = col

      def +(other: Matrix) = new Matrix(r + other.r, c + other.c)
      def *(other: Matrix) = new Matrix(r * other.r, c * other.c)
      def *(t: Int) = new Matrix(r * t, c * t)

      def unapply(mat: Matrix) = Some((mat.r, mat.c))
    }
    ```

        scala> val mat = new Matrix(3,5)
        mat: Matrix = Matrix@6e1c9a

        scala> val mat2 = new Matrix(7,10)
        mat2: Matrix = Matrix@75d407

        scala> val mat(row,col) = mat2
        row: Int = 7
        col: Int = 10

9. 只是演示，没有做有效性check

    ```scala
    class RichFile {
      def unapply(file: String) = { 
        val lastSlash = file.lastIndexOf("/")
        val lastDot = file.lastIndexOf(".")
        Some(file.take(lastSlash),
          file.substring(lastSlash + 1, lastDot),
          file.substring(lastDot + 1))
      }
    }
    ```

        scala> val r = new RichFile
        r: RichFile = RichFile@1a7762e

        scala> val r(path, filename ,ext) = "/home/jmu/tmp.txt"
        path: String = /home/jmu
        filename: java.lang.String = tmp
        ext: java.lang.String = txt

10. 

    ```scala
    class RichFile {
      def unapplySeq(file: String): Option[Seq[String]] = {
        Some(file.split("/"))
      }
    }
    ```

----
<div align="right">use Scala 2.9.1</div>
