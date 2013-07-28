---
layout: post
title: "Scala in Impatient 习题解答10 特质L1"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

    ```scala
    import java.awt.geom.Ellipse2D

    trait RectangleLike extends Ellipse2D.Double {
      def translate(dx: Int, dy: Int) {
        x = dx
        y = dy
      }

      def grow(h: Int, v: Int) {
        x = x - h
        y = y - v
        width = width + 2 * h
        height = height + 2 * v
      }
    }

    object Run extends App {
      val egg = new java.awt.geom.Ellipse2D.Double(5, 10, 20, 30) with RectangleLike
      egg.translate(10, -10)
      egg.grow(10, 20)
      println("x = " + egg.x + ",y = " + egg.y + ",w = " + egg.width + ",h = " +
      egg.height)
    }
    ```
2. 

    ```scala
    import java.awt.Point

    class OrderedPoint extends Point with math.Ordered[Point] {
      def compare(that: Point): Int = {
        if (x < that.x) -1
        else if (x == that.x && y < that.y) -1
        else if (x == that.x && y == that.y) 0
        else 1
      }
    }
    ```

3. 不知道题中所指的是immutable下的还是mutable下BitSet。姑且先就使用
  `scala.collection.mutable.BitSet`吧。

        class BitSet extends Set[Int] with BitSet with BitSetLike[BitSet] with
        SetLike[Int, BitSet] with Serializable
        lin(BitSet) = BitSet >> Serializable >> SetLike >> BitSetLike >> 
        collection.BitSet >> Set

4. 此题英文原版进行了更正<http://horstmann.com/scala/bugs.html>。应该是实现一个
trait。

        Page 127 \[2\]
            Change “Provide a CryptoLogger class” to “Provide a CryptoLogger trait”

   解答如下

    ```scala
    import scala.util.logging.Logged

    trait ConsoleLogger extends Logged {
      override def log(msg: String) {
        super.log(msg)
        println(msg)
      }
    }

    trait CryptoLogger extends Logged {
      val shift: Int = 3

      override def log(msg: String) {
        val cryptMsg = msg.map(c => if (c.isLetter) {
            val newc = ('a' +(shift % 26 + c.toLower-'a' + 26) % 26).toChar
            if (c.isLower) newc else newc.toUpper
          } else c).toString

        super.log(cryptMsg)
      }
    }

    object Run extends App {
      class Test extends Logged {
        def run = log("Hello, Scala world!\nabcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")
      }

      val t = new Test with ConsoleLogger with CryptoLogger
      t.run

      val t2 = new {
        override val shift = -3
      } with Test with ConsoleLogger with CryptoLogger

      t2.run
    }
    ```

  执行结果：

        $ scala Run
        Khoor, Vfdod zruog!
        defghijklmnopqrstuvwxyzabcDEFGHIJKLMNOPQRSTUVWXYZABC
        Ebiil, Pzxix tloia!
        xyzabcdefghijklmnopqrstuvwXYZABCDEFGHIJKLMNOPQRSTUVW

5. Java中也很少使用继承`PropertyChangeSupport`的方法，而是使用“合成”。
  如下写法，虽然免去了合成，但是无法检测到属性改动。对于每个bean属性改动的通知，
  还是要将`fireProperyChange`嵌入get/set方法中。

    ```scala
    import java.beans.{PropertyChangeSupport => JPropertyChangeSupport, _}

    trait PropertyChangeSupport {
      val pcs = new JPropertyChangeSupport(this)

      def addPropertyChangeListener(listener: PropertyChangeListener) {
        pcs.addPropertyChangeListener(listener)
      }
      def addPropertyChangeListener(propertyName: String, listener: PropertyChangeListener) {
        pcs.addPropertyChangeListener(propertyName, listener)
      }
      def firePropertyChange(propertyName: String, oldValue: Object, newValue: Object) {
        pcs.firePropertyChange(propertyName, oldValue, newValue)
      }
      //another methods...
    }

    object Run extends App {
      val p = new java.awt.Point with PropertyChangeSupport
      p.addPropertyChangeListener("x", new PropertyChangeListener() {
          override def propertyChange(evt: PropertyChangeEvent) {
            println(evt.getPropertyName)
          }
        })
    }
    ```
