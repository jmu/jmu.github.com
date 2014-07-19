---
layout: post
title: "Scala in Impatient 习题解答20 Actor A3"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

    ```scala
    import scala.actors.Actor
    import scala.util.Random

    case class Work(arr: Array[Int], index: Int, page: Int = 100)

    case class Report(result: Double, index: Int)

    class MainActor extends Actor {
      private var count = 0
      private var result = 0D
      private var total = 0

      def act {
        loop {
          react {
            case Work(arr, index, p) =>
              total = arr.length
              if (total % p != 0)
                count = total / p + 1
              else
                count = total / p

              for (i <- 0 until count) {
                val w = new Worker
                w.start()
                w ! Work(arr.slice(i * p, (i + 1) * p), i, count)
              }

            case Report(re, index) =>
              result = result + re
              count = count - 1
              if (count == 0) {
                println("Result:" + result / total)
                exit()
              }
          }
        }
      }
    }

    class Worker extends Actor {
      def act() {
        react {
          case Work(arr, index, p) => sender ! Report(arr.sum, index)
        }
      }
    }

    object main extends App {

      val master = new MainActor
      master.start()

      val random = new Random(933)
      val arr = new Array[Int](1000000)

      master ! Work(arr.map(a => random.nextInt()), 0)
     // master ! Work((1 to 1000000).toArray, 0)
    }
    ```

2. 

    ```scala
    import java.awt.image.BufferedImage
    import java.io.File
    import javax.imageio.ImageIO
    import javax.imageio.stream.FileImageOutputStream

    import scala.actors.Actor._
    import scala.actors.{Actor, OutputChannel}
    import scala.collection.mutable

    case class Mission(image: BufferedImage, scanSize: Int, workers: Int, startAt: Long)
    case class Work(image: BufferedImage, arr: Array[Int], index: Long, scanSize: Int)
    case class Report(index: Long, timeCost: Long)

    class Worker extends Actor {
      def act() {
        react {
          case Work(image, arr, index, sz) =>
            System.out.println(Thread.currentThread() + "[" + index + "] start")
            val start = System.currentTimeMillis()
            image.setRGB(0, (index * sz).toInt, image.getWidth, sz, arr.map(_ ^ 0xffffff), 0, image.getWidth)
            sender ! Report(index, System.currentTimeMillis() - start)
            act()
          case "Exit" => exit()
        }
      }
    }

    class ImageRevertActor extends Actor {
      var count = 0
      var areaCount = 0
      var startAt: Long = 0L
      var channel: OutputChannel[Any] = null
      var workerArea: Array[Actor] = null
      var runSet: mutable.BitSet = null
      var image: BufferedImage = null
      var scanSize = 0


      def act() {
        loop {
          react {
            case Mission(img, sz, workers, startAt) =>
              channel = sender
              this.startAt = startAt
              this.image = img
              this.scanSize = sz

              areaCount = img.getHeight % sz == 0 match {
                case true => img.getHeight / sz
                case false => img.getHeight / sz + 1
              }

              count = areaCount
              System.out.println("total block: " + count)
              runSet = new mutable.BitSet(count)

              workerArea = new Array[Actor](workers)

              for (i <- 0 until (if (workers > count) count else workers)) {
                val w = new Worker
                w.start()
                workerArea(i) = w
                runSet.add(i)
                w ! Work(img,
                  img.getRGB(0, i * sz, img.getWidth, sz, null, 0, img.getWidth),
                  i, sz)
              }

              System.out.println("init work finish")

            case Report(index, time) =>
              println("[" + index + "]: " + time)
              for (i <- 0 until areaCount) {
                if (!runSet.contains(i)) {
                  runSet.add(i)
                  sender ! Work(image,
                    image.getRGB(0, i * scanSize, image.getWidth, scanSize, null, 0, image.getWidth),
                    i, scanSize)
                }
              }
              count = count - 1
              if (count == 0) {
                channel ! (System.currentTimeMillis() - startAt)
                if (workerArea != null) {
                  workerArea.foreach(_ ! "Exit")
                }
                exit()
              }

          }
        }
      }


    }

    //sample implements, just for comparison
    object SimpleImage {
      def readImage(image: BufferedImage) {
        for (y <- 0 until image.getHeight)
          for (x <- 0 until image.getWidth)
            image.setRGB(x, y, 0xffffff ^ image.getRGB(x, y))
      }
    }

    object ImageMain extends App {
      util.Properties.setProp("scala.time", "true")
      if(args.length != 2) {
        System.out.println("USAGE: scala ImageMain <scanSize> <workerNum>")
        System.exit(1)
      }

      val file = new File("/media/jmu/store/resource/Big_Mandelbrot_set.jpg")

      val image2: BufferedImage = ImageIO.read(file)

      actor {
        val imageActor = new ImageRevertActor
        imageActor.start()
        imageActor ! Mission(image2, args(0).toInt, args(1).toInt, System.currentTimeMillis())
        receive {
          case time: Long =>
            System.out.println("Acter finish :" + time)
            val writing = System.currentTimeMillis()
            val out = new FileImageOutputStream(new File(file.getParent, file.getName + ".actor"))
            ImageIO.write(image2, "JPG", out)
            out.close()
            System.out.println(System.currentTimeMillis - writing)
            exit()
        }
      }
    }
    ```

