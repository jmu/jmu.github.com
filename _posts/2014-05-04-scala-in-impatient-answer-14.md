---
layout: post
title: "Scala in Impatient 习题解答14 模式匹配和样例类A2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}


1. JDK发型包有一个src.zip文件包含了JDK的大多数源代码。解压并搜索样例标签（使用正
则表达式`case[^:]+:`)。然后查找以`//`开头并包含`'[Ff]alls?thr`的注释，捕获类似//
Falls through或// just fall thru这样的注释。假定JDK的程序员们遵守Java编码习惯，
在该写注释的地方写下了这些注释，有多少百分比的样例是会掉入下一个分支的？

    ```scala
    import scala.io.Source
    import java.io._

    //download ZipArchive from https://gist.github.com/jmu/ac482a089312fe9c8642
    object JDKScan extends App {
      val tmpDir = System.getProperty("java.io.tmpdir") + "/JDKScanTemp"
      val casePattern = "(case [^:]+:)".r
      val fallPattern = ".*([Ff]alls? thr).*".r

      args match {
        case Array(file) if file.endsWith(".zip") =>
          if (!new ZipArchive().unZip(file, tmpDir)) {
            println("unzip file failed")
          }

          val result = findCase(new File(tmpDir))
          println("case: " + result._1 + " fall through: " + result._2)
          println("percent: "+ (if (result._1 == 0) 0 else 100 * result._2/result._1 + "%"))
        case _ => println("Usage: scala JDKScan <Path to JDK's src.zip>")
        //val str = Source.fromFile(arg).mkString
      }

      def findCase(dir: File):(Int,Int) = {
        var c = 0
        var f = 0

        dir.listFiles.foreach {
          case fi: File if fi.isFile() =>
            for(line <- Source.fromFile(fi).getLines) {
              line.trim match {
                case casePattern(a) =>  c = c + 1
                case fallPattern(a) =>  f = f + 1
                case _ =>
              }
            }

          case d: File if d.isDirectory() =>
            val re = findCase(d) match {
              case (x:Int,y:Int) => c = c + x; f = f + y
            }
          case _ => println("not found")
        }
        (c,f)
      }
    }
    ```

    ```scala
    case: 6966 fall through: 116
    percent: 1%
    ```

2. 利用模式匹配，编写一个swap函数，接受一个整数的对偶，返回对偶的两个组成部件交
换位置的新对偶。

    ```scala
    def swap(t: (Int, Int)) = {
      t match {
        case (x, y) => (y, x)
      }
    }
    ```

3. 利用模式匹配，编写一个swap函数，交换数组中前两个元素的位置，前提条件是数组长
度至少为2。

    ```scala
    def swap(t: Array[Any]) = {
      t match {
      case Array(x, y, last @ _*) => Array(y,x) ++ last
      }
    }
    ```
    ```scala
    scala> swap(Array("ss",BigInt(2),4,5))
    res42: Array[Any] = Array(2, ss, 4, 5)
    ```

4. 添加一个样例类Multiple，作为Item类的子类。举例来说，
`Multiple(10,Article("Blackwell Toster", 29.95))`描述的是10个烤面包机。当然了，
你应该可以在第二个参数的位置接受任何Item，不论是Bundle还是另一个Multiple。扩展`price`函数以应对这个新的样例。

    ```scala
    abstract class Item
    case class Article(description: String, price: Double) extends Item
    case class Bundle(description: String, discount: Double, items: Item*) extends Item
    case class Multiple(amount: Int, it: Item) extends Item

    def price(it: Item): Double = it match {
      case Article(_, p) => p
      case Bundle(_, disc, its @ _*) => its.map(price _).sum - disc
      case Multiple(amt, it) => amt * price(it)
    }

    price(Multiple(5,Bundle("Father's day special", 20.0, 
      Article("Scala for the impatient", 39.95),
      Bundle("Anchor Distillery Sampler", 10.0,
        Article("Old Potrero Straight Rye Whisky", 79.95),
        Article("Junipero Gin", 32.95)))))
    ```

5. 我们可以用列表制作只在一个叶子节点存放值的树。举例来说，列表`((3 8) 2 (5)`描述的是如下这样一棵树：

        
                ●
              / | \
             ●  2  ●
            / \    |
           3   8   5
   

  不过，有些列表元素是数字，而另一些是列表。