6. `JComponent`继承自`Container`而不是`Component`的原因不在于它本身，而是在于
 `JContainer`。`JContainer`需要`JComponent`和`Container`两个class的特性，在Java不允许
 多继承的情况下，只好通过`JComponent`继承自`Container`的方式折中了。

 Scala中，可以利用trait的特性，将多继承的可以理解为特质的class声明成trait。
 想到这里，倒是觉得trait和interface的概念是差不多的。用Java的眼光看的话，
 trait是一个interface + 伴生class的总称。

    ```scala
    class Component {}
    trait Container extends Component {}
    class JComponent extends Component {}
    class JContainer extends JComponent with Container{}
    ```

7. 

    ```scala
    trait OS {
      val version: Int

      def boot 
      def playGame: Boolean = {false}
      def shutdown: Boolean = {false}
    }

    trait iOS extends OS {

      override def boot = println("start up slowly")
      override def playGame: Boolean = {
        if (super.playGame == true) {
          println("Phone not locking, Have Fun!")
          true
        } else {
          println("Mom not allowed");
          false
        }
      }
      override def shutdown: Boolean = {
        println("clicked menu... ")
        if (super.shutdown == false) {
          val goOff = scala.util.Random.nextBoolean
          if (goOff) println("iOS "+ version +" is power off")
            goOff
        } else true
      }
    }

    trait JailBreak extends OS {
      val version = 7

      override def boot = println("start up fast")
      override def playGame: Boolean = {
        if (super.playGame == true) {
          println("Phone not locking, Have Fun!")
          true
        } else {
          println("I jailBreaked it.");
          true 
        }
      }
      override def shutdown: Boolean = {
        if (super.shutdown == false)
          println("iOS "+ version +" is power off")
        true 
      }
    }

    abstract class Phone {
      val offTime: Int
      def powerOn = println("pressing ON button 2sec")
      def powerOff = println("pressing ON button more than " + offTime + "sec")
    }

    abstract class iPhone extends Phone with OS {
      override val offTime = 15

      def boot = "iPhone is boot"
      override def powerOn = {
        super.powerOn
        boot
      }
      override def powerOff = {
        if (shutdown != true) {
          println("power not off, try again")
          if (shutdown != true) super.powerOff
        }
      }
    }

    object Run extends App {
      val newOne = new {override val version = 6 } with iPhone with iOS with JailBreak
      newOne.powerOn 
      newOne.playGame
      newOne.powerOff
      val anotherOne = new iPhone with JailBreak with iOS 
      anotherOne.powerOn 
      anotherOne.playGame
      anotherOne.powerOff
    }
    ```
8. 

    ```scala
    import scala.util.logging.Logged

    trait ConsoleLogger extends Logged {
      override def log(msg: String) {
        super.log(msg)
        println(msg)
      }
    }

    trait ScalaBufferedInputStream extends InputStream with ConsoleLogger {
      val ssize = 8
      var sbuf: Array[Byte] = Array[Byte]()

      override def read: Int = {
        if (sbuf.size == 0) {
          sbuf = new Array[Byte](ssize)
          super.read(sbuf) <= 0
          log("Read Buffers" + sbuf.mkString("[",",","]"))
        } 
        val result = if (sbuf.size > 0) sbuf.head & 0xff else -1
        sbuf = sbuf.drop(1)
        result
      }
    }


    object Run extends App {
      val a = new ByteArrayInputStream("Hello, Scala world!".getBytes) with ScalaBufferedInputStream

      collection.immutable.Stream.continually(a.read).
      takeWhile(_ > 0).foreach(by => print(by + " "))
    }
    ```

9. 同8

10. 

    ```scala
    import java.io.InputStream
    import scala.collection.Iterator

    abstract class IterableInputStream extends InputStream with Iterable[Byte] {

      override def iterator = new  Iterator[Byte] {
        var bufByte: Byte = -1
        var hasValue: Boolean = true

        def hasNext: Boolean = {
          if (hasValue != true) false
          else {
            bufByte = read().toByte
            hasValue = bufByte > 0
            hasValue
          }
        }

        def next: Byte = if (bufByte > 0) bufByte else Iterator.empty.next
      }
    }

    class MyInputStream(str: String) extends IterableInputStream {
      private var buf = str.getBytes

      override def read: Int = {
        if (buf.size > 0) {
          val re = buf.head & 0xff 
          buf = buf.drop(1)
          re
        } else -1
      }
    } 

    object Run extends App {
      new MyInputStream("Hello, Scala world!").iterator.foreach(println)
    }
    ```

----
<div align="right">use Scala 2.9.1</div>