3. 

    ```scala
    import java.io.File

    import scala.actors.Actor
    import scala.io.Source
    import scala.util.matching.Regex


    case class FileSearch(file: File, regex: Regex, out: Actor)
    case class CountResult(total: Long)

    class DirSearchActor(path: File, regex: Regex) extends Actor {
      private var collect: Actor = null

      def act() {
        collect = new CollectActor
        collect.start()
        println("Start search " + path.getAbsoluteFile)
        search(path)

      }


      private def search(file: File) {
        if (file.canRead) {
          file.listFiles.foreach {
            case f: File if f.isFile && f.canRead =>
              val readActor = new FileReadActor
              readActor.start()
              readActor ! FileSearch(f, regex, collect)
            case d: File if d.isDirectory => search(d)
            case _ =>
          }
        }
      }
    }

    class FileReadActor() extends Actor {
      def act() {
        react {
          case FileSearch(file, regex, out) =>
            println(Thread.currentThread().getName + file.getAbsoluteFile)
            out ! CountResult(Source.fromFile(file).getLines().map(regex.findAllIn(_).size).sum)
        }


      }

    }

    class CollectActor extends Actor {
      var total = 0L

      def act() {
        loop {
          react {
            case CountResult(count) =>
              total = total + count
              println(total)
          }
        }
      }

    }

    object WordCountMain extends App {
      args match {
        case Array(path, regex) =>
          val root = new File(path)
          root match {
            case f: File if f.isDirectory =>
              val actor = new DirSearchActor(f, regex.r)
              actor.start()
            case _ => println("ERROR: not a valid path.")
          }

        case _ => println("Usage: WordCountMain <path> <regex>")
      }
    }
    ```

4. 上题稍加修改即可

    ```scala
    import java.io.File

    import scala.actors.Actor
    import scala.io.Source
    import scala.util.matching.Regex


    case class FileSearch(file: File, regex: Regex, out: Actor)
    case class CountResult(matched: Seq[String])

    class DirSearchActor(path: File, regex: Regex) extends Actor {
      private var collect: Actor = null

      def act() {
        collect = new CollectActor
        collect.start()
        println("Start search " + path.getAbsoluteFile)
        search(path)

      }


      private def search(file: File) {
        if (file.canRead) {
          file.listFiles.foreach {
            case f: File if f.isFile && f.canRead =>
              val readActor = new FileReadActor
              readActor.start()
              readActor ! FileSearch(f, regex, collect)
            case d: File if d.isDirectory => search(d)
            case _ =>
          }
        }
      }

      def over {
        collect ! "Exit"
      }
    }

    class FileReadActor() extends Actor {
      def act() {
        react {
          case FileSearch(file, regex, out) =>
            println(Thread.currentThread().getName + file.getAbsoluteFile)
            out ! CountResult(Source.fromFile(file).getLines().flatMap(regex.findAllIn(_).toSeq).toSeq)
        }


      }

    }

    class CollectActor extends Actor {
      //var total: List[String] = Nil

      def act() {
        loop {
          react {
            case CountResult(seq) =>
              //total = total ++ seq
              seq.foreach(println)

            case "Exit" =>
              exit()
          }
        }
      }

    }

    object WordCountMain extends App {
      args match {
        case Array(path, regex) =>
          val root = new File(path)
          root match {
            case f: File if f.isDirectory =>
              val actor = new DirSearchActor(f, regex.r)
              actor.start()
              Thread.sleep(10000)
              actor.over
            case _ => println("ERROR: not a valid path.")
          }

        case _ => println("Usage: WordCountMain <path> <regex>")
      }
    }
    ```