在Scala中，你不能拥有异构的列表，因此你必须使用`List[Any]`。编写一个leafSum函数，计算所有叶子节点中的元素之和，用模式匹配来区分数字和列表。

    ```scala
    def leafSum(list: List[Any]): Int = {
      list match {
        case head :: tail => head match {
          case i: Int =>  i + leafSum(tail)
          case l: List[Any] => leafSum(l) + leafSum(tail)
        }
        case _ => 0
      }
    }
    ```

  需要改进的是如何改进`List[Any]`的类型声明，使其只接受`Int`和`List[Int]`
执行结果

        scala> leafSum(List(List(3,8),2,List(5)))
        res52: Int = 18

6. 制作这样的树更好的做法是使用样例类。我们不妨从二叉树开始。

    ```scala
    sealed abstract class BinaryTree
    case class Leaf(value: Int) extends BinaryTree
    case class Node(left: BinaryTree, right: BinaryTree) extends BinaryTree
    ```

  编写一个函数计算所有叶子节点中的元素之和。

    ```scala
    sealed abstract class BinaryTree
    case class Leaf(value: Int) extends BinaryTree
    case class Node(left: BinaryTree, right: BinaryTree) extends BinaryTree

    def leafSum(root: BinaryTree): Int = {
      root match {
        case Leaf(v) => v
        case Node(l, r) => leafSum(l) + leafSum(r)
      }
    }
    ```

7. 扩展前一个练习中的树，使得每个节点可以任意多的后代，并重新实现`leafSum`函数。
第5题中的树应该能够通过下述代码表示：

        Node(Node(Leaf(3), Leaf(8)), Leaf(2), Node(Leaf(5)))

    ```scala
    sealed abstract class BinaryTree
    case class Leaf(value: Int) extends BinaryTree
    case class Node(leaf: BinaryTree*) extends BinaryTree

    def leafSum(root: BinaryTree): Int = {
      root match {
        case Leaf(v) => v
        case Node(first, rest @ _*) => leafSum(first) + rest.map(leafSum _).sum
      }
    }
    ```

  scala能自动处理`rest @ _*`不存在的情况

        scala> leafSum(Node(Node(Leaf(3), Leaf(8)), Leaf(2), Node(Leaf(5))))
        res71: Int = 18

        scala> leafSum(Node(Leaf(1)))
        res72: Int = 1

8. 扩展前一个练习中的树，使得每个非叶子节点除了后代之外，能够存放一个操作符。然
后编写一个eval函数来计算它的值。举例来说：

                +
              / | \
             *  2  -
            / \    |
           3   8   5

  上面这棵树的值为`(3 * 8) + 2 + (-5) = 21`

    ```scala
    sealed abstract class BinaryTree
    case class Leaf(value: Int) extends BinaryTree
    case class Node(leaf: BinaryTree*) extends BinaryTree
    case class EvaluableNode(opt: Char, leaf: BinaryTree*) extends BinaryTree

    def eval(root: BinaryTree): Int = {
      root match {
        case Leaf(v) => v
        case Node(first, rest @ _*) => eval(first) + rest.map(eval _).sum
        case EvaluableNode(opt, all @ _*) => opt match {
          case '+' =>  all.map(eval _).reduce(_ + _)
          case '-' =>  all.map(eval _) match {
                        case s if(s.size == 1) => -s(0)
                        case re => re.reduce(_ - _)
                      }
          case '*' =>  all.map(eval _).reduce(_ * _)
          case '/' =>  all.map(eval _).reduce(_ / _)
        } 
      }
    }
    ```

9. 编写
一个函数，计算`List[Option[Int]]`中所有非None值之和。不得使用match函数。

    ```scala
    def sumNone(list: List[Option[Int]]) = {
      list.foldLeft(0)(_ + _.getOrElse(0))
    }
    ```

10. 编写一个函数，将两个类型为`Double =>
Option[Double]`的函数结合在一起，产生另一个同样类型的函数。如果其中一个函数返回
None,则结合函数也应返回None。例如:

    ```scala
    def f(x: Double) = if (x >= 0) Some(sqrt(x)) else None
    def g(x: Double) = if (x != 1) Some(1 / (x - 1)) else None
    val h = compose(f, g)
    ```

  h(2)将得到Some(1), 而h(1)和h(0)将得到None。

    ```scala
    def compose(f: Double => Option[Double], g: Double => Option[Double]) = {
      (v: Double) => f(v) match { 
          case Some(_) => g(v)
          case None => None
      }
    }
    ```

----
<div align="right">use Scala 2.11.1</div>
