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

to be continued...

----
<div align="right">use Scala 2.11.1</div>

