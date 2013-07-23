---
layout: post
title: "Scala in Impatient 习题解答6"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

本章只有8题

1. 

    ```scala
    object Conversions {
        private val i2c = 30.48
        private val g2l = 3.785411784
        private val m2k = 1.609344

        def inchesToCentimeters(inch: Double): Double = {
            inch * i2c 
        }

        def gallonsToLiters(gallon: Double): Double = {
            gallon * g2l
        }

        def milesToKilometers(mile: Double): Double = {
            mile * m2k
        }
    }
    ```

        scala> Conversions.inchesToCentimeters(1)
        res23: Double = 30.48

2. 

    ```scala
    abstract class UnitConversion {
        def converse(from: Double): Double
    }
    object InchesToCentimeters extends UnitConversion {
        private val i2c = 30.48

        override def converse(inch: Double): Double = {
            inch * i2c 
        }
    }

    object GallonsToLiters extends UnitConversion {
        private val g2l = 3.785411784

        override def converse(gallon: Double): Double = {
            gallon * g2l
        }
    }

    object MilesToKilometers extends UnitConversion {
        private val m2k = 1.609344

        override def converse(mile: Double): Double = {
            mile * m2k
        }
    }
    ```

3. 这肯定不是一个好主意。`Point`有move方法，setLocation方法。这些作为Origin(原
    点) 来说都不是很合适。

    ```scala
    import java.awt.Point

    object Origin extends Point {
    }
    ```


4. 

    ```scala
    class Point(var x: Int, var y: Int) {}
    object Point {
        def apply(x: Int, y: Int) = {
            new Point(x, y)
        }
    }
    ```

5. 

    ```scala
    object Reverse extends App {
        for (str <- args.reverse) print(str + " ")
    }
    ```

6. 这四个花色的符号win下选输入法的特殊符号软键盘。在lin下可以用vim。在vim中输入`:dig`可以找到分别对应cS,cH,cD,cC。进入vim插入模式按`<CTRL-K>`，然后分别输入即可（注意大小写）

    ```scala
    object PokerFace extends Enumeration {
        type PokerFace = Value
        val SPADES = Value("♠")
        val HEARTS = Value("♡")
        val DIAMONDS = Value("♢")
        val CLUBS = Value("♣")
    }
    ```

  验证

        scala> PokerFace.SPADES
        res59: PokerFace.Value = ♠

        scala> PokerFace.HEARTS
        res60: PokerFace.Value = ♡

        scala> PokerFace.DIAMONDS
        res61: PokerFace.Value = ♢

        scala> PokerFace.CLUBS
        res62: PokerFace.Value = ♣

7. 直接在6的对象基础上加方法

    ```scala
    object PokerFace extends Enumeration {
        type PokerFace = Value
        val SPADES = Value("♠")
        val HEARTS = Value("♡")
        val DIAMONDS = Value("♢")
        val CLUBS = Value("♣")

        def isRed(poker: PokerFace): Boolean = {
            poker.id == HEARTS.id || poker.id == DIAMONDS.id
        }
    }
    ```

8. RGB如果分别用8位表示，红是`0xff0000`,绿是`0x00ff00`,蓝是`0x0000ff`。以此类推
  ，8个顶点分别是(0,0,0)(0,0,1)(0,1,0)(0,1,1)(1,0,0)(1,0,1)(1,1,0)(1,1,1)

  RGB的立方体模型参考维基百科<http://en.wikipedia.org/wiki/RGB_color_model>

    ```scala
    object RGBCube extends Enumeration {
        type RGBCube = Value
        private val base = 0x0000ff
        private val vertexes = new Array[Int](8)
        for (i <- 0 until vertexes.length; j <- 0 to 2) {
            if ((i & 1 << j) != 0) vertexes(i) += base << j * 8
        }

        val V1 = Value(vertexes(0))
        val V2 = Value(vertexes(1))
        val V3 = Value(vertexes(2))
        val V4 = Value(vertexes(3))
        val V5 = Value(vertexes(4))
        val V6 = Value(vertexes(5))
        val V7 = Value(vertexes(6))
        val V8 = Value(vertexes(7))
    }
    ```

  验证

        scala> RGBCube.V4.id.toHexString
        res24: String = ffff

----
<div align="right">use Scala 2.9.1</div>