5. 

    ```scala
    import java.io.File

    import scala.actors.Actor
    import scala.io.Source
    import scala.util.matching.Regex


    case class FileSearch(file: File, regex: Regex, out: Actor)
    case class CountResult(matched: Seq[String], fileName: String)

    class DirSearchActor(path: File, regex: Regex) extends Actor {
      private var collect: Actor = null

      def act() {
        collect = new CollectActor
        collect.start()
        println("Start search " + path.getAbsoluteFile)
        search(path)

      }


      private def search(file: File) {
        if (file.canRead) {
          file.listFiles.foreach {
            case f: File if f.isFile && f.canRead =>
              val readActor = new FileReadActor
              readActor.start()
              readActor ! FileSearch(f, regex, collect)
            case d: File if d.isDirectory => search(d)
            case _ =>
          }
        }
      }

      def over {
        collect ! "Exit"
      }
    }

    class FileReadActor() extends Actor {
      def act() {
        react {
          case FileSearch(file, regex, out) =>
            println(Thread.currentThread().getName + file.getAbsoluteFile)
            out ! CountResult(Source.fromFile(file).getLines().flatMap(regex.findAllIn(_).toSeq).toSeq, file.getName)
        }


      }

    }

    class CollectActor extends Actor {
     var total  = collection.mutable.Map[String, Set[String]]()

      def act() {
        loop {
          react {
            case CountResult(seq, fileName) =>
              seq.foreach { value =>
                total(value) = total.get(value).map(_ + fileName).getOrElse(Set(fileName))
              }

            case "Exit" =>
              println(total)
              exit()
          }
        }
      }

    }

    object WordCountMain extends App {
      args match {
        case Array(path, regex) =>
          val root = new File(path)
          root match {
            case f: File if f.isDirectory =>
              val actor = new DirSearchActor(f, regex.r)
              actor.start()
              Thread.sleep(10000)
              actor.over
            case _ => println("ERROR: not a valid path.")
          }

        case _ => println("Usage: WordCountMain <path> <regex>")
      }
    }
    ```

6. 

    ```scala
    import scala.actors.Actor

    class HundredActor extends Actor {
      def act() {
        while(true) {
          receive {
            case 'Hello => println(Thread.currentThread())
          }
        }
      }
    }

    class HundredActor2 extends Actor {
      def act() {
        loop {
          react {
            case 'Hello => println(Thread.currentThread())
          }
        }
      }
    }

    object HMain extends App {
      (1 to 100).foreach { n =>
        //val a = new HundredActor
        val a = new HundredActor2
        a.start()
        a ! 'Hello
      }
    }
    ```

  可以看到，使用`recevie`的方法启动了100个线程，而`react`的方式只使用4个。

7. 

    ```scala
    import java.io.{IOException, File}

    import scala.actors.{UncaughtException, Exit, AbstractActor, Actor}
    import scala.io.Source
    import scala.util.matching.Regex


    case class FileSearch(file: File, regex: Regex, out: Actor)
    case class CountResult(matched: Seq[String], fileName: String)
    case class Watches(act :Actor)

    class DirSearchActor(path: File, regex: Regex) extends Actor {
      private var collect: Actor = null

      def act() {
        collect = new CollectActor
        collect.start()
        println("Start search " + path.getAbsoluteFile)
        val watcher = new ObserveActor
        watcher.start()
        search(path, watcher)

      }


      private def search(file: File, watcher: ObserveActor) {
        if (file.canRead) {
          file.listFiles.foreach {
            case f: File if f.isFile && f.canRead =>
              val readActor = new FileReadActor
              readActor.start()
              watcher ! Watches(readActor)
              readActor ! FileSearch(f, regex, collect)
            case d: File if d.isDirectory => search(d, watcher)
            case _ =>
          }
        }
      }

      def over {
        collect ! "Exit"
      }
    }

    class FileReadActor() extends Actor {
      def act() {
        react {
          case FileSearch(file, regex, out) =>
            println(Thread.currentThread().getName + file.getAbsoluteFile)
            out ! CountResult(Source.fromFile(file).getLines().flatMap(regex.findAllIn(_).toSeq).toSeq, file.getName)
        }


      }

    }

    class CollectActor extends Actor {
     var total  = collection.mutable.Map[String, Set[String]]()

      def act() {
        loop {
          react {
            case CountResult(seq, fileName) =>
              seq.foreach { value =>
                total(value) = total.get(value).map(_ + fileName).getOrElse(Set(fileName))
              }

            case "Exit" =>
              println(total)
              exit()
          }
        }
      }

    }

    class ObserveActor extends Actor {
      trapExit = true

      def act() {
        while (true) {
          receive {
            case Watches(act) => link(act)
            case Exit(linked, UncaughtException(_, _, _, _, cause: IOException)) =>
              println("[" + linked + "] =>" + cause.getMessage)
            case Exit(linked, reason) => println("[" + linked + "] " + reason)
          }
        }
      }
    }

    object WordCountMain extends App {
      args match {
        case Array(path, regex) =>
          val root = new File(path)
          root match {
            case f: File if f.isDirectory =>
              val actor = new DirSearchActor(f, regex.r)
              actor.start()
              Thread.sleep(10000)
              actor.over
            case _ => println("ERROR: not a valid path.")
          }

        case _ => println("Usage: WordCountMain <path> <regex>")
      }
    }
    ```

  增加 `ObserveActor` 来`link`其他子Actor即可。需要注意的是`link`方法必须由发起actor来调用， 使用发送消息的方式通知`ObserveActor`发起link可以轻松解决此问题。

