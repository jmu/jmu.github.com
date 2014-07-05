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

----
<div align="right">use Scala 2.11.1</div>

