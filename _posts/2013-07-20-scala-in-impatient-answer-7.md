---
layout: post
title: "Scala in Impatient 习题解答7"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

练习的时候出现一个[REPL的问题]({% post_url 2013-07-20-scala-qa %}/#1)


1. 区别是后者

  创建b.scala内容为

        package com {
          package horstmann {
            object A {
              def hi = println("I'm A")
            }
            package impatient {
              object B extends App {
                def hi = A.hi
                hi
              }
            }
          }
        }

  终端上运行

        $ scalac b.scala

        $ scala com.horstmann.impatient.B
        I'm A

  再创建c.scala，写入以下内容

        package com.horstmann.impatient {
          object C extends App {
            B.hi
            A.hi //编译时出错
          }
        }
  编译时找不到A， 说明串联声明时似乎不包含上级的声明的。

        $ scalac c.scala 
        c.scala:4: error: not found: value A
            A.hi
            ^
        one error found

  写成第二种方式，则顺利编译通过

        package com
        package horstmann
        package impatient {
          object C extends App {
            B.hi
            A.hi
          }
        }

        $ scalac c.scala 
        $ scala com.horstmann.impatient.C
        I'm A
        I'm A

2. 这题比较有意思，要写一个让人困惑的代码,还有人提出这种要求。使用不在顶部的com的package。

        //a2.scala
        package com {
          package horstmann {
            package com {
              package horstmann {
                object A {
                  def hi = println("I'm the Ghost A")
                }
              }
            }
          }
        }

        //a.scala
        package com {
          package horstmann {
            object A {
              def hi = println("I'm A")
            }
            package impatient {
              object B extends App {
                def hi = com.horstmann.A.hi
                hi
              }
            }
          }
        }

 先编译a2.scala，然后再编译a.scala，就会看到输出

         $ scala com.horstmann.impatient.B
         I'm the Ghost A

  足够抓狂一阵子了。如果先编译a.scala，会输出"I'm A"。

3. 伪随机

        package random {
          object Random {
            private val a = 1664525
            private val b = 1013904223
            private val n = 32

            private var seed = 0
            private var follow: BigInt = 0
            private var previous: BigInt = 0

            def nextInt(): Int = {
              follow = (previous * a + b) % BigInt(math.pow(2, n).toLong)
              previous = follow

              (follow % Int.MaxValue).intValue
            }

            def nextDouble(): Double = {
              nextInt.toDouble
            }

            def setSeed(newSeed: Int) {
              seed = newSeed
              previous = seed
            }
          }
        }

        object Test extends App {
          val r = random.Random
          r.setSeed(args(0).toInt)
          for (i <- 1 to 10) println(r.nextInt)
          for (i <- 1 to 10) println(r.nextDouble)
        }

4. 直接加函数和变量声明到包中，比如com.a.b.c。这样就跟c下面的的class或者object差
了一个层级。他们实际上是c下面的所有类的共同的上级定义。这样一来就没有了封装性。
而实现上来说估计也比较麻烦。java中是一定要放到某个类/接口中的，怎样让这个类/接口
的方法注入到子包的所有类/对象/特质中？

5. com包下可见。可以扩大函数的可见范围。

6. 

        import java.util.{HashMap => JHashMap}
        import scala.collection.mutable.HashMap

        def transMapValues(javaMap: JHashMap[Any,Any]): HashMap[Any,Any] = {
          val result = new HashMap[Any, Any]
          for(k <- javaMap.keySet.toArray) {
            result += k -> javaMap.get(k)
          }
          result
        }

7. import可以放到任意区域，不错。直接放到方法里，也没有问题。

        def transMapValues(javaMap: JHashMap[Any,Any]): HashMap[Any,Any] = {
          import java.util.{HashMap => JHashMap}
          import scala.collection.mutable.HashMap
          val result = new HashMap[Any, Any]
          for(k <- javaMap.keySet.toArray) {
            result += k -> javaMap.get(k)
          }
          result
        }

8. 引入了java和javax的所有内容。因为Scala会自动覆盖java的同名类，不会有冲突。即
使这样，引入过多的包，也会让人很迷惑。况且Scala的编译已经够慢的了。

        import java._
        import javax._

9. 

        import java.lang.System._

        if (pass != null && "secret" == new String(console().readPassword)) {
          val name = getProperty("user.name")
          out.println("Greetings, %s!", name)
        } 
        else {
          err.println("error")
        }

10. Console,Math, 还有基本类型包装对象，Long,Double,Char,Short等等都被Scala覆盖了。

----
<div align="right">use Scala 2.9.1</div>