8. 

    ```scala
    import scala.actors.Actor
    import scala.actors.Actor._

    class LockActor extends Actor {
      def act() {
        receive {
          case "Good" => sender ! "End"
        }
      }
    }

    object LockApp extends App {
      actor {
        new LockActor().start() !? "Good" // lock
        while (true) {
          receive {
            case "End" => exit()
          }
        }
      }
    }
    ```

9. 在练习3基础上测试修改共享变量，估计是电脑速度快，文件量小，很难测试出来。
  故专门写了个小程序，可以直观地测试出来。

    ```scala
    import scala.actors.Actor
    import scala.actors.Actor._

    class PlusActor extends Actor {
      var shareValue = 0L
      private var safeValue = 0L

      def act() {
        (1 to 10000).foreach { a =>
          new MyActor().start() ! "plus"
        }
        while (true) {
          receive {
            case "+" => safeValue = safeValue + 1
            case "Good" =>
              println("share var " + shareValue)
              println("safe var " + safeValue)
              exit()
          }
        }
      }

    }

    class MyActor extends Actor {
      def act() {
        receive {
          case "plus" =>
            val a = sender.asInstanceOf[PlusActor].shareValue
            sender.asInstanceOf[PlusActor].shareValue = a + 1
            sender ! "+"
        }
      }
    }

    object PlusApp extends App {
      val la = new PlusActor
      la.start()
      Thread.sleep(5000)
      la ! "Good"
    }
    ```

  运行结果如下。多运行几次就会发现同步问题。而使用消息传递方式的结果总是正确的。

        share var 9946
        safe var 10000

10. 

    ```scala
    import scala.actors.Actor._
    import scala.actors.{Actor, Channel}
    import scala.util.Random

    case class Work(arr: Array[Int], index: Int, page: Int = 100, result: Channel[Report])

    case class Report(result: Double, index: Int)

    class MainActor extends Actor {
      private var count = 0
      private var result = 0D
      private var total = 0

      def act {
        loop {
          react {
            case Work(arr, index, p, ch) =>
              total = arr.length
              if (total % p != 0)
                count = total / p + 1
              else
                count = total / p

              val channel = new Channel[Report];

              for (i <- 0 until count) {
                val w = new Worker
                w.start()
                w ! Work(arr.slice(i * p, (i + 1) * p), i, count, channel)
              }
              while (true) {
                channel.receive {
                  case Report(re, index) =>
                    result = result + re
                    count = count - 1
                    if (count == 0) {
                      ch ! Report(result / total, 0)
                      exit()
                    }
                }
              }
          }
        }
      }
    }

    class Worker extends Actor {
      def act() {
        react {
          case Work(arr, index, p, ch) => ch ! Report(arr.sum, index)
        }
      }
    }

    object main extends App {
      val random = new Random(933)
      val arr = new Array[Int](1000000)

      actor {
        val channel = new Channel[Report]
        val master = new MainActor
        master.start()
        master ! Work(arr.map(a => random.nextInt()), 0, 100, channel)
        channel.receive {
          case Report(x, _) => println("Result: " + x)
        }
      }
    }
    ```

----
<div align="right">use Scala 2.11.1</div>

