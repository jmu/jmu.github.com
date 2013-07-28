---
layout: post
title: "Scala in Impatient 习题解答9"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

本章又发现原版正确而中文版翻出来的错误

    val tokens = source.mkString.split("\\S+")
                                          ^ 应为小写

1. 

    ```scala
    import scala.io.Source
    import java.io.PrintWriter

    object Run extends App { 
        val fileName = args(0)
        val sources = Source.fromFile(fileName)
        val out = new PrintWriter(fileName + ".reverse")
        sources.getLines.toArray.reverse.foreach(l => out.println(l))
        out.close
    }
    ```
2. 

    ```scala
    import scala.io.Source
    import java.io.PrintWriter

    object Run extends App { 
        val fileName = args(0)
        val sources = Source.fromFile(fileName)
        var str = ""
        for (chr <- sources) {
            if (chr == '\t') str += "  "
            else str += chr
        }
        sources.close

        val out = new PrintWriter(fileName)
        out.print(str)
        out.close
    }
    ```

3. 用一行完成有奖励

    ```scala
    io.Source.fromFile("filename").mkString.split("\\s+").foreach(s => if (s.length > 12) println(s))
    ```

4. 从只包含浮点数的文本文件中读取数字。

    ```scala
    val floats = io.Source.fromFile("floats.txt").mkString.split("\\s+").map(_.toFloat)

    println(floats.sum)
    println(floats.sum / floats.size)
    println(floats.max)
    println(floats.min)
    ```

5. 答案稍微有点不完美， 第一行应该都是1， 目前不知道除了判断1之外，还有什么优雅的方式
  翻了一下别人的解答， `BigDecimal(2).pow(n)`可以解决这个问题

    ```scala
    import java.io.PrintWriter

    object Run extends App { 
        val fileName = args(0)
        val out = new PrintWriter(fileName)
        for (n <- 0 to 20) out.println(makeStr(n))
        out.close

        def makeStr(n: Int) = {
            val t = math.pow(2, n)
            " " * 10 + math.pow(2, n).toInt + " " * (11-t.toString.size) + 
            math.pow(2,-n) 
        }
    }
    ```

6. 仔细对照了英文版，发现or的字体是一起的。所以认为是搜索整串。

    ```scala
    import scala.io.Source
    import java.io.PrintWriter

    object Run extends App { 
        val fileName = args(0)
        val sources = Source.fromFile(fileName)
        val pat = """"like this, maybe with \" or \\"""".r

        for (line <- sources.getLines) {
            for (mth <- pat.findAllIn(line)) println(mth)
        }

        sources.close
    }
    ```

7. 正则写了好久- -!! 重点在于匹配所有不是浮点数的token。类似`20`这样的在这里我认
为是整数，不是浮点数。

    ```scala
    import scala.io.Source

    object Run extends App { 
        val fileName = args(0)
        val sources = Source.fromFile(fileName)
        val pat = """^((?!^[-]?\d*\.\d+$).)+$""".r

        for (token <- sources.mkString.split("""\s+""")) {
            for (mth <- pat.findAllIn(token)) println(mth)
        }

        sources.close
    }
    ```

  测试用文件abc.txt的内容

        $ cat abc.txt
        sd2.1fsd 843 48,34.locs njv lbxc.74.32 q50ijnls
        n 3.493 , 3490-.a 21 .3 -2.5 988.0 a! (b&^bb) 2..10

  执行结果

        $ scala Run abc.txt
        sd2.1fsd
        843
        48,34.locs
        njv
        lbxc.74.32
        q50ijnls
        n
        ,
        3490-.a
        21
        a!
        (b&^bb)
        2..10

8. 

    ```scala
    val pat = """<img.*?src=["'](.+?)["'].*?>""".r
    for (pat(src) <-
      pat.findAllIn(io.Source.fromURL("http://www.baidu.com").mkString)) {
      println(src)
    }
    ```

  执行结果
  $ scala run.scala 
  http://www.baidu.com/img/bdlogo.gif
  http://www.baidu.com/cache/global/img/gs.gif

9. 编译后调用`scala Run /home .class`

    ```scala
    import java.io._

    object Run extends App {
      val file = new File(args(0))
      val ext = if (args.size > 1) args(1) else ""

      val allDirs = if (file.isDirectory) Iterator[File](file) ++ subdirs(file, ext)
      else subdirs(file, ext)

      allDirs.foreach(dir => for (f <- dir.listFiles(new FileFilter() {
          override def accept(file: File):Boolean = {
            if(ext != null) file.getName.endsWith(ext) else true
          }
        })) println(f))

      def subdirs(dir: File, ext: String): Iterator[File] = {
        val child = dir.listFiles.filter(_.isDirectory)
        child.toIterator ++ child.toIterator.flatMap(subdirs(_, ext))
      }
    }
    ```

10. 这个直接抄书了

    ```scala
    import java.io._

    class Person(val name: String) extends Serializable{
      private var friend: String = ""

      def makeFriend(p: Person){
        friend = p.name
      }
      def currentFriend = friend
    }
    object Run extends App {
      val p1 = new Person("jintao")
      val p2 = new Person("jinping")
      val jmu = new Person("jmu")

      jmu.makeFriend(p2)
      p2.makeFriend(p1)

      val persons = Array[Person](p1, p2, jmu)
      val out = new ObjectOutputStream(new FileOutputStream("/tmp/peoples"))
      out.writeObject(persons)
      out.close

      val in = new ObjectInputStream(new FileInputStream("/tmp/peoples"))
      val savedPersons = in.readObject.asInstanceOf[Array[Person]]

      println(savedPersons(2).name)
      println(savedPersons(2).currentFriend)
    }
    ```

----
<div align="right">use Scala 2.9.1</div>
